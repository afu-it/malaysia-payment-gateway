---
name: setup-tng-ewallet
description: Set up, build, debug, review, and explain Touch 'n Go eWallet (TNG Digital, TNGD) payment integrations using official TNGD Mini Program Open API docs and the TNGD WooCommerce plugin source, covering /v1/payments/pay cashier and agreement payments, my.tradePay, notifyPayment callbacks, inquiryPayment, refund and inquiryRefund, applyToken/prepare/cancelToken auto debit, RSA256 request and response signatures, Alipay+ style web acquiring (order create/query/refund), sandbox vs production credentials, SFTP settlement files, and partner onboarding. Use when working with Touch 'n Go eWallet, TNG eWallet, TNGD mini programs, TNG cashier redirect, MYR e-wallet checkout, auto debit, payment notification handling, refunds, reconciliation, or production readiness.
---

# Touch 'n Go eWallet (TNG Digital)

Use this skill for direct Touch 'n Go eWallet merchant integrations with TNG Digital Sdn Bhd (TNGD).

## Source

Read `references/tng-ewallet-api.md` before giving factual API guidance or writing integration code. It holds the Mini Program Open API: message structure, RSA256 signing and verification, endpoint catalogue, full field tables for pay, inquiryPayment, notifyPayment, refund, inquiryRefund, authorization and user info APIs, data dictionary, result codes, my.tradePay and my.getAuthCode, and the SFTP settlement file.

Read `references/tng-ewallet-web-acquiring.md` when the merchant wants a website checkout (no mini program). It documents the `alipayplus.acquiring.*` API as implemented by the TNGD WooCommerce plugin, the only public source for that flow, and lists its gaps.

Read `references/account-setup.md` for onboarding, credentials, sandbox vs production, env vars, settlement, and pricing links.

Read `references/source-map.md` to pick the right official page and to see coverage and what is partner-only.

## Core Workflow

1. Identify the surface: TNGD Mini Program (Open API + `my.tradePay`), website or app checkout (web acquiring API), auto debit (agreement pay), or no-code (Payment Link, QR). No-code needs no API work.
2. Confirm onboarding is done. TNGD issues Client-Id, Partner ID (Merchant ID), product code, TNGD public key, API host, and SFTP access by email after sandbox UAT. There is no self-serve API key page.
3. Generate an RSA key pair; give TNGD the public key; keep the private key server-side.
4. Create a local pending payment with a unique `paymentRequestId` (mini program) or `merchantTransId` (web) before calling TNGD.
5. Call the create API server-side with a signed request. Verify the response signature with the TNGD public key.
6. Mini program: pass `actionForm.redirectionUrl` to `my.tradePay({ paymentUrl })`. Web: redirect the browser to `checkoutUrl`.
7. Treat `my.tradePay` result codes and the browser return URL as UX only.
8. Handle the payment notification: verify the signature, check IDs and amount, acknowledge with a signed `resultStatus: "S"` response at once, then process async and idempotently.
9. If no final notification arrives, query status (`inquiryPayment` or `order.query`). Never settle from `U` or `PROCESSING`.
10. Reconcile daily against the T+1 SFTP settlement CSV.

## Core Facts

- Open API URL pattern: `https://{host}/acl/api/{restful_path}`, for example `/v1/payments/pay`. TNGD assigns the host; production endpoints are sent by email.
- Every Open API call is `POST` with JSON body and headers `Client-Id`, `Request-Time` (RFC3339, millisecond precision), `Signature`.
- Signature header: `Signature: algorithm=RSA256, keyVersion=<n>, signature=<value>`. RSA256 (SHA256withRSA) is default; ECC224 is also listed.
- String to sign: `POST <HTTP_URI>\n<Client-Id>.<Request-Time>.<HTTP_BODY>`. Responses are verified with `<Client-Id>.<Response-Time>.<HTTP_BODY>` and the TNGD public key.
- Signing is mandatory for all requests and responses, including your acknowledgement of `notifyPayment`.
- Amounts are `{currency, value}` with `value` a string integer in the smallest unit (sen). RM100.00 is `"10000"`.
- Cashier payment: `productCode` `51051000101000100001`, `paymentFactor.isCashierPayment: true`. Agreement pay: `productCode` `51051000101000100031`, `paymentFactor.isAgreementPay: true`, `paymentAuthCode` = access token.
- `resultStatus`: `S` success, `F` failed, `U` unknown, `A` accepted (follow `actionForm`).
- `inquiryPayment` `paymentStatus`: `SUCCESS`, `FAIL`, `PROCESSING`, `CANCELLED`. Poll every 5 s for up to 1 minute.
- `my.tradePay` result codes: `9000` success, `8000` processing, `4000` failed, `6001` user cancel, `6002` network error, `6004` unknown.
- Mini program calls need `extendInfo` with `customerBelongsTo` (for example `"{\"customerBelongsTo\":\"TNG\"}"`).
- Idempotency keys: `partnerId` + `paymentRequestId` for pay, `partnerId` + `refundRequestId` for refund.
- Settlement is T+1 to the bank account given at onboarding; CSV `MERCHANT_SETTLEMENT_<PartnerID>_<yyyymmdd>` is on SFTP at 9 AM T+1.
- Web acquiring sandbox base (plugin default): `https://api-sd.tngdigital.com.my`; paths `/alipayplus/acquiring/order/create.htm`, `/order/query.htm`, `/order/refund.htm`, `/refund/query.htm`.

## Critical Rules

- Keep the merchant private key, client secret, and SFTP password server-side only.
- Sign the exact bytes you send. Serialize the body once and send that same string.
- Verify every TNGD response and notification signature with the TNGD public key for the same environment before trusting it.
- Never mark paid from `my.tradePay` `9000`, `paymentReturnUrl`, or the web `PAY_RETURN` redirect alone.
- A `U` result is not a failure. Do not fail, refund offline, or re-create the payment; query or wait for the notification.
- `notifyPayment` acknowledgement must return `S` as soon as the message is received; do the business work asynchronously. Non-`S` triggers retries and duplicates.
- Settle only when `paymentId`/`acquirementId`, your request ID, amount, currency `MYR`, and partner ID all match the stored pending payment.
- Partial refunds are allowed; total refunded must not exceed the payment. Use a new `refundRequestId` per refund and keep it for inquiry.
- Do not invent the production host, notify URL registration, or notification headers; ask TNGD where docs are silent.
- The WooCommerce plugin's notification handler does not verify signatures. Do not copy that gap.

## Completion Criteria

A TNG eWallet integration is done only when:

1. Create, query, and refund calls run server-side with correct `Client-Id`, `Request-Time`, and RSA256 `Signature`.
2. Response signatures are verified with the TNGD public key; failures are rejected and logged.
3. The notification endpoint verifies the signature, acks with a signed `S` response, and processes idempotently.
4. Amount and currency are checked in sen against the stored order before marking paid.
5. A status inquiry job covers missed notifications and never settles `U`/`PROCESSING`.
6. Refund flow stores `refundRequestId`, handles `U` with `inquiryRefund`, and blocks over-refund.
7. Tests cover bad signature, duplicate notification, wrong amount, `U` result, expired order, and success.
8. Config documents `TNG_CLIENT_ID`, `TNG_PARTNER_ID`, `TNG_PRIVATE_KEY`, `TNG_PUBLIC_KEY`, API host, notify URL, and sandbox/live separation.

## Loophole Review Loop

When asked whether a TNG eWallet strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: mini program or browser, `my.tradePay` result, return URL, merchant backend, TNGD Open API, notification receiver, key store, database, refund job, SFTP settlement.
2. Check bypasses: forged notification, unsigned or wrongly signed body, body re-serialized after signing, wrong environment key, sandbox/live mix, wrong amount unit (ringgit vs sen), reused `paymentRequestId` with different data (`REPEAT_REQ_INCONSISTENT`), `U` treated as failure, duplicate notification, notification acked only after processing, expired order paid late, over-refund, refund `U` refunded offline.
3. Add fixes: server-side signing, exact-byte signing, response and notification verification, strict ID/amount/currency checks, durable event log, idempotent settlement, inquiry fallback, daily CSV reconciliation.
4. Re-read the relevant sections of `references/tng-ewallet-api.md` or `references/tng-ewallet-web-acquiring.md`.
5. State remaining uncertainty: production host, notify URL setup for web acquiring, exact notification headers, `customerBelongsTo` values, key version, rate limits, and anything TNGD only shares in its partner Postman pack.
