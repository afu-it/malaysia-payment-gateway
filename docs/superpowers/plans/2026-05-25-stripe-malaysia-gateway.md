# Stripe Malaysia Payment Gateway Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add `setup-stripe-malaysia` skill to the malaysia-payment-gateway collection, following established provider patterns (SKILL.md, agents/openai.yaml, references/account-setup.md, references/stripe-api.md, references/source-map.md) and updating README.md.

**Architecture:** Single provider skill folder with three reference files, one agent interface file, and one primary skill file. Mirrors the structure of existing skills like `setup-hitpay` and `setup-xendit`. The skill teaches agents to use Stripe as a server-verified state machine: local pending payment, server-side Checkout Session creation, raw-body webhook signature verification, and idempotent settlement.

**Tech Stack:** Markdown, YAML. No code execution required.

---

## File Map

| File | Purpose |
|------|---------|
| `setup-stripe-malaysia/SKILL.md` | Primary skill: frontmatter, quick start, core workflow, integration rules, core facts, critical rules, completion criteria, loophole review loop |
| `setup-stripe-malaysia/agents/openai.yaml` | Minimal agent interface: display name, short description, default prompt |
| `setup-stripe-malaysia/references/source-map.md` | Quick-link map to Stripe docs, API ref, SDKs, CLI, samples |
| `setup-stripe-malaysia/references/account-setup.md` | Registration, API keys (publishable + secret), test/live separation, dashboard webhook config, env vars, MYR/Malaysia caveats |
| `setup-stripe-malaysia/references/stripe-api.md` | Auth, Checkout Sessions, webhooks (Stripe-Signature), CLI listen/trigger, idempotency, test mode, relevant endpoint shapes |
| `README.md` | Add `setup-stripe-malaysia` to skills table and prompt examples |

---

### Task 1: Create folder structure

**Files:**
- Create: `setup-stripe-malaysia/references/` (directory)
- Create: `setup-stripe-malaysia/agents/` (directory)

- [ ] **Step 1: Create directories**

```bash
mkdir -p setup-stripe-malaysia/references setup-stripe-malaysia/agents
```

- [ ] **Step 2: Verify directories exist**

```bash
ls -d setup-stripe-malaysia setup-stripe-malaysia/references setup-stripe-malaysia/agents
```

Expected: All three paths printed with no errors.

- [ ] **Step 3: Commit**

```bash
git add -f setup-stripe-malaysia/
git commit -m "feat: scaffold setup-stripe-malaysia directory structure"
```

Note: Use `-f` for git add since the repo .gitignore may be aggressive.

---

### Task 2: Write agents/openai.yaml

**Files:**
- Create: `setup-stripe-malaysia/agents/openai.yaml`

- [ ] **Step 1: Write the agent interface file**

```yaml
interface:
  display_name: "Setup Stripe Malaysia"
  short_description: "Set up Stripe Malaysia payments"
  default_prompt: "Use $setup-stripe-malaysia to design or debug a Stripe Malaysia payment gateway integration."
```

- [ ] **Step 2: Verify file was written**

```bash
cat setup-stripe-malaysia/agents/openai.yaml
```

- [ ] **Step 3: Commit**

```bash
git add -f setup-stripe-malaysia/agents/openai.yaml
git commit -m "feat: add setup-stripe-malaysia agent interface"
```

---

### Task 3: Write references/source-map.md

**Files:**
- Create: `setup-stripe-malaysia/references/source-map.md`

- [ ] **Step 1: Write the source map reference**

```markdown
# Stripe Source Map

Use this map to pick the right source before answering or coding.

## Product and Overview

- Docs home: `https://docs.stripe.com`
- Payments overview: `https://docs.stripe.com/payments`
- Get started: `https://docs.stripe.com/get-started/development-environment/overview`
- Samples: `https://docs.stripe.com/samples`
- GitHub samples: `https://github.com/stripe-samples`

Use for: product positioning, integration path selection, sample code reference.

## API Reference

- API reference root: `https://docs.stripe.com/api`
- Checkout Sessions API: `https://docs.stripe.com/api/checkout/sessions`
- Payment Intents API: `https://docs.stripe.com/api/payment_intents`

Use for: endpoint schemas, request/response fields, HTTP auth format, idempotency, parameter details.

## Checkout and Payments

- Build a payments page: `https://docs.stripe.com/payments/checkout`
- Checkout Sessions quickstart: `https://docs.stripe.com/payments/quickstart-checkout-sessions`
- Payment Intents guide: `https://docs.stripe.com/payments/payment-intents`
- Elements reference: `https://docs.stripe.com/payments/elements`
- Payment Links (no-code): `https://docs.stripe.com/no-code/payment-links`

Use for: choosing Checkout Session vs Payment Intents, hosted page vs embedded form vs Elements, payment link creation, redirect behavior, client-side integration patterns.

## Webhooks

- Webhooks overview: `https://docs.stripe.com/webhooks`
- Webhook signature verification: `https://docs.stripe.com/webhooks#verify-official-libraries`
- Event types reference: `https://docs.stripe.com/api/events/types`

Use for: `Stripe-Signature` header format, HMAC verification, raw body requirement, event subscription, idempotency via event IDs, retry behavior, IP allowlisting, replay attack prevention.

## Stripe CLI

- CLI overview: `https://docs.stripe.com/stripe-cli`
- CLI reference: `https://docs.stripe.com/cli`

Use for: `stripe listen --forward-to`, `stripe trigger`, local webhook testing, webhook signing secret, test event generation, API calls from terminal.

## SDKs

- Server SDKs: `https://docs.stripe.com/sdks`
- Stripe.js: `https://docs.stripe.com/js`
- React Stripe.js: `https://docs.stripe.com/js/react`

Use for: server library installation, publishable key usage, Elements initialization, client-side confirmation, React component patterns (CheckoutElementsProvider, etc.).

## Development Environment

- Development environment: `https://docs.stripe.com/get-started/development-environment`
- Test mode setup: `https://docs.stripe.com/get-started/development-environment/overview`
- API keys: `https://docs.stripe.com/keys`
- Dashboard webhooks: `https://dashboard.stripe.com/webhooks`

Use for: test API keys, live API keys, publishable keys, restricted keys, webhook endpoint registration, environment separation, Stripe CLI setup.

## Malaysia-Specific Notes

Stripe supports Malaysia accounts. Key considerations:
- Available payment methods: cards, FPX (bank redirect), GrabPay, and other methods available in the MY market.
- Currency: MYR is supported.
- Check the Dashboard under Settings > Payment Methods to confirm currently active methods for your Malaysia Stripe account.
- FPX requires a Malaysia Stripe account and lists available banks at checkout.
- Verify current payment method availability, fees, and settlement timing in the Stripe Dashboard and official docs before claiming production readiness.
```

- [ ] **Step 2: Verify file was written**

```bash
cat setup-stripe-malaysia/references/source-map.md
```

- [ ] **Step 3: Commit**

```bash
git add -f setup-stripe-malaysia/references/source-map.md
git commit -m "feat: add Stripe source-map reference"
```

---

### Task 4: Write references/account-setup.md

**Files:**
- Create: `setup-stripe-malaysia/references/account-setup.md`

- [ ] **Step 1: Write the account setup reference**

```markdown
# Stripe Account Setup

Use this file when user asks how to register, what keys are needed, or what dashboard setup is required before coding Stripe.

## Accounts

- Create/access Stripe Dashboard at `https://dashboard.stripe.com`.
- No referral link is configured for Stripe.
- Use test mode for development. Test mode keys start with `sk_test_` and `pk_test_`.
- Use live mode only after webhook URL, payment methods, and account settings are ready.
- Keep test and live keys, webhooks, payment objects, and reporting separate.
- Malaysia accounts: register at `https://dashboard.stripe.com/register` with Malaysian business details.

## Credentials Needed

- **Secret API key** (`sk_live_...` or `sk_test_...`): server-side API authentication. Used for creating Checkout Sessions, PaymentIntents, and all API requests from the backend.
- **Publishable API key** (`pk_live_...` or `pk_test_...`): client-side Stripe.js initialization. Used in `Stripe(publishableKey)` calls and for Elements.
- **Webhook signing secret** (`whsec_...`): endpoint-specific secret used to verify `Stripe-Signature` header on incoming webhooks. Found in Dashboard > Webhooks after creating an endpoint.
- **Restricted keys** (optional): scoped API keys with limited permissions for specific operations.

## Where Used

- API calls: HTTP Bearer auth with secret key. Header: `Authorization: Bearer {secret_key}`.
- Stripe.js: initialize with publishable key only. Never pass secret key to the client.
- Webhook receiver: verify `Stripe-Signature` header against raw request body using the webhook signing secret and the Stripe library's `constructEvent` (or `construct_event`) method.
- CLI: `stripe login` or `STRIPE_API_KEY` environment variable for local testing.

## Recommended Env Vars

```text
STRIPE_ENV=test
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_WEBHOOK_URL=https://example.com/api/payments/stripe/webhook
STRIPE_RETURN_URL=https://example.com/checkout/result
```

Use separate values for production. For production:

```text
STRIPE_ENV=live
STRIPE_SECRET_KEY=sk_live_...
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

## Setup Checklist

1. Register/access Stripe Dashboard at `https://dashboard.stripe.com`.
2. Enable test mode using the toggle in the Dashboard.
3. Copy test publishable key and test secret key from Dashboard > Developers > API keys.
4. Enable desired payment methods in Dashboard > Settings > Payment Methods.
5. Install Stripe CLI: follow `https://docs.stripe.com/stripe-cli`.
6. Run `stripe login` to authenticate CLI with your Stripe account.
7. Start local webhook forwarding: `stripe listen --forward-to localhost:3000/api/payments/stripe/webhook`.
8. Note the webhook signing secret printed by `stripe listen` for local testing.
9. Test creating a Checkout Session from server.
10. Test triggering webhook events with `stripe trigger checkout.session.completed`.
11. Register production webhook endpoint in Dashboard > Webhooks after deployment.
12. Generate live keys only after production webhook setup and payment method activation.

## Do Not Assume

- Do not expose secret API key in frontend/mobile code or environment variables prefixed with `NEXT_PUBLIC_` or similar.
- Do not pass the raw request body through JSON parsing before webhook signature verification. Stripe's verification functions need the raw body bytes.
- Do not fulfil from client redirect (`success_url`) alone.
- Do not mix test and live keys or webhook secrets.
- Do not hardcode the webhook signing secret from `stripe listen` — it changes each time the listener restarts. Use it only for local testing.
```

- [ ] **Step 2: Verify file was written**

```bash
cat setup-stripe-malaysia/references/account-setup.md
```

- [ ] **Step 3: Commit**

```bash
git add -f setup-stripe-malaysia/references/account-setup.md
git commit -m "feat: add Stripe account-setup reference"
```

---

### Task 5: Write references/stripe-api.md

**Files:**
- Create: `setup-stripe-malaysia/references/stripe-api.md`

- [ ] **Step 1: Write the API reference**

Write a file at `setup-stripe-malaysia/references/stripe-api.md` with the following content:

```markdown
# Stripe API Reference

Use this reference for factual Stripe API guidance. Verify current Dashboard settings, payment method activation, pricing, and operational status when those details matter.

## Source URLs

- `https://docs.stripe.com/api`
- `https://docs.stripe.com/payments`
- `https://docs.stripe.com/payments/checkout`
- `https://docs.stripe.com/payments/quickstart-checkout-sessions`
- `https://docs.stripe.com/webhooks`
- `https://docs.stripe.com/webhooks#verify-official-libraries`
- `https://docs.stripe.com/stripe-cli`
- `https://docs.stripe.com/sdks`
- `https://docs.stripe.com/js`
- `https://docs.stripe.com/get-started/development-environment/overview`
- `https://github.com/stripe-samples`

## Integration Surfaces

Stripe supports multiple payment collection surfaces:

- **Checkout Sessions API** (recommended): prebuilt hosted page, embedded form, or embedded Elements. Handles payment method display, tax, shipping, discounts, subscriptions, Adaptive Pricing. Minimal code.
- **Payment Intents API**: lower-level API for custom payment flows. You manage the full checkout UI and state. Only use when Checkout Sessions cannot meet requirements.
- **Payment Links** (no-code): Dashboard-created payment pages. No integration code needed.
- **Elements**: Stripe.js UI components (Payment Element, Express Checkout Element, Address Element, etc.) for custom checkout forms.
- **Invoicing API**: for bill-based payment collection.
- **Subscriptions API**: for recurring billing.

For a standard ecommerce checkout, the default path is Checkout Sessions API (full page or embedded) plus webhook settlement.

## Environments

Production API base:
```text
https://api.stripe.com
```

All requests use the same base URL. The API key determines whether the request operates in test mode or live mode.

- Test mode: use `sk_test_...` keys. No real money moves. Test card numbers available.
- Live mode: use `sk_live_...` keys. Real payments flow.

Stripe CLI connects to test mode by default.

## Authentication

API requests use HTTP Basic Auth or Bearer token with the secret key:

```text
Authorization: Bearer sk_test_...
```

Or HTTP Basic Auth where username is the secret key and password is empty:
```text
Authorization: Basic {base64(sk_test_...:)}
```

Keep secret keys server-side only. Publishable keys (`pk_test_...`, `pk_live_...`) are safe for client-side Stripe.js initialization.

## Checkout Sessions Flow

Recommended integration flow:

1. Create a local pending payment/order in your database.
2. Call Stripe `POST /v1/checkout/sessions` from your server with the secret key.
3. Store Stripe session ID, local reference, amount, currency, and initial status.
4. Return the session `url` to the client and redirect the customer, or return the `client_secret` for embedded/Element mode.
5. Receive and validate the webhook `checkout.session.completed` on your server.
6. Settle idempotently after strict local/provider matching.
7. Use `GET /v1/checkout/sessions/{id}` as fallback when webhook delivery is missing or failed.

### Create Checkout Session

```text
POST https://api.stripe.com/v1/checkout/sessions
```

Headers:
```text
Authorization: Bearer {secret_key}
Content-Type: application/x-www-form-urlencoded
```

Important fields for `mode: payment`:

| Field | Notes |
| --- | --- |
| `mode` | `payment`, `subscription`, or `setup` |
| `line_items[][price_data][currency]` | Three-letter currency code such as `myr` |
| `line_items[][price_data][product_data][name]` | Product name |
| `line_items[][price_data][unit_amount]` | Amount in smallest currency unit (e.g., cents for MYR, so RM 10.00 = 1000) |
| `line_items[][quantity]` | Quantity |
| `success_url` | Browser redirect after successful payment. Stripe appends `?session_id={CHECKOUT_SESSION_ID}` |
| `cancel_url` | Browser redirect if customer cancels |
| `client_reference_id` | Your internal order/reference ID for reconciliation |
| `customer_email` | Pre-fill customer email |
| `metadata` | Key-value pairs for your own reference |
| `payment_method_types` | List of enabled payment method types (e.g., `[card, fpx, grabpay]`) |
| `expires_at` | Session expiry timestamp |

The response includes an `id` (session ID), `url` (hosted checkout page), and `status`.

### Retrieve Checkout Session

```text
GET https://api.stripe.com/v1/checkout/sessions/{id}
```

Use to fetch current session state including `payment_status` (`paid`, `unpaid`, `no_payment_required`) and `payment_intent` ID.

## Payment Intents (Advanced)

Only use Payment Intents when Checkout Sessions cannot meet requirements. Stripe recommends Checkout Sessions for most integrations.

```text
POST https://api.stripe.com/v1/payment_intents
```

Key fields: `amount`, `currency`, `payment_method_types`, `metadata`.

The response includes `id`, `client_secret`, `status`. Pass `client_secret` to the client for Elements-based confirmation.

## Webhooks

### Signature Verification

Stripe signs webhook events with HMAC-SHA256. The signature is in the `Stripe-Signature` header with format:

```
t=1492774577,v1=5257a869e7...,v0=6ffbb59b23...
```

Verification requires:
1. The raw request body (not parsed JSON).
2. The `Stripe-Signature` header value.
3. The webhook endpoint signing secret (`whsec_...`).

Use Stripe's official library:

**Node.js:**
```javascript
import Stripe from 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

const sig = request.headers['stripe-signature'];
const event = stripe.webhooks.constructEvent(
  rawBody, // raw request body as Buffer or string
  sig,
  process.env.STRIPE_WEBHOOK_SECRET
);
```

**Python:**
```python
import stripe
event = stripe.Webhook.construct_event(
    payload=raw_body,       # raw request body as bytes
    sig_header=sig_header,  # Stripe-Signature header
    secret=webhook_secret   # whsec_...
)
```

### Key Event Types

| Event | Use |
| --- | --- |
| `checkout.session.completed` | Checkout Session payment succeeded. Fulfil order here. |
| `checkout.session.async_payment_succeeded` | Async payment method (e.g., FPX bank redirect) confirmed. |
| `checkout.session.async_payment_failed` | Async payment method failed. |
| `payment_intent.succeeded` | PaymentIntent succeeded (for Payment Intents path). |
| `payment_intent.payment_failed` | PaymentIntent failed. |

### Idempotency

- Stripe events have unique `id` fields. Log processed event IDs to prevent duplicate handling.
- Stripe may deliver events out of order. Don't depend on event sequence.
- Stripe retries failed deliveries for up to 3 days with exponential backoff in live mode.
- Return a `2xx` response quickly; defer heavy processing.

### Local Testing with CLI

Listen and forward to local endpoint:
```bash
stripe listen --forward-to localhost:3000/api/payments/stripe/webhook
```

Trigger test events:
```bash
stripe trigger checkout.session.completed
stripe trigger payment_intent.succeeded
```

The `stripe listen` command prints a temporary webhook signing secret. Use it locally only — it changes each restart. Production endpoints use the secret from the Dashboard.

## Stripe.js Client-Side

Initialize with publishable key only:
```javascript
import { loadStripe } from '@stripe/stripe-js';
const stripe = await loadStripe('pk_test_...');
```

For Checkout Sessions redirect (hosted page):
```javascript
const { error } = await stripe.redirectToCheckout({ sessionId });
```

For Checkout Sessions with embedded Elements:
```javascript
import { CheckoutElementsProvider } from '@stripe/react-stripe-js/checkout';
// Use client_secret from Checkout Session
```

## Test Mode

- Test card: `4242 4242 4242 4242` with any future expiry, any CVC, any postal code.
- Test FPX: use any bank during test checkout.
- Test webhooks: use `stripe trigger` or manually create objects in Dashboard.
- Test mode is free. No real charges.
```

- [ ] **Step 2: Verify file was written**

```bash
cat setup-stripe-malaysia/references/stripe-api.md
```

- [ ] **Step 3: Commit**

```bash
git add -f setup-stripe-malaysia/references/stripe-api.md
git commit -m "feat: add Stripe API reference"
```

---

### Task 6: Write SKILL.md

**Files:**
- Create: `setup-stripe-malaysia/SKILL.md`

- [ ] **Step 1: Write the primary skill file**

Write `setup-stripe-malaysia/SKILL.md` with the following content:

```markdown
---
name: setup-stripe-malaysia
description: Set up, build, debug, review, and explain Stripe payment gateway integrations for Malaysia. Use when working with Stripe Checkout Sessions, Payment Intents, Payment Links, Elements, webhooks, Stripe.js, Stripe CLI, refunds, sandbox/live setup, MYR payments, cards, FPX, GrabPay, and production readiness.
---

# Stripe Malaysia Payment Gateway

Use this skill for Stripe payment gateway integration work in Malaysia.

## Source

Read `references/stripe-api.md` before giving factual API guidance or writing integration code. It contains source URLs, environment bases, auth, Checkout Sessions flow, Payment Intents guidance, webhooks (Stripe-Signature verification), Stripe.js initialization, CLI testing, test mode cards, and settlement/security cautions.

Read `references/account-setup.md` when user asks how to register, which credentials are needed, where keys come from, sandbox setup, dashboard setup, payment method activation, Malaysia-specific methods, pricing, or operational status.

Read `references/source-map.md` when choosing which Stripe docs page applies to API checkout, webhooks, CLI, SDKs, Elements, Payment Links, subscriptions, or samples.

## Core Workflow

1. Identify environment: test mode or live mode.
2. Identify integration surface: Checkout Sessions (hosted page, embedded form, or embedded Elements), Payment Intents, Payment Links, or Elements-only.
3. Confirm dashboard setup: API keys (publishable + secret), webhook endpoint, webhook signing secret, enabled payment methods, and environment-specific configuration.
4. Create a local pending payment before creating the Stripe Checkout Session or PaymentIntent.
5. Create the Checkout Session or PaymentIntent server-side with the secret key.
6. For hosted Checkout: redirect the customer to the returned `url`. For embedded: pass the `client_secret` to the frontend.
7. Treat `success_url` redirect as browser UX only; do not fulfil from redirect alone.
8. Verify webhook `Stripe-Signature` using the Stripe library's `constructEvent` method over the raw request body with the webhook signing secret.
9. Settle idempotently only after session/PaymentIntent ID, local reference, amount, currency, payment status, and environment match expectations.
10. Use session/PaymentIntent retrieval as fallback when webhooks are delayed or failed; do not replace webhooks with polling.

## Core Facts

- API base: `https://api.stripe.com`.
- Auth: HTTP Bearer `Authorization: Bearer {secret_key}`.
- Secret keys start with `sk_live_` or `sk_test_`.
- Publishable keys start with `pk_live_` or `pk_test_`. These are safe for client-side use.
- Checkout Sessions endpoint: `POST /v1/checkout/sessions`.
- Payment Intents endpoint: `POST /v1/payment_intents` (only when Checkout Sessions cannot meet requirements).
- Webhook signing secrets start with `whsec_`. Each webhook endpoint has its own secret.
- Webhook signatures use the `Stripe-Signature` header with HMAC-SHA256 over `{timestamp}.{raw_body}`.
- For Checkout Sessions, Stripe recommends fulfilling orders from the `checkout.session.completed` webhook event.
- For async payment methods (like FPX bank redirect), use `checkout.session.async_payment_succeeded`.
- Checkout Session `payment_status` values: `paid`, `unpaid`, `no_payment_required`.
- Test cards: `4242 4242 4242 4242` (Visa), any future expiry, any CVC.
- Stripe CLI: `stripe listen --forward-to` for local webhook testing, `stripe trigger` for event generation.
- Malaysia Stripe accounts support MYR, cards, FPX, and GrabPay. Verify current availability in the Dashboard.
- Stripe supports 125+ payment methods globally.

## Critical Rules

- Keep secret API keys and webhook signing secrets server-side only.
- Never use secret keys in frontend code, mobile apps, or client-side environment variables.
- Do not mark paid from `success_url` redirect or client-side status alone.
- Always validate webhook signatures using the Stripe library's `constructEvent`/`construct_event` with the raw request body before parsing JSON.
- Use the exact webhook signing secret matching the endpoint that received the event.
- Store provider session ID, local reference, expected amount, expected currency, and environment before redirecting the customer.
- Make webhook processing idempotent by logging processed event IDs; duplicate events or retries must not double-fulfil.
- Confirm amount, currency, session/PaymentIntent ID, local reference, and payment status before marking paid.
- Use separate webhook URLs for test mode and live mode.
- Use webhook confirmation as the default settlement path and API retrieval only as a fallback.
- Do not hardcode the temporary webhook secret from `stripe listen` — it changes each restart. Use it only for local development.
- Verify live payment method availability, activation, fees, settlement timing, and current incidents in the Dashboard/docs/status page before claiming production certainty.
- Stripe recommends Checkout Sessions over Payment Intents for most integrations. Only use Payment Intents when the user explicitly needs custom payment UI or deeply owned checkout state.

## Completion Criteria

A Stripe integration is done only when:

1. Checkout Session or PaymentIntent creation runs server-side with the secret key.
2. Redirect checkout, embedded checkout, or Elements returns only provider state to the backend; it does not fulfil access directly.
3. Webhook endpoint is registered in the Stripe Dashboard or tested with `stripe listen`.
4. Webhook handler verifies `Stripe-Signature` over the raw request body with the correct webhook signing secret.
5. Settlement checks session/PaymentIntent ID, local reference, amount, currency, and payment status.
6. Duplicate webhooks and event retries are idempotent.
7. Tests cover invalid signature, duplicate webhook/event replay, wrong amount/currency/reference, failed/expired status, and successful completed payment.
8. Deployment docs include `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`, `STRIPE_WEBHOOK_SECRET`, redirect URL, webhook URL, and test/live separation.

## Loophole Review Loop

When asked whether a Stripe strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, checkout page, backend payment creator, Stripe API, webhook receiver, raw-body parser, database, fulfilment job, refund job, dashboard settings, status page, and payment method activation.
2. Check bypasses: fake success redirect, forged webhook, signature over parsed JSON instead of raw body, wrong webhook secret, test/live mode mix, wrong API key, wrong amount, wrong currency, wrong reference, stale status poll, duplicate webhook, webhook retry after partial processing, inactive payment method, unavailable Stripe service, duplicate refund.
3. Add fixes: server-side Checkout Session/PaymentIntent creation, local pending payment first, raw-body signature verification with Stripe library, strict amount/currency/reference checks, durable event log, idempotency keys, separate environment config, IP allowlisting where needed, status page check, API retrieval fallback, and dashboard capability verification.
4. Re-read `references/stripe-api.md` sections relevant to any gap.
5. State remaining uncertainty if it depends on current account verification, enabled payment methods, activation windows, settlement timing, pricing, or live operational status.
```

- [ ] **Step 2: Verify file was written**

```bash
cat setup-stripe-malaysia/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add -f setup-stripe-malaysia/SKILL.md
git commit -m "feat: add setup-stripe-malaysia SKILL.md"
```

---

### Task 7: Update README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add `setup-stripe-malaysia` to the skills table**

Add this row to the skills table after `setup-hitpay`:

```markdown
| `setup-stripe-malaysia` | Stripe Malaysia Checkout, Payment Element, webhooks, Stripe CLI, MYR, cards, FPX |
```

The current skills table ends with `setup-hitpay`. Insert before the closing row separator.

- [ ] **Step 2: Add `setup-stripe-malaysia` to the prompt examples section**

Add this after the `setup-hitpay` prompt example:

```text
Use $setup-stripe-malaysia to integrate Stripe Malaysia checkout, webhook verification, and idempotent settlement.
```

- [ ] **Step 3: Verify README was updated**

```bash
git diff README.md
```

- [ ] **Step 4: Commit**

```bash
git add -f README.md
git commit -m "docs: add setup-stripe-malaysia to README skills table and examples"
```

---

## Verification

After all tasks are complete:

```bash
ls -la setup-stripe-malaysia/
ls -la setup-stripe-malaysia/references/
ls -la setup-stripe-malaysia/agents/
cat setup-stripe-malaysia/SKILL.md
```

All six files should exist:
- `setup-stripe-malaysia/SKILL.md`
- `setup-stripe-malaysia/agents/openai.yaml`
- `setup-stripe-malaysia/references/source-map.md`
- `setup-stripe-malaysia/references/account-setup.md`
- `setup-stripe-malaysia/references/stripe-api.md`
- `README.md` (updated)
