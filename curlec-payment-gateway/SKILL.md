---
name: curlec-payment-gateway
description: Build, debug, review, and explain Razorpay Curlec payment gateway integrations using Curlec and shared Razorpay documentation. Use when working with Curlec/Razorpay Standard Checkout, FPX or redirect payment methods, Orders API, Payments API, Refunds API, Payment Links, Basic Auth API keys, checkout.js options, callback_url, payment signature verification, webhook setup, webhook HMAC validation, sandbox/test mode, payment capture, payment status reconciliation, or Malaysia payment gateway go-live checks.
---

# Curlec Payment Gateway

Use this skill for Razorpay Curlec payment gateway integration work.

## Source

Read `references/curlec-payment-gateway.md` before giving factual API guidance or writing integration code. It contains source URLs, integration flow, endpoint map, signature rules, webhook rules, sandbox notes, and common failure fixes from Curlec docs.

## Core Workflow

1. Identify product surface: Standard Checkout, redirect/FPX flow, Payment Links, Orders/Payments/Refunds API, mobile SDK, plugin, or payout/RazorpayX.
2. Identify environment: Test mode or Live mode. Keep keys, orders, webhooks, and dashboard checks separated by mode.
3. Create an internal transaction/order row first, then create a Curlec Order server-side.
4. Pass only the public Key ID and Curlec `order_id` to Checkout. Keep Key Secret server-side only.
5. On Checkout success, store `razorpay_payment_id`, `razorpay_order_id`, and `razorpay_signature`, then verify the signature on the server.
6. Use webhooks for durable asynchronous fulfilment and API fetches for immediate user-facing confirmation where needed.
7. Before fulfilment, verify amount, currency, order mapping, payment status, capture state, signature, idempotency, and duplicate events.

## Critical Rules

- Base URL for most APIs is `https://api.razorpay.com/v1`.
- API auth is Basic Auth with `Key ID:Key Secret`; do not expose Key Secret in browser code.
- Use smallest currency units for amounts, for example sen for MYR. Confirm currency/account support when integrating non-MYR flows.
- Payment signature message is `order_id + "|" + razorpay_payment_id`, where `order_id` is the server-stored Curlec Order ID. Do not trust or substitute client-sent `razorpay_order_id` while generating the signature.
- Webhook signature uses raw request body as message, webhook secret as key, SHA-256 HMAC, and `X-Razorpay-Signature` header. Do not JSON-parse, stringify, or mutate the body before verification.
- Webhooks can be duplicated and out of order. Deduplicate with `x-razorpay-event-id` and use monotonic state transitions.
- Checkout success means successful authorization; verify capture state (`captured`) before irreversible fulfilment unless your business explicitly fulfils on authorization.
- For redirect payment methods such as FPX, configure `callback_url` and `redirect: true`; browser `handler` alone is not enough.
- Old webhook retries may need the old webhook secret after rotation.
- For latest pricing, enabled methods, account limits, webhook IP ranges, dashboard-only settings, or production credentials, verify live Dashboard/docs.

## Loophole Review Loop

When asked whether a Curlec strategy is "100% confident", run this loop until no factual gaps remain:

1. List every trust boundary: browser, backend, Curlec API, webhook receiver, database, fulfilment job, admin dashboard.
2. Check each bypass: fake success callback, wrong amount/currency, replayed webhook, duplicate event, out-of-order event, late authorization, uncaptured payment, secret leakage, test/live key mix, order mismatch, retry timeout, refund double-submit.
3. Add fixes for each bypass: server-side order creation, server-stored order ID for HMAC, raw-body webhook verification, idempotency keys/unique constraints, API status re-fetch, durable event log, strict status machine, env separation, audit logs.
4. Re-read `references/curlec-payment-gateway.md` sections relevant to any gap.
5. State remaining uncertainty explicitly if it depends on account configuration, enabled payment methods, pricing, or current Dashboard state.
