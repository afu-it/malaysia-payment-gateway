---
name: setup-luminarix
description: Set up, build, debug, review, and explain Luminarix payment integrations using official Luminarix sources (the official Luminarix WooCommerce plugin published by Luminarix Sdn. Bhd. on WordPress.org, since no public developer docs exist), the v1 payments API (deposit, transaction-status, refund), hosted checkoutUrl redirect, channel payment instructions, X-Signature HMAC-SHA256 Base64 request checksums, signed callbacks with X-Timestamp and X-Signature, replay protection, per-payment-method Merchant ID, Sub-Merchant ID, Client Secret and Payment System Code, sandbox/live separation, status polling, full and partial refunds, and Cloudflare allowlisting. Use when working with Luminarix (lmnrx.com) Malaysia, FPX, DuitNow QR, cards, Touch 'n Go, GrabPay, Boost, ShopeePay, MAE, WooCommerce Luminarix setup, callback settlement, refunds, or production readiness.
---

# Luminarix Payment Gateway

Use this skill for Luminarix (Luminarix Sdn. Bhd., `https://www.lmnrx.com`) payment integration work.

Luminarix publishes no public developer docs (checked 2026-10-21). Every API fact here comes from the official WooCommerce plugin source that Luminarix publishes on WordPress.org (v1.6.0). Treat the plugin as the reference implementation and confirm anything marked "Not documented" with Luminarix onboarding.

## Source

Read `references/luminarix-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, headers, every request and response field the plugin uses, the exact string-to-sign for each call, callback verification, status values, refunds, retry and rate-limit handling, and the gaps.

Read `references/account-setup.md` when the user asks how to register, which credentials are needed, sandbox vs live, per-method credentials, env var names, WooCommerce plugin setup, pricing, or settlement.

Read `references/source-map.md` when choosing which source backs a fact, or to see what was checked and what is not documented.

## Core Workflow

1. Get from Luminarix onboarding: sandbox and live API base URLs, and for each payment method a Payment System Code, Merchant ID, numeric Sub-Merchant ID and Client Secret (sandbox and live sets are separate).
2. Create a local pending order with the expected amount in MYR.
3. Generate a fresh `orderId` for every payment attempt (never reuse one after a failed or abandoned attempt). Store it on the local order before calling the API.
4. `POST {base}/v1/payments/deposit` server-side with JSON body and header `X-Signature` = Base64(HMAC-SHA256(clientSecret, `subMerchantId|orderId|amount|currency|returnUrl|callbackUrl`)).
5. Store `data.transactionId`, then redirect to `data.checkoutUrl`. If no `checkoutUrl` is returned, show the payment instructions fields (QR string, virtual account, bank details).
6. Treat the browser return to `returnUrl` as UX only. Luminarix appends a query string (`orderId`, `transactionId`); check them against stored values, then requery.
7. On callback, verify `X-Signature` = Base64(HMAC-SHA256(clientSecret, `X-Timestamp + "|" + rawBody`)), enforce a timestamp window (plugin default 600 s), dedupe, and reply HTTP 200 `{"status":"SUCCESS"}`.
8. Before marking paid, confirm with `POST /v1/payments/transaction-status` (string-to-sign `subMerchantId|transactionId|orderId|amount|currency`).
9. Settle idempotently: SUCCESS is terminal; later PENDING, FAILED or CANCELLED events must not downgrade a paid order.
10. Poll pending orders as a fallback (plugin: every 5 minutes, up to 72 hours, small batches, backoff on 429).

## Core Facts

- API base URLs are not published; Luminarix supplies sandbox and live base URLs during onboarding.
- Endpoints (relative to base): `POST /v1/payments/deposit`, `POST /v1/payments/transaction-status`, `POST /v1/payments/refund`.
- All calls are JSON POSTs with `Content-Type: application/json`, `Accept: application/json` and `X-Signature`. The plugin also sends `X-Trace-Id` (UUID) and a descriptive `User-Agent`.
- Checksum: Base64 of raw HMAC-SHA256 bytes, key = Client Secret, message = fields joined with `|`. Signatures go in the `X-Signature` header, not the body (plugin 1.4.1).
- `amount` is a string in ringgit with 2 decimals (`"10.00"`), not sen.
- Credentials are per payment method: each Payment System Code (for example `MYR_TNG`) has its own Merchant ID, Sub-Merchant ID and Client Secret. Verify callbacks with the secret of the method and environment that created the order.
- Response envelope: `success` (bool), `data` (object), `code`, `description`, `errors[].message`.
- Codes seen in the plugin: `OK`, `TRANSACTION_NOT_FOUND`, `VALIDATION_FAILED`, `INTERNAL_ERROR`, `PROVIDER_UNAVAILABLE`. HTTP 429 carries `Retry-After`.
- Statuses handled: `SUCCESS`, `PENDING`, `ACCEPTED`, `APPROVED`, `FAILED`, `CANCELLED`, `REFUNDED` (plus aliases the plugin normalises defensively).
- Refunds: full or partial, signed like a status call with the refund amount; the response returns a separate refund `transactionId`, and refund status arrives on the same callback URL.
- The API sits behind Cloudflare. An HTML 403 means the server IP is blocked; ask Luminarix to allowlist it.
- If a payer object is sent, `countryCallingCode` is required by the deposit API (plugin 1.6.0 changelog).

## Critical Rules

- Keep the Client Secret server-side only. Never sign in the browser.
- Verify the callback over the raw request body bytes, before any JSON parsing or sanitising.
- Use constant-time comparison for signatures and stored references.
- Reject callbacks whose timestamp is outside your tolerance window, and dedupe on a hash of raw body plus signature.
- Reject callbacks whose `orderId` or `transactionId` does not match what you stored (plugin answers 409).
- Do not fulfil from the return URL or from a callback status alone; confirm SUCCESS with transaction-status first (plugin 1.5.0 behaviour).
- Use the exact `returnUrl` and `callbackUrl` strings in both the body and the string-to-sign. Keep `returnUrl` free of an existing query string, because Luminarix appends its own `?`.
- Use a new `orderId` per attempt, at most 255 characters; the plugin restricts it to `[A-Za-z0-9._-]`.
- Keep sandbox and live base URLs and credentials separate, and record the environment on each order.
- Do not invent Payment System Codes or channel codes; get the list from Luminarix.

## Completion Criteria

A Luminarix integration is done only when:

1. Deposit creation runs server-side with a correct `X-Signature` and a unique `orderId` saved before the call.
2. The customer is redirected to `checkoutUrl`, or instructions are shown when no URL is returned.
3. The callback handler verifies `X-Timestamp` and `X-Signature` over the raw body with the right per-method, per-environment secret, and returns HTTP 200 `{"status":"SUCCESS"}` for new and duplicate events.
4. Settlement confirms via transaction-status and checks orderId, transactionId, amount, currency and environment.
5. Duplicates, out-of-order callbacks, refunds and polling are idempotent, and paid orders never downgrade.
6. Tests cover bad signature, stale timestamp, replay, reference mismatch, wrong amount, FAILED, CANCELLED, SUCCESS and refund callbacks.
7. Deployment docs list `LUMINARIX_BASE_URL`, `LUMINARIX_MERCHANT_ID`, `LUMINARIX_SUB_MERCHANT_ID`, `LUMINARIX_CLIENT_SECRET`, `LUMINARIX_PAYMENT_SYSTEM_CODE`, the callback URL and the return URL, per environment.

## Loophole Review Loop

When asked whether a Luminarix strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser return, hosted checkout, backend deposit creator, Luminarix API behind Cloudflare, callback receiver, raw-body reader, database, fulfilment job, refund job, polling job, per-method credential store.
2. Check bypasses: forged return query, forged callback, signature over parsed JSON, wrong method secret, sandbox/live mix, replayed callback, stale timestamp, mismatched orderId or transactionId, wrong amount, reused orderId, late FAILED after SUCCESS, refund callback mistaken for payment, 429 storms, Cloudflare block hiding errors.
3. Add fixes: raw-body HMAC, constant-time compare, timestamp window, event hash dedupe, reference checks, server-side requery before fulfil, terminal SUCCESS, per-attempt orderId, per-method secrets, backoff with `Retry-After`, IP allowlisting request.
4. Re-read `references/luminarix-api.md` sections relevant to any gap.
5. State remaining uncertainty: base URLs, Payment System Code list, callback retry policy, timestamp format, sandbox test data, rate limit numbers and error codes are not publicly documented.
