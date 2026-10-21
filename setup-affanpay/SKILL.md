---
name: setup-affanpay
description: Set up, build, debug, review, and explain AffanPay payment integrations using official AffanPay API docs, Bearer API tokens, Create Bill, hosted bill payment page redirect, callback_url webhooks, X-AFFANPAY-HMAC HMAC-SHA256 signatures with X-AFFANPAY-TIMESTAMP, Requery by bill_id or payment_ref, sandbox vs live hosts, status codes, and idempotent settlement. Use when working with AffanPay Malaysia, FPX, DuitNow Online Banking, DuitNow QR, e-wallets, cards, bills, payment links, callback verification, requery fallback, or production readiness.
---

# AffanPay Payment Gateway

Use this skill for AffanPay payment gateway integration work.

## Source

Read `references/affanpay-api.md` before giving factual API guidance or writing integration code. It contains the endpoint catalogue, base URLs, token auth, Create Bill fields, Requery, callback body, the HMAC signature algorithm, status codes, and what the docs do not cover.

Read `references/account-setup.md` when the user asks how to register, which documents are needed, where the API token and secret key live, sandbox vs live accounts, env var names, payment method approval, settlement, or pricing.

Read `references/source-map.md` to see which official page backs each fact and what was checked but not found.

## Core Workflow

1. Identify environment: sandbox (`https://sandbox.affanpay.my`) or live (`https://app.affanpay.my`).
2. Confirm dashboard setup: API token and secret key from Settings > Developer in the same environment, optional bill Category ID, and approved payment channels.
3. Create a local pending order with a unique `external_ref` and the expected amount before calling AffanPay.
4. Create the bill server-side: `POST {base}/api/v1/bill` with `Authorization: Bearer <API_TOKEN>`, `Accept: application/json`, `Content-Type: application/json`.
5. Store the returned bill `id` against the order, then redirect the customer to the returned `url`.
6. Treat `redirect_url` as browser UX only; do not fulfil from the redirect.
7. Receive the POST to `callback_url`. Verify `X-AFFANPAY-HMAC` = lowercase hex HMAC-SHA256 of `timestamp + "." + rawBody` keyed with the secret key, using constant-time comparison, and reject timestamps older than 300 seconds.
8. Decide from integer `status_code` (1 Success, 3 Failed), never from `status`. Match `bill.id`, `bill.external_ref`, and amount before marking paid.
9. Settle idempotently keyed on `payment_ref`; every callback is sent twice (about 5 and 10 seconds apart).
10. Use `GET {base}/api/v1/requery?bill_id=...` (or `payment_ref=...`) as fallback when a callback is missing.

## Core Facts

- Live base: `https://app.affanpay.my/api`. Sandbox base: `https://sandbox.affanpay.my/api` (same docs, separate host and account).
- Endpoints: `POST /v1/bill` (create bill), `GET /v1/requery` (status), and the outbound callback POST to your `callback_url`. No other endpoints are documented.
- Auth: `Authorization: Bearer <token>`; token is generated in Settings > Developer > API tokens and is shown only once.
- All API responses, including errors, are JSON. Unauthenticated calls return HTTP 401 `{"message":"Unauthenticated."}`.
- Create Bill required fields: `name`, `amount`, `external_ref`, `customer_name`, `customer_email`. Amount is decimal ringgit, up to 2 decimals, minimum `5.00`.
- Create Bill response: `{"success": true, "id": "<bill_id>", "url": "https://app.affanpay.my/<bill_id>"}`.
- Requery takes exactly one of `bill_id` or `payment_ref` and returns `data.payments[]` with `payment_ref`, `status_code`, `status`, `nett_amount`, `payment_channel`.
- Status codes: `1` Success, `2` Pending, `3` Failed, `4` No Action. Callbacks fire only for 1 and 3.
- Callback headers: `Content-Type: application/json`, `X-AFFANPAY-TIMESTAMP` (Unix seconds), `X-AFFANPAY-HMAC`.
- Signature: `hex(HMAC_SHA256(secret_key, timestamp + "." + raw_body))`, lowercase. The secret key is in Settings > Developer and is different from the API token.
- Callback must return HTTP 200 only after processing; any status 400 or above is recorded as a failed delivery.
- `paid_amount` can exceed `bill.amount` when `fee_charge_payer` passes the service charge to the payer.
- Refund, cancel, recurring, tokenisation, payout, and settlement report APIs are not documented (checked 2026-10-21).

## Critical Rules

- Keep the API token and secret key server-side only; never ship them to the browser or mobile app.
- Verify the HMAC over the raw request body before parsing JSON. Re-encoded JSON will not match.
- Signature verification is optional on AffanPay's side but treat it as mandatory in your handler; the callback URL is public.
- Use constant-time comparison and a 300-second timestamp window to block replays.
- Use the token, secret key, and base URL of the same environment; sandbox and live are separate hosts.
- Decide on `status_code` cast to integer. Never branch on the `status` label.
- Cast `paid_amount`, `bill.amount`, and requery `nett_amount` (which can carry a leading space) to decimals before comparing; never compare as strings.
- Compare against `bill.amount` for the order value; `paid_amount` may include the payer-borne fee.
- Make settlement idempotent on `payment_ref`; callbacks always arrive twice and one bill can have several payment attempts.
- Return non-2xx when processing fails so the failure is flagged; do not return 200 from a crashed handler.
- Do not invent refund or cancel endpoints. Refunds are handled outside the documented API; confirm with AffanPay support.

## Completion Criteria

An AffanPay integration is done only when:

1. Bill creation runs server-side with the Bearer token and a unique `external_ref` per order.
2. The returned bill `id` is stored before redirecting to `url`.
3. `redirect_url` handling only shows status; it never fulfils.
4. The callback handler verifies `X-AFFANPAY-HMAC` over `timestamp.rawBody` with constant-time comparison and a timestamp window.
5. Settlement checks `status_code === 1`, `bill.id`, `bill.external_ref`, and decimal amount, keyed idempotently on `payment_ref`.
6. A requery job resolves orders whose callback never arrived, and handles bills with multiple payment attempts.
7. Tests cover invalid signature, stale timestamp, duplicate callback, wrong amount or reference, failed status, and success.
8. Deployment docs list `AFFANPAY_BASE_URL`, `AFFANPAY_API_TOKEN`, `AFFANPAY_SECRET_KEY`, redirect URL, callback URL, and sandbox/live separation.

## Loophole Review Loop

When asked whether an AffanPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, hosted bill page, backend bill creator, AffanPay API, callback receiver, raw-body parser, database, fulfilment job, requery job, and dashboard settings.
2. Check bypasses: fake success redirect, forged callback, HMAC over parsed JSON, wrong secret, API token used as HMAC key, sandbox/live mix, replayed old callback, wrong amount, fee-inflated `paid_amount`, wrong `external_ref`, string-compared amounts, duplicate callback, handler returning 200 after failure, missing callback, and a bill with several attempts where an earlier one failed.
3. Add fixes: server-side bill creation, local pending order first, raw-body HMAC verification, constant-time compare, timestamp window, strict reference and decimal amount checks, durable callback log, idempotency on `payment_ref`, separate environment config, and requery fallback.
4. Re-read `references/affanpay-api.md` sections relevant to any gap.
5. State remaining uncertainty for undocumented areas: redirect query parameters, bill expiry behaviour, error body shapes, rate limits, IP allowlists, refunds, sandbox test data, and channel approval.
