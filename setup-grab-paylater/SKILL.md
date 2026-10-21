---
name: setup-grab-paylater
description: Set up, build, debug, review, and explain GrabPay and PayLater by Grab online checkout integrations using official Grab Developer docs, the One Time Charge (OTC) Partner API, GrabID OAuth 2.0 with PKCE, partner HMAC-SHA256 signatures, X-GID-AUX-POP proof-of-possession signatures, charge init and complete, web URL redirect, payment notification webhooks, status inquiry, refunds, tokenised (recurring) payments v4, Price Divider widgets, plugins, and staging/production setup. Use when working with GrabPay Malaysia, PayLater by Grab, Pay Next Month (POSTPAID), 4/8/12 instalments, Grab checkout on a web store, partner_id/partner_secret/client_id/client_secret/merchant_id credentials, webhook settlement, refunds, wallet binding, or production readiness.
---

# GrabPay & PayLater by Grab (Online Checkout)

Use this skill for GrabPay wallet and PayLater by Grab online checkout work (Grab Partner API, not in-store QR).

## Source

Read `references/grab-paylater-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, environments, both HMAC algorithms, the full One Time Charge flow with field tables, webhook payload, status inquiry, refunds, amount rules, rate limits, and token lifetimes.

Read `references/grab-paylater-codes.md` for every txStatus, reason code, redirect `error` value, 409 reason, OAuth error, and payment method enum.

Read `references/grab-paylater-tokenisation.md` only when the merchant needs saved-wallet, repeat, or subscription charges (Tokenised Payments v4: bind, charge, top-up, promo, unbind).

Read `references/account-setup.md` for sign-up, credentials, Integration Manager onboarding, plugins, Price Divider, branding rules, and settlement reports.

Read `references/source-map.md` to pick the right official page, and for doc coverage and known doc conflicts.

## Core Workflow

1. Confirm the merchant is onboarded: Grab issues `partner_id`, `partner_secret`, `client_id`, `client_secret`, `merchant_id` and registers the redirect URL and webhook URL. There is no self-serve sandbox signup.
2. Pick the surface: plugin (Shopify, WooCommerce, Magento, EasyStore) or Direct API. For Direct API one-time checkout use OTC; for repeat charges use Tokenisation v4.
3. Create a local pending order with a unique `partnerTxID` (max 32 chars, `[a-zA-Z0-9-_]`), amount in sen, currency `MYR`.
4. Server-side `POST /grabpay/partner/v2/charge/init` signed with partner HMAC (`Authorization: {partner_id}:{signature}`). Store the returned `request`.
5. Build the GrabID authorize URL with PKCE (`S256`), `state`, `nonce`, `acr_values=consent_ctx:countryCode=MY,currency=MYR`, `request`, scope `openid payment.one_time_charge`. Store `code_verifier` and `state` server-side keyed to the order.
6. Redirect the shopper. On return, check `state`; handle `error` values; take `code`.
7. Server-side `POST /grabid/v1/oauth2/token` with `code`, `code_verifier`, `client_id`, `client_secret`, `redirect_uri`, `grant_type=authorization_code`. Store `access_token` encrypted (needed for complete, status, and refunds).
8. `POST /grabpay/partner/v2/charge/complete` with `Authorization: Bearer {access_token}` and a fresh `X-GID-AUX-POP`. This step moves the money; without it the auth expires and Grab cancels and returns funds.
9. Mark paid only on final `txStatus: success` from complete, status inquiry, or a verified webhook, after matching `partnerTxID`, amount, currency, and merchant.
10. Handle `processing` by waiting for the webhook or polling status no more than once every 2 minutes. Refund with a new refund `partnerTxID` and the charge `txID` as `originTxID`.

## Core Facts

- Production API base: `https://partner-api.grab.com`. Staging base: `https://partner-api.stg-myteksi.com`.
- Current OTC docs say test with Production test credentials and a test Grab account issued by the Integration Manager; staging URLs are still listed per endpoint.
- Partner HMAC string-to-sign: `METHOD\nContent-Type\nDate\nPATH[?QUERY]\nBASE64(SHA256(body))\n`, HMAC-SHA256 keyed with `partner_secret`, base64.
- `X-GID-AUX-POP` = base64url(JSON `{"time_since_epoch":<unix>,"sig":base64url(HMAC-SHA256(client_secret, <unix> + access_token))}`), no padding.
- `Date` header: RFC 7231 GMT, same value in header and signature; clock skew over 5 minutes causes 401.
- Amounts are integers in the smallest unit: MYR 10.00 = `1000`.
- PayLater is not a separate API: it appears inside Grab checkout (`POSTPAID`, `INSTALMENT_4/8/12`). Hide it with `hidePaymentMethods: ["INSTALMENT","POSTPAID"]`; the wallet cannot be hidden. `metaInfo` is required for PayLater instalment transactions.
- Lifetimes: request code 20 minutes, OAuth code 15 minutes, OTC access token about 1 year (`expires_in` 31535999). Uncompleted charges are cancelled and auto-refunded after 15 minutes; call complete within 10 minutes of the auth webhook or redirect.
- Webhook: `Authorization: {partner_id}:{hmac}`; when recomputing use Content-Type `application/json; charset=utf-8`. Use `txStatus`, not deprecated `status`/`newStatus`.
- Rate limit: max 50 API calls per second per Partner ID; back off with jitter on 429.
- Refund window Malaysia: 365 days. Merchant wallet is cashed out daily, so refunds can fail with `merchant_insufficient_balance`.
- Settlement reports: SFTP or Grab Merchant Portal (`https://merchant.grab.com/`), delivered T+1 12:00, cut-off 23:59:59, CSV (pipe) or XLS.

## Critical Rules

- Keep `partner_secret`, `client_secret`, and access tokens server-side only; Grab says store secrets in a KMS, not config files, repos, or Redis.
- Never mark paid from the redirect `code` alone; the charge is only captured after `/charge/complete` returns `success`.
- Sign the exact body bytes you send; never re-serialise JSON between signing and sending.
- Generate a new `partnerTxID` for every init and every refund; reuse returns the existing transaction or `transaction_already_exists`.
- Validate `state` on redirect and `nonce` in `id_token`; reject mismatches.
- Treat any unknown reason code as non-final and re-query; only `success` and `failed` are final for charges.
- Verify the webhook signature, then confirm with a status inquiry before fulfilment, because the webhook signing path is not spelled out in the docs.
- Do not test production with cURL or Postman (Grab security guidance); keep secrets out of shell history.
- Do not copy pricing; MDR is negotiated. Link to the Grab merchant page instead.

## Completion Criteria

A GrabPay/PayLater integration is done only when:

1. Charge init, token exchange, complete, status, and refund calls run server-side with correct HMAC or Bearer plus POP auth.
2. PKCE verifier, state, and nonce are generated per order, stored server-side, and checked.
3. Redirect handler covers `code`, every documented `error`, and missing or mismatched `state`.
4. Webhook handler verifies `Authorization`, is idempotent per `partnerTxID` + `txType`, and returns HTTP 200 fast.
5. Settlement checks `partnerTxID`, `txID`, amount, currency, `merchantID`, final `txStatus`, and environment.
6. `processing` and authorised-but-not-completed states are recovered by a job (status inquiry, then complete or mark failed).
7. Tests cover bad signature, clock skew, duplicate webhook, user cancel, session expiry, insufficient balance, partial refund, and refund `processing`.
8. Deployment docs list `GRAB_PARTNER_ID`, `GRAB_PARTNER_SECRET`, `GRAB_CLIENT_ID`, `GRAB_CLIENT_SECRET`, `GRAB_MERCHANT_ID`, `GRAB_REDIRECT_URI`, webhook URL, and base URL per environment.

## Loophole Review Loop

When asked whether a Grab strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, GrabID authorize page, backend signer, token store, Grab Partner API, webhook receiver, raw-body parser, database, fulfilment job, refund job, settlement report.
2. Check bypasses: forged redirect `code`, replayed `state`, missing PKCE check, paid-on-redirect without complete, re-serialised JSON breaking HMAC, wrong Content-Type in webhook HMAC, clock skew, staging/production credential mix, reused `partnerTxID`, auth expiry before complete, duplicate webhook, refund with wrong `originTxID`, refund while merchant wallet is empty, leaked access token.
3. Add fixes: server-only secrets, per-order PKCE/state/nonce, exact-bytes signing, NTP sync, idempotency keys, durable event log, recovery job using status inquiry, strict amount/currency/merchant checks, encrypted token storage.
4. Re-read the relevant section of `references/grab-paylater-api.md` or `references/grab-paylater-codes.md` for any gap.
5. State remaining uncertainty: webhook signing path, GET body-hash convention, enabled payment methods, PayLater approval, Malaysian test accounts, and pricing all depend on Grab onboarding and the Integration Manager.
