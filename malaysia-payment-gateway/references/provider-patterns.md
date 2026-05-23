# Provider Patterns

Use official docs for exact current endpoints and payload fields. These patterns are guardrails, not a substitute for live provider documentation.

## CHIP Collect

Typical flow:

1. Server creates local pending payment.
2. Server calls CHIP Collect purchases API.
3. Store CHIP purchase id as `gateway_ref`.
4. Browser redirects to `checkout_url`.
5. CHIP sends signed success callback/webhook.
6. Server verifies signature using configured or fetched public key.
7. Server retrieves purchase from CHIP.
8. Settle only if purchase is paid and amount/reference match.

Important checks:

- Bearer secret key stays server-side.
- Brand ID usually must be UUID-like.
- Amount commonly uses minor units for purchase product price.
- Redirect success alone must not unlock access.

## Curlec / Razorpay

Typical flow:

1. Server creates local pending payment.
2. Server creates Razorpay/Curlec Order with Basic Auth `KEY_ID:KEY_SECRET`.
3. Store order id as `gateway_ref`.
4. Browser opens Checkout.js with public key id and order id.
5. Callback returns payment id, order id, signature.
6. Server verifies checkout signature from `order_id + "|" + payment_id`.
7. Server retrieves payment from API.
8. Settle only if payment is captured and amount/currency/order match.
9. Webhook also verifies raw body HMAC with webhook secret.

Important checks:

- Key secret never goes to browser.
- Use server-stored order id when computing checkout signature.
- FPX/redirect methods need proper `callback_url` and redirect settings.
- Webhook body must not be mutated before HMAC verification.

## Xendit Malaysia

Typical hosted checkout flow:

1. Server creates local pending payment.
2. Server creates Xendit Payment Session or Payment Request.
3. Store session/request id as `gateway_ref`.
4. Browser redirects to provider payment link/redirect URL.
5. Xendit posts webhook with callback token header.
6. Server verifies callback token.
7. Server retrieves session/request from Xendit.
8. Settle only if status is completed/succeeded and amount/currency/reference match.

Important checks:

- Basic Auth username is secret key, password empty; preserve trailing colon before Base64 encoding.
- Keep Test and Live keys/webhooks separate.
- Confirm enabled Malaysian channels in dashboard.

## Bayarcash / BCL

Use live provider docs or existing local skill/docs for exact fields.

Guardrails:

- Keep portal/API keys server-side unless provider explicitly marks a key public.
- Verify checksum/signature/callback according to provider docs.
- Treat callbacks as asynchronous settlement, not UI confirmation.
- Store local reference and gateway transaction id.
- Reconcile by provider transaction lookup before fulfillment when API supports it.
