---
name: setup-hitpay
description: Set up, build, debug, review, and explain HitPay payment integrations using official HitPay docs, Payment Request API, hosted checkout, Drop-In UI, webhooks, HMAC-SHA256 signatures, sandbox/live setup, API keys, webhook salt, payment methods, plugins, refunds, recurring billing, payouts, platform APIs, and status page checks. Use when working with HitPay Malaysia, FPX, DuitNow, Touch 'n Go, cards, e-wallets, QR payments, Payment Links, API checkout, webhook settlement, refunds, or production readiness.
---

# HitPay Payment Gateway

Use this skill for HitPay payment gateway integration work.

## Source

Read `references/hitpay-api.md` before giving factual API guidance or writing integration code. It contains source URLs, environment bases, auth, payment request flow, checkout presentation, webhooks, status checks, refunds, rate limits, sandbox, IP allowlisting, and settlement/security cautions.

Read `references/account-setup.md` when user asks how to register, which credentials are needed, where keys come from, sandbox setup, dashboard setup, payment method activation, Malaysia-specific methods, pricing, payout timing, or operational status.

Read `references/source-map.md` when choosing which HitPay docs page applies to API checkout, webhooks, sandbox, refunds, plugins, payment methods, status, recurring billing, payouts, platform APIs, or production readiness.

## Core Workflow

1. Identify environment: sandbox or production.
2. Identify integration surface: Payment Request API, hosted checkout redirect, Drop-In UI, embedded QR, plugin, payment link, invoice, recurring billing, payout, platform account, or POS.
3. Confirm dashboard setup: API key, webhook endpoint, webhook salt, enabled payment methods, business verification, and environment-specific URLs.
4. Create a local pending payment before creating the HitPay Payment Request.
5. Create the Payment Request server-side with `X-BUSINESS-API-KEY`.
6. Send the customer to the returned `url`, or render the supported Drop-In UI using values returned by the server-side flow.
7. Treat `redirect_url` as browser UX only; do not fulfil from redirect alone.
8. Verify webhook `Hitpay-Signature` using HMAC-SHA256 over the raw JSON payload and the correct environment salt.
9. Settle idempotently only after payment request ID, local reference, amount, currency, status, environment, merchant account, and payment method match expectations.
10. Use status requery as fallback when webhooks are delayed or failed; do not replace webhooks with polling.

## Core Facts

- Production API base: `https://api.hit-pay.com`.
- Sandbox API base: `https://api.sandbox.hit-pay.com`.
- Payment Request endpoint: `POST /v1/payment-requests`.
- Payment status endpoint: `GET /v1/payment-requests/{request_id}`.
- Auth uses the `X-BUSINESS-API-KEY` header.
- API keys are found in Dashboard > Settings > API Keys.
- Webhooks are registered in Dashboard > Developers > Webhook Endpoints.
- Webhook signatures use the `Hitpay-Signature` header and HMAC-SHA256 over the raw JSON payload with the dashboard salt.
- For API checkout, HitPay says orders should be marked paid only after a webhook is received and validated.
- Payment Request statuses include `pending`, `completed`, `failed`, `expired`, `canceled`, and `inactive`.
- API overview rate limits are 400 requests per minute generally and 70 payment creations per minute.
- Sandbox and production are separate accounts with separate keys, salts, URLs, dashboards, webhooks, and request logs.
- Sandbox dashboard: `https://dashboard.sandbox.hit-pay.com/login`.
- Production dashboard: `https://dashboard.hit-pay.com`.
- Webhook IPs documented by HitPay: production `3.1.13.32`, `52.77.254.34`; sandbox `54.179.156.147`.
- HitPay status page: `https://status.hitpayapp.com/`.

## Critical Rules

- Keep API keys and webhook salt server-side only.
- Do not mark paid from `redirect_url`, frontend callback, Drop-In UI result, QR display, or dashboard screenshot alone.
- Always validate webhook signatures against the raw request body before parsing or normalizing JSON.
- Use constant-time signature comparison.
- Use the salt and API key for the same environment as the webhook/API base URL.
- Store provider request ID, local reference, expected amount, expected currency, selected environment, and settlement state before redirecting the customer.
- Make webhook processing idempotent; duplicate events or retries must not double-fulfil.
- Confirm amount, currency, payment request ID, local `reference_number`, and status before marking paid.
- Treat webhook IP allowlisting as a network hardening step, not as a substitute for HMAC verification.
- Use separate webhook URLs for sandbox and production.
- Use webhook confirmation as the default settlement path and status API only as a fallback.
- Do not rely on the deprecated payment request `webhook` parameter when dashboard webhook endpoints are the correct current path for centralized webhook management.
- Verify live payment method availability, activation time, payout schedule, pricing, current incidents, and business verification in the dashboard/docs/status page before claiming production certainty.

## Completion Criteria

A HitPay integration is done only when:

1. Payment Request creation runs server-side with `X-BUSINESS-API-KEY`.
2. Redirect checkout, Drop-In UI, or embedded QR returns only provider state to the backend; it does not fulfil access directly.
3. Dashboard webhook endpoint is registered for the correct environment.
4. Webhook handler verifies `Hitpay-Signature` over the raw request body with the correct webhook salt.
5. Settlement checks payment request ID, local `reference_number`, amount, currency, status, environment, and merchant account.
6. Duplicate webhooks, delayed QR completion, and status requery are idempotent.
7. Tests cover invalid signature, duplicate webhook/event replay, wrong amount/currency/reference, failed/expired status, and successful completed payment.
8. Deployment docs include `HITPAY_API_KEY`, `HITPAY_WEBHOOK_SALT` or the app's equivalent webhook secret name, redirect URL, webhook URL, and sandbox/live separation.

## Loophole Review Loop

When asked whether a HitPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, Drop-In UI, checkout page, backend payment creator, HitPay API, webhook receiver, raw-body parser, database, fulfilment job, refund job, dashboard settings, status page, and payment method activation.
2. Check bypasses: fake success redirect, forged webhook, signature over parsed JSON instead of raw body, wrong salt, sandbox/live mix, wrong API key, wrong amount, wrong currency, wrong reference, stale status poll, duplicate webhook, webhook retry after partial processing, abandoned QR marked failed too early, inactive payment method, unverified business, unavailable HitPay service, duplicate refund, and plugin webhook blocked by server security.
3. Add fixes: server-side Payment Request creation, local pending payment first, raw-body HMAC verification, constant-time comparison, strict amount/currency/reference checks, durable event log, idempotency keys, separate environment config, webhook IP allowlisting where needed, status page check, status API fallback, and dashboard capability verification.
4. Re-read `references/hitpay-api.md` sections relevant to any gap.
5. State remaining uncertainty if it depends on current account verification, enabled payment methods, activation windows, payout schedule, pricing, platform approval, plugin behavior, or live operational status.
