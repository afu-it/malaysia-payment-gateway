---
name: setup-kineticpay
description: Set up, build, debug, review, and explain KineticPay payment integrations using official KineticPay docs and KineticPay's official WordPress plugins, FPX bill creation via /payment/create, auto-submit HTML checkout, callback_success/callback_error/callback_status URLs, server-side status requery via /payment/status, merchant key from the Collection page, API v2 bearer key, FPX bank codes, sandbox mode, rate limits, and error shapes. Use when working with KineticPay Malaysia, FPX online banking, collection forms, WooCommerce, GiveWP, Gravity Forms, Forminator, custom PHP/Node/Python checkout, callback settlement, or production readiness.
---

# KineticPay Payment Gateway

Use this skill for KineticPay (kineticpay.my) payment gateway integration work.

## Source

Read `references/kineticpay-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, the bill creation fields, the callback and requery flow, status codes, FPX bank codes, rate limits, errors, and the gaps in the official docs.

Read `references/account-setup.md` when the user asks how to register, where the merchant key or API key lives, how sandbox mode works, which env vars to use, payment method availability, settlement, or pricing.

Read `references/source-map.md` when choosing which official page or plugin file backs a fact, or to see what the public docs do not cover.

## Core Workflow

1. Identify environment: live or sandbox (sandbox is a toggle on the Collection page, same endpoints).
2. Identify surface: custom server integration, WooCommerce, GiveWP, Gravity Forms, Forminator, WHMCS, or a no-code collection form/link.
3. Confirm dashboard setup: approved company, a Collection with name and website URL, its merchant key, and (for API v2) the API key.
4. Create a local pending payment with a unique `invoice` string before calling KineticPay.
5. Let the buyer pick an FPX bank; send its code as `bank`.
6. `POST https://manage.kineticpay.my/payment/create` server-side with `merchant_key`, `invoice`, `amount`, `description`, `bank`, `callback_success`, `callback_error`, `callback_status`.
7. Output the returned `html` (an auto-submit form) to the browser to send the buyer to the bank. On `error`, show the messages.
8. On any callback hit, do not trust the request body. Requery `GET /payment/status?merchant_key=...&invoice=...` server-side.
9. Settle idempotently only when the requery returns `code` `"00"` and the invoice, amount, and local state match.
10. Keep a requery job for pending payments whose callback never arrived.

## Core Facts

- Payment base (used by all official plugins): `https://manage.kineticpay.my`.
- Create bill: `POST /payment/create`. Requery: `GET /payment/status`.
- Bill auth is the Collection `merchant_key` sent in the body (create) or query string (status). No header auth, no signature.
- API v2 base: `https://manage.kineticpay.my/api/v2`, auth `Authorization: Bearer <API_KEY>`, headers `Accept` and `Content-Type: application/json`.
- API v2 documented endpoints: `GET /ping`, `GET /bank`, `GET /company`. Bills, Collections, Collection Forms, Transactions, Settlements and Points are listed in the docs nav but have no published page (checked 2026-10-21).
- Successful payment code is `"00"` (`code` on requery, `record.transaction_status_code` on the WooCommerce callback POST).
- `amount` is in ringgit with 2 decimals (for example `"12.50"`), not sen. MYR only.
- FPX bank codes such as `MB2U0227`, `BCBB0235`, `PBB0233`; fetch the live list from `GET /api/v2/bank`.
- Rate limit: 60 requests per minute per user; `X-RateLimit-Limit`, `X-RateLimit-Remaining` headers; HTTP 429 when exceeded.
- Validation errors return JSON `{"error": {"field": ["message"]}}` (observed with HTTP 200 on `/payment/status`).
- Pagination on v2 list endpoints: `page`, `perPage` (default 20, max 40), metadata in `meta.pagination`.
- Merchant key: Collection page, generated after you enter Collection name and website URL. API key: `https://manage.kineticpay.my/api-key`.

## Critical Rules

- Keep `merchant_key` and the API key server-side only. Never put them in browser code.
- KineticPay documents no callback signature or checksum. Never mark paid from a callback body, redirect, or query string alone; always requery `/payment/status` server-side.
- The requery puts `merchant_key` in a URL query string. Do not log full requery URLs; redact them in HTTP client logs.
- Make `invoice` unique per payment attempt and store it before redirecting. Official plugins use a site prefix plus order id plus a suffix (for example `MYSHO123KNWC`).
- Compare requery `code`, invoice, and amount against the local record. The status response schema is not documented; log the full response and confirm which amount field it returns.
- Settle idempotently: the same invoice can hit `callback_success`, `callback_status`, and a requery job.
- Send `amount` as a 2-decimal ringgit string. Do not send sen integers.
- Treat the WooCommerce plugin's 3-second cron and the plugin defects listed in the API reference as patterns to avoid, not copy.
- Disable sandbox mode on the Collection before going live, and test once in live with a small amount.
- Card, e-wallet and BNPL are not exposed in any published API field; the public bill flow is FPX with a `bank` code.

## Completion Criteria

A KineticPay integration is done only when:

1. Bill creation runs server-side with `merchant_key` from env, and the `html` response is rendered only after a local pending record exists.
2. The checkout collects a valid FPX `bank` code (from `GET /api/v2/bank` or a vetted list).
3. All three callback URLs point to a server route that requeries `/payment/status` and never trusts the request body.
4. Settlement checks `code === "00"`, invoice, amount, MYR, and local status, inside an idempotent transaction.
5. A requery job reconciles pending payments within the rate limit (60/min).
6. Tests cover: forged callback body with `"00"`, unknown invoice, amount mismatch, duplicate callbacks, failed code, `error` response on create, 429 handling.
7. Deployment docs list `KINETICPAY_MERCHANT_KEY`, `KINETICPAY_API_KEY` (if v2 is used), callback base URL, and the sandbox toggle state.

## Loophole Review Loop

When asked whether a KineticPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser, auto-submit form, FPX bank page, callback routes, backend bill creator, KineticPay `/payment/*`, API v2, database, fulfilment job, requery job, dashboard Collection settings.
2. Check bypasses: forged callback POST with `transaction_status_code: "00"`, replayed callback, guessed invoice, invoice reuse across attempts, amount tampering before create, sandbox left on in production, merchant key leaked in logs or frontend, stale pending never requeried, rate limit exhaustion from aggressive polling, wrong Collection key for the site.
3. Add fixes: server-side requery as the only source of truth, unique unguessable invoice, amount check against local record, idempotent settle, redacted logs, backoff on 429, sandbox check at deploy.
4. Re-read `references/kineticpay-api.md` sections relevant to any gap.
5. State remaining uncertainty: undocumented status response fields, undocumented callback payload, no refund or recurring API, card plan access, and whether `callback_status` is called server-to-server. Tell the user to confirm these with KineticPay support (support@kineticpay.my).
