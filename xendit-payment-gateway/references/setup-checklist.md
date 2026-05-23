# Xendit Setup Checklist

Sources read:

- https://docs.xendit.co/
- https://docs.xendit.co/docs/integration-setup-3
- https://docs.xendit.co/api-integration
- https://docs.xendit.co/docs/available-payment-channels
- https://docs.xendit.co/docs/choosing-the-right-payment-product

## Product Choice

Start from docs home when deciding product family:

- Payment Sessions: newer unified checkout/session model; supports hosted payment link mode and embedded Components mode.
- Payments API v3: direct server-side payment orchestration with custom checkout UI.
- Payment Links: hosted payment UI.
- Subscriptions: recurring payments.
- Payouts / Payout Links / Batch Payouts: sending money.
- xenPlatform: platform/sub-account and marketplace fund flows.
- Fraud Detection: fraud protection and risk flows.

## Dashboard Prerequisites

Before implementation:

- Activate target payment channels in Dashboard > Payment Channels before live launch.
- Test mode has channels available by default, but partner/KYC/regional constraints can still limit some flows.
- Create secret API keys in Dashboard > Settings > Developers > API Keys.
- Configure webhook endpoints in Dashboard > Settings > Webhooks.
- Test webhook endpoint from Dashboard and confirm exact test payload is received.

## API Keys

Xendit has separate test and live keys.

Secret API key:

- Server-side only.
- Can perform API requests according to permissions.
- Never commit or expose in frontend/mobile apps.

Public API key:

- Identifies account in frontend/mobile contexts.
- Safe to publish only for supported public-key use cases such as card tokenization.

Permissions:

- `None`: no access.
- `Read`: read/fetch access.
- `Write`: read/write access, needed to create payments/payouts.

Best practice:

- Use environment variables or secret manager.
- Grant minimum product permissions required.
- Keep test and live keys separate.
- Rotate/delete unused or compromised keys.

## Webhook Topics To Configure

For payment projects, configure relevant topics:

- Payment Requests v3 - Payment Status
- Payment Requests v3 - Payment Request Status
- Payment Tokens v3 - Payment Token Status
- Payment Session - Completed
- Payment Session - Expired
- Unified Refunds - Refund Request Succeeded
- Unified Refunds - Refund Request Failed
- Payment Link paid/expired if using Payment Links
- Subscriptions lifecycle if using Subscriptions

## Project Setup Order

1. Pick product path and integration mode.
2. Activate target channels in dashboard.
3. Generate least-privilege test secret key.
4. Create local environment variables.
5. Build server-side Xendit client wrapper.
6. Add webhook endpoint with callback-token verification.
7. Add idempotency/state tables before payment creation.
8. Test happy path, failed payment, expired/abandoned payment, duplicate webhook, out-of-order webhook, refund.
9. Generate live key and switch config only after live channels are active.

