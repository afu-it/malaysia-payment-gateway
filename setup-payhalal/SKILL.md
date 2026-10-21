---
name: setup-payhalal
description: Set up, build, debug, review, and explain Payhalal payment integrations using official Payhalal docs, hosted payment page V1 and V2 payment link API, QR Code API, SHA256 request hash, callback hash verification, return/success/cancel/callback URLs, transaction reconciliation APIs with MD5 hash, UAT sandbox/live setup, app ID and app secret, official WooCommerce/OpenCart/PrestaShop plugins, and recurring (plugin-only) flow. Use when working with Payhalal or Souqa Fintech, Shariah-compliant checkout, FPX, cards, e-wallets, BNPL, callback settlement, reconciliation, or production readiness.
---

# Payhalal Payment Gateway

Use this skill for Payhalal (operated by Souqa Fintech Sdn Bhd) payment gateway integration work.

## Source

Read `references/payhalal-api.md` before giving factual API guidance or writing integration code. It contains environments, payment request fields, the exact request and callback hash strings, callback fields, statuses, reconciliation APIs, errors, and the known conflicts between the docs and the official plugins.

Read `references/account-setup.md` when the user asks how to register, which documents are needed, where the App ID and secret live, UAT vs live, dashboard URL configuration, merchant requirements, or pricing.

Read `references/source-map.md` when choosing which official page or plugin file applies.

## Core Workflow

1. Identify environment: UAT (testing keys) or production (live keys).
2. Identify surface: V1 hosted page (form POST/GET redirect), V2 JSON payment link (selected merchants only), QR Code API, or an official plugin.
3. Confirm dashboard setup in Merchant Dashboard > Developer Tools: App ID, App Secret, Return URL, Success URL, Cancel URL, Callback URL.
4. Create a local pending payment with your own `order_id` and the exact amount string before redirecting.
5. Build the request server-side and compute `hash = sha256(app_secret + amount + currency + product_description + order_id + customer_name + customer_email + customer_phone)`.
6. Send the browser to Payhalal with an auto-submitting form (V1) or the returned `payment_link` (V2).
7. Treat the browser return as UX only; verify its hash too, but settle from the server callback.
8. Verify the callback hash over the posted values (see the hash conflict note in the API reference), compare with constant-time equality.
9. Settle idempotently only after `order_id`, amount, currency, `app_id`/environment and `status == SUCCESS` all match the local record.
10. Reconcile with the single/batch transaction reconcile API or the transaction list API when a callback is missing.

## Core Facts

- V1 production pay URL: `https://api.payhalal.my/pay`. V1 UAT pay URL: `https://api-testing.payhalal.my/pay`.
- V2 URL (JSON response with `payment_link`): `https://api.payhalal.my/v2/pay.php`. Live or testing is chosen by the key, not the URL. V2 is "in testing phase and only opened to selected merchants".
- QR Code API: production `https://api.payhalal.my/qrCreate`, UAT `https://api-testing.payhalal.my/qrCreate`.
- Pay endpoints accept both `GET` and `POST`.
- Required request fields: `app_id`, `currency`, `amount`, `product_description`, `order_id`, `hash`. Optional: `customer_name`, `customer_email`, `customer_phone`.
- Request hash: SHA256 hex of `APP_SECRET.AMOUNT.CURRENCY.PRODUCT_DESCRIPTION.ORDER_ID.CUSTOMER_NAME.CUSTOMER_EMAIL.CUSTOMER_PHONE` (plain concatenation, no separators).
- Callback hash per GitBook docs: same string plus `.TRANSACTION_ID.STATUS`. Every official plugin (WooCommerce 2018 and 2026, OpenCart 3.x) verifies WITHOUT `transaction_id`. Confirm in UAT.
- Callback status values: docs list `SUCCESS`, `FAILED`, `PENDING`; official plugins also handle `FAIL` and `TIMEOUT`.
- Request amount is a decimal "without thousand separators"; callback amount is documented "with thousand separator".
- Minimum amount error: `Minimum Amount is RM5.00`.
- Callback URL is optional; if not set, Payhalal sends no server data. Set at Merchant Dashboard > Developer Tools > Edit App.
- Reconcile APIs: `https://api-merchant.payhalal.my/` (production) and `https://api-merchant-uat.payhalal.my/` (UAT), hash `md5(merchant_id.app_key.app_secret)`.
- Unvalidated new merchants are in UAT mode with sandbox keys only.
- Refunds, rate limits, IP allowlists and test cards: not documented in official docs (checked 2026-10-21).

## Critical Rules

- Keep the App Secret server-side only; never compute the hash in browser JavaScript.
- Use the App ID, secret and URL of the same environment; a live key against `api-testing` (or the reverse) will not work.
- Build the hash from the exact strings you send, including empty strings for omitted optional customer fields.
- Format the request amount yourself as a fixed two-decimal string with no thousand separators (for example `1200.00`).
- Verify the callback hash over the raw posted values, then compare amount separately after removing thousand separators.
- Use constant-time comparison (`hash_equals`, `crypto.timingSafeEqual`).
- Do not mark paid from the Return/Success URL redirect without a verified hash and matching local record; prefer the server callback.
- Treat every status other than `SUCCESS` as not paid; `PENDING` must stay pending, not failed.
- Make callback handling idempotent: duplicate callbacks or a callback plus a return must not double-fulfil.
- Store `transaction_id` from the callback; reconcile APIs look up by `transaction_id`, not by your order id.
- Do not use the "Quick integration" snippet on the payhalal.my developers page (`/pay_v2`, Bearer token, `payhalal-signature` header). It is labelled pseudo-example and does not match the GitBook API docs or any official plugin.

## Completion Criteria

A Payhalal integration is done only when:

1. The request hash is computed server-side with the correct environment secret, over the exact sent values.
2. The browser is sent to the correct V1 URL (or V2 link) for the environment, with all required fields.
3. Return, Success, Cancel and Callback URLs are configured in Developer Tools for both UAT and live apps.
4. The callback handler verifies the hash, checks `order_id`, amount, currency, `app_id` and status, and settles idempotently.
5. The callback hash variant (with or without `transaction_id`) was confirmed with a real UAT transaction and the other variant is not silently accepted forever.
6. A reconciliation job uses the reconcile or transaction list API for orders stuck in pending.
7. Tests cover invalid hash, wrong amount, thousand-separator amount, duplicate callback, `FAIL`/`FAILED`/`TIMEOUT`/`PENDING`, and success.
8. Deployment docs list `PAYHALAL_APP_ID`, `PAYHALAL_APP_SECRET`, `PAYHALAL_MERCHANT_ID` (merchant email for reconcile), environment flag, and the four dashboard URLs.

## Loophole Review Loop

When asked whether a Payhalal strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: checkout form, browser return/success/cancel redirect, Payhalal hosted page, callback receiver, hash function, database, fulfilment job, reconcile job, dashboard Developer Tools settings.
2. Check bypasses: forged return POST, tampered amount or order id in the form, hash computed in browser, hash variant mismatch, accepting both variants indefinitely, amount compared with thousand separator, sandbox/live key mix, missing callback URL, `PENDING` treated as failed, duplicate callback, callback for an unknown order, currency forced to non-MYR, reused `order_id`.
3. Add fixes: server-side hash, local pending record first, constant-time verification, strict amount/currency/order/app_id checks, idempotent settlement keyed on `order_id` + `transaction_id`, reconcile job, separate UAT and live config.
4. Re-read `references/payhalal-api.md` sections relevant to any gap.
5. State remaining uncertainty: callback hash variant, exact status strings, refund process, V2 access, recurring enablement, and anything the docs mark as not documented.
