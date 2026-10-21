---
name: setup-adyen
description: Set up, build, debug, review, and explain Adyen payment integrations using official Adyen docs, Checkout API v72 (/sessions, /paymentMethods, /payments, /payments/details), Hosted Checkout, Web Drop-in and Components, HMAC-SHA256 signed webhooks, captures, refunds, cancels, reversals, tokenization, payment links, test cards, live endpoint prefix, API key and client key setup, and go-live checks. Use when working with Adyen for Malaysian merchants, MYR, FPX (MOLPay), DuitNow QR, Touch 'n Go, GrabPay, Boost, Atome, Alipay, Alipay+, cards, Apple Pay, Google Pay, Adyen webhook settlement, reconciliation, or production readiness.
---

# Adyen Payment Gateway

Use this skill for Adyen online payment integration work for Malaysian merchants.

## Source

Read `references/adyen-api.md` first. It is the entry point: environments, auth, idempotency, endpoint catalogue, flow summary, HMAC algorithm, statuses, amounts, IP ranges, common errors, and links to the topic files.

Read the topic file for the task:

- `references/adyen-checkout.md`: Sessions flow, Hosted Checkout, Advanced flow, Drop-in, actions, result codes, 3DS2.
- `references/adyen-webhooks.md`: webhook setup, payload, HMAC verification code, event codes, retries.
- `references/adyen-modifications.md`: capture, refund, cancel, reversal, amount update, lifecycle, disputes API.
- `references/adyen-payment-methods-my.md`: FPX issuers, DuitNow, Touch 'n Go, GrabPay, Boost, Atome, Alipay, Alipay+, WeChat Pay, cards.
- `references/adyen-tokenization-links.md`: tokens and payment links.
- `references/adyen-errors-testing.md`: sandbox data, refusal reasons, error codes.
- `references/adyen-fields.md`: full request and response field tables from the official OpenAPI.

Read `references/account-setup.md` for registration, documents, Customer Area paths, credentials, env vars, payment method activation, payouts, reports, and go live.

Read `references/source-map.md` to pick the right official page and to see what the skill covers.

## Core Workflow

1. Identify environment: test (`checkout-test.adyen.com`) or live (`<PREFIX>-checkout-live.adyenpayments.com`).
2. Pick the flow: Hosted Checkout, Sessions flow with Drop-in, or Advanced flow; payment links for invoices.
3. Confirm Customer Area setup: merchant account, API credential roles, API key, client key with allowed origins, payment methods added (FPX and others), webhook with HMAC key.
4. Create a local pending order (amount in sen, `MYR`) before calling Adyen.
5. Call `POST /sessions` (or `/paymentMethods` then `/payments`) server-side with `x-API-key`, `idempotency-key`, `reference` = order id, `countryCode: "MY"`, `returnUrl`.
6. Render Drop-in with `session.id` and `sessionData` and the client key, or redirect to the Hosted Checkout `url`.
7. On return, submit `redirectResult` and show `resultCode` as UX only.
8. Verify each webhook item's `additionalData.hmacSignature` (HMAC-SHA256, hex-decoded key, 8 fields joined by `:`), store it, return 2xx within 10 seconds, then process.
9. Settle idempotently from `AUTHORISATION` with `success == "true"` after matching reference, amount, currency, merchant account and environment.
10. Run captures, refunds, cancels and reversals server-side; treat their webhooks as the outcome.

## Core Facts

- Checkout API version used by the docs: v72. Test base `https://checkout-test.adyen.com/v72`.
- Live base `https://<PREFIX>-checkout-live.adyenpayments.com/checkout/v72`; prefix from live Customer Area > Developers > API URLs.
- Auth: `x-API-key` header from Developers > API credentials; client key (`test_` / `live_`) for the frontend only.
- `amount.value` is an integer in minor units; MYR has 2 decimals.
- `idempotency-key` header, max 64 characters, kept at least 7 days.
- Webhook HMAC string: `pspReference:originalReference:merchantAccountCode:merchantReference:amount.value:amount.currency:eventCode:success`.
- Webhooks must get 2xx within 10 seconds; retries run up to 30 days.
- Modifications return `status: "received"`; outcomes come by `CAPTURE`, `REFUND`, `CANCELLATION`, `CANCEL_OR_REFUND`, `TECHNICAL_CANCEL` webhooks.
- FPX type `molpay_ebanking_fpx_MY` with required `issuer`; all 18 issuers must be shown (PayNet rule).
- DuitNow is QR only and its result comes only by webhook; DuitNow and Touch 'n Go are enabled through Support.
- Malaysian local methods have no recurring and no separate capture; Boost allows full refunds only.
- No webhook is sent at settlement; use the Settlement details report.

## Critical Rules

- Keep the API key, HMAC key and report credentials server-side. The client key is public but restrict its allowed origins.
- Never fulfil from `resultCode`, `onPaymentCompleted`, `redirectResult`, `sessionResult` or a return URL alone.
- Verify HMAC per notification item with the hex-decoded key and constant-time comparison; reject items without `hmacSignature`.
- For header-signed webhooks (token life cycle, Management), verify over the raw body.
- Use test keys with test hosts and live keys with the live prefix host; check webhook `live` matches.
- Accept, store and acknowledge webhooks before business logic; make processing idempotent on `pspReference`, `eventCode`, `success`.
- Compare `amount.value` as an integer and `amount.currency` against the stored order.
- Send every POST with an `idempotency-key`; retry only with the same key.
- Show all 18 FPX banks; do not filter the issuer list.
- Respect method limits: GrabPay `reference` max 32 `[A-Za-z0-9_-]`; Atome MYR 10 to 1,500 debit or 5,000 credit, required shopper fields, no portable electronics.
- Do not copy fees; link `https://www.adyen.com/pricing`.
- Say "Not documented in official docs" instead of guessing.

## Completion Criteria

An Adyen integration is done only when:

1. Session or payment creation runs server-side with `x-API-key` and `idempotency-key`.
2. The frontend only renders Drop-in or redirects; it cannot mark an order paid.
3. A Standard webhook is configured per environment with its own HMAC key.
4. The webhook handler verifies HMAC exactly as documented, stores, acknowledges within 10 seconds, and processes asynchronously.
5. Settlement checks `merchantReference`, amount, currency, `merchantAccountCode`, `live`, and `success`.
6. Duplicate webhooks, late `AUTHORISATION` after `Pending`, `OFFER_CLOSED`, and modification failures are handled idempotently.
7. Tests cover bad HMAC, replayed event, wrong amount or currency, refused payment, pending FPX or wallet, refund failure, and a successful payment.
8. Deployment docs list `ADYEN_API_KEY`, `ADYEN_MERCHANT_ACCOUNT`, `ADYEN_CLIENT_KEY`, `ADYEN_HMAC_KEY`, `ADYEN_ENVIRONMENT`, `ADYEN_LIVE_URL_PREFIX`, return URL, webhook URL, and test or live separation.

## Loophole Review Loop

When asked whether an Adyen strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser, Drop-in, return URL, Hosted Checkout page, backend creator, Adyen API, webhook receiver, queue, database, fulfilment, modification jobs, Customer Area settings, reports.
2. Check bypasses: forged return URL, forged webhook, HMAC over wrong fields or unhexed key, test key in live, wrong prefix host, wrong merchant account, amount or currency mismatch, duplicate or out-of-order events, slow handler causing retries, pending marked failed too early, refunds before capture, cancel after capture, method not enabled, missing FPX banks.
3. Add fixes: server-side creation, local pending order first, documented HMAC check, constant-time compare, strict field matching, durable event log, idempotency keys, environment-separated config, DNS-based allowlisting, report reconciliation.
4. Re-read the relevant reference file for any gap.
5. State remaining uncertainty that depends on the merchant's contract, enabled methods, live region, payout model, or Support-enabled features.
