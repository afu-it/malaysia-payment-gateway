---
name: setup-fasspay
description: Set up, build, debug, review, and explain Fasspay (Fass Payment Solutions, part of the Soft Space group) payment integrations using official Fasspay sources, the official Fasspay for WooCommerce plugin source, and the Soft Space Developer Portal, covering the e-commerce webInitiateRequest hosted payment page, HMAC-SHA256 X-Signature request signing, backendURL callback verification, statusId handling, checkTransactionStatus, void, pre-auth capture, white-label branding, UAT vs production, and the in-person Fasstap SoftPOS SDK, Fasstap App-to-App and Smart Terminal SDKs. Use when working with Fasspay or Soft Space for MYR card (Visa, Mastercard, JCB, MyDebit), FPX, DuitNow QR, e-wallet, tap-on-phone, Android POS terminal, callbacks, requery, voids, or production readiness.
---

# Fasspay Payment Gateway

Use this skill for Fasspay (Soft Space) payment integration work.

## Source

Read `references/fasspay-api.md` before giving factual guidance or writing code for online checkout. It holds the endpoint catalogue, environments, request signing, the create-payment fields, the callback format and its verification, status codes, requery, void, capture, and known gaps. Its facts come from the official Fasspay for WooCommerce plugin source (v1.0.14, published by Fasspay on WordPress.org), because Fasspay publishes no public e-commerce API document.

Read `references/fasspay-in-person.md` for card-present work: Fasstap SDK (tap on phone), Fasstap App-to-App, Smart Terminal SDK, and the provisioning Web API from `developers.softspace.com.my`. Field-level tables live in `references/fasspay-fasstap-sdk.md`, `references/fasspay-app-to-app.md`, `references/fasspay-smart-terminal.md`, and error codes in `references/fasspay-fasstap-codes.md`.

Read `references/account-setup.md` for registration, credentials, UAT vs production, payment method activation, settlement, and pricing links.

Read `references/source-map.md` to pick the right official page and to see coverage.

## Core Workflow

1. Identify surface: online checkout (e-commerce API), Fasstap SDK, Fasstap App-to-App, or Smart Terminal.
2. Identify environment: UAT `https://wallet-uat.fasspay.com` or production `https://ecom.fasspay.com`.
3. Confirm credentials from Fasspay: Merchant ID (`mid`) and the shared Secret Key for that environment.
4. Create a local pending payment with a unique `trxRefNo` per attempt, expected amount in sen, and currency `MYR`.
5. Call `POST /v3/ecom/webInitiateRequest` server-side with the `messages` envelope, `X-Timestamp`, and `X-Signature`.
6. Read `paymentURL` from the JSON string in the response `messages` field and redirect the customer there.
7. Treat `responsePageURL` as browser UX only; never mark paid from it.
8. On the `backendURL` callback, verify `X-Signature` over the raw body plus `X-Timestamp`, enforce the 300 second window, then parse.
9. Settle idempotently on `statusId` `100`, `104` (or legacy `00`) only after `trxRefNo`, amount, currency, and environment match.
10. Use `POST /v3/ecom/checkTransactionStatus` as a fallback when a callback is missing or unclear.

## Core Facts

- Production API base: `https://ecom.fasspay.com`. UAT base: `https://wallet-uat.fasspay.com`.
- All e-commerce calls are `POST` JSON over HTTPS under `/v3/ecom/`.
- Signature: `X-Signature = base64(HMAC-SHA256(secretKey, bodyJson + X-Timestamp))`, header `X-Signature-Alg: HmacSHA256`, `X-Timestamp` in Unix seconds.
- The secret is used as the literal string key; do not base64-decode it.
- Sign the exact bytes you send; the plugin uses `json_encode(..., JSON_UNESCAPED_SLASHES)`.
- Create payment wraps fields in `{"messages": "<JSON string>", "sandboxMode": <bool>}`.
- `amount` is an integer in sen (RM 1.00 is `100`). Only `MYR` is accepted by the plugin.
- Callback headers: `X-Signature` (or `X-Fasspay-Signature`) and `X-Timestamp`; body may be `{"messages": "<JSON string>"}`.
- Paid statuses: `100` APPROVED, `104` SETTLED, legacy `00`. Failed: `102` DECLINED, legacy `02`. Voided/cancelled: `107`, legacy `03`. Refunded: `06`.
- Pending statuses: `101` PROCESSING, `106` OPENED (pending from bank), `111` PROCESSING_VOID, `123` PROCESSING_CAPTURE, `115` PENDING_APPROVAL (FPX B2B), `122` AUTHORIZED (pre-auth).
- `transactionMethod` in callbacks: `0` FPX, `5` card, `34` e-wallet, `35` DuitNow QR.
- Void: `POST /v3/ecom/auth/void`; success is `statusId` `107`; the plugin blocks FPX voids and voids after the order's calendar day.
- No public refund-initiate endpoint is documented; refunds arrive as callback status `06`.
- In-person `statusId`/response codes differ from e-commerce codes (for example `102` is VOIDED in Fasstap). Never share one status map across surfaces.
- Public UAT credentials are embedded in the official plugin (values in `references/account-setup.md`, sandbox only); production MID and secret come from Fasspay at onboarding.

## Critical Rules

- Keep the Secret Key server-side only. Never call the e-commerce API from a browser or mobile client.
- Verify the callback signature on the raw request body before JSON parsing, using constant-time comparison.
- Reject callbacks whose `X-Timestamp` is more than 300 seconds from server time; reject a missing signature.
- Check amount, currency, `trxRefNo`, and environment yourself. The official plugin does not compare callback amount to order total.
- Generate a new `trxRefNo` per attempt and store it; ignore non-funds callbacks from superseded attempts.
- Treat a second funds-moved callback (`00`, `100`, `104`, `122`) on an already-paid order as a duplicate payment for manual refund or void.
- Never let a late decline (`02`, `102`) undo a recorded payment.
- Return HTTP 200 only after the callback is durably recorded; return 401 for bad signatures and 400 for bad JSON.
- Use UAT and production MID, secret, base URL, and `sandboxMode` as one set. Do not mix them.
- Do not copy in-person status codes into e-commerce handlers.

## Completion Criteria

A Fasspay integration is done only when:

1. `webInitiateRequest` runs server-side with correct envelope, `X-Timestamp`, and `X-Signature`.
2. The customer is redirected to `paymentURL`; the return page does not fulfil.
3. `backendURL` is public HTTPS and verifies `X-Signature` over raw body plus timestamp.
4. Settlement checks `trxRefNo`, `statusId`, amount, currency, environment, and attempt.
5. Duplicate callbacks, superseded attempts, and status requery are idempotent.
6. Tests cover bad signature, stale timestamp, duplicate callback, wrong amount, declined, pending (`101`, `106`, `115`), and approved.
7. Deployment docs list `FASSPAY_MID`, `FASSPAY_SECRET_KEY`, `FASSPAY_BASE_URL`, `FASSPAY_SANDBOX`, the callback URL, and the return URL.

## Loophole Review Loop

When asked whether a Fasspay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser return page, Fasspay hosted page, backend initiator, Fasspay API, callback receiver, raw-body reader, database, fulfilment job, void job, admin tooling.
2. Check bypasses: forged callback, signature over re-encoded JSON, base64-decoded secret, stale or replayed timestamp, UAT/production mix, reused `trxRefNo`, wrong amount unit, late decline, duplicate capture, FPX void attempt, next-day void, out-of-order attempt callbacks.
3. Add fixes: raw-body HMAC, constant-time compare, 300 second window, per-attempt `trxRefNo`, amount in sen check, idempotency key on `transactionId|statusId|trxRefNo`, requery fallback, separate env config.
4. Re-read `references/fasspay-api.md` sections relevant to any gap.
5. State remaining uncertainty: callback amount unit, `checkTransactionStatus` response shape, `refundAmount` unit, meaning of `transactionMethod` `100` in requests, IP allowlists, rate limits, and anything confirmable only with Fasspay support.
