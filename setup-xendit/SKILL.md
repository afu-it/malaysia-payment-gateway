---
name: setup-xendit
description: Set up, build, debug, and explain Xendit payment gateway integrations, including Payments API v3, payment requests, payment tokens, webhooks, refunds, and xenPlatform sub-account routing. Use when implementing checkout, server-side payment collection, saved payment methods, webhook handling, payment status reconciliation, or platform marketplace flows with Xendit.
---

# Xendit Payment Gateway

## Quick Start

Use this skill for Xendit integration work. Start with the task shape, then load only the relevant reference:

- Payments API or checkout flow: read [payments-api.md](references/payments-api.md).
- Webhook receiver, retries, duplicate events, or callback-token validation: read [webhooks.md](references/webhooks.md).
- Marketplace/platform, sub-accounts, split fees, balance transfer, or `for-user-id` routing: read [xenplatform.md](references/xenplatform.md).
- First-time project setup, API keys, dashboard prerequisites, or channel activation: read [setup-checklist.md](references/setup-checklist.md).
- Endpoint/category lookup and live documentation links: read [source-map.md](references/source-map.md).

## Integration Rules

- Use server-side code for secret-key requests and webhook handling.
- Authenticate Xendit API calls with HTTP Basic Auth where username is the secret API key and password is empty.
- Preserve trailing colon before Base64 encoding when manually building `Authorization: Basic ...`.
- Treat `reference_id` as merchant idempotency/correlation key; store Xendit IDs returned by the API.
- Never trust client-side payment completion alone. Fulfill orders from verified server-side state or webhook state.
- Make webhook handlers idempotent by Xendit object/event identifiers such as `payment_id`, `capture_id`, or product-specific IDs.
- Return a 2xx response quickly from webhook endpoints, then process heavier work asynchronously.
- Do not assume webhook order. Reconcile from current object state when events arrive out of sequence.

## Current API Shape

Prefer Payments API v3 for new payment collection:

- `POST /v3/payment_requests`: create payment intent/request.
- `POST /v3/payment_tokens`: save payment information for later reuse.
- `/refunds`: refund payment funds.

Common payment request types:

- `PAY`: collect one payment.
- `PAY_AND_SAVE`: collect one payment and save reusable payment information.
- `PAY` with token: collect using previously saved payment token.
- `REUSABLE_PAYMENT_CODES`: collect multiple payments from reusable code or static channel setup.

## Measure Twice

Before coding against Xendit:

1. Decide product path: Payment Sessions/Components, Payments API v3, Payment Links, Subscriptions, Payouts, or xenPlatform.
2. Confirm country, currency, channel, and channel properties.
3. Confirm live vs test API key, key permissions, and callback/webhook URLs.
4. Confirm whether flow requires customer action via returned `actions` or hosted/embedded checkout.
5. Confirm status mapping and order-fulfillment state machine.
6. Confirm webhook token verification and idempotency storage.
7. For xenPlatform, confirm master account vs sub-account ownership and routing header/body fields.

## Verification

For production-facing answers, verify live docs if exact field names, enum values, or endpoint schemas matter. Xendit docs are active and may change; this skill carries stable workflow rules plus source links, not a complete frozen OpenAPI copy.
