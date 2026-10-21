---
name: setup-2c2p
description: Set up, build, debug, review, and explain 2C2P payment integrations using official 2C2P docs, PGW v4.3/v4.5 Payment Token API with JWT HS256, hosted redirect checkout, Direct API Do Payment, Secure Pay JS, Drop-In UI, backend notifications, Payment Inquiry, Payment Maintenance refund/void/settle with JWE and JWS, recurring plans, card and customer tokens, QuickPay links, payouts, mobile SDK, batch SFTP and reconciliation, sandbox demo merchants and test cards. Use when working with 2C2P Malaysia, MYR, FPX online banking, DuitNow QR, Touch 'n Go, GrabPay, Boost, ShopeePay, cards, MyDebit, IPP instalments, Atome, SPayLater, Apple Pay, Google Pay, webhook settlement, refunds, or production readiness.
---

# 2C2P Payment Gateway

Use this skill for 2C2P (PGW v4.x) payment gateway integration work.

## Source

Read `references/2c2p-api.md` first, before giving factual API guidance or writing integration code. It has environments, the four security schemes, the full endpoint catalogue, Payment Token fields, the redirect flow, backend and frontend notification formats, Payment Inquiry, amount rules, the settlement checklist and sandbox data.

Read `references/2c2p-direct-api.md` for an own checkout UI (Payment Option, Do Payment, Transaction Status, Secure Pay JS, Drop-In UI, Apple Pay, Google Pay, Malaysian channel shapes).

Read `references/2c2p-maintenance.md` for refund, void, settle, refund status, refund notification, recurring maintenance, card and customer token maintenance, and JWE + JWS key setup.

Read `references/2c2p-codes.md` for response codes, Malaysian channel, FPX and agent codes, cut-off times and test cards.

Read `references/2c2p-mobile-sdk.md` for Android, iOS, Flutter and React Native apps; `references/2c2p-quickpay-payout.md` for payment links and payouts; `references/2c2p-batch-recon.md` for SFTP batch, reconciliation, SoftPOS, plugins and SNAP.

Read `references/account-setup.md` for onboarding, keys, sandbox, pricing and settlement questions, and `references/source-map.md` to find the official page behind any fact.

## Core Workflow

1. Identify environment (sandbox or production) and surface: hosted redirect, Direct API, Drop-In UI, mobile SDK, plugin, QuickPay link, recurring plan, payout or SoftPOS.
2. Confirm credentials and setup: Merchant ID, Secret Key, enabled channels, return URLs, and for maintenance the RSA key pair and 2C2P public key.
3. Create a local pending order with a unique `invoiceNo`, expected amount and currency before calling 2C2P.
4. Call Payment Token server-side with a JWT HS256 `{"payload"}` body. Continue only if `respCode` is `0000`.
5. Redirect to `webPaymentUrl`, or run the Direct API / SDK flow with the `paymentToken`.
6. Treat the frontend return (`paymentResponse`, unsigned base64url) and Transaction Status as UX only.
7. Verify the backend notification JWT with the Secret Key, then check merchant, invoice, amount, currency and `respCode`.
8. Settle idempotently. Use Payment Inquiry for missing, late or doubtful results.
9. Run refunds, voids and captures through Payment Maintenance with JWE + JWS, with idempotency and status checks.

## Core Facts

- PGW API hosts: sandbox `https://sandbox-pgw.2c2p.com`, production `https://pgw.2c2p.com`.
- Payment Token: `POST /payment/4.5/paymentToken` (docs pages) or `/payment/4.3/paymenttoken` (OpenAPI 4.3). Use one version consistently.
- Payment Inquiry: `POST /payment/4.5/paymentInquiry` with `merchantID` plus `invoiceNo` or `paymentToken`.
- PGW signing: JWT HS256 with the merchant Secret Key string; request and response bodies are `{"payload":"<jwt>"}`.
- Payment Token mandatory fields: `merchantID`, `invoiceNo`, `description`, `amount`, `currencyCode`. `paymentToken` is valid 30 minutes.
- `amount` is a decimal in ringgit (`150.00`), not sen; `currencyCode` is `MYR`.
- Backend notification: POST `{"payload":"<JWT>"}` to `backendReturnUrl` (or the portal default); claims match Payment Inquiry.
- Success is `respCode` `0000`; `0001` pending; flow code `2000` means "completed, check backend or inquire".
- Payment Maintenance: `https://demo2.2c2p.com/PaymentAction/2.0/action` (sandbox), `https://t.2c2p.com/PaymentAction/2.0/action` (production); XML `PaymentProcessRequest` v4.6, JWE RSA-OAEP + A256GCM with the 2C2P public key, then JWS PS256 with the merchant private key, `text/plain`.
- `processType`: `I` inquiry, `R` refund, `RS` refund status, `V` void, `S` settle. Maintenance `respCode` `00` is success.
- FPX uses channel `123`, agent channel `WEBPAY`, agent codes `MY...` (for example `MYM2U`); legacy `FPX_*` codes must be migrated.
- Malaysian sandbox demo merchant `JT07` (MYR); FPX test banks `MYSBIA`, `MYSBIB`, `MYSBIC`; test Visa `4111111111111111`, OTP `123456`.

## Critical Rules

- Keep the Secret Key and merchant private key server-side only; never in browser, app or repo.
- Do not mark paid from the frontend return, Transaction Status, Do Payment response, SDK result, iframe `postMessage` or QR display.
- Verify every JWT with HS256 and reject other algorithms; verify JWS PS256 before decrypting maintenance responses.
- Match `merchantID`, `invoiceNo`, `amount`, `currencyCode` and `respCode` against the stored order before settling.
- Make notification handling and settlement idempotent; notifications can repeat or arrive out of order.
- Never reuse an `invoiceNo`; it triggers duplicate-invoice errors (`9015`, `5005`).
- Refund only settled transactions; void only same day before the acquirer cut-off; send `idempotencyID` and check `RS` / `I` before retrying.
- Treat REFUND_PENDING (`RP`, code 42) as unfinished; poll refund status or verify the `notifyURL` callback.
- Do not quote fees or settlement times; 2C2P publishes no Malaysian rate card.
- Say "Not documented in official docs" for rate limits, notification IP ranges and retry schedules instead of guessing.

## Completion Criteria

A 2C2P integration is done only when:

1. Payment Token runs server-side with JWT HS256, the correct host and one consistent API version.
2. The browser or app only carries `webPaymentUrl` or `paymentToken`; no secrets reach the client.
3. The backend notification handler verifies the JWT and checks merchant, invoice, amount, currency and `respCode`.
4. Payment Inquiry is used as the fallback for missing or doubtful results, with the same checks.
5. Settlement, refunds and voids are idempotent and logged.
6. Tests cover a forged or wrong-key JWT, a wrong amount or currency, a duplicate notification, a pending (`0001`) result, a failed result, an expired token and a successful payment.
7. Deployment docs list Merchant ID, Secret Key, return URLs, maintenance keys and separate sandbox and production values.

## Loophole Review Loop

When asked whether a 2C2P strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser return, iframe, Drop-In UI, mobile SDK, Payment Token creator, PGW API, backend notification receiver, Payment Inquiry, maintenance client, key storage, database, fulfilment job, refund job, merchant portal settings and enabled channels.
2. Check bypasses: forged `paymentResponse`, unsigned or `alg:none` JWT, wrong Secret Key, sandbox and live mix, 4.3 and 4.5 path mix, reused `invoiceNo`, wrong amount or currency, Transaction Status used for settlement, duplicate or late notification, pending FPX or wallet marked failed too early, refund retried after timeout, void after cut-off, legacy `FPX_*` agent codes, and a plugin notification blocked by the store firewall.
3. Add fixes: server-side token creation, local pending order first, strict JWT verification, amount, currency and invoice checks, durable event log, idempotency keys, Payment Inquiry fallback, maintenance status checks, separate environment config, and channel checks with Payment Option.
4. Re-read the relevant sections of `references/2c2p-api.md`, `references/2c2p-maintenance.md` and `references/2c2p-codes.md` for any gap.
5. State remaining uncertainty where it depends on the merchant profile version, enabled channels, acquirer support for card-on-file, settlement terms, or anything marked "Not documented in official docs".
