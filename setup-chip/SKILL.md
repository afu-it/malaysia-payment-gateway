---
name: setup-chip
description: Set up, build, debug, and explain CHIP Collect payment integrations using the official CHIP Collect API documentation. Use when working with CHIP Collect purchases, checkout URLs, payment links, direct post, FPX, DuitNow QR, cards, e-wallets, callbacks, webhooks, signature verification, clients, recurring tokens, subscriptions, pre-authorization, refunds, captures, releases, payment methods, statements, account balance, account turnover, API keys, Brand ID, or CHIP Collect endpoint schemas.
---

# CHIP Collect API

Use this skill for CHIP Collect payment integration work.

## Source

Read `references/chip-collect-docs.md` before giving factual API guidance or writing integration code. It contains all downloaded CHIP Collect markdown pages plus embedded OpenAPI blocks from:

`https://docs.chip-in.asia/chip-collect/overview/introduction`

## Core Facts

- Base URL: `https://gate.chip-in.asia/api/v1/`
- Auth: `Authorization: Bearer <secret key>` for every request.
- Credentials: API key and Brand ID come from CHIP Collect Developers pages in portal.
- Amounts: money values are smallest currency units. Example: `price: 100` equals `RM 1.00`.
- Create checkout/payment link flow: `POST /purchases/`, then use response `checkout_url`.
- Payment status options: success callback, `GET /purchases/{id}/`, or webhook events.
- Test cards: `4444 3333 2222 1111` non-3DS, `5555 5555 5555 4444` 3DS, CVC `123`, expiry >= current month/year.
- Callback signature: `X-Signature` is base64 RSA PKCS#1 v1.5 over SHA256 digest of raw request body.
- Success callback public key: `GET /public_key/`; webhook public key: `Webhook.public_key`.

## Lookup Map

- Purchases: create, retrieve, cancel, capture, charge token, refund, release, resend invoice, mark paid.
- Clients: create/list/retrieve/update/delete, recurring token list/retrieve/delete.
- Webhooks: create/list/retrieve/update/delete.
- Payment methods: `GET /payment_methods/`, usually with `brand_id` and `currency`.
- Direct post: intro, card, FPX, DuitNow QR, e-wallet.
- Online flows: payment link, pre-authorization, subscriptions, custom flow.
- Account/reporting: balance, turnover, company statements.

## Working Rules

- Use exact endpoint paths and request fields from reference OpenAPI blocks.
- Preserve trailing slashes on CHIP Collect endpoints as documented.
- For code, keep secrets in env vars and never hard-code API keys or Brand IDs.
- For webhooks/callbacks, require raw request body for signature verification.
- If user asks for latest pricing, legal terms, supported methods, or production portal state, verify live docs first.
