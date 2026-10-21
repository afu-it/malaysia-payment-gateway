# Fasspay API Reference (online checkout)

Use this reference for factual Fasspay e-commerce guidance. Fasspay does not publish a public e-commerce API document. Every fact below is taken from the official **Fasspay for WooCommerce** plugin source, version 1.0.14 (author "Fasspay", WordPress.org profile `fasspay`, last updated 2026-09-01), checked 2026-10-21. Card-present SDKs are in `fasspay-in-person.md`.

Where the plugin does not show something, this file says "Not documented in official sources (checked 2026-10-21)". Confirm those items with Fasspay before production.

## Source URLs

- Plugin page: `https://wordpress.org/plugins/fasspay-for-woocommerce/`
- Plugin source zip: `https://downloads.wordpress.org/plugin/fasspay-for-woocommerce.1.0.14.zip`
- Plugin files used: `includes/class-fasspay-api.php`, `includes/class-fasspay-gateway.php`, `includes/class-fasspay-webhook-handler.php`, `includes/class-fasspay-admin.php`, `fasspay-for-woocommerce.php`, `readme.txt`
- Product page: `https://fasspay.com/productsandservices`

## Integration Surfaces

| Surface | What it is | Reference |
|---|---|---|
| E-commerce API (hosted payment page) | Server creates a payment, customer pays on Fasspay page by card, FPX, DuitNow QR, or e-wallet | this file |
| Fasspay for WooCommerce | Official plugin built on the same API | this file, `account-setup.md` |
| Fasstap SDK | Android tap-on-phone (SoftPOS/MPoC) SDK | `fasspay-in-person.md`, `fasspay-fasstap-sdk.md` |
| Fasstap App-to-App | Your Android app hands off to the Fasstap app | `fasspay-in-person.md`, `fasspay-app-to-app.md` |
| Smart Terminal SDK | Android POS talks to a Soft Space terminal over USB | `fasspay-in-person.md`, `fasspay-smart-terminal.md` |
| Provisioning Web API | Create company, MID, TID, mobile users for Fasstap | `fasspay-in-person.md` |

## Environments

| Environment | API base | Notes |
|---|---|---|
| UAT (sandbox) | `https://wallet-uat.fasspay.com` | `sandboxMode: true`. Branding (logo, theme colour) is ignored in UAT. |
| Production | `https://ecom.fasspay.com` | `sandboxMode: false`. |

The plugin refuses any non-HTTPS URL and sets `sslverify` true. Request timeout used by the plugin: 30 seconds.

## Endpoint Catalogue

All endpoints are `POST`, JSON body, signed with the headers below.

| Method | Path | Purpose | Body shape |
|---|---|---|---|
| POST | `/v3/ecom/webInitiateRequest` | Create a payment and get `paymentURL` | `{"messages": "<JSON string>", "sandboxMode": bool}` |
| POST | `/v3/ecom/checkTransactionStatus` | Requery a payment by `trxRefNo` | flat `{"mid","trxRefNo","sandboxMode"}` |
| POST | `/v3/ecom/auth/void` | Void a transaction (same day, not FPX) | `{"messages": "<JSON string>"}` |
| POST | `/v3/ecom/auth/capture` | Capture a pre-authorised transaction | flat `{"mid","trxRefNo","sandboxMode","amount"?}` |
| POST | `/v3/ecom/whiteLabelUpdate` | Set hosted page logo and theme colour (production only) | `{"messages": "<JSON string>"}` |
| POST | `/v3/ecom/test/connection` | Test credentials and connectivity | `{"messages": "<JSON string>"}` |
| POST | your `backendURL` | Server-to-server callback from Fasspay | see Callback |
| GET | your `responsePageURL` | Browser return after payment | browser only |

Refund initiation: Not documented in official sources (checked 2026-10-21). Refunds are reported to you as callback `statusId` `06`.

The body shape differs per endpoint exactly as the plugin sends it. `checkTransactionStatus` and `capture` are not wrapped in `messages`. Keep the shapes as listed.

## Authentication And Request Signing

Headers sent on every call:

| Header | Value |
|---|---|
| `Content-Type` | `application/json` |
| `Accept` | `application/json` |
| `X-Timestamp` | Unix time in seconds, for example `1761000000` |
| `X-Signature` | `base64( HMAC-SHA256( key = secretKey, data = bodyJson + X-Timestamp ) )` (raw binary digest, then standard base64) |
| `X-Signature-Alg` | `HmacSHA256` |
| `X-Source` | The plugin sends `WOOCOMMERCE`. Whether other values are required is not documented. |

Rules:

- `bodyJson` is the exact request body string you send. Build it once, sign it, send the same bytes.
- The plugin encodes with `JSON_UNESCAPED_SLASHES`, so `/` is not escaped as `\/`. Inside `messages`, the inner JSON is also encoded that way, then embedded as a string.
- `secretKey` is used as the literal UTF-8 string key, exactly as given by Fasspay. The UAT secret looks like base64 but is not decoded.
- The timestamp is appended directly, no separator.
- There is no signature inside the body. Plugin comments say "Envelope without RSA signature (HMAC will be sent in headers)".

Pseudo-code:

```text
inner      = json({ mid, currency, amount, trxRefNo, ... })        // no escaped slashes
body       = json({ "messages": inner, "sandboxMode": true })
ts         = str(unix_seconds())
sig        = base64(hmac_sha256(key=SECRET_KEY, msg=body + ts))
POST base + "/v3/ecom/webInitiateRequest"
  headers: X-Timestamp: ts, X-Signature: sig, X-Signature-Alg: HmacSHA256
  body:    body
```

Node example:

```js
const body = JSON.stringify({ messages: JSON.stringify(messages), sandboxMode });
const ts = Math.floor(Date.now() / 1000).toString();
const sig = crypto.createHmac('sha256', process.env.FASSPAY_SECRET_KEY)
  .update(body + ts).digest('base64');
```

`JSON.stringify` does not escape `/`, matching the plugin.

## Create Payment: webInitiateRequest

Fields inside the `messages` JSON string, as built by `Fasspay_API::web_initiate_request()`:

| Field | Type | Required | Notes |
|---|---|---|---|
| `mid` | string | Yes | Merchant ID for the environment. |
| `currency` | string | Yes | `MYR`. The plugin disables itself for any other store currency. |
| `amount` | integer | Yes | Minor units (sen): `round(amount * 100)`. RM 10.50 is `1050`. |
| `trxRefNo` | string | Yes | Your unique reference per attempt. Plugin format `WC_{orderId}_{unixSeconds}`. |
| `merchantMetadata` | string | Plugin sends | Free text, plugin sends `Payment for order #123`. |
| `backendURL` | string | Yes in plugin | Public HTTPS callback URL. Plugin uses `https://shop/wc-api/fasspay_webhook`. |
| `responsePageURL` | string | Yes in plugin | Browser return URL. |
| `email` | string | Plugin sends | Customer email. |
| `mobileNo` | string | Plugin sends | Digits only, max 15 digits, no `+`. Plugin falls back to `601000000000` when empty. |
| `mobileCountryCode` | string | Plugin sends | Digits only calling code, for example `60`. Empty string if unknown. |
| `transactionMethod` | string | Plugin sends | Always `"100"` in the plugin. Meaning not documented; keep `"100"` to let the hosted page offer all enabled methods. |
| `actionMethod` | integer | Plugin sends | Always `0`. Meaning not documented. |
| `isPreAuth` | boolean | Plugin sends | Always `false`. `true` presumably requests pre-authorisation (status `122`, then capture); not documented. |
| `firstName` | string | Optional | Billing first name. |
| `lastName` | string | Optional | Billing last name. |
| `addressLine1` | string | Optional | |
| `addressLine2` | string or null | Optional | Plugin sends `null` when empty. |
| `city` | string | Optional | |
| `state` | string | Optional | |
| `postcode` | string | Optional | |
| `country` | string | Optional | Plugin hardcodes `MY` (changelog 1.0.12 "bug fix for country handling"). |

Envelope:

```json
{
  "messages": "{\"mid\":\"<MID>\",\"currency\":\"MYR\",\"amount\":1050,\"trxRefNo\":\"ORDER123_1761000000\",\"merchantMetadata\":\"Payment for order #123\",\"backendURL\":\"https://example.com/fasspay/callback\",\"responsePageURL\":\"https://example.com/checkout/return\",\"email\":\"buyer@example.com\",\"mobileNo\":\"60123456789\",\"mobileCountryCode\":\"60\",\"transactionMethod\":\"100\",\"actionMethod\":0,\"isPreAuth\":false,\"firstName\":\"Ali\",\"lastName\":\"Abu\",\"country\":\"MY\"}",
  "sandboxMode": true
}
```

Response (HTTP 200):

| Field | Notes |
|---|---|
| `messages` | JSON string. Decode it to read `paymentURL` (redirect target) and `amount`. |
| `transactionId`, `trxRefNo`, `amount`, `currency` | The plugin stores these if `transactionId` appears at top level; may be absent. |
| `errorCode`, `errorMessage` | Error, may arrive with HTTP 200. |
| `errorDTO.code`, `errorDTO.message` | Alternative error shape, on 200 or non-200. |

If `paymentURL` is missing, treat as failure. Redirect the customer to `paymentURL`.

## Browser Return (responsePageURL)

The customer returns to `responsePageURL` after paying. The plugin does not read any parameters from this return and does not settle on it. The plugin also exposes `/fasspay/redirect?trxRefNo=...&key=...` which only looks up the order and redirects to the order-received page. Never mark paid from the return page; show "processing" until the callback or a requery confirms.

## Callback (backendURL)

Fasspay sends `POST` to `backendURL`.

Headers:

| Header | Notes |
|---|---|
| `X-Signature` | Base64 HMAC-SHA256. The plugin also accepts `X-Fasspay-Signature`. |
| `X-Timestamp` | Unix seconds. |

Verification, from `validate_webhook_signature()`:

1. Read the raw body before any parsing. Reject an empty body with 400.
2. Reject a missing signature with 401.
3. If `X-Timestamp` is present, reject when `abs(now - X-Timestamp) > 300` seconds.
4. `expected = base64(HMAC-SHA256(secretKey, rawBody + X-Timestamp))`. If no timestamp, the plugin signs `rawBody` alone.
5. Compare with constant-time equality (`hash_equals`). Mismatch returns 401.
6. Only then `json_decode`. If the body has `messages` as a string, decode it and merge its keys over the top level.

Recommended: require `X-Timestamp`; the plugin's no-timestamp fallback weakens replay protection.

Callback fields read by the plugin:

| Field | Notes |
|---|---|
| `trxRefNo` | Your reference. Required. |
| `statusId` | Status code, see table. |
| `transactionId` | Fasspay transaction ID. Needed for void. |
| `transactionMethod` | Payment method code, see table. `"0"` is FPX, so do not use a falsy check. |
| `gatewayTransactionId` | Third-party reference number (bank or scheme). |
| `buyerName` | FPX buyer name, when sent. |
| `statusMessage` | Reason text on failure. |
| `errorCode`, `errorMessage` | Error details; an empty `statusId` with these means the request was rejected and no status change should happen. |
| `refundAmount` | On status `06`. The plugin passes it straight to a WooCommerce refund in ringgit, falling back to the order total. Unit not documented; confirm. |
| `amount`, `currency` | Stored by the plugin. Unit in callbacks not documented; confirm against a UAT payment before comparing. |

Response: return HTTP 200 on success. The plugin sends no body. 401 for signature failure, 400 for empty body or bad JSON. Retry behaviour on non-200: Not documented in official sources (checked 2026-10-21).

## Status Codes (e-commerce statusId)

| statusId | Meaning (plugin comments) | Plugin action |
|---|---|---|
| `00` | Success (legacy) | Mark paid |
| `100` | APPROVED | Mark paid |
| `104` | SETTLED | Mark paid |
| `02` | Failed (legacy) | Mark failed |
| `102` | DECLINED | Mark failed |
| `03` | Cancelled (legacy) | Mark cancelled |
| `107` | VOIDED | Mark cancelled |
| `06` | Refunded | Create refund |
| `101` | PROCESSING (pending) | Note only |
| `106` | OPENED (pending from bank) | Note only |
| `111` | PROCESSING_VOID | Note only |
| `123` | PROCESSING_CAPTURE | Note only |
| `122` | AUTHORIZED (pre-auth) | On-hold, funds held |
| `115` | PENDING_APPROVAL, FPX B2B waiting for the issuer bank checker | On-hold |
| empty + `errorCode` | Request rejected | Note only, customer may retry |

Funds-moved statuses: `00`, `100`, `104`, `122`.

## Payment Method Codes (transactionMethod in callbacks)

| Code | Method |
|---|---|
| `0` | FPX |
| `5` | Credit/Debit card |
| `34` | E-wallet |
| `35` | DuitNow QR |

Other codes: Not documented. Bank codes for FPX: Not documented (selection happens on the hosted page).

## Settlement Logic From The Official Plugin

- Idempotency key: `md5(transactionId|statusId|trxRefNo)`; skip if equal to the last processed key.
- Active attempt: the latest stored `trxRefNo`. A callback with a different `trxRefNo` is superseded if its timestamp is older, or if the order is already paid or on-hold.
- Superseded and not funds-moved: record, do not change order status.
- Superseded and funds-moved while funds already recorded: flag "duplicate funds", leave order unchanged, merchant refunds or voids one.
- Superseded and funds-moved with no funds recorded: apply it (late success of an earlier attempt).
- Decline (`02`, `102`) on an order with funds recorded under another attempt: record only.
- Store `transactionId`, `gatewayTransactionId`, `transactionMethod`, and the attempt that moved funds.
- The plugin does not verify callback amount against the order total. Add that check.
- Before initiating, block payment for an already-paid order.

## Requery: checkTransactionStatus

Request body (not wrapped):

```json
{ "mid": "<MID>", "trxRefNo": "ORDER123_1761000000", "sandboxMode": true }
```

Signed with the same headers. Response fields: Not documented in official sources (checked 2026-10-21); the plugin returns the decoded JSON as-is. Expect the same `messages` and `statusId` conventions as other responses, and verify with UAT. Use it when a callback is late or when the return page needs a status.

## Void: auth/void

`messages` fields: `mid`, `trxRefNo`, `transactionId` (sent when known; the plugin requires it).

Response: may be wrapped in `messages`. `statusId` `107` is success. `111` means void in progress; wait for the callback. Anything else is failure with `statusMessage`.

Plugin rules before calling void:

- Only for orders that were paid or on-hold and move to cancelled.
- Not allowed for FPX (`transactionMethod` `0`).
- Same calendar day as the order (UTC date in the plugin).
- Only once per order after a `107`.
- If the secret is empty the plugin logs a critical error; void must be signed.

## Capture: auth/capture

Body (not wrapped): `mid`, `trxRefNo`, `sandboxMode`, optional `amount` in sen. Used after a `122` AUTHORIZED pre-auth. The plugin defines this call but never invokes it, and always sends `isPreAuth: false`. Pre-auth availability: confirm with Fasspay.

## White Label: whiteLabelUpdate

`messages` fields: `mid`, `merchantLogo` (data URI `data:image/png;base64,...`, max 200 KB), `merchantThemeColour` (hex such as `#0055ff`). Empty values tell the backend to clear. Production only; skipped in UAT.

## Test Connection: test/connection

`messages` fields: `mid`, `test: true`. Success when decoded `messages.statusId` is `100` or `00`.

## Amounts, Currency, Limits

- Request `amount` is integer sen. Do not send decimals.
- Currency: `MYR` only (plugin restriction since 1.0.1).
- `mobileNo` max 15 digits.
- Rate limits: Not documented. IP allowlist for callbacks: Not documented. Callback retry schedule: Not documented.

## Sandbox (UAT)

The official plugin embeds shared UAT credentials, published publicly by Fasspay in the plugin source. Sandbox only, never production:

- UAT MID: `800010000000179`
- UAT secret: constant `SANDBOX_SECRET` in `includes/class-fasspay-gateway.php` of the plugin zip.

Test cards, test FPX banks, and test wallets: Not documented in official sources (checked 2026-10-21). Ask Fasspay for UAT test data.

## Common Errors

| Symptom | Likely cause | Fix |
|---|---|---|
| Signature rejected | Re-encoded JSON, escaped slashes, decoded secret, timestamp mismatch | Sign the exact body bytes with the literal secret and the same `X-Timestamp` |
| `paymentURL` missing | Error inside response, or `messages` not decoded | Check `errorCode`/`errorDTO`, decode `messages` string |
| Callback 401 | Wrong environment secret, clock skew over 300 s | Match env secret, sync server clock (NTP) |
| Order paid twice | Re-pay on a paid order | Block initiation when already paid; handle duplicate funds |
| Void refused | FPX, next day, or already voided | Refund path instead (via Fasspay) |
