---
name: setup-axaipay
description: Set up, build, debug, review, and explain Axaipay payment integrations using official Axaipay docs, eComPay hosted payment page (HTML form POST), bypass payment page with FPX acquirer codes, DuitNow QR and Alipay+ QR code generation, HMAC-SHA512 Base64 request and response signatures, redirect URL and backend URL callbacks, inquiry and void APIs, AutoDebit (DirectDebit eMandate) enrolment and recurring charge callbacks, staging and production setup, merchant ID, signing key, API key, test cards, error codes, refunds by form, and WooCommerce, OpenCart, PrestaShop and VirtueMart plugins. Use when working with Axaipay, Axai Digital, eComPay, FPX B2C/B2B, cards, e-wallets, DuitNow QR, recurring direct debit, callback verification, transaction status requery, or production readiness.
---

# Axaipay Payment Gateway

Use this skill for Axaipay (Axai Digital Sdn Bhd) payment gateway integration work.

## Source

Read `references/axaipay-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, every request/response field, the exact string-to-sign for each signature (checked against the documented sample digests), status and acquirer enums, bank codes, error codes, test cards, and known doc conflicts.

Read `references/account-setup.md` when the user asks how to register, which credentials exist, where keys come from, staging vs production, plugins, refunds, AutoDebit products, dashboard security, or pricing.

Read `references/source-map.md` to pick the right official page or PDF, and for the coverage record.

## Core Workflow

1. Identify environment: staging (demo account from `support@axaipay.com`) or production.
2. Identify surface: hosted payment page, bypass page (direct FPX bank or e-wallet), server-to-server QR (DuitNow QR / Alipay+), AutoDebit enrolment, or a shop plugin.
3. Create a local pending order with a unique `mchtTrxnId` before calling Axaipay.
4. Build the signature server-side: concatenate the documented field values in the documented order, HMAC-SHA512 with the signing key, Base64 encode.
5. Hosted page: render an auto-submitting HTML form POST to `/gateway/v1/payment`. QR: POST form-encoded body to `/trxn/qrcode/generate` with `x-api-key` and `x-signature` headers, then show `qrCodePayload` as a QR.
6. Treat the POST to `redirectUrl` as browser UX only.
7. On the POST to `backendUrl`, verify `signature` over the 11 response fields before trusting anything.
8. Settle idempotently on `trxnStatus = 11` after matching `mchtId`, `mchtTrxnId`, amount and the stored order.
9. Use `/trxn/inquiry` (by `trxnId` or `mchtTrxnId`) as fallback when the callback is late or missing, and verify the inquiry response signature too.
10. Refunds are not an API: the merchant submits Axaipay's Refund Request Form to support.

## Core Facts

- Hosted page: staging `https://staging.axaipay.my/gateway/v1/payment`, production `https://secured.axaipay.my/gateway/v1/payment`.
- REST base: staging `https://api-staging.axaipay.my`, production `https://api.axaipay.my` (`/trxn/qrcode/generate`, `/trxn/inquiry`, `/trxn/void`, `/emandate/v1/inquiry`).
- AutoDebit enrolment: `https://staging.axaipay.my/gateway/emandate/v1/enrolment` and `https://secured.axaipay.my/gateway/emandate/v1/enrolment`.
- Credentials: merchant ID (`mchtId`), signing key (HMAC secret), API key (`x-api-key` header for REST calls).
- Every signature is HMAC-SHA512 over plain concatenated values (no separators, no field names), output Base64.
- Payment request string: `backendUrl + customerEmail + customerName + customerPhone + mchtId + mchtTrxnId + orderDescription + redirectUrl + txnAmount` (optional URLs only when sent).
- Response/callback string: `mchtId + mchtTrxnId + trxnAcquirer + trxnAmount + trxnBankName + trxnFpxMethod + trxnFpxType + trxnId + trxnPaymentMethod + trxnStatus + trxnTime`. Values keep their spaces. `trxnBuyerName` is excluded.
- `paymentMethod` and `acquirerCode` (bypass page) are NOT part of the request signature.
- Amounts are ringgit with 2 decimals (`Decimal(12,2)`, e.g. `4.25`), MYR only.
- `trxnStatus`: 1 Created, 2 In Processing, 3 Pending Authorization, 11 Paid, 22 Failed, 23 Timeout, 33 Void, 44 Refund, 55 Cancel.
- Default redirect and backend URLs are registered with Axaipay at onboarding; `redirectUrl`/`backendUrl` in the request override them.
- Staging FPX test banks: `B2C_TEST`, `B2B_TEST`. Signature checker: `https://staging.axaipay.my:8888/generate-signature`.
- AutoDebit enrolment is an RM1.00 FPX payment (`trxnRecurringNo = 0`); later debits post to the backend URL with the same payload shape.

## Critical Rules

- Keep the signing key and API key server-side only; the form POST carries only the signature.
- Never mark paid from the redirect POST alone; settle from a verified backend callback or a verified inquiry response.
- Compare signatures in constant time. Rebuild the string from the received POST values exactly as received (no trimming, no number reformatting).
- Use the key set for the same environment as the URL. Staging and production credentials differ.
- Check `trxnAmount` equals the stored amount and `mchtId` equals your merchant ID before settling.
- Reuse of a `mchtTrxnId` fails with `PY022`; generate a fresh one per attempt.
- The official WooCommerce plugin skips signature checks on redirect when `trxnStatus = 55`; never apply an unsigned status to an order. Confirm with inquiry first.
- Backend callback handlers should return HTTP 200 quickly (the official plugin answers `OK`); the required ack body is not documented.
- Inquiry by `mchtTrxnId` has a doc conflict on field order; see `references/axaipay-api.md`.
- Verify enabled payment methods, settlement terms and pricing with Axaipay before claiming production certainty.

## Completion Criteria

An Axaipay integration is done only when:

1. The payment form or QR request is built server-side with a signature that matches the documented examples in a unit test.
2. The redirect handler only shows status; fulfilment happens from the backend callback or inquiry.
3. The backend handler verifies `signature`, then checks `mchtId`, `mchtTrxnId`, `trxnAmount` and `trxnStatus`.
4. Duplicate callbacks, redirect-plus-callback races and inquiry fallbacks are idempotent.
5. Tests cover invalid signature, wrong amount, unknown `mchtTrxnId`, status 22/23/55, duplicate callback, and status 11.
6. Deployment docs list `AXAIPAY_MERCHANT_ID`, `AXAIPAY_SIGNING_KEY`, `AXAIPAY_API_KEY`, environment flag, redirect URL and backend URL.

## Loophole Review Loop

When asked whether an Axaipay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser form POST, redirect POST, backend callback, QR display, REST client, signing key store, database, fulfilment job, AutoDebit recurring callbacks, refund form process, plugin code.
2. Check bypasses: forged redirect, forged callback, signature built from trimmed or reformatted values, unsigned status 55, wrong environment key, amount tampering in the form, reused `mchtTrxnId`, late callback after timeout, recurring callback replay, inquiry response not verified.
3. Add fixes: server-side signing, raw POST values for verification, constant-time compare, amount and merchant checks, unique transaction IDs, durable event log keyed by `trxnId` plus `trxnRecurringNo`, inquiry fallback, environment-scoped config.
4. Re-read `references/axaipay-api.md` sections relevant to any gap.
5. State remaining uncertainty: callback retry policy, ack format, IP allowlist, rate limits and void response shape are not documented; confirm with `support@axaipay.com`.
