---
name: setup-billplz
description: Use when working with Billplz Bills, Collections, Open Collections, callback_url, redirect_url, X Signature verification, HMAC_SHA256 callbacks, V4 tokenization, V5 Payment Orders, HMAC_SHA512 checksum, FPX, e-wallets, Direct Payment Gateway, sandbox/live setup, or idempotent Billplz settlement.
---

# Billplz Payment Gateway

Use this skill for Billplz payment gateway integration work.

## Source

Read `references/billplz-api.md` before giving factual API guidance or writing integration code. It contains source URLs, environment bases, endpoint map, X Signature rules, V5 checksum rules, callback/redirect payload fields, FPX bank codes, and sandbox notes.

## Core Facts

- Production base URL: `https://www.billplz.com/api/`
- Sandbox base URL: `https://www.billplz-sandbox.com/api/`
- Auth: HTTP Basic Auth - API Secret Key as username, no password (append `:` after key).
- Currency: **MYR only**. All amounts in **smallest currency unit** (e.g. `200` = RM 2.00).
- API versions: V3 (stable, no new features), V4 (active, adds tokenization + 2-recipient split), V5 (active, Payment Orders with HMAC_SHA512 checksum).
- Credentials: API Secret Key + XSignature Key - both from account settings page. Sandbox and production are **separate accounts**.

## Core Workflow

1. Identify environment: sandbox or production.
2. Identify integration surface: Bill checkout, Open Collection (payment form), V5 Payment Order, or card tokenization.
3. Create a local pending payment record before calling Billplz API.
4. Create a Collection (or reuse existing) server-side.
5. Create a Bill server-side - return `url` to redirect customer.
6. Configure `callback_url` (compulsory) and `redirect_url` (optional, for UX).
7. Verify X Signature on every callback and redirect before settling.
8. Settle idempotently after verifying provider, bill ID, amount, currency, and `paid=true`.

## Integration Rules

- `callback_url` is **compulsory**. Never rely on `redirect_url` alone for settlement.
- `redirect_url` and `callback_url` execution order is **not guaranteed**. Implement idempotency guards.
- Never expose API Secret Key or XSignature Key in browser code, logs, or client bundles.
- Verify X Signature (HMAC_SHA256) on every callback and redirect before marking paid.
- For X Signature, include empty values, flatten nested keys by prepending parent keys, sort case-insensitively, and exclude only `x_signature`.
- For V5 Payment Orders, compute checksum (HMAC_SHA512) on every request using XSignature Key.
- For V5 Payment Orders, use Get Payment Order or callbacks for status reconciliation; handle `completed`, `refunded`, and `cancelled` terminal callback states.
- Do not invent bank codes, payment channel values, or status enums. Use reference docs.
- Store Billplz `bill_id`, `collection_id`, callback checksum result, and processed status to prevent duplicate fulfillment.
- Return HTTP `200` from callback endpoint after verification and durable recording.
- Pull FPX bank list on **hourly** basis - bank availability changes dynamically.

## X Signature Verification (Callbacks and Redirects)

### Callback (POST)
1. Extract all POST body fields **except `x_signature`**.
2. For each field: concatenate key + value (no separator between key and value).
3. For nested fields, prepend parent keys to child keys before concatenating.
4. Include empty values exactly as received.
5. Sort all entries **case-insensitively ascending**.
6. Join with `|` pipe.
7. Compute `HMAC_SHA256(source_string, XSignature_Key)`.
8. Compare with received `x_signature`. Match = authentic.

### Redirect (GET)
1. Extract all `billplz[...]` query params **except `billplz[x_signature]`**.
2. For each: concatenate `billplz` + key + value.
3. Sort case-insensitively ascending, join with `|`.
4. Compute `HMAC_SHA256(source_string, XSignature_Key)`.
5. Compare with `billplz[x_signature]`. Match = authentic.

## V5 Checksum (Payment Orders)

Every V5 request requires `epoch` (UNIX timestamp) and `checksum`:

1. Concatenate **values only** of required args in the **strict order** specified per endpoint (no delimiters).
2. Compute `HMAC_SHA512(raw_string, XSignature_Key)`.

Endpoint checksum arg orders:
- Create Payment Order Collection: `[title, callback_url*, epoch]`
- Get Payment Order Collection: `[payment_order_collection_id, epoch]`
- Create Payment Order: `[payment_order_collection_id, bank_account_number, total, epoch]`
- Get Payment Order: `[payment_order_id, epoch]`
- Get Payment Order Limit: `[epoch]`
- Payment Order Callback: `[id, bank_account_number, status, total, reference_id, epoch]`

(`*` = optional; omit from checksum string if not used)

## Completion Criteria

Bill checkout integration is done only when:

- Bill is created server-side from authenticated user flow.
- `callback_url` is configured and receives POST on payment completion.
- X Signature is verified on every callback and redirect before any settlement.
- Duplicate callback is harmless (idempotent settlement).
- Wrong amount, wrong bill ID, or invalid signature cannot mark payment paid.
- Successful sandbox payment marks local payment paid.
- Fulfillment/access unlock happens from verified server-side callback state only.
- Tests cover X Signature verification, duplicate callbacks, wrong amount, and successful settlement.
- Deployment docs mention required secrets (`BILLPLZ_API_KEY`, `BILLPLZ_X_SIGNATURE_KEY`, `BILLPLZ_COLLECTION_ID`), callback URL registration, and sandbox/live key separation.

For V5 Payment Orders or card tokenization, adapt the same completion bar: signed server-side requests, verified callbacks, idempotent state transitions, and tests for invalid signatures/checksums, duplicate notifications, and successful sandbox flow.
