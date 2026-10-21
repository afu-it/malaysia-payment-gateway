---
name: setup-eghl
description: Set up, build, debug, review, and explain eGHL (now ADAPTIS by NTT DATA Payment Services) payment integrations using official eGHL/ADAPTIS sources, the Payment.aspx hosted payment page, SALE/AUTH requests, SHA-256 HashValue and HashValue2 checks, MerchantReturnURL and MerchantCallBackURL handling, the "OK" callback acknowledgement, QUERY status requery, REFUND, RSALE reversal, CAPTURE, test vs live URLs, Merchant Service ID and Merchant Password setup, and official plugins. Use when working with eGHL, GHL ePayments, ADAPTIS, NTT DATA ePayments Malaysia, FPX/direct debit, cards, e-wallets, PymtMethod ANY/CC/DD/WA, TxnStatus codes, callback settlement, refunds, or production readiness.
---

# eGHL / ADAPTIS Payment Gateway

Use this skill for eGHL payment gateway integration work. eGHL is now branded ADAPTIS by NTT DATA Payment Services; the API, hosts, and field names are still the eGHL ones.

## Source

Read `references/eghl-api.md` before giving factual API guidance or writing integration code. It contains the endpoint catalogue, payment URLs, request and response field tables, the exact hash string orders, return vs callback handling, Query/Refund/Reversal/Capture/Settle/SOPBL, amount rules, and gotchas.

Read `references/eghl-codes.md` for enums: TransactionType, PymtMethod, TxnStatus, TokenType, currencies, data types, acknowledgement bodies, and validator error messages.

Read `references/account-setup.md` when the user asks how to register, which credentials are needed, where they come from, test vs live, the ADAPTIS rebrand, plugins, pricing, or downtime notices.

Read `references/source-map.md` when choosing which official source (Bitbucket SDK, plugin, guide PDF, NTT DATA page) backs a fact.

The full eGHL Merchant API specification is distributed to merchants and is not published openly. Facts here come from eGHL's official public Bitbucket SDKs and plugins. When a detail is not in those sources, say so and tell the user to confirm with their account manager.

## Core Workflow

1. Identify environment: test (`pay.e-ghl.com`) or live (`securepay.e-ghl.com`).
2. Confirm credentials: Merchant Service ID (`ServiceID`, 3 chars) and Merchant Password, issued by eGHL/ADAPTIS.
3. Create a local pending payment with a unique `PaymentID` (max 20 chars) and your `OrderNumber`.
4. Build the SALE request server-side and compute `HashValue` = SHA-256 over password + the SALE fields in the documented order.
5. Send the browser to `Payment.aspx` with an auto-submitting HTML form (POST).
6. Handle `MerchantCallBackURL` (server to server): verify `HashValue2`, settle idempotently, reply with the plain text `OK`.
7. Handle `MerchantReturnURL` (browser): verify `HashValue2`, show the result; settle only if the callback has not already done so, using the same idempotent code.
8. For `TxnStatus=2` (pending) or a missing callback, requery with `TransactionType=QUERY` and verify the response `HashValue`.
9. Refund (cards only), reverse (`RSALE`), or capture (`AUTH` then `CAPTURE`) through the same `Payment.aspx` endpoint with a signed server-side POST.

## Core Facts

- Live payment URL: `https://securepay.e-ghl.com/IPG/Payment.aspx`.
- Test payment URL: `https://pay.e-ghl.com/IPGSG/payment.aspx`.
- Query, Refund, Reversal, and Capture are server-side form POSTs to the same URL; the response is a URL-encoded string.
- Hash algorithm: lowercase hex `sha256(MerchantPassword + field1 + field2 + ...)`, no separators, missing optional fields contribute an empty string.
- SALE `HashValue` order: `ServiceID, PaymentID, MerchantReturnURL, MerchantApprovalURL, MerchantUnApprovalURL, MerchantCallBackURL, Amount, CurrencyCode, CustIP, PageTimeout, CardNo, Token`.
- Payment response `HashValue2` order: `TxnID, ServiceID, PaymentID, TxnStatus, Amount, CurrencyCode, AuthCode, OrderNumber, Param6, Param7`.
- Query/Refund/Reversal/Capture request hash order: `ServiceID, PaymentID, Amount, CurrencyCode`.
- API response `HashValue` order: `TxnID, ServiceID, PaymentID, TxnStatus, Amount, CurrencyCode, AuthCode`.
- `TxnStatus`: `0` success, `1` failed, `2` pending; Query adds `9` reversed, `10` refunded, `15` authorised, `16` captured, `31` reversal pending, `-1` not found, `-2` system error.
- `TxnStatus=1` with `TxnMessage` `Buyer cancelled` means the shopper cancelled.
- Amount is ringgit (major unit) with exactly 2 decimals and no commas, for example `10.00`.
- URLs inside request fields use `;` instead of `&` in query strings.
- Callback must answer with the body `OK`.
- `PymtMethod` values used in official plugins: `ANY`, `CC` (card), `DD` (direct debit / online banking), `WA` (e-wallet).
- Online refund is supported only for `PymtMethod=CC`; partial refund allowed up to the original amount.

## Critical Rules

- Keep the Merchant Password server-side only; never compute hashes in the browser.
- Do not mark paid from the return URL alone without verifying `HashValue2`, and never trust `TxnStatus` from an unverified request.
- Compare hashes in constant time, case-insensitively (official plugins use case-insensitive compare).
- Before settling, check `ServiceID`, `PaymentID`, `OrderNumber`, `Amount`, `CurrencyCode`, and `TransactionType` against your stored pending payment.
- Make settlement idempotent: callback and return can both arrive, and callbacks can repeat. A late `failed` must not overwrite a paid order.
- Use a fresh unique `PaymentID` per attempt. Official plugins resend the same `OrderNumber` on retries of one order, but keep `OrderNumber` unique per order across every store sharing a Service ID (WooCommerce: "ADAPTIS will not allow orders with the same invoice number").
- Load the Merchant Password from server config only. The official PrestaShop 1.6 handler accepts a password from the request; do not copy it.
- Reject a response whose `CurrencyCode` differs from the order unless your account uses shopper currency choice (then see Multi-Currency in `references/eghl-api.md`).
- Use the password and URL for the same environment; test and live credentials are different.
- Treat `pending` (`2`) as unresolved; requery later instead of failing the order.
- Do not invent other endpoints (REST, JSON, webhooks with headers). None are in the official public sources. Do not build `SETTLE`, `SOP`, `SOP2`, or `SOPCARD` without the merchant API spec; only partial details are public.

## Completion Criteria

An eGHL integration is done only when:

1. SALE request and `HashValue` are built server-side with correct field order, 2-decimal amount, and `;`-escaped URLs.
2. Callback handler verifies `HashValue2`, settles idempotently, and responds `OK`.
3. Return handler verifies `HashValue2` and shares the same settlement code.
4. Amount, currency, `PaymentID`, `OrderNumber`, and `ServiceID` match the stored payment before marking paid.
5. Pending and missing-callback cases use QUERY with response `HashValue` verification.
6. Tests cover bad hash, duplicate callback, wrong amount/currency, buyer cancelled, pending then success, and success.
7. Deployment docs list `EGHL_SERVICE_ID`, `EGHL_MERCHANT_PASSWORD`, `EGHL_PAYMENT_URL`, return URL, callback URL, and test/live separation.

## Loophole Review Loop

When asked whether an eGHL strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser form post, return URL, callback URL, hash builder, Payment.aspx, query job, refund job, database, fulfilment.
2. Check bypasses: forged return post, hash over wrong field order, unescaped `&` in URLs, amount formatting mismatch (`10` vs `10.00`), test/live password mix, reused `PaymentID`, duplicate callback, late failed after success, pending treated as failed, refund on non-card method, reversal after bank settlement.
3. Add fixes: server-side hashing, strict field matching, idempotent state machine, QUERY fallback with hash check, environment-separated config, `OK` acknowledgement only after durable write.
4. Re-read `references/eghl-api.md` sections relevant to any gap.
5. State remaining uncertainty: anything only in the private API spec (callback retry schedule, IP allowlist, full error code list, recurring/tokenisation details, sandbox test cards) must be confirmed with eGHL/ADAPTIS support.
