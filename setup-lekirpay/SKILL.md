---
name: setup-lekirpay
description: Set up, build, debug, review, and explain LekirPay payment integrations using official LekirPay docs, OAuth2 client_credentials access tokens, Send Payment and Secure Send Payment APIs, Lekir-Signature HMAC-SHA256 checksum, payment_url hosted checkout, callback_url notifications, redirect_url and cancel_url returns, Payment Status requery, Payment Groups, sandbox/live hosts, Client ID, Lekirkey, and the official GiveWP plugin. Use when working with LekirPay Malaysia, Lekir Tech, FPX, credit card checkout, payment links, payment forms, donation forms, callback settlement, or production readiness.
---

# LekirPay Payment Gateway

Use this skill for LekirPay (Lekir Tech (M) Sdn Bhd) payment gateway integration work.

## Source

Read `references/lekirpay-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, base URLs, token flow, full request/response field tables, the Lekir-Signature checksum algorithm with a verified worked example, callback fields, status values, and documented gaps.

Read `references/account-setup.md` when the user asks how to register, where the Client ID, Lekirkey and Lekir-Signature key live, sandbox vs live, env var names, payment methods, refunds, settlement, or pricing.

Read `references/source-map.md` to pick the right official page, and to see which doc pages are hidden or legacy.

## Core Workflow

1. Identify environment: sandbox (`https://sandbox.lekirpay.com`) or live (`https://app.lekirpay.com`). Confirm the sandbox host responds before relying on it.
2. Confirm the merchant profile is verified; the payment APIs refuse unverified profiles.
3. Get credentials from the dashboard API tab: ID (`client_id`), Lekirkey (`client_secret`), Lekir-Signature key.
4. Create a local pending payment with a unique `reference_no` (max 140 characters) and the exact amount string.
5. Request a Bearer token: `POST /oauth/token` with `grant_type=client_credentials`, `client_id`, `client_secret`. Cache it server-side (documented `expires_in` 1296000 s, 15 days).
6. Create the payment server-side with `POST /api/payments/payment/secure` (preferred, sends `checksum`) or `POST /api/payments/payment`.
7. Store `payment_id`, then redirect the customer to `payment_url`.
8. On `callback_url` POST (and on `redirect_url` return), treat the payload as untrusted: it carries no documented signature.
9. Requery `GET /api/payments/{payment_id}/status` with the Bearer token; settle only when status is `Paid` and `payment_id`, `reference_no`, and `amount` match the local record.
10. Settle idempotently; callback, redirect and manual requery may all arrive for the same payment.

## Core Facts

- Live API base: `https://app.lekirpay.com/api`. Sandbox API base (docs): `https://sandbox.lekirpay.com/api`.
- Token endpoint (live): `POST https://app.lekirpay.com/oauth/token`; response `token_type` `Bearer`, `expires_in`, `access_token`.
- Create payment: `POST /api/payments/payment`; signed variant: `POST /api/payments/payment/secure`.
- Required create fields: `item`, `amount`, `reference_no`, `callback_url`, `email`, `name` (plus `checksum` on `/secure`).
- Create response: `payment_id`, `reference_no`, `payment_url`, `status` (`created`), `created_date`.
- Checksum: HMAC-SHA256 (hex) with the Lekir-Signature key over `amount{amount}|email{email}|item{item}|name{name}|callback_url{callback_url}|reference_no{reference_no}` in that fixed order.
- Callback: LekirPay POSTs `payment_id`, `reference_no`, `trx_no`, `merchant_name`, `amount`, `name`, `email`, `phone`, `item`, `timestamp`, `payment_method`, `status`, `type` (`callback` or `redirect`) on success or failure.
- Status requery: `GET /api/payments/{payment_id}/status`; statuses `Created`, `Proceed`, `Paid`, `Failed`.
- `method` (`GET` or `POST`, default `POST`) controls how data is sent to the redirect.
- Amount is a ringgit decimal (`double`), not sen. Website states MYR only.
- API accepts `application/x-www-form-urlencoded` and `application/json`; documented HTTP codes are 200 and 400.
- Payment Groups: `POST /api/payment_group/create`, `GET /api/payment_group/data`; pass `group_id` on create payment.
- No refund, recurring, tokenisation, or webhook-signature API is documented.

## Critical Rules

- Keep `client_secret` (Lekirkey), the Lekir-Signature key and access tokens server-side only.
- Never mark paid from the callback body, the `redirect_url` hit, or `status` in a POST alone; always requery the status API.
- Compare status case-insensitively (`created` in create responses, `Paid` in the official plugin check).
- Build the checksum string from the exact values you send (same amount formatting, same `callback_url` with scheme); do not sort keys alphabetically even though the docs prose says "ascending".
- Make `reference_no` unique per attempt and use it to map callbacks back to local orders.
- Return HTTP 200 quickly from the callback; do the requery and settlement idempotently.
- Do not call the hidden bulk payout or transfer pages (`lekirpay.co`); they are legacy and the host does not resolve.
- Do not copy the GiveWP plugin's secure-payment code: it never calls `/secure` and uses an undefined key variable.
- Verify current sandbox availability, plan (cards need Enterprise), and settlement terms with LekirPay before claiming production certainty.

## Completion Criteria

A LekirPay integration is done only when:

1. Token request and payment creation run server-side; the token is cached and refreshed before expiry.
2. Local pending payment with unique `reference_no` and expected amount exists before redirecting to `payment_url`.
3. `checksum` (if using `/secure`) is computed over the documented fixed-order string and unit-tested against the doc example.
4. Callback and redirect handlers requery `GET /api/payments/{payment_id}/status` and check `payment_id`, `reference_no`, amount and `Paid` status.
5. Settlement is idempotent across callback, redirect, and retry.
6. Tests cover forged callback (status Paid but API says otherwise), wrong amount, wrong reference, `Failed` status, duplicate callback, expired token.
7. Deployment docs list `LEKIRPAY_BASE_URL`, `LEKIRPAY_CLIENT_ID`, `LEKIRPAY_CLIENT_SECRET`, `LEKIRPAY_SIGNATURE_KEY`, callback URL, redirect URL, and sandbox/live separation.

## Loophole Review Loop

When asked whether a LekirPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, `payment_url` page, backend creator, token cache, LekirPay API, callback receiver, status requery, database, fulfilment job, dashboard settings.
2. Check bypasses: forged callback POST (no signature), fake redirect POST with `status=Paid`, `payment_id` swapped to another merchant order, amount tampering, reused `reference_no`, stale token, sandbox/live mix, case-sensitive status compare, duplicate callback, callback for `Failed` after `Paid`, unverified profile, card payment on a plan without cards.
3. Add fixes: server-side creation, local pending record, status requery as the only truth, strict `payment_id`/`reference_no`/amount match, unique constraint on `payment_id`, durable event log, token refresh, separate env config.
4. Re-read `references/lekirpay-api.md` sections relevant to any gap.
5. State remaining uncertainty: callback retry policy, expected callback response, error body shape, sandbox host status, refund process, and settlement schedule are not documented.
