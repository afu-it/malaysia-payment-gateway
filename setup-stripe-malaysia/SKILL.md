---
name: setup-stripe-malaysia
description: Set up, build, debug, review, and explain Stripe payment gateway integrations for Malaysia. Use when working with Stripe Checkout Sessions, Payment Intents, Payment Links, Elements, webhooks, Stripe.js, Stripe CLI, refunds, sandbox/live setup, MYR payments, cards, FPX, GrabPay, and production readiness.
---

# Stripe Malaysia Payment Gateway

Use this skill for Stripe payment gateway integration work in Malaysia.

## Source

Read `references/stripe-api.md` before giving factual API guidance or writing integration code. It contains source URLs, environment bases, auth, Checkout Sessions flow, Payment Intents guidance, webhooks (Stripe-Signature verification), Stripe.js initialization, CLI testing, test mode cards, and settlement/security cautions.

Read `references/account-setup.md` when user asks how to register, which credentials are needed, where keys come from, sandbox setup, dashboard setup, payment method activation, Malaysia-specific methods, pricing, or operational status.

Read `references/source-map.md` when choosing which Stripe docs page applies to API checkout, webhooks, CLI, SDKs, Elements, Payment Links, subscriptions, or samples.

## Core Workflow

1. Identify environment: test mode or live mode.
2. Identify integration surface: Checkout Sessions (hosted page, embedded form, or embedded Elements), Payment Intents, Payment Links, or Elements-only.
3. Confirm dashboard setup: API keys (publishable + secret), webhook endpoint, webhook signing secret, enabled payment methods, and environment-specific configuration.
4. Create a local pending payment before creating the Stripe Checkout Session or PaymentIntent.
5. Create the Checkout Session or PaymentIntent server-side with the secret key.
6. For hosted Checkout: redirect the customer to the returned `url`. For embedded: pass the `client_secret` to the frontend.
7. Treat `success_url` redirect as browser UX only; do not fulfil from redirect alone.
8. Verify webhook `Stripe-Signature` using the Stripe library's `constructEvent` method over the raw request body with the webhook signing secret.
9. Settle idempotently only after session/PaymentIntent ID, local reference, amount, currency, payment status, and environment match expectations.
10. Use session/PaymentIntent retrieval as fallback when webhooks are delayed or failed; do not replace webhooks with polling.

## Core Facts

- API base: `https://api.stripe.com`.
- Auth: HTTP Bearer `Authorization: Bearer {secret_key}`.
- Secret keys start with `sk_live_` or `sk_test_`.
- Publishable keys start with `pk_live_` or `pk_test_`. These are safe for client-side use.
- Checkout Sessions endpoint: `POST /v1/checkout/sessions`.
- Payment Intents endpoint: `POST /v1/payment_intents` (only when Checkout Sessions cannot meet requirements).
- Webhook signing secrets start with `whsec_`. Each webhook endpoint has its own secret.
- Webhook signatures use the `Stripe-Signature` header with HMAC-SHA256 over `{timestamp}.{raw_body}`.
- For Checkout Sessions, Stripe recommends fulfilling orders from the `checkout.session.completed` webhook event.
- For async payment methods (like FPX bank redirect), use `checkout.session.async_payment_succeeded`.
- Checkout Session `payment_status` values: `paid`, `unpaid`, `no_payment_required`.
- Test cards: `4242 4242 4242 4242` (Visa), any future expiry, any CVC.
- Stripe CLI: `stripe listen --forward-to` for local webhook testing, `stripe trigger` for event generation.
- Malaysia Stripe accounts support MYR, cards, FPX, and GrabPay. Verify current availability in the Dashboard.
- Stripe supports 125+ payment methods globally.

## Critical Rules

- Keep secret API keys and webhook signing secrets server-side only.
- Never use secret keys in frontend code, mobile apps, or client-side environment variables.
- Do not mark paid from `success_url` redirect or client-side status alone.
- Always validate webhook signatures using the Stripe library's `constructEvent`/`construct_event` with the raw request body before parsing JSON.
- Use the exact webhook signing secret matching the endpoint that received the event.
- Store provider session ID, local reference, expected amount, expected currency, and environment before redirecting the customer.
- Make webhook processing idempotent by logging processed event IDs; duplicate events or retries must not double-fulfil.
- Confirm amount, currency, session/PaymentIntent ID, local reference, and payment status before marking paid.
- Use separate webhook URLs for test mode and live mode.
- Use webhook confirmation as the default settlement path and API retrieval only as a fallback.
- Do not hardcode the temporary webhook secret from `stripe listen` — it changes each restart. Use it only for local development.
- Verify live payment method availability, activation, fees, settlement timing, and current incidents in the Dashboard/docs/status page before claiming production certainty.
- Stripe recommends Checkout Sessions over Payment Intents for most integrations. Only use Payment Intents when the user explicitly needs custom payment UI or deeply owned checkout state.

## Completion Criteria

A Stripe integration is done only when:

1. Checkout Session or PaymentIntent creation runs server-side with the secret key.
2. Redirect checkout, embedded checkout, or Elements returns only provider state to the backend; it does not fulfil access directly.
3. Webhook endpoint is registered in the Stripe Dashboard or tested with `stripe listen`.
4. Webhook handler verifies `Stripe-Signature` over the raw request body with the correct webhook signing secret.
5. Settlement checks session/PaymentIntent ID, local reference, amount, currency, and payment status.
6. Duplicate webhooks and event retries are idempotent.
7. Tests cover invalid signature, duplicate webhook/event replay, wrong amount/currency/reference, failed/expired status, and successful completed payment.
8. Deployment docs include `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`, `STRIPE_WEBHOOK_SECRET`, redirect URL, webhook URL, and test/live separation.

## Loophole Review Loop

When asked whether a Stripe strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, checkout page, backend payment creator, Stripe API, webhook receiver, raw-body parser, database, fulfilment job, refund job, dashboard settings, status page, and payment method activation.
2. Check bypasses: fake success redirect, forged webhook, signature over parsed JSON instead of raw body, wrong webhook secret, test/live mode mix, wrong API key, wrong amount, wrong currency, wrong reference, stale status poll, duplicate webhook, webhook retry after partial processing, inactive payment method, unavailable Stripe service, duplicate refund.
3. Add fixes: server-side Checkout Session/PaymentIntent creation, local pending payment first, raw-body signature verification with Stripe library, strict amount/currency/reference checks, durable event log, idempotency keys, separate environment config, IP allowlisting where needed, status page check, API retrieval fallback, and dashboard capability verification.
4. Re-read `references/stripe-api.md` sections relevant to any gap.
5. State remaining uncertainty if it depends on current account verification, enabled payment methods, activation windows, settlement timing, pricing, or live operational status.
