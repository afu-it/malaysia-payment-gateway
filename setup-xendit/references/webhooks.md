# Xendit Webhooks

Source read:

- https://docs.xendit.co/docs/handling-webhooks

## Required Behavior

Handle Xendit webhooks on server-side endpoints only. Do not process webhook trust decisions in client apps.

Verify sender by checking `x-callback-token` header against webhook token from Xendit Dashboard webhook settings. Keep token secret.

Respond quickly with 2xx. Put slow operations into queue/background work after durable receipt.

## Duplicates

Duplicate webhooks can happen when acknowledgement fails or provider recovery resends events. Handler must be idempotent.

Use unique Xendit IDs to guard state transitions:

- `payment_id`
- `capture_id`
- product-specific event/object IDs when present

Avoid duplicate fulfillment, duplicate balance movement, duplicate email, and duplicate inventory release.

## Retry Model

If endpoint does not return 2xx, Xendit retries delivery up to six times with exponential backoff. Dashboard may allow manual webhook retry.

## Ordering

Do not assume fixed webhook order. Events may arrive out of sequence due to network or provider delays.

Safer pattern:

1. Store webhook raw payload and headers.
2. Verify token.
3. Acknowledge quickly.
4. Process asynchronously.
5. Load current local aggregate by Xendit ID/reference.
6. Apply monotonic state transition rules.
7. Reconcile from Xendit API if state is ambiguous or event arrives out of expected order.

## Security Checklist

- HTTPS endpoint only.
- Compare callback token using constant-time comparison where available.
- Reject missing/invalid token before state change.
- Keep endpoint unguessability as defense-in-depth, not primary auth.
- Log request ID, Xendit object IDs, and state transition result.
- Alert on repeated invalid callback token attempts.

