---
name: setup-herepay
description: Set up, build, debug, review, and explain Herepay payment integrations using official Herepay docs, the V1 direct API, Payment Channels, Initiate Invoice, callback, checksum (HMAC-SHA256 over comma-joined values sorted by key), Get Transactions requery, payment links, UAT/production setup, SecretKey/XApiKey/Private Key credentials, FPX test banks, and the official PHP/Node.js samples and WooCommerce plugin. Use when working with Herepay Malaysia, FPX Online Banking (B2C), FPX Corporate Banking (B2B), bank_prefix selection, callback verification, idempotent settlement, or production readiness.
---

# Herepay Payment Gateway

Use this skill for Herepay payment gateway integration work.

## Source

Read `references/herepay-api.md` before giving factual API guidance or writing integration code. It contains environments, auth headers, the payment channel and initiate flow, request fields, checksum algorithm, callback and redirect fields, status codes, requery, payment links, sandbox test banks, and documented inconsistencies.

Read `references/herepay-codes.md` for bank codes (B2C and B2B), UAT test banks, status and callback enums, and HTTP codes.

Read `references/account-setup.md` when the user asks how to register, which credentials are needed, where keys come from, UAT setup, callback/redirect URL setup, payment methods, pricing, or settlement timing.

Read `references/source-map.md` when choosing which Herepay docs page or official repo file applies.

## Core Workflow

1. Identify environment: UAT (`https://uat.herepay.org`) or production (`https://app.herepay.org`).
2. Identify surface: direct API (merchant builds bank picker), payment link (Herepay pay page), or WooCommerce plugin.
3. Confirm credentials for that environment: `SecretKey`, `XApiKey`, and the Private Key used for checksums, plus the callback and redirect URLs set in the Herepay dashboard.
4. Create a local pending payment with a unique `payment_code` before calling Herepay.
5. Fetch channels server-side with `GET /api/v1/herepay/payment/channels`; show only `Active` channels and keep `prefix` and `payment_method`.
6. Build the initiate payload, compute `checksum` over every field you send (excluding `checksum`), and `POST /api/v1/herepay/initiate` from the server with `SecretKey` and `XApiKey` headers.
7. Return the HTML body from Herepay to the browser; it form-POSTs the customer to the bank.
8. Treat the redirect URL POST as browser UX only; do not fulfil from it.
9. On the server-to-server callback, verify `checksum` with the Private Key using a constant-time compare, then settle idempotently on `payment_code`, amount, currency, and status.
10. Requery `GET /api/v1/herepay/transactions/{reference_code}` when a callback is missing, pending, or ambiguous.

## Core Facts

- UAT base: `https://uat.herepay.org`. Production base: `https://app.herepay.org`. Keys are not interchangeable.
- Auth headers on every V1 call: `SecretKey` and `XApiKey`. Payment links use `SecretKey` only.
- Payment channels: `GET /api/v1/herepay/payment/channels`.
- Create payment: `POST /api/v1/herepay/initiate` with `payment_code`, `created_at` (`YYYY-MM-DD hh:mm:ss`), `amount` (ringgit, not sen), `name`, `email`, `phone`, `description`, `bank_prefix`, `payment_method`, `checksum`; official samples also send optional `redirect_url`.
- Status requery: `GET /api/v1/herepay/transactions/{reference_code}`. List: `GET /api/v1/herepay/transactions`.
- Payment link (official samples only): `POST /api/integration/create-payment-link`, then redirect to `data.pay_url`.
- Checksum: sort fields by key, JSON-encode nested values, join values with `,`, HMAC-SHA256 with the Private Key, lowercase hex.
- Callback is a `POST` (form data) with `reference_code`, `payment_code`, `transaction_id`, `status`, `status_code`, `message`, `amount`, `currency`, `payment_method`, `fpx_type`, `bank_name`, `checksum`.
- Status codes: Status Codes page and official samples use `00` success, `29` pending, `30` failed, `41` unauthorized. The Callback reference page lists `1` completed, `2` pending, `0` failed. Handle both.
- A malformed initiate request only shows "Checksum verification failed"; there is no detailed error in the browser.
- Test banks (UAT): `TEST0021`, `TEST0022`, `TEST0023` (SBI Bank A/B/C), `ABB0234`, `LOAD001`. Test bank login `1234` / `1234`.
- Payment methods in API: `Online Banking` (B2C) and `Corporate Banking` (B2B). B2B can stay pending until an authorizer approves.
- Refunds, recurring billing, and tokenisation have no API in the official docs (checked 2026-10-21).
- Support: WhatsApp `+60 11-5140 4297` (from Getting Started page).

## Critical Rules

- Keep `SecretKey`, `XApiKey`, and the Private Key server-side only; never call Herepay from the browser.
- Sign the exact values you send, in the exact string form you send (for example `"10.50"` vs `10.5` changes the hash).
- If you add `redirect_url` or any other field, it must be inside the checksum input.
- Verify callback checksum over all posted fields except `checksum`, before trusting any field.
- Use constant-time comparison (`hash_equals`, `crypto.timingSafeEqual`, `hmac.compare_digest`).
- Do not mark paid from the redirect POST, the bank page, or a frontend message alone.
- Confirm `payment_code`, amount, currency `MYR`, and a success status before marking paid.
- Make callback processing idempotent; repeated callbacks must not double-fulfil.
- Store `reference_code` from the callback; requery needs it.
- Keep UAT and production keys, URLs, and callback endpoints separate.
- Do not use the published PHP SDK `generateChecksum` as-is; it passes the key through `openssl_pkey_get_private`, unlike every other official sample.
- Do not copy fee numbers into code or docs; link to the pricing page.

## Completion Criteria

A Herepay integration is done only when:

1. Channel fetch and initiate run server-side with `SecretKey` and `XApiKey` for the right environment.
2. Initiate checksum matches the official algorithm and covers every sent field.
3. Callback URL is registered in the dashboard for the right environment and is publicly reachable.
4. Callback handler verifies `checksum` with the Private Key in constant time.
5. Settlement checks `payment_code`, amount, currency, and status, accepting both `00` and `1` style success codes only after checksum passes.
6. Pending (`29` / `2`) and B2B authorizer-pending payments stay pending and are requeried later.
7. Tests cover bad checksum, duplicate callback, wrong amount, failed status, pending status, and success.
8. Deployment docs list `HEREPAY_SECRET_KEY`, `HEREPAY_API_KEY`, `HEREPAY_PRIVATE_KEY`, `HEREPAY_SANDBOX`, callback URL, and redirect URL.

## Loophole Review Loop

When asked whether a Herepay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser checkout form, merchant initiate route, Herepay API, bank page, redirect handler, callback receiver, database, fulfilment job, dashboard URL settings.
2. Check bypasses: forged callback, checksum computed over different field set, number formatting drift, unsigned `redirect_url`, fake redirect POST, UAT/production key mix, wrong amount, reused `payment_code`, duplicate callback, B2B pending treated as failed, status code `1` vs `00` confusion, inactive bank selected.
3. Add fixes: server-only initiate, local pending row first, unique `payment_code`, constant-time checksum check, strict amount/currency/status checks, idempotent settlement, requery by `reference_code`, environment-scoped config.
4. Re-read `references/herepay-api.md` sections relevant to any gap.
5. State remaining uncertainty: which status code set your account's callback sends, callback retry policy, rate limits, card/e-wallet `payment_method` values, refund process, and `created_at` timezone are not documented; confirm with Herepay support.
