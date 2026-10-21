# Atome API Reference (Online, Open API v2)

Use this reference for factual Atome Buy Now Pay Later (BNPL) API guidance. All facts below come from the official Atome Open API v2 spec and the Atome Developer Portal, checked 2026-10-21. Verify credentials, enabled country, callback secret, and contract limits with your Atome Account Manager.

## Source URLs

- `https://doc.apaylater.com/v2/` (Redoc page; renders the spec below)
- `https://doc.apaylater.com/v2/swagger.yaml` (OpenAPI 3.0 spec, title "Atome Payment API v2 (For Online Integration)", the primary source)
- `https://doc.apaylater.com/v1/` (offline / POS integration, not covered here)
- `https://aaclub-sg-prod.s3.amazonaws.com/public/open-api/atome_open_api_flowchart.jpg` (flow diagram linked from the spec)
- `https://partner.apaylater.com/docs/Help/FAQ`
- `https://partner.apaylater.com/docs/Help/CustomerPaymentProcess`
- `https://github.com/Atome-FE` (official app SDK demo repos)

## Scope

- V2 API is for online integration. V1 API is for offline (POS) integration.
- One API serves Singapore, Malaysia, Indonesia, Thailand, and the Philippines. Malaysia uses country code `MY` and currency `MYR`.
- **Payment** is the core object. It tracks status, cancellation, and refunds.
- Platform plugins exist for Shopify, WooCommerce, Magento, PrestaShop, OpenCart, EasyStore, Wix, and Salesforce. Use the native API below when the store platform is not on that list. Plugin setup, plugin refunds, and app SDK calls are in `references/atome-plugins-sdk.md`.

## Endpoint Catalogue

Every operation in the official specs (`https://doc.apaylater.com/v2/swagger.yaml` and `https://doc.apaylater.com/v1/swagger.yaml`), checked 2026-10-21. All paths are relative to the base URL; all use HTTP Basic auth except the callback (Atome calls you).

| # | Method | Path | operationId | Purpose | Spec |
| --- | --- | --- | --- | --- | --- |
| 1 | POST | `/auth` | (none) | Check Configuration: test credentials, country, and callback reachability. | v2, v1 |
| 2 | POST | `/payments` | `createPayment` | Create new payment; returns `redirectUrl` and `appPaymentUrl`. Idempotent by `referenceId`. | v2, v1 |
| 3 | GET | `/payments/{referenceId}` | `getPayment` | Get payment information (status requery). | v2, v1 |
| 4 | POST | `/payments/{referenceId}/cancel` | `cancelPayment` | Cancel an unpaid payment. | v2, v1 |
| 5 | POST | `/payments/{referenceId}/refund` | `refundPayment` | Full or partial refund, synchronous, idempotent by `refundId`. | v2, v1 |
| 6 | POST | `<your callbackUrl>` | `sendCallback` | Status-change callback sent by Atome to the merchant, signed with `X-Signature`. | v2, v1 |
| 7 | POST | `/payments/{referenceId}/barcode_pay` | (none) | V1 offline only: POS scans the shopper's Atome barcode (`barcodeContent`, prefix `ATME`). Not for online checkout. | v1 only |

Nothing else is documented: no list/search payments, no capture or void, no recurring or token endpoints, no payout or settlement API, no webhook management API (checked 2026-10-21).

Plugin-only call (not in either spec, seen in the official WooCommerce plugin source v6.9.0): `GET /variables/{COUNTRY}` without auth returns country display settings, for example `{"country":"MY","minSpend":1000,...}` with `minSpend` in sen. Treat it as an internal plugin helper; do not depend on it in a native integration.

## Environments

| Environment | Base URL |
| --- | --- |
| Test | `https://api.apaylater.net/v2` |
| Production | `https://api.apaylater.com/v2` |

- TLS 1.2 or greater is required for all communication.
- Test and production use different credentials. The developer portal FAQ says a payment page that cannot be completed in production is usually caused by using test endpoints or test credentials.
- Hosted payment pages seen in spec examples are on `gateway.apaylater.net` (test) and `gateway.apaylater.com` (production). Always use the `redirectUrl` returned by the API; never build it yourself.

## Authentication

HTTP Basic auth on every request:

```text
Authorization: Basic base64("<ATOME_API_USERNAME>:<ATOME_API_PASSWORD>")
Content-Type: application/json
```

- The username and password are issued by your Atome Account Manager. Plugin docs label them "API Key" and "API Password".
- A wrong credential returns HTTP 401 with code `UNAUTHORIZED`.
- Keep both values server-side only.

## Checkout Flow

From the spec flow diagram:

1. Customer places an order on the merchant site or app.
2. Merchant server calls `POST /payments` (create new payment).
3. Atome returns the payment object.
4. Merchant sends the customer to `redirectUrl` (web) or `appPaymentUrl` (merchant app webview / SDK).
5. Customer jumps to the Atome site or app.
6. Customer pays (first instalment) on Atome.
7. Atome server sends a callback to the merchant `callbackUrl`.
8. Merchant server returns 200 to the callback.
9. Atome site gets the payment result.
10. Atome redirects the customer to `paymentResultUrl` (success) or `paymentCancelUrl` (failed or abandoned).

If creating the payment returns an error, stop; do not redirect.

## Check Configuration (one-time connectivity test)

```text
POST {base}/auth
```

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `countryCode` | string | yes | `SG`, `MY`, `ID`, `TH`, `PH`. Use `MY` for Malaysia. |
| `callbackUrl` | string | no | Atome sends a `POST` here and treats any 20x as success. Use it to confirm your firewall lets Atome in. |

Responses:

| HTTP | Body | Meaning |
| --- | --- | --- |
| 200 | (no schema) | Credentials valid; callback (if sent) answered 20x. |
| 401 | `{"code":"UNAUTHORIZED","message":"Username or password wrong."}` | Bad Basic auth. |
| 400 | `{"code":"CALLBACK_FAILED","message":"Request to callback failed or received a non-200 response. Check your server settings."}` | Atome could not reach your `callbackUrl`. |
| 400 | `{"code":"COUNTRY_NOT_SUPPORTED",...}` | `countryCode` not enabled for these credentials. |

The FAQ says you only need to call this once, not before every payment. The official WooCommerce plugin also sends `platform` and `pluginConfiguration` on `/auth`; those fields are not in the spec, so a native integration should not send them.

## Create Payment

```text
POST {base}/payments
```

Idempotent by `referenceId`. A payment that is not paid is auto-cancelled after **12 hours** (default and maximum).

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `referenceId` | string | see note | Your unique payment id; regex `[0-9a-zA-Z-_]{1,40}`. Used for idempotency and as the path key for every later call. Not listed in the spec `required` array, but every later endpoint needs it; always send it. |
| `currency` | string | yes | ISO 4217. `MYR` for Malaysia. Must match the merchant's country. |
| `amount` | integer | yes | Smallest currency unit. `MYR` is in sen: `1234` = RM12.34. Minimum **RM10.00** (`1000`). |
| `callbackUrl` | string | yes | Server-to-server status-change notification URL. |
| `paymentResultUrl` | string | yes | Browser redirect for successful transactions. |
| `paymentCancelUrl` | string | no | Browser redirect when the payment fails or the customer leaves ("Back to Merchant"). |
| `merchantReferenceId` | string | no | Your order id; regex `[0-9a-zA-Z-_.]{1,50}`. Shown in the settlement report and merchant portal as "E-commerce Platform Order ID". |
| `customerInfo` | object | yes | `mobileNumber` (required, E.164, e.g. `+60123456789`), `fullName`, `email`. Used to pre-fill the Atome form. |
| `shippingAddress` | object | yes | `countryCode` (required, `MY`), `lines` (required, array of strings), `postCode` (required), `selfPickUp` (boolean; true means the address is a pickup point). |
| `billingAddress` | object | no | `countryCode`, `lines`, `postCode` (all required when the object is sent). |
| `taxAmount` | integer | no | Tax part of `amount`, same unit. |
| `shippingAmount` | integer | no | Shipping part of `amount`, same unit. |
| `originalAmount` | integer | no | Amount before discount, same unit. |
| `voucherCode` | string | no | Voucher applied. |
| `items` | array | yes | Each item: `itemId` (required, max 50), `name` (required, max 250), `quantity` (required, integer), `price` (required, integer, same unit), `variationName` (max 100), `originalPrice` (integer), `categories` (array of strings, max 100 each). |
| `additionalInfo` | object | no | `custom` object for merchant-defined string properties. |
| `expirationTime` | integer | no | Seconds before the payment expires. Minimum `300` (5 minutes), default and maximum 12 hours. |

Missing-field error codes that are specific to V2: `CUSTOMER_MOBILE_NUMBER_MISSING`, `CUSTOMER_SHIPPING_ADDRESS_MISSING`, `ITEM_INFO_MISSING`, `ITEM_NAME_MISSING`, `ITEM_QUANTITY_MISSING`, `ITEM_PRICE_MISSING`. Also `CALLBACK_URL_MISSING`, `INVALID_CALLBACK_URL`, `RESULT_URL_MISSING`.

Malaysia example body (placeholders):

```json
{
  "referenceId": "ORD-10023-1",
  "currency": "MYR",
  "amount": 15900,
  "callbackUrl": "https://api.example.my/atome/callback",
  "paymentResultUrl": "https://www.example.my/orders/10023/result",
  "paymentCancelUrl": "https://www.example.my/checkout",
  "merchantReferenceId": "10023",
  "customerInfo": { "mobileNumber": "+60123456789", "fullName": "Test Buyer", "email": "buyer@example.my" },
  "shippingAddress": { "countryCode": "MY", "lines": ["1 Jalan Contoh", "Kuala Lumpur"], "postCode": "50000" },
  "items": [ { "itemId": "SKU-1", "name": "Example item", "quantity": 1, "price": 15900 } ]
}
```

Success `200` returns the payment object (below) with `status: "PROCESSING"`, `redirectUrl`, and `appPaymentUrl`. The spec says a 200 here also triggers a callback with the latest payment data. Errors: `400 AMOUNT_NOT_VALID`, `CURRENCY_NOT_SUPPORTED`, `INVALID_FORMAT_REFERENCE_ID`, `INVALID_CALLBACK_URL`, plus the missing-field codes above.

### Nested request objects (full field tables)

`customerInfo`:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `mobileNumber` | string | yes | E.164, for example `+60123456789`. Missing gives `CUSTOMER_MOBILE_NUMBER_MISSING`. |
| `fullName` | string | no | Pre-fills the Atome form. |
| `email` | string | no | Pre-fills the Atome form. |

`shippingAddress` (missing gives `CUSTOMER_SHIPPING_ADDRESS_MISSING`):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `countryCode` | string | yes | Enum `SG`, `MY`, `ID`, `TH`, `PH`. |
| `lines` | array of string | yes | Address lines. |
| `postCode` | string | yes | Send as a string (keeps leading zeros). |
| `selfPickUp` | boolean | no | `true` means the address is a pickup point. |

`billingAddress` (optional object; if sent, all three fields are required): `countryCode` (same enum), `lines` (array of string), `postCode` (string). No `selfPickUp`.

`items[]` (missing array gives `ITEM_INFO_MISSING`):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `itemId` | string | yes | Product or SKU id, max 50 chars. |
| `name` | string | yes | Max 250 chars. Missing gives `ITEM_NAME_MISSING`. |
| `quantity` | integer | yes | Missing gives `ITEM_QUANTITY_MISSING`. |
| `price` | integer | yes | Selling price, same unit as `amount`. Missing gives `ITEM_PRICE_MISSING`. |
| `variationName` | string | no | Max 100 chars. |
| `originalPrice` | integer | no | Price before discount, same unit. |
| `categories` | array of string | no | Max 100 chars each. |

`additionalInfo.custom`: free-form object of merchant-defined string properties (`propertyName*: string`).

The spec does not say that `amount` must equal the sum of items plus shipping plus tax; the spec example does not add up exactly either. Not documented in official docs (checked 2026-10-21); send accurate values anyway.

### Create payment response (200)

Returns the full payment object (see Payment Object). Spec example (SGD):

```json
{
  "referenceId": "P1293201980299030",
  "currency": "SGD",
  "amount": 12010,
  "refundableAmount": 0,
  "status": "PROCESSING",
  "paymentTransaction": null,
  "refundTransactions": [],
  "merchantReferenceId": "P1293201980299030",
  "redirectUrl": "https://gateway.apaylater.com/payments/hCTwVplvXTXlyqpuUogb",
  "appPaymentUrl": "https://app.apaylater.com/entry?q=<base64>"
}
```

`refundableAmount` is `0` while `PROCESSING`. The official WooCommerce plugin treats a create as successful only when HTTP is 200 and `status` is `PROCESSING`.

### redirectUrl vs appPaymentUrl

- `redirectUrl`: use in desktop or mobile browsers. On mobile browsers the Atome page shows "pay using app" and "continue in browser".
- `appPaymentUrl`: use inside your app's in-app browser with Atome's SDK. It shows only "continue in browser".
- In an app webview, enable `localStorage` and `sessionStorage` (Android `setDomStorageEnabled(true)`; iOS `WKWebsiteDataStore`).
- Developer portal FAQ: in Malaysia, payment is done in the Atome app ("Pay in App"); "Continue in Browser" is not listed for MY.

## Payment Object

| Field | Type | Notes |
| --- | --- | --- |
| `referenceId` | string | Your id. |
| `currency` | string | `MYR` for Malaysia. |
| `amount` | integer | Smallest unit (sen). |
| `refundableAmount` | integer | Same unit; always `<= amount`. |
| `status` | string | `PROCESSING`, `PAID`, `FAILED`, `REFUNDED`, `CANCELLED`. |
| `paymentTransaction` | object or null | After payment: `orderId` (Atome user order id), `transactionId`, `createAt` (timestamp string, ms in examples), `tenor` (number of instalments). |
| `refundTransactions` | array | Latest first: `refundId`, `transactionId`, `amount`, `createAt`. |
| `merchantReferenceId` | string | Your order id. |
| `redirectUrl` | string | Frontend URL to continue payment. |
| `appPaymentUrl` | string | Deep link for the Atome app via SDK. |
| `additionalInfo` | object | `custom` properties returned by Atome. |

`paymentTransaction` (null until paid):

| Field | Type | Notes |
| --- | --- | --- |
| `orderId` | string | Atome user order id, for example `O0498841483`. |
| `transactionId` | string | Unique payment transaction id, for example `MTX0508001109120`. |
| `createAt` | string | Order creation time as a timestamp string (examples are epoch milliseconds, for example `"1605085352000"`). |
| `tenor` | integer | Number of instalments. |

`refundTransactions[]` (latest first):

| Field | Type | Notes |
| --- | --- | --- |
| `refundId` | string | Your refund id, `[0-9a-zA-Z-_]{1,40}`. |
| `transactionId` | string | Unique refund transaction id. |
| `amount` | integer | Refund amount, same unit as the payment. |
| `createAt` | string | Refund creation timestamp string. |

`paymentInfo.shippingInfo` (schema defined in the spec but not used by any documented endpoint; fields for reference only): `shippingCompany`, `shippingMethod`, `trackingNumber`, `trackingUrl`, `returnShippingCompany`, `returnTrackingNumber`, `returnTrackingUrl` (all strings; `shippingInfo` required inside `paymentInfo`). Do not send it unless your Account Manager asks.

## Payment States

| Status | Meaning |
| --- | --- |
| `PROCESSING` | Initial. Customer has not paid anything yet. |
| `PAID` | Atome received the first instalment. Settle the order. |
| `FAILED` | Atome rejected: fraud/risk decline, credit limit too low, or payment method (down payment card) rejected. |
| `CANCELLED` | Abandoned (no payment after 12 hours or your `expirationTime`), or cancelled by you via the cancel API. No refund needed. |
| `REFUNDED` | Fully refunded. |

Transitions: `PROCESSING -> PAID`, `PROCESSING -> CANCELLED`, `PROCESSING -> FAILED`, `PAID -> REFUNDED`.

BNPL rules that affect your settlement logic:

- `PAID` stays `PAID` even if the customer later misses the 2nd or 3rd instalment. Collection risk is Atome's; do not reverse orders for missed instalments.
- A partial refund keeps status `PAID`; only a full refund moves it to `REFUNDED`.
- Common failure causes per FAQ: amount above the customer's Atome credit limit, Atome risk decline, or the customer's bank rejecting the down payment (first one third of the total).

## Get Payment Information (status requery)

```text
GET {base}/payments/{referenceId}
```

Returns the payment object. `404 PAYMENT_NOT_FOUND` if unknown. This is the only way to learn the status after a callback, because the callback body carries only `referenceId`.

## Cancel Payment

```text
POST {base}/payments/{referenceId}/cancel
```

- Only before the customer has paid. After cancel, the payment can never be paid.
- If your checkout has its own expiry (stock hold, booking hold), you must call cancel so Atome cannot take payment after you released the order.
- Success `200` returns the payment object with `status: "CANCELLED"`. Errors: `400 PAYMENT_NOT_CANCELLABLE` ("Customer has paid this payment."), `404 PAYMENT_NOT_FOUND`.
- If cancel returns `PAYMENT_NOT_CANCELLABLE`, requery: the customer paid, so either fulfil or refund.

## Refund Payment

```text
POST {base}/payments/{referenceId}/refund
```

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `refundAmount` | integer | yes | Same currency and unit as the payment (sen for MYR). Must be `<= refundableAmount`. |
| `refundId` | string | no (strongly recommended) | Your refund id, regex `[0-9a-zA-Z-_]{1,40}`. Idempotency key: a repeated `refundId` does not refund twice; a new `refundId` refunds again. |

- Synchronous: the response arrives after the refund completes, with the updated payment object.
- Multiple partial refunds are allowed; total refunded must not exceed the paid amount.
- Refund any time after `PAID`; Atome has no time limit (apply your own policy).
- Refunds are merchant-initiated only.
- Refund is refused if the customer's Atome account is inactive.
- Errors: `400 AMOUNT_NOT_VALID` ("Refund amount must be not larger than refundableAmount"), `UNABLE_TO_REFUND`, `INVALID_FORMAT_REFUND_ID`, `404 PAYMENT_NOT_FOUND`.

Idempotency example from the spec:

| | 1st refund | 2nd refund | 3rd refund |
| --- | --- | --- | --- |
| `refundId` | `R111` | `R111` | `R222` |
| Atome | Refunds | No refund | Refunds |

Request example: `{"refundId": "R13141232", "refundAmount": 6000}`.

Response `200` (spec example after two partial refunds that total the payment, SGD):

```json
{
  "referenceId": "P1293201980299030",
  "currency": "SGD",
  "amount": 12010,
  "refundableAmount": 0,
  "paymentTransaction": {"orderId": "O0498841483", "transactionId": "MTX0508001109120", "createAt": "1605085352000"},
  "refundTransactions": [
    {"refundId": "R13141232", "transactionId": "MTX0508000859110", "amount": 6000, "createAt": "1605085552000"},
    {"refundId": "R13141233", "transactionId": "MTX0508000859121", "amount": 6010, "createAt": "1605085120000"}
  ],
  "status": "REFUNDED",
  "merchantReferenceId": "P1293201980299030",
  "redirectUrl": "https://gateway.apaylater.net/payment?token=<token>",
  "appPaymentUrl": "https://app.apaylater.com/entry?q=<base64>"
}
```

Note: the official WooCommerce plugin sends only `refundAmount` (no `refundId`), so a retried plugin refund is not idempotent. A native integration should always send `refundId`.

## Callback

Atome `POST`s JSON to your `callbackUrl` when payment status changes.

Body:

```json
{ "referenceId": "ORD-10023-1" }
```

- The body contains only `referenceId` as the status-change indicator. Call `GET /payments/{referenceId}` to get status and amounts.
- Atome says to rely on this server-to-server callback, not on the browser redirect to `paymentResultUrl`.
- Return HTTP `200`. Atome records non-200 responses.
- Retries (developer portal FAQ): Atome retries 5 times over 5 hours, then emails the merchant about the callback failure.
- A successful create (200) also triggers a callback, so expect a callback while the payment is still `PROCESSING`. Ignore it after the requery (the official WooCommerce plugin does exactly this: "Payment is still processing, ignored.").

Callback request:

| Part | Name | Type | Notes |
| --- | --- | --- | --- |
| Method | `POST` | | To the exact `callbackUrl` you sent on create. |
| Header | `Content-Type` | string | `application/json`. |
| Header | `X-Signature` | string | HMAC-SHA256 over the raw body (see below). |
| Body | `referenceId` | string | `[0-9a-zA-Z-_]{1,40}`. Your payment id. |
| Response | HTTP `200` | | Any body. Non-200 is recorded and retried. |

Pattern from the official WooCommerce plugin source (v6.9.0): it puts the id in the callback URL query string (`?wc-api=ATOME&referenceId=<id>`), reads `referenceId` from the query, calls `GET /payments/{referenceId}`, and maps `PAID` to paid, `CANCELLED`/`FAILED`/`REFUNDED` to the matching order status, and ignores `PROCESSING`. It only moves orders that are still pending, which makes duplicate callbacks harmless. The plugin does not verify `X-Signature`, so it gives no answer on the signature encoding.

### Callback signature (HMAC-SHA256)

Documented by the spec:

- Header: `X-Signature`.
- Algorithm: HMAC-SHA256.
- Input: the **raw HTTP request body** (exact bytes received).
- Key: a shared secret.
- The spec says partners who want to verify must request "confirmation/details" from the Atome Account Manager.

Not documented in official docs (checked 2026-10-21): where the shared secret is issued, the output encoding (hex vs Base64, case), and whether the secret is the API password or a separate value. Get all three from your Account Manager before coding the comparison.

Pseudo-code (confirm encoding first):

```text
raw = request.raw_body_bytes            # before any JSON parsing
sig = request.headers["X-Signature"]
expected = HMAC_SHA256(key=ATOME_CALLBACK_SECRET, message=raw)
expected_str = encode(expected)         # hex or base64, as confirmed by Atome
if not constant_time_equal(expected_str, sig): return 401
ref = json.parse(raw).referenceId
payment = GET {base}/payments/{ref}     # authoritative state
settle_idempotently(payment)
return 200
```

Defence in depth even with HMAC: the callback carries no status or amount, so a forged callback can at most trigger a requery. Always settle from the authenticated `GET` response, never from the callback body.

## Production IP Allowlist (Atome outbound, for your callback endpoint)

Malaysia:

- `47.254.230.20`
- `47.254.238.153`
- `47.254.247.54` (test environment)

Use allowlisting as network hardening, not as a replacement for the requery.

## Amount Rules

- Integer, smallest currency unit. MYR is sen; never send decimals.
- Spec minimum for MYR: RM10.00 (`1000`).
- Maximum: not documented in official docs (checked 2026-10-21). It depends on the customer's credit limit and your merchant contract; out-of-range amounts return `AMOUNT_NOT_VALID`.
- The developer portal customer-process page says plugins show Atome only above MYR50; that is a plugin display rule, not the API minimum.
- `taxAmount`, `shippingAmount`, `originalAmount`, item `price`, `refundAmount` all use the same unit.
- The official WooCommerce plugin converts ringgit to sen with `round(value * 100)` for MY and hard-codes a MY minimum spend of RM10; its `Cancel Timeout` setting is 10 to 720 minutes and is sent as `expirationTime = minutes * 60`.
- The spec's `AMOUNT_NOT_VALID` example message is "Atome only support amount between 30 and 500 SGD", so the message may state your contracted range; still branch on `code`.

## Enums

| Enum | Values | Source |
| --- | --- | --- |
| Payment `status` | `PROCESSING`, `PAID`, `FAILED`, `REFUNDED`, `CANCELLED` | v2 spec `payment.status` |
| `currency` | `SGD`, `MYR`, `IDR`, `THB`, `PHP` (must match merchant country) | v2 spec |
| `countryCode` (auth, addresses) | `SG`, `MY`, `ID`, `TH`, `PH` | v2 spec |
| Minimum `amount` per currency | SGD 150 (S$1.50), MYR 1000 (RM10.00), IDR 50000, THB 2000 (20.00), PHP 10000 (100.00) | v2 spec `amount` |
| `errorResponse.code` (schema enum) | `PAYMENT_NOT_FOUND`, `AMOUNT_NOT_VALID`, `CURRENCY_NOT_SUPPORTED`, `INVALID_FORMAT_REFERENCE_ID`, `INVALID_CALLBACK_URL`, `PAYMENT_NOT_CANCELLABLE`, `UNABLE_TO_REFUND`, `CALLBACK_FAILED` | v2 spec; the Error code table below is the longer list |

Bank, card, or payment-method codes: none. Atome is a single BNPL method; the shopper chooses how to pay the down payment inside Atome. Not documented in official docs (checked 2026-10-21).

## Error Codes

All errors return `{ "code": "...", "message": "..." }`. Branch on `code`; `message` is for humans only. Full table from the spec "Error code" section (messages verbatim):

| HTTP | Code | Message | Remarks |
| --- | --- | --- | --- |
| 401 | `UNAUTHORIZED` | You are not authorized to access this page | `/auth` example message: "Username or password wrong." |
| 400 | `INVALID_INPUT` | Invalid Input | |
| 400 | `INVALID_FORMAT_REFERENCE_ID` | The reference Id is in invalid format | |
| 400 | `INVALID_FORMAT_REFUND_ID` | The refund Id is in invalid format | |
| 400 | `INVALID_FORMAT_MERCHANT_REFERENCE_ID` | The merchant reference Id is in invalid format | |
| 400 | `COUNTRY_NOT_SUPPORTED` | Country Not Supported | |
| 400 | `CURRENCY_NOT_SUPPORTED` | Atome does not support currency. | |
| 400 | `UNABLE_TO_REFUND` | Payment is not allowed to refund | For example inactive customer account. |
| 400 | `PAYMENT_NOT_CANCELLABLE` | Customer has paid this payment. | Requery, then fulfil or refund. |
| 400 | `AMOUNT_NOT_VALID` | Amount outside range | Also refund above `refundableAmount`. |
| 400 | `CUSTOMER_MOBILE_NUMBER_MISSING` | Your request is missing customer mobile number | V2 only |
| 400 | `CUSTOMER_SHIPPING_ADDRESS_MISSING` | Your request is missing customer shipping address | V2 only |
| 400 | `ITEM_INFO_MISSING` | Your request is missing item information | V2 only |
| 400 | `ITEM_NAME_MISSING` | Your request is missing item name | V2 only |
| 400 | `ITEM_QUANTITY_MISSING` | Your request is missing item quantity | V2 only |
| 400 | `ITEM_PRICE_MISSING` | Your request is missing item price | V2 only |
| 400 | `CALLBACK_URL_MISSING` | Your request is missing callback url | |
| 400 | `INVALID_CALLBACK_URL` | Your request contains invalid callback url | |
| 400 | `PARAMETER_ERROR` | depends on what kind of error | |
| 400 | `RESULT_URL_MISSING` | Your request is missing payment result url | |
| 400 | `BARCODE_INVALID` | Barcode is Invalid. | V1 offline only |
| 400 | `IMPROPER_OPERATION` | This operation is not appropriate. | |
| 400 | `CALLBACK_FAILED` | Request to callback failed or received a non-200 response. Check your server settings. | `/auth` only |
| 404 | `PAYMENT_NOT_FOUND` | The requested payment is not found | Endpoint examples: "Unable to find payment with this referenceId." |
| 500 | `ERROR` | Server error | Retry create with the same `referenceId`. |
| 503 | `SERVICE_UNAVAILABLE` | Service is not available. Please try again later. | Retry later. |

Status-code classes per spec: 200 success, 4XX client error (for example missing fields), 5XX Atome server error.

## Sandbox / Test

- Test base URL: `https://api.apaylater.net/v2`. Test credentials come from your Atome Account Manager.
- Malaysia test-environment callback source IP: `47.254.247.54` (production uses `47.254.230.20` and `47.254.238.153`).
- FAQ: test with small amounts so you do not hit the test user's credit limit.
- Public test customer accounts, OTPs, or test cards: not documented in official docs (checked 2026-10-21). The spec links Postman test screenshots in a Google Doc; ask your Account Manager for test logins.
- Rate limits: not documented in official docs (checked 2026-10-21). Tell Atome a week ahead of large campaigns so they can adjust capacity and risk rules.

## Mobile App SDKs

Official demo repos for using `appPaymentUrl` from a native app:

- iOS: `https://github.com/Atome-FE/AtomeMerchantDemo-iOS`
- Android: `https://github.com/Atome-FE/AtomeMerchantDemo-Android`
- React Native: `https://github.com/Atome-FE/AtomeMerchantDemo-RN`
- Flutter: `https://github.com/Atome-FE/AtomeMerchantDemo-Flutter` (in the org; not linked from the spec)

Each SDK has two calls: "is the Atome app installed" and "open this `appPaymentUrl`". Package names and exact method names are in `references/atome-plugins-sdk.md`.

## Not Covered By The v2 Spec

Not documented in official docs (checked 2026-10-21): recurring billing, tokenisation, saved payment methods, partial capture or authorisation holds, settlement report API (a settlement report and SFTP delivery exist for merchants, but no API), and webhook event types other than the payment status callback. The spec also defines a `paymentInfo.shippingInfo` schema but no endpoint that uses it (see Payment Object).
