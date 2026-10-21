---
name: setup-commercepay
description: Set up, build, debug, review, and explain CommercePay (Commerce DotAsia Payments) payment integrations using official CommercePay docs, TokenAuth bearer tokens, Abp-TenantId header, cap-signature HMAC-SHA256 request signing, Direct Integration RequestPayment, Hosted Session Checkout InitialSession, signed host-to-host callbacks, Query Payment, refunds, saved payments and tokenisation, recurring payments, pre-auth ProcessPayment, payouts via bank account or DuitNow Proxy, payout wallets, channel and provider channel codes, and staging/production setup. Use when working with CommercePay Malaysia, FPX B2C/B2B, DuitNow Online Banking, DuitNow QR, cards, Apple Pay, Google Pay, TNG eWallet, GrabPay, ShopeePay, Boost, Alipay, Atome, callback verification, settlement, or production readiness.
---

# CommercePay Payment Gateway

Use this skill for CommercePay (payments.commerce.asia) payment gateway integration work.

## Source

Read `references/commercepay-api.md` before giving factual API guidance or writing integration code. It holds environments, auth, headers, the exact `cap-signature` algorithm with verified test vectors, the endpoint catalogue, request/response field tables, callback verification, status enums, refunds, saved payments, recurring, payouts, wallets, and error codes.

Read `references/commercepay-codes.md` when you need `channelId` values, FPX/DuitNow `providerChannelId` bank codes, payout bank codes, DuitNow Proxy codes, or staging payout test accounts.

Read `references/account-setup.md` when the user asks how to register, which credentials are issued, back-office location, staging vs production, env var names, recurring/payout enablement, plugins, settlement, or pricing.

Read `references/source-map.md` to pick the right official page for a topic.

## Core Workflow

1. Identify environment: staging (`https://staging-payments.commerce.asia/`) or production (`https://payments.commerce.asia/`).
2. Identify surface: Direct Integration (merchant renders channels), Hosted Session Checkout (CommercePay page), recurring, saved payment, pre-auth, refund, or payout.
3. Confirm the four credentials for that environment: Merchant Id, Username, Password, Secret Key.
4. Call `POST /api/TokenAuth/Authenticate` (header `Abp-TenantId`, no signature) and cache the `accessToken` for up to 24 hours.
5. Create a local pending payment with your unique `referenceCode` and expected amount in sen before calling CommercePay.
6. Sign every other request: lowercase(full URL + recursively key-sorted, minified, null-free camelCase JSON) then HMAC-SHA256 hex with the Secret Key into `cap-signature`.
7. Call `RequestPayment` (Direct) or `InitialSession` (Hosted) server-side with a `callbackUrl`; store `transactionNumber` or `sessionNumber`.
8. Send the browser to `result.redirectUrl` (or run `clientScript` when `redirectionType` is `2`).
9. Treat `returnUrl` as UX only. Verify the callback `cap-signature` (computed with YOUR `callbackUrl` as the prefix), answer HTTP 200, then confirm with `GET Query`.
10. Settle idempotently when status is `1` (Success) and `referenceCode`, `transactionNumber`, amount, currency and environment match.

## Core Facts

- Base URLs: staging `https://staging-payments.commerce.asia/`, production `https://payments.commerce.asia/`.
- Every API call except Authenticate needs `Content-Type: application/json`, `Authorization: Bearer <accessToken>`, `Abp-TenantId: <MERCHANT_ID>`, `cap-signature`.
- Token: `POST /api/TokenAuth/Authenticate` body `userNameOrEmailAddress`, `password`; returns `accessToken`, `expireInSeconds` 86400.
- Payment: `POST /api/services/app/PaymentGateway/RequestPayment`. Hosted session: `POST /api/services/app/PaymentGateway/InitialSession`.
- Status: `GET /api/services/app/PaymentGateway/Query?TransactionNumber=...&Timestamp=...` (or `SessionNumber`).
- Refund: `POST .../PaymentGateway/RefundPayment`; refund status: `GET .../PaymentGateway/QueryRefund`.
- Amounts are integers in minor units: `1000` = MYR 10.00.
- `timestamp` is Unix epoch milliseconds (`new Date().getTime()`), required on signed bodies.
- Responses use an ABP envelope: `{ result, targetUrl, success, error: { code, message }, unAuthorizedRequest, __abp }`.
- Callback is a POST to `callbackUrl` with `amount`, `channelId`, `currencyCode`, `referenceCode`, `status`, `transactionNumber`, `providerTransactionNumber` (+ `paymentSessionNumber` for sessions), signed in `cap-signature`.
- Payment status `0` Pending, `1` Success, `2` Failed, `3` Cancelled, `8` UserCancelled, `9` Authorized, `10` TransactionExpired, `11` Refunded (full table in api reference).
- No `callbackUrl` means no callback; you must poll Query Payment.
- Payout uses the same `RequestPayment` endpoint with payout `channelId` (staging `15`, production `9`) and a funded payout wallet.
- Channel IDs differ between staging and production (for example TNG is `10` in staging, `7` in production).
- Signature Validator Tool (staging back office): `https://staging-backoffice-payments.commerce.asia/app/paymentgateway/signaturevalidator`.

## Critical Rules

- Keep Password, Secret Key and access token server-side only.
- Do not mark paid from `returnUrl`, browser query strings, or frontend state.
- Verify callback `cap-signature` with constant-time comparison before trusting any field.
- Sign with raw UTF-8 (no `\uXXXX` escapes), only JSON-required escaping (`+ < > & '` stay literal), Unicode-aware lowercase.
- Omit null fields at every nesting level before signing; keep empty strings.
- For GET, sign the URL without the query string plus the query params as sorted JSON, numbers unquoted.
- Use the Secret Key, channel IDs, and callback URL of the same environment.
- Pass `providerChannelId` only when the channel has `isProviderHostChannel: true`; otherwise omit it.
- Keep `referenceCode` unique per payment attempt (max 50) and the callback handler idempotent.
- Refunds only on `Success` transactions; omit `amount` for full refund; `reason` is required; cross-border refunds are not supported.
- Recurring (Credit Card, TNG only) and payouts require Business Development approval first.

## Completion Criteria

A CommercePay integration is done only when:

1. Token is fetched server-side, cached, and refreshed on expiry or auth failure.
2. All signed requests pass in staging (use the Signature Validator Tool when debugging).
3. `RequestPayment` or `InitialSession` runs server-side with `callbackUrl`, and the returned id is stored before redirect.
4. Callback handler verifies `cap-signature`, returns HTTP 200, and requeries `Query` before settling.
5. Settlement checks `referenceCode`, `transactionNumber`/`paymentSessionNumber`, amount, currency, status `1`, and environment.
6. Duplicate callbacks, missing callbacks (Query polling fallback), and expired/failed statuses are handled idempotently.
7. Tests cover invalid signature, null-field and non-ASCII signing, duplicate callback, wrong amount/reference, and status `0`, `2`, `8`, `10`.
8. Deployment docs list `COMMERCEPAY_MERCHANT_ID`, `COMMERCEPAY_USERNAME`, `COMMERCEPAY_PASSWORD`, `COMMERCEPAY_SECRET_KEY`, `COMMERCEPAY_BASE_URL`, callback and return URLs per environment.

## Loophole Review Loop

When asked whether a CommercePay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser return, hosted session page, provider page, backend payment creator, token cache, CommercePay API, callback receiver, database, fulfilment, refund job, payout job, wallet balance, back office.
2. Check bypasses: forged return URL, forged callback, signature over the wrong URL (request path instead of your `callbackUrl`), escaped Unicode, nulls left in, unsorted nested keys, staging/production key or channel mix, expired token, reused `referenceCode`, wrong amount unit, session with several attempts, refund double-submit, payout to unregistered DuitNow proxy.
3. Add fixes: server-side creation, local pending record first, canonical signing helper with the published test vectors, constant-time compare, strict field matching, Query requery, durable event log, idempotency keys, per-environment config.
4. Re-read `references/commercepay-api.md` sections relevant to any gap.
5. State remaining uncertainty: callback retry schedule, IP allowlist, timestamp tolerance, return URL parameters, and rate limits are not documented in official docs (checked 2026-10-21).
