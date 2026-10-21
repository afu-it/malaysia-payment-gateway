---
name: setup-ezypay
description: Set up, build, debug, review, and explain Ezypay payment integrations using official Ezypay developer docs, OAuth 2.0 password-grant tokens, merchant header, customers, Payment Capture Page iframe tokenisation, on-demand invoices, plans and subscriptions, Checkout sessions, Pay Now recovery, refunds and credit notes, webhooks with X-Ezypay-Signature HMAC-SHA1 verification, failed payment handling, settlement reports, sandbox/live setup, and Apple Pay or Google Pay. Use when working with Ezypay Malaysia, MYR card recurring billing, gym or membership subscriptions, Visa and Mastercard tokens, invoice_paid or invoice_past_due webhooks, retries, write-offs, settlement, or production readiness.
---

# Ezypay Payment Gateway

Use this skill for Ezypay subscription billing and card collection work.

## Source

Read `references/ezypay-api.md` first. It holds environments, base URLs, auth, the Malaysia scope, the end-to-end server flow, the full endpoint catalogue, statuses, error codes, amount rules, and the signature summary.

Read `references/ezypay-billing.md` for request and response field tables: customers, payment methods, invoices, refunds, plans, subscriptions, future invoices, checkout, transactions, credit notes, settlements, reports, and partner billing.

Read `references/ezypay-webhooks.md` for every event type, payload shapes, HMAC-SHA1 verification with the official test vector, retries, and duplicates.

Read `references/ezypay-payment-methods.md` for the Payment Capture Page (iframe, postMessage, styling, mobile WebViews), Apple Pay, Google Pay, payment method payloads, sandbox test data, and the Malaysia card failure codes.

Read `references/account-setup.md` for onboarding, credentials, sandbox access, go-live checklists, fees model, and settlement timing.

Read `references/source-map.md` to find the official page for any fact and to see doc coverage.

## Core Workflow

1. Confirm the model: on-demand invoices (you schedule) or Ezypay subscriptions (Ezypay schedules). Checkout is a hosted one-off card page.
2. Confirm credentials: partner `client_id` and `client_secret`, plus per-merchant username, password, and merchant ID. Each branch has its own merchant account.
3. Get a bearer token from the identity host with `grant_type=password` and reuse it until it expires (60 minutes).
4. Create the customer with `POST /v2/billing/customers` and store `id`.
5. Collect the card with the Payment Capture Page iframe; read the token from `postMessage` and check `link_status`.
6. Create a local pending payment, then `POST /v2/billing/invoices` with `externalInvoiceId` as your idempotency key, or create a plan and subscription.
7. Treat Checkout `successUrl` parameters and iframe messages as UX only.
8. Verify the `X-Ezypay-Signature` webhook header (HMAC-SHA1 hex of the raw body with your client key), dedupe on `requestId`, then requery `GET /v2/billing/invoices/{invoiceId}`.
9. Settle on `invoice_paid` with matching invoice id, `externalInvoiceId`, merchant ID, currency `MYR`, and your item amount.
10. Handle `invoice_past_due`: show `failedPaymentReason`, replace invalid payment methods, retry, write off, or record external payment.

## Core Facts

- Sandbox hosts: `identity-sandbox.ezypay.com`, `vault-sandbox.ezypay.com`, `api-sandbox.ezypay.com`, `hosted-global-sandbox.ezypay.com`.
- Production hosts: `identity.ezypay.com`, `vault.ezypay.com`, `api-global.ezypay.com`, `hosted-global.ezypay.com`.
- Billing base: `https://api-sandbox.ezypay.com/v2` (paths start `/billing/...`). Webhooks base: `.../v2/webhooks`.
- Token: `POST https://identity-sandbox.ezypay.com/token`, form-encoded, scope `integrator billing_profile create_payment_method offline_access`.
- Every billing call sends `Authorization: Bearer <token>` and the `merchant: <MERCHANT_ID>` header.
- Malaysia: Visa and Mastercard; Apple Pay and Google Pay are beta. No bank direct debit, no PayTo, no wallet, no FPX or DuitNow.
- 3D Secure is not supported and CVV is not required. Charge in `MYR` only (no cross-border); decimals allowed.
- Amounts are decimal major units in `{ "currency": "MYR", "value": 49.90 }`, not sen.
- Ezypay fees can be added to the invoice as extra items (`transaction_fee`, `load_fee`, `failed_payment_fee`).
- Webhook signature: `X-Ezypay-Signature` = lowercase hex HMAC-SHA1(clientKey, raw body). Retries hourly, up to 24 times.
- Refund: `PUT /v2/billing/invoices/{invoiceId}/refund`, empty body for full; track `credit_note_paid` or `credit_note_failed`.
- Settlement holding period is T+3 business days for Malaysia; frequency weekly or monthly.

## Critical Rules

- Keep `client_secret`, merchant password, client key, and bearer tokens server-side. The PCP iframe URL exposes a token, so build it per session.
- Never mark paid from the iframe message, Checkout `successUrl` query string, or invoice create response (`PROCESSING` is not paid).
- Verify the signature over the exact raw bytes before parsing JSON; use constant-time comparison.
- Register a `clientKey` on every webhook; without one there is nothing to verify.
- Send a unique `externalInvoiceId` per invoice so retries of your own request do not double charge.
- Compare your charged item, not the invoice total, when customer-paid fees are on.
- Store invoice `id`, `documentNumber`, `customerId`, `paymentMethodToken`, and merchant ID before settling.
- Use one merchant account per branch; never share one across branches.
- Do not promise FPX, DuitNow, e-wallets, or 3DS for Malaysia; they are not offered.
- Confirm Apple Pay and Google Pay enablement with Ezypay before launch; both are beta.

## Completion Criteria

An Ezypay integration is done only when:

1. Tokens are fetched server-side, cached, and refreshed or re-created after 60 minutes, within the token rate limit.
2. Customers are created and synced on profile changes; PCP returns a token linked to the customer.
3. Invoices or subscriptions are created server-side with the `merchant` header and a unique `externalInvoiceId`.
4. A webhook per merchant is registered with a `clientKey` and the minimum events from `references/ezypay-webhooks.md`.
5. The webhook handler verifies `X-Ezypay-Signature` over the raw body, dedupes `requestId`, requeries the invoice, and settles once.
6. Failed payments show reason and fees, support retry, write-off, record external payment, and Pay Now.
7. Refunds and chargebacks update local state from credit note events.
8. Tests cover bad signature, replayed `requestId`, wrong merchant, wrong currency or amount, `PAST_DUE`, refund failure, and a paid sandbox invoice using the test cards.
9. Deploy docs list `EZYPAY_*` variables, webhook URL, client key, and sandbox versus production hosts.

## Loophole Review Loop

When asked whether an Ezypay plan is "100% confident", repeat until no factual gaps remain:

1. List trust boundaries: browser iframe, postMessage listener, Checkout redirect, backend token cache, identity host, billing API, vault, webhook receiver, raw-body parser, database, fulfilment, refund job, retry job.
2. Check bypasses: forged postMessage from another origin, fake `successUrl` query, unsigned webhook, signature over re-serialised JSON, wrong client key, sandbox and production mix, wrong merchant header, invoice reused across customers, duplicate `requestId`, `PROCESSING` treated as paid, fee items breaking amount checks, expired token mid-batch, invalid payment method auto-rebill stop, refund with low wallet balance, chargeback reversal after paid.
3. Add fixes: origin check on `message` events, server requery of the invoice, raw-body HMAC with constant-time compare, per-merchant webhook and key, idempotent settlement keyed on invoice id, `externalInvoiceId` uniqueness, event log, separate env config.
4. Re-read the relevant reference file for any gap.
5. State what still depends on Ezypay account setup: enabled methods, fee configuration, retry settings, checkout attempt limit, settlement schedule, Apple Pay domain registration, and Google Pay approval.
