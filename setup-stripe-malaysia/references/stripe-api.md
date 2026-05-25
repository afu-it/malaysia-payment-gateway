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

API requests use HTTP Bearer with the secret key:

```text
Authorization: Bearer sk_test_...
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
| `line_items[][price_data][currency]` | Three-character currency code such as `myr` |
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

For Checkout Sessions with embedded Elements, use `client_secret` from the Checkout Session with `CheckoutElementsProvider` from `@stripe/react-stripe-js/checkout`.

## Test Mode

- Test card: `4242 4242 4242 4242` with any future expiry, any CVC, any postal code.
- Test FPX: use any bank during test checkout.
- Test webhooks: use `stripe trigger` or manually create objects in Dashboard.
- Test mode is free. No real charges.
