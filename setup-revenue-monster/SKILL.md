---
name: setup-revenue-monster
description: Set up, build, debug, review, and explain Revenue Monster payment integrations using official Revenue Monster docs and SDKs, OAuth 2.0 client credentials, RSA SHA256 request signatures (X-Signature, X-Nonce-Str, X-Timestamp), hosted Web Payment checkout (v3/payment/online), Direct Payment Checkout, notify URL callbacks, redirect handling, query transaction by order ID, refund, reverse, reconciliation, tokenization, recurring payments, sandbox/live setup, and Merchant Portal credentials. Use when working with Revenue Monster (RM) Malaysia, FPX, DuitNow QR, cards, Apple Pay, Google Pay, TNG, Boost, GrabPay, ShopeePay, Alipay+, WeChat Pay, checkout redirects, notify settlement, INVALID_REQUEST_SIGNATURE errors, refunds, or production readiness.
---

# Revenue Monster Payment Gateway

Use this skill for Revenue Monster (RM) Open API payment integration work.

## Source

Read `references/revenue-monster-api.md` before giving factual API guidance or writing integration code. It contains environments, OAuth token flow, the RSA signature algorithm with exact string-to-sign, hosted checkout fields, redirect and notify payloads, query, refund, reverse, reconciliation, methods, statuses, and error codes.

It also holds the Endpoint Catalogue (every documented operation) and points to the detail files:

- `references/revenue-monster-objects.md`: Transaction Object, nested objects, type/status enums, method and region matrix, FPX B2C/B2B bank codes.
- `references/revenue-monster-tokenization.md`: recurring, tokenized customer, charge bound card, card-on-file tokens.
- `references/revenue-monster-instore.md`: Quick Pay, terminal events, POS, Android deeplink.
- `references/revenue-monster-invoice.md`: invoice / payment link API (`/api/v3/...`).
- `references/revenue-monster-errors.md`: every documented error code.

Read `references/account-setup.md` when the user asks how to register, where Client ID / Client Secret / Store ID / RSA keys live, sandbox activation, payment method activation, env var names, pricing, or plugins.

Read `references/source-map.md` when choosing which official RM docs page or SDK file applies.

## Core Workflow

1. Identify environment: sandbox (`sb-oauth`, `sb-open`) or production (`oauth`, `open`).
2. Identify integration surface: hosted Web Payment checkout, Mobile Payment, Direct Payment Checkout (own UI, QR, FPX bank, card native), Quick Pay (POS scan), tokenization/recurring, low-code button, or plugin.
3. Confirm Merchant Portal setup: application Client ID and Client Secret, your RSA public key uploaded, RM server public key saved, Store ID, and activated payment methods.
4. Get an access token server-side with `POST /v1/token` (Basic auth of `clientId:clientSecret`, `grantType: client_credentials`); cache it and refresh before expiry.
5. Create a local pending payment with a unique `order.id` (max 24 chars) and amount in sen before calling RM.
6. Call `POST /v3/payment/online` with Bearer token plus `X-Signature`, `X-Nonce-Str`, `X-Timestamp`; redirect the customer to `item.url`.
7. Treat `redirectUrl` (`?status=&orderId=`) as browser UX only.
8. On notify (`eventType: PAYMENT_WEB_ONLINE`), record the payload, then requery `GET /v3/payment/transaction/order/{orderId}` with a signed request.
9. Settle idempotently only when the queried transaction is `SUCCESS` and `order.id`, `order.amount`, `currencyType`, store, and environment match.
10. Run a requery job for pending orders, because notify fires only on success and may be missed.

## Core Facts

- OAuth: `https://oauth.revenuemonster.my/v1/token`; sandbox `https://sb-oauth.revenuemonster.my/v1/token`.
- Open API: `https://open.revenuemonster.my`; sandbox `https://sb-open.revenuemonster.my`. Payment APIs are under `/v3`.
- Hosted checkout: `POST /v3/payment/online`, `type` `WEB_PAYMENT` or `MOBILE_PAYMENT`, `layoutVersion` `v4`.
- Amounts are integers in sen (`100` = RM 1.00); minimum RM 0.10; currency `MYR` only.
- Signature: RSA PKCS#1 v1.5 with SHA-256 over `data=<b64>&method=<lower>&nonceStr=<n>&requestUrl=<url>&signType=sha256&timestamp=<unix>`, where `<b64>` is Base64 of key-sorted, compact JSON with `<`, `>`, `&` escaped as `\u003c`, `\u003e`, `\u0026`.
- Header format: `X-Signature: sha256 <base64signature>`. Nonce must be unique for 120 seconds; timestamp is UTC Unix seconds within 120 seconds of server time.
- Empty body (GET): omit the `data=` pair. Verifying RM's callback: `requestUrl` can be skipped; verify with the RM server public key.
- Redirect statuses: `SUCCESS`, `FAILED`, `CANCELLED`, `EXPIRED`.
- Notify is sent only on successful payment; failures and refunds do not trigger notify.
- Transaction statuses: `SUCCESS`, `FAILED`, `IN_PROCESS`, `FULL_REFUNDED`, `PARTIAL_REFUNDED`, `REVERSED`.
- Refund: `POST /v3/payment/refund` with `transactionId`, `refund.type` `FULL`, `refund.currencyType`, `refund.amount`, `reason`.
- Reverse (timeouts, about 15 minutes): `POST /v3/payment/reverse` with `orderId`.
- Credentials live in Merchant Portal > Developer > Applications.

## Critical Rules

- Keep Client Secret, access/refresh tokens, and your RSA private key server-side only.
- Never mark paid from `redirectUrl` query params or the notify body alone; settle from a signed server-side query.
- Sign the exact `requestUrl` you call, including any query string, and lowercase the HTTP method.
- Sort JSON keys recursively and compact before Base64; a mismatch returns `INVALID_REQUEST_SIGNATURE` (sandbox returns a `debug` object showing each step).
- Generate a fresh 32-character nonce and fresh timestamp per request; reused nonces cause duplicate request errors.
- Use sandbox keys, tokens, Store ID, and server public key only with sandbox hosts; never mix environments.
- Keep `order.id` unique per attempt (`ORDER_ID_DUPLICATE` otherwise) and store `checkoutId` and `transactionId`.
- If a payment call times out, reverse by `orderId` before retrying with a new order to avoid double charge.
- Make notify handling and refunds idempotent; check `balanceAmount` before refunding.
- Verify live method activation (for example `BOOST_MALAYSIA_NOT_ACTIVE`), pricing, and settlement in the portal before claiming production certainty.

## Completion Criteria

A Revenue Monster integration is done only when:

1. Token fetch, caching, and refresh run server-side with Basic auth of Client ID and Secret.
2. Every Open API call carries Bearer token, `X-Signature`, `X-Nonce-Str`, `X-Timestamp`, and passes in sandbox.
3. Hosted checkout is created server-side with amount in sen and a unique `order.id`; the customer goes to `item.url`.
4. Redirect handler shows status only and triggers a requery; it does not fulfil.
5. Notify handler records the payload, requeries the transaction by order ID, and settles only on `SUCCESS` with matching amount, currency, order ID, and store.
6. Duplicate notify, missing notify, failed/expired/cancelled checkouts, and refunds are idempotent and tested.
7. Tests cover signature string building (sorted JSON, escapes, empty body), wrong amount/order, duplicate notify, and timeout reverse.
8. Deployment docs list `RM_CLIENT_ID`, `RM_CLIENT_SECRET`, `RM_PRIVATE_KEY`, `RM_SERVER_PUBLIC_KEY`, `RM_STORE_ID`, env, redirect URL, and notify URL.

## Loophole Review Loop

When asked whether a Revenue Monster strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, checkout page, backend checkout creator, OAuth server, Open API, notify receiver, signature builder, key storage, database, fulfilment job, refund job, Merchant Portal settings, and method activation.
2. Check bypasses: fake redirect `status=SUCCESS`, forged notify body, notify missing (fires on success only), signature over unsorted or unescaped JSON, wrong `requestUrl`, uppercase method, stale timestamp, reused nonce, sandbox/live key mix, expired access token, wrong amount (ringgit vs sen), reused order ID, timeout double charge, duplicate refund, refund outside provider window, and inactive method.
3. Add fixes: server-side checkout, local pending payment first, signed requery before settlement, strict amount/currency/order/store checks, durable notify log, idempotency keys, token cache with refresh, reverse on timeout, separate environment config, and portal capability checks.
4. Re-read `references/revenue-monster-api.md` sections relevant to any gap.
5. State remaining uncertainty if it depends on notify header names, method activation, refund windows, pricing, settlement timing, or merchant approval.
