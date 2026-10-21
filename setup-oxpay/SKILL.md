---
name: setup-oxpay
description: Set up, build, debug, review, and explain OxPay payment integrations using official OxPay docs, the OxPay Payments API (v2), Hosted Payment Page (/payment-page), merchant-hosted Create Payment (/payment), void, refund, payment query by ID, session or merchant reference, payment/refund/void webhooks, Bearer API keys, HMAC-SHA256 Signature header with the Sign Key, card pre-auth capture flag, MPQR/CPQR QR wallets, payouts, merchant portal setup, and production go-live testing. Use when working with OxPay Malaysia (OxPay (M) Sdn Bhd, formerly MC Payment), OxPay Financial, cards, Alipay, WeChat Pay, GrabPay, ShopeePay, Atome, UPI, QR payments, hosted checkout, webhook settlement, refunds, voids, payouts, or production readiness.
---

# OxPay Payment Gateway

Use this skill for OxPay (OxPay Financial / OxPay Malaysia) payment gateway integration work.

## Source

Read `references/oxpay-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, environments, required headers, the Signature (HMAC-SHA256) algorithm, full request/response field tables for hosted page, create payment, void, refund, queries, payouts, all webhooks, status codes, transaction states, brand enums and known doc gaps.

Read `references/account-setup.md` when the user asks how to become an OxPay merchant, where API keys and Sign Keys live, the merchant portal, MID/TID, go-live testing, payment method activation, settlement reports, pricing or support channels.

Read `references/source-map.md` to pick the right official page or to check docs coverage.

## Core Workflow

1. Confirm the merchant is onboarded and has MID (`merchant_id`, max 9 chars) and TID (`terminal_id`, exactly 14 chars) from OxPay.
2. Choose surface: Hosted Payment Page (`POST /payment-page`, terminal needs the Ecommerce feature) or merchant-hosted Create Payment (`POST /payment`, card data requires your own PCI DSS compliance).
3. Create a local pending order first, with a `merchant_reference_id` unique per terminal (1 to 100 chars).
4. Serialize the request JSON once in compact form, sign it with HMAC-SHA256 using the Sign Key, send the same bytes with `Authorization: Bearer <API_KEY>`, `Signature: <hex>` and a `User-Agent` header.
5. For hosted checkout, redirect the browser to `body.oxpay_hpp_url`. For QR (MPQR) render `body.qr_string` before `payment_expiry_time`.
6. Treat `success_url` / `failure_url` as browser UX only. Do not fulfil from them.
7. Receive the payment webhook at `notification_url`, verify the `Signature` header with the Sign Key, then parse.
8. Settle idempotently only when `headers.status_code` is `0`, `body.transaction_state` is `success` (or `captured` for pre-auth capture), and `merchant_reference_id`, `oxpay_txn_id`, amount and currency match the local order.
9. If the webhook is late or missing, requery with `GET /payment/{oxpay_txn_id}` or `GET /payment?merchant_reference_id=...`.
10. Refund with `POST /refund` (successful sale) or void/cancel with `POST /void`; final outcome comes via the refund or void webhook.

## Core Facts

- Live base URL: `https://api.oxpayfinancial.com`. Sandbox listed in the OpenAPI spec: `https://api.uat.oxpayfinancial.com`, but Getting Started says "no UAT environment available" and testing is done in production with OxPay test cases. Confirm with OxPay.
- API version in spec: 2.0.2; optional header `OxPay-Version: 2.0.1`.
- Auth header: `Authorization: Bearer <API_KEY>` (keys look like `ak-...`).
- Integrity header: `Signature` = lowercase hex HMAC-SHA256 of the compact JSON body (no whitespace between tokens, original key order) keyed with the Sign Key. Required on POST requests; sent by OxPay on every webhook.
- `User-Agent` header is required on every request.
- GET query endpoints and `GET /payouts/{unique_reference}` take Authorization and User-Agent only (no Signature in spec).
- Amounts are positive integers in the smallest currency unit (docs example: SGD 1 = 100). For MYR this means sen.
- Currency is ISO 4217 alpha-3. Public examples use SGD only; MYR support depends on your account.
- `headers.status_code`: 0 Success, 1 Pending, 2 Rejected, 3 Error. Rejected/Error return HTTP 400.
- `transaction_state` enum: success, requested, authorized, pending, rejected, error, expired, voided, refunded, captured, capture_submitted, capture_rejected.
- Documented brands: `card`, `alipay`, `wechat pay`, `grabpay`, `shopeepay`, `atome`, `dash`, `PayNow`, `upi`. FPX, DuitNow and Touch 'n Go are not in the public API docs (checked 2026-10-21).
- Card `capture: false` means pre-auth. No capture endpoint is documented.
- Merchant portal: `https://merchant.oxpayfinancial.com`; login `https://login.oxpayfinancial.com/login`. API keys: Configurations > API Keys > Add New Key.
- Support: `support@oxpayfinancial.com`, `https://oxpay.freshdesk.com/support/home`.

## Critical Rules

- Keep the API key and Sign Key server-side only.
- Sign exactly the bytes you send. Do not pretty-print after signing or let an HTTP client re-serialize the body.
- Verify the webhook `Signature` before trusting any field; use a constant-time comparison.
- Never mark paid from `success_url`, a frontend callback, a `status_code` of 1, or a create-payment response alone (MPQR returns Pending).
- Check amount, currency, `merchant_reference_id`, `oxpay_txn_id`, MID/TID and state before settling.
- Make webhook handling idempotent; key on `oxpay_txn_id` plus transaction type.
- Do not send raw card data to `/payment` unless the merchant is PCI DSS compliant; otherwise use the Hosted Payment Page.
- Keep `merchant_reference_id` unique per terminal and stable across retries.
- Session IDs from `/payment-page` expire; request a new one after expiry.
- Webhook retry policy, source IPs, required HTTP response and rate limit numbers are not documented. Do not claim them.
- Confirm MYR, FPX, DuitNow, e-wallet and BNPL availability with OxPay before promising them.

## Completion Criteria

An OxPay integration is done only when:

1. Hosted page or create payment runs server-side with Bearer API key, Signature and User-Agent headers.
2. The signed string and the sent body are byte-identical, tested with a known vector.
3. Webhook handler verifies `Signature` with the Sign Key before parsing and responds quickly.
4. Settlement checks status_code, transaction_state, reference IDs, amount, currency and terminal.
5. Duplicate webhooks, late webhooks and requery are idempotent.
6. Refund and void flows store `oxpay_original_txn_id` and settle from their webhooks.
7. Tests cover bad signature, wrong amount/currency/reference, pending, rejected, expired, duplicate and success.
8. Deployment docs list `OXPAY_API_KEY`, `OXPAY_SIGN_KEY`, `OXPAY_MERCHANT_ID`, `OXPAY_TERMINAL_ID`, `OXPAY_BASE_URL`, notification URL and go-live test results.

## Loophole Review Loop

When asked whether an OxPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, hosted page, merchant card form, backend signer, OxPay API, webhook receiver, JSON serializer, database, fulfilment, refund/void jobs, payout jobs, merchant portal.
2. Check bypasses: fake success redirect, forged webhook, signature over re-ordered or pretty JSON, unicode escaping mismatch, wrong Sign Key, wrong amount/currency/reference, reused `merchant_reference_id`, Pending treated as paid, expired QR or session, duplicate webhook, refund replay, pre-auth treated as captured, UAT/live confusion.
3. Add fixes: server-side signing, single serialization, constant-time compare, strict field checks, durable event log, idempotency keys, requery fallback, environment config separation.
4. Re-read `references/oxpay-api.md` sections relevant to any gap.
5. State remaining uncertainty: sandbox availability, MYR and local method support, webhook retries and response contract, rate limits, capture flow, settlement timing and pricing.
