---
name: setup-moby
description: Set up, build, debug, review, and explain MOBY (MobyPay, Moby Omni) payment integrations using official MOBY docs, the MOBY OpenAPI spec and MOBY plugin source, covering Bearer token auth with Client ID and Secret Key, hosted checkout payLink, Direct API payUrl for cards, FPX, e-wallets, DuitNow QR and shariah-compliant BNPL, callbackUrl webhooks, HMAC-SHA256 callback signatures, payment status and detail requery, capture, void, refunds, card tokenisation and recurring charges, subscriptions, POS and kiosk TCP with installment plans, sandbox test cards, rate limits, and plugins for WooCommerce, Shopify, Magento, OpenCart and EasyStore. Use when working with MOBY or MobyPay checkout, Moby Islamic BNPL, pay.mobycheckout.com, callback verification, settlement, refunds, recurring billing, or production readiness.
---

# MOBY (MobyPay) Payment Gateway

Use this skill for MOBY / MobyPay payment integration work.

## Source

Read `references/moby-api.md` before giving factual API guidance or writing integration code. It holds environments, auth, the endpoint catalogue, hosted checkout and Direct API field tables, callback payload, the signature algorithm, status/detail requery, capture/void/refund, sandbox data, rate limits, errors, plugins, and the known conflicts between MOBY's sources.

Read `references/moby-recurring.md` for card tokens, token charges, charge status/refund, the subscription API, and dashboard Recurring Billing.

Read `references/moby-pos.md` for POS/kiosk TCP integration, IPP installment plans, and Visa plan cancel.

Read `references/account-setup.md` for onboarding, where credentials come from, sandbox vs live, dashboard features, env vars, settlement and pricing caveats.

Read `references/source-map.md` to pick the official page for a topic and to see coverage.

## Core Workflow

1. Identify environment: sandbox `https://dev.pay.mobycheckout.com` or production `https://pay.mobycheckout.com`.
2. Identify surface: hosted checkout (default), Direct API for one method, token/recurring, dashboard payment link or recurring billing, plugin, or POS/kiosk.
3. Confirm credentials: Client ID and Secret Key for that environment, issued by `merchantsupport@moby.my`; merchant category (insurance merchants must send NRIC).
4. Create a local pending payment with a unique `referenceNo`, expected amount (MYR, 2 decimals), and environment before calling MOBY.
5. Get a Bearer token with `POST /api/auth/token`; cache until `expired_at`, refresh on `401`.
6. Create the session server-side (`/api/merchant/payment/checkout/hosted` returns `payLink`; `/api/merchant/payment/direct/pay` returns `payUrl`) and redirect the customer.
7. Treat the `returnUrl` redirect as UX only, even though it carries a signed query string.
8. On the `callbackUrl` POST, verify `signature` (HMAC-SHA256, Secret Key, 8 fields in fixed order) before anything else.
9. Settle idempotently only when `status` is `success` and `referenceNo`, `amount`, environment and merchant match; reply HTTP 200 `{"status":"success"}`.
10. Requery with `/api/merchant/payment/detail` or `/status` when a callback is missing, late, or not `success`.

## Core Facts

- Token: `POST /api/auth/token` with `clientId` + `secretKey`; use `Authorization: Bearer <token>`; docs say 30 minute lifetime.
- Hosted checkout: `POST /api/merchant/payment/checkout/hosted`, required `clientId`, `customerName`, `referenceNo`, `details`, `amount`; response `{ success, payLink }`.
- Direct pay: `POST /api/merchant/payment/direct/pay` with `payMethod` from `POST /api/merchant/payment/method/list`; response `{ success, payUrl }`.
- Amounts are MYR decimals (ringgit, not sen) on live v1 endpoints. The `/v2/...` cents API is Coming Soon only.
- Callback fields: `referenceNo`, `transactionId`, `amount`, `payMethod`, `status`, `time`, `merchantOrderRef`, `description`, `signature`.
- Signature: lowercase hex HMAC-SHA256 with the Secret Key over the concatenated values of `amount`, `description`, `merchantOrderRef`, `payMethod`, `referenceNo`, `status`, `time`, `transactionId` (null becomes empty string, no separator).
- Statuses: docs list `success`, `failed`, `pending`; MOBY's plugins also handle `cancelled` and `processing`.
- Requery: `POST /api/merchant/payment/status` (`clientId`, `referenceNo`, `transactionId`) or `/detail` (`clientId`, `referenceNo`).
- Refund `POST /api/merchant/payment/refund`; void `POST /api/merchant/payment/void` same day only; capture `POST /api/merchant/payment/capture`.
- Recurring: `POST /api/v2/tokens` returns a save-card `redirect_url`; charge with `POST /api/v2/tokens/{token}/charges` (`3ds` required).
- Rate limits: auth 10/min per client, general 60/min per token, `429` with `Retry-After`.
- Sandbox: Mastercard test cards, expiry drives result (`01/39` approved); FPX only ACF Bank B2B `user1`/`password1`; no e-wallet sandbox (real charges).
- No webhook IP list, signature header, dashboard webhook registry, or public status page is documented.

## Critical Rules

- Keep the Secret Key server-side. It mints tokens and is the callback HMAC key.
- Never mark paid from `returnUrl`, `payLink`/`payUrl` creation, a frontend message, or an unsigned payload.
- Verify the signature over values exactly as received; do not reformat `amount` or `time`; map `null` to `""` (JavaScript `String(null)` is `"null"`).
- Use constant-time comparison and reject on mismatch; log the raw payload instead of loosening the check.
- Do not put your own query params in `callbackUrl`/`returnUrl` unless you strip them before hashing; also strip a leading `?` from keys MOBY may append.
- Make `referenceNo` unique per attempt; hosted checkout returns no session ID, so it is your only correlation key.
- Make callback handling idempotent; one `referenceNo` may see several transactions and repeated callbacks.
- Compare amounts as 2-decimal strings or integer sen, never floats.
- Do not test e-wallets casually in sandbox; they charge real money.
- Do not build on `/v2/...`, Financing, Loyalty, or Merchant Management endpoints; they are not in production.
- Where docs, OpenAPI and plugins disagree (listed in the references), confirm with MOBY Merchant Support before go-live.

## Completion Criteria

A MOBY integration is done only when:

1. Token, session creation and requery run server-side with environment-matched Client ID, Secret Key and base URL.
2. A local pending payment with unique `referenceNo` and expected amount exists before redirecting.
3. The callback handler verifies the HMAC over the documented 8 fields with the Secret Key, in constant time.
4. Settlement checks `status === "success"`, `referenceNo`, amount, environment and merchant, and is idempotent.
5. The handler returns HTTP 200 `{"status":"success"}` only after durable processing.
6. Missing or non-final callbacks are reconciled through `/api/merchant/payment/detail` or `/status`.
7. Refunds and voids are recorded as pending until a signed callback or status check confirms them.
8. Tests cover valid signature, tampered field, null `merchantOrderRef`, wrong amount, duplicate callback, `failed`/`pending`/`cancelled`, and a `?`-prefixed query key.
9. Deployment docs list `MOBY_CLIENT_ID`, `MOBY_SECRET_KEY`, base URL, callback URL, return URL, and sandbox/live separation.

## Loophole Review Loop

When asked whether a MOBY strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser return redirect, MOBY hosted page, backend session creator, token cache, MOBY API, callback receiver, body parser, database, fulfilment job, refund/void job, recurring charge scheduler, plugin, dashboard, POS/kiosk TCP link.
2. Check bypasses: forged return URL, unsigned or tampered callback, `null` hashed as `"null"`, reformatted amount, wrong Secret Key or environment, reused `referenceNo`, wrong amount, duplicate callback, callback before local record committed, expired token mid-flow, rate-limit storms from per-request auth, partial refund field guess, sandbox e-wallet charges, replayed kiosk message, default kiosk secret.
3. Add fixes: server-side session creation, pending record first, documented 8-field HMAC, constant-time compare, strict amount/reference checks, durable event log, idempotent settlement, token caching, status/detail requery, nonce and timestamp checks on TCP, environment-separated config.
4. Re-read the relevant section of `references/moby-api.md`, `references/moby-recurring.md` or `references/moby-pos.md` for any gap.
5. State remaining uncertainty where it depends on MOBY confirmation: direct-pay callback delivery, partial refund field, callback retry policy, insurance NRIC field name, token charge `client_id` placement and `custom_data` type, `x-token` issuance, settlement timing, account method activation, and pricing.
