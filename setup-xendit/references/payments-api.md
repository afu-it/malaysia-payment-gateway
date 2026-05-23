# Xendit Payments API

Sources read:

- https://docs.xendit.co/apidocs#xenplatform
- https://docs.xendit.co/docs/en/payments-via-api-overview
- https://docs.xendit.co/docs/how-payments-api-work
- https://docs.xendit.co/docs/payment-sessions-overview
- https://docs.xendit.co/docs/components-overview/

## Core Model

Xendit API is REST-oriented, HTTPS-only, returns JSON, and uses HTTP response codes for errors. Test-environment requests do not interact with banking networks or incur charges.

Authentication uses HTTP Basic Auth:

- Username: secret API key.
- Password: empty.
- Manual format before Base64: `{secret_api_key}:`
- Header: `Authorization: Basic <base64(secret_api_key + ":")>`

Keep secret keys server-side. Never expose secret keys in frontend/mobile apps.

## Primary Endpoints

Payments API v3 centers on:

- `POST /v3/payment_requests`: initiate and orchestrate collection.
- `POST /v3/payment_tokens`: save payment information for reuse.
- `/refunds`: return money to payer.

Most new collection flows should start with `v3/payment_requests`.

Payment Sessions are another current payment path:

- Create a secure, stateful Payment Session on Xendit.
- Use `session_type` to choose `PAY` or `SAVE`.
- Use mode `PAYMENT_LINK` for Xendit-hosted checkout.
- Use mode `COMPONENTS` for embedded secure Xendit-managed fields.
- Listen to session webhooks, especially `payment_session.completed` and `payment_session.expired`.

Choose Payment Sessions when user wants a hosted checkout page or embedded PCI-scoped-down components. Choose Payments API v3 when user needs a custom server-orchestrated flow with direct endpoint handling.

## Request Types

`PAY`
: Collect one regular payment from end user.

`PAY_AND_SAVE`
: Collect one payment and save reusable payment information. Payment token is delivered through webhook.

`PAY with token`
: Collect one payment using previously saved payment token.

`REUSABLE_PAYMENT_CODES`
: Collect multiple payments using an allocated reusable payment code, such as static QR or fixed account-like channel code.

`SAVE` on `/payment_tokens`
: Save end-user payment information without immediately collecting payment.

Payment Session `PAY`
: Collect payment in session flow. Can optionally save method.

Payment Session `SAVE`
: Save method without charging customer.

Payment Session `PAY_AND_SAVE`
: Used in Components docs for pay and save flow. Verify exact enum availability in current endpoint docs before production.

## Payment Request Skeleton

Example shape from docs:

```json
{
  "reference_id": "90392f42-d98a-49ef-a7f3-abcezas123",
  "type": "PAY",
  "country": "PH",
  "currency": "PHP",
  "request_amount": 10000.01,
  "capture_method": "AUTOMATIC",
  "channel_code": "GCASH",
  "channel_properties": {
    "failure_return_url": "https://example.com/failure",
    "success_return_url": "https://example.com/success"
  },
  "description": "Order description",
  "metadata": {
    "order_id": "order_123"
  }
}
```

Response includes Xendit identifiers, status, and sometimes `actions`.

## Actions

Payment flows may require customer action. Handle returned `actions` instead of hard-coding one flow.

Important action patterns:

- `REDIRECT_CUSTOMER`: redirect payer to hosted page, wallet app, bank flow, or 3DS flow.
- Other channel-specific actions may exist; treat unknown action types as integration blockers and verify docs before production.

## Components Flow

For embedded checkout with Xendit Components:

1. Server creates Payment Session with `mode = "COMPONENTS"`.
2. Server includes `components_configuration.origins` with allowed frontend origins.
3. Xendit returns `components_sdk_key`.
4. Server returns `components_sdk_key` to frontend only for that session.
5. Frontend initializes Xendit Components with that key.
6. Frontend mounts active channel component.
7. Frontend calls submit on customer action.
8. Server treats webhooks as authoritative for order state.

Important:

- `components_sdk_key` is short-lived and session-specific.
- Do not store or log `components_sdk_key`.
- Never expose secret API key to frontend.
- As of docs read, Xendit Components availability mentioned `CARDS`; verify current channel support before promising embedded support for other channels.

## State Handling

Store at least:

- merchant `reference_id`
- Xendit `payment_request_id`
- Xendit payment or capture IDs when present
- status
- selected channel and country/currency
- raw webhook payload hash or event ID if available

Fulfill orders only after verified successful state. For action-required flows, synchronous API response is not final confirmation.

## Common Build Pattern

1. Server creates payment request with validated cart/order amount.
2. Persist request and Xendit IDs before responding to frontend.
3. Frontend follows returned action, usually redirect.
4. Webhook updates order/payment state.
5. Return page shows pending/success based on server-side state, not URL alone.
6. Reconciliation job polls or searches by Xendit ID/reference if webhook is delayed.
