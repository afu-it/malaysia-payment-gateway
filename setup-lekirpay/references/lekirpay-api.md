# LekirPay API Reference

Use this reference for factual LekirPay API guidance. Everything here comes from the official docs at `https://app.lekirpay.com/doc/` and the official LekirPay GiveWP plugin (`lekirpay-givewp` 3.1.2, author `lekirtech` on WordPress.org), checked 2026-10-21. Where something is not documented, this file says so; do not fill gaps from memory.

## Source URLs

- `https://app.lekirpay.com/doc/index`
- `https://app.lekirpay.com/doc/flow`
- `https://app.lekirpay.com/doc/token`
- `https://app.lekirpay.com/doc/payment/normal`
- `https://app.lekirpay.com/doc/payment/secure`
- `https://app.lekirpay.com/doc/payment/callback`
- `https://app.lekirpay.com/doc/paymentstatus`
- `https://app.lekirpay.com/doc/paymentGroup/create`
- `https://app.lekirpay.com/doc/paymentGroup/data`
- `https://wordpress.org/plugins/lekirpay-givewp/` (source: `https://downloads.wordpress.org/plugin/lekirpay-givewp.3.1.2.zip`)

## Integration Surfaces

- REST API: token, create payment (plain or signed), callback, status requery, payment groups.
- Hosted payment page: the `payment_url` returned by create payment (`https://app.lekirpay.com/public/pay/{payment_id}`).
- No-code: dashboard payment form, custom field, payment link, QR code (website pricing section).
- Plugin: official GiveWP donation plugin. No official WooCommerce plugin found on WordPress.org (checked 2026-10-21).

For a custom checkout the default path is: token, `POST /api/payments/payment/secure`, redirect to `payment_url`, receive callback, requery status, settle.

## Environments

| Environment | Host | API base | Notes |
|---|---|---|---|
| Live | `https://app.lekirpay.com` | `https://app.lekirpay.com/api` | Docs and plugin |
| Sandbox | `https://sandbox.lekirpay.com` | `https://sandbox.lekirpay.com/api` | Docs (`/doc/index`) and plugin readme signup link |
| Sandbox (plugin code) | `https://matkilau.lekirpay.com` | `https://matkilau.lekirpay.com/api` | Used by GiveWP plugin when "Server End Point" is disabled |

Checked 2026-10-21: `sandbox.lekirpay.com` returned HTTP 500 at `/` and 403 at `/api/`; `matkilau.lekirpay.com` did not resolve. Confirm the current sandbox host with LekirPay support before building tests around it.

Token URL: docs only show the live URL `https://app.lekirpay.com/oauth/token`. The plugin builds it as `{host}/oauth/token` for both hosts, so sandbox is `https://sandbox.lekirpay.com/oauth/token` by the same pattern (inferred from plugin code, not stated in docs).

Sandbox and live use separate merchant accounts (plugin readme gives separate signup links). Do not mix credentials across hosts.

## Request Conventions

- RESTful HTTP, JSON responses. The API accepts `application/x-www-form-urlencoded` and `application/json` request bodies (`/doc/index`).
- Headers: `Accept: application/json`, `Content-Type` matching the body, `Authorization: Bearer <access_token>`.
- The doc examples use multipart (`curl -F`); the plugin sends `application/x-www-form-urlencoded`. Both are accepted per the index page.
- Documented HTTP status codes: `200 OK` (succeeded), `400 Bad Request` (unsucceeded). Error body shape: not documented in official docs (checked 2026-10-21).
- Rate limits, IP allowlists, idempotency headers: not documented in official docs (checked 2026-10-21).

## Endpoint Catalogue

| # | Method | Path (live host) | Purpose | Docs status |
|---|---|---|---|---|
| 1 | POST | `https://app.lekirpay.com/oauth/token` | Get Bearer access token (client_credentials) | Listed |
| 2 | POST | `https://app.lekirpay.com/api/payments/payment` | Create payment, returns `payment_url` | Listed |
| 3 | POST | `https://app.lekirpay.com/api/payments/payment/secure` | Create payment with `checksum` (Lekir-Signature) | Listed |
| 4 | POST | `{callback_url}` (merchant) | LekirPay notifies merchant of success/failure | Listed |
| 5 | GET | `https://app.lekirpay.com/api/payments/{payment_id}/status` | Requery payment status | Listed |
| 6 | POST | `https://app.lekirpay.com/api/payment_group/create` | Create payment group | Listed |
| 7 | GET | `https://app.lekirpay.com/api/payment_group/data` | List payment groups | Listed |
| 8 | POST | `https://lekirpay.co/payments/payout` | Bulk payment / payout | Hidden in nav (HTML comment); legacy host does not resolve |
| 9 | POST | `https://lekirpay.co/payments/transfer` | Wallet-to-wallet transfer request | Hidden in nav (HTML comment); legacy host does not resolve |

Not documented (checked 2026-10-21): refund, cancel, capture, void, recurring/subscription, card tokenisation, settlement report, transaction list, webhook registration API.

## 1. Access Token

`POST https://app.lekirpay.com/oauth/token`

| Field | Type | Position | Required | Value |
|---|---|---|---|---|
| `grant_type` | string | Body | Yes | `client_credentials` |
| `client_id` | string | Body | Yes | Your ID (client_id) |
| `client_secret` | string | Body | Yes | Your Lekirkey |

Example:

```bash
curl -X POST https://app.lekirpay.com/oauth/token \
  -F grant_type=client_credentials \
  -F client_id=<CLIENT_ID> \
  -F client_secret=<LEKIRKEY>
```

Response:

| Field | Example | Notes |
|---|---|---|
| `token_type` | `Bearer` | |
| `expires_in` | `1296000` | Seconds (15 days) in the doc example |
| `access_token` | JWT string | Send as `Authorization: Bearer <access_token>` |

Cache the token server-side and refresh before `expires_in`. The plugin requests a new token per checkout and per callback; that works but is not required.

The hidden `/doc/introtoken` page and the 2018 sample client describe an OAuth2 authorization-code flow (`/oauth/authorize`, "callback url under API Setting"). The current listed docs and the plugin use `client_credentials` only. Use `client_credentials`.

## 2. Create Payment

`POST https://app.lekirpay.com/api/payments/payment`

Banner on the page: "Your profile needs to be verified before using this APIs".

| Field | Type | Position | Required | Notes |
|---|---|---|---|---|
| `Content-Type` | string | Header | No | `application/json` |
| `Accept` | string | Header | No | `application/json` |
| `Authorization` | string | Header | Yes | `Bearer <access_token>` |
| `item` | string | Body | Yes | Item name, shown as payment title |
| `description` | string | Body | No | Payment description |
| `amount` | double | Body | Yes | Payment amount in ringgit (example `1`; plugin sends e.g. `"10.00"`) |
| `reference_no` | string | Body | Yes | Your unique system reference, `<= 140 characters`; usable for status checks |
| `quantity` | string | Body | No | Item quantity |
| `callback_url` | string | Body | Yes | Webhook URL; LekirPay POSTs a payment object after the transaction completes |
| `redirect_url` | string | Body | No | Customer redirect after payment; if absent the customer sees the LekirPay receipt |
| `cancel_url` | string | Body | No | Customer redirect if the payment is cancelled |
| `method` | string | Body | No | `GET` or `POST`: how data is sent on redirect. Default `POST` (legacy compatibility) |
| `phone` | number | Body | No | Customer phone |
| `email` | string | Body | Yes | Customer email |
| `name` | string | Body | Yes | Customer name |
| `charge_to_payer` | string | Body | No | `"true"` or `"false"`; transfer the total transaction fee to the payer |
| `group_id` | string | Body | No | Payment group; default group if absent |

`currency` appears in the page source but is commented out (hidden). The website states "MYR Only". Do not send `currency` unless LekirPay confirms it.

Example:

```bash
curl -X POST https://app.lekirpay.com/api/payments/payment \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  -F amount=1 \
  -F reference_no=ORDER99181TEST1 \
  -F 'item=baju kemeja' \
  -F callback_url=https://www.example.com/callback \
  -F email=example@lekirpay.com \
  -F name=example
```

Response:

```json
{
  "payment_id": "833711424511",
  "reference_no": "2",
  "payment_url": "https://app.lekirpay.com/public/pay/833711424511",
  "status": "created",
  "created_date": {
    "date": "2018-07-21 05:28:25.000000",
    "timezone_type": 3,
    "timezone": "Asia/Kuala_Lumpur"
  }
}
```

| Field | Notes |
|---|---|
| `payment_id` | LekirPay payment id (string). Store it; needed for status requery |
| `reference_no` | Echo of your reference |
| `payment_url` | Hosted payment page; redirect the customer here |
| `status` | `created` on success. The plugin treats anything else as a creation failure |
| `created_date` | Object with `date`, `timezone_type`, `timezone` |

## 3. Create Secure Payment (Lekir-Signature)

`POST https://app.lekirpay.com/api/payments/payment/secure`

Same headers and body fields as section 2, plus:

| Field | Type | Position | Required | Notes |
|---|---|---|---|---|
| `checksum` | string | Body | Yes | HMAC-SHA256 hex of the source string, keyed with your Lekir-Signature key |

### Checksum algorithm

Docs (`/doc/payment/secure`):

1. Build the source string from key+value pairs, joined by `|`. Each element is the field name immediately followed by its value (no `=`).
2. The documented format is: `amount|email|item|name|callback_url|reference_no`.
3. HMAC-SHA256 the source string with the Lekir-Signature key from the profile page. Put the hex digest in `checksum`.

Exact string-to-sign:

```text
amount{amount}|email{email}|item{item}|name{name}|callback_url{callback_url}|reference_no{reference_no}
```

The docs prose says elements are "sorted in ascending order, case-insensitive", but the documented format and worked example use the fixed order above, which is not alphabetical. The official plugin uses the same fixed order. Use the fixed order.

Documented worked example (verified 2026-10-21: recomputing gives the documented digest):

```text
key:    uxDSFkVdywUiIUgX
string: amount1|emailexample@lekirpay.com|itembaju kemeja|namenurhidayah|callback_urlwww.example.com/callback|reference_noORDER99181TEST1
hmac:   2fc72fe487b1acfba6d682e588bed4baaeff43d21e7377107f09e613511de7aa
```

Note: the worked example signs `name=nurhidayah` and `callback_url=www.example.com/callback` (no scheme), while the curl example on the same page sends `name=example` and `https://www.example.com/callback` with a different checksum. Sign exactly the values you send, byte for byte (same amount formatting, same URL including scheme and query string).

Pseudo-code:

```python
import hmac, hashlib

def lekirpay_checksum(p: dict, signature_key: str) -> str:
    s = (
        "amount" + str(p["amount"]) +
        "|email" + p["email"] +
        "|item" + p["item"] +
        "|name" + p["name"] +
        "|callback_url" + p["callback_url"] +
        "|reference_no" + p["reference_no"]
    )
    return hmac.new(signature_key.encode(), s.encode(), hashlib.sha256).hexdigest()

# unit test against the doc example
assert lekirpay_checksum({
    "amount": "1", "email": "example@lekirpay.com", "item": "baju kemeja",
    "name": "nurhidayah", "callback_url": "www.example.com/callback",
    "reference_no": "ORDER99181TEST1",
}, "uxDSFkVdywUiIUgX") == "2fc72fe487b1acfba6d682e588bed4baaeff43d21e7377107f09e613511de7aa"
```

Response: same shape as section 2.

The checksum protects the create request only. LekirPay does not document any signature on the callback or redirect.

Plugin caution: in `lekirpay-givewp` 3.1.2, `lekirpayGiveAPI::sentPaymentSecure()` calls the plain `sentPayment()`, and `lekirpayGiveWPConnect::sentPaymentSecure()` posts to `/api/payments/payment` with an undefined `$sonixSignature` variable. Do not copy it; follow the docs above.

## 4. Payment Callback Notification

LekirPay POSTs to your `callback_url` "upon a successful transaction or fail".

| Field | Description |
|---|---|
| `payment_id` | Assigned payment id from LekirPay |
| `reference_no` | Your unique reference |
| `trx_no` | Assigned transaction number from LekirPay |
| `merchant_name` | Merchant name registered with LekirPay |
| `amount` | Payment amount |
| `name` | Customer name |
| `email` | Customer email |
| `phone` | Customer phone |
| `item` | Purchased item details |
| `timestamp` | Purchase time |
| `payment_method` | Payment method selected by the payer |
| `status` | Transaction status |
| `type` | `callback` or `redirect` |

Behaviour from the official plugin (`return_listener`):

- The same handler serves `callback_url`, `redirect_url` and `cancel_url`. It reads `payment_id`, `reference_no`, `status`, `type` from `$_POST`.
- It ignores the posted `status` and calls `GET /api/payments/{payment_id}/status` with a fresh token, then marks paid only if the returned `status == 'Paid'`.
- For `type=redirect` it sends the customer to a success page when `Paid`, else a failure page.
- It protects the listener URL with a random passphrase query parameter. A secret, unguessable callback path is a useful extra layer; it does not replace the requery.

Not documented (checked 2026-10-21): signature or checksum on the callback, retry schedule, required response body/status, source IPs, content type of the POST (plugin reads it as form data). Return HTTP 200 quickly and make processing idempotent.

When `method=GET`, redirect data arrives as query parameters instead of POST body (per the `method` field description). Treat both as untrusted.

## 5. Payment Status

`GET https://app.lekirpay.com/api/payments/{payment_id}/status`

| Field | Type | Position | Required | Notes |
|---|---|---|---|---|
| `Content-Type` | string | Header | Yes | |
| `Accept` | string | Header | Yes | |
| `Authorization` | string | Header | Yes | `Bearer <access_token>` |
| `payment_id` | string | Path (doc table also lists Body) | Yes | The plugin sends it in the path only |

Example:

```bash
curl -X GET https://app.lekirpay.com/api/payments/833711424511/status \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  -H 'Accept: application/json'
```

Response:

```json
{
  "payment_id": "833711424511",
  "reference_no": "2",
  "item": "3",
  "amount": 1,
  "currency": "USD",
  "quantity": 4,
  "payer_id": 0,
  "status": "created"
}
```

| Field | Notes |
|---|---|
| `payment_id` | Must equal the stored id |
| `reference_no` | Must equal your local reference |
| `item` | Item name |
| `amount` | Number; compare to expected amount with decimal-safe logic |
| `currency` | Doc example shows `USD` (legacy example); expect MYR in practice |
| `quantity` | |
| `payer_id` | |
| `status` | See enum below |

Status values (`/doc/paymentstatus`):

| Status | Meaning |
|---|---|
| `Created` | Payment link created, no payment yet |
| `Proceed` | Customer is mid-process between gateway and bank |
| `Paid` | Paid successfully by the customer |
| `Failed` | Failed during payment |

Case varies across sources (`created` in responses, `Paid` in the plugin check). Normalise to lower case before comparing. Treat `Proceed` as pending; do not mark failed early.

Lookup by `reference_no` alone: not documented (the create page says reference_no "may" be used to check status, but the only documented status endpoint takes `payment_id`).

## 6. Payment Groups

Groups label payments in the dashboard. Pass `group_id` when creating a payment.

### Create group

`POST https://app.lekirpay.com/api/payment_group/create`

| Field | Type | Position | Required | Notes |
|---|---|---|---|---|
| `Authorization` | string | Header | Yes | `Bearer <access_token>` |
| `Content-Type`, `Accept` | string | Header | No | `application/json` |
| `name` | string | Body | Yes | |
| `description` | string | Body | Yes | |

Response:

```json
{
  "name": "Bills",
  "description": "Utilities bills",
  "group_id": "2jhnt9fz2m",
  "status": "Active",
  "created_at": { "date": "2019-12-24 14:33:34.000000", "timezone_type": 3, "timezone": "Asia/Kuala_Lumpur" }
}
```

### List groups

`GET https://app.lekirpay.com/api/payment_group/data` (headers `Content-Type`, `Accept`, `Authorization` required)

Response: `code` (200), `status` (true), `data[]` with `id`, `merchant_id`, `group_id`, `name`, `description`, `status`, `created_by`, `updated_by`, `created_at`, `updated_at`, `deleted_at`.

The doc curl examples for both group endpoints omit the Authorization header; the tables mark it required. Send it.

## 7. Hidden Legacy Pages (do not build on)

These pages are reachable by URL but removed from the docs navigation (HTML comments) and point at `https://lekirpay.co`, which did not resolve on 2026-10-21. Listed for completeness only.

Bulk payment `POST https://lekirpay.co/payments/payout`: headers `Content-Type`, `Accept`, `Authorization: Bearer`; body `reference_no`, `items[i][item_id]`, `items[i][recipient]`, `items[i][currency]`, `items[i][amount]`, `items[i][description]` (all required). Response: `reference_no`, `payment_id`, `payer_id`, `total_recipient`, `valid_recipient{count,items}`, `invalid_recipient{count,items}`, `amount`, `fee`, `total_amount`.

Transfer request `POST https://lekirpay.co/payments/transfer`: body `pay_to` (wallet account), `payer_id` (wallet account), `amount`, `currency` (e.g. RM, USD, EURO), `reference_no`, `item`, `redirect_url`, `cancel_url` (required); `quantity`, `description` (optional). Response like create payment, with `payment_url` on `staging.lekirpay.co`.

Website mentions "Payout" and "Recurring Billing" as dashboard features; no current API for them is documented.

## Amount Rules

- `amount` is a ringgit value (`double`), e.g. `1` or `10.00`. Not sen.
- Decimal places and minimum/maximum amount: not documented (checked 2026-10-21).
- Format the amount once (e.g. two decimals) and use that same string in the body and the checksum.

## Refunds, Recurring, Tokenisation

- Refund API: not documented. The merchant terms (section 7 Refund Policy) say refunds are debited from the merchant account and requested by an authorised person through LekirPay's system or on letterhead.
- Recurring billing: website feature, no API documented.
- Card tokenisation / saved cards: not documented.

## Sandbox Test Data

No test cards, test bank credentials, or simulated statuses are published (checked 2026-10-21).

## Common Errors And Gotchas

- 400 on create: check profile verification, required fields, and `reference_no` length/uniqueness. Error body not documented; log the raw response.
- Checksum mismatch: wrong field order, alphabetical sorting, `=` between key and value, URL without/with scheme mismatch, different amount formatting.
- Token expired: request a new one; cache with expiry.
- Callback "paid" but API says otherwise: trust the API.
- Doc examples show `reference_no: "2"` and `currency: "USD"` in responses; these are stale sample values, not behaviour to rely on.
- The status page text says "return from SoniXPay" and plugin settings use `sonix_signature_key`; LekirPay docs are a rebrand of an older platform. Use LekirPay hosts only.
