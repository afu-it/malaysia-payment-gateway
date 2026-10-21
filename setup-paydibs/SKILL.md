---
name: setup-paydibs
description: Set up, build, debug, review, and explain Paydibs payment integrations using official Paydibs v3 Merchant Integration API docs, PAY request hosted checkout form post to /PPG/trigger, SHA512 Sign hashing, MerchantRURL browser redirect, MerchantCallbackURL server-to-server callback, QUERY status inquiry, REFUND API, Hosted Payment Session card SDK, recurring CIT/MIT payments, card tokens, PTxnStatus codes, and staging/production setup. Use when working with Paydibs Malaysia, FPX online banking, credit/debit cards, e-wallets, BNPL, DuitNow, callback verification, payment status requery, refunds, recurring billing, or production readiness.
---

# Paydibs Payment Gateway

Use this skill for Paydibs payment gateway integration work (Merchant Integration API v3, docs version 4.0 dated 24 Sep 2026).

## Source

Read `references/paydibs-api.md` before giving factual API guidance or writing integration code. It contains source URLs, environments, the PAY request field table, response and callback fields, every Sign formula with verified examples, PTxnStatus codes, QUERY, REFUND, Hosted Payment Session, recurring, and documented gaps.

Read `references/account-setup.md` when the user asks how to register, which credentials are needed, where the Merchant ID and Merchant Password come from, staging vs production, env var names, payment method activation, settlement, or pricing.

Read the companion files for their areas: `references/paydibs-codes.md` (all enums and error codes), `references/paydibs-recurring.md` (CIT, billing rules, MIT), `references/paydibs-hosted-session.md` (card SDK), `references/paydibs-tokens-instore.md` (BINDOPT, REVOKE, DuitNow QR), `references/paydibs-emandate.md` (E-Mandate and Direct Debit). The Endpoint Catalogue in `paydibs-api.md` lists every documented operation.

Read `references/source-map.md` when choosing which Paydibs docs page applies to payment, response, query, refund, sign hashing, hosted session, recurring, tokens, e-mandate, or in-store QR.

## Core Workflow

1. Identify environment: staging (test account) or production. Both URLs come from Paydibs with the account.
2. Identify integration surface: hosted checkout (PAY form post), Hosted Payment Session card SDK, recurring CIT/MIT, saved card token, or in-store DuitNow QR.
3. Confirm credentials from Paydibs: Merchant ID (6 chars), Merchant Password (hash secret), and the payment URL host.
4. Create a local pending payment with a unique `MerchantPymtID` (max 20) before building the PAY request.
5. Build the PAY request server-side, computing `Sign` = SHA512 hex of the documented field concatenation with Merchant Password first.
6. Auto-submit an HTML form (POST) from the browser to `<payment URL>/PPG/trigger`. Never expose Merchant Password to the browser.
7. Treat the `MerchantRURL` POST as browser UX only; verify its `Sign` but settle from the server callback or QUERY.
8. Handle `MerchantCallbackURL`: parse the urlencoded body (it carries a JSON string), verify `Sign`, check fields, return HTTP 200.
9. Settle idempotently only after `Sign`, `MerchantID`, `MerchantPymtID`, `MerchantTxnAmt`, `MerchantCurrCode`, and `PTxnStatus = 0` match.
10. Use `TxnType=QUERY` to `/PPG/trigger` as fallback when the callback is missing or status is pending (`2`).

## Core Facts

- Payment, query, MIT, bind and revoke endpoint path: `/PPG/trigger`. Refund: `/PPG/refund`. Hosted session: `/PPG/create-pay-session`. In-store QR: `/PPG/offline/trigger-qr`. E-Mandate: `/PPG/trigger-emandate`; Direct Debit: `/PPG/create-ddi`, `/PPG/create-ddi-txn`, `/PPG/cancel-ddi`.
- No void, capture, partial refund, settlement report or payout API is published.
- Host: "URL provided by Paydibs". The official Hosted Session SDK hardcodes `https://stgapi.paydibs.com` (stg) and `https://v3api.paydibs.com` (prod).
- `TxnType` values (uppercase): `PAY`, `QUERY`, `REFUND`, `REVOKE`, `BINDOPT`, `MIT`.
- Sign algorithm: SHA512, hexadecimal string, 128 chars. Plain concatenation, no separators, Merchant Password first.
- PAY Sign: `MerchantPassword + TxnType + MerchantID + MerchantPymtID + MerchantOrdID + MerchantRURL + MerchantTxnAmt + MerchantCurrCode + CustIP + PageTimeout + MerchantCallbackURL + Token` (absent optional fields = empty string).
- Response/Query Sign: `MerchantPassword + MerchantID + MerchantPymtID + PTxnID + MerchantOrdID + MerchantTxnAmt + MerchantCurrCode + PTxnStatus + AuthCode`.
- Amount: ringgit with exactly 2 decimals, no thousands separator (`1000.00`, not `1000` or `1,000.00`).
- PTxnStatus: `0` success, `1` failed, `2` pending, `9` voided, `10` refunded, `17`/`18` cancelled, `19` expired, `20` chargeback, `22` bank maintenance, `50`/`51` amount limit, `-1` not found, `-2` internal error.
- Callback: content type `application/x-www-form-urlencoded`, body is a JSON string; retried up to 5 times until HTTP 200.
- QUERY `PAYExists`: `0` exists, `1` does not exist, `2` error, retry. Note `0` means found.
- Method codes: `OB` online banking, `CC` card, `WA` e-wallet. Currencies include MYR, SGD, PHP, VND, CNY, HKD, IDR, KHR, KRW, THB.
- Production allows TLS 1.2 only; test TLS 1.2 against production before go-live.
- Replace `&` with `;` inside `MerchantRURL` and `MerchantCallbackURL` query strings.

## Critical Rules

- Keep Merchant Password server-side only; it is the hash secret.
- Do not mark paid from the `MerchantRURL` redirect alone, even with a valid Sign; prefer callback or QUERY.
- Recompute Sign over the exact received field values and compare in constant time (case-insensitive hex).
- Reject callbacks whose `MerchantID`, amount, currency, or `MerchantPymtID` differ from the stored pending payment.
- `MerchantPymtID` must be unique per attempt; reuse `MerchantOrdID` across retries of one order.
- Return HTTP 200 only after the callback is durably recorded; make processing idempotent across the 5 retries.
- Treat status `2` (pending) as not final; requery later. Do not fail it early.
- Use the Merchant ID, Password and host of the same environment; never mix staging and production.
- Refund response `RefundResult` is boolean; Paydibs sample shows refunds only allowed from the next day onwards.
- Verify payment method availability, fees, settlement and TLS in production with Paydibs before claiming production certainty.

## Completion Criteria

A Paydibs integration is done only when:

1. PAY request and Sign are built server-side and posted to `/PPG/trigger` via auto-submit form.
2. `MerchantRURL` handler verifies Sign and only shows status; it does not fulfil on its own.
3. `MerchantCallbackURL` handler parses the JSON string body, verifies Sign, checks fields, returns HTTP 200, and is idempotent.
4. QUERY fallback runs for missing callbacks and pending (`2`) payments.
5. Settlement checks MerchantID, MerchantPymtID, amount, currency, and `PTxnStatus = 0`.
6. Tests cover a bad Sign, a duplicate callback, wrong amount/currency/ID, pending then success, failed/cancelled/expired, and a successful payment.
7. Deployment docs list `PAYDIBS_MERCHANT_ID`, `PAYDIBS_MERCHANT_PASSWORD`, `PAYDIBS_BASE_URL`, return URL, callback URL, and staging/production separation.

## Loophole Review Loop

When asked whether a Paydibs strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser form post, Paydibs payment page, MerchantRURL handler, callback receiver, QUERY client, database, fulfilment job, refund job, hosted session SDK iframe, and Paydibs account settings.
2. Check bypasses: forged RURL post, forged callback, Sign over wrong field order, null vs empty AuthCode, Password leaked to frontend, staging/live mix, wrong amount format, reused MerchantPymtID, duplicate callback, retry after partial processing, pending treated as failed, `PAYExists=0` misread as missing, duplicate refund, and `&` in return URLs.
3. Add fixes: server-side Sign, local pending payment first, constant-time Sign check, strict field checks, durable event log, idempotency keys, environment-scoped config, QUERY fallback, and refund state tracking.
4. Re-read `references/paydibs-api.md` sections relevant to any gap.
5. State remaining uncertainty where docs are silent: base URLs (provided by Paydibs), test cards/banks, IP allowlists, rate limits, refund windows, and null-field handling in Sign.
