---
name: setup-antom
description: Set up, build, debug, review, and explain Antom payment integrations using official Antom docs, One-time Payments pay and createPaymentSession APIs, hosted and embedded Checkout Page, Payment Element, API-only redirects, RSA256 request signing and response or notification signature checks, notifyPayment, inquiryPayment, capture, cancel, refund, inquiryRefund, sandbox, test wallet, and Antom Dashboard keys. Use when working with Antom (Ant International, formerly Alipay+ merchant acquiring) for MYR, FPX online banking, cards, Touch 'n Go eWallet, Boost, GrabPay MY, ShopeePay MY, Alipay+ wallets, webhooks, refunds, or production readiness.
---

# Antom Payment Gateway

Use this skill for Antom (Ant International) merchant payment integration work in Malaysia.

## Source

Read `references/antom-api.md` first. It holds environments, domains, headers, the signature method, idempotency, amount rules, the full endpoint catalogue, and the index of the other reference files.

Read `references/antom-checkout.md` for pay, createPaymentSession, and consult field tables and the integration modes (Checkout Page, Element, API-only).

Read `references/antom-payment-ops.md` for inquiryPayment, capture, cancel, refund, and inquiryRefund fields and rules.

Read `references/antom-notifications.md` for notifyPayment, notifyCapture, notifyRefund fields, verification, the acknowledgement body, and retries.

Read `references/antom-errors.md` for result and error codes per API, and `references/antom-codes.md` for payment method enums, FPX bank codes, card brands, statuses, and Malaysian method limits.

Read `references/account-setup.md` for onboarding, keys, sandbox, go-live, settlement, and pricing location. Read `references/source-map.md` to pick the right official page.

## Core Workflow

1. Identify environment: sandbox (client ID starts with `SANDBOX_`, path `/ams/sandbox/api/...`) or live (`/ams/api/...`).
2. Identify integration mode: hosted Checkout Page, embedded Checkout Page, Payment Element, or API-only (`pay` with a `paymentMethodType`).
3. Confirm Dashboard setup: client ID, gateway domain, merchant key pair, Antom public key, notification URL, enabled payment methods, settlement currency.
4. Create a local pending order with your own `paymentRequestId` before calling Antom.
5. Sign every request with your private key (RSA256) and verify every response signature with the Antom public key.
6. Call `createPaymentSession` (Checkout Page or Element) or `pay` (API-only) server-side. Redirect to `normalUrl` / `applinkUrl` / `schemeUrl`, or pass `paymentSessionData` to the client SDK.
7. Treat `paymentRedirectUrl` as display only. Never mark paid from a browser return.
8. Verify the `notifyPayment` signature over the raw body, then reply with the fixed SUCCESS acknowledgement.
9. Settle idempotently after checking `paymentRequestId`, `paymentId`, amount, currency, and `result.resultStatus`. For cards, Apple Pay, and Google Pay, ship only after capture succeeds.
10. Fall back to `inquiryPayment` when a notification is late or missing.

## Core Facts

- All APIs are `POST` with JSON bodies, version `v1`, path `https://<domain>/ams/api/v1/...`.
- Asia domain: `https://open-sea-global.alipay.com` (also `https://open-sea.alipay.com`). Use the domain shown in your Dashboard.
- Sandbox online-payment path is `/ams/sandbox/api/<endpoint>`. The official SDKs rewrite `/ams/api` to `/ams/sandbox/api` when the client ID starts with `SANDBOX_`.
- Required request headers: `client-id`, `request-time`, `signature`. `Content-Type: application/json; charset=UTF-8` is optional.
- Sign `POST <http-uri>\n<client-id>.<request-time>.<request-body>` with SHA256withRSA, then base64, then URL-encode.
- Signature header: `algorithm=RSA256, keyVersion=1, signature=<urlencoded-signature>`.
- Verify responses with `<client-id>.<response-time>.<raw-response-body>`, and notifications with `<client-id>.<request-time>.<raw-notify-body>`.
- Amounts are strings in the smallest unit. MYR has 2 decimals, so RM13.14 is `"1314"`.
- One-time Payments use `productCode` `CASHIER_PAYMENT`. `productScene` is `CHECKOUT_PAYMENT` or `ELEMENT_PAYMENT` for sessions.
- Malaysian enums: `ONLINEBANKING_FPX`, `TNG`, `BOOST`, `GRABPAY_MY`, `SHOPEEPAY_MY`, `CARD`.
- Default payment timeout is 14 minutes. A payment session defaults to 1 hour.
- Notification acknowledgement body is fixed: `{"result":{"resultCode":"SUCCESS","resultStatus":"S","resultMessage":"success"}}`. Do not sign it.
- Notification retries run for up to 24 hours at 0s, 2m, 10m, 10m, 1h, 2h, 6h, 15h.
- A notification URL set in the API overrides the Dashboard value, except for capture notifications, where the Dashboard value wins.
- Manual capture must happen within 7 days of authorization. Only full capture is supported.
- Refunds are asynchronous. Card refunds never return `S` synchronously. FPX refund window is 180 days; TNG, Boost, GrabPay MY, ShopeePay MY are 365 days.
- Fees are not published in the docs. They are shown in Antom Dashboard > Payments > Payment methods > Price.

## Critical Rules

- Keep the merchant private key and client ID server-side. Use placeholders like `<CLIENT_ID>`, `<MERCHANT_PRIVATE_KEY>`, `<ANTOM_PUBLIC_KEY>`.
- Test and live use different keys, client IDs, and URLs. Never mix them.
- Verify signatures over the raw body bytes. Do not parse and re-serialize JSON first.
- Sign the exact `request-time` string you send in the header.
- `result.resultStatus` `U` is not a failure. Handle `PAYMENT_IN_PROCESS` by redirecting, other `U` by inquiry or a new `paymentRequestId` as the docs say.
- Never reuse a `paymentRequestId` with a changed amount, currency, or method (`REPEAT_REQ_INCONSISTENT`).
- Use the `paymentId` from `notifyPayment` for refunds. Card retries on Antom pages can create a new `paymentId`.
- Do not retry a refund that returned `REFUND_IN_PROCESS`. Query with `inquiryRefund`.
- Make notification handling idempotent. Record processed notifications and ignore duplicates.
- Confirm live method availability, acquirer entity, limits, settlement period, and pricing with Antom before claiming production certainty.

## Completion Criteria

An Antom integration is done only when:

1. Requests are signed server-side and every response signature is verified (or the unsigned F/U case is handled).
2. `paymentRequestId` is unique per attempt and stored with the expected amount and currency before calling Antom.
3. The notification endpoint verifies the signature over the raw body, settles idempotently, and returns the fixed acknowledgement.
4. Settlement checks `paymentRequestId`, `paymentId`, amount, currency, and status. Card flows wait for `CAPTURE_RESULT` `S`.
5. `inquiryPayment` fallback exists for missing or delayed notifications and for `U` results.
6. Refund and cancel flows handle `U`, `F`, and the asynchronous `notifyRefund`.
7. Tests cover bad signature, duplicate notification, wrong amount, `U` results, timeout (`USER_NOT_SUBMITTED`, `ORDER_NOT_EXIST`), and sandbox-to-live switch.
8. Deployment docs name `<CLIENT_ID>`, `<MERCHANT_PRIVATE_KEY>`, `<ANTOM_PUBLIC_KEY>`, gateway domain, notification URL, and redirect URL per environment.

## Loophole Review Loop

When asked whether an Antom strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, client SDK, backend signer, Antom API, notification receiver, raw-body parser, database, capture job, refund job, Dashboard settings.
2. Check bypasses: fake redirect success, forged notification, signature over parsed JSON, wrong public key, sandbox key on live, reused `paymentRequestId`, wrong amount or currency, shipping on authorization instead of capture, missed notification, duplicate notification, refund retried while in process, refund against a stale `paymentId`.
3. Add fixes: server-side signing and verification, raw-body checks, strict amount and ID matching, durable notification log, idempotent settlement, `inquiryPayment` polling fallback, capture-based shipping for cards, environment-separated config.
4. Re-read the relevant section of `references/antom-api.md` and the topic file for any gap.
5. State remaining uncertainty that depends on the merchant's contract: acquirer entity (AntomMY or other), enabled methods, settlement currency and period, pricing, and whether the sandbox path or the `/ams/api` path applies to the account.
