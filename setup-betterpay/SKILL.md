---
name: setup-betterpay
description: Set up, build, debug, review, and explain Betterpay (formerly QlicknPay, Pay Direct Technology) payment integrations using official Betterpay docs, Postman collections and the WooCommerce plugin source, covering PRO and LITE accounts, Standard and Direct Payment, MD5 and HMAC-SHA256 hashing, callbacks, requery, FPX and DuitNow QR refunds, settlement reports, payment links, invoices, e-Mandate and card recurring, OCBC collection, card tokenization and pre-authorization. Use when working with Betterpay or QlicknPay Malaysia, FPX, DuitNow QR, cards, e-wallets, BNPL, IPP, callback verification, sandbox setup, or production readiness.
---

# Betterpay Payment Gateway

Use this skill for Betterpay payment gateway integration work.

## Source

Read `references/betterpay-api.md` before giving factual API guidance or writing integration code. It covers:

- PRO vs LITE, environments and hosts, auth
- Both hash algorithms, the endpoint catalogue
- Standard and Direct Payment fields, callbacks, requery
- Refunds, reports, payment links, invoices
- Rate limit, sandbox data, errors

Read `references/betterpay-recurring.md` for e-Mandate direct debit, OCBC collection batches or LITE card recurring.

Read `references/betterpay-tokenization.md` for card tokens, token charge, batch charge, token pre-auth or Card Pre-authorization (MPGS).

Read `references/betterpay-codes.md` for any status, error, bank, channel or enum value.

Read `references/account-setup.md` for registration, where keys live, API Management settings, sandbox vs live, env vars, method activation, plugins or settlement.

Read `references/source-map.md` to pick the right official docs anchor or Postman collection.

## Core Workflow

1. Identify the product from the merchant ID: an `R` prefix (e.g. `R1001`) is LITE (HMAC-SHA256); numeric (e.g. `10001`) is PRO (MD5).
2. Identify the environment: sandbox `https://www.demo.betterpay.me`, or production. Confirm the production paths in the logged-in docs.
3. Identify the surface: Standard Payment (hosted checkout), Direct Payment (bank/e-wallet/card/DuitNow QR), payment link, bill, invoice, recurring, token, pre-auth or plugin.
4. Configure API Management: callback URL, Whitelist Domain (it must match the `callback_url_be` host), and return URLs.
5. Create a local pending order with a unique invoice (max 14 chars, no `& ' -`) and the amount in ringgit with 2 decimals.
6. Build the request and hash server-side with the endpoint's exact field order (PRO) or the sorted-values rule (LITE).
7. Send the browser via form POST (Standard) or redirect to `payment_url` (Direct with `respond=1`).
8. Treat `callback_url_fe_succ` / `callback_url_fe_fail` as UX only.
9. On the `callback_url_be` POST:
   1. Verify the hash.
   2. Check invoice, amount and `txn_status`.
   3. Settle idempotently.
   4. Reply with plain `OK`.
10. Requery (Get Payment Status) when no callback arrives or the status is pending.

## Core Facts

- Betterpay is the QlicknPay stack renamed. The docs say Betterpay.me and qnp.com hosts are both valid. The docs still say QlicknPay in many places.
- Docs: `https://www.betterpay.me/docs/` (one page, endpoints behind login). Sandbox paths are in `https://github.com/betterpayme/POSTMAN-APIs`.
- PRO production host per the official WP plugin: `https://www.betterpay.me`. LITE plugin checkout: `https://lite.betterpay.me/api/merchant/v1/checkout`.
- Auth: `merchant_id` + `hash` in the body. The API key only signs. HTTPS only. Every request is hashed.
- PRO Standard hash: `md5(api|merchant_id|invoice|amount|payment_desc|currency)`.
- PRO callback check: `md5(api . trx_id . invoice_no . txn_status . msg)` with no separators. `trx_id` is `fpx_fpxTxnId`, `paypal_trx_id`, `mastercard_trx_id` or `others_trx_id`.
- Some PRO calls put `merchant_id` before the API key: requery, refunds, refund status.
- LITE hash: sort all sent fields except `hash` by name, concatenate the values, remove spaces, then `hash_hmac('sha256', s, API_KEY)`. Verify the callback the same way over the received fields.
- Status codes:
  - PRO: `00` approved, `09` pending, `99` pending authorization (B2B).
  - LITE: `00` success, `11` failed, `22` pending, `failed` cancelled.
- Amount limits: RM1.50 minimum and under RM30,000 (B2C). Amounts are strings with 2 decimals.
- Callback retry: Betterpay resends at most 3 times every 10 minutes until it gets `OK`.
- Rate limit: GET endpoints share 100 requests per 5 minutes per IP or account. Over the limit gives HTTP 429 `{"response":"99",...}`.
- FPX refund API: only with your own FPX seller ID, max 10 per request, within 5 days. DuitNow QR refunds have their own endpoint.

## Critical Rules

- Keep the API key server-side. Never put it in the browser form. Compute the hash on the server.
- Never mix PRO and LITE hashing or endpoints. Never mix sandbox and live keys.
- Use the exact per-endpoint field order from the reference tables. Never guess an order.
- Hash the plain values exactly as sent (the PHP samples `urldecode` each value). Send amounts as strings.
- Never fulfil from the return URL or a client-side message. Fulfil only from a hash-verified backend callback or a requery.
- Compare hashes in constant time. Then check that the invoice exists, the amount and currency match, and `txn_status` is the success code.
- Make the callback handler idempotent and always reply plain `OK` once the callback is stored. Retries and duplicates happen.
- Keep `callback_url_be` on the Whitelist Domain host, or the payment is rejected ("Invalid URL variable specified").
- Each invoice must be unique per transaction. Retrying with the same invoice fails.
- Treat endpoints marked "Not published" and the doc/Postman conflicts in the references as unconfirmed. Test in sandbox and confirm with support.

## Completion Criteria

A Betterpay integration is done only when:

1. The product (PRO/LITE), environment, host, merchant ID and API key come from config, never hardcoded.
2. Payment creation and hashing run server-side with the documented field order (PRO) or sort rule (LITE).
3. API Management has the callback URL, Whitelist Domain and return URLs for the right environment.
4. The callback handler:
   1. verifies the hash over the documented input,
   2. checks invoice, amount, currency and `txn_status`,
   3. settles once,
   4. replies `OK`.
5. Pending results (`09`, `99`, `22`) and missing callbacks are resolved by Get Payment Status, idempotently.
6. Tests cover a bad hash, a replayed callback, a wrong amount, an unknown invoice, a failed/cancelled status, a pending status and a successful payment.
7. Deployment docs list `BETTERPAY_PRODUCT`, `BETTERPAY_ENV`, `BETTERPAY_BASE_URL`, `BETTERPAY_MERCHANT_ID`, `BETTERPAY_API_KEY` and the callback/return URLs, with sandbox and live kept apart.

## Loophole Review Loop

When asked whether a Betterpay strategy is "100% confident", run this loop until no factual gaps remain:

1. **List trust boundaries**: browser form POST, hosted checkout, bank/e-wallet page, return URLs, backend creator, Betterpay API, callback receiver, database, fulfilment, refund job, API Management settings, plugin.
2. **Check bypasses**:
   - fake return redirect, forged callback, hash built from parsed or re-encoded values
   - wrong field order, PRO/LITE mix, sandbox/live key mix
   - wrong amount or invoice, reused invoice
   - pending treated as paid, duplicate callback
   - missing `OK` causing retries, callback host not whitelisted
   - stale requery, refund replay, `failed` (cancel) treated as error-free
3. **Add fixes**:
   - server-side hashing, local pending order first
   - constant-time compare, strict amount/invoice/status checks
   - durable callback log, idempotent settlement
   - separate env config, requery fallback, sandbox tests for every unconfirmed formula
4. **Re-read** the relevant section of `references/betterpay-api.md` (or the recurring/tokenization/codes file) for any gap.
5. **State remaining uncertainty**: production endpoint paths (behind login), LITE callback field set, LITE payment link hash conflict, plan-to-product mapping, enabled channels, refund eligibility, settlement timing.
