---
name: setup-securepay
description: Set up, build, debug, review, and explain SecurePay (Malaysia) payment integrations using official SecurePay docs, API v1 FPX payments, Payment Session, API v2 payment intents, HMAC-SHA256 checksums, callback and redirect URLs, status requery, transaction list and search, FPX B2C/B2B1 bank lists, DuitNow, eMandate direct debit, split settlement, sandbox/live setup, API UID, Auth Token, Checksum Token, client ID/secret, and the official WooCommerce plugin. Use when working with SecurePay, securepay.my, FPX online banking, DuitNow, direct debit mandates, callback checksum verification, payment status checks, settlement timing, or production readiness.
---

# SecurePay Payment Gateway

Use this skill for SecurePay Sdn Bhd (securepay.my) payment gateway integration work.

## Source

Read `references/securepay-api.md` before giving factual API guidance or writing integration code. It is the entry point: version choice, environments, credentials, the full endpoint catalogue, v1 payment creation, the request checksum, callback/redirect fields and checksum verification (with test vectors), status, list, search, errors and gaps.

Read `references/securepay-v2.md` when the merchant uses API v2 (SecurePay Console, client ID/secret, `auth_token`, payment intents, DuitNow, eMandate, split settlement, closed wallet, memberships, partner API, certificates).

Read `references/securepay-codes.md` for FPX bank codes, status values, auth codes, enums and sandbox test banks.

Read `references/account-setup.md` for registration, documents, where credentials live, sandbox vs live, interface settings, settlement schedule, refunds policy, plugins and support.

Read `references/source-map.md` to pick the right official page, and for docs coverage notes.

## Core Workflow

1. Pick the API version. SecurePay's docs hub marks v1 as the production API and v2 as a preview. Default to v1 unless the merchant's account is on SecurePay Console (v2 client ID/secret).
2. Identify environment: sandbox or production. v1 sandbox `https://sandbox.securepay.my/api`, production `https://securepay.my/api`.
3. Get the interface credentials for that environment: API UID, Auth Token, Checksum Token (v1).
4. Create a local pending order with a unique `order_number` and the expected amount before calling SecurePay.
5. v1: build the request checksum, then POST the form fields to `/v1/payments` (usually as an auto-submitted browser form, like the official plugin), or create a Payment Session and redirect to `payment_url`.
6. Treat `redirect_url` as browser UX only. Fulfil from the server-to-server `callback_url` POST.
7. Verify the callback `checksum`: sort all received keys ascending (excluding `checksum`), join values with `|`, HMAC-SHA256 with the Checksum Token, constant-time compare.
8. Requery `GET /v1/payments/status/:order_number` with Basic auth before settling, and confirm `payment_status`, `transaction_amount_received`, currency and `order_number`.
9. Settle idempotently. For FPX B2B1 treat `fpx_debit_auth_code` `99` as pending; wait for the later `00` callback.
10. Reconcile with `GET /v1/transactions/list` or `/v1/transactions/search` when callbacks are missed.

## Core Facts

- v1 create payment: `POST /v1/payments` (form-encoded). v1 Payment Session (beta): `POST /apis/payments` returning JSON `payment_url`.
- v1 request checksum string order: `buyer_email|buyer_name|buyer_phone|callback_url|order_number|product_description|redirect_url|transaction_amount|uid`, HMAC-SHA256 hex, key = Checksum Token. Empty optional values keep an empty slot.
- v1 amounts are ringgit with two decimals (`1540.40`). v2 payment intents use `amount` in cents (`154040`).
- v1 auth for status/list/search/validate/session: HTTP Basic `uid:auth_token`. The payment form POST carries `token` (Auth Token) and `checksum` as fields.
- Validate credentials: `POST /v1/merchants/validate` with Basic auth and `checksum_token`.
- Public FPX bank list (no auth): `GET /public/v1/banks/b2c` and `/public/v1/banks/b2b`, optional `?status=online|offline`. Discard a response whose `VerifySign` is not `true`.
- Callback/redirect are POSTed with fields such as `payment_status`, `transaction_amount`, `transaction_amount_received`, `order_number`, `exchange_number`, `fpx_debit_auth_code`, `checksum`.
- v2 base URLs: production `https://console.securepay.my/api`, staging `https://sandbox.securepay.dev/api`. Auth: `POST /v1/auth` with Basic `client_id:client_secret`, then `Authorization: Bearer <auth_token>`.
- No refund, void or capture API is documented. Refunds are manual via support (Refund & Cancellation Policy).
- Settlement is T+1 business day per SecurePay; cut-off 11:59pm MYT; Friday to Sunday settle Monday.
- Rate limits and callback source IPs are not documented.

## Critical Rules

- Keep Auth Token, Checksum Token, client secret and partner secrets server-side. The browser form may carry `token` and `checksum` (as the official plugin does) but never the Checksum Token itself.
- Compute the request checksum server-side; never let the browser supply amount or checksum inputs.
- Do not mark paid from the redirect POST, the SecurePay status page, or `payment_status` alone. The official WooCommerce plugin does not verify the callback checksum; do not copy that.
- Verify the callback checksum over the exact received parameters (all keys, including ones you do not use). SecurePay says params keep being added, so never hard-code the key list.
- Strip zero-width characters if you copy strings from the docs; several doc samples contain U+200B.
- Always requery status server-side and check amount, currency (`MYR`), `order_number` and environment before settling.
- Keep `order_number` unique per attempt; status lookups use it.
- Use sandbox credentials only with sandbox URLs, and production with production.
- v2 callbacks carry a nested `payment` object and the docs do not define how to flatten it for the checksum; settle v2 from an authenticated status requery.
- Confirm live payment method availability, pricing, settlement arrangement and B2B1 enablement with SecurePay before claiming production certainty.

## Completion Criteria

A SecurePay integration is done only when:

1. Payment creation runs from server-generated fields with a correct HMAC-SHA256 checksum, or via Payment Session / v2 intent created server-side.
2. Redirect handling only shows status; it does not fulfil.
3. Callback handler verifies `checksum` with the Checksum Token over all received params, constant-time.
4. Settlement requeries status and checks `order_number`, `payment_status`, `transaction_amount_received`, currency and environment.
5. Duplicate callbacks, redirect-before-callback, B2B1 `99` then `00`, and failed payments are idempotent.
6. Tests cover bad checksum, tampered amount, unknown order, failed payment, duplicate callback, and the documented checksum test vectors.
7. Deployment docs list `SECUREPAY_UID`, `SECUREPAY_AUTH_TOKEN`, `SECUREPAY_CHECKSUM_TOKEN` (or v2 `SECUREPAY_CLIENT_ID`, `SECUREPAY_CLIENT_SECRET`), base URL, callback URL, redirect URL and sandbox/live separation.

## Loophole Review Loop

When asked whether a SecurePay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser form POST, SecurePay bank selection page, redirect POST, callback POST, checksum verifier, status requery, database, fulfilment job, dashboard interface settings, settlement account.
2. Check bypasses: forged redirect POST, forged callback without checksum, checksum over a hard-coded key subset, wrong Checksum Token (other interface), sandbox/live mix, tampered `transaction_amount`, reused `order_number`, B2B1 `99` treated as paid, duplicate callback, callback before order commit, offline bank selected, USD Payment Session amount converted to MYR, nested v2 payload.
3. Add fixes: server-side checksum, local pending order first, full-param checksum verification, constant-time compare, status requery, strict amount/currency/order checks, unique order numbers, idempotent settle, per-environment config, reconciliation via list/search.
4. Re-read `references/securepay-api.md` (and `securepay-v2.md` for v2) for any gap.
5. State remaining uncertainty: undocumented v1 `/v1/payments` response format, `uid` as a form field, v2 response bodies, v2 checksum for nested payloads, PQC request signing, rate limits, callback IPs, retry policy, pricing.
