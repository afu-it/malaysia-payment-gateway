---
name: setup-payright
description: Set up, build, debug, review, and explain Payright (Malaysia, payright.my) payment integrations using official Payright docs, the Collection and Bill APIs, FPX bank (payment provider) selection, hosted payment_url redirect, JWT (HS256) encoded callbacks decoded with the Hash Key, Get Bill status requery, optional X-Signature JWT request signing, auth-token header auth, sandbox/live setup, Payout API, payout providers, payout callbacks, and Get Balance. Use when working with Payright Sdn Bhd, payright.my, FPX B2C/B2B online banking, Payright bills, collections, callbacks, payouts, virtual pool balance, or production readiness. Not for Payright Australia (payright.com.au BNPL).
---

# Payright Payment Gateway

Use this skill for Payright (Payright Sdn Bhd, Kuala Lumpur) payment gateway integration work.

## Source

Read `references/payright-api.md` before giving factual API guidance or writing integration code. It contains the endpoint catalogue, base URLs, auth header, Collection and Bill request/response fields, the JWT callback format and decode rules, Get Bill requery, request signing (X-Signature), payout and balance APIs, error codes, payment provider IDs, and documented gaps.

Read `references/account-setup.md` when the user asks how to register, where the Auth Token and Hash Key live, sandbox vs live, env var names, JWT verification toggle, payment methods, settlement, or pricing.

Read `references/source-map.md` when choosing which official Payright page applies, or to see which areas are undocumented and which legacy (2022) plugin sources must not be copied.

## Core Workflow

1. Identify environment: sandbox (`https://api.payright-sandbox.my/api/v1`) or production (`https://api.payright.my/api/v1`).
2. Identify surface: Bill API (FPX checkout), Payout API (disbursement), or no-code (Payment Forms, Bills in the portal, plugins).
3. Confirm dashboard setup: Auth Token, Hash Key, JWT Verification on/off, a Collection (`collection_uuid`), and active payment services.
4. Create a local pending payment with your own unique `invoice_ref` before calling Payright.
5. Fetch providers with `POST /merchant/list-payment-provider` and let the payer choose a bank; `provider_id` is required on the bill.
6. Create the bill server-side with `POST /merchant/bills`; store `invoice_no`, `invoice_ref`, amount; redirect the payer to `payment_url`.
7. Treat `redirect_url` as browser UX only; nothing in the docs makes it a trusted payment result.
8. On callback, read the JSON body `{"data": "<JWT>", "response_code": ...}` and verify the JWT with HS256 and the Hash Key. Reject on any verification failure.
9. Settle idempotently after `invoice_ref`, amount, and `invoice_status == "SUCCESS"` match; then confirm with `GET /merchant/bills/{invoice_ref}`.
10. Use Get Bill as the fallback for missing callbacks; the docs only promise callbacks after successful transactions.

## Core Facts

- Production API base: `https://api.payright.my/api/v1`. Sandbox: `https://api.payright-sandbox.my/api/v1`.
- Auth header: `auth-token: <AUTH_TOKEN>` (docs prose says "Authorization header with value auth-token {token}", but every example and the official Postman collection send a header named `auth-token`).
- Auth Token: Accounts > API Keys > View API Keys > Auth Token. Hash Key: Account > API Keys > View API Key > Hash Key.
- Create Bill: `POST /merchant/bills`, required `collection_uuid`, `amount` (MYR, decimal 10,2), `provider_id`, `callback_url`, `redirect_url`, `full_name`, `email`, `phone_number`; `invoice_ref` is in the example (your reference, error 425 "Invoice already exists" on reuse).
- Create Bill returns `invoice_no` (e.g. `Py...rgt`), `invoice_ref`, `payment_url`.
- Get Bill: `GET /merchant/bills/{invoice_no}` or `GET /merchant/bills/{invoice_ref}`; returns `transaction_details.status` (e.g. `PENDING`) and `fpx_invoice_no`.
- Callback body is JSON with `data` (HS256 JWT signed with the Hash Key) and `response_code` (sample `2100`). Decoded fields include `invoice_no`, `invoice_ref`, `invoice_status` (`SUCCESS`), `amount` (string `"240.00"`).
- The callback `invoice_no` is the FPX invoice number (`FPX...`), which matches Get Bill `fpx_invoice_no`, not the `Py...` bill `invoice_no`. Match on `invoice_ref`.
- Request signing is optional: when JWT Verification is enabled (Account > API Keys > Edit), send `x-signature`, an HS256 JWT of `{auth-token, http_method, url_path, iat, exp}` signed with the Hash Key.
- Amounts are ringgit with two decimals, not sen. MYR only.
- Response envelope: `status` (`ok`/`fail`), `code`, `data`, `uuid`, `message`, `error`. Business errors can arrive with HTTP 200; always check `status` and `code`.
- Payout: `POST /merchant/payout`, `GET /merchant/payout/{invoice_no|external_invoice_ref}`, providers via `POST /merchant/list-payout-providers`, balance via `GET /merchant/balance`.
- Duplicate payout guard: error 10149 if amount, account_no and recipient_reference repeat within 3 hours.
- Sandbox failed bills do not count toward dashboard or settlement totals.

## Critical Rules

- Keep Auth Token and Hash Key server-side only. Never ship them to the browser or a mobile app.
- Never mark paid from `redirect_url` or any browser query string.
- Verify the callback JWT signature (HS256, Hash Key) and pin the algorithm to HS256; never accept `alg: none` or decode without verifying.
- Match on `invoice_ref` (and amount), not on the callback `invoice_no`, because it is the FPX number.
- Compare amounts as decimal strings or integer sen; the callback sends `"240.00"` as a string.
- Make callback handling idempotent; a late or repeated callback must not double-fulfil.
- After a verified callback, requery Get Bill when possible and settle from server-side state.
- Generate a unique `invoice_ref` per attempt; reusing one returns 425.
- Do not use the 2022 GitHub plugins (`payright.my/api`, Bearer auth, HMAC `signature` field) as a reference for the current API; they target a retired API.
- Do not confuse Payright Malaysia with Payright Australia (`payright-engineering` GitHub, BNPL).
- Verify current payment method activation, settlement schedule and pricing in the portal before claiming production certainty.

## Completion Criteria

A Payright integration is done only when:

1. Bill creation runs server-side with the `auth-token` header against the correct environment base URL.
2. A local pending payment with a unique `invoice_ref` exists before the bill is created, and `invoice_no` is stored after.
3. `provider_id` comes from the live `list-payment-provider` response for that environment, not a hardcoded table.
4. The callback handler verifies the `data` JWT with HS256 and the Hash Key, then matches `invoice_ref`, amount, and `invoice_status`.
5. Settlement is idempotent and confirmed with Get Bill; a scheduled requery covers missing callbacks.
6. If JWT Verification is enabled in the portal, every request carries a fresh `x-signature`.
7. Tests cover a forged or tampered JWT, wrong Hash Key, duplicate callback, wrong amount, unknown `invoice_ref`, non-SUCCESS status, and a successful payment.
8. Deployment docs list `PAYRIGHT_AUTH_TOKEN`, `PAYRIGHT_HASH_KEY`, `PAYRIGHT_COLLECTION_UUID`, `PAYRIGHT_BASE_URL`, callback URL, redirect URL, and sandbox/live separation.

## Loophole Review Loop

When asked whether a Payright strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, bank selection UI, backend bill creator, Payright API, callback receiver, JWT verifier, database, fulfilment job, payout job, portal settings.
2. Check bypasses: fake redirect success, forged callback JWT, unverified decode, `alg` confusion, wrong Hash Key, sandbox/live mix, matching on FPX `invoice_no` instead of `invoice_ref`, wrong amount, reused `invoice_ref`, stale Get Bill, duplicate callback, missing callback for failed payments, HTTP 200 with `status: fail`, expired `x-signature`, duplicate payout within 3 hours, insufficient payout pool.
3. Add fixes: server-side bill creation, local pending payment first, HS256-pinned JWT verification, strict `invoice_ref`/amount/status checks, durable callback log, idempotency keys, separate environment config, Get Bill requery job, short-lived `x-signature`.
4. Re-read `references/payright-api.md` sections relevant to any gap.
5. State remaining uncertainty where the docs are silent: refunds, status enum beyond `PENDING`/`SUCCESS`, callback retries, redirect parameters, rate limits, IP allowlists, and callback `exp`/`iat`.
