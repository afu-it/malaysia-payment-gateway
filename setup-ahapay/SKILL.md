---
name: setup-ahapay
description: Set up, build, debug, review, and explain AhaPay payment integrations using official AhaPay docs, External API V2 (and legacy V1), X-ApiKey auth, Create Default Order hosted checkout, POS and QR POS orders, online_order_payment_link redirect, callback_url webhooks, X-Api-Key-Hash (SHA-256 of the API key) callback verification, order status requery, full refunds, daily limit status, health check, Merchant vs Partner API keys, external_merchant_id, sandbox/production setup, and the official WooCommerce plugin. Use when working with AhaPay Malaysia, Aharetail Solutions, Buy Now Pay Later (BNPL) instalment checkout, MYR orders, WooCommerce or EasyStore AhaPay setup, webhook settlement, refunds, or production readiness.
---

# AhaPay Payment Gateway

Use this skill for AhaPay (Malaysian BNPL) payment integration work.

## Source

Read `references/ahapay-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, environments, auth, every order-creation field, response fields, status values, callback format and verification, status requery, refunds, daily limit, health check, and the gaps the official docs leave open.

Read `references/account-setup.md` when the user asks how to register, which documents are needed, where the API key comes from, Merchant vs Partner keys, sandbox access, plugin setup (WooCommerce, EasyStore), minimum amounts, payout, fees, or support contacts.

Read `references/source-map.md` when choosing which AhaPay docs page or plugin file backs a fact, or to see which pages were read and what is not documented.

## Core Workflow

1. Identify environment: sandbox (`https://api.bnpl-my.dev.mychili.id`) or production (`https://api.ahapay.my`).
2. Identify the key type: Merchant (Trade Outlet) API key for direct integration, or Partner API key (needs `external_merchant_id` in every create request).
3. Identify the surface: online checkout (`POST /v2/orders/default`), POS screen QR (`POST /v2/orders/pos`), QR POS scanner (`POST /v2/orders/qr_pos`), or a platform plugin (WooCommerce, EasyStore).
4. Create a local pending order first, with its expected amount in MYR.
5. Create the AhaPay order server-side with `X-ApiKey`, passing your local ID as `reference_number` and an HTTPS `callback_url`.
6. Store `online_order_id` from the response, then redirect the customer to `online_order_payment_link` (online) or show `link_qr` (POS).
7. Treat `success_redirect_url` / `failed_redirect_url` as browser UX only; never fulfil from a redirect.
8. On callback, check `X-Api-Key-Hash` equals hex SHA-256 of your API key (constant-time), return HTTP 200 fast.
9. Because that hash does not sign the body, confirm with `GET /v2/orders/{online_order_id}/status` before settling, then match `online_order_id`, `reference_number`, amount, and status.
10. Settle idempotently on `PAYMENT_SUCCESSFUL`; never let a late `PAYMENT_IN_PROGRESS` move a paid order back.

## Core Facts

- Production API base: `https://api.ahapay.my`. Sandbox API base: `https://api.bnpl-my.dev.mychili.id`.
- Auth header on every request: `X-ApiKey: <API_KEY>`. The server decides whether it is a Merchant or Partner key.
- Current API is V2: `POST /v2/orders/default`, `POST /v2/orders/pos`, `POST /v2/orders/qr_pos`, `GET /v2/orders/{online_order_id}/status`, `POST /v2/orders/{online_order_id}/refund`, `GET /v2/daily_limit_status`, `GET /v2/health`.
- Legacy V1 (`POST /v1/orders`, `GET /v1/orders/{id}/status`, `POST /v1/orders/{id}/refund`, `GET /v1/health`) is still documented; the official plugin 1.1.0 migrated to V2.
- Amounts are decimal MYR (ringgit, not sen), for example `288.90`. MYR is the only documented currency.
- Customer must be sent to `online_order_payment_link`; `link` is the V1 app deeplink per the official plugin.
- Order statuses: `PAYMENT_IN_PROGRESS`, `PAYMENT_SUCCESSFUL`, `CANCELLED`, `REFUNDED`, `UNAVAILABLE`.
- Callback: POST to the order's `callback_url` on status change, header `X-Api-Key-Hash`, body with `online_order_status`. Must answer HTTP 200; timeout is 65 seconds.
- Official plugin verifies `X-Api-Key-Hash` as `hash('sha256', api_key)` (64 hex chars) with `hash_equals`.
- Refunds are full-order only: `POST /v2/orders/{online_order_id}/refund`, no body.
- QR POS `client_hash` is sent without the `aha-` prefix and lives 10 minutes.
- HTTP codes: 200 OK, 401 outdated/archived key or partner not bound to outlet, 404 outlet or partner not found, 500 invalid key format or key never created.
- Official WooCommerce plugin: `ahapay-buy-now-pay-later` on WordPress.org, default minimum order 55 MYR.

## Critical Rules

- Keep the API key server-side only; it is both the request credential and the callback shared secret.
- Do not mark paid from a redirect, the AhaPay app screen, or a callback alone; requery status first.
- Compare `X-Api-Key-Hash` with a constant-time function and reject non-64-hex values.
- The hash authenticates the sender, not the payload or the order: check that the callback's `online_order_id` is the one you stored for that `reference_number`.
- Check the requeried amount equals your stored order total before fulfilment; flag mismatches for manual review.
- Make callback handling idempotent and lock per order; duplicates and out-of-order deliveries happen.
- Allow only `REFUNDED` out of a paid state; ignore regressions to `PAYMENT_IN_PROGRESS`.
- Never send a second refund when the first one timed out or gave no clear answer; requery status first.
- Use the API key and base URL of the same environment; sandbox and production keys differ.
- For Partner keys, always send `external_merchant_id`; for Merchant keys leave it out.
- Confirm current fees, limits, payout schedule, and enabled outlets with AhaPay before claiming production certainty.

## Completion Criteria

An AhaPay integration is done only when:

1. Order creation runs server-side with `X-ApiKey` on the right base URL, with `reference_number` and `callback_url` set.
2. `online_order_id` and the expected amount are stored before the customer is redirected to `online_order_payment_link`.
3. The callback endpoint is HTTPS, verifies `X-Api-Key-Hash`, returns 200 within 65 seconds, and is reachable from the internet.
4. Settlement requeries `GET /v2/orders/{online_order_id}/status` and checks ID, reference, amount, and status.
5. Duplicate, out-of-order, and forged callbacks cannot double-fulfil or un-pay an order.
6. Refund flow sends full refunds only, once, and reconciles unknown outcomes by requery.
7. Tests cover a bad hash, missing hash, unknown order ID, amount mismatch, duplicate success, late in-progress after success, cancelled, unavailable, and refunded.
8. Deployment docs list `AHAPAY_API_KEY`, `AHAPAY_BASE_URL`, optional `AHAPAY_EXTERNAL_MERCHANT_ID`, the callback URL, and sandbox/live separation.

## Loophole Review Loop

When asked whether an AhaPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, AhaPay payment page and app, backend order creator, AhaPay API, callback receiver, database, fulfilment job, refund job, plugin settings, and the AhaPay merchant portal.
2. Check bypasses: fake success redirect, forged callback with a leaked key hash, callback naming another order's `reference_number`, wrong environment key, wrong amount, stale status, duplicate callback, late `PAYMENT_IN_PROGRESS` after success, retry link paid twice, refund timeout resent, partner key without `external_merchant_id`, daily limit reached, order below the minimum.
3. Add fixes: server-side creation, stored `online_order_id` and amount, constant-time hash check, status requery before settlement, per-order locking, terminal-state guards, refund attempt stamp, environment-scoped config.
4. Re-read `references/ahapay-api.md` sections relevant to any gap.
5. State remaining uncertainty where the docs are silent: response envelope (`data` wrapper), error body shape, callback retry policy, IP allowlist, partial refunds, and the Daily Limit parameter location.
