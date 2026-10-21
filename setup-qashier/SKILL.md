---
name: setup-qashier
description: Set up, build, debug, review, and explain Qashier payment integrations using official Qashier docs, QashierPay Online Checkout Session API, hosted checkout redirect, HTTP Basic Auth API keys, Idempotency-Key, X-Qashier-Signature HMAC-SHA256 webhooks, checkout.session.completed and checkout.session.expired events, refunds, staging/production setup, test cards, and the In-Store Payment Open API for QashierPay terminals. Use when working with Qashier Malaysia, QashierPay Online, Qashier checkout sessions, DuitNow QR, cards, e-wallets, Pay-by-Link, Shopify/WooCommerce/Wix plug-ins, terminal payment initiate/cancel/void/status, webhook settlement, refunds, or production readiness.
---

# Qashier Payment Gateway

Use this skill for Qashier (QashierPay Online and QashierPay terminal) payment integration work.

## Source

Read `references/qashier-api.md` before giving factual API guidance or writing integration code. It is the entry point: environments, auth, endpoint catalogue, Checkout Session fields, webhooks and the exact signature algorithm, refunds, errors, rate limits, test cards, and documented gaps.

Read `references/qashier-instore.md` only when the user integrates their own POS with a QashierPay card terminal (partner API with `x-qashier-partner` and `x-qashier-sign`).

Read `references/account-setup.md` when the user asks how to sign up, where keys come from, staging vs production, payment method activation, plug-ins, Pay-by-Link, settlement, or pricing.

Read `references/source-map.md` to pick the right official page or to check coverage.

## Core Workflow

1. Identify the surface: QashierPay Online hosted checkout (API), plug-in (Shopify, WooCommerce, Wix), Pay-by-Link (no code), or in-store terminal partner API.
2. Identify environment: staging (`api.staging.qashier.com`) or production (`api.qashier.com`), with matching API key.
3. Create a local pending order first, with amount in sen and currency `MYR`.
4. Create the Checkout Session server-side: `POST /checkout/sessions` with Basic Auth (API key as username, empty password) and an `Idempotency-Key` header.
5. Store the returned session `id` against the local order, then redirect the customer (HTTP 303) to the returned `url`.
6. Treat `success_url` and `return_url` as browser UX only; never fulfil from them.
7. Verify `X-Qashier-Signature` on every webhook over the raw request body before parsing JSON.
8. On `checkout.session.completed`, fulfil only when `data.object.payment_status` is `paid`; `unpaid` means the attempt was declined.
9. Re-fetch the session with `GET /checkout/sessions/{id}` and match id, amount, currency, status before settling idempotently.
10. For abandonment, retrieve the session after `expires_at` and confirm `status: expired`; no webhook fires for natural expiry.

## Core Facts

- Online API base: production `https://api.qashier.com/v3/payment/qpay`, staging `https://api.staging.qashier.com/v3/payment/qpay`.
- Auth: HTTP Basic, API key as username, empty password (`Authorization: Basic base64("<API_KEY>:")`). Keys are managed in the Qashier Dashboard.
- Endpoints: `POST /checkout/sessions`, `GET /checkout/sessions`, `GET /checkout/sessions/{id}`, `POST /checkout/sessions/{id}/expire`, `POST /refund`.
- Create requires `currency`, `line_items[]` (`unit_amount` integer in cents, `quantity` limited to 1), `return_url`, `success_url`; optional `expires_at`, `customer_email`.
- `expires_at` is 30 minutes to 24 hours after creation; default 24 hours.
- Session `status`: `open`, `complete`, `expired`. `payment_status`: `paid`, `unpaid`.
- Webhook events: `checkout.session.completed`, `checkout.session.expired` (only when you call expire), and refund updates.
- Signature: header `t=<ts>,v1=<hex>`; `signing_key = SHA256(webhook_secret)`; `v1 = HMAC_SHA256(signing_key, "<t>.<raw_body>")`; constant-time compare.
- Refunds: `POST /refund` with `sessionId` and optional `amount` in cents; async, status `pending`, `succeeded`, `failed`.
- Idempotency keys: max 255 chars, kept 24 hours, reuse with different params returns 409 `idempotency_error`.
- Rate limit: 100 requests per second per API key; 429 `rate_limit_error`.
- Merchants do not pick payment methods per session; checkout shows what the account has enabled.

## Critical Rules

- Keep the API key and webhook secret server-side only.
- Never fulfil on the event name alone; `checkout.session.completed` is sent for declines too.
- Verify the signature over the exact raw bytes received; do not re-serialize JSON.
- Do not infer abandonment from webhook silence; retrieve the session and check `status`.
- Use separate keys, webhook URLs, and secrets for staging and production.
- Make settlement idempotent on session `id` (and webhook event `id`); duplicates must not double-fulfil.
- Send amounts as integers in sen (RM20.00 = `2000`); confirm amount and `MYR` currency on settlement.
- Ignore the docs page `/docs/api/payments-example` (`POST /v1/payments`, Bearer token): it is a template example, not the real API.
- Confirm documented gaps (webhook URL registration, secret location, signing key encoding, timestamp tolerance) with Qashier before production; see `references/qashier-api.md`.

## Completion Criteria

A Qashier integration is done only when:

1. Checkout Session creation runs server-side with Basic Auth and an `Idempotency-Key`.
2. The local order stores session `id`, expected amount, currency, environment, and state before redirect.
3. The webhook handler verifies `X-Qashier-Signature` over the raw body with the correct environment secret, constant-time.
4. Settlement requires `payment_status: paid`, matching session id, amount, and currency, confirmed by a server-side retrieve.
5. Declines (`unpaid`), duplicate events, delayed webhooks, and natural expiry are handled idempotently.
6. Refunds use `POST /refund`, and refund state is reconciled from refund webhooks or `refund_details`.
7. Tests cover invalid signature, replayed event, declined card (`unpaid`), wrong amount/currency, expired session, and a paid session, using the documented test cards on staging.
8. Deployment docs list `QASHIER_API_KEY`, `QASHIER_WEBHOOK_SECRET`, `QASHIER_API_BASE`, success/return URLs, webhook URL, and staging/production separation.

## Loophole Review Loop

When asked whether a Qashier strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser, success/return pages, backend session creator, Qashier API, webhook receiver, raw-body parser, database, fulfilment job, refund job, dashboard settings, and payment method activation.
2. Check bypasses: fake success redirect, forged webhook, signature over parsed JSON, wrong secret or key encoding, staging/live mix, `completed` treated as paid, wrong amount or currency, replayed event with old timestamp, duplicate webhook, abandoned session marked failed too early, refund double-sent, disabled payment method.
3. Add fixes: server-side session creation, local pending order first, raw-body HMAC verification, constant-time compare, timestamp age check, retrieve-before-settle, strict amount/currency/id checks, durable event log, idempotency keys, separate environment config.
4. Re-read `references/qashier-api.md` sections relevant to any gap.
5. State remaining uncertainty where it depends on undocumented items (webhook URL setup, secret location, timestamp tolerance, refund idempotency), account approval, enabled methods, settlement timing, or pricing.
