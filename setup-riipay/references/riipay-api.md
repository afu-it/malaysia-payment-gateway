# Riipay API Reference

Use this reference for factual Riipay API guidance. All sources checked 2026-10-21. The public developer guide is short (4 pages); facts it does not state are marked "Not documented in official docs (checked 2026-10-21)". Behaviour taken from the official WooCommerce plugin is labelled "(plugin)".

## Source URLs

- Developer guide index: `https://developers.riipay.my/llms.txt`
- Introduction: `https://developers.riipay.my/master.md`
- Getting Started: `https://developers.riipay.my/getting-started.md`
- Payment Request: `https://developers.riipay.my/endpoints/payment-request.md`
- Transaction Requery: `https://developers.riipay.my/endpoints/transaction-requery.md`
- Guide source repo (same content): `https://github.com/riipay/riipay-gitbook`
- Official plugin: `https://wordpress.org/plugins/riipay-for-woocommerce/` (v1.0.27, `src/riipay.php`)
- Customer flow: `https://docs.riipay.my/merchant/payment-experience`

## Product Model

Riipay is a Malaysian Buy Now Pay Later provider. The shopper pays one third at checkout and the rest in monthly instalments at 0% interest, using a debit or credit card. The merchant integrates one payment method ("Riipay") through a hosted redirect page.

Payment process (developer guide Introduction):

1. Customer places an order and selects Riipay.
2. Merchant redirects the customer to the Riipay payment page with all request data as a GET request. Riipay verifies the request signature.
3. New Riipay users fill in basic information (phone number, OTP, details).
4. Customer is redirected to a bank-hosted payment page to pay.
5. Customer is redirected to the merchant return URL with response data. The merchant must verify the response signature.
6. Riipay also sends an asynchronous callback to the merchant callback URL with the same response data, in the form configured in the merchant portal.
7. Merchant shows a thank-you page.
8. A failed transaction means the customer retries the whole process.

Customer guide adds: the customer receives an email when the payment is being processed and another email 5 to 10 minutes later with the purchase status. On failure the customer is redirected to the store.

## Environments

| Environment | Payment base URL | Merchant portal |
| --- | --- | --- |
| Staging (no real transactions) | `https://secure.uat.riipay.my` | `https://merchant.uat.riipay.my` |
| Production | `https://secure.riipay.my` | `https://merchant.riipay.my` |

The docs use `{{BASE_URL}}` for the payment base URL. Portal profile edit page (plugin): `/profile/update` on each portal host.

## Authentication

No API key header or OAuth. Every request carries an MD5 `signature` built from the Merchant Code and Secret Key, both found on the merchant portal profile page.

## Endpoint Catalogue

| Method | Path | Purpose | Source |
| --- | --- | --- | --- |
| GET (browser redirect) | `/v1/payment` | Initiate a payment request; renders the Riipay payment page | Developer guide |
| GET | `/v1/query/transaction` | Requery a transaction status by Riipay `transaction_reference` | Developer guide |
| GET (merchant endpoint) | your Return URL | Browser return with response parameters (query string) | Developer guide |
| GET / POST form / POST JSON (merchant endpoint) | your Callback URL | Server-to-server callback with response parameters | Developer guide |
| GET (browser popup) | `/v1/payment/preview?merchant_code=...&amount=...` | Instalment preview page ("More info" link) | Plugin only, not in the guide |

No other endpoints are documented: refund, cancel, void, capture, recurring, tokenisation, payout, and settlement report APIs are Not documented in official docs (checked 2026-10-21).

## Payment Request: `GET {BASE_URL}/v1/payment`

Redirect the customer's browser to this URL with query parameters. Response: the customer sees the Riipay payment page (HTTP 200).

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `merchant_code` | string | yes | Your Merchant Code |
| `reference` | string | yes | Your unique order reference, can be the order ID. Part of the signature |
| `description` | string | yes (plugin always sends) | Order description, e.g. `Payment for order 123` |
| `currency_code` | string | yes | e.g. `MYR`. Part of the signature |
| `amount` | number (send as string) | yes | 2 decimals, no thousand separator, e.g. `12.00`, `12345.60`. Minimum `1.00`. Part of the signature |
| `signature` | string | yes | MD5 request signature (below) |
| `customer_name` | string | listed | Customer name |
| `customer_email` | string | listed | Customer email |
| `customer_phone` | string | listed | Customer phone, e.g. `0123456789` |
| `customer_ip` | string | listed | Customer IP address |
| `return_url` | string | optional | Overrides the portal default Return URL |
| `callback_url` | string | optional | Overrides the portal default Callback URL |

The guide does not mark individual fields as required or optional except `return_url` and `callback_url` ("Submit this field if you wish to override"). The official plugin sends all 12 fields on every request.

Encoding: URL-encode every value (the guide sample uses `+` for spaces and `%40` for `@`). The guide sample URL contains a typo (`&=amount=1234.00`); the correct form is `&amount=1234.00`. The plugin concatenates values without encoding; do not copy that.

### Request signature

MD5 of the plain concatenation, in this order, with no separators:

1. `merchant_code`
2. Secret Key
3. `reference`
4. `currency_code`
5. `amount` (string, 2 decimals, no thousand separator)

```text
signature = lowercase_hex( md5( merchant_code + secret_key + reference + currency_code + amount_2dp ) )
```

Verified sample from the guide (sandbox example values, not real credentials):

- merchant_code `TEST`, Secret Key `a1b2c3d4e5f6`, reference `SO20201109-01`, currency `MYR`, amount `1234.00`
- input `TESTa1b2c3d4e5f6SO20201109-01MYR1234.00`
- output `759c1d9805ba0f4bf624098a36258cb3` (recomputed and matches)

```python
import hashlib
from urllib.parse import urlencode

def riipay_payment_url(base, merchant_code, secret_key, order):
    amount = f"{order.total:.2f}"          # use Decimal in real code
    sig = hashlib.md5(f"{merchant_code}{secret_key}{order.reference}MYR{amount}".encode()).hexdigest()
    params = {
        "merchant_code": merchant_code, "reference": order.reference,
        "description": order.description, "currency_code": "MYR", "amount": amount,
        "customer_name": order.name, "customer_email": order.email,
        "customer_phone": order.phone, "customer_ip": order.ip,
        "return_url": order.return_url, "callback_url": order.callback_url,
        "signature": sig,
    }
    return f"{base}/v1/payment?{urlencode(params)}"
```

## Payment Response (Return and Callback)

Two response types carry the same fields:

- Client-to-server return (Return URL): always a GET with query parameters. It may fail depending on the customer's network or device.
- Server-to-server callback (Callback URL): GET with query parameters, POST `application/x-www-form-urlencoded`, or POST JSON, as set in the merchant portal. Default is POST JSON.

Callbacks are only sent for completed transactions with a gateway "success" or "failed" result. When there is no payment attempt (for example customer cancel or invalid signature) only the browser return happens.

### Response fields

| Field | Type | In signature | Notes |
| --- | --- | --- | --- |
| `merchant_code` | string | (your own value) | Your Merchant Code |
| `reference` | string | yes | Your order reference |
| `description` | string | no | Order description |
| `currency_code` | string | yes | e.g. `MYR` |
| `amount` | string | yes | 2-decimal ringgit. The JSON sample sends a number (`1234`); format to `1234.00` before hashing |
| `transaction_reference` | string | yes | Riipay transaction reference, e.g. `RP-20201109-ABCDEFGH`. Store it; requery needs it |
| `transaction_datetime` | string | no | e.g. `2020-11-20 15:32:12`; may be empty |
| `status_code` | string | yes | `F`, `A`, or `S` |
| `status_message` | string | no | Status description |
| `error_code` | string | no | Empty if not applicable |
| `error_message` | string | no | Empty if not applicable |
| `signature` | string | n/a | MD5 response signature |

### Response signature

MD5 of the plain concatenation, in this order:

1. Merchant Code (your configured value)
2. Secret Key
3. `reference`
4. `currency_code`
5. `amount` (2 decimals, no thousand separator)
6. `transaction_reference`
7. `status_code`

```text
expected = lowercase_hex( md5( merchant_code + secret_key + reference + currency_code + amount_2dp + transaction_reference + status_code ) )
valid    = constant_time_equals(expected, response.signature)
```

Verified sample from the guide: the sample response below uses Secret Key `a1b2c3d4e5` (the guide text says so; note it differs from the request sample key). Input `TESTa1b2c3d4e5SO20201109-01MYR1234.00RP-20201109-ABCDEFGHF` gives `fe3c5fb7596fec4afeadd05abe4316ff` (recomputed and matches).

```json
{
  "merchant_code": "TEST",
  "reference": "SO20201109-01",
  "description": "Payment for order SO20201109-01",
  "currency_code": "MYR",
  "amount": 1234,
  "transaction_reference": "RP-20201109-ABCDEFGH",
  "transaction_datetime": "",
  "status_code": "F",
  "status_message": "Failed",
  "error_code": "405",
  "error_message": "User Limit Exceeded",
  "signature": "fe3c5fb7596fec4afeadd05abe4316ff"
}
```

The sample pairs `error_code` `405` with "User Limit Exceeded"; the error table lists 405 as "Payment not Allowed" and 403 as "User Limit Exceeded". Key on `error_code`, not the message.

The guide's PHP example hashes the amount taken from the response. The official plugin instead hashes its own stored order id, currency, and order total with the response `transaction_reference` and `status_code`. Use the plugin approach: it also proves the amount matches.

```python
import hashlib, hmac

def verify_riipay_response(data, merchant_code, secret_key, payment):
    amount = f"{payment.expected_amount:.2f}"
    src = (merchant_code + secret_key + payment.reference + payment.currency + amount
           + data.get("transaction_reference", "") + data.get("status_code", "").upper())
    expected = hashlib.md5(src.encode()).hexdigest()
    return (hmac.compare_digest(expected, str(data.get("signature", "")).lower())
            and data.get("reference") == payment.reference
            and data.get("merchant_code") == merchant_code
            and data.get("currency_code") == payment.currency)
```

### Status codes

| Code | Meaning | Action |
| --- | --- | --- |
| `S` | Success. Transaction is successful | Mark paid (idempotent) |
| `A` | Accepted. Received and under process | Keep pending / on hold; confirm by callback or requery |
| `F` | Failed | See error code handling |

The plugin rejects any other `status_code` value.

### Error codes

| Code | Description |
| --- | --- |
| 400 | Invalid Request Parameter |
| 401 | Invalid Signature |
| 402 | Merchant Limit Exceeded |
| 403 | User Limit Exceeded |
| 404 | Invalid Merchant Code |
| 405 | Payment not Allowed |
| 406 | Payment Expired |
| 409 | Invalid Session |
| 410 | Cancelled by Customer |
| 412 | Merchant Invalid Callback |
| 422 | Payment Failed |
| 429 | Payment duplicated (plugin comment only, not in the guide table) |
| 500 | Server Error |
| 501 | Failed to Initialize |
| 502 | Gateway Request Error |
| 503 | Gateway Response Error |
| 504 | Gateway Signature Error |

Plugin handling of `F` (v1.0.27): it adds an order note with `error_code: error_message`, and only sets the order to failed when the code is NOT in `400, 401, 402, 404, 409, 410, 412, 429, 500, 501, 502, 503, 504`. For `410` it redirects the customer back to the order pay page. So in practice only `403`, `405`, `406`, `422` (and unknown codes) close the order as failed.

Plugin handling of `A` and `S`: verify signature first; `A` sets the order on-hold; `S` calls payment complete with `transaction_reference`. It skips processing when the order is already processing, completed, or refunded (idempotency). Invalid signature adds a note and does not change status.

### Callback acknowledgement

The expected HTTP response to a callback, and whether Riipay retries, is Not documented in official docs (checked 2026-10-21). The plugin answers a valid callback with HTTP 200 and a JSON success body, and errors with 4xx JSON. Return HTTP 200 after you have durably recorded the result.

## Transaction Requery: `GET {BASE_URL}/v1/query/transaction`

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `merchant_code` | string | yes | Your Merchant Code |
| `transaction_reference` | string | yes | Riipay transaction reference from the return or callback |
| `signature` | string | yes | MD5 query signature |

Query signature, in order: Merchant Code, Secret Key, `transaction_reference`.

```text
signature = lowercase_hex( md5( merchant_code + secret_key + transaction_reference ) )
```

Response: the same fields as the payment response, as JSON. Verify its `signature` with the response signature rule.

Rate limit: all query endpoints allow at most 2 queries per second.

Gap: requery by your own `reference` is Not documented in official docs (checked 2026-10-21). If neither a return nor a callback arrives you have no `transaction_reference`; contact Riipay or check the merchant portal.

Observed (not documented): an unsigned request to the staging requery endpoint returned HTTP 401 JSON `{"name":"Unauthorized","message":"Invalid Signature","code":0,"status":401,...}`.

## Amount And Currency Rules

- Unit: ringgit, not sen. Always a string with 2 decimals, `.` as the decimal point, no thousand separator.
- Minimum `1.00`.
- Currency: the guide says "e.g. MYR"; the plugin only enables Riipay when the store currency is `MYR` (filter `riipay_supported_currencies`).
- Maximum: the plugin readme says orders above RM1000 are rejected "for the time being until further notice" and defaults Maximum Order Amount to 1000. Confirm with Riipay.
- Instalments: plugin display text assumes the first payment is total / 3, remainders go to the first repayment. Plugin display setting allows 2 to 12 instalments for the price text only.

## Sandbox

- Use the staging base URL and staging portal with staging credentials. "There are no real transactions happening on it."
- Test cards, test phone numbers, and OTP values are Not documented in official docs (checked 2026-10-21). Ask Riipay.
- The plugin offers a "Only visible to admins" visibility setting for testing on a live store.

## Other Gaps (Not documented in official docs, checked 2026-10-21)

- Refund, partial refund, cancel, void API. Merchant FAQ: refunds follow the merchant's policy; once approved the customer is credited.
- Webhook IP allowlist and callback source IPs.
- Callback retry schedule and timeout.
- Whether a `reference` can be reused after a failed attempt (the plugin reuses the order id on retry).
- Payment expiry window (error `406` exists but no duration is given).
- Settlement timing and reports.

## Common Errors

- `401 Invalid Signature`: wrong field order, amount not 2 decimals (`1234` vs `1234.00`), wrong environment key, or hashing URL-encoded values. Hash raw values, then URL-encode.
- `404 Invalid Merchant Code`: staging code used on production or the reverse.
- `412 Merchant Invalid Callback`: callback URL missing or rejected; check portal profile and the `callback_url` override.
- Signature mismatch on callback: response amount not re-formatted, or comparing against `status_message`.
- Order stuck pending: customer cancelled (`410`, return only, no callback) or return lost; requery with the stored `transaction_reference`.
