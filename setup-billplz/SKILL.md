---
name: setup-billplz
description: Set up, build, debug, review, and explain Billplz payment integrations using Billplz API docs, help center, GitHub plugins, status pages, payment collections, payment forms, Payment Order payouts, X Signature callbacks/redirects, V5 checksums, sandbox/live setup, API Secret Key, Collection ID, and X Signature Key.
---

# Billplz Payment Gateway

Use this skill for Billplz payment collection, payment form, catalog, and Payment Order payout work.

## Source

Read `references/billplz-docs.md` before giving factual API guidance or writing integration code. It contains source URLs, environment bases, API flow, endpoints, X Signature, V5 checksum, Payment Order, status pages, GitHub plugin notes, and settlement/security cautions.

Read `references/account-setup.md` when user asks how to register, which API keys are needed, where credentials come from, or what dashboard setup is required.

## Core Workflow

1. Identify product: Collection/Bill checkout, Payment Form/Catalog, Payment Order payout, OAuth partner flow, plugin, or zakat flow.
2. Identify environment: sandbox at `billplz-sandbox.com` or production at `billplz.com`.
3. Confirm credentials: API Secret Key, Collection ID, and X Signature Key.
4. Create local pending payment before creating Billplz bill.
5. Create Bill server-side under a Collection with amount in cents, `callback_url`, and optional `redirect_url`.
6. Redirect payer to returned Bill URL.
7. Verify X Signature on callback/redirect with HMAC-SHA256 and X Signature Key.
8. Settle idempotently after Bill ID, Collection ID, amount, paid/state, and local reference match.

## Core Facts

- Production API base: `https://www.billplz.com/api/`.
- Sandbox API base: `https://www.billplz-sandbox.com/api/`.
- Auth: HTTPS Basic Auth where API Secret Key is username and password is blank.
- API accepts `application/x-www-form-urlencoded` and JSON; responses are JSON.
- Currency: MYR only. Amounts use smallest currency unit, for example `100` equals RM 1.00.
- Every Bill belongs to a Collection. Open Collection cannot create bills via API.
- `callback_url` is compulsory for reliable backend updates; `redirect_url` is optional for user experience.
- Callback and redirect execution order is not fixed; handlers must prevent duplicate updates.
- X Signature uses HMAC-SHA256 over sorted key-value source string with X Signature Key.
- V5 endpoints such as Payment Order use `epoch` and `checksum` with HMAC-SHA512.

## Critical Rules

- Keep API Secret Key and X Signature Key server-side only.
- Use X Signature callback/redirect. Basic callback/redirect are disabled by default for newer accounts due security reasons.
- Do not rely on redirect URL alone; browser redirect is not guaranteed.
- Return HTTP `200` quickly for valid callbacks. Billplz retries failed callbacks and account rank can be degraded for unsuccessful callback attempts.
- Verify amount, paid/state, Bill ID, Collection ID, and local reference before marking paid.
- For Payment Order payouts, verify HMAC-SHA512 checksum and use unique `reference_id` to prevent duplicate payout creation.
- Do not scrape status pages for business logic; use them for operational awareness.

## Loophole Review Loop

When asked whether a Billplz strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, backend bill creation, Billplz API, callback receiver, database, fulfilment job, dashboard settings, and payout job.
2. Check bypasses: fake redirect, forged callback, wrong X Signature source string, wrong amount, wrong bill/collection, duplicate callback, callback before redirect, redirect before callback, unpaid/due bill marked paid, deleted bill, API Secret Key leak, sandbox/live mix, Payment Order duplicate payout, wrong V5 checksum order.
3. Add fixes: server-side bill creation, local pending payment first, X Signature HMAC verification, strict amount/status checks, idempotency keys, durable event log, public HTTPS callback, environment separation, Payment Order `reference_id`, and audit trail.
4. Re-read `references/billplz-docs.md` sections relevant to any gap.
5. State remaining uncertainty if it depends on account plan, enabled payment gateways, current pricing, operational status pages, partner-only OAuth, or dashboard-only settings.
