---
name: setup-paycools
description: Set up, build, debug, review, and explain PayCools payment integrations using official PayCools Malaysia Open API docs, WebPay hosted checkout, DuitNow dynamic QR, payment notifications, RSA SHA256withRSA signatures over the param string, appId and RSA key exchange, order status requery, refunds, refund notifications, payouts/disbursement, account validation, balance inquiry, channel codes, and response codes. Use when working with PayCools (Micro Bluetech Sdn Bhd), FPX, DuitNow QR, DuitNow RPP online banking, Touch 'n Go, Boost, GrabPay, MCash, cards, payouts to Malaysian banks or e-wallets, callback verification, idempotent settlement, or production readiness.
---

# PayCools Payment Gateway

Use this skill for PayCools (Malaysia) Open API integration work.

## Source

Read `references/paycools-api.md` before giving factual API guidance or writing integration code. It contains the endpoint catalogue, the request envelope, the RSA signature method (verified against the docs' own example), every request/response/notification field table, channel codes, status enums, response codes, and documented gaps and inconsistencies.

Read `references/account-setup.md` when the user asks how to become a merchant, what credentials exist, how the RSA key exchange works, env var names, sandbox/UAT, channel activation, pricing, or settlement.

Read `references/source-map.md` when choosing which official PayCools docs page applies, or to check coverage.

## Core Workflow

1. Get from PayCools: `appId`, the API base URL (not published in the docs), UAT/sandbox details if any, and the PayCools platform RSA public key.
2. Generate a 4096-bit RSA key pair; send the public key to PayCools; keep the private key server-side.
3. Create a local pending payment with a unique alphanumeric `mchOrderId` (max 32) and the expected amount in cents.
4. Build the business `param` object, serialize it to a JSON string once, sign that exact string with SHA256withRSA, and POST `{appId, sign, param}` as JSON.
5. WebPay: call `POST /open-api/webpay/generate` with a `channelCode`, redirect the customer to `data.payUrl`. DuitNow QR: call `POST /open-api/qr/generate` and render `data.qrCodeContent`.
6. Treat `redirectUrl` as browser UX only.
7. On the merchant `notifyUrl`, verify `sign` over the received `param` string with the PayCools public key before parsing `param`.
8. Settle idempotently only when `transactionStatus` is `COMPLETED` and `mchOrderId`, `transactionId`, `amount`, and `channelCode` match the local record. Reply `{"code":10000,"message":"Success"}`.
9. Requery with `POST /open-api/webpay/detail` or `POST /open-api/qr/detail` when a notification is missing or the order is still pending.
10. Refunds use `POST /open-api/payment/refund` with the PayCools `originalTransactionId`; settle refunds from the verified refund notification or `POST /open-api/refund/payments`.

## Core Facts

- Every merchant-to-PayCools call is `POST`, `Content-Type: application/json`, body `{"appId","sign","param"}`.
- `param` is a JSON-serialized string; `sign` is Base64 of SHA256withRSA (RSASSA-PKCS1-v1_5) over the exact `param` bytes, signed with the merchant private key (PKCS#8).
- PayCools-to-merchant notifications are `{"sign","param"}`, signed with the PayCools private key; verify with the PayCools public key (X.509 SubjectPublicKeyInfo), which the merchant must request from PayCools.
- RSA key length must be 4096.
- Response envelope: `{code, message, data}`; `10000` means success. Codes `21000` to `21016` are errors (sign, param, appId, duplicate order id, timestamp, IP forbidden, and others).
- Amounts are integers in cents (sen) for WebPay, QR, refund, and payout.
- Every `param` carries `timestamp` (current time, documented as milliseconds on several pages).
- WebPay create requires `timestamp`, `channelCode`, `mchOrderId`, `amount`, `notifyUrl`, `redirectUrl`, `customerName`, `email`, `mobile`; optional `remark`, `expireSeconds` (30 to 3600, default 3600).
- Transaction status values: `PENDING`, `COMPLETED`, `FAILED`.
- WebPay notification events: `order.payment.success`, `order.payment.failed`. QR events: `qrcode.payment.success`, `qrcode.payment.failed`.
- Refund status enum: `PENDING`, `PROCESS`, `COMPLETED`, `FAILED`, `CANCELLED` (examples also show `COMPLETE`). Transaction fees are not returned on refunds.
- WebPay channel codes include `FPX_*_URL`, `RPP_*_URL`, `TOUCHNGO_URL`, `BOOST_URL`, `GRPY_URL`, `MCASH_URL`, `CARD_URL`. QR channel: `DUITNOW_DYNAMIC_QR`.
- API base URL, sandbox host, callback retry policy, and PayCools egress IPs are not documented publicly (checked 2026-10-21).

## Critical Rules

- Keep the merchant RSA private key server-side only. Never use the example key pair printed in the docs.
- Sign the exact string you send as `param`; do not re-serialize after signing.
- Verify notifications over the `param` string value as received (after decoding the outer JSON), never over a re-serialized object.
- Use a constant-time-safe library verify call; reject on any verify failure or missing `sign`.
- Do not mark paid from `redirectUrl`, `payUrl` creation, QR display, or a `PENDING` status.
- Check amount (cents), `mchOrderId`, `transactionId`, and `channelCode` against the local record before settling.
- Make notification handling idempotent; duplicate or late notifications must not double-fulfil.
- `mchOrderId` must be unique (`21007` on duplicate); payouts also need a unique `idempotencyId` (`21015`).
- Send `notifyUrl` in WebPay create (the docs' example shows `callbackUrl`; the field table says `notifyUrl`). Confirm with PayCools.
- Expect IP restrictions (`21016 Merchant ip forbidden`); ask PayCools to allowlist your server egress IPs.
- Confirm base URL, UAT environment, enabled channels, and pricing with PayCools before claiming production readiness.

## Completion Criteria

A PayCools integration is done only when:

1. All Open API calls run server-side with `appId` and a SHA256withRSA signature over the exact `param` string.
2. The local payment record exists before WebPay or QR creation and stores `mchOrderId`, `transactionId`, amount in cents, and `channelCode`.
3. Redirect returns only show status; fulfilment waits for a verified notification or a requery showing `COMPLETED`.
4. The notification handler verifies `sign` with the PayCools public key, parses `param` after verification, settles idempotently, and replies `{"code":10000,"message":"Success"}`.
5. Requery via the detail endpoint covers missing notifications and pending orders.
6. Tests cover invalid signature, tampered `param`, duplicate notification, wrong amount, wrong `mchOrderId`, `FAILED`, expired order, and successful `COMPLETED`.
7. Deployment docs list `PAYCOOLS_APP_ID`, `PAYCOOLS_PRIVATE_KEY`, `PAYCOOLS_PLATFORM_PUBLIC_KEY`, `PAYCOOLS_API_BASE_URL`, the notify URL, redirect URL, and UAT/live separation.

## Loophole Review Loop

When asked whether a PayCools strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, payUrl page, QR display, backend order creator, PayCools API, notification receiver, JSON parser, database, fulfilment job, refund job, payout job, and key storage.
2. Check bypasses: fake redirect, forged notification, signature checked over re-serialized JSON, wrong public key (merchant key instead of PayCools key), UAT/live key mix, wrong amount unit (ringgit vs cents), reused `mchOrderId`, stale `PENDING` requery, duplicate notification, refund status `COMPLETE` vs `COMPLETED` mismatch, timestamp unit mismatch, IP not allowlisted, and duplicate payout without `idempotencyId`.
3. Add fixes: server-side signing, local pending record first, verify-then-parse, strict amount/order/channel checks, durable event log, unique constraints, separate env config, requery fallback, and tolerant refund status mapping.
4. Re-read `references/paycools-api.md` sections relevant to any gap.
5. State remaining uncertainty where it depends on undocumented items: base URL, UAT host, retry policy, notification source IPs, timestamp unit, and channel activation for the merchant account.
