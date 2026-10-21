---
name: setup-ringgitpay
description: Set up, build, debug, review, and explain RinggitPay payment integrations using official RinggitPay docs, the RinggitPay Integration and Testing specification, the official RinggitPay for WooCommerce plugin source, and the RinggitPay Direct Debit (e-Mandate) API guide, covering hosted payment page form POST, AppId, RequestKey, ResponseKey, SHA-256 pipe-delimited checksums, APIURL server-to-server direct response, RedirectURL and ReturnURL browser response, transaction enquiry, RP and IR status codes, UAT simulators and test cards, FPX B2C and B2B, cards, payer account verification, and e-mandate DDA/DDI recurring debits. Use when working with RinggitPay Malaysia, ringgitpay.com or ringgitpay.co payment endpoints, FPX, Visa/Mastercard, direct debit, e-mandate, WooCommerce RinggitPay, callback checksum verification, UAT sign-off, or production readiness.
---

# RinggitPay Payment Gateway

Use this skill for RinggitPay (RinggitPay Sdn Bhd, Ascertain Technologies group) payment integration work.

## Source

Read `references/ringgitpay-api.md` before giving factual API guidance or writing integration code. It has the endpoint catalogue, environments, request and response field tables, checksum formulas, status codes, enquiry API, UAT test data, and the Direct Debit (e-Mandate) REST API.

Read `references/account-setup.md` when the user asks how to register, where AppId/RequestKey/ResponseKey come from, how to set APIURL and RedirectURL, UAT vs production, UAT sign-off, payment methods, settlement timing, or pricing.

Read `references/source-map.md` to see which source backs which fact, and the source-trust caveat: the integration PDFs are RinggitPay-authored but were found on a third-party GitHub mirror, not on a RinggitPay domain.

## Core Workflow

1. Identify environment: UAT (`ringgitpay.co`) or production (`ringgitpay.com`).
2. Identify surface: hosted payment page (one-off FPX/card payment), WooCommerce plugin, or Direct Debit e-Mandate REST API (recurring).
3. Confirm with RinggitPay: AppId, RequestKey, ResponseKey, and that APIURL (server-to-server) and RedirectURL (browser) are registered for this AppId. They are configured by RinggitPay, not sent per request (except optional `returnURL`).
4. Create a local pending order with a unique `orderId` (max 20 chars) and the exact amount string (2 decimals) before redirecting.
5. Compute `checkSum = SHA256(appId|currency|amount|orderId|REQUESTKEY)` server-side and auto-submit an HTML form POST to `/payment`.
6. Treat the RedirectURL/ReturnURL POST as browser UX; verify its checksum but settle from the APIURL direct message or a transaction enquiry.
7. Verify `rp_checkSum = SHA256(rp_appId|rp_currency|rp_amount|rp_statusCode|rp_orderId|rp_transactionRef|RESPONSEKEY)` using the raw received values.
8. Settle idempotently only on `RP00` after checksum, AppId, orderId, amount, currency and environment all match. `RP09` is pending, never final.
9. Use `POST /transactionenquiry` (JSON) for pending, missing or disputed results.

## Core Facts

- Payment request: `POST https://ringgitpay.co/payment` (UAT), `POST https://ringgitpay.com/payment` (production). Form POST, name/value pairs.
- Transaction enquiry: `POST https://ringgitpay.co/transactionenquiry` (UAT), `POST https://ringgitpay.com/transactionenquiry` (production). JSON in, JSON out.
- Request fields: `appId`, `currency` (`MYR`), `amount` (2 decimals), `orderId`, `checkSum`; optional `buyerEmail`, `returnURL` (URL-encoded, same domain), `accNumber` and `accBuyerId` (payer verification).
- Response fields are prefixed `rp_`: `rp_appId`, `rp_currency`, `rp_amount`, `rp_statusCode`, `rp_orderId`, `rp_transactionRef`, `rp_txnTime` (UTC `YYYYMMDDHH24mmSS`), `rp_paymentMode` (`FPX`, `CC`, `DC`, `NA`), `rp_checkSum`, `rp_remarks`, `rp_xtraInfo`, `rp_paymentCountry`.
- Request and enquiry checksums use RequestKey; response checksums use ResponseKey. All are SHA-256 over pipe-joined values with the key appended last.
- Spec says request checksum must be upper case hex; the official plugin sends lower-case hex and compares responses case-insensitively. Compare case-insensitively.
- Status `RP00` = success, `RP09` = pending, `IR1x/IR20` = request validation errors, other `RPxx` = failures (full table in the API reference).
- RinggitPay sends a DIRECT message (host-to-host to APIURL) and an INDIRECT message (browser form POST to RedirectURL, or ReturnURL if sent).
- Only `MYR` is supported (official plugin FAQ).
- FPX limits: retail B2C RM1.00 to RM30,000.00; corporate B2B RM2.00 to RM1,000,000.00. B2B stays `RP09` until the corporate authoriser acts.
- Direct Debit API: base `https://bills2u-api.ringgitpay.co` (UAT) / `https://bills2u-api.ringgitpay.biz` (production), OAuth2 token from `https://identity.ringgitpay.{co|biz}/connect/token`, paths under `/api/v1/mandate/...`.
- Refund, void, capture, payout and settlement-report APIs: not documented in any official source found (checked 2026-10-21).

## Critical Rules

- Keep RequestKey, ResponseKey and DD API client secret/password server-side only; never compute checksums in the browser.
- Verify the response checksum over the received string values exactly (do not reformat `rp_amount`); do not include `rp_xtraInfo`, `rp_remarks` or `rp_paymentMode` in it.
- Do not mark paid from the RedirectURL/ReturnURL alone; the spec calls browser redirection not 100% reliable.
- A checksum mismatch means "ignore and log", not "mark order failed" (the official plugin marks it failed, which lets a forged POST fail real orders).
- Check `rp_amount` and `rp_currency` against the stored order; the official plugin does not, so add it.
- Treat `RP09` as non-final and re-check with transaction enquiry; deferred updates also arrive at APIURL.
- Keep `orderId` unique per attempt (`RP45` = duplicate seller order number) and within 20 chars.
- APIURL and RedirectURL must not contain an IP address or port number (UAT sign-off rule).
- Use UAT keys with `ringgitpay.co` and production keys with `ringgitpay.com`; never mix.

## Completion Criteria

A RinggitPay integration is done only when:

1. Payment form and checksum are generated server-side from a stored pending order.
2. APIURL handler verifies `rp_checkSum` with ResponseKey, checks AppId, orderId, amount, currency, and returns HTTP 200.
3. Settlement on `RP00` is idempotent across APIURL, RedirectURL and enquiry results.
4. `RP09` orders are re-checked via transaction enquiry and later APIURL updates.
5. Tests cover bad checksum, wrong amount, wrong currency, unknown orderId, duplicate notification, `RP09` then `RP00`, failure codes, and enquiry with and without `transactionRef`.
6. UAT sign-off checklist items are captured (FPX SBI Bank A/B, card success/failure, receipt, redirect and API logs, enquiry log, email).
7. Deployment docs list `RINGGITPAY_APP_ID`, `RINGGITPAY_REQUEST_KEY`, `RINGGITPAY_RESPONSE_KEY`, `RINGGITPAY_ENV`, and the APIURL/RedirectURL registered with RinggitPay per environment.

## Loophole Review Loop

When asked whether a RinggitPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: checkout page, auto-submit form, RinggitPay payment page, FPX/card channel, RedirectURL/ReturnURL browser POST, APIURL server POST, enquiry API, database, fulfilment job, RinggitPay portal settings.
2. Check bypasses: forged browser POST, forged APIURL POST, checksum over reformatted amount, wrong key (RequestKey vs ResponseKey), case-sensitive hex compare, UAT/production key mix, wrong amount or currency, reused orderId, `RP09` treated as final or as failed, missing APIURL registration, ReturnURL on another domain, duplicate notifications, enquiry checksum without the empty `transactionRef` slot.
3. Add fixes: server-side checksum, constant-time case-insensitive compare, strict order matching, durable notification log, idempotent settlement, enquiry fallback, per-environment config.
4. Re-read `references/ringgitpay-api.md` sections relevant to any gap.
5. State remaining uncertainty: undocumented retry policy, no published IP allowlist, no refund API, and PDF versions newer than v1.13 (status codes "are regularly updated"). Ask RinggitPay for the latest spec.
