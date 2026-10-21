---
name: setup-setel
description: Set up, build, debug, review, and explain Setel payment integrations using official Setel developer docs, Pay with Setel online checkout sessions, payment intents, direct capture or hold-and-capture, webhook signature verification (HMAC-SHA256 keyed with SHA-256 of x-api-secret), session requery, refunds, cancellations, Pay with Setel code scan wallet transactions, SSO, Mesra loyalty, settlement reports (Setel Dashboard, SFTP, manual settlement API), and sandbox/production setup. Use when working with Setel, Pay with Setel, Setel Wallet, Setel CardTerus, Setel Checkout, Setel x-api-key/x-api-secret, Setel webhooks, Setel settlement files (MPF), or production readiness for a Setel merchant integration.
---

# Setel Payment Gateway

Use this skill for Setel (Setel Ventures Sdn Bhd) merchant payment integration work. The main online product is "Pay with Setel online": a checkout session that sends the customer to Setel Checkout or the Setel app, where they pay with Setel Wallet or Setel CardTerus.

## Source

Read `references/setel-api.md` before giving factual API guidance or writing integration code. It holds environments, auth headers, the endpoint catalogue, checkout session and payment intent field tables, statuses, the webhook payload and signature algorithm, refunds, cancellation, capture, error codes and known documentation inconsistencies.

Read `references/setel-other-apis.md` for Pay with Setel code scan (wallet transactions), SSO/OAuth, user accounts, user payment methods, Mesra loyalty, external orders, automated parking and EV charging APIs.

Read `references/setel-settlement.md` for settlement reports (Setel Dashboard, SFTP), the MPF file format, and the manual settlement API.

Read `references/account-setup.md` for onboarding, credentials, dashboard, sandbox vs production, env var names and commercial contact.

Read `references/source-map.md` to pick the right official page and to see docs coverage.

## Core Workflow

1. Confirm the merchant has been onboarded by Setel and holds `x-api-key`, `x-api-secret` and `merchantId` for the right environment.
2. Pick the surface: online checkout session (web/mobile), code scan wallet transaction (in-person), or a partner flow (SSO, loyalty, parking, EV).
3. Create a local pending order with a unique `referenceId` before calling Setel.
4. Decide capture mode: `capture: true` (direct) or `capture: false` (hold, then capture within 30 minutes).
5. Call `POST /v1/checkout/sessions` server-side with `x-api-key` and `x-api-secret`, `amount` (string, MYR), `currency: "MYR"`, `redirect.successUrl` / `redirect.cancelUrl`, and `webhook.url`.
6. Send the customer to the returned checkout URL (PC: Setel Checkout page; mobile: redirect link, checkout URL, or app deep link). Log the real response field names in sandbox.
7. Treat `successUrl` as browser UX only. Do not fulfil from the redirect.
8. On webhook, recompute the `signature` header: HMAC-SHA256 keyed with hex SHA-256 of `x-api-secret`, over the documented field concatenation. Constant-time compare.
9. Confirm with `GET /v1/checkout/sessions/{session id}` (or `GET /v1/payments/payment-intents/{id}`) and settle idempotently only when `paymentIntentStatus` is `succeeded` and amount, currency, referenceId and session id match.
10. If a held payment is final, capture with `POST /v1/payments/payment-intents/{id}/capture` (never above the authorised amount) or cancel it.
11. Requery sessions that have no webhook after a timeout; Setel says webhooks may be lost or sent more than once.

## Core Facts

- Sandbox base URL: `http://api-pre-prod.setel.com` (System environment page, last updated 18 Nov 2024). Production: `https://api.setel.com`.
- Partner auth: headers `x-api-key` and `x-api-secret` on every request; `Content-Type: application/json`. Keys are backend-only.
- Create session: `POST /v1/checkout/sessions`. Required body fields: `amount`, `currency`, `redirect`.
- Retrieve session: `GET /v1/checkout/sessions/{id}`. Cancel: `PATCH /v1/checkout/sessions/{id}/cancel`.
- Payment intents: `GET /v1/payments/payment-intents/{id}`, `POST .../{id}/capture`, `POST .../{id}/cancel`, `POST .../{id}/refund`.
- `amount` is a string in ringgit (examples use `"10"`); only `MYR` is supported.
- `capture` defaults to `true`. Held (authorised) payments expire after 30 minutes.
- Payment intent statuses: `requires_payment_method`, `requires_action` (not supported yet), `authorised`, `processing`, `failed`, `succeeded`, `cancelled`, `expired`, `partially_refunded`, `refunded`.
- Webhook is set per session via `webhook.url`; the header is `signature`; the body has `id`, `createdAt`, `updatedAt`, `apiKey`, `paymentIntentId`, `paymentIntentStatus`, `amount`, `referenceId`.
- Signature: `secretHash = sha256_hex(x-api-secret)`; `data = id+createdAt+updatedAt+apiKey+paymentIntentId+paymentIntentStatus+amount+referenceId` (missing field = empty string); `signature = hmac_sha256_hex(key=secretHash, data)`.
- Online payment methods: Setel Wallet and Setel CardTerus. Code scan supports Setel Wallet only and has no webhook.
- Duplicate `referenceId` returns `400 REFERENCE_ID_EXISTED` (`9300101`).
- Refunds draw from the merchant available balance (not pending balance); partial refunds are allowed until `refundedAmount` equals `amount`.
- Settlement reports are generated daily at 01:00 GMT+8 and cover 00:00 to 23:59 of the previous day.

## Critical Rules

- Keep `x-api-key` and `x-api-secret` server-side; never ship them to a browser or mobile bundle, never commit them.
- Always send a unique `referenceId` per checkout attempt; Setel warns this prevents multiple payments and enables reconciliation.
- Verify the webhook `signature` before trusting the body, and still requery the session or payment intent before fulfilment.
- Build the signature string from the exact received string values; do not reformat timestamps or amounts.
- Handle out-of-order and duplicate webhooks: compare `updatedAt` and keep the latest; make settlement idempotent on session id and `paymentIntentId`.
- Do not treat `processing` or `authorised` as paid. Only `succeeded` (or later refund states) means funds were collected.
- For `capture: false`, capture or cancel within 30 minutes, or the intent becomes `expired`.
- Third-party acquirers must send `subMerchant` (`name`, `reference`, `merchantCategoryCode`, `state`).
- Treat field-name and method inconsistencies listed in `references/setel-api.md` as items to confirm in sandbox, not as settled facts.
- Do not quote fees; Setel does not publish merchant rates.

## Completion Criteria

A Setel integration is done only when:

1. Checkout sessions are created server-side with both auth headers and a unique `referenceId`.
2. The redirect page only shows status; fulfilment comes from a verified webhook plus a session/payment intent requery.
3. Webhook handler recomputes `signature` with the documented algorithm and passes the published test vector.
4. Settlement checks session id, `paymentIntentId`, `referenceId`, amount, currency `MYR`, status `succeeded`, and environment.
5. Duplicate and out-of-order webhooks, lost webhooks (requery job), and expired holds are handled idempotently.
6. Refund, cancel and capture paths are guarded against double execution and over-amount.
7. Tests cover bad signature, duplicate webhook, wrong amount/reference, `failed`/`expired`/`cancelled`, and `succeeded`.
8. Deployment docs list `SETEL_API_KEY`, `SETEL_API_SECRET`, `SETEL_MERCHANT_ID`, `SETEL_BASE_URL`, success/cancel URLs and webhook URL per environment.

## Loophole Review Loop

When asked whether a Setel strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, Setel Checkout page, Setel app deep link, backend session creator, Setel API, webhook receiver, database, fulfilment job, capture/cancel/refund jobs, settlement report import.
2. Check bypasses: fake success redirect, forged webhook, signature built from re-serialised JSON, wrong secret or environment, reused `referenceId`, amount mismatch, `processing` treated as paid, duplicate or stale webhook overriding a newer state, lost webhook, hold expiring before capture, capture above authorised amount, refund failing on low available balance, duplicate refund.
3. Add fixes: server-side session creation, local pending order first, documented HMAC check with constant-time compare, requery before settle, `updatedAt` ordering, idempotency keys, capture deadline job, refund guard, environment-separated config.
4. Re-read the relevant section of `references/setel-api.md`.
5. State remaining uncertainty: exact checkout URL field names, retrieve-by-referenceId method, webhook retry schedule, webhook source IPs, rate limits, sandbox test accounts, sandbox scheme (http vs https), and commercial terms. These are not documented and must be confirmed with Setel.
