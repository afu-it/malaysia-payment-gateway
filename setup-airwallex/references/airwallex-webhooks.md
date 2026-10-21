# Airwallex Webhooks

Checked 2026-10-21.

## Source URLs

- Webhooks overview: `https://www.airwallex.com/docs/developer-tools/webhooks/webhooks-overview`
- Listen for webhook events (headers, signature, IPs): `https://www.airwallex.com/docs/developer-tools/webhooks/listen-for-webhook-events`
- Online Payments events: `https://www.airwallex.com/docs/developer-tools/webhooks/listen-for-webhook-events/online-payments`
- Payments webhooks reference (same list): `https://www.airwallex.com/docs/payments/reference/payments-webhooks`
- View and re-trigger events: `https://www.airwallex.com/docs/developer-tools/webhooks/view-and-re-trigger-webhook-events`
- Webhook API: `https://www.airwallex.com/docs/api/webhooks/webhooks`

## Create a subscription

Web app (Developer, Admin or Owner role): **Developer > Webhooks > New webhook**, set Name, Notification URL (HTTPS), API version (controls payload shape), pick events, and accounts. Copy the secret for that URL. Or use `POST /api/v1/webhooks/create` (the response returns `secret`). Events on an API-created webhook cannot be modified after creation. Create separate subscriptions (and secrets) for sandbox and production.

## Delivery

- `POST` with JSON body to your URL. Return `200` immediately; do the work async.
- Non-200 or timeout means failure; Airwallex retries with exponential back-off for about three days.
- Duplicates happen: the event `id` is stable across retries; dedupe on it.
- Order is not guaranteed; use `created_at` (and a fresh retrieve) when ordering matters.
- Events are kept 30 days in **Settings > Developer > Events** (status Success, Queued, Failed); you can re-trigger them there within 30 days.

## Headers

| Header | Meaning |
| --- | --- |
| `x-timestamp` | Unix timestamp in milliseconds, for example `1357872222592` |
| `x-signature` | HMAC-SHA256 hex digest of `x-timestamp` concatenated with the request body, keyed with the webhook secret. Sent when the webhook is configured with a secret |

## Verify the signature (exact steps from the docs)

1. Read `x-timestamp` and `x-signature`.
2. `value_to_digest = x-timestamp (as string) + raw JSON body (as string)`. Timestamp first. No separator.
3. Compute HMAC with SHA-256, key = the notification URL's secret, message = `value_to_digest`; hex encode.
4. Compare with `x-signature`. If equal, check `now - timestamp` is within your tolerance.

Common failures Airwallex lists: using a re-serialized parsed object instead of the raw body; parsing before verifying; altering the timestamp; using another subscription's secret; wrong concatenation order. Test events sent from the web app are signed with the secret given in their `client-secret-key` header.

Node.js (Express, raw body):

```js
import crypto from 'node:crypto';
import express from 'express';
const app = express();
app.post('/webhooks/airwallex', express.raw({ type: 'application/json' }), (req, res) => {
  const ts = req.get('x-timestamp') || '';
  const sig = req.get('x-signature') || '';
  const raw = req.body.toString('utf8');
  const expected = crypto.createHmac('sha256', process.env.AIRWALLEX_WEBHOOK_SECRET).update(ts + raw).digest('hex');
  const ok = sig.length === expected.length && crypto.timingSafeEqual(Buffer.from(sig), Buffer.from(expected));
  if (!ok || Math.abs(Date.now() - Number(ts)) > 5 * 60 * 1000) return res.status(400).send('bad signature');
  const event = JSON.parse(raw);
  res.sendStatus(200);           // ack first
  queue.push(event);             // then process idempotently by event.id
});
```

PHP (from the official example, plus constant-time compare):

```php
$timestamp = $_SERVER['HTTP_X_TIMESTAMP'] ?? '';
$body = file_get_contents('php://input');
$expected = hash_hmac('sha256', $timestamp . $body, getenv('AIRWALLEX_WEBHOOK_SECRET'));
if (!hash_equals($expected, $_SERVER['HTTP_X_SIGNATURE'] ?? '')) { http_response_code(400); exit; }
```

Python:

```python
import hmac, hashlib, time
ts = request.headers.get("x-timestamp", "")
raw = request.get_data(as_text=True)          # raw body, not request.json
expected = hmac.new(SECRET.encode(), (ts + raw).encode(), hashlib.sha256).hexdigest()
if not hmac.compare_digest(expected, request.headers.get("x-signature", "")) or abs(time.time()*1000 - int(ts or 0)) > 300000:
    abort(400)
```

The tolerance window is your choice; Airwallex does not publish a value.

## Event payload

```json
{
  "id": "evt_...",
  "name": "payment_intent.succeeded",
  "account_id": "<ACCOUNT_ID>",
  "data": { "object": { "...": "same schema as Retrieve a Payment Intent" } },
  "created_at": "2019-10-22T01:54:09+0000",
  "version": "2024-02-22"
}
```

`account_id` is set for account-level events, `org_id` for organization-level events. `data.object` matches the Retrieve API of that resource (Payment Intent, Payment Attempt, Refund, and so on). One documented sample uses `accountId` and omits a comma; parse defensively and rely on the fields above.

## Online payment events

PaymentIntent: `payment_intent.created`, `payment_intent.requires_payment_method`, `payment_intent.updated`, `payment_intent.requires_capture`, `payment_intent.requires_customer_action`, `payment_intent.pending`, `payment_intent.pending_review`, `payment_intent.succeeded`, `payment_intent.cancelled`, `payment_intent.payment_failed`.

PaymentAttempt: `payment_attempt.received`, `payment_attempt.authentication_failed`, `payment_attempt.authentication_redirected`, `payment_attempt.pending_authorization`, `payment_attempt.authorization_failed`, `payment_attempt.authorized`, `payment_attempt.capture_requested`, `payment_attempt.settled` (funds received by Airwallex from the acquirer), `payment_attempt.paid` (funds paid to your wallet), `payment_attempt.cancelled`, `payment_attempt.expired`, `payment_attempt.risk_declined`, `payment_attempt.failed_to_process`, `payment_attempt.capture_failed`.

PaymentConsent: `payment_consent.created`, `payment_consent.updated`, `payment_consent.pending`, `payment_consent.verified`, `payment_consent.disabled`, `payment_consent.paused`, `payment_consent.requires_payment_method`, `payment_consent.requires_customer_action`, `payment_consent.verification_failed`.

Customer: `customer.created`, `customer.updated`.

Refund: `refund.received`, `refund.accepted`, `refund.settled`, `refund.failed`.

PaymentMethod: `payment_method.created`, `payment_method.updated`, `payment_method.attached`, `payment_method.detached`, `payment_method.disabled`.

Dispute: `payment_dispute.requires_response`, `payment_dispute.challenged`, `payment_dispute.accepted`, `payment_dispute.expired`, `payment_dispute.pending_closure`, `payment_dispute.pending_decision`, `payment_dispute.won`, `payment_dispute.lost`, `payment_dispute.reversed`.

PaymentLink: `payment_link.created`, `payment_link.paid`.

Fraud: `fraud.merchant_notified` (refund and hold delivery).

Funds split: `funds_split.created`, `funds_split.failed`, `funds_split.released`, `funds_split.settled`.

POS terminal: `pos.terminal.activated`, `pos.terminal.deactivated`, `pos.terminal.terminated`, `pos.terminal.updated`, and admin/refund password status events.

Minimum set for a shop: `payment_intent.succeeded`, `payment_intent.payment_failed`, `payment_intent.cancelled`, `payment_intent.requires_capture` (if manual capture), `refund.accepted`, `refund.settled`, `refund.failed`, `payment_dispute.requires_response`, `fraud.merchant_notified`.

## Settlement handler checklist

1. Verify signature on the raw body; reject on mismatch or stale timestamp.
2. Insert `event.id` into a processed-events table with a unique constraint; if it exists, return 200.
3. For `payment_intent.*`, re-fetch `GET /api/v1/pa/payment_intents/{data.object.id}` with your token.
4. Match `merchant_order_id` to your order, `currency`, and `amount` (or `captured_amount`) to what you charged.
5. Transition your order only forward (pending to paid); ignore older events that arrive late.

## Airwallex webhook source IPs (allowlist these)

Production: `35.240.218.67`, `35.185.179.53`, `34.87.64.173`, `35.220.213.251`, `34.92.128.176`, `34.91.47.254`, `34.91.75.229`, `35.230.185.215`, `34.86.42.60`, `34.177.101.131`, `34.143.140.106`, `34.158.57.211`, `34.124.183.136`, `34.126.131.52`, `34.21.185.174`, `34.142.140.136`, `35.236.197.105`, `34.11.6.7`, `34.145.133.253`, `34.182.195.122`, `35.186.175.150`, `34.85.218.163`, `136.107.34.62`, `35.186.166.138`, `34.21.104.184`.

Sandbox: `35.240.211.132`, `35.187.239.216`, `34.87.139.23`, `136.110.35.143`, `34.142.189.194`, `35.197.128.86`.

IP allowlisting is hardening, not a replacement for signature checks. Re-check the list on the official page before deploying firewall rules.

## Webhook API field tables
### Create a Webhook

`POST /api/v1/webhooks/create`

Creates a new Webhook and subscribes to a list of webhook events from Airwallex.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `events` | array[string] | yes | A list of events subscribed by this webhook. Events cannot be modified after creation. |
| `request_id` | string | yes | A unique request identifier specified by the client for idempotency. Up to 50 characters. |
| `url` | string | yes | The endpoint where events are sent to. |
| `version` | string | yes | The API version which controls the event payload structure. It should be in YYYY-MM-DD format, e.g. `2022-11-11`. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `created_at` | string | yes | The time when this webhook was created. |
| `events` | array[string] | yes | A list of events subscribed by this webhook. Events cannot be modified after creation. |
| `id` | string | yes | The unique identifier of the Webhook object. |
| `request_id` | string | yes | A unique request identifier specified by the client for idempotency. Up to 50 characters. |
| `secret` | string | yes | The secret to verify that events are sent from Airwallex. See this guide for details. |
| `updated_at` | string | yes | The time when this webhook was last updated. |
| `url` | string | yes | The endpoint where events are sent to. |
| `version` | string | yes | The API version which controls the event payload structure. It should be in YYYY-MM-DD format, e.g. `2022-11-11`. |

Documented errors: `already_exists`; `field_required`, `invalid_argument`; `unauthorized`

### Retrieve a Webhook

`GET /api/v1/webhooks/{id}`

Retrieves the details of a Webhook.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Unique identifier of the Webhook object. |

Documented errors: `invalid_argument`; `unauthorized`

### List all Webhooks

`GET /api/v1/webhooks`

Retrieves a list of Webhooks.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `page_size` | integer | no | Number of Webhooks per page. Defaults to 20. |
| `page` | string | no | A bookmark for use in pagination to retrieve either the next page or the previous page of results. You can fetch the value for this identifier from the response of the previous API call. |

Documented errors: `credentials_invalid`, `credentials_expired`; `invalid_argument`

### Update a Webhook

`POST /api/v1/webhooks/{id}/update`

Updates the attributes of an existing Webhook. Only updates to `url` are currently supported.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Unique identifier of the Webhook object. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `url` | string | yes | The endpoint where events are sent to. |

Documented errors: `field_required`, `invalid_argument`; `unauthorized`

### Delete a Webhook

`POST /api/v1/webhooks/{id}/delete`

Deletes a Webhook. A deleted webhook will no longer receive webhook events of any Airwallex organisations or accounts.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Unique identifier of the Webhook object. |

Documented errors: `credentials_invalid`, `credentials_expired`; `field_required`, `invalid_argument`

