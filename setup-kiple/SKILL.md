---
name: setup-kiple
description: Set up, build, debug, review, and explain Kiple (kiplePay, formerly Webcash) payment integrations using official Kiple sources, which are the plugins and SDKs Kiplepay Sdn Bhd publishes on GitHub (WooCommerce, OpenCart, Magento, PrestaShop, WHMCS, CubeCart, osCommerce, PHP SDK, iOS SDK), the hosted payment form POST to wcgatewayinit.php, SHA-1 merchant_hashvalue and ord_key verification, return and dynamic_callback_url handling, returncode values, the enquiry requery, recurring card payments, split orders, UAT/production setup, Merchant ID and Secret Key, and settlement. Use when working with Kiple Malaysia, kiplePay, Webcash gateway, FPX, cards, DuitNow QR, e-wallets, callback verification, idempotent settlement, or production readiness.
---

# Kiple Payment Gateway

Use this skill for Kiple (kiplePay) payment gateway integration work.

## Source

Read `references/kiple-api.md` before giving factual API guidance or writing integration code. It contains environments, the endpoint catalogue, hosted form fields, amount rules, return and callback fields, return codes, the SHA-1 hash formulas with a worked example, the enquiry API, recurring and split-order fields, and the legacy iOS SDK.

Read `references/account-setup.md` when the user asks how to register, which credentials are needed, plugin settings, UAT vs live, payment method activation, settlement, refunds, or pricing.

Read `references/source-map.md` to find the exact official plugin file behind a fact, or to see what is not covered.

Kiple has no public API docs. Facts here come from Kiple's own official plugins (GitHub org `m2c`, "Kiplepay Sdn Bhd") and live gateway responses. Say so when you answer, and tell the user to confirm anything marked "Not documented" with Kiple.

## Core Workflow

1. Identify environment: UAT (`uat.kiplepay.com`) or production (`kiplepay.com`).
2. Identify surface: hosted form (one-off), recurring card payment, split order, or a platform plugin.
3. Confirm credentials: Merchant ID and Secret Key for that environment, MYR store currency.
4. Create a local pending payment with a unique `ord_mercref` and the exact amount string (two decimals) before building the form.
5. Build the form server-side: required fields plus `version=2.0`, `ord_returnURL`, `dynamic_callback_url`, and `merchant_hashvalue = sha1(secret + ord_mercID + ord_mercref + amountDigits)`.
6. Auto-submit the form by POST to `/wcgatewayinit.php`.
7. On the server callback (`nbcb` present) and on the browser return, recompute `ord_key = sha1(secret + merchantId + ord_mercref + amountDigits + returncode)` from stored values and compare in constant time.
8. Before marking paid, call the enquiry URL and require body `S`.
9. Settle idempotently on `returncode=100`, matching reference, amount, merchant ID and a not-yet-seen `wcID`.
10. Treat the browser return as UX; if the callback never arrives, requery by enquiry.

## Core Facts

- Payment entry: UAT `https://uat.kiplepay.com/wcgatewayinit.php`, live `https://kiplepay.com/wcgatewayinit.php` (PHP SDK uses `/gateway` on the same hosts).
- Enquiry: `GET /enquiry.php` or `/enquiries` with `ord_mercID`, `ord_mercref`, `ord_totalamt`; plain-text `S`, `F`, `A`, `Invalid Parameters`, `Records not Found`.
- Hash algorithm: SHA-1 lowercase hex over plain concatenation, secret first, no separators.
- `amountDigits` is the posted `ord_totalamt` with `.` and `,` removed (`50.00` becomes `5000`); split orders also remove `|`.
- Minimum amount RM1.00; return URL must be fully qualified (gateway validation page).
- `returncode`: `100` success, `E1` failed, `E2` aborted/timeout, `-1` invalid merchant request.
- Response includes `wcID` (Kiple transaction ID), `ord_key`, `ord_mercref`, `ord_totalamt`, `payment_type`, `entry_id`, `error_description`.
- Server-to-server callback goes to `dynamic_callback_url` and carries `nbcb`.
- `ord_date` format `Y-m-d H:i:s`; `version` is `2.0`.
- Currency MYR only (all official plugins).
- Recurring: card only (`payment_code=CC`), `ord_recurring_enable=1`, monthly/quarterly/yearly, needs a Kiple recurring subscription; cancellation in Kiple's portal.
- Split orders: pipe-separated `ord_mercID` and `ord_totalamt`, `ord_mmercID` master, hash with master secret.
- No refund, void, capture or payout API in any official source.

## Critical Rules

- Keep the Secret Key server-side; never put it in client JS or the form.
- Never fulfil from the browser return alone; verify `ord_key` and requery.
- Recompute the response hash from your stored merchant ID, reference and amount, not from posted values.
- Format amounts to exactly two decimals, no thousands separators, and hash the same string you post.
- Use a fresh `ord_mercref` per attempt and map it back to your order (plugins append `uniqid`).
- Dedupe on `wcID`; return and callback can both arrive for one payment.
- Use the UAT secret with UAT hosts and the live secret with live hosts.
- On `E1`/`E2` callbacks the official WooCommerce plugin sets orders on-hold, not failed; requery before final cancellation.
- The WooCommerce "Verify Key" help text says "Just enter any value"; ignore it, the value must be the Secret Key.
- Set session cookies `SameSite=None; Secure` if the return handler needs the buyer session.
- Do not invent refund, webhook retry, or method-code behaviour; mark it Not documented and ask Kiple.

## Completion Criteria

A Kiple integration is done only when:

1. The form is built server-side with `version=2.0`, two-decimal amount, unique `ord_mercref`, and correct `merchant_hashvalue`.
2. `ord_returnURL` and `dynamic_callback_url` are absolute HTTPS URLs for the right environment.
3. Both handlers verify `ord_key` with constant-time comparison against stored values.
4. Settlement requires `returncode=100`, a valid hash, an enquiry result `S`, and matching amount, reference and merchant ID.
5. Processing is idempotent on `wcID` and local payment state.
6. Tests cover bad hash, wrong amount, wrong reference, `E1`, `E2`, `-1`, duplicate callback, missing callback with enquiry fallback, and success.
7. Deployment docs list `KIPLE_MERCHANT_ID`, `KIPLE_SECRET_KEY`, `KIPLE_ENV`, return and callback URLs, and UAT/live separation.

## Loophole Review Loop

When asked whether a Kiple strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser form, Kiple hosted page, return handler, callback handler, enquiry call, database, fulfilment job, recurring callback, Kiple portal.
2. Check bypasses: forged return POST, hash over posted values, amount format mismatch (`50` vs `50.00`), reused `ord_mercref`, UAT secret in production, legacy `webcash.com.my` host mismatch, duplicate `wcID`, unsigned recurring callback, enquiry spoofing via DNS or HTTP, split order using sub-merchant secret.
3. Add fixes: stored-value hash check, constant-time compare, enquiry before fulfil over HTTPS, unique references, `wcID` unique index, environment-bound config, durable callback log.
4. Re-read `references/kiple-api.md` sections relevant to any gap.
5. State remaining uncertainty: callback acknowledgement format, retry policy, recurring callback signing, refunds, method codes, and anything in Kiple's private API document. These need confirmation from Kiple.
