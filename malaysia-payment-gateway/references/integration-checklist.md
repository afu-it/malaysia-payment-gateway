# Integration Checklist

## Discovery

- Find framework, routing style, database client, auth helper, environment loading, and test runner.
- Find existing payments/orders/subscriptions/entitlements.
- Find deployment target, because raw-body handling and secrets differ by platform.
- Identify provider, environment, currency, amount unit, supported channels, and go-live requirements.

## Data Model

Use local payment intent as source of truth for app fulfillment.

Recommended columns:

- `id`: local UUID/correlation key.
- `user_id`: owner.
- `product` or `order_id`: purchasable thing.
- `amount_original`, `discount_amount`, `amount_paid`: store in app business unit or minor unit consistently.
- `currency`: use `MYR` unless provider/account supports otherwise.
- `provider`: `chip`, `curlec`, `xendit`, `bayarcash`, `bcl`, or project value.
- `status`: `pending`, `paid`, `rejected`, `expired`, `refunded`.
- `gateway_ref`: provider order/session/purchase/payment id.
- `gateway_status`: provider status.
- `gateway_payload`: compact JSON for audit/debug.
- `created_at`, `updated_at`, `paid_at`.

Add:

- Unique `(provider, gateway_ref)` when `gateway_ref` is not null.
- Gateway event table with unique `(provider, event_id)` for webhook dedupe.

## Checkout Route

Flow:

1. Require authenticated user.
2. Validate product/amount server-side.
3. Apply discounts server-side.
4. Insert local `pending` payment.
5. Create provider checkout/order/session using server secret.
6. Store provider id, status, checkout URL/payload.
7. Return checkout URL or public browser SDK options.

Never accept amount from browser as final truth.

## Webhook/Callback Route

Flow:

1. Read raw body when signature requires it.
2. Verify signature/token before parsing or trusting body.
3. Insert event id with `INSERT OR IGNORE` or equivalent.
4. Extract provider object id.
5. Fetch current object from provider API when available.
6. Call one settlement function.
7. Return 2xx quickly for valid duplicate/ignored events.

## Settlement Function

Settlement should be isolated and testable:

1. Ignore non-final status.
2. Load local payment by local reference.
3. Check provider, gateway ref, amount, currency, status.
4. If already paid, return idempotent success.
5. If pending, mark paid and set `paid_at`.
6. Unlock access/order fulfillment.
7. Create side effects once.

## Verification

Run:

- Unit tests for amount conversion, payload builders, signature helpers, settlement rules.
- Typecheck/lint/build.
- Sandbox happy path.
- Sandbox invalid signature.
- Duplicate webhook replay.
- Wrong amount/currency simulation.
