# Omise Webhooks and Events

Checked 2026-10-21 against `https://docs.omise.co/en/api-webhooks/malaysia` and `https://docs.omise.co/en/events-api/malaysia`.

## Source URLs

- Webhooks: `https://docs.omise.co/en/api-webhooks/malaysia`
- Event API: `https://docs.omise.co/en/events-api/malaysia`
- IPs: `https://docs.omise.co/en/api-ips/malaysia`
- Versioning (serialization): `https://docs.omise.co/en/api-versioning/malaysia`
- Testing (webhook FAQ): `https://docs.omise.co/en/api-testing/malaysia`

## Configuration

- One static webhook endpoint per mode, set in the dashboard: test `https://dashboard.omise.co/test/webhooks`, live `https://dashboard.omise.co/live/webhooks`. Also settable with `PATCH /account` field `webhook_uri`.
- The static endpoint receives every event type; there is no per-type filter.
- Dynamic webhooks: pass `webhook_endpoints[]` (max 2 HTTPS URLs; no localhost, IP literals, or numeric-only hosts) on charge creation. All events for that charge and its refunds go only to those URLs, not to the static endpoint.
- Requirements: HTTPS, valid certificate (self-signed rejected), TLS 1.2 or later.
- Firewall: allow `54.169.118.227`, `52.74.199.175`, `18.139.13.19` (also in `https://cdn.omise.co/ips.json`).
- Respond `200 OK` after processing. Non-2xx may be treated as a failed delivery.
- Retries: the webhooks FAQ says Omise does not currently guarantee automatic retries; the testing FAQ says returning non-200 triggers Omise's retry mechanism. Build as if there are no retries: reconcile with the Events API or by re-fetching charges.
- Same endpoint for test and live is possible but not recommended, since the secrets differ.

## Payload

The POST body is an Event object:

```json
{
  "object": "event",
  "id": "evnt_test_...",
  "livemode": false,
  "location": "/events/evnt_test_...",
  "key": "charge.complete",
  "created_at": "2019-12-31T12:59:59Z",
  "data": { "object": "charge", "id": "chrg_test_...", "status": "successful", "amount": 12345, "currency": "MYR", "...": "..." },
  "webhook_deliveries": []
}
```

- `data` is the full object (charge, refund, customer, and so on) serialized with the account default API version at the time of the event, regardless of any `Omise-Version` header on the triggering request.
- Event attributes: `object`, `id` (`/evnt(_test)?_[0-9a-z]+/`), `livemode`, `location`, `key`, `data`, `created_at`, `webhook_deliveries` (each `object=webhook_delivery`, `id` `whdl_...`, `uri`, `status` HTTP code, `key`, `error`, `payload`, `created_at`), `actor_uid`, `user_uid`, `team_uid` (PayFac).
- Payment Links+ webhook payloads use `link_id` for the link id.

## Signature verification

Omise signs with HMAC-SHA256 when a webhook secret exists for the account (dashboard > Webhooks Settings; separate secret for test and live).

Headers:

| Header | Value |
| --- | --- |
| `Omise-Signature` | Hex HMAC. During rotation, two comma-separated signatures |
| `Omise-Signature-Timestamp` | Unix timestamp when the signature was generated |

Steps (exact order from the docs):

1. Read `Omise-Signature` and `Omise-Signature-Timestamp`.
2. Signed payload = timestamp header value + `.` + raw request body (UTF-8, unmodified).
3. Base64-decode the webhook secret to get the HMAC key.
4. Expected = hex(HMAC-SHA256(key, signed payload)).
5. Split `Omise-Signature` on `,`; accept if any value matches using a constant-time compare; otherwise reject.
6. Optional replay check: reject if `|now - timestamp|` exceeds your window (Omise example: 5 minutes). A signature match alone already proves authenticity.

Pseudo-code:

```text
ts      = header("Omise-Signature-Timestamp")
sigs    = header("Omise-Signature").split(",")
key     = base64_decode(OMISE_WEBHOOK_SECRET)
expect  = hmac_sha256(key, ts + "." + raw_body)           # bytes
ok      = any(timing_safe_equal(hex_decode(s.strip()), expect) for s in sigs)
```

Node.js (from the Omise example, with a length guard added because `timingSafeEqual` throws on unequal lengths):

```js
const crypto = require('crypto');
app.post('/webhooks/omise', express.raw({ type: 'application/json' }), (req, res) => {
  const ts = req.get('omise-signature-timestamp');
  const header = req.get('omise-signature') || '';
  const key = Buffer.from(process.env.OMISE_WEBHOOK_SECRET, 'base64');
  const expected = crypto.createHmac('sha256', key).update(`${ts}.${req.body.toString('utf8')}`).digest();
  const ok = header.split(',').some((s) => {
    const b = Buffer.from(s.trim(), 'hex');
    return b.length === expected.length && crypto.timingSafeEqual(b, expected);
  });
  if (!ok) return res.status(401).send('Invalid signature');
  const event = JSON.parse(req.body.toString('utf8'));
  // enqueue: re-fetch event.data.id via GET /charges/{id}, then settle idempotently
  res.status(200).send('OK');
});
```

Python:

```python
import base64, hmac, hashlib
def verify(raw_body: bytes, ts: str, sig_header: str, secret_b64: str) -> bool:
    key = base64.b64decode(secret_b64)
    expected = hmac.new(key, ts.encode() + b"." + raw_body, hashlib.sha256).hexdigest()
    return any(hmac.compare_digest(expected, s.strip()) for s in sig_header.split(","))
```

PHP:

```php
$key = base64_decode(getenv('OMISE_WEBHOOK_SECRET'));
$expected = hash_hmac('sha256', $ts . '.' . $rawBody, $key);
$ok = false;
foreach (explode(',', $sigHeader) as $s) { if (hash_equals($expected, trim($s))) { $ok = true; } }
```

Troubleshooting failures: wrong-mode secret, body changed by a JSON parser or middleware before hashing, secret not Base64-decoded.

### Secret rotation

- "Roll" creates a new secret immediately; the old one stays valid for 24 hours, and every webhook carries both signatures in that window.
- You cannot delete the primary active secret. You can revoke the expiring one during the 24 hours.
- Max two active secrets (current and expiring); wait or revoke before rolling again.
- If compromised: roll, revoke the expiring secret, deploy the new one.
- Test rotation in test mode first.

### Event verification (alternative and complement)

Omise's documented fallback when signatures are not feasible: take the id from the payload and `GET /charges/{id}` to confirm status. It does not stop fake requests reaching you, so combine it with signature checks.

## Event keys

| Group | Keys |
| --- | --- |
| Card | `card.destroy`, `card.update` |
| Charge | `charge.capture` (manual capture), `charge.complete` (completed; not sent for non-3DS card charges), `charge.create`, `charge.expire` (documented as Barcode Alipay only), `charge.reverse` (manual reversal), `charge.update` |
| Customer | `customer.create`, `customer.destroy`, `customer.update`, `customer.update.card` |
| Dispute | `dispute.accept`, `dispute.close`, `dispute.create`, `dispute.update` |
| Recipient | `recipient.activate`, `recipient.create`, `recipient.deactivate`, `recipient.destroy`, `recipient.update`, `recipient.verify` |
| Refund | `refund.create` |
| Schedule | `schedule.create`, `schedule.destroy`, `schedule.expire`, `schedule.expiring`, `schedule.suspend` |
| Transfer | `transfer.create`, `transfer.destroy`, `transfer.fail`, `transfer.pay`, `transfer.send`, `transfer.update` |

New event types can appear without an API version change; ignore unknown keys.

Settlement rule: `charge.complete` does not by itself mean success. Re-fetch and check `status` (`successful`, `failed`, `expired`). For non-3DS cards, use the synchronous charge response (`status=successful`) as the trigger, then still record the charge id idempotently.

## Events API

| Endpoint | Purpose | Params |
| --- | --- | --- |
| `GET /events` | List account events (may contain several serialization versions) | `from`, `to`, `limit` (default 20), `offset`, `order` |
| `GET /events/{id}` | Retrieve one event | |
| `GET /charges/{id}/events` | Events for one charge | list params |

Use `GET /events?from=<last_seen>&order=chronological` in a scheduled job to catch missed deliveries; dedupe by event `id`.
