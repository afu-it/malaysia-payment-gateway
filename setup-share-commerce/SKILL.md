---
name: setup-share-commerce
description: Set up, build, debug, review, and explain Share Commerce payment integrations using official Share Commerce docs (devdocs.share-commerce.com) and official plugin source, hosted Payment Request redirect, Seemless (iframe) payment, Enquiry requery, same-day Payment Void, Pre-Auth/Capture, Recurring tokenised billing, server-to-server callbacks, HMAC-SHA256 SCSign signatures, staging/production setup, Merchant ID and Secret Key, IP whitelisting, payment method codes, and WooCommerce/OpenCart/Magento/PrestaShop/EasyStore plugins. Use when working with Share Commerce Malaysia, FPX, cards, Touch 'n Go, Boost, GrabPay, ShopeePay, Maybank QR, Alipay, WeChat Pay, MyIOU BNPL, callback settlement, voids, pre-authorisation, subscriptions, or production readiness.
---

# Share Commerce Payment Gateway

Use this skill for Share Commerce online payment gateway integration work (Share Commerce Sdn Bhd, Kuala Lumpur).

## Source

Read `references/share-commerce-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, every request/response field table, the SCSign algorithms per endpoint with verified worked examples, status/response code enums, payment method codes, and the documented doc inconsistencies.

Read `references/account-setup.md` when the user asks how to onboard, which credentials exist, where they come from, staging vs production, IP whitelisting, callback URL registration, plugins, or pricing.

Read `references/source-map.md` when choosing which official page or plugin file backs a fact, and for coverage notes.

## Core Workflow

1. Identify environment: staging (`stagingpayment.share-commerce.com`) or production (`payment.share-commerce.com`).
2. Identify surface: hosted Payment Request (`/payment`), Seemless iframe (`/seemlesspayment`), Pre-Auth (`/CreateCheckout/PreAuth`), Recurring (`/CreateCheckout/Recurring`), or a plugin.
3. Confirm onboarding: Merchant ID, Secret Key, and testing credentials per environment (emailed by Share Commerce), server IP whitelisted, callback URL registered with Share Commerce.
4. Create a local pending order with a unique `MerchantOrderNo` before building the request.
5. Build the request server-side and compute `SCSign` server-side; never ship the Secret Key to the browser.
6. Send the browser to the gateway (auto-submitted form POST or GET URL), or redirect to the returned `CheckoutUrl`.
7. Treat the browser return to `RedirectUrl` as UX only, even though it is signed.
8. Verify the server-to-server callback `SCSign` with HMAC-SHA256 over the exact received values, then settle idempotently.
9. Use `/enquiry` (or `/PreAuth/Enquiry`, `/Recurring/Enquiry`) as fallback when the callback is missing.
10. Re-check amount, currency, `MerchantOrderNo`, `MerchantID`, and `RespCode == "00"` before marking paid.

## Core Facts

- Staging base: `https://stagingpayment.share-commerce.com`. Production base: `https://payment.share-commerce.com`.
- Hosted payment: `GET` or `POST /payment`. Requery: `POST /enquiry`. Void (same day only): `POST /Payment/Void`.
- Auth is `MerchantID` plus an HMAC-SHA256 `SCSign` keyed with the merchant Secret Key. No bearer token or API key header.
- Form-style calls (`/payment`, `/seemlesspayment`, `/PreAuth`, `/CreateCheckout/PreAuth`) carry `SCSign` as a field: HMAC of all field values concatenated in order, no separators.
- JSON server calls (`/enquiry`, `/Payment/Void`, `/PreAuth/*`, `/CreateCheckout/Recurring`, `/Recurring/Enquiry`) carry `SCSign` in an HTTP header: HMAC of the raw JSON body.
- `PaymentMethod` and `Versioning` are in the request sign string only when you send them (current plugins send both with `Versioning=7`).
- Redirect (GET query) and callback (HTTP POST JSON) carry `SCSign`: HMAC over the received values in order, excluding `SCSign`.
- Response code `RespCode`: `00` success, `09` pending, `99` failed, `IE` application error, `10` voided.
- Amounts are ringgit decimals with 2 places (`Decimal(18,2)`, e.g. `101.50`), not sen.
- Callback URL is not a request field; Share Commerce stores it at onboarding.
- No refund API is documented; only same-day `Payment/Void` (full void, response `FULL_REFUNDED`).
- Official plugins: WooCommerce, OpenCart 3/4, Magento 2, PrestaShop 1.7 (github.com/share-commerce), EasyStore built in.

## Critical Rules

- Keep the Secret Key server-side. Different keys for staging and production.
- Do not copy sample sign strings from the docs page: they contain hidden characters (a zero-width space in `https://`, a non-breaking space in a description).
- Sign the exact strings you send, in the exact order you send them. Do not reformat amounts (`101.50` must stay `101.50`, not `101.5`).
- For callbacks, verify over the raw JSON values as received; decoding numbers to floats changes `101.50` to `101.5` in many languages.
- Use constant-time comparison for SCSign (the official plugins use `==`; do better).
- Do not fulfil from the redirect alone; wait for a verified callback or a verified Enquiry result.
- Treat `RespCode` as a string (`"00"`). Exception: `/CreateCheckout/PreAuth` returns integer `1`/`9`.
- `Recurring/CreateCheckout` and `CreateCheckout/PreAuth` responses are not signed; trust only later signed redirect/callback/enquiry data.
- Settlement must be idempotent on `MerchantOrderNo` plus `TxnRefNo`.
- Some published sample hashes do not reproduce (see api reference). Prove your signer against staging before go-live.

## Completion Criteria

A Share Commerce integration is done only when:

1. Requests are built and signed server-side with the environment's Secret Key.
2. Every request field is present (optional fields as empty strings) in the documented order.
3. The callback handler verifies SCSign over the received values, constant-time, before any state change, and returns a 200 response (the official plugins reply `OK`).
4. Settlement checks `MerchantID`, `MerchantOrderNo`, `TxnAmount`, `CurrencyCode`, `RespCode`, and stores `TxnRefNo`.
5. Duplicate callbacks, redirect-after-callback, and Enquiry fallback are idempotent.
6. Tests cover bad signature, tampered amount, wrong order number, `09` pending, `99` failed, `IE`, and duplicate success.
7. Config documents `SHARECOMMERCE_MERCHANT_ID`, `SHARECOMMERCE_SECRET_KEY`, `SHARECOMMERCE_ENV`, the redirect URL, and the callback URL registered with Share Commerce, per environment.

## Loophole Review Loop

When asked whether a Share Commerce strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser form/redirect, iframe postMessage, backend signer, Share Commerce gateway, callback receiver, raw-body parser, database, fulfilment job, void job, Enquiry poller, whitelisted IPs.
2. Check bypasses: forged redirect query, forged postMessage, forged callback, signature over re-serialised JSON, float-formatted amount, wrong field order, missing optional field, sandbox/production key mix, reused `MerchantOrderNo`, pending `09` treated as paid, void after day end, unsigned CreateCheckout response trusted.
3. Add fixes: server-side signing, raw-value verification, constant-time compare, strict amount/currency/order checks, durable event log, unique constraints, Enquiry fallback, per-environment config.
4. Re-read `references/share-commerce-api.md` sections relevant to any gap.
5. State remaining uncertainty: unverified doc sample hashes, Payment Enquiry signing input, callback retry policy, seemless response `Signature` algorithm, refund availability, pricing, and settlement timing must be confirmed with Share Commerce.
