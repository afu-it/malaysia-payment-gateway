---
name: setup-airwallex
description: Set up, build, debug, review, and explain Airwallex online payment integrations for Malaysian merchants using official Airwallex docs, Payment Intents API, Hosted Payment Page, Checkout Sessions, Drop-in and Embedded Elements (Airwallex.js), Native API, FPX, DuitNow QR, Touch 'n Go, Boost, GrabPay, ShopeePay, Atome, cards, Apple Pay, Google Pay, access tokens, scoped API keys, webhooks with x-signature HMAC-SHA256 verification, refunds, cancellations, captures, payment consents, recurring payments, disputes, settlements, sandbox simulation, test cards, and error codes. Use when working with Airwallex Malaysia, MYR payment acceptance, Payment Intents, client_secret checkout, next_action redirects, webhook settlement, refunds, reconciliation, or production readiness.
---

# Airwallex Payments (Malaysia)

Use this skill for Airwallex online payment acceptance work, scoped to what a Malaysian merchant uses (MYR, FPX, DuitNow, local e-wallets, cards, wallets). Airwallex also sells FX, payouts, issuing, spend and treasury; those APIs are listed in the endpoint catalogue only.

## Source

Read `references/airwallex-api.md` first. It is the entry point: environments, auth, amounts, idempotency, end-to-end flow, webhook signature, statuses, errors, rate limits, and the full endpoint catalogue (every documented operation).

Then read the topic file you need:

- `references/airwallex-payment-intents.md`: create, retrieve, list, update, confirm, confirm_continue, capture, cancel, increment authorization, payment attempts; `next_action`; 3DS.
- `references/airwallex-checkout.md`: Hosted Payment Page (`redirectToCheckout`), Drop-in Element, Checkout Sessions API, Payment Links API.
- `references/airwallex-malaysia-methods.md`: FPX, DuitNow, Touch 'n Go, Boost, GrabPay, ShopeePay, Atome, 7-Eleven MY, cards, Apple Pay, Google Pay; limits, settlement, confirm payloads, bank codes, config APIs.
- `references/airwallex-webhooks.md`: subscription, delivery headers, signature verification, event list, IP allowlist, retries, Webhook API.
- `references/airwallex-refunds-disputes.md`: refunds, cancellations, disputes.
- `references/airwallex-recurring.md`: customers, payment consents, payment methods, CIT/MIT, Touch 'n Go recurring.
- `references/airwallex-settlement.md`: settlements, settlement records, financial transactions, financial reports.
- `references/airwallex-testing-errors.md`: sandbox, test cards, simulation APIs, error codes, access token API.
- `references/account-setup.md`: registration, activation, keys, env vars, sandbox vs live, settlement and pricing pointers.
- `references/source-map.md`: every official page used, coverage counts, and what was left out.

## Core Workflow

1. Identify environment: sandbox (`https://api.sandbox.airwallex.com`) or production (`https://api.airwallex.com`).
2. Pick the surface: Hosted Payment Page (JS `redirectToCheckout` or Checkout Session URL), Drop-in Element, Embedded Elements, Native API (card PAN needs PCI-DSS ROC), Payment Links, or a plugin (WooCommerce, Shopify).
3. Confirm the account: Payments activated, each payment method activated in **Payments > Payment methods**, settlement currencies set, webhook created, scoped API key made.
4. Server: get an access token with `POST /api/v1/authentication/login` (`x-client-id`, `x-api-key`), cache it until `expires_at` (30 minutes).
5. Create a local pending order, then `POST /api/v1/pa/payment_intents/create` with `request_id` (UUID), `amount` (major units, `20.50` = RM20.50), `currency: "MYR"`, `merchant_order_id`, `return_url`.
6. Send only `id` + `client_secret` to the browser (valid 60 minutes), or confirm server-side and follow `next_action` (redirect URL or QR text).
7. Treat `return_url`, `successUrl`, and the Drop-in `success` event as UX only.
8. Verify each webhook: HMAC-SHA256 hex of `x-timestamp + raw body` with the webhook secret, constant-time compare, check timestamp tolerance, then parse.
9. Settle idempotently on `payment_intent.succeeded` after a `GET /api/v1/pa/payment_intents/{id}` re-check of `status`, `amount`, `currency`, `merchant_order_id`.
10. Use retrieve as fallback when webhooks are late; dedupe events by `id`; refunds via `POST /api/v1/pa/refunds/create`.

## Core Facts

- Auth: `POST /api/v1/authentication/login` returns `token` + `expires_at`; send `Authorization: Bearer <token>`. Login limit 100/min per key.
- Amounts are decimals in major units (MYR has 2 decimals). Not sen.
- `request_id` is the idempotency key on every POST (max 64 chars); retry with the same value.
- Payment Intent statuses: `REQUIRES_PAYMENT_METHOD`, `REQUIRES_CUSTOMER_ACTION`, `REQUIRES_CAPTURE`, `PENDING`, `PENDING_REVIEW`, `SUCCEEDED`, `CANCELLED` (list is not exhaustive).
- Fulfil on `payment_intent.succeeded`. A failed attempt emits `payment_intent.payment_failed` and the intent reverts to `REQUIRES_PAYMENT_METHOD` (`payment_intent.requires_payment_method`), so the shopper can retry on the same intent.
- Webhook headers: `x-timestamp` (Unix ms) and `x-signature` (hex HMAC-SHA256 over `x-timestamp` string + raw JSON body, key = that webhook's secret). Reply 200 fast; retries back off for about 3 days; order is not guaranteed.
- MY method types: `fpx` (needs `bank_name`), `duit_now`, `tng` (needs `flow`), `boost`, `grabpay`, `shopee_pay`, `atome`, `seven_eleven`, `card`, `applepay`, `googlepay`.
- FPX: MYR 1.01 to 2,000, T+3, refunds within 180 days, no chargebacks, no recurring.
- Rate limits: production 100 req/s global, 20 req/s per endpoint, 50 concurrent; sandbox 20, 10, 10. HTTP 429 `too_many_requests`.
- Latest API version `2026-08-21`; override per call with `x-api-version`.
- Webhook source IPs (production and sandbox) are listed in `references/airwallex-webhooks.md`.

## Critical Rules

- Keep Client ID, API key, access token, and webhook secret server-side. Only `client_secret` may reach the browser. Never log `client_secret`.
- Compute the amount on the server; never accept it from the browser.
- Verify the signature on the raw body before any JSON parsing. Use the secret of the exact webhook subscription (each URL has its own). Test events are signed with the key in the `client-secret-key` header.
- Do not mark paid from a redirect, `successUrl`, Drop-in `success`, a QR display, or the confirm response alone; FPX, DuitNow, and wallets finish asynchronously.
- Re-check status, amount, currency, and `merchant_order_id` against your record before settling; store intent `id` and event `id` with unique constraints.
- Sandbox and production use different credentials, hosts, webhook secrets, and webhook IPs.
- Create a new Payment Intent when `client_secret` expires; it cannot be refreshed. Update the intent (not a new one) if the cart changes before confirm.
- Raw card data via Native API only with PCI-DSS ROC and Airwallex approval; otherwise use HPP, Drop-in, or Elements (SAQ A).
- Use scoped API keys and IP whitelisting for production keys.
- Pricing, limits, activation windows, and availability change; link to official pages, confirm in the web app.

## Completion Criteria

An Airwallex integration is done only when:

1. Token is fetched server-side, cached, and refreshed on 401 or near `expires_at`.
2. Payment Intent is created server-side with a unique `request_id`, server-computed `amount`, `currency`, `merchant_order_id`, and `return_url`.
3. The frontend gets only `intent_id`, `client_secret`, `currency`; checkout uses HPP, Drop-in, Elements, a Checkout Session URL, or a server confirm with `next_action` handling.
4. A webhook subscription exists for the right environment with at least `payment_intent.succeeded`, `payment_intent.payment_failed`, `payment_intent.cancelled`, and `refund.*`.
5. The handler verifies `x-signature` over `x-timestamp + raw body`, enforces timestamp tolerance, responds 200 quickly, and processes idempotently by event `id`.
6. Settlement re-reads the intent and checks status, amount, currency, and order id; duplicates and out-of-order events cannot double-fulfil.
7. Tests cover bad signature, replay, wrong amount or currency, failed payment, expired `client_secret`, FPX or DuitNow pending then succeeded (sandbox simulation), refund success and failure.
8. Deployment docs list `AIRWALLEX_CLIENT_ID`, `AIRWALLEX_API_KEY`, `AIRWALLEX_WEBHOOK_SECRET`, `AIRWALLEX_ENV`, return URL, webhook URL, and sandbox/live separation.

## Loophole Review Loop

When asked whether an Airwallex strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser, Airwallex.js, hosted page, backend token cache, intent creator, Airwallex API, webhook receiver, raw-body parser, database, fulfilment job, refund job, web app settings, payment method activation.
2. Check bypasses: fake success redirect, forged webhook, signature over parsed JSON, wrong webhook secret, sandbox/live mix, stale or leaked token, amount tampering, wrong currency, reused `merchant_order_id`, duplicate or out-of-order events, expired `client_secret`, unactivated method (`payment_method_not_allowed`), FPX over RM2,000, refund failing for lack of refundable funds, DuitNow 5-minute timeout marked failed too early.
3. Add fixes: server-side creation, raw-body HMAC with constant-time compare, timestamp tolerance, retrieve-before-settle, unique keys on intent and event ids, per-environment config, scoped keys, IP allowlists, status fallback polling.
4. Re-read the relevant reference file for any gap.
5. State remaining uncertainty that depends on account activation, reserve plan, settlement terms, pricing, or live availability.
