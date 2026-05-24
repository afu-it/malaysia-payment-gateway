---
name: malaysia-payment-gateway
description: Build, debug, review, and explain Malaysia payment gateway integrations. Use when implementing checkout, payment links, hosted payment pages, redirects, callbacks, webhooks, HMAC/signature verification, idempotent settlement, paid-access unlocking, refunds, reconciliation, sandbox/live setup, or provider switching for Malaysian gateways such as CHIP Collect, Curlec/Razorpay, Xendit Malaysia, Bayarcash, BCL Pay, toyyibPay, Billplz, FPX, DuitNow QR, cards, or e-wallets.
---

# Malaysia Payment Gateway

Use this skill to add production-grade payment collection to an app, not only to make a checkout button. Treat payment as a state machine: local intent first, gateway session second, verified server-side settlement last.

## Quick Start

1. Identify provider, product, currency, amount unit, and environment.
2. Read [references/integration-checklist.md](references/integration-checklist.md).
3. If provider-specific guidance is needed, read [references/provider-patterns.md](references/provider-patterns.md).
4. Before marking complete, check [references/security-settlement.md](references/security-settlement.md).

## Implementation Workflow

### 1. Map Existing App

Find:

- Auth/session helper.
- Database access layer.
- Existing order, invoice, subscription, entitlement, or payment tables.
- Server route pattern.
- Environment/secret handling.
- Test style and deployment target.

Do not start with provider API calls before local payment records and settlement rules are clear.

### 2. Create Local Payment Intent

Create or reuse a `payments` table with at least:

- `id`
- `user_id` or customer id
- `product` or order id
- `amount_original`
- `discount_amount`
- `amount_paid`
- `currency`
- `provider`
- `status`
- `gateway_ref`
- `gateway_status`
- `gateway_payload`
- `created_at`
- `updated_at`
- `paid_at`

Use `pending` before calling the gateway. Store gateway id after session/order/purchase creation. Add a unique index on `(provider, gateway_ref)` where `gateway_ref` is not null.

### 3. Create Checkout Server-Side

Create gateway checkout from a server route. Never expose secret keys. Return only:

- Hosted checkout URL, or
- Public key plus gateway order/session id required by browser SDK.

### 4. Verify Callback/Webhook

Browser redirect is user experience only. Webhook/callback verification must happen server-side with raw body when required.

After verification, fetch current gateway object from provider API when possible. Settle from fetched server-side state, not from client-sent success data alone.

### 5. Settle Idempotently

Before marking paid, verify:

- Local payment exists.
- Provider matches.
- Gateway reference matches.
- Status is final paid/captured/succeeded.
- Amount matches expected amount.
- Currency is expected, usually `MYR`.
- Product/order belongs to local user.
- Payment is still `pending` or already `paid`.

Then update status to `paid`, set `paid_at`, unlock entitlement/order fulfillment, and create referral/reward side effects once.

## Provider Choice

Use project needs to choose:

- CHIP Collect: Malaysian checkout, FPX/DuitNow/cards/e-wallets, purchase/webhook flow.
- Curlec/Razorpay: Orders API plus Checkout.js, FPX/redirect methods, Razorpay-style signatures.
- Xendit Malaysia: Hosted payment sessions or payment requests, FPX and e-wallet channels.
- Bayarcash/BCL/toyyibPay/Billplz: Malaysian payment portals/API flows; verify exact docs/live dashboard before schemas.

If latest provider API details, channel availability, fees, dashboard paths, or webhook event names matter, check official current docs before coding.

## Completion Criteria

Payment integration is done only when:

- Checkout can be created from authenticated user flow.
- Failed/missing config returns clear server error.
- Webhook/callback rejects invalid signature.
- Duplicate webhook is harmless.
- Wrong amount/currency/provider cannot mark paid.
- Successful sandbox payment marks local payment paid.
- Fulfillment/access unlock happens from verified server state.
- Tests cover helpers and settlement edge cases.
- Deployment docs mention required secrets, webhook URLs, and migrations.
