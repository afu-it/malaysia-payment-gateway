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

Use for: server library installation, publishable key usage, Elements initialization, client-side confirmation, React component patterns.

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
