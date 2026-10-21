---
name: setup-boldpay
description: Set up, build, debug, review, and explain BOLD.Pay (by MACROKIOSK) payment integrations using official BOLD.Pay sources (the MACROKIOSK WooCommerce plugin source and product pages), hosted PaymentRequest redirect, MD5 AccessToken, CallbackURL browser return, NotificationURL server POST, QueryRequest status requery, PayID, CustIP, currency codes, error codes 400 to 500, BOLD.Pass portal API key and Domain/IP Restrictions, sandbox (UAT) vs live setup, and the WooCommerce plugin. Use when working with BOLD.Pay Malaysia, MACROKIOSK payments, boldpass.com, pay.etracker.cc, FPX online banking, cards, e-wallets, BNPL, OTC, carrier billing, WooCommerce or OpenCart BOLD.Pay setup, callback settlement, or production readiness.
---

# BOLD.Pay Payment Gateway

Use this skill for BOLD.Pay (MACROKIOSK) payment integration work.

## Source

Read `references/boldpay-api.md` before giving factual API guidance or writing integration code. BOLD.Pay has no public API reference; that file is distilled from the official MACROKIOSK WordPress.org plugin source (v1.7.1) and lists the endpoint catalogue, every request and response field, the AccessToken algorithm, requery, status and error codes, and what is not documented.

Read `references/account-setup.md` when the user asks how to register, where the username and API key live, Domain/IP Restrictions, WooCommerce plugin setup, sandbox access, channels, pricing or settlement.

Read `references/source-map.md` when choosing which source file backs a fact, or to see coverage and gaps.

## Core Workflow

1. Identify environment: sandbox (`uat.pay.etracker.cc`) or live (`pay.etracker.cc`).
2. Identify surface: WooCommerce plugin (use it as is), OpenCart/EasyStore plugin, or custom integration that mirrors the plugin protocol.
3. Confirm portal setup: approved BOLD.Pass username, API key, and your domain and server IP in Domain/IP Restrictions.
4. Create a local pending payment first with expected amount (2 decimals, ringgit), a new unique `PayID`, the `CustIP` you will send, and the callback and notification URLs used.
5. Compute `AccessToken` server-side: `UPPER(MD5(UPPER(Username + "4" + Amount + CustIP + PayID + CallbackURL + NotificationURL + ApiKey)))`.
6. Redirect the browser to `PaymentRequest?` with all fields (URLs and text URL-encoded).
7. On browser return or notification, recompute the token from stored values and compare (constant-time).
8. Call `QueryRequest` server-side and read the result from the redirect `Location` query string.
9. Check query `TransactionID`, `OrderNumber`, `PayID` match, then settle idempotently on query `Status` = `SUCCESS`.
10. Reply `OK` to the notification POST.

## Core Facts

- Live payment page: `https://pay.etracker.cc/BoldPayApi/PaymentRequest`.
- Sandbox payment page: `https://uat.pay.etracker.cc/BoldPayApi/PaymentRequest`.
- Status query: `https://pay.etracker.cc/BoldPayApi/QueryRequest` (sandbox host `uat.pay.etracker.cc`; plugin uses `http://` there, HTTPS also answers).
- Create payment is a browser GET redirect with a query string, not a JSON API.
- Request fields: `AccessToken`, `Username`, `CustIP`, `PymtMethod` (`4`), `Amount`, `Description`, `CustName`, `CustEmail`, `CustPhone`, `OrderNumber`, `PayID`, `CallbackURL`, `NotificationURL`, `ApiKey`, `Origin`, `CurrencyCode`.
- `CurrencyCode`: `1` MYR, `3` THB.
- Amount is ringgit with exactly 2 decimals, not sen.
- AccessToken is uppercase hex MD5 of the uppercased concatenation; URLs are raw (not encoded) inside the hash.
- Response fields: `AccessToken`, `TransactionID`, `OrderNumber`, `PayID`, `Status`, `Reason`, `PymtMethod`, `IssuingBank`.
- Status values: `SUCCESS`, `PENDING`, `FAIL`, `REFUNDED`, `REVERSED`; rejected requests return a numeric error code instead.
- Channel codes: `CC`, `DD` (online banking), `WA` (e-wallet), `BNPL`, `OTC`.
- QueryRequest params: `AccessToken`, `APIKey`, `Username`, `PayID`, `CallBackUrl` (note casing).
- Notification is a server POST; reply body `OK`.
- API key lives in BOLD.Pass > Admin Management > Business Setting > API Key.
- No refund, recurring, instalment-field or payout API is published.

## Critical Rules

- The response `AccessToken` is the request token echoed back. It does not sign `Status` and it was visible in the browser URL. Never settle from callback fields alone; always settle from `QueryRequest`.
- Recompute the token from stored expected values (amount, `CustIP`, URLs), never from values in the callback.
- Keep TLS verification on for `QueryRequest` (the plugin turns it off; do not copy that).
- The API key appears in the payment URL by design. Lock it down with Domain/IP Restrictions and rotate it if leaked.
- Generate a fresh `PayID` per attempt; reuse returns 416.
- Send `CustPhone` as digits only.
- Use one stable `CustIP` per payment and store it; the token breaks if it changes between request and callback.
- Make settlement idempotent: browser return and notification both arrive for the same payment.
- Never move a paid order back on a late `PENDING` or `FAIL`.
- Do not quote fees, limits, or settlement times; they are per merchant agreement.

## Completion Criteria

A BOLD.Pay integration is done only when:

1. The token is built server-side and matches the documented order; a known-good sandbox payment succeeds.
2. Local pending record stores `PayID`, amount, currency code, `CustIP`, callback and notification URLs.
3. Callback and notification handlers recompute and compare the token in constant time.
4. Every state change is confirmed by `QueryRequest`, with `TransactionID`, `OrderNumber`, `PayID` matched.
5. Duplicate return plus notification settle once.
6. Error-code returns (no `AccessToken`) leave the order unpaid and are logged.
7. Tests cover bad token, wrong amount, query mismatch, `FAIL`, `PENDING`, `REFUNDED`, duplicate notification and `SUCCESS`.
8. Deployment docs list `BOLDPAY_USERNAME`, `BOLDPAY_API_KEY`, `BOLDPAY_ENV`, callback and notification URLs, server IP, and the Domain/IP Restrictions step.

## Loophole Review Loop

When asked whether a BOLD.Pay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect URL, BOLD.Pay hosted page, callback return, notification receiver, QueryRequest call, database, fulfilment job, BOLD.Pass portal settings.
2. Check bypasses: forged callback with a copied token, `Status=SUCCESS` without query, query skipped on `FAIL`, amount changed in the URL, reused `PayID`, changed callback URL breaking in-flight tokens, sandbox/live mix, TLS off on query, leaked API key, duplicate notification, late `PENDING` after `SUCCESS`, refund done in portal not reflected locally.
3. Add fixes: stored-value token recompute, mandatory QueryRequest, strict field matching, unique `PayID`, per-payment stored URLs and IP, idempotent settlement, Domain/IP Restrictions, TLS on, refund/reversal handling.
4. Re-read `references/boldpay-api.md` sections relevant to any gap.
5. State remaining uncertainty: no public API guide, `Origin` value for custom integrations, `PymtMethod` meaning, notification retry policy, sandbox credentials, limits, refund process. Tell the user to confirm these with MACROKIOSK.
