---
name: setup-riipay
description: Set up, build, debug, review, and explain Riipay payment integrations using official Riipay docs, the Riipay developer guide, the official Riipay for WooCommerce plugin source, GET redirect payment requests, return URL and server-to-server callbacks, MD5 request and response signatures, transaction requery, merchant portal setup, staging and production environments, status codes F/A/S, and error codes. Use when working with Riipay Malaysia, Buy Now Pay Later, 3-month 0% instalments with debit or credit cards, Riipay checkout redirects, callback verification, requery, WooCommerce, or production readiness.
---

# Riipay Payment Gateway

Use this skill for Riipay (Malaysian BNPL, 0% instalments on debit or credit card) integration work.

## Source

Read `references/riipay-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, environments, payment request fields, request/response/query signature algorithms with verified sample values, return vs callback behaviour, status and error codes, requery, rate limit, and documented gaps.

Read `references/account-setup.md` when the user asks how to register as a merchant, where the Merchant Code and Secret Key live, staging vs production portals, return/callback URL settings, refunds, pricing, settlement, or the WooCommerce plugin.

Read `references/source-map.md` when choosing which official page or plugin file backs a fact, or to see what was checked and not found.

## Core Workflow

1. Identify environment: staging (`https://secure.uat.riipay.my`) or production (`https://secure.riipay.my`).
2. Identify surface: custom API redirect, WooCommerce plugin, or another e-commerce platform plugin (ask Riipay; only WooCommerce source is public).
3. Confirm merchant portal profile: Merchant Code, Secret Key, Return URL, Callback URL, Callback Method (GET, POST form, or POST JSON; default POST JSON), bank account.
4. Create a local pending payment with a unique `reference`, amount in ringgit as a 2-decimal string, and currency `MYR`.
5. Build the MD5 request signature server-side, then redirect the browser with a GET to `{BASE_URL}/v1/payment?...` (URL-encode every value).
6. On return (browser GET) and callback (server-to-server), verify the MD5 response signature using your stored merchant code, secret key, reference, currency, and expected amount.
7. Fulfil only on status `S`. Treat `A` as pending (on hold). Treat `F` as failed, but check `error_code` (for example `410` customer cancel) before closing the order.
8. Store `transaction_reference` from the first verified response; use `GET /v1/query/transaction` (max 2 queries per second) to confirm `A` or missing callbacks.
9. Settle idempotently: one transition to paid per local payment, keyed on reference plus `transaction_reference`.

## Core Facts

- Payment request: `GET {BASE_URL}/v1/payment` with query parameters; the customer is redirected to the Riipay hosted page.
- Requery: `GET {BASE_URL}/v1/query/transaction` with `merchant_code`, `transaction_reference`, `signature`; returns the payment response fields as JSON.
- Request signature: `md5(merchant_code + secret_key + reference + currency_code + amount)`.
- Response signature: `md5(merchant_code + secret_key + reference + currency_code + amount + transaction_reference + status_code)`.
- Query signature: `md5(merchant_code + secret_key + transaction_reference)`.
- `amount` is ringgit with exactly 2 decimals, no thousand separator (`1234.00`), minimum `1.00`. Re-format the response amount before hashing (JSON callbacks may send `1234`).
- Status codes: `F` failed, `A` accepted and under process, `S` success.
- Callbacks are sent only for transactions that reached a gateway result (success or failed). Customer cancel or invalid signature produce only a browser return, no callback.
- Return is always a browser GET with query parameters and may never arrive.
- `return_url` and `callback_url` per request override the portal defaults; they are not covered by the signature.
- Merchant portals: production `https://merchant.riipay.my`, staging `https://merchant.uat.riipay.my`.
- The official WooCommerce plugin supports `MYR` only and defaults the maximum order amount to RM1000, stating Riipay rejects orders above RM1000 "until further notice". Confirm the current limit with Riipay.
- No refund, cancel, capture, recurring, or tokenisation API is documented. Refunds follow the merchant's policy (merchant FAQ).
- No sandbox test cards, IP allowlist, or callback retry policy are published.

## Critical Rules

- Keep the Secret Key server-side only. The request URL passes through the browser, so never put the key in it, only the signature.
- Never mark paid from the browser return alone without verifying the response signature, and prefer callback or requery confirmation before shipping.
- Verify every response signature, including `F`, with a constant-time compare on the lowercase hex MD5.
- Hash with your stored expected amount for that reference, not a value you trust from the response; a mismatch then fails closed.
- Do not trust `status_message`, `error_message`, `description`, or `transaction_datetime`; they are outside the signature.
- Check `merchant_code` and `currency_code` in the response equal your configured values.
- Treat `A` as not paid. Only `S` fulfils.
- Do not mark an order failed on `F` with error codes the official plugin skips (`400`, `401`, `402`, `404`, `409`, `410`, `412`, `429`, `500` to `504`); let the customer retry.
- Respect the 2 queries per second requery rate limit; back off and do not poll in tight loops.
- Use separate Merchant Code, Secret Key, portal settings, and URLs for staging and production.

## Completion Criteria

A Riipay integration is done only when:

1. The payment request URL is built server-side with a correct MD5 request signature and URL-encoded values.
2. A local pending payment with unique reference, expected amount, and currency exists before redirect.
3. Return and callback handlers accept the configured callback method (GET, POST form, or POST JSON) and verify the response signature.
4. Settlement checks merchant code, reference, currency, amount, `transaction_reference`, and status `S`.
5. `A` results are confirmed later by callback or requery; duplicate callbacks and returns are idempotent.
6. Tests cover invalid signature, wrong amount, wrong reference, `F` with skip codes, `410` cancel, `A` then `S`, and duplicate callbacks.
7. Deployment docs list `RIIPAY_MERCHANT_CODE`, `RIIPAY_SECRET_KEY`, `RIIPAY_BASE_URL`, return URL, callback URL, and the portal callback method.

## Loophole Review Loop

When asked whether a Riipay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect URL, Riipay hosted page, return URL handler, callback handler, requery client, database, fulfilment job, merchant portal settings.
2. Check bypasses: forged return query string, tampered amount in the request URL, signature computed over response amount, `A` treated as paid, missing callback for cancel, lost return, duplicate callback, staging key in production, overridden `callback_url`, requery rate limit hit, order above the plugin RM1000 limit.
3. Add fixes: server-side signature, local expected amount in hash, constant-time compare, strict status `S`, stored `transaction_reference`, requery fallback with back-off, idempotent settlement, environment-specific config.
4. Re-read `references/riipay-api.md` sections relevant to any gap.
5. State remaining uncertainty: callback retry policy, expected callback HTTP response, refund handling, reference reuse after failure, current order limits, and fees are not documented publicly.
