# Ezypay Webhooks

Sources: `https://developer.ezypay.com/docs/webhook`, `/docs/webhook-events`, `/docs/webhook-security`, `/docs/webhook-behaviour`, `/docs/webhook-management`, and the reference pages for create, update, list, retrieve, delete webhook, notification logs and resend event. Checked 2026-10-21.

## Setup

- Need a public HTTPS endpoint that returns `2xx` after processing.
- Each merchant subscribes its own webhook; several merchants may point at the same URL. Identify the merchant with `merchantId` in the payload.
- Create per merchant (automate this when you provision merchant credentials):

```http
POST https://api-sandbox.ezypay.com/v2/webhooks/
Authorization: Bearer <ACCESS_TOKEN>
merchant: <MERCHANT_ID>
Content-Type: application/json

{ "url": "https://example.com/api/webhooks/ezypay",
  "eventTypes": ["invoice_created","invoice_paid","invoice_past_due",
                 "payment_method_invalid","payment_method_linked","payment_method_replaced",
                 "subscription_payment_stopped","subscription_payment_reactivate",
                 "credit_note_created","credit_note_paid","credit_note_failed","transaction_settled"],
  "clientKey": "<EZYPAY_WEBHOOK_CLIENT_KEY>" }
```

Response: `url`, `eventTypes`, `id` (store it), `createdOn`. Create accepts exactly three properties: `clientKey`, `url`, `eventTypes`. Same URL twice for a merchant: `duplicate_webhook_url`.

- Update: `PUT /v2/webhooks/{webhookId}` with `url`, `eventTypes`, and to rotate the key `"updateSecurity": true` plus `"updatedClientKey": "<NEW_KEY>"`. An update overwrites the previous URL, events and key.
- Delete: `DELETE /v2/webhooks/{webhookId}` (offboarding). List: `GET /v2/webhooks/` (`limit`, `cursor`). Retrieve: `GET /v2/webhooks/{webhookId}`.
- Delivery log: `GET /v2/webhooks/logs` with `limit`, `cursor`, `eventId`, `eventType`, `status`: shows delivery status and full payloads.
- Resend: `POST /v2/events/{eventId}/resendwebhook` fires all related webhooks again.

`eventTypes` accepted by Create Webhook: `customer_create`, `customer_update`, `invoice_created`, `invoice_past_due`, `invoice_paid`, `invoice_batch_created`, `invoice_batch_processing`, `invoice_batch_success`, `invoice_batch_invoice_failed`, `subscription_create`, `subscription_cancel`, `subscription_activate`, `payment_method_invalid`, `payment_method_valid`, `payment_method_linked`, `payment_method_replaced`, `payment_method_changed`, `subscription_payment_reactivate`, `subscription_payment_stopped`, `subscription_complete`, `credit_note_created`, `credit_note_paid`, `credit_note_failed`, `partner_invoice_created`, `partner_invoice_paid`, `partner_invoice_past_due`, `qr_response`, `transaction_settled`. The events guide also lists `terminal_registration` and `terminal_response` (Tap to Pay). The `invoice_batch_*` events have no guide page.

## Minimum events Ezypay recommends

| Customer related | Invoice management |
| --- | --- |
| `payment_method_invalid` | `subscription_payment_stopped` (subscription model) |
| `payment_method_linked` | `subscription_payment_reactivate` (subscription model) |
| `payment_method_replaced` | `invoice_created` (subscription model) |
| `payment_method_valid` (PayTo only) | `invoice_paid` |
| | `invoice_past_due` |
| | `credit_note_created`, `credit_note_paid`, `credit_note_failed` |
| | `transaction_settled` |

## Event list

| Event | Occurs when |
| --- | --- |
| `customer_create` / `customer_update` | customer created / updated |
| `invoice_created` | an invoice is created (including by the subscription engine) |
| `invoice_past_due` | invoice became `past_due`: failed transaction, or due date passed with no attempt |
| `invoice_paid` | invoice became `paid` from a successful transaction |
| `subscription_create`, `subscription_cancel`, `subscription_activate`, `subscription_complete` | subscription lifecycle |
| `subscription_payment_stopped` | billing schedule became inactive |
| `subscription_payment_reactivate` | stopped subscription resumed after invoices were resolved |
| `payment_method_invalid` | a valid method became invalid (not sent if invalid at creation) |
| `payment_method_valid` | PayTo mandate approved (PayTo starts invalid) |
| `payment_method_linked` | method linked to a customer (PCP, Checkout, Pay Now) |
| `payment_method_replaced` | method replaced by another |
| `payment_method_changed` | primary method changed |
| `credit_note_created` / `credit_note_paid` / `credit_note_failed` | refund or chargeback credit note lifecycle |
| `partner_invoice_created` / `_paid` / `_past_due` | partner (merchant to merchant) invoices |
| `transaction_settled` | money from an invoice was settled to the merchant |
| `terminal_registration`, `terminal_response` | Tap to Pay device and payment outcome |
| `qr_response` | QR string ready or QR cancellation result (PromptPay) |

## Payload structure

```json
{
  "requestId": "<uuid v4, same across retries of this delivery>",
  "merchantId": "<merchant uuid>",
  "eventType": "INVOICE_PAID",
  "createdOn": "2022-04-01T01:57:56.345",
  "data": { "...business object..." }
}
```

`eventType` arrives upper case (`INVOICE_PAID`, `INVOICE_PAST_DUE`, `CREDIT_NOTE_PAID`, `TRANSACTION_SETTLED`, ...). `createdOn` does not change across retries. `data` is the same object the matching API returns:

- Invoice events: full invoice (see `ezypay-billing.md`). `invoice_past_due` adds `failedPaymentReason {code, description}`, `paymentProviderResponse {code, description}`, `payNowUrl`, `paymentMethodInvalid`, `manualRetryPossible`, and `scheduledPaymentDate` for the next retry. `invoice_paid` `channel` can be `online_payment_recovery` when paid through Pay Now.
- Subscription events: full subscription object.
- Payment method events: payment method payload (`paymentMethodToken`, `customerId`, `type`, `card`, `invalidReason`, `valid`, `primary`, `replacedPaymentMethodData` on replace).
- Credit note events: `id`, `invoiceId`, `documentNumber` (`CN...`), `date`, `status` (`PROCESSING`, `PAID`, `FAILED`), `items[]`, `amount`, `totalTax`, `reason` (`REFUND` or `CHARGEBACK`), `customerId`, `paymentMethodToken`.
- `transaction_settled`: `settlementDocumentNumber`, `settlementTransactionId`, `invoiceTransactionId`, `merchantId`, `currencyCode`, `countryCode`, `invoiceId`, `externalInvoiceId`, `invoiceNumber`, `invoiceDate`, `transactionDate`, `transactionAmount`, `customerId`.
- Partner invoice events: `id`, `documentNumber`, `date`, `dueDate`, `status`, `items[]` (`type` `partner_on_demand`), `amount`, `totalTax`, `issuerMerchant`, `recipientMerchant`, `countryCode`, `partnerSubscriptionId`.
- `terminal_registration`: `merchantId`, `appDeviceId`, `deviceName`, `status` (`SUCCESS`), `modifiedOn`, `failReason`. Only sent on success; if missing after an hour, contact Ezypay.
- `qr_response`: `invoiceId`, `invoiceStatus`, `type` (`QRPAYMENT`), `qrType`, `qrData.qrString`, `qrData.expires`, `amount`, `currency`, and on cancel `cancelStatus`.

Chargeback mapping: received gives `credit_note_paid` (reason `CHARGEBACK`) plus `invoice_past_due`; won gives `invoice_paid` plus `credit_note_failed`; lost gives no event.

## Signature verification

From `/docs/webhook-security`:

1. Register a `clientKey` when creating the webhook (or `updatedClientKey` with `updateSecurity: true`).
2. Ezypay computes HMAC SHA-1 with the client key over the request payload and sends it in the `X-Ezypay-Signature` header.
3. Read the raw body bytes exactly as received. Do not parse, re-encode, or reformat.
4. Compute HMAC-SHA1(key = clientKey, message = raw body), hex encoded lowercase.
5. Compare to `X-Ezypay-Signature`. Use a constant-time comparison (not stated in the docs, standard practice).

Official test vector (re-computed on 2026-10-21, matches):

```text
client key = key
payload    = {"requestId":"4cb74646-4d30-4d6a-ac71-7a26a4a623d3","merchantId":"aad25d2b-85f2-40cf-8b6d-44b5439a220f","eventType":"INVOICE_BATCH_CREATED","createdOn":"2024-07-11T06:42:28.296","data":{"id":"26055389-927b-41b9-bc13-77a50887db5c","batchReference":"tyj56","createdOn":"2024-07-11T06:42:28.135","status":"SUBMITTED"}}
expected   = 6354ecd501ca4c87da2b42872949c7fa02fefd89
```

Node.js (matches the doc's `crypto.createHmac('sha1', clientKey).update(payload).digest('hex')`):

```js
import crypto from "node:crypto";

export function verifyEzypay(rawBody /* Buffer */, header, clientKey) {
  if (!header) return false;
  const expected = crypto.createHmac("sha1", clientKey).update(rawBody).digest("hex");
  const a = Buffer.from(expected, "utf8");
  const b = Buffer.from(String(header).trim().toLowerCase(), "utf8");
  return a.length === b.length && crypto.timingSafeEqual(a, b);
}
```

Python 3 (the doc's Python sample uses Python 2 calls such as `.encode("hex")`; this is the equivalent):

```python
import hashlib, hmac

def verify_ezypay(raw_body: bytes, header: str, client_key: str) -> bool:
    expected = hmac.new(client_key.encode("utf-8"), raw_body, hashlib.sha1).hexdigest()
    return bool(header) and hmac.compare_digest(expected, header.strip().lower())
```

PHP (doc): `hash_hmac('sha1', $payload, $clientKey, false)`. Java (doc): `HmacUtils.hmacSha1Hex(clientKey, payload)`. C#, Ruby and Perl samples in the doc produce the same lowercase hex.

Pitfalls:

- Frameworks that parse JSON first (Express `json()`, Next.js body parser) change bytes. Capture the raw buffer.
- The client key is set per webhook. If merchants share one URL with different keys, either give each merchant its own URL path (for example `/webhooks/ezypay/<merchant-slug>`) or read `merchantId` from the unverified body only to pick the key, then verify. Reject unknown merchants.
- Header absent means no key was registered or the request is forged. Reject.
- No timestamp is signed. Replay protection comes from deduping `requestId` and idempotent settlement.

## Delivery, retries, duplicates

- Sent immediately on the event. Failure = timeout or non-`2xx`. Retries every 60 minutes, up to 24 times. `requestId` is the same on retries.
- Log every `requestId`; a repeat is a retry and can be acknowledged without re-processing.
- Ezypay does not alert you when your endpoint keeps failing. Monitor your own errors and use the logs API and resend API to recover.
- Respond `2xx` quickly after storing the event; do slow work async.
- Order is not guaranteed in the docs. After verifying, fetch the current object (`GET /v2/billing/invoices/{id}`) and act on its status.

## Settlement handler pseudo-code

```text
on POST /api/webhooks/ezypay:
  raw = read raw body
  if not verifyEzypay(raw, header["X-Ezypay-Signature"], key_for_this_endpoint): return 401
  evt = parse(raw)
  if seen(evt.requestId): return 200
  record(evt.requestId, evt.eventType, evt.merchantId, raw)
  if evt.merchantId not in my_merchants: return 200 (ignore)
  if evt.eventType in (INVOICE_PAID, INVOICE_PAST_DUE):
     inv = GET /v2/billing/invoices/{evt.data.id} with merchant header = evt.merchantId
     local = find payment by inv.externalInvoiceId (or by stored invoice id)
     check inv.customerId == local.customerId, currency == "MYR",
           sum of items where type in (on_demand_payment, subscription_payment, setup_payment, addon_payment) == local.amount
     if inv.status == "PAID" and local.status == "pending": mark paid once, fulfil
     if inv.status == "PAST_DUE": mark failed with inv.failedPaymentReason.code, keep retry path
  if evt.eventType == CREDIT_NOTE_PAID: apply refund or chargeback by data.reason
  if evt.eventType == TRANSACTION_SETTLED: mark invoice settled
  return 200
```
