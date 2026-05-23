# Curlec Payment Gateway Reference

## Contents

- Sources
- Product surfaces
- Standard Checkout flow
- Authentication and environment
- Core endpoints
- Checkout options
- Signature verification
- Webhooks
- Payment status and capture
- Refunds
- Sandbox and testing
- Common loopholes and fixes

## Sources

Docs downloaded/read from Curlec/Razorpay Curlec docs on 2026-05-13:

- Docs home: https://curlec.com/docs/
- API reference: https://curlec.com/docs/api/
- API authentication: https://curlec.com/docs/api/authentication/
- API best practices: https://curlec.com/docs/api/best-practices/
- Standard Checkout prerequisites: https://curlec.com/docs/payments/payment-gateway/web-integration/standard/
- Standard Checkout integration steps: https://curlec.com/docs/payments/payment-gateway/web-integration/standard/integration-steps/
- Standard Checkout callback URL: https://curlec.com/docs/payments/payment-gateway/callback-url/
- Standard Checkout best practices: https://curlec.com/docs/payments/payment-gateway/web-integration/standard/best-practices/
- Standard Checkout troubleshooting/FAQs: https://curlec.com/docs/payments/payment-gateway/web-integration/standard/troubleshooting-faqs/
- Payment gateway flow: https://curlec.com/docs/payments/payment-gateway/how-it-works/
- Orders API: https://curlec.com/docs/api/orders/
- Payments API: https://curlec.com/docs/api/payments/
- Refunds API: https://curlec.com/docs/api/refunds/
- Payment Links API: https://curlec.com/docs/api/payments/payment-links/
- Webhooks overview: https://curlec.com/docs/webhooks/
- Webhooks setup: https://curlec.com/docs/webhooks/setup-edit-payments/
- Webhooks validation/testing: https://curlec.com/docs/webhooks/validate-test/
- Webhooks FAQ: https://curlec.com/docs/webhooks/faqs/
- General FAQ: https://curlec.com/docs/faqs/
- Shared Razorpay web integration, order, payment, refund, webhook, and signature docs were cross-checked for concepts that Curlec inherits because Curlec uses Razorpay APIs/Checkout.

## Product Surfaces

- Payment Gateway/Standard Checkout: hosted Curlec Checkout UI loaded by merchant site/app.
- Orders API: create/fetch orders and list payments linked to an order.
- Payments API: fetch, capture, update, refund, and inspect payments.
- Refunds API: create/fetch refunds and reconcile refund state.
- Payment Links/Pages/Buttons: hosted no-code or low-code alternatives; Payment Links also have API support.
- Webhooks: async notifications for orders, payments, refunds, settlements, and related events.
- RazorpayX/Payouts: separate payout surface; do not mix with payment collection unless user asks.

## Standard Checkout Flow

1. Customer places order in merchant app.
2. Merchant backend creates internal `transaction_id` or checkout row.
3. Merchant backend creates Curlec Order using Orders API.
4. Curlec returns `order_id`, for example `order_EKwxwAgItmmXdp`.
5. Backend stores mapping: internal transaction/order -> Curlec `order_id`, amount, currency, user, status.
6. Frontend opens Checkout using public Key ID and stored Curlec `order_id`.
7. Checkout collects customer payment details.
8. On successful authorization, Checkout returns:
   - `razorpay_payment_id`
   - `razorpay_order_id`
   - `razorpay_signature`
9. Backend verifies signature, validates amount/currency/order mapping, and checks capture/payment status.
10. Webhooks drive durable fulfilment and reconciliation.

Redirect/FPX variant:

- Add `callback_url` to Checkout options.
- Set `redirect: true`.
- Expect browser redirect to `callback_url` with payment identifiers/signature after successful payment.
- Still verify signature server-side and reconcile with webhooks/API.

## Authentication and Environment

- Base URL for most APIs: `https://api.razorpay.com/v1`
- Auth: HTTP Basic Auth.
- Username: `[YOUR_KEY_ID]`
- Password: `[YOUR_KEY_SECRET]`
- Header format when manually building headers: `Authorization: Basic <base64(KEY_ID:KEY_SECRET)>`
- Curlec docs warn that invalid casing/quoting such as `BASIC`, `basic`, `Basic "token"`, or `Basic $token` causes auth failures.
- API keys exist separately for Test and Live modes.
- Only Key ID is visible after generation; Key Secret must be saved securely at generation time.
- Use Live keys only after integration, webhook handling, and go-live checklist pass.

Example curl shape:

```bash
curl -u "$CURLEC_KEY_ID:$CURLEC_KEY_SECRET" \
  -X GET "https://api.razorpay.com/v1/payments"
```

## Core Endpoints

Use exact parameters from live docs/SDK for final code. Common paths:

- Create order: `POST /v1/orders`
- Fetch all orders: `GET /v1/orders`
- Fetch order by ID: `GET /v1/orders/{order_id}`
- Fetch payments for an order: `GET /v1/orders/{order_id}/payments`
- Fetch all payments: `GET /v1/payments`
- Fetch payment by ID: `GET /v1/payments/{payment_id}`
- Capture payment: `POST /v1/payments/{payment_id}/capture`
- Refund payment: `POST /v1/payments/{payment_id}/refund`
- Fetch payment refunds: `GET /v1/payments/{payment_id}/refunds`
- Create payment link: `POST /v1/payment_links`
- Fetch payment link: `GET /v1/payment_links/{payment_link_id}`
- Cancel payment link: `POST /v1/payment_links/{payment_link_id}/cancel`
- Send payment link notifications: `POST /v1/payment_links/{payment_link_id}/notify_by/{medium}`

Core order fields usually include:

- `amount`: integer in smallest currency unit.
- `currency`: ISO currency code, for Malaysia commonly `MYR`.
- `receipt`: merchant reference.
- `notes`: optional key-value metadata.

Order state guidance:

- Orders start as created/attemptable records and can move to paid when payment succeeds.
- Reconcile order state with payments linked to that order.
- Store `amount`, `amount_paid`, `amount_due`, `currency`, `receipt`, and `attempts` when returned; they help detect partial/inconsistent states.

## Checkout Options

Use `https://checkout.razorpay.com/v1/checkout.js` for web Standard Checkout.

Important options:

- `key`: public Key ID.
- `amount`: amount in smallest currency unit when not using an order; prefer server-created `order_id`.
- `currency`: currency code.
- `name`: merchant/business name.
- `description`: payment description.
- `image`: logo URL or base64.
- `order_id`: Curlec Order ID from server.
- `handler`: frontend callback for success; send returned IDs/signature to backend.
- `callback_url`: server URL to receive successful authorization response.
- `redirect`: required for some redirect flows; use `true` with FPX-style callback URL flows.
- `modal`: controls close/escape/backdrop behavior and can include `ondismiss`.
- `retry`: controls Checkout retry behavior.
- `readonly`: makes selected prefilled fields non-editable.
- `hidden`: hides selected fields where supported.
- `notes`: metadata sent with payment.
- `prefill`: `name`, `email`, `contact`.
- `theme.color`: Checkout brand color.

Rules:

- Prefer creating an order server-side and passing `order_id`.
- Do not trust client amount, currency, order ID, or status.
- Internet Explorer is not supported by Checkout.
- Callback URL only receives successful authorizations; handle failures separately and still rely on webhooks/API checks for final state.
- Browser `handler` is suitable for non-redirect flows; `callback_url` plus `redirect: true` is required for redirect methods such as FPX.
- Do not put Checkout options containing sensitive backend-only data in frontend code.

## Signature Verification

Checkout success signature:

```text
message = server_stored_order_id + "|" + razorpay_payment_id
expected = hmac_sha256_hex(message, key_secret)
expected == razorpay_signature
```

Mandatory checks:

- Retrieve `order_id` from your own database row; do not use client-sent `razorpay_order_id` as the HMAC order component.
- Use server-side Key Secret.
- Compare signatures with constant-time comparison where available.
- After signature passes, fetch/validate payment or order state when fulfilment is high value.
- Store payment ID, signature verification result, verified timestamp, and raw gateway status/audit metadata.

Node example:

```js
import crypto from "node:crypto";

export function verifyCurlecCheckoutSignature({
  serverOrderId,
  paymentId,
  receivedSignature,
  keySecret,
}) {
  const expected = crypto
    .createHmac("sha256", keySecret)
    .update(`${serverOrderId}|${paymentId}`)
    .digest("hex");

  return crypto.timingSafeEqual(
    Buffer.from(expected, "hex"),
    Buffer.from(receivedSignature, "hex"),
  );
}
```

## Webhooks

Setup:

- Configure separate Test and Live webhook URLs in Dashboard.
- Webhook URLs must be public and typically use ports 80 or 443.
- Common public tunnel/testing domains can be blocked. Curlec docs list multiple blacklisted domains such as `requestbin.com`, `webhook.site`, `ngrok.io`, and `loca.lt`.
- For Test mode webhook create/edit/delete OTP, docs list default OTP `754081`.
- Dashboard can support multiple payment webhook URLs; docs FAQ says up to 30 for Payments.

Validation:

- Header: `X-Razorpay-Signature`
- Message: raw webhook request body.
- Key: webhook secret configured in Dashboard.
- Algorithm: HMAC SHA-256 hex digest.
- Never parse, cast, stringify, pretty-print, or alter webhook body before signature verification.
- If webhook secret changed, retries generated under older secret must be validated with the old secret.

Idempotency:

- Header `x-razorpay-event-id` is unique per event.
- Store event ID with unique constraint.
- Return 2xx only after durable record/queue.
- Webhooks can be delivered more than once.
- Event order is not guaranteed; `payment.captured` can arrive before `payment.authorized` in practice.
- Non-2xx webhook responses are treated as delivery failures.
- Webhooks can be resent if your server accepts an event but does not respond within about 5 seconds.
- Failed webhook delivery uses exponential backoff for up to 24 hours after event creation; continuing failures can disable the webhook.
- Curlec-side replay can be requested for eligible events, but docs list a 15-day age limit and no bulk replay.
- Production webhook delivery requires modern TLS; docs explicitly say TLS 1.0 and TLS 1.1 are not supported in production.

Recommended events:

- `payment.authorized`
- `payment.captured`
- `payment.failed`
- `order.paid`
- Refund events relevant to refund workflow
- `payment.authorized` can matter when manual capture, late authorization, or fraud/risk checks exist.

Use webhooks for durable automation. For immediate user-facing confirmation, fetch Payment/Order API when webhook has not arrived fast enough.

## Payment Status and Capture

- Checkout success indicates authorization, not always final captured settlement state.
- Merchant may use automatic or manual capture depending on account/settings.
- If authorized payments are not captured within the gateway's allowed window, docs/shared Razorpay behavior can auto-refund them. Common Razorpay guidance is about 3 days for uncaptured payments unless settings differ.
- Fulfil irreversible goods/services only after status policy passes, commonly `captured` for payments or paid/captured equivalent for order flow.
- Late authorization can happen when bank/Curlec communication is delayed. Webhooks are important to detect and decide whether to capture.
- Dashboard payment status should show captured for completed payment.

State machine advice:

- Keep merchant states separate from gateway states.
- Accept transitions that move forward only, for example `created -> authorized -> captured -> fulfilled`.
- Treat `failed` after `captured` as suspicious/out-of-order, not downgrade without API verification.
- Record raw events for audit.

## Refunds

- Use Refunds/Payments API for refund operations.
- Refund amount is in the smallest currency unit.
- Track full versus partial refund; do not assume one refund per payment.
- Make refund actions idempotent in merchant backend with a unique refund request ID/reference.
- Store Curlec refund ID, payment ID, amount, status, request body, response body, and operator/source.
- Reconcile refund status using API and refund webhooks where enabled.
- Do not issue duplicate refunds on repeated admin clicks or retried jobs.

## Payment Methods and Go-Live

- Available methods depend on Curlec account configuration and approval.
- Test mode can show methods differently from Live mode.
- Enable/check cards, FPX, wallets, BNPL, or other methods in Dashboard before blaming code.
- For FPX/redirect flows, ensure `callback_url` is reachable over HTTPS in production.
- Run go-live checks for Live keys, Live webhook URL, payment method enablement, domain/brand details, settlement/refund permissions, and support contacts.

## Sandbox and Testing

- Test mode simulates payment flows; no real money moves.
- Test and Live mode data, keys, webhooks, and transactions are separate.
- Test webhook payload structure matches Live mode enough for staging validation.
- Test with success, failure, duplicate webhook, bad signature, wrong amount, wrong order, out-of-order events, network timeout, and retry.
- Test redirect/FPX callback flow separately from browser `handler` flow.
- Replace test keys with live keys only after webhook verification and go-live checklist pass.

## Common Loopholes and Fixes

Fake frontend success:

- Fix: verify Checkout signature server-side and fetch payment/order state before fulfilment.

Client tampers amount/currency/order:

- Fix: create order server-side and compare gateway amount/currency/order to stored record.

Using `razorpay_order_id` from browser to generate HMAC:

- Fix: load `order_id` from server database using merchant transaction/session.

Webhook body parsed before HMAC:

- Fix: configure framework raw body for webhook route and verify before JSON parsing.

Duplicate webhooks or retries:

- Fix: unique `x-razorpay-event-id`, idempotent fulfilment, durable event log.

Out-of-order webhooks:

- Fix: monotonic state machine plus API fetch before state downgrade or fulfilment.

Authorized but not captured:

- Fix: confirm `captured` or execute capture flow according to business/account setting.

FPX/redirect payment never returns to app:

- Fix: set `callback_url`, set `redirect: true`, use HTTPS/public URL, and verify callback plus webhook handling.

Payment method missing in Checkout:

- Fix: check Dashboard enablement/approval, environment mode, method restrictions, amount/currency support, and account activation.

Secret/key leakage:

- Fix: Key Secret only in server env/secret manager; rotate exposed keys; never log Basic Auth header or webhook secret.

Test/live environment mix:

- Fix: separate env vars, webhook URLs, dashboards, database labels, and alerts.

Old webhook retries after secret rotation:

- Fix: retain old secret for retry validation window.

Gateway unavailable or timeout:

- Fix: record pending state, poll/fetch order/payment, rely on webhook, avoid duplicate order creation unless idempotently mapped.
