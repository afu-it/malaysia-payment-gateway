---
name: setup-senangpay
description: Set up, build, debug, review, and explain senangPay payment integrations using official senangPay guide pages, Manual Integration API, Direct API, callbacks, return URLs, query status APIs, refund API, sandbox/live setup, Merchant ID, Secret Key, Hash Type, shopping cart plugins, e-commerce integrations, recurring payments, payout API, tokenisation, split settlements, and DOKU migration references. Use when working with senangPay hosted checkout, payment forms, callbacks, SHA256/MD5 hash verification, Direct API client sessions, JavaScript Web SDK, FPX bank lists, refunds, package capabilities, sandbox testing, or senangPay dashboard setup.
---

# SenangPay

Use this skill for senangPay payment gateway integration work.

## Source

Read `references/senangpay-api.md` before giving factual API guidance or writing integration code. It contains source URLs, integration surfaces, environment URLs, endpoint roles, parameter rules, hash formulas, callback behavior, query APIs, Direct API, FPX bank list, refund API, and settlement/security cautions.

Read `references/account-setup.md` when user asks how to register, which credentials are needed, where keys come from, sandbox setup, dashboard setup, package capabilities, pricing, support, or migration context.

Read `references/source-map.md` when choosing which senangPay guide page applies to hosted checkout, Direct API, plugins, callbacks, query APIs, refunds, recurring, tokenisation, split settlement, payout, sandbox, or migration.

## Core Workflow

1. Identify environment: sandbox or production.
2. Identify integration surface: hosted/manual payment form, shopping cart plugin, e-commerce platform integration, Direct API, recurring payment, refund API, payout API, tokenisation, or split settlement.
3. Confirm credentials and dashboard setup: Merchant ID, Secret Key, Hash Type, Return URL, and Callback URL.
4. Create a local pending payment before sending the customer to senangPay.
5. Generate request hash server-side using the correct formula and account Hash Type.
6. Redirect to hosted checkout or create a Direct API client session server-side.
7. Treat Return URL as browser UX only; do not fulfil from return alone.
8. Verify callback/return hash server-side, requery order or transaction status when needed, then settle only after amount, order ID, status, merchant, environment, and provider transaction reference match.
9. Make settlement idempotent because senangPay can send multiple callbacks and later callbacks can supersede earlier statuses.

## Core Facts

- Production hosted checkout URL: `https://app.senangpay.my/payment/{merchant_id}`.
- Sandbox hosted checkout URL: `https://sandbox.senangpay.my/payment/{merchant_id}`.
- Hosted/manual checkout accepts GET or POST parameters.
- Hosted/manual checkout amount uses two decimal places, for example `25.50`.
- Direct API amount uses cents, for example `200` means RM 2.00.
- Manual request hash source sequence: `{secret_key}{detail}{amount}{order_id}`.
- Return/callback status values: `1` successful, `0` failed, `2` pending authorization.
- Return/callback verification source sequence: `{secret_key}{status_id}{order_id}{transaction_id}{msg}`.
- Query order status endpoint: `GET https://app.senangpay.my/apiv1/query_order_status`.
- Query transaction status endpoint: `GET https://app.senangpay.my/apiv1/query_transaction_status`.
- Direct API endpoints: `POST https://api.senangpay.my/payment/v3` and `POST https://api.sandbox.senangpay.my/payment/v3`.
- Direct API requires Enterprise package, authorised merchant access, and outgoing server IP whitelisting.
- Merchant ID and Secret Key are in Dashboard > Settings > Profile > Shopping Cart Integration Link.
- SHA256 is recommended. MD5 is legacy/deprecated and should only be used for systems that explicitly require it.
- Callback URL uses HTTP POST and must output exactly `OK` without HTML tags for successful receipt.

## Critical Rules

- Keep Secret Key and private hash inputs server-side only.
- Do not mark paid from Return URL, frontend event, SDK response, or browser redirect alone.
- Prefer SHA256/HMAC-SHA256 and avoid MD5 unless supporting an explicit legacy integration.
- Always verify hash with the exact sequence documented for the active flow.
- Verify local order, expected amount, status, transaction ID/reference, merchant ID, and environment before marking paid.
- Log callback events durably and make processing idempotent before returning `OK`.
- Expect duplicate callbacks for the same transaction.
- Expect pending or failed callbacks before a later successful callback; latest callback can be the most up-to-date status.
- Avoid callback redirects and session-dependent callback handlers.
- Separate sandbox and production credentials, URLs, callback URLs, Return URLs, and dashboard settings.
- For Direct API, never expose Secret Key in browser JavaScript and remember UUID is valid for only 60 seconds.
- For refund API, store local refund attempts and do not retry blindly after duplicate, partial-refund, refund-window, or amount-exceeded errors.
- If package capability, payment method availability, settlement timing, current pricing, IP whitelisting, migration behavior, or dashboard state matters, verify live senangPay dashboard/docs/support before claiming certainty.

## Loophole Review Loop

When asked whether a senangPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser Return URL, backend checkout creator, senangPay hosted checkout, Direct API session endpoint, JavaScript SDK, callback receiver, query APIs, database, fulfilment job, refund job, dashboard settings, and package/channel activation.
2. Check bypasses: fake success return, forged callback, wrong hash type, wrong hash sequence, wrong amount format, wrong order ID, wrong merchant ID, sandbox/live mix, duplicate callback, failed callback followed by success, pending marked paid, stale Direct API UUID, Direct API Secret Key leak, unwhitelisted Direct API server IP, inactive FPX bank, duplicate refund request, partial refund unsupported, and refund after 180-day window.
3. Add fixes: server-side local pending payment, server-side hash generation, callback hash verification, status requery, strict amount/status/order checks, idempotency keys, callback event log, exact `OK` response after durable recording, environment-separated config, bank status filtering, package capability checks, and refund idempotency.
4. Re-read `references/senangpay-api.md` sections relevant to any gap.
5. State remaining uncertainty if it depends on current package, channel enablement, pricing, dashboard configuration, DOKU migration state, or senangPay support approval.
