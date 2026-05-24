---
name: setup-toyyibpay
description: Set up, build, debug, review, and explain toyyibPay payment integrations using official toyyibPay API references, onboarding manuals, DuitNow QR notes, pricing/support pages, WooCommerce plugin notes, and WorkDo setup docs. Use when working with toyyibPay categories, bills, payment links, FPX, cards, DuitNow QR, callbacks, return URLs, MD5 callback verification, transaction lookups, inactive bills, sandbox/live setup, secret keys, category codes, WooCommerce setup, or ToyyibPay settlement checks.
---

# toyyibPay Payment Gateway

Use this skill for toyyibPay payment gateway integration work.

## Source

Read `references/toyyibpay-docs.md` before giving factual API guidance or writing integration code. It contains source URLs, environment bases, endpoint map, callback hash rules, amount units, DuitNow QR fields, dashboard setup notes, pricing/support notes, WooCommerce setup notes, and WorkDo configuration notes from the provided docs plus follow-up source checks.

Read `references/account-setup.md` when user asks how to register, which API keys are needed, where credentials come from, or what dashboard setup is required.

## Core Workflow

1. Identify environment: sandbox at `dev.toyyibpay.com` or production at `toyyibpay.com`.
2. Confirm credential scope: User Secret Key, Category Code, and optional Enterprise User Secret Key.
3. Create or reuse a toyyibPay Category, then create local pending payment before creating a Bill.
4. Create Bill server-side with fixed amount in cents, internal reference in `billExternalReferenceNo`, return URL, and callback URL.
5. Redirect customer to `https://toyyibpay.com/{BillCode}` or sandbox equivalent.
6. Verify callback hash with `MD5(userSecretKey + status + order_id + refno + "ok")`.
7. Re-check payment using `getBillTransactions` when possible, then settle idempotently after status, amount, channel, provider, and reference match.

## Core Facts

- Production API base: `https://toyyibpay.com/index.php/api/`
- Sandbox API base: replace `toyyibpay.com` with `dev.toyyibpay.com` and use sandbox credentials.
- Requests are `POST`; docs accept `multipart/form-data` or `application/x-www-form-urlencoded`.
- `billAmount` uses cents, for example `100` equals RM 1.00.
- Main flow: `createCategory` -> `createBill` -> hosted bill URL -> callback/return -> `getBillTransactions`.
- Payment status values in callbacks/return URLs: `1` success, `2` pending, `3` fail.
- `getBillTransactions` status filter values: `1` successful, `2` pending transaction, `3` unsuccessful, `4` pending.
- DuitNow QR requires account activation, `enableDuitNowQR=1`, and `chargeDuitNowQR` set when enabled.
- WorkDo-style setup usually needs Secret Key and Category Code.

## Critical Rules

- Keep User Secret Key and Enterprise User Secret Key server-side only.
- Do not trust Return URL parameters for fulfilment; treat them as browser UX only.
- Reject callbacks when `hash` does not match the documented MD5 formula.
- Treat `order_id` as local external reference from `billExternalReferenceNo`; verify it maps to exactly one local pending payment.
- Verify callback `amount` and/or fetched `billpaymentAmount` against local expected amount before marking paid.
- Callback URL cannot be localhost; use public HTTPS or a tunnel for testing.
- Do not enable dynamic amount bills (`billPriceSetting=0`) for fixed-price orders unless local settlement can validate allowed amount.
- Do not invent payment channel availability, fees, card support, DNQR activation, settlement timing, or package limits. Verify live portal or current docs when these matter.

## Loophole Review Loop

When asked whether a toyyibPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List every trust boundary: browser return, backend checkout route, toyyibPay Bill API, callback receiver, database, fulfilment job, dashboard settings, and settlement report.
2. Check each bypass: fake success return, forged callback, MD5 hash mismatch, wrong order reference, wrong amount, wrong status, duplicated callback, pending marked as paid, dynamic amount over/underpayment, inactive/expired bill, sandbox/live credential mix, DNQR disabled account, missing public callback URL, and split-payment side effects.
3. Add fixes for each bypass: server-side Bill creation, local pending payment first, callback hash verification, transaction re-fetch, unique provider reference, strict amount/status checks, idempotent fulfilment, environment-separated config, expiry handling, and audit logs.
4. Re-read `references/toyyibpay-docs.md` sections relevant to any gap.
5. State remaining uncertainty explicitly if it depends on current account activation, dashboard package, payment channel enablement, pricing, settlement timing, or WorkDo product-specific settings.
