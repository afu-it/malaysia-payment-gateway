---
name: setup-paypal
description: Set up, build, debug, review, and explain PayPal payment integrations for Malaysian merchants using official PayPal developer docs, REST API OAuth 2.0, Orders v2 create/capture/authorize, Payments v2 refunds/voids/reauthorization, JavaScript SDK v6 and v5 buttons, webhooks with CRC32 plus SHA256withRSA signature verification or verify-webhook-signature postback, Subscriptions and Catalog Products, Transaction Search, Payouts, disputes, sandbox accounts, test cards, negative testing, MYR rules, and go-live. Use when working with PayPal Checkout, PayPal buttons, PayPal REST APIs, MYR payments, cross-border sales from Malaysia, PayPal webhooks, PayPal recurring billing, refunds, reconciliation, or production readiness.
---

# PayPal Payment Gateway (Malaysia)

Use this skill for PayPal integration work for a Malaysia-registered PayPal Business account.

## Source

Read `references/paypal-api.md` first. It has the Malaysia scope summary, environments, auth, headers, amount rules, the default checkout flow, statuses, and the full endpoint catalogue.

Then load only what the task needs:

- `references/paypal-orders.md`: Orders v2 and Payments v2 field tables, capture checks, PATCH, refunds.
- `references/paypal-webhooks.md`: signature verification, headers, events, Webhooks Management API.
- `references/paypal-checkout-js.md`: JavaScript SDK v6 and v5, callbacks, server routes.
- `references/paypal-subscriptions.md`: products, plans, subscriptions, failure retry.
- `references/paypal-errors-testing.md`: error codes, sandbox, test cards, negative testing.
- `references/paypal-reporting-payouts.md`: Transaction Search, Payouts (MYR rules), disputes.
- `references/account-setup.md`: registration, credentials, env vars, dashboard settings, Malaysian account notes, pricing link.
- `references/source-map.md`: which official page backs each fact, coverage, and known gaps.

## Core Workflow

1. Identify environment (sandbox or live) and use matching credentials, host, client ID and Webhook ID.
2. Identify surface: JS SDK buttons (v6 for new work), server redirect to the approval link, Subscriptions, Payment Links, or Payouts.
3. Create a local pending payment with amount, `MYR` (or the chosen currency) and your reference.
4. Server: get an OAuth token (cache it), then `POST /v2/checkout/orders` with `intent: CAPTURE`, `custom_id`/`invoice_id`, and a `PayPal-Request-Id`.
5. Browser: buyer approves through the SDK or the `approve` / `payer-action` link.
6. Server: `POST /v2/checkout/orders/{id}/capture` with a `PayPal-Request-Id`.
7. Settle only when `purchase_units[].payments.captures[].status` is `COMPLETED` and amount, currency and reference match.
8. Verify every webhook signature over the raw body, deduplicate by event `id`, re-fetch the order or capture, then settle idempotently.
9. Handle `PENDING` captures, `INSTRUMENT_DECLINED`, reversals, refunds and disputes.

## Core Facts

- REST base: sandbox `https://api-m.sandbox.paypal.com`, live `https://api-m.paypal.com`.
- Token: `POST /v1/oauth2/token`, Basic auth `client_id:secret`, body `grant_type=client_credentials`; use `Authorization: Bearer <token>` until `expires_in`.
- Orders: `POST /v2/checkout/orders`, `GET /v2/checkout/orders/{id}`, `POST /v2/checkout/orders/{id}/capture`, `/authorize`, `PATCH /v2/checkout/orders/{id}`.
- Payments: `POST /v2/payments/captures/{id}/refund`, `GET /v2/payments/captures/{id}`, authorization `capture`, `void`, `reauthorize`.
- Amounts are decimal strings in `{"currency_code","value"}`; not sen.
- MYR is supported as payment and balance currency for in-country (Malaysian) PayPal accounts only.
- Order `COMPLETED` can still hold a declined payment; check the capture status.
- Webhook self-verify string: `transmission_id|transmission_time|webhook_id|crc32(raw body, decimal)`, verified with the cert at `paypal-cert-url` (SHA256withRSA). Postback: `POST /v1/notifications/verify-webhook-signature` must return `verification_status: SUCCESS`.
- Webhook retries: up to 25 times over 3 days for non-2xx. Up to 10 webhook URLs per app.
- JS SDK v6: `https://www.paypal.com/web-sdk/v6/core` (sandbox `https://www.sandbox.paypal.com/web-sdk/v6/core`). v5: `https://www.paypal.com/sdk/js?client-id=...&currency=MYR`.
- Authorization: 29-day validity, 3-day honor period. Refund within 180 days of capture.
- No published rate limit; `429 RATE_LIMIT_REACHED` on abuse. Use webhooks, not polling.
- Not available to Malaysian merchants per the docs: FPX, DuitNow, local e-wallets, Expanded Checkout card fields, card vaulting, Pay Later, PayPal Credit, Venmo. GrabPay via PayPal is Singapore only.

## Critical Rules

- Keep the client secret server-side. Only the client ID goes to the browser.
- Create and price orders on the server; never trust amounts sent by the browser.
- Never fulfil from `onApprove`, a redirect to `return_url`, or a browser-sent status.
- Verify webhooks over the raw body bytes with the Webhook ID of the same environment; never re-serialize JSON before CRC32 or postback.
- Use `PayPal-Request-Id` on create, capture, authorize and refund; reuse the same ID when retrying a timeout or 5xx.
- Treat `ORDER_ALREADY_CAPTURED` as "fetch and settle", not as failure.
- Keep `invoice_id` unique per order; duplicates are rejected by default.
- Match SDK `currency` and `intent` to the API order.
- Set Payment Receiving Preferences so MYR or foreign-currency payments do not sit pending for manual acceptance.
- Do not use the API for personal payments in Malaysia (PayPal policy).
- Do not claim local Malaysian methods or Expanded Checkout features for a Malaysian account.

## Completion Criteria

A PayPal integration is done only when:

1. Orders are created and captured server-side with cached OAuth tokens and `PayPal-Request-Id`.
2. The browser only returns the order ID; the server decides paid state from the capture response or a verified webhook.
3. Webhooks are subscribed per environment and verified (self or postback) over the raw body.
4. Settlement checks order ID, capture ID, capture status, amount, currency, `custom_id`/`invoice_id` and environment.
5. Duplicate webhooks, capture retries, pending captures, reversals and refunds are idempotent.
6. Tests cover bad signature, replayed event, `INSTRUMENT_DECLINED`, pending capture, duplicate invoice ID, partial and over refund.
7. Deployment docs list `PAYPAL_CLIENT_ID`, `PAYPAL_CLIENT_SECRET`, `PAYPAL_WEBHOOK_ID`, API base, webhook URL and sandbox/live separation.

## Loophole Review Loop

When asked whether a PayPal strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser SDK, return/cancel URLs, server order creator, OAuth token cache, PayPal API, webhook receiver, raw-body parser, cert download, database, fulfilment, refund job, dashboard settings.
2. Check bypasses: fake `onApprove`, client-sent amount, forged webhook, re-serialized body, wrong environment Webhook ID, sandbox/live mix, `COMPLETED` order with declined capture, pending capture fulfilled, duplicate webhook, capture retry without idempotency key, MYR pending manual acceptance, reversal after fulfilment, duplicate refund, cert URL from a non-PayPal host.
3. Add fixes: server-side pricing, local pending record, capture-status checks, raw-body verification, event ID dedupe, `PayPal-Request-Id`, environment-scoped config, re-fetch before settle, reversal and dispute handlers.
4. Re-read the relevant reference file for any gap.
5. State remaining uncertainty that depends on account eligibility, receiving preferences, fees, PayPal-wallet vaulting for Malaysia, or undocumented fields listed in `references/source-map.md`.
