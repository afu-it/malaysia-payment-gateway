# Security And Settlement

## Must Not Do

- Do not mark paid from browser redirect alone.
- Do not trust client amount, currency, product, or gateway id.
- Do not expose secret keys in browser code, public env vars, logs, screenshots, or committed files.
- Do not JSON-parse then stringify body before webhook HMAC verification when provider signs raw body.
- Do not assume webhooks arrive once or in order.
- Do not fulfill irreversible goods before final captured/succeeded/paid state unless business explicitly accepts authorization risk.

## Required Protections

- Signature/token verification.
- Event dedupe.
- Idempotent settlement.
- Amount and currency match.
- Provider and gateway reference match.
- Local payment status transition guard, such as `WHERE status = 'pending'`.
- Server-side entitlement/order fulfillment.
- Audit payload storage with secrets redacted.

## Loophole Review

Before final answer, ask:

- Can user fake a success URL and unlock access?
- Can user change amount/product in request body?
- Can duplicate webhook create duplicate rewards/access?
- Can webhook from another provider mark this payment paid?
- Can stale test keys hit live endpoint or live keys hit sandbox endpoint?
- Can invalid/missing raw body verification pass?
- Can fulfillment happen before provider confirms final payment?
- Can wrong currency pass because numeric amount matches?
- Can a failed checkout leave bad pending records without admin visibility?

Fix every yes or unknown before claiming done.

## Deployment Checklist

Document:

- Required environment variables/secrets.
- Webhook URLs per environment.
- Dashboard event types to enable.
- Migration commands.
- Sandbox test cards/banks or provider test method location.
- How to switch active gateway if app supports multiple providers.
