# Qashier Online Payment API Reference

Use this reference for factual QashierPay Online guidance. It is distilled from the official OpenAPI file `payment-online-checkout-api.yaml` (version 1.0.0) and the "Accept an Online Payment" guide on docs.qashier.com, checked 2026-10-21. Verify account approval, enabled payment methods, settlement timing, and pricing with Qashier when those details matter.

For the in-store terminal partner API, read `qashier-instore.md`.

## Source URLs

- `https://docs.qashier.com/docs/intro`
- `https://docs.qashier.com/docs/guides/checkout-session`
- `https://docs.qashier.com/api/payment-online/`
- `https://docs.qashier.com/redocusaurus/payment-online-checkout-api.yaml` (raw OpenAPI 3.0.3 spec behind the reference page)
- `https://qashier.com/my/qashierpay-online`

## Integration Surfaces

- Checkout Session API with a Qashier-hosted payment page (redirect to `url`). This is the only online API documented.
- Plug-ins for Shopify, Wix, WooCommerce (no-code; listed on the QashierPay Online page; no plug-in docs published).
- Pay-by-Link from the QashierHQ dashboard (no code).
- In-store terminal partner API (see `qashier-instore.md`).

Embedded card forms, saved cards, tokenisation, recurring billing, payouts, and settlement report APIs: Not documented in official docs (checked 2026-10-21).

## Environments

| Environment | API base |
|---|---|
| Production | `https://api.qashier.com/v3/payment/qpay` |
| Staging | `https://api.staging.qashier.com/v3/payment/qpay` |

- The docs example key looks like `sk_test_...` (placeholder). Hosted checkout URLs look like `https://qpay.qashier.com/c/pay/cs_test_...` in the guide example.
- All requests must use HTTPS; plain HTTP fails.
- Use the key that belongs to the same environment as the base URL.

## Authentication

HTTP Basic Auth. API key is the username, password is empty.

```bash
curl https://api.qashier.com/v3/payment/qpay/checkout/sessions -u <QASHIER_API_KEY>:
```

```text
Authorization: Basic <base64("<QASHIER_API_KEY>:")>
```

Errors: missing or bad key returns 401 `authentication_error` ("Invalid API Key provided"); key without permission returns 403 `authentication_error`.

## Common Headers

| Header | Direction | Notes |
|---|---|---|
| `Authorization` | request | Basic, as above. |
| `Content-Type: application/json` | request | JSON bodies. |
| `Idempotency-Key` | request, optional | On `POST /checkout/sessions` and `POST /checkout/sessions/{id}/expire`. Any unique string, max 255 chars. Same key returns the original response without reprocessing. Keys are removed after 24 hours. Reuse with different endpoint or params returns 409 `idempotency_error`. |
| `Request-Id` | response | Format `req_` plus alphanumerics/underscores. Quote it to Qashier support. |

The spec does not list `Idempotency-Key` for `POST /refund`. Not documented whether refund accepts it (checked 2026-10-21).

## Endpoint Catalogue

| Method | Path | Purpose |
|---|---|---|
| POST | `/checkout/sessions` | Create a Checkout Session |
| GET | `/checkout/sessions` | List Checkout Sessions (cursor pagination) |
| GET | `/checkout/sessions/{id}` | Retrieve a Checkout Session (status requery) |
| POST | `/checkout/sessions/{id}/expire` | Expire an `open` Checkout Session |
| POST | `/refund` | Refund a completed Checkout Session (full or partial) |
| POST (to you) | your webhook URL | `checkout.session.completed`, `checkout.session.expired` events |
| POST (to you) | your webhook URL | Refund update events (`refund.update` in spec) |

No other endpoints are documented. No capture, void, cancel-payment, retrieve-refund, list-refunds, customer, token, recurring, payout, or settlement endpoints exist in the online spec.

## Create a Checkout Session

`POST /checkout/sessions` (operationId `createCheckoutSession`)

Request body (`CreateCheckoutSessionRequest`):

| Field | Type | Required | Notes |
|---|---|---|---|
| `currency` | string | yes | ISO 4217, uppercase, e.g. `MYR`, `SGD`, `PHP`. |
| `line_items` | array of LineItem | yes | Items purchased. |
| `line_items[].unit_amount` | integer | yes | Non-negative integer in cents (sen for MYR). |
| `line_items[].quantity` | integer | yes | Spec says "Limit to 1". |
| `return_url` | string | yes | Where the customer returns after authenticating or cancelling in the payment method's app or site. |
| `success_url` | string | yes | Where Qashier sends the customer when payment is complete. |
| `expires_at` | integer | no | Unix seconds. 30 minutes to 24 hours after creation. Default 24 hours. |
| `customer_email` | string | no | Receives the payment receipt. |

No merchant reference / order id / metadata field is documented. Store the returned session `id` against your order before redirecting.

Webhook URL: the spec's callback is declared at `{$request.body#/webhook_url}`, which implies a `webhook_url` request field, but `webhook_url` is not in the request schema and no dashboard webhook setting is documented. Confirm with Qashier how the webhook URL is registered (checked 2026-10-21).

Example (MYR, RM20.00):

```bash
curl -X POST https://api.staging.qashier.com/v3/payment/qpay/checkout/sessions \
  -u <QASHIER_API_KEY>: \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: order_1234_attempt_1" \
  -d '{
    "currency": "MYR",
    "line_items": [{ "unit_amount": 2000, "quantity": 1 }],
    "return_url": "https://shop.example.com/checkout/return",
    "success_url": "https://shop.example.com/checkout/success",
    "customer_email": "buyer@example.com"
  }'
```

Then redirect the browser (the guide uses HTTP 303) to the response `url`.

Responses: 200 CheckoutSession (base fields below); 400 `invalid_request_error` (e.g. `parameter_missing`, param `return_url`); 401; 403; 409 `idempotency_error`; 429 `rate_limit_error`.

## Checkout Session Object

Base fields (returned by create):

| Field | Type | Notes |
|---|---|---|
| `object` | string | `checkout.session` |
| `id` | string | Session id. Examples show both `cs_test_...` and UUID formats; treat as opaque. |
| `currency` | string | ISO currency code. |
| `amount_total` | integer | Total after discounts and taxes, in cents. |
| `created_at` | integer | Creation time (webhook example shows `created`). |
| `expires_at` | integer | Unix time of expiry. |
| `payment_status` | string | `paid` (money moved) or `unpaid` (no successful payment yet; on a completed event, a decline). |
| `payment_method` | string | Method used. Value list not documented. |
| `status` | string | `open`, `complete`, `expired`. |
| `return_url` | string | As sent. |
| `success_url` | string | As sent. |
| `url` | string | Hosted checkout URL to redirect to. |

Extra fields on retrieve, list, expire, and webhook payloads:

| Field | Type | Notes |
|---|---|---|
| `refunded` | boolean | Whether the session has been refunded. |
| `refund_details` | array of Refund | Refunds on this session. |
| `receipt_number` | string | Receipt number. |
| `customer_email` | string | Receipt email. |

The webhook example also shows `line_items` as `{object: "list", data: [{amount_total, currency, description, quantity}]}` and `mode: "payment"`; these are not in the schema.

Status meanings: `open` (in progress, payment not started), `complete` (payment processing may still be in progress), `expired` (no further processing).

## Retrieve a Checkout Session (requery)

`GET /checkout/sessions/{id}`. Path `id` required. Returns CheckoutSession. 404 `invalid_request_error` "No such checkout session".

Use it before settling and for abandoned-session checks after `expires_at`.

## List Checkout Sessions

`GET /checkout/sessions`

| Query param | Type | Notes |
|---|---|---|
| `created` | object | `gt`, `gte`, `lt`, `lte` integers (Unix seconds). |
| `ending_before` | string | Cursor: object id; fetch previous page. |
| `starting_after` | string | Cursor: object id; fetch next page. |
| `limit` | integer | 1 to 100, default 10. |
| `status` | string | `open`, `complete`, `expired`. |

Response: `{ "object": "list", "data": [CheckoutSession], "has_more": boolean, "url": "/checkout/sessions" }`. 400 example: code `parameter_invalid_integer`, param `limit`. The encoding of the `created` object in a query string is not documented.

## Expire a Checkout Session

`POST /checkout/sessions/{id}/expire`, optional `Idempotency-Key`. Only allowed when `status` is `open`. Returns CheckoutSession. After expiry the customer sees an expired message. 400 code `checkout_session_not_expireable` if not open or already expired; 404 if not found. Emits `checkout.session.expired`.

## Refund a Payment

`POST /refund` (operationId `refundPayment`)

| Field | Type | Required | Notes |
|---|---|---|---|
| `sessionId` | string | yes | Checkout Session id (camelCase in this endpoint). |
| `amount` | integer | no | Cents. Omit for a full refund. |

Response Refund object:

| Field | Type | Notes |
|---|---|---|
| `object` | string | `refund` |
| `session_id` | string | Refunded session id. |
| `created_at` | integer | Unix seconds. |
| `amount` | integer | Cents. |
| `currency` | string | Uppercase ISO code. |
| `status` | string | `pending`, `succeeded`, `failed`. |

Refunds go back to the original payment method and are processed asynchronously. Errors: 400 `parameter_missing` (param `sessionId`), 401, 403, 404 "No such checkout session: <id>", 429. There is no refund id in the schema and no refund retrieve endpoint; reconcile from refund webhooks and the session's `refund_details`. Guard against double refunds in your own database.

## Webhooks

Event envelope (both checkout and refund events):

| Field | Type | Notes |
|---|---|---|
| `id` | string | Event id (examples: `evt_1234567890`, `sess_123`). Use for dedupe. |
| `api_version` | string | `1.0.0` is latest. |
| `request.id` | string | Request that triggered the event (same as TraceId in `traceparent`). |
| `request.idempotency_key` | string | Idempotency key of that request. |
| `type` | string | See below. |
| `account` | string | Account id, e.g. `acct_...`. |
| `created` | integer | Unix seconds. |
| `data.object` | object | CheckoutSession, or Refund for refund events. |

Event types:

| Type | When | Action |
|---|---|---|
| `checkout.session.completed` | A payment attempt finished, success or decline. | Fulfil only if `data.object.payment_status == "paid"`. `unpaid` = declined. |
| `checkout.session.expired` | You expired the session through the expire endpoint. | Mark cancelled. Natural expiry at `expires_at` sends no event. |
| refund event (spec name `refund.update`, example `type` value `refund`) | A refund was updated. | Update refund state from `data.object.status`. |

Respond 200 to acknowledge. Retry schedule, timeout, and delivery IPs: Not documented in official docs (checked 2026-10-21). Webhooks may arrive late; do not treat silence as failure.

### Signature Verification

Header:

```text
X-Qashier-Signature: t=1625669316,v1=5257a869e7ecebeda32affa62cdca3fa51cad7e77a0e56ff536d0ce8e108d8bd
```

- `t=` is the timestamp. Each signature is prefixed by a scheme; only `v1` is valid today. There may be more than one signature.
- Algorithm (exact doc steps):
  1. Extract the timestamp and signatures from the header.
  2. `signing_key = SHA256(your_webhook_secret)`.
  3. `signed_payload = <timestamp string> + "." + <raw request body before JSON parsing>`.
  4. `expected = HMAC_SHA256(key = signing_key, message = signed_payload)`.
  5. Compare to the `v1` value with a constant-time comparison.

The docs example `v1` is 64 hex characters, so compare as lowercase hex. The docs do not say whether `signing_key` is the raw 32-byte digest or its hex string, and do not state a timestamp tolerance. Implement raw digest first, confirm against a real staging event, and reject stale `t` with your own window (for example 5 minutes). Where the webhook secret is shown in the dashboard is not documented.

```js
import crypto from "node:crypto";

export function verifyQashier(rawBody, header, webhookSecret, toleranceSec = 300) {
  const parts = header.split(",").map((p) => p.trim().split("="));
  const t = parts.find(([k]) => k === "t")?.[1];
  const sigs = parts.filter(([k]) => k === "v1").map(([, v]) => v);
  if (!t || sigs.length === 0) return false;
  if (Math.abs(Date.now() / 1000 - Number(t)) > toleranceSec) return false;
  const signingKey = crypto.createHash("sha256").update(webhookSecret).digest(); // raw bytes; confirm vs hex
  const expected = crypto.createHmac("sha256", signingKey).update(`${t}.${rawBody}`).digest("hex");
  return sigs.some((s) => s.length === expected.length &&
    crypto.timingSafeEqual(Buffer.from(s), Buffer.from(expected)));
}
```

`rawBody` must be the exact bytes received (for Express use `express.raw({ type: "application/json" })`).

## Settlement Rule

1. Verify signature. 2. Dedupe on event `id`. 3. Look up local order by `data.object.id`. 4. Retrieve the session from the API. 5. Require `payment_status == "paid"`, `amount_total` equal to expected sen, `currency == "MYR"`. 6. Mark paid once inside a transaction.

## Errors

Error body:

```json
{ "error": { "type": "invalid_request_error", "code": "parameter_missing", "message": "Missing required parameter: return_url", "param": "return_url" } }
```

| Field | Notes |
|---|---|
| `type` | Required. `api_error`, `authentication_error`, `idempotency_error`, `invalid_request_error`, `rate_limit_error`. |
| `message` | Required. Human readable. |
| `code` | Optional. Documented codes: `parameter_missing`, `parameter_invalid_integer`, `checkout_session_not_expireable`. |
| `param` | Optional. Field at fault. |

HTTP status: 200 OK, 400 bad request, 401 no valid key, 403 no permission, 404 not found, 409 idempotency conflict, 429 too many requests, 500 Qashier server error.

## Rate Limits

100 requests per second per API key. Excess returns 429 with `rate_limit_error`. Back off and retry with the same `Idempotency-Key`.

## Amount Rules

- Integers in the smallest unit: `unit_amount` and refund `amount` are cents (sen). RM1.00 = `100`.
- Currency uppercase ISO 4217. Malaysian merchants use `MYR`.
- `quantity` limited to 1 per the spec; put the full price in `unit_amount`.

## Payment Methods

You do not choose methods per session. Checkout renders whatever the merchant account has enabled, and this varies by country and over time. To see what an account offers, create a test session and open its `url`. A missing method is an account setting; ask the Qashier representative. The MY marketing page lists DuitNow QR, cards, e-wallets (GrabPay, Touch 'n Go, Boost), cross-border, and BNPL. No payment method codes are documented.

## Sandbox Test Cards (staging only)

Test numbers depend on the account's country. For MY (and SG) accounts:

| Card | CVV | Expiry | Brand | Result |
|---|---|---|---|---|
| 4005520000000129 | 002 | 10/30 | Visa | Success |
| 5413330089010640 | 002 | 10/30 | Mastercard | Success |
| 4147463011110083 | any | any | Visa | 3DS frictionless, authenticated |
| 5239290700000028 | any | any | Mastercard | 3DS frictionless, authenticated |
| 4147463011110091 | any | any | Visa | 3DS frictionless, not authenticated |
| 5239290700000036 | any | any | Mastercard | 3DS frictionless, not authenticated |
| 4147463011110117 | any | any | Visa | 3DS frictionless, attempted |
| 5239290700000044 | any | any | Mastercard | 3DS frictionless, attempted |
| 4147463011110042 | any | any | Visa | 3DS frictionless, rejected |
| 5239290700000051 | any | any | Mastercard | 3DS frictionless, rejected |
| 4147463011110109 | any | any | Visa | 3DS challenge, authenticated |
| 4147463011110034 | any | any | Visa | 3DS challenge, rejected |
| 5239290700000010 | any | any | Mastercard | 3DS challenge, rejected |

Thailand-only rows (not for MY accounts): 4111111111111111 and 5372074248113841 success; 4953261730509988 3DS challenge; 4943129900084541 declined insufficient funds; 4111111111000025 declined by issuer (CVV 123).

Notes: "any" means any future expiry and correctly sized CVV; enter any billing postal code. 3DS rows test authentication only and are not built to settle. Declined rows still emit `checkout.session.completed` with `payment_status: unpaid`. QR and wallet methods are tested by completing payment in the app the checkout hands you. Check the payment in QashierHQ > Transactions.

## Documented Gaps (confirm with Qashier)

- How the webhook URL is registered (`webhook_url` body field vs dashboard).
- Where the webhook secret and API keys live in the dashboard, and how staging accounts are issued.
- Whether `SHA256(secret)` is used as raw bytes or hex; timestamp tolerance; retry policy.
- Merchant reference or metadata on sessions; refund idempotency; webhook source IPs.
- The guide mentions "Elements" and a "TBD privacy center"; no Elements SDK is documented.

## Do Not Use

`https://docs.qashier.com/docs/api/payments-example` shows `POST https://api.qashier.com/v1/payments` with a Bearer token and fields `amount`, `currency`, `description`, `customer_id`. It is a template demonstration page, not part of either OpenAPI spec. The related `/docs/api/openapi-reference-example` page only shows "Loading endpoint documentation..." placeholders.
