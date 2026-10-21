---
name: setup-favepay
description: Set up, build, debug, review, and explain FavePay (Fave / FaveBiz) payment integrations using official Fave docs, the FavePay Omni (FPO) API, hosted web_url checkout, dynamic and static QR, merchant scan, webhooks, HMAC-SHA256 sign generation and verification, AES-128-CBC encrypted omni_ref redirects, transaction lookup, acknowledge, cancel QR, list transactions, full and partial refunds, sandbox UAT with test=true, and the official FavePay for WooCommerce plugin. Use when working with FavePay Malaysia or Singapore, Fave app QR payments, cards, DuitNow, GrabPay, ShopeePay, Alipay, WeChat Pay, Atome, Cashback, eCards, omni_reference, app_id, outlet_id, webhook settlement, refunds, or production readiness.
---

# FavePay (FavePay Omni) Payment Gateway

Use this skill for FavePay Omni (FPO) integration work.

## Source

Read `references/favepay-api.md` before giving factual API guidance or writing integration code. It holds environments, the signing algorithm with a verified worked example, the endpoint catalogue, full field tables for create, merchant scan, static QR, webhook, lookup, acknowledge, cancel, list and refund, statuses, errors and plugin behaviour.

Read `references/account-setup.md` when the user asks how to register, which credentials Fave issues, sandbox UAT and sign-off, env var names, payment methods, plugins, pricing or settlement.

Read `references/source-map.md` when choosing which official page or plugin file supports a fact.

## Core Workflow

1. Identify environment: Sandbox (`https://omni.dino.fave.ninja`) or Production (`https://omni.myfave.com`). Credentials are separate per environment.
2. Identify country path segment: `my` (MYR) or `sg` (SGD). Currency follows the path; there is no currency field.
3. Identify flow: online checkout (`format=web_url`), in-store dynamic QR (`txt`, `base64`, `favepay`), wallet QR (`grab`, `alipay`, `wcp`, `shopeepay`, `atome`, `paynow`), merchant scan, or static QR (planned).
4. Create a local pending payment with a unique `omni_reference` (`<Fave prefix>-<id>`, max 28 chars) before calling Fave.
5. `POST /api/fpo/v1/<cc>/qr_codes` server-side with `total_amount_cents`, `app_id`, `outlet_id`, `format`, `redirect_url`, `callback_url`, and `sign` last.
6. Redirect the customer to `code` (for `web_url`), or render the QR payload or image.
7. Treat the redirect back to `redirect_url` as browser UX only.
8. On the webhook, verify `sign` over the received fields in received order, then return HTTP 2XX quickly and process idempotently.
9. Confirm with `GET /api/fpo/v1/<cc>/transactions` (by `omni_reference` or `receipt_id`) when the webhook is late or missing.
10. Cancel unpaid QRs with `POST .../transactions/cancel`; refund paid ones with `POST .../transactions` and `status=refunded` (same calendar day by default).

## Core Facts

- Production base: `https://omni.myfave.com`. Sandbox base: `https://omni.dino.fave.ninja`. All paths start `/api/fpo/v1/<country_code>/`.
- Auth is a `sign` field: lowercase hex HMAC-SHA256 of the URL-encoded fields (form encoding, space as `+`) in the order sent, keyed with the Fave secret key.
- Always exclude `sign` and `country_code` from the signing string; also exclude path `outlet_id` (list), path `id` (acknowledge) and `details` (merchant scan).
- Do not sort keys. Nested objects flatten to bracket form, for example `shopper_details[email]=...`.
- Amounts are integer cents (`total_amount_cents`). Most response amounts are strings.
- Create response: `code`, `format`, `expires_in` (seconds). Default `format` is `base64`.
- Webhook statuses: `successful` and `rejected`. Expired or cancelled unpaid QRs can send `rejected`.
- `status_code`: 0 `pending_payment`, 1 `payment_processing`, 2 `successful`, 3 `disputed`, 4 `rejected`, 5 `refunded`.
- Transaction `id` prefixes: `FP_` FavePay, `RPP_` DuitNow, `GRAB_`, `PAYNOW_`, `EXT_` other wallets.
- Fave retries non-2XX webhooks at increasing intervals; the same payment can arrive more than once.
- `test=true` on create (sandbox only) returns a test page URL in `code` to simulate success or failure. Never send it to production.
- Errors use `error` and `error_description`. Bad sign is 401; unknown `app_id` is 404 `resource_not_found`; duplicate `omni_reference` is 400 `validation`.
- Refund window: same calendar day by default (`stale_transaction` after). Partial refunds not supported for DuitNow or PayNow.
- Rate limits and webhook source IPs: not documented in official docs (checked 2026-10-21).

## Critical Rules

- Keep the secret key and `app_id` server-side; the key signs requests, verifies webhooks and decrypts `omni_ref`.
- Build the signing string from the exact fields you send, in the exact order you serialise them; put `sign` last.
- Encode like PHP `urlencode` / `http_build_query`: space to `+`, everything except `A-Z a-z 0-9 - _ .` percent-encoded (uppercase hex).
- Verify webhooks over the received JSON keys in received order (use an order-preserving parser), and compare with a constant-time check.
- Do not mark paid from the redirect alone. Mark paid only after a verified webhook or a signed `GET /transactions` returns `successful`.
- Check `omni_reference`, `total_amount_cents`, `currency`, `outlet_id` and `status` against the local record before settling.
- Settle idempotently keyed on `receipt_id` or `id`; duplicate webhooks are expected.
- Generate a new `omni_reference` for every new QR on the same order; references must be unique.
- Use sandbox keys only with the sandbox base URL; Fave issues new production keys after UAT sign-off.
- Confirm static QR availability with Fave before use; the docs mark it as planned.

## Completion Criteria

A FavePay integration is done only when:

1. QR or `web_url` creation runs server-side with a correct `sign`, and returns `code`.
2. The redirect page shows status only, or re-queries the transaction server-side before showing success.
3. The webhook URL is registered with Fave (kick-off meeting) or passed as `callback_url`, per environment.
4. The webhook handler verifies `sign`, returns 2XX fast, and settles once per payment.
5. Settlement checks reference, amount, currency, outlet and `successful` status.
6. A scheduled requery handles missed webhooks; static QR flows call Acknowledge.
7. Tests cover bad sign, duplicate webhook, wrong amount, `rejected`, expired QR, cancel, full refund, and `stale_transaction`.
8. The UAT checklist (A1 to A4, B1 to B2, C1, D1 to D3) is passed in sandbox before asking for production keys.

## Loophole Review Loop

When asked whether a FavePay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, `omni_ref` decryption, backend QR creator, Fave Omni API, webhook receiver, JSON parser key order, database, fulfilment job, refund job, POS display, and FaveBiz dashboard.
2. Check bypasses: forged redirect, forged webhook, sign computed over sorted or re-serialised fields, wrong encoding (`%20` vs `+`, `~`, `*`), null or boolean values, sandbox key in production, `test=true` in production, reused `omni_reference`, wrong outlet, amount mismatch, duplicate webhook, late success after cancel, refund after the same-day window.
3. Add fixes: server-side create, local pending record first, order-preserving verification, constant-time compare, strict amount and reference checks, idempotent settlement, requery fallback, per-environment config, cancel before reissuing a QR.
4. Re-read `references/favepay-api.md` sections relevant to any gap.
5. State remaining uncertainty where docs are silent: `omni_ref` IV and padding, boolean and null encoding in `sign`, rate limits, webhook IPs, retry schedule, settlement timing, pricing.
