# HitPay API Reference

Use this reference for factual HitPay API guidance. Verify current dashboard settings, payment method activation, payout schedule, pricing, account verification, and operational status when those details matter.

## Source URLs

- `https://docs.hitpayapp.com/introduction`
- `https://docs.hitpayapp.com/llms.txt`
- `https://docs.hitpayapp.com/apis/overview`
- `https://docs.hitpayapp.com/apis/guide/online-payments`
- `https://docs.hitpayapp.com/apis/guide/events`
- `https://docs.hitpayapp.com/apis/guide/sandbox`
- `https://docs.hitpayapp.com/apis/payment-request/create-request`
- `https://docs.hitpayapp.com/apis/payment-request/get-payment-status`
- `https://docs.hitpayapp.com/apis/payment-request/refund`
- `https://docs.hitpayapp.com/apis/guide/embedded-qr-code-payments`
- `https://docs.hitpayapp.com/apis/guide/recurring-billing`
- `https://docs.hitpayapp.com/apis/guide/payouts`
- `https://docs.hitpayapp.com/apis/guide/platform-apis`
- `https://status.hitpayapp.com/`

## Integration Surfaces

HitPay supports multiple payment and commerce surfaces:

- Payment Request API for hosted online checkout.
- Hosted checkout redirect using the `url` returned from Payment Request creation.
- Drop-In UI where supported by the chosen checkout mode.
- Embedded QR code payments for local rails.
- Dashboard Payment Links and invoices for no-code collection.
- First-party plugins for ecommerce platforms.
- Recurring billing and saved payment methods.
- Payout APIs and platform APIs.
- POS, card terminal, and static QR flows.

Choose the simplest integration that satisfies the merchant requirement. For a custom ecommerce checkout, the default path is Payment Request API plus dashboard webhooks.

## Environments

Production:

```text
https://api.hit-pay.com
```

Sandbox:

```text
https://api.sandbox.hit-pay.com
```

Sandbox dashboard:

```text
https://dashboard.sandbox.hit-pay.com/login
```

Production dashboard:

```text
https://dashboard.hit-pay.com
```

Sandbox and production are separate accounts. Do not reuse API keys, salts, webhook URLs, dashboards, checkout URLs, or request logs across environments.

## Authentication

API requests use the business API key header:

```text
X-BUSINESS-API-KEY: your_api_key_here
```

HitPay documents API keys under Dashboard > Settings > API Keys. Keep API keys server-side only.

## Online Payment Request Flow

Recommended custom integration flow:

1. Create a local pending payment/order in your database.
2. Call HitPay `POST /v1/payment-requests` from your server.
3. Store HitPay `id`, returned `url`, local reference, amount, currency, environment, and initial status.
4. Redirect the customer to the returned hosted checkout URL or present supported Drop-In UI.
5. Receive and validate the dashboard webhook.
6. Settle idempotently after strict local/provider matching.
7. Use `GET /v1/payment-requests/{request_id}` as fallback when webhook delivery is missing or failed.

### Create Payment Request

```text
POST https://api.hit-pay.com/v1/payment-requests
POST https://api.sandbox.hit-pay.com/v1/payment-requests
```

Headers:

```text
Content-Type: application/json
X-BUSINESS-API-KEY: your_api_key_here
```

Important body fields:

| Field | Notes |
| --- | --- |
| `amount` | Required decimal amount. Docs show minimum range from `0.3` to `999999999.99`; payment method or currency rules can impose more specific minimums. |
| `currency` | Required three-character currency code such as `MYR` or `SGD`. |
| `payment_methods` | Optional list. If omitted, HitPay uses available methods on the account. |
| `email`, `name`, `phone` | Buyer information. |
| `purpose` | Payment request purpose, max 255 characters. |
| `reference_number` | Local reference that cannot be edited by the customer, max 255 characters. |
| `redirect_url` | Browser return URL. HitPay appends query arguments such as payment request reference and status. |
| `allow_repeated_payments` | Allows multiple payments on a payment request link when true. |
| `expiry_date` | Expiry for repeated payments, in Singapore time format. |
| `expires_after` | Relative expiry such as `5 minutes`, `30 mins`, `hours`, or `days`. |
| `generate_qr` | Valid only for specific QR-capable methods. |
| `metadata` | Key-value data, value strings up to 500 characters, up to 50 pairs. |

The response includes a Payment Request `id`, `status`, `reference_number`, `amount`, `currency`, `payment_methods`, and checkout `url`.

### Payment Request Status

```text
GET https://api.hit-pay.com/v1/payment-requests/{request_id}
GET https://api.sandbox.hit-pay.com/v1/payment-requests/{request_id}
```

Use this as a fallback or reconciliation tool. Do not replace webhooks with polling. Status values documented in the API reference include:

- `pending`
- `completed`
- `failed`
- `expired`
- `canceled`
- `inactive`

Only settle paid access after the expected payment request is `completed` and the amount, currency, local reference, account, and environment match.

## Checkout Presentation

After creating the Payment Request, the server returns the `payment_request_id` and `url` to the client.

Supported presentation paths:

- Redirect customer to the returned hosted checkout URL.
- Use Drop-In UI where supported by the selected flow and payment methods.

For Drop-In UI, HitPay docs note that a default link can be found in Dashboard > Payment Links > Default link. Drop-In UI currently does not support Apple Pay.

## Redirect URL

`redirect_url` is a browser experience path. Use it to show success, pending, failed, or "we are confirming your payment" screens.

Do not mark orders paid from redirect alone. HitPay explicitly warns that redirect can be triggered by anyone and that webhook validation is required before fulfilment.

## Webhooks

Register webhook endpoints in Dashboard > Developers > Webhook Endpoints.

For Payment Request checkout, subscribe to `payment_request.completed` and any additional events needed by the app.

Webhook headers documented by HitPay include:

| Header | Meaning |
| --- | --- |
| `Hitpay-Signature` | HMAC-SHA256 signature of the raw JSON payload using the dashboard salt. |
| `Hitpay-Event-Type` | Event type, for example `completed`. |
| `Hitpay-Event-Object` | Event object, for example `payment_request`. |
| `User-Agent` | HitPay sender user agent. |

Webhook validation rules:

1. Preserve the raw request body bytes/string.
2. Read `Hitpay-Signature`.
3. Compute HMAC-SHA256 over the raw JSON payload using the environment-specific webhook salt.
4. Compare with the header using constant-time comparison.
5. Parse JSON only after signature validation.
6. Process idempotently and return a successful HTTP response quickly after durable recording.

Common signature mismatch causes:

- Wrong salt for sandbox versus production.
- HMAC computed over parsed or reserialized JSON instead of raw payload.
- Reading the wrong signature header.

Webhook IPs documented by HitPay:

| Environment | IPs |
| --- | --- |
| Production | `3.1.13.32`, `52.77.254.34` |
| Sandbox | `54.179.156.147` |

Use IP allowlisting only as an additional network control. HMAC verification remains mandatory.

## QR Payment Timing

HitPay docs warn that QR code payments cannot be canceled once generated. A customer can scan or screenshot a QR code and complete payment after leaving checkout.

Recommended handling:

- Record QR creation time.
- Keep the local payment pending.
- Listen for webhooks for a buffer period.
- For PayNow-style QR flows, HitPay notes a 10 to 15 minute backend buffer is safer than immediately canceling when the page is dismissed.

## Refunds

Use the Create Refund reference for current request shape and constraints:

```text
POST https://api.hit-pay.com/v1/refund
POST https://api.sandbox.hit-pay.com/v1/refund
```

Refund rules for agents:

- Store local refund attempts before calling HitPay.
- Use idempotency at the application layer to avoid duplicate refunds.
- Send the successful `payment_id` and the remaining amount to refund.
- Verify original payment request status, captured payment ID, and refundable amount.
- Support partial refunds only where the selected method/account supports them.
- Reconcile refund webhooks or status responses before final customer-facing state.

## Rate Limits

HitPay API overview documents:

- General API rate limit: 400 requests per minute.
- Payment Request creation limit: 70 requests per minute.

Use webhooks for state changes and keep status API calls as fallback/reconciliation to avoid unnecessary polling.

## Production Readiness

Before production:

- Check HitPay status page.
- Complete business verification.
- Activate required payment methods.
- Generate production API key and webhook salt.
- Register production webhook URL.
- Update base URL to `https://api.hit-pay.com`.
- Update checkout/Drop-In default link values if used.
- Configure production callback routes and allowlisted IPs if the server uses network restrictions.
- Run sandbox test payment and webhook verification before live traffic.
