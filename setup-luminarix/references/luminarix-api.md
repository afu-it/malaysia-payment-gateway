# Luminarix API Reference

Use this reference for factual Luminarix API guidance.

Luminarix publishes no public developer documentation (checked 2026-10-21). Everything below is taken from the official plugin "Luminarix Payment Gateway for WooCommerce" v1.6.0, published by Luminarix Sdn. Bhd. (WordPress.org author `lmnrx`), file `luminarix-woocommerce-gateway.php` and `readme.txt`. Where this file says "plugin", it means that source. Facts the plugin does not show are marked "Not documented in official sources (checked 2026-10-21)". Confirm those with Luminarix onboarding before going live.

## Source URLs

- Plugin page: `https://wordpress.org/plugins/luminarix-payment-gateway-for-woocommerce/`
- Plugin ZIP (v1.6.0): `https://downloads.wordpress.org/plugin/luminarix-payment-gateway-for-woocommerce.1.6.0.zip`
- Plugin metadata API: `https://api.wordpress.org/plugins/info/1.2/?action=plugin_information&request[slug]=luminarix-payment-gateway-for-woocommerce`
- Company site: `https://www.lmnrx.com/`

## Integration Surfaces

- Hosted checkout: deposit returns `checkoutUrl`; redirect the customer there.
- Channel-specific instructions: when no `checkoutUrl` is returned, deposit may return QR, virtual account or bank details to show the customer.
- WooCommerce plugin (official).
- Payment links, recurring billing and other platform plugins (Shopify, Magento, OpenCart, PrestaShop) are mentioned on the marketing site but have no public API or plugin source. Not documented in official sources (checked 2026-10-21).

## Environments

| Item | Value |
|---|---|
| Sandbox API base URL | Supplied by Luminarix onboarding. Not published. |
| Live API base URL | Supplied by Luminarix onboarding. Not published. |
| Environments | Sandbox and live have separate base URLs and separate credentials. |

The plugin builds each URL as `rtrim(base, "/") + "/" + path`, for example `{base}/v1/payments/deposit`. Store the environment on each order so later callbacks are verified with the matching secret.

## Credentials

Per payment method (each Payment System Code), per environment:

| Credential | Notes |
|---|---|
| Payment System Code | Identifies the payment method. Examples in the plugin: `MYR_TNG` (settings hint), `BoostBankDeposit` (code fallback default). Full list not published. |
| Channel Code | Optional. Values not published. |
| Merchant ID | String. |
| Sub-Merchant ID | Numeric (readme: "numeric sub-merchant ID"). Used as the first field of every checksum. |
| Client Secret | HMAC key for request checksums and callback verification. Server-side only. |

The plugin (v1.6.0) lets a merchant add several methods (for example FPX and Touch 'n Go), each with its own Merchant ID, Sub-Merchant ID and Client Secret. A direct integration must look up credentials by the method used for the order.

## Endpoint Catalogue

| Method | Path | Purpose |
|---|---|---|
| POST | `/v1/payments/deposit` | Create a payment (deposit) and get `checkoutUrl` or payment instructions |
| POST | `/v1/payments/transaction-status` | Query a transaction's current status |
| POST | `/v1/payments/refund` | Request a full or partial refund of a transaction |
| POST | `{your callbackUrl}` | Outbound: Luminarix sends signed payment and refund status callbacks |
| GET | `{your returnUrl}?...` | Outbound: customer browser returns with a query string appended |

No other endpoints (cancel, capture, void, tokenisation, recurring, payout, settlement report, balance) appear in official sources. Not documented in official sources (checked 2026-10-21).

## Request Headers (all API calls)

| Header | Value | Notes |
|---|---|---|
| `Content-Type` | `application/json` | Body is JSON. The plugin encodes with unescaped slashes. |
| `Accept` | `application/json` | |
| `X-Signature` | Base64 HMAC-SHA256 checksum | See Signatures. Moved from body to header in plugin 1.4.1. |
| `X-Trace-Id` | UUID v4 | Sent by the plugin per request. Whether required is not documented. |
| `User-Agent` | for example `Luminarix-WooCommerce/1.6.0; https://shop.example/` | Plugin 1.3.4 added a descriptive User-Agent. Use your own descriptive value. |

Timeout: plugin default 30 s, bounded 10 to 120 s.

## Signatures

### Request checksum (outbound to Luminarix)

```
checksum = base64( HMAC_SHA256( key = clientSecret, message = join("|", parts) ) )   // raw digest bytes, then Base64
```

The HMAC output is raw binary, then standard Base64 (not hex). Send it in `X-Signature`.

| Call | `parts` in order |
|---|---|
| Deposit | `subMerchantId`, `orderId`, `amount`, `currency`, `returnUrl`, `callbackUrl` |
| Transaction status | `subMerchantId`, `transactionId`, `orderId`, `amount`, `currency` |
| Refund | `subMerchantId`, `transactionId`, `orderId`, `amount` (refund amount), `currency` |

Notes:
- Use exactly the same strings as in the JSON body (`amount` as `"10.00"`, the exact URLs).
- `merchantId`, `paymentSystemCode`, `description`, `channelCode` and `payer` are not in the deposit string-to-sign.
- The refund body carries `callbackUrl` but it is not in the refund string-to-sign.

Example (Node.js):

```js
import crypto from "node:crypto";
const sign = (secret, parts) =>
  crypto.createHmac("sha256", secret).update(parts.join("|"), "utf8").digest("base64");

const sig = sign(process.env.LUMINARIX_CLIENT_SECRET, [
  subMerchantId, orderId, "10.00", "MYR",
  "https://shop.example/pay/luminarix/return",
  "https://shop.example/pay/luminarix/callback",
]);
```

PHP equivalent (from the plugin): `base64_encode(hash_hmac('sha256', implode('|', $parts), $secret, true))`.

### Callback verification (inbound from Luminarix)

Headers on the callback:

| Header | Meaning |
|---|---|
| `X-Timestamp` | Event time. The plugin parses it with PHP `strtotime`, so it is a date/time string (exact format not documented). |
| `X-Signature` | Base64 HMAC-SHA256 |

```
expected = base64( HMAC_SHA256( key = clientSecret, message = X-Timestamp + "|" + rawBody ) )
valid    = timestamp present AND signature present AND rawBody non-empty
           AND |now - parse(X-Timestamp)| <= tolerance      // plugin default 600 s, minimum 60 s
           AND constantTimeEqual(expected, X-Signature)
```

- `rawBody` is the exact request body bytes, before JSON decoding or sanitising.
- `X-Timestamp` is used as the literal header string in the message, not a re-formatted value.
- Use the Client Secret of the payment method and environment that created the order.

## Deposit (Create Payment)

`POST {base}/v1/payments/deposit`, `X-Signature` = checksum(`subMerchantId|orderId|amount|currency|returnUrl|callbackUrl`).

### Request body

| Field | Type | Required | Notes |
|---|---|---|---|
| `merchantId` | string | yes | Per method and environment. |
| `subMerchantId` | string (numeric) | yes | Per method and environment. |
| `paymentSystemCode` | string | yes | For example `MYR_TNG`. From onboarding. |
| `orderId` | string | yes | Your reference. Must be new for every payment attempt (plugin 1.5.2: "The provider requires a fresh reference for every payment attempt"). Plugin max 255 chars, prefix limited to `[A-Za-z0-9._-]`, format `{prefix}{wcOrderId}-{uuid without dashes}`. |
| `amount` | string | yes | Ringgit, 2 decimals, dot separator, no thousands separator: `"10.00"`. |
| `currency` | string | yes | Uppercase ISO code. Plugin allows only the configured currency, default `MYR`. |
| `description` | string | plugin always sends | Max 255 chars in the plugin. |
| `returnUrl` | string (URL) | yes | Browser return. Keep it free of a query string; Luminarix appends `?...`. |
| `callbackUrl` | string (URL) | yes | Server-to-server status callbacks. |
| `channelCode` | string | no | Sent only when configured. |
| `payer` | object | no | See below. Plugin sends it when "Send billing payer details" is on (default on). |

### `payer` object (empty values are omitted)

| Field | Type | Notes |
|---|---|---|
| `email` | string | |
| `countryCode` | string | The plugin sends the phone country calling code here (default `"60"`), the same value as `countryCallingCode`. Confirm the expected value with Luminarix. |
| `countryCallingCode` | string | Digits, default `"60"`. Required by the deposit API when payer is sent (plugin 1.6.0 changelog). |
| `phoneNumber` | string | Digits only, with the leading country calling code stripped (for example `123456789`). |
| `firstName` | string | |
| `lastName` | string | |
| `name` | string | Full name. |
| `userId` | string | Your customer ID (plugin sends `"0"` for guests). |
| `addresses` | array | One object: `addressLine1`, `addressLine2`, `city`, `state`, `country` (ISO 2-letter in WooCommerce), `zipcode`, `addressType` = `"BILLING"`. |

### Response

Envelope: `{ "success": true, "data": { ... }, "code": "...", "description": "..." }`. The plugin treats the call as failed unless `success` is truthy and `data` is a non-empty object.

`data` fields read by the plugin:

| Field | Notes |
|---|---|
| `transactionId` | Luminarix transaction ID. Store it; needed for status and refund. |
| `orderId` | Echo of your `orderId` (plugin falls back to the sent value). |
| `amount` | Transaction amount. |
| `currency` | Transaction currency. |
| `status` | Initial status, typically `PENDING` (plugin default when absent). |
| `checkoutUrl` | Hosted checkout. Redirect the customer here when present. |
| `bankTransactionReference` | Bank reference, stored and shown. |
| `virtualAccountNumber`, `vaNumber`, `accountNumber`, `bankName` | Transfer instructions when no `checkoutUrl`. |
| `qrString`, `qrCode` | QR instructions when no `checkoutUrl`. |
| `message`, `expiryTime` | Instruction text and expiry. |

If neither `checkoutUrl` nor any instruction field is present, the plugin treats it as an error.

Example request body:

```json
{
  "merchantId": "<MERCHANT_ID>",
  "subMerchantId": "<SUB_MERCHANT_ID>",
  "paymentSystemCode": "<PAYMENT_SYSTEM_CODE>",
  "orderId": "WC-1234-3f2b9c0e8d7a4b6c9e1f2a3b4c5d6e7f",
  "amount": "10.00",
  "currency": "MYR",
  "description": "WooCommerce order 1234",
  "returnUrl": "https://shop.example/wc-api/luminarix_return/",
  "callbackUrl": "https://shop.example/wc-api/luminarix_callback/",
  "payer": { "email": "buyer@example.com", "countryCode": "60", "countryCallingCode": "60", "phoneNumber": "123456789", "name": "Ali Abu" }
}
```

## Return URL (browser)

- Luminarix redirects the customer to `returnUrl` with a query string appended. Parameters the plugin reads: `orderId`, `transactionId`. Other appended parameters are not documented.
- A `returnUrl` that already has `?` produces a malformed double `?` (plugin 1.3.9 fix). Use a path-only URL.
- Never trust status from the return. The plugin checks `orderId`/`transactionId` against stored values (403 on mismatch), then calls transaction-status before showing the result.

## Callback (Webhook)

- Sent as an HTTP POST with a JSON body to the `callbackUrl` given in the deposit or refund request.
- Headers `X-Timestamp` and `X-Signature`; verify as in Signatures.
- Body fields the plugin reads: `orderId`, `transactionId`, `status` (falls back to `transactionStatus`, then `paymentStatus`), plus `amount`, `currency`, `bankTransactionReference` when present. The full payload schema is not documented.
- Acknowledge with HTTP 200 and JSON body `{"status":"SUCCESS"}` (plugin 1.4.4; 1.4.3 used a plain `SUCCESS` body). Reply the same for duplicates.
- Plugin error responses (your choice, not a Luminarix requirement): 400 invalid JSON, 404 order not found, 409 reference mismatch, 401 invalid signature.
- Retry policy, source IPs and delivery order are not documented in official sources (checked 2026-10-21).

Plugin handling sequence (recommended):

1. Read raw body, `X-Timestamp`, `X-Signature`.
2. Find the order by `transactionId`, else by `orderId`.
3. Reject if callback `orderId` or `transactionId` differs from stored values.
4. Verify signature with the secret of the order's method and environment.
5. Event ID = SHA-256(rawBody + "|" + signature); if seen, reply success and stop.
6. Queue the event, reply `{"status":"SUCCESS"}`.
7. Process: if `transactionId` matches a stored refund, update that refund. Else if status is success, call transaction-status and settle from its result (retry 300, 900, 1800, 3600 s on error). Else apply the status.

## Transaction Status (Requery)

`POST {base}/v1/payments/transaction-status`, `X-Signature` = checksum(`subMerchantId|transactionId|orderId|amount|currency`).

| Field | Type | Required | Notes |
|---|---|---|---|
| `merchantId` | string | yes | |
| `subMerchantId` | string | yes | |
| `transactionId` | string | yes | From deposit response. |
| `orderId` | string | yes | Your orderId for this attempt. |
| `amount` | string | yes | Stored transaction amount, 2 decimals. |
| `currency` | string | yes | Stored transaction currency. |

Response: same envelope; `data` carries `status` (or `transactionStatus` / `paymentStatus`), `transactionId`, `orderId`, `amount`, `currency`, `bankTransactionReference`.

Connection test used by the plugin: call transaction-status with dummy `transactionId` `TEST-<unix time>`, `orderId` `TEST-<4 digits>`, amount `1.00`. A `code` of `TRANSACTION_NOT_FOUND`, `OK` or `VALIDATION_FAILED` means the API was reached and the request was processed.

Polling (plugin defaults): every 300 s, orders up to 72 h old, 10 orders per run (max 25), skipping orders already in a terminal status.

## Refund

`POST {base}/v1/payments/refund`, `X-Signature` = checksum(`subMerchantId|transactionId|orderId|amount|currency`), where `amount` is the refund amount.

| Field | Type | Required | Notes |
|---|---|---|---|
| `merchantId` | string | yes | |
| `subMerchantId` | string | yes | |
| `transactionId` | string | yes | Original payment transaction. |
| `orderId` | string | yes | Original payment orderId. |
| `amount` | string | yes | Refund amount, greater than zero, 2 decimals. Partial refunds supported (readme). |
| `currency` | string | yes | |
| `callbackUrl` | string | yes in plugin | Refund status callbacks go here. Not in the string-to-sign. |

Response `data`: `transactionId` (a new refund transaction ID, distinct from the payment), `status` (plugin default `PENDING`). Refund callbacks carry the refund `transactionId`; match them to the refund record, not the original payment (plugin: "Refund callback tracking isolated from the original payment transaction").

Refund limits, windows and per-method support are not documented in official sources (checked 2026-10-21).

## Status Values

| Status | Meaning in the plugin |
|---|---|
| `PENDING`, `ACCEPTED`, `APPROVED` | Not final. Keep the order pending. |
| `SUCCESS` | Paid. Terminal. Confirm with transaction-status before fulfilling. |
| `FAILED` | Failed. Do not apply to an already paid order. |
| `CANCELLED` | Cancelled or expired. Do not apply to an already paid order. |
| `REFUNDED` | Refunded. |

The plugin also normalises aliases (`PAID`, `COMPLETED`, `SUCCESSFUL`, `SUCCEEDED` to SUCCESS; `FAILURE`, `DECLINED`, `REJECTED` to FAILED; `CANCELED`, `EXPIRED` to CANCELLED). Whether Luminarix ever sends the aliases is not documented; handle them defensively.

## Errors, Retries And Rate Limits

| Signal | Meaning and handling (plugin) |
|---|---|
| `success: false` | Error. Show `description`, plus each `errors[].message`. |
| `code: OK` | Success code. |
| `code: TRANSACTION_NOT_FOUND` | Unknown transaction on status. |
| `code: VALIDATION_FAILED` | Request validation failed. |
| HTTP 500/503 with `code` `INTERNAL_ERROR` or `PROVIDER_UNAVAILABLE` | Plugin retries the same request once immediately. For deposits, prefer a status check before retrying to avoid duplicates. |
| HTTP 429 | Rate limited. Respect `Retry-After` seconds (plugin default 300 when absent). Numeric limits not documented. |
| HTTP 403 with HTML mentioning Cloudflare / "you have been blocked" | Cloudflare blocked your server. Ask Luminarix to allowlist the server IP. Log the `cf-ray` header. |
| Non-JSON body | Treat as error (`luminarix_invalid_json` in the plugin). |

Full error code list: not documented in official sources (checked 2026-10-21).

## Settlement Checklist

Mark paid only when all hold:

1. Callback signature valid and timestamp in window (or the result came from your own transaction-status call).
2. transaction-status returns `SUCCESS` for the stored `transactionId` and `orderId`.
3. `amount` and `currency` equal what you stored.
4. Environment and payment method match the order.
5. The local order is still pending, or already paid (no-op).

Never move a paid order back to pending, failed or cancelled because of a late event.

## Not Documented (checked 2026-10-21)

Base URLs, the Payment System Code and channel code list (FPX, DuitNow QR, cards, e-wallet codes), callback retry policy and source IPs, `X-Timestamp` format, full callback schema, error code list, rate limit numbers, sandbox test cards or banks, refund rules, payment expiry, recurring billing and payment link APIs, settlement and payout reports, and the exact meaning of `payer.countryCode`.
