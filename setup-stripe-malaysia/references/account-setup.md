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
- Webhook receiver: verify `Stripe-Signature` header against raw request body using the webhook signing secret and the Stripe library's `constructEvent` method.
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
