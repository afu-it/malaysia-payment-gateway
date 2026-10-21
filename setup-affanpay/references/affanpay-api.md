# AffanPay API Reference

Use this reference for factual AffanPay API guidance. Everything here comes from the single official API documentation page (live and sandbox copies), checked 2026-10-21. Where the docs are silent, this file says so; do not fill the gap by guessing.

## Source URLs

- `https://app.affanpay.my/api` (live docs, sections: Introduction, Token, Create Bill, Requery, Callback)
- `https://sandbox.affanpay.my/api` (sandbox docs, identical text with sandbox host in every URL)

## Integration Surfaces

- Create Bill API: server creates a bill and gets a hosted payment page `url`.
- Hosted bill payment page on `app.affanpay.my/<bill_id>` (or the sandbox host): customer picks the channel and pays.
- Callback (webhook) POST to your `callback_url` for final states.
- Requery API as fallback.
- No-code: payment links, QR codes, and dashboard (marketing site `https://affanpay.my/`). Not covered by the API docs.

## Environments

| Environment | API base | Docs | Dashboard sign-in |
|---|---|---|---|
| Live | `https://app.affanpay.my/api` | `https://app.affanpay.my/api` | `https://app.affanpay.my/sign-in` |
| Sandbox | `https://sandbox.affanpay.my/api` | `https://sandbox.affanpay.my/api` | `https://sandbox.affanpay.my/sign-in` |

Sandbox is a separate host with its own registration (`https://sandbox.affanpay.my/register`), so tokens and secret keys differ per environment. Bill page URLs in responses use the host of the environment.

## Endpoint Catalogue

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/bill` | Create a bill and get its hosted payment page URL |
| GET | `/api/v1/requery?bill_id=<id>` or `?payment_ref=<ref>` | Get latest payment status for a bill or a single payment |
| POST (outbound) | your `callback_url` | AffanPay notifies a final payment state (Success or Failed) |

No other endpoints are documented (refund, cancel, void, capture, recurring, tokenisation, payout, settlement report, category or bill listing: not documented in official docs, checked 2026-10-21).

## Authentication

- Every request carries `Authorization: Bearer <API_TOKEN>`.
- Also send `Accept: application/json`. For POST with a JSON body, send `Content-Type: application/json` (as in the official example).
- Generate the token: log in, Settings > Developer, under API tokens give it a name (for example "Production store"), click Generate token, copy it immediately. It is shown once and cannot be retrieved again.
- All responses are JSON, including errors. Observed: a request without a token returns HTTP 401 `{"message":"Unauthenticated."}`.

## Create Bill

`POST {base}/v1/bill`

Creates a bill that can be used directly to initiate and receive payment.

### Request body (JSON)

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | string | Yes | Name or title of the bill. |
| `amount` | decimal | Yes | Total amount in ringgit (MYR). Up to 2 decimal places. Minimum `5.00`. The official example sends it as a JSON string. |
| `external_ref` | string | Yes | Your unique reference for the bill (order ID). Echoed back as `bill.external_ref` in callbacks. |
| `customer_name` | string | Yes | Full name. Max 60 characters. |
| `customer_email` | (email) | Yes | Max 50 characters. Type not stated in docs; send a string. |
| `category` | string | No | Category ID, found in the Category section of the dashboard. |
| `description` | string | No | Short description, max 200 characters. |
| `html` | string | No | HTML content for the bill description. Must contain valid HTML tags. |
| `fee_charge_payer` | boolean | No | `true` makes the payer bear the transaction fee. |
| `redirect_url` | string | No | Where the customer is sent after payment. |
| `callback_url` | string | No | Endpoint for async status updates. No callback is sent if omitted. |
| `customer_phone` | string | No | Max 20 characters. |
| `source` | (string) | No | Source identifier for bill creation. Max 40 characters. |
| `expiry_date` | string | No | `YYYY-MM-DD HH:MM:SS`, must be today or later. Timezone not stated (assume Malaysia time, confirm with AffanPay). |

Note: the official PHP sample lists only `name`, `amount`, `customer_name`, `customer_email`, but the field table marks `external_ref` as required too. Always send `external_ref`.

### Example request

```http
POST https://sandbox.affanpay.my/api/v1/bill
Accept: application/json
Content-Type: application/json
Authorization: Bearer <API_TOKEN>

{
  "name": "Order ORDER-1001",
  "amount": "50.00",
  "external_ref": "ORDER-1001",
  "customer_name": "Ahmad bin Ali",
  "customer_email": "ahmad@example.my",
  "redirect_url": "https://example.com/pay/affanpay/return",
  "callback_url": "https://example.com/api/webhooks/affanpay"
}
```

### Response

```json
{ "success": true, "id": "pujsSUNLeIYo", "url": "https://app.affanpay.my/pujsSUNLeIYo" }
```

| Field | Notes |
|---|---|
| `success` | `true` on creation. |
| `id` | Bill code. Same value that appears in the payment page URL and as `bill.id` in callbacks. Store it. |
| `url` | Hosted payment page. Redirect the customer here. |

Error body shapes (validation errors, HTTP codes other than 401) are not documented in official docs (checked 2026-10-21). Treat any response without `success: true` and an `id` as a failure and log the body.

## Redirect URL

The docs only say `redirect_url` is where the customer is sent after payment completion. Query parameters appended on redirect (if any) are not documented (checked 2026-10-21). Use the redirect only to show a "checking payment" page, then rely on the callback or requery.

## Requery

`GET {base}/v1/requery?bill_id=<bill_id>` or `GET {base}/v1/requery?payment_ref=<payment_ref>`

Use when a callback failed or was not received. Supply exactly one query parameter; the two cannot be combined.

| Query param | Description |
|---|---|
| `bill_id` | Bill identifier. Returns the bill and all its payment records. Example `bill_id=wxUyT47ko3v`. |
| `payment_ref` | Payment reference. Returns that transaction's details. Example `payment_ref=AP01384088585413050521`. |

Headers: `Accept: application/json`, `Authorization: Bearer <API_TOKEN>`.

### Response (bill_id example from docs)

```json
{
  "data": {
    "id": "wxUyT47ko3v",
    "name": "Multi open amount",
    "payments": [
      { "payment_ref": "AP01384263015313050521", "status_code": 4, "status": "No Action", "nett_amount": " 0.00", "payment_channel": null },
      { "payment_ref": "AP01384355115313050521", "status_code": 3, "status": "Failed", "nett_amount": " 0.00", "payment_channel": "fpx" },
      { "payment_ref": "AP01384088585413050521", "status_code": 1, "status": "Success", "nett_amount": " 31.20", "payment_channel": "fpx" }
    ]
  }
}
```

| Field | Notes |
|---|---|
| `data.id` | Bill ID. |
| `data.name` | Bill name. |
| `data.payments[]` | Every payment attempt on the bill. A bill can have several attempts; look for any with `status_code` 1. |
| `payments[].payment_ref` | Transaction reference (same as callback `payment_ref`). |
| `payments[].status_code` | See Status Codes. |
| `payments[].status` | Display label. |
| `payments[].nett_amount` | String, may carry a leading space (`" 31.20"`). Trim and parse as decimal. Whether "nett" is after fees is not explained in docs; do not use it as the order-value check without confirming. |
| `payments[].payment_channel` | e.g. `fpx`; `null` when no channel chosen. |

The response shape for a `payment_ref` query is not shown separately in the docs (checked 2026-10-21). Handle it defensively.

## Status Codes

| `status_code` | `status` | Callback sent? |
|---|---|---|
| 1 | Success | Yes |
| 2 | Pending | No |
| 3 | Failed | Yes |
| 4 | No Action | No |

Decide from `status_code` (cast to integer), never from `status`.

## Payment Channels

Values documented in examples: `fpx`, `duitnow_qr`, `grabpay`. A full channel code list is not documented (checked 2026-10-21). Channels can be `null`. Store the value as an opaque string.

## Callback (Webhook)

AffanPay sends `POST` to the bill's `callback_url` when a payment reaches a final state.

- Sent only for Success (1) or Failed (3). Pending and No Action never trigger one.
- Each payment is sent TWICE, roughly 5 and 10 seconds apart. The handler must be idempotent.
- Nothing is sent if the bill has no `callback_url`.
- Respond HTTP 200 once processed; the body is not read. Any status 400 or above is treated as failure and recorded against the transaction.
- If your handler fails, return 4xx/5xx so the delivery is retried and flagged. Retry schedule beyond the two sends is not documented.

### Headers

```text
Content-Type: application/json
X-AFFANPAY-TIMESTAMP: 1756684800
X-AFFANPAY-HMAC: f10f01be8675bfaa...
```

### Body

```json
{
  "payment_ref": "AP01384355115313050521",
  "status_code": 1,
  "status": "Success",
  "paid_amount": "324.76",
  "payer": { "name": "Ahmad bin Ali", "phone": "0123456789", "email": "ahmad@example.my" },
  "payment_channel": "fpx",
  "bank_reference": "2609011012340000",
  "bank_response_datetime": "2026-09-01 10:12:34",
  "bill": {
    "id": "k3jh2x",
    "name": "Invoice 12/2026",
    "amount": "320.75",
    "external_ref": "ORDER-1001",
    "created_at": "2026-09-01 09:00:00"
  }
}
```

| Field | Notes |
|---|---|
| `payment_ref` | AffanPay transaction reference, unique per payment attempt. Use as the idempotency key. |
| `status_code` | `1` Success or `3` Failed. Decide from this. |
| `status` | Display label only. |
| `paid_amount` | What the payer actually paid (string). Higher than `bill.amount` when the bill passes the service charge to the payer. Cast before comparing. |
| `payer.name`, `payer.phone`, `payer.email` | As entered by the payer. Any can be `null`. |
| `payment_channel` | e.g. `fpx`, `duitnow_qr`, `grabpay`. Can be `null`. |
| `bank_reference` | Reference from bank or wallet. Can be `null`. |
| `bank_response_datetime` | `YYYY-MM-DD HH:MM:SS` (not described in field table; shown in example). |
| `bill.id` | Bill code, same as in the payment page URL. |
| `bill.name` | Bill name. |
| `bill.amount` | Amount the bill was created for, before any service charge. |
| `bill.external_ref` | Your reference exactly as sent. `null` if you did not send one. |
| `bill.created_at` | Bill creation time (example only). |

## Signature Verification

- Header `X-AFFANPAY-HMAC`: lowercase hex HMAC-SHA256.
- String to sign: `X-AFFANPAY-TIMESTAMP` value + `"."` + raw request body (exact bytes received).
- Key: the secret key shown in Settings > Developer. It is never sent in callbacks. It is NOT the API token.
- Verification is optional for AffanPay (unsigned-aware handlers keep working) but strongly recommended; treat it as required.
- Read the raw body (`php://input`, `express.raw`, etc.). Decoding and re-encoding JSON changes the string and breaks the match.
- Compare with a constant-time function (`hash_equals`, `crypto.timingSafeEqual`).
- Reject when `abs(now - timestamp) > 300` seconds (the docs' replay guard). The timestamp is Unix seconds.

Official PHP example (condensed):

```php
$secret    = getenv('AFFANPAY_SECRET_KEY');
$body      = file_get_contents('php://input');
$timestamp = $_SERVER['HTTP_X_AFFANPAY_TIMESTAMP'] ?? '';
$received  = $_SERVER['HTTP_X_AFFANPAY_HMAC'] ?? '';
$expected  = hash_hmac('sha256', $timestamp . '.' . $body, $secret);
if (!hash_equals($expected, $received)) { http_response_code(400); exit('invalid signature'); }
if (abs(time() - (int) $timestamp) > 300) { http_response_code(400); exit('stale callback'); }
$payload = json_decode($body, true);
if ((int) $payload['status_code'] === 1) { /* mark paid, idempotently */ }
http_response_code(200);
```

Node pseudo-code:

```js
// app.post('/api/webhooks/affanpay', express.raw({ type: 'application/json' }), handler)
const ts = req.get('X-AFFANPAY-TIMESTAMP') || '';
const sig = req.get('X-AFFANPAY-HMAC') || '';
const expected = crypto.createHmac('sha256', process.env.AFFANPAY_SECRET_KEY)
  .update(ts + '.' + req.body.toString('utf8')).digest('hex');
const ok = sig.length === expected.length &&
  crypto.timingSafeEqual(Buffer.from(sig), Buffer.from(expected));
if (!ok || Math.abs(Date.now() / 1000 - Number(ts)) > 300) return res.sendStatus(400);
```

## Settlement Checklist

1. Signature valid and timestamp fresh.
2. `bill.id` matches the stored bill for the order, and `bill.external_ref` matches the order.
3. `Number(status_code) === 1`.
4. Decimal `bill.amount` equals the expected order amount; `paid_amount` is greater than or equal to it (higher only if `fee_charge_payer` is true).
5. `payment_ref` not already processed (unique constraint), then mark paid in one transaction.
6. `status_code === 3`: record the failed attempt but keep the order open; the same bill may get another attempt.

## Amounts

- Ringgit decimal with up to 2 decimal places, not sen. Minimum bill amount `5.00`.
- Responses return amounts as strings (`"324.76"`, `" 31.20"`). Trim and parse with a decimal type.
- Currency field is not present; MYR is implied. Not documented further (checked 2026-10-21).

## Not Documented (checked 2026-10-21)

Refunds, cancel/void, capture, recurring or tokenisation, payouts, settlement reports, bill listing or retrieval, category API, rate limits, IP allowlists for callbacks, sandbox test cards or test banks, error code list, redirect query parameters, callback retry schedule beyond the two sends, open-amount bill creation (the requery example shows a bill named "Multi open amount", but no field for it is documented), and SDKs (the homepage mentions SDKs and a sandbox; only the sandbox host was found). Confirm any of these with AffanPay support (`contact@affanpay.my`) before relying on them.

## Production Readiness

- Live token and secret key from the live dashboard, live base URL, HTTPS `callback_url` reachable from the internet.
- Payment channels approved (QR, e-wallet, and card channels are subject to approval per the homepage pricing section).
- Requery job for orders pending longer than your threshold.
- Logs of raw callback bodies and headers (without secrets) for disputes.
