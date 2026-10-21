---
name: setup-spaylater
description: Set up, build, debug, review, and explain SPayLater (Shopee PayLater, ShopeePay BNPL) payment integrations using official ShopeePay developer docs, Payment Gateway v1 hosted checkout with allowed_payment_method spay_later, Checkout with ShopeePay v3 with preferred_payment_method_type spay_later, X-Airpay-ClientId and X-Airpay-Req-H HMAC-SHA256 Base64 signatures, Notify Transaction Status callbacks, Check Transaction Status polling, refunds, invalidate and cancel, payment channel tenure codes, Account Linking, Link & Pay SPayLater instalment options, Auth & Capture, the SPayLater promotional messaging widget and SDKs, Shopify and WooCommerce plugins, and sandbox vs production hosts. Use when working with SPayLater Malaysia, Monee Capital BNPL, ShopeePay SPayLater checkout, instalment tenure display, ShopeePay callbacks, refunds, or production readiness.
---

# SPayLater (ShopeePay BNPL)

Use this skill for SPayLater acceptance on a merchant's own website, app or store in Malaysia. SPayLater is accepted through ShopeePay's APIs; it is a payment method (`spay_later`), not a separate API. For Shopee marketplace sellers there is nothing to integrate.

## Source

Read `references/spaylater-api.md` before giving factual API guidance or writing integration code. It holds hosts, auth and signature (with a verified test vector), the endpoint catalogue, Gateway v1 and Checkout v3 field tables, status, callbacks, refunds, invalidate/cancel, errcodes, and all enums including SPayLater payment channels.

Read `references/spaylater-tokenized.md` when the merchant wants to show SPayLater tenures on its own checkout (Link & Pay API-based), link ShopeePay accounts, charge recurring payments, or hold and capture.

Read `references/spaylater-widget.md` for "As low as RM X/mth" promotional messaging (Web, Android, iOS, Shopify) and branding rules.

Read `references/account-setup.md` for onboarding, credentials, env vars, sandbox, plugins, settlement, and pricing pointers.

Read `references/source-map.md` to pick the right official page, see coverage, and see the gaps only ShopeePay can answer.

## Core Workflow

1. Confirm the merchant has a signed ShopeePay agreement that includes SPayLater, plus Client ID, Secret Key, `merchant_ext_id`, `store_ext_id` for the environment.
2. Pick the surface: Payment Gateway v1 hosted checkout, Checkout with ShopeePay v3, Link & Pay API-based (own tenure picker), plugin (Shopify, WooCommerce), or in-person MPM/CPM.
3. Pick the host pair for that surface and environment (wallet `api.uat.wallet.airpay.com.my` / `api.wallet.airpay.com.my`, gateway `api.gw.uat.airpay.com.my` / `api.gw.airpay.com.my`).
4. Create a local pending payment with reference, expected amount x100, currency `MYR`, environment.
5. Create the order server-side, signing the exact JSON string sent with HMAC-SHA256 + Base64 in `X-Airpay-Req-H`.
6. Select SPayLater: `allowed_payment_method: ["spay_later"]` (v1), `preferred_payment_method_type: "spay_later"` (v3), or `payment_method_type: "spay_later"` + tenure `payment_option_reference` (Link & Pay).
7. Redirect to `checkout_url` or `redirect_url_http`. Treat the return URL as UX only.
8. Verify each callback signature over the raw body, match reference, merchant, store, amount and currency, dedupe, settle, reply `{"errcode":0}`.
9. Because Malaysian v3 callbacks are sent once with no retry, poll Check Transaction Status (or Get Checkout Status) on the documented schedule when no callback arrives, then invalidate or cancel.
10. Refund through the API for the same surface; requery refund status.

## Core Facts

- Malaysia region code `com.my`. Wallet API production `https://api.wallet.airpay.com.my`, staging `https://api.uat.wallet.airpay.com.my`.
- Payment Gateway production `https://api.gw.airpay.com.my`, sandbox `https://api.gw.uat.airpay.com.my`.
- Headers: `X-Airpay-ClientId: <CLIENT_ID>`, `X-Airpay-Req-H: base64(hmac_sha256(secret, raw_body))`.
- Checkout v3: `POST /v3/merchant-host/order/create`; MY `validity_period` max 3600 s, default 1200 s.
- Gateway v1: `POST /v1/checkout`, `GET /v1/checkout/{checkout_id}`, `POST /v1/checkout/cancel/{checkout_id}`, `POST /v1/refund`, `GET /v1/refund/{refund_id}`.
- Status: `POST /v3/merchant-host/transaction/check` (`transaction_type` 13 payment, 15 refund); `status` 3 = success.
- Refund v3: `POST /v3/merchant-host/transaction/refund/create-new`; partial refunds allowed; blocked 12 am to 5 am local by default.
- Amounts are integers x100 (RM 12.34 = `1234`).
- SPayLater payment channels: 4 BNPL, 5 = 2M, 6 = 3M, 7 = 6M, 8 = 12M, 9 = 18M, 10 = 24M, 11 = 4M, 12 = 5M.
- MY v3 callback success: `payment_status` 1 (Checkout/MPM/CPM) or `transaction_status` 3 (Link & Pay/Subscription).
- Gateway callback events: `checkout.successful`, `checkout.expired`, `checkout.cancelled`, `refund.successful`, `refund.failed`.
- Polling: every 5 s increasing to 100 s, then every 5 minutes up to 24 hours.
- SPayLater in Malaysia is provided by Monee Capital Malaysia Sdn. Bhd.; ShopeePay Malaysia is platform partner and payment facilitator.
- Settlement: daily T+1, weekly Monday, or monthly first day; payout up to 2 more days.
- Rate limits, callback IPs, SPayLater min/max amounts and public test users are not documented (checked 2026-10-21).

## Critical Rules

- Keep the Secret Key server-side. Only the SPL widget Access Key may go to the browser.
- Sign and send the same serialized string; never re-serialize after signing.
- Verify callback signatures on the raw body with constant-time comparison before parsing.
- Never mark paid from the return URL or its query parameters.
- Confirm amount and reference against the original request before settling (ShopeePay requirement).
- Make callback handling idempotent; Gateway `event_id` or reference + `transaction_sn` as keys.
- Do not rely on callbacks alone in Malaysia: they are not retried.
- Treat SPayLater CPM/Checkout "Processing" as pending, not failed.
- Compare Gateway status strings case-insensitively; accept `succeeded` and `successful` for refunds.
- Do not send `validity_period` and `expiry_time` together.
- Do not domain-whitelist or IP-restrict ShopeePay redirect URLs.
- Show only `is_available: true` SPayLater tenures and display `compliance_notices`.
- Never show guaranteed-approval wording; follow the widget compliance rules.
- Do not quote merchant fees; they are set in the commercial agreement.

## Completion Criteria

An SPayLater integration is done only when:

1. Order creation runs server-side with correct host, `X-Airpay-ClientId`, and a signature that matches the docs test vector in a unit test.
2. SPayLater is selected or offered per the chosen surface, and the agreement confirms SPayLater is enabled.
3. Callback URL is registered with ShopeePay for the right environment.
4. Callback handler verifies `X-Airpay-Req-H` over the raw body, checks reference, merchant, store, amount, currency, dedupes, and replies `errcode 0`.
5. A poller covers missing callbacks with the documented schedule, then invalidates or cancels.
6. Refunds (full and partial) and refund requery work and are idempotent.
7. Tests cover bad signature, replayed callback, wrong amount/reference, processing then success, expired, cancelled, refund failure, and 12 am to 5 am refund block.
8. Deployment docs list `SHOPEEPAY_CLIENT_ID`, `SHOPEEPAY_SECRET_KEY`, `SHOPEEPAY_MERCHANT_EXT_ID`, `SHOPEEPAY_STORE_EXT_ID`, `SHOPEEPAY_ENV`, callback URL, return URL, and staging/production separation.

## Loophole Review Loop

When asked whether an SPayLater strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser return URL, Shopee/ShopeePay app jump, hosted checkout page, merchant backend, ShopeePay wallet API, gateway API, callback receiver, raw-body parser, database, poller, refund job, widget script, plugin settings.
2. Check bypasses: forged callback, signature over parsed JSON, wrong secret or environment, wrong host family, fake return URL params, wrong amount/currency/reference/store, duplicate callback, missed callback with no retry, SPayLater "Processing" treated as failure, expired order paid late, double refund, refund during block window, tenure option unavailable, widget shown where SPayLater is not live.
3. Add fixes: server-side creation, local pending record, raw-body HMAC with constant-time compare, strict field matching, durable event log, idempotency keys, documented polling, invalidate/cancel on timeout, separate staging/production config, compliance review.
4. Re-read the relevant sections of `references/spaylater-api.md` and `references/spaylater-tokenized.md`.
5. State remaining uncertainty from the gaps list in `references/source-map.md` (GET signing, response signature header, Gateway callback ack, min/max amounts, refund window, product availability in Malaysia).
