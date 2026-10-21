# Setel API Reference

Entry point for Setel merchant API facts, distilled from the official developer portal `https://developer.setel.com` (checked 2026-10-21). Confirm anything marked "Not documented" or "Inconsistency" with Setel during onboarding.

Other reference files:

- `setel-other-apis.md`: code scan wallet transactions, SSO/OAuth, accounts, user payment methods, loyalty, external orders, parking, EV charging.
- `setel-settlement.md`: settlement reports, MPF file format, manual settlement API.

## Source URLs

- `https://developer.setel.com/llms.txt` (portal index)
- `https://developer.setel.com/docs/integrating-your-platform-with-setel-api`
- `https://developer.setel.com/docs/system-environment`
- `https://developer.setel.com/docs/pay-with-setel-online`
- `https://developer.setel.com/docs/how-payment-intents-work`
- `https://developer.setel.com/docs/placing-a-hold-and-capturing-payments`
- `https://developer.setel.com/docs/receive-payment-notifications-with-webhooks`
- `https://developer.setel.com/docs/refunding-and-canceling-payments`
- `https://developer.setel.com/reference/create-a-checkout-session` (and the other `/reference/*` pages listed in `source-map.md`)
- `https://developer.setel.com/reference/checkout` (error codes)

Every portal page is also served as Markdown with an embedded OpenAPI 3.1 definition by appending `.md` (for example `https://developer.setel.com/reference/create-a-checkout-session.md`).

## Environments

| Environment | Base URL | Notes |
| --- | --- | --- |
| Sandbox | `http://api-pre-prod.setel.com` | From the System environment page ("Last updated: 18 Nov 2024"). Also the `servers` URL in 42 of 43 OpenAPI definitions. Note the scheme is `http` as published. |
| Production | `https://api.setel.com` | System environment page. |

- The OAuth token OpenAPI definition alone lists `https://api-sandbox.setel.my` as its server. Treat this as legacy or a docs inconsistency; confirm the sandbox host with Setel.
- Older links in the docs point to `developer.setel.my` and `setel.readme.io`; the current portal is `developer.setel.com`.
- Setel Dashboard (merchant portal, settlement reports, settings): `https://dashboard.setel.my/login`.

## Authentication And Headers

Partner (merchant) requests carry:

| Header | Value | Notes |
| --- | --- | --- |
| `x-api-key` | string | Provided during onboarding. |
| `x-api-secret` | string | Provided during onboarding. Also the input to the webhook signature key. |
| `Content-Type` | `application/json` | "Currently accepts request with JSON content only." |

- Onboarding also gives a `merchantId` (Integrating page).
- "API keys are strictly for backend functions and should not be passed to the frontend" and "must be kept secure and not uploaded to any version control system" (Authentication headers for Setel partners).
- There is no per-request signing. The API secret is sent as a header.

Some endpoints act on behalf of a Setel user and also need:

| Header | Notes |
| --- | --- |
| `access-token` | User access token (from SSO or account session APIs). Required by Finish checkout session, user payment methods, balance, loyalty card, fuel options and account passcode endpoints. |
| `x-interface-id` | "Default value is `setel` but a proper value will be advised by Setel during onboarding." Optional on loyalty checkout session, wallet transaction and external order; required on account and loyalty endpoints. |
| `x-interface-type` | "This value will be provided by Setel during onboarding." Same optional/required pattern as `x-interface-id`. |

## HTTP Status Codes

| Code | Meaning (per docs) |
| --- | --- |
| `200` | OK. Docs tie it to `PATCH`; several `GET` endpoints also return 200. |
| `201` | Created, for successful `POST` (create session, capture, cancel, refund). |
| `204` | No content. |
| `400` | Invalid request or missing required parameters. |
| `401` | Invalid Access ID or user token; usually includes an internal error code. |
| `403` | Refused: "usually caused by an invalid API key, incorrect request signature, or access is denied." |
| `404` | Session or transaction not found. |
| `500`, `502` | Server error; message and internal error code included. |

Error body shape:

```json
{ "statusCode": 400, "message": "Session's referenceId already existed", "code": "REFERENCE_ID_EXISTED", "errorCode": "9300101" }
```

## Endpoint Catalogue

All paths are relative to the base URL. "Partner" = `x-api-key` + `x-api-secret`.

### Checkout and payments (Pay with Setel online)

| Method | Path | Purpose | Auth |
| --- | --- | --- | --- |
| POST | `/v1/checkout/sessions` | Create a checkout session | Partner |
| POST | `/v1/checkout/loyalty-session` | Create a checkout session that grants Mesra loyalty points | Partner (+ optional interface headers) |
| GET | `/v1/checkout/sessions/{id}` | Retrieve a checkout session by session id | Partner |
| POST (as documented) | `/v1/checkout/sessions?referenceId={referenceId}` | Retrieve a checkout session by `referenceId` (see Inconsistencies) | Partner |
| PATCH | `/v1/checkout/sessions/{id}/finish` | Finish a session after the user PIN is verified (partner-hosted UI) | Partner + `access-token` |
| PATCH | `/v1/checkout/sessions/{id}/cancel` | Cancel a checkout session | Partner |
| GET | `/v1/payments/payment-intents/{id}` | Retrieve a payment intent | Partner |
| POST | `/v1/payments/payment-intents/{id}/capture` | Capture an authorised (held) payment intent (docs show the path as `/id/capture`) | Partner |
| POST | `/v1/payments/payment-intents/{id}/loyalty-capture` | Capture and grant loyalty points by product | Partner |
| POST | `/v1/payments/payment-intents/{id}/cancel` | Cancel a payment intent | Partner |
| POST | `/v1/payments/payment-intents/{id}/refund` | Full or partial refund | Partner |
| POST | `/v1/settlements/non-terminal-settlements` | Generate a manual settlement (see `setel-settlement.md`) | Partner |
| (merchant-hosted) POST | your `webhook.url` | Setel posts session updates to you | `signature` header |

### Other documented APIs (details in `setel-other-apis.md`)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/v1/wallet/transactions` | Create a wallet transaction (code scan) |
| GET | `/v1/wallet/transactions/{id}` | Retrieve a wallet transaction |
| GET | `/v1/wallet/transactions?referenceId=` | Retrieve a wallet transaction by `referenceId` |
| GET | `/v1/code/{codeString}` | Retrieve a payment code (OTAC) |
| GET | `/v1/payments/payment-methods` | List a user's payment methods |
| GET | `/v1/payments/payment-methods/{id}/balance` | Balance of a payment method |
| POST | `/v1/oauth/token` | Issue OAuth token (SSO) |
| POST | `/v1/account/unverified` | Create account with unverified phone number |
| POST | `/v1/account/passcode` | Create passcode |
| PATCH | `/v1/account/passcode` | Reset passcode |
| POST | `/v1/account/sessions/passcode` | Create account session using passcode |
| POST | `/v1/account/sessions/otp/verification` | Create (send) OTP |
| PUT | `/v1/account/sessions/otp/verification` | Verify OTP |
| POST | `/v1/account/sessions/otp` | Create account session using OTP |
| DELETE | `/v1/account/sessions/current` | Delete account session |
| GET | `/v1/retail/fuel-order-options` | Fuel order preset options |
| POST | `/v1/retail/external-orders` | Create external order (earn Mesra points without Setel payment) |
| GET | `/v1/retail/external-orders?referenceId={referenceId}` | Retrieve external order |
| PUT | `/v1/loyalty/cards` | Activate Mesra card |
| POST | `/v1/loyalty/cards/switch-to-physical-card` | Link physical Mesra card |
| GET | `/v1/loyalty/cards` | Loyalty card details (access token) |
| GET | `/v1/loyalty/members/by-card/{cardNumber}` | Loyalty card details by card number |
| POST | `/b2c/api/parking/sessions` | Create parking session |
| GET | `/b2c/api/parking/sessions/{sessionId}` | Retrieve parking session |
| PUT | `/b2c/api/parking/sessions/{sessionId}/end` | End parking session (v1.0, deprecated) |
| PUT | `/b2c/api/parking/sessions/{sessionId}/close` | Close parking session |
| PUT | `/b2c/api/parking/sessions/{sessionId}/charge` | Charge parking session |
| PUT | `/b2c/api/parking/sessions/{sessionId}` | Void parking session |
| POST | `/api/parking/events` | Update parking status |
| POST | `/v1/api/ev-charging/webhook/{vendorName}/events` | EV charging status event |
| POST | `/v1/api/ev-charging/webhook/{vendorName}/payment` | EV charging payment invoice |

## Pay With Setel Online Flow

1. Create a checkout session. A payment intent is generated automatically.
2. PC users are redirected to the Setel Checkout page.
3. Mobile users have three options (guide wording): `redirectUrl` (redirects to a specified link; if not given, same as `checkoutUrl`), `checkoutUrl` (web page with Setel App or Setel mobile web sign-in), `deepLinkUrl` (opens the Setel App directly).
4. Customer pays with Setel Wallet or Setel CardTerus.
5. On success, "a Setel server will POST a webhook request to the merchant's webhook URL". The browser is redirected to `redirect.successUrl`; on cancel or expiry, to `redirect.cancelUrl`.
6. Merchant verifies the webhook signature, requeries, and settles.

Setel recommends "a new session ... each time the customer attempts to pay" (Checkout sessions page).

### Create a checkout session

`POST /v1/checkout/sessions`. Alert on the page: "Please provide a unique `referenceId` to avoid multiple payments and for a successful reconciliation."

Request body (OpenAPI required: `amount`, `currency`, `redirect`):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `referenceId` | string | No (strongly recommended) | Merchant order/cart id. Must be unique; duplicates return `REFERENCE_ID_EXISTED`. |
| `amount` | string | Yes | Total after discounts and taxes, ringgit. Examples use `"10"`. |
| `currency` | string | Yes | ISO code; only `MYR`. Default `MYR`. |
| `capture` | boolean | No | Default `true` (direct capture). `false` places a hold. |
| `redirect` | object | Yes | |
| `redirect.successUrl` | string | | Redirect after a successful session. |
| `redirect.cancelUrl` | string | | Redirect on cancel or expiry. |
| `webhook` | object | No | |
| `webhook.url` | string | | Receives updates when the session changes. |
| `paymentMethod` | object | No | If sent: `family` (required), `type` (required), `brand`. Example `wallet` / `setel` / `setel`. |
| `subMerchant` | object | Third-party acquirers | `name`, `reference`, `merchantCategoryCode`, `state` required; `outletName` optional. Shown on checkout page and e-receipt. |
| `subMerchant.state` | string | | One of `Johor`, `Kuala Lumpur`, `Pulau Pinang`, `Putrajaya`, `Selangor`, `Kedah`, `Kelantan`, `Melaka`, `Negeri Sembilan`, `Pahang`, `Perak`, `Perlis`, `Terengganu`, `Sabah`, `Sarawak`. |
| `referenceMeta` | object | No | Order details. |
| `referenceMeta.items[]` | array | | Each: `itemName` (req), `unitPrice` (number, req), `quantity` (number, req), `totalPrice` (number, req), `variationName`, `totalDiscountedPrice`, `imageUrl`, `loyaltyCategoryCode` (integer). |
| `referenceMeta.totalDiscount` | number | | |
| `referenceMeta.subtotal` | number | | Before taxes, discounts, fees. |
| `referenceMeta.shippingFee` | number | | E-commerce. |
| `referenceMeta.contractId` | string | | "A recurring ID that the transaction corresponds to." No recurring API is documented. |
| `referenceMeta.customFee` | object | | `amount` (integer), `feeName`, `feeType`; must be a predefined fee type. |
| `customer` | object | No | `id`, `email`, `name`. |
| `locale` | string | No | `en`, `ms_my`, `ta`, `zh_hans`, `zh_hant`. |
| `description` | string | No | Order description. |
| `referenceType` | string | No | Enum `season_parking`, `external-service`; default `external-service`. |

Response `201` fields (doc table): `id` (session id), `paymentIntentId`, `paymentIntentStatus`, `referenceId`, `referenceType`, `paymentMethod.{family,type,brand}`, `subMerchant.{name,reference}`, `capture`, `amount`, `currency`, `referenceMeta.*` (also `pumpNumber`, `stationName`, `cartId` in examples), `customer.{id,email,name}`, `redirect.{successUrl,cancelUrl}`, `webhook.url`, `locale`, `description`, `createdAt`, `updatedAt`.

Checkout link fields: the create response example does not show them. The Cancel session response example shows `redirectLink`, `checkoutUrl` and `deepLinkUri` (value like `setelapp://..`). The guide calls them `redirectUrl`, `checkoutUrl`, `deepLinkUrl`. Log the real create response in sandbox and read whichever is present.

Minimal request:

```http
POST {SETEL_BASE_URL}/v1/checkout/sessions
x-api-key: <SETEL_API_KEY>
x-api-secret: <SETEL_API_SECRET>
Content-Type: application/json

{
  "referenceId": "<ORDER_ID>",
  "amount": "10.00",
  "currency": "MYR",
  "capture": true,
  "redirect": { "successUrl": "https://shop.example/pay/return", "cancelUrl": "https://shop.example/pay/cancel" },
  "webhook": { "url": "https://shop.example/webhooks/setel" },
  "description": "Order <ORDER_ID>",
  "locale": "en"
}
```

Amount format: the docs only show whole-ringgit strings (`"10"`, `"3"`, `"99"`); settlement files use floats with up to 2 decimals. The exact accepted decimal format is not documented (checked 2026-10-21); test `"10.00"` style strings in sandbox.

### Create a checkout session that grants loyalty points

`POST /v1/checkout/loyalty-session`. Same body as above except: required fields are `referenceId`, `subMerchant`, `amount`, `currency`; `referenceType` is not in the schema; optional headers `x-interface-id`, `x-interface-type`. Items may carry `loyaltyCategoryCode` (example `"0106"`). Response matches create session.

### Retrieve a checkout session

`GET /v1/checkout/sessions/{id}`. Response `200` fields: `id`, `apiKey`, `paymentIntentId`, `paymentIntentStatus`, `status` (session status), `paymentMethod.*`, `merchantId`, `subMerchant.*`, `referenceId`, `capture`, `amount`, `capturedAmount`, `refundedAmount`, `currency`, `referenceMeta.*`, `locale`, `createdAt`, `updatedAt`, `error` (string). Examples also include `redirect`, `customer`, `webhook`, `description`. `404` returns `SESSION_NOT_FOUND` (`9300100`).

This is the documented fallback when a webhook is not received.

### Retrieve a checkout session by reference ID

Documented as `POST /v1/checkout/sessions?referenceId={referenceId}` with required query `referenceId`. Response as above (no `status`, `capturedAmount`, `refundedAmount` in the table). `404` `SESSION_NOT_FOUND`.

### Finish a checkout session

`PATCH /v1/checkout/sessions/{id}/finish`, header `access-token` (user). For partners that collect the PIN in their own UI after SSO; not needed for the hosted Setel Checkout flow.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `pinToken` | string | Yes | `sha256_hex(pin + userId)`; doc sample: `crypto.createHash('sha256').update(`${pin}${userId}`).digest('hex')`. |
| `paymentMethodDetails.paymentMethodId` | string | Yes | `id` from Retrieve all payment methods of a user. |
| `paymentMethodDetails.userId` | string | Yes | Setel user id. |

Response `200` fields: `merchantId`, `apiKey`, `referenceId`, `referenceType` (example `api-ecom-orders`), `capture`, `amount`, `currency`, `paymentMethod.*`, `referenceMeta.*`, `subMerchant.*`, `id`, `paymentMethodDetail.{paymentMethodId,userId}`, `paymentProcessor` (example `apiPayment`), `paymentIntentId`, `paymentIntentStatus`, `expirationTime` ("900000 ms after creating a session", so 15 minutes), `error`, `status` (example `created`), `createdAt`, `updatedAt`.

### Cancel a checkout session

`PATCH /v1/checkout/sessions/{id}/cancel`, body `{ "reason": "<text>" }` (optional). Response `201` with session fields (see link-field note above).

## Payment Intents

A payment intent is created automatically for every checkout session or wallet transaction.

Statuses (How payment intents work):

| Status | Meaning |
| --- | --- |
| `requires_payment_method` | Set when a checkout session is created; customer must provide payment method. |
| `requires_action` | Not yet supported (3D Secure); skipped. |
| `authorised` | Held, not captured (`capture: false`). |
| `processing` | After `authorised` is captured, for direct capture, or when a wallet transaction is created. |
| `failed` | Payment attempt failed. |
| `succeeded` | Processed; funds in the Setel Dashboard account. |
| `cancelled` | Cancelled; held funds returned. Not possible when `processing` or `succeeded`. |
| `expired` | Uncaptured after 30 minutes. |
| `partially_refunded` | Refund smaller than `amount`; can repeat until `refundedAmount` equals `amount`. |
| `refunded` | `refundedAmount` equals `amount`. |

Examples also show `created` for session/intent status in the Finish response. Session-level `status` values are not fully enumerated in the docs (checked 2026-10-21).

Flows: direct capture `requires_payment_method -> processing -> succeeded|failed`; hold `requires_payment_method -> authorised -> (capture) processing -> succeeded`, or `cancelled`, or `expired` after 30 minutes.

### Retrieve a payment intent

`GET /v1/payments/payment-intents/{id}`. Fields: `id`, `amount`, `succeededAmount`, `refundedAmount`, `capture`, `currency`, `referenceId`, `paymentMethod.*`, `referenceMeta.*`, `subMerchant.*`, `status`, `refundedReason`, `createdAt`, `updatedAt`; examples add `customer`, `description`, `succeededAt`, `cancelledAt`, `cancelledReason`, `refundedAt`.

### Capture a payment intent

`POST /v1/payments/payment-intents/{id}/capture`. Only for sessions with `capture: false` and intent status `authorised`. "Any uncaptured PaymentIntents will be cancelled exactly 30 minutes after they are created." "You cannot capture any amount above the authorised amount."

| Field | Type | Notes |
| --- | --- | --- |
| `amount` | string | Amount to capture. |

Response `201`: `id`, `amount`, `succeededAmount`, `refundedAmount`, `currency`, `referenceId`, `paymentMethod.*`, `subMerchant.*` (example adds `merchantCategoryCode`, `state`), `capture`, `referenceMeta.*`, `customer`, `description`, `status`, `errorMessage`, `succeededAt`, `cancelledAt`, `cancelledReason`, `refundedAt`, `refundedReason`, `createdAt`, `updatedAt`, and `loyalty.{issuedBy,status,mesraCardNumber,earnedPoints,mesraCardBalance,error}`.

Errors: `400 SESSION_COULD_NOT_CAPTURE 9300112`, `400 INVALID_CAPTURE_AMOUNT 9300113`, `400 PAYMENT_INTENT_TX_NOT_FOUND 9300120`, `404 PAYMENT_INTENT_NOT_FOUND 9300117`, `500 INTERNAL_SERVER_ERROR 9300001`, `500 EXTERNAL_SERVER_ERROR 9300007`.

### Capture and grant loyalty points

`POST /v1/payments/payment-intents/{id}/loyalty-capture`. Required: `amount` (string), `products[]` (each `unitPrice` number, `quantity` integer, `loyaltyCategoryCode` 4-digit string), `taxInvoiceNumber` (string). Response `200` as capture, with `loyalty` populated (`loyalty.status` is `failed` or `succeeded`; `issuedBy` example `petronas`). Errors: `400 CAN_NOT_CAPTURE 9300112`, `404 PAYMENT_INTENT_NOT_FOUND`, `500`, `503`.

### Cancel a payment intent

`POST /v1/payments/payment-intents/{id}/cancel`, body `{ "reason": "<text>" }`. Allowed when status is `requires_payment_method`, `authorised`, or `requires_action`; for `authorised` "the remaining amount will automatically be refunded" (released). Response `201` with `status: "cancelled"`, `cancelledAt`, `cancelledReason`. Errors: `400 CAN_NOT_CANCEL`, `404 PAYMENT_INTENT_NOT_FOUND`.

### Refund a payment intent

`POST /v1/payments/payment-intents/{id}/refund`.

| Field | Type | Notes |
| --- | --- | --- |
| `amount` | string | Optional. Omit for a full refund. Partial must not exceed the original charge. |
| `reason` | string | Optional context. |

- "Refunds uses your merchant's available balance, this doesn't include any pending balance. If your available balance doesn't cover the amount of the refund, the refund transaction will fail."
- Response `201`: `id`, `amount`, `succeededAmount`, `refundedAmount`, `currency`, `referenceId`, `capture`, `referenceMeta`, `customer`, `description`, `subMerchant`, `status` (`refunded` or `partially_refunded`), `refundedAt`, `refundedReason`, `createdAt`, `updatedAt`.
- Errors: `400 CAN_NOT_REFUND`, `404 PAYMENT_INTENT_NOT_FOUND`.
- Refunds appear in settlement files as negative amounts with `Original transaction ID`.
- Refund webhooks: not documented (checked 2026-10-21). Requery the intent after refunding.

## Webhooks

- Supported for Pay with Setel online only; "Webhook is not supported" for code scan.
- Registered per session through `webhook.url` at session creation. No dashboard-level webhook setting is documented.
- Considerations from the docs: webhooks may not arrive because of network issues, so "a retry query using the retrieve a checkout session API is recommended"; "Multiple webhook requests for a single order might be sent. Please ensure there is an implementation to verify which request is the latest."

Request Setel sends:

```text
POST <webhook.url>
Content-Type: application/json
signature: <hex string>

{
  "id": "test-id",
  "createdAt": "2021-06-09T10:09:37.927Z",
  "updatedAt": "2021-06-09T10:09:37.927Z",
  "apiKey": "test-x-api-key",
  "paymentIntentId": "test-payment-intent-id",
  "paymentIntentStatus": "succeeded",
  "amount": "10",
  "referenceId": "test-reference-id"
}
```

The docs do not say what `id` refers to. The sample uses `test-id`, and the fields match the checkout session object, so `id` is most likely the checkout session id. Confirm in sandbox by matching it against the stored session id and `paymentIntentId`.

### Signature algorithm

1. `secretHash = hex(SHA-256(x-api-secret))` (lowercase hex string).
2. `data` = concatenation, in this order, of `id`, `createdAt`, `updatedAt`, `apiKey`, `paymentIntentId`, `paymentIntentStatus`, `amount`, `referenceId`. A missing value contributes an empty string (the sample uses `body[field] || ''`). No separators.
3. `signature = hex(HMAC-SHA256(key = secretHash as UTF-8 text, message = data))`.
4. Compare with the `signature` header using a constant-time comparison.

Official sample (Node.js, from the webhooks page):

```js
const { createHmac, createHash } = require('crypto');
const secretHash = createHash('sha256').update(secret).digest('hex');
const fields = ['id','createdAt','updatedAt','apiKey','paymentIntentId','paymentIntentStatus','amount','referenceId'];
let data = '';
fields.forEach(field => data += body[field] || '');
const hmac = createHmac('sha256', secretHash).update(data).digest('hex');
```

Published test vector (docs sample values, not real credentials):

- `x-api-secret` = `test-x-api-secret`
- `secretHash` = `874d70d91c89cfa556797c1754ea186e060a189a0cc398b000f94630c1b1674e`
- `data` = `test-id2021-06-09T10:09:37.927Z2021-06-09T10:09:37.927Ztest-x-api-keytest-payment-intent-idsucceeded10test-reference-id`
- valid signature = `77b928780f10a0d2339d93be7319eda4dda4472d5a9fdf7bcc53768a2a61faf0`

Verified locally on 2026-10-21 with Python `hashlib`/`hmac`: the vector reproduces.

Pseudo-code:

```text
secretHash = sha256_hex(SETEL_API_SECRET)
body = json_parse(raw_body)          # values used as the exact strings received
data = concat(str_or_empty(body[f]) for f in [id, createdAt, updatedAt, apiKey,
              paymentIntentId, paymentIntentStatus, amount, referenceId])
expected = hmac_sha256_hex(secretHash, data)
if !constant_time_equal(expected, header["signature"]): return 401
if body.apiKey != SETEL_API_KEY: reject
session = GET /v1/checkout/sessions/{body.id}
if session.updatedAt older than stored state: ack and ignore
settle if session.paymentIntentStatus == "succeeded" and amount/currency/referenceId match
return 2xx
```

Cautions:

- Because the signature covers individual fields, keep values as received. If `amount` arrives as a JSON number in production, `String(10)` gives `"10"`; avoid number formatting that changes the text (for example `10.00` vs `10`). Confirm the wire type in sandbox.
- JavaScript `||` treats `0`, `false`, `null` and `""` as empty; mirror that only for missing fields.
- Not documented (checked 2026-10-21): expected response code/body, retry schedule, timeout, source IPs, event types other than session status changes.

## Error Codes (Checkout)

| Code | Name | Message |
| --- | --- | --- |
| `9300001` | `INTERNAL_SERVER_ERROR` | Internal server error occurred |
| `9300002` | `AUTHORIZATION_ERROR` | An exception occurred during the authorization process |
| `9300003` | `UNAUTHORIZED` | You are unauthorized to perform this action |
| `9300004` | `VALIDATION_ERROR` | Validation error |
| `9300005` | `SERVICE_UNAVAILABLE` | Service temporarily unavailable |
| `9300006` | `INVALID_CONFIGURATION` | Invalid configuration |
| `9300007` | `EXTERNAL_SERVER_ERROR` | External server error |
| `9300011` | `ACCESS_TOKEN_REQUIRED` | An access-token is required to confirm the status code |
| `9300012` | `INVALID_JWT_TOKEN` | Not valid JWT |
| `9300013` | `JWT_TOKEN_EXPIRED` | JWT token expired |
| `9300014` | `INVALID_RESET_PASSWORD_TOKEN` | Token to reset password is invalid |
| `9300015` | `INVALID_NAMESPACE` | Invalid namespace |
| `9300100` | `SESSION_NOT_FOUND` | Session not found |
| `9300101` | `REFERENCE_ID_EXISTED` | Session's reference ID already exist |
| `9300102` | `SESSION_ALREADY_COMPLETED` | Session has completed |
| `9300103` | `SESSION_PAYMENT_ALREADY_COMPLETED` | Session's payment has already completed |
| `9300104` | `SESSION_ALREADY_SUCCEEDED` | Session has succeeded |
| `9300105` | `SESSION_ALREADY_EXPIRED` | Session has expired |
| `9300106` | `SESSION_ALREADY_CANCELLED` | Session has been cancelled |
| `9300107` | `SESSION_ALREADY_FAILED` | Session has failed |
| `9300108` | `PAYMENT_INTENT_EXISTED` | Payment intent ID already exist |
| `9300109` | `INVALID_SESSION` | Session is invalid |
| `9300110` | `MISMATCH_PAYMENT_AMOUNT` | The payment amount is different from the session amount |
| `9300111` | `INVALID_TRANSACTION_STATUS` | Could not map transaction status to payment intent status |
| `9300112` | `SESSION_COULD_NOT_CAPTURE` | Unable to capture session |
| `9300113` | `INVALID_CAPTURE_AMOUNT` | The capture amount is greater than the session amount |
| `9300114` | `MISSING_PIN_TOKEN` | PIN token is required for this method |
| `9300115` | `SESSION_COULD_NOT_CANCEL` | Unable to cancel session |
| `9300300` | `INVALID_CUSTOMER` | Invalid customer |
| `9300400` | `API_KEY_NOT_FOUND` | API key not found |
| `9300401` | `INVALID_ACCESS_TOKEN` | Invalid access token |
| `9300402` | `INVALID_PAYMENT_PROCESSOR` | Payment processor service is not supported |

Seen only in endpoint examples: `9300117 PAYMENT_INTENT_NOT_FOUND`, `9300120 PAYMENT_INTENT_TX_NOT_FOUND`, `CAN_NOT_CAPTURE`, `CAN_NOT_CANCEL`, `CAN_NOT_REFUND`, `INVALID_OTAC` (wallet), `9300202 SERVICE_UNAVAILABLE` (wallet, 502). Several examples reuse `9300101` for unrelated errors; branch on `code` and HTTP status, not `errorCode` alone.

## Payment Methods

| Method | Online checkout | Code scan |
| --- | --- | --- |
| Setel Wallet | Yes ("Pay with the balance in the Setel Wallet") | Yes |
| Setel CardTerus | Yes ("Pay with selected credit or debit cards directly") | Not supported yet |

`paymentMethod` examples: `{ "family": "wallet", "type": "setel", "brand": "setel" }`. User payment method types listed: `wallet_setel`, `smartpay`, `card_visa` (families `wallet`, `card`, `smartpay`). FPX, DuitNow and third-party e-wallets are not offered through these APIs per the docs.

## Not Documented (checked 2026-10-21)

- Rate limits.
- IP allowlists for API calls or webhook source IPs (SFTP setup may need Setel's IP whitelisted; Setel provides it).
- Webhook retry policy and expected acknowledgement.
- Sandbox test accounts: the code scan page lists test scenarios (restricted account, limits exceeded, expired card, insufficient balance) with test accounts "To be created".
- Minimum/maximum amount for online checkout.
- Recurring or tokenised charging API (only `referenceMeta.contractId` is described).
- Payout or disbursement API.

## Inconsistencies To Confirm

1. Sandbox host: `http://api-pre-prod.setel.com` (current) vs `https://api-sandbox.setel.my` (OAuth spec only). Scheme `http` as published.
2. Checkout link fields: guide `redirectUrl` / `checkoutUrl` / `deepLinkUrl`; cancel response example `redirectLink` / `checkoutUrl` / `deepLinkUri`; create response example shows none.
3. Retrieve by `referenceId` is documented as `POST /v1/checkout/sessions?referenceId=`; that path is also the create endpoint. Try `GET` with the query parameter in sandbox and confirm with Setel.
4. Capture path is printed as `/v1/payments/payment-intents/id/capture` and its `id` parameter is described as "The ID for the checkout session". Use the payment intent id in `{id}` and confirm.
5. The refund guide says payments can be refunded "when `status` is `completed`"; the status table has no `completed` (use `succeeded`).
6. Session expiry: Finish response says `expirationTime` is 900000 ms (15 minutes) after creation; held intents expire after 30 minutes.
7. `capture` appears as string `"true"` in some response examples and boolean elsewhere.
