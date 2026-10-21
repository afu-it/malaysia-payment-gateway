# FavePay Omni (FPO) API Reference

Distilled from the official FPO API Guide and the official FavePay for WooCommerce plugin source (by Fave Asia, v1.0.17). Checked 2026-10-21. The guide says it reflects behaviour "as of 20 October 2026", is a live reference with no revision number, and that partners should contact their Fave technical contact if something looks off.

## Source URLs

- FPO API Guide (single page, all endpoints): `https://developers.myfave.com/fpo-guide.html`
- Partner docs index: `https://developers.myfave.com/`
- FavePay for WooCommerce plugin: `https://wordpress.org/plugins/favepay-for-woocommerce/`
- Plugin source zip: `https://downloads.wordpress.org/plugin/favepay-for-woocommerce.latest-stable.zip`

Anchors used below are sections of the FPO guide, for example `fpo-guide.html#auth`.

## Integration Surfaces

| Surface | Use | Notes |
| --- | --- | --- |
| Dynamic QR online, `format=web_url` | E-commerce hosted checkout | Customer pays on Fave's page, returns to `redirect_url`, webhook confirms. |
| Dynamic QR offline, `txt` / `base64` / `favepay` | POS shows one-time QR | Customer scans with the Fave app. |
| Wallet-specific QR, `grab` / `alipay` / `wcp` / `shopeepay` / `atome` / `paynow` | Single wallet acceptance | Availability depends on country and what Fave enabled. |
| Merchant scan | POS scans customer QR or barcode | Synchronous charge, returns transaction. |
| Static QR | One persistent outlet QR, amount entered on phone | Planned (target Q3); needs Acknowledge. |
| FavePay for WooCommerce | WordPress store | Uses `web_url` and a different redirect format. |

Source: `fpo-guide.html#flows`, `#qr-formats`.

## Environments

| Environment | Base URL |
| --- | --- |
| Production | `https://omni.myfave.com` |
| Sandbox | `https://omni.dino.fave.ninja` |
| FaveBiz sandbox (optional, Staff Mode cross-check) | `https://omni-staging.favebiz.fave.ninja` |

- All paths are under `/api/fpo/v1/<country_code>/`. Live markets: `my` (Malaysia), `sg` (Singapore). The docs index also lists `ID`; the WooCommerce plugin maps `MYR` to `my`, `SGD` to `sg`, `IDR` to `id`.
- Currency follows the country code in the path. There is no currency request field.
- Sandbox `app_id`, secret, `outlet_id` and `omni_reference` prefix are issued by Fave and differ from production. Staging keys are never promoted to production.
- The plugin sends `Accept: application/json` and `Content-Type: application/json`, JSON bodies for POST, query parameters for GET, 30 second timeout.

Source: `fpo-guide.html#overview`, `#uat`; plugin `includes/abstracts/abstract-favepay-wc-client.php`, `includes/functions.php`.

## Authentication And Signing

There are no auth headers. Every request carries a `sign` field.

- Algorithm: HMAC-SHA256, output lowercase hex (64 chars).
- Key: the secret key Fave issued for your integration. Some setups use an outlet-specific secret; use the key Fave confirms for your account.
- Message: the request fields URL-encoded as `key=value` pairs joined with `&`, in the same order you send them. Do not sort.
- Always exclude `sign` and `country_code`.
- Also exclude: `outlet_id` when it is in the path (`/outlets/<outlet_id>/transactions`); `id` when acknowledging via `/transactions/<id>`; `details` on merchant scan.
- Nested objects flatten to bracket form, for example `shopper_details[email]=...`.
- Values are URL-encoded with spaces as `+` (not `%20`).

Documented example signing string (`#auth`):

```
omni_reference=PL-0001&total_amount_cents=1000&app_id=a3osyvuayt&outlet_id=11637&format=txt
```

Sent body:

```json
{
  "omni_reference": "PL-0001",
  "total_amount_cents": 1000,
  "app_id": "a3osyvuayt",
  "outlet_id": 11637,
  "format": "txt",
  "sign": "4e23af6ca0c0831e9fe3524ac7943d5e8860f0843a343f7c9c4a3f172b408972"
}
```

(The docs do not publish the key used for this request example, so it cannot be reproduced.)

### Encoding rules (from the official plugin)

The plugin builds the string with PHP `http_build_query($params)` (RFC 1738, same as `urlencode`) and signs with `hash_hmac('sha256', $string, $api_key)`. To match it in other languages:

- Keep `A-Z a-z 0-9 - _ .` as is; space becomes `+`; every other byte is `%XX` with uppercase hex (UTF-8 bytes). This means `~` becomes `%7E` and `*` becomes `%2A`.
- Nested arrays become `parent%5Bchild%5D=value` (PHP encodes the brackets). The docs show the unencoded `shopper_details[email]=` form; confirm with Fave if you sign nested fields from a non-PHP stack. Simplest: omit `shopper_details` unless needed.
- PHP drops keys whose value is `null`, writes `true` as `1` and `false` as `0`. The docs do not say how booleans (`test`, `ack`) and nulls are signed; when in doubt send `test` and `ack` as the strings `"true"` / `"false"` and sign the same string, then confirm in sandbox (UAT case A1).
- Integers and their string form encode the same, so `1000` and `"1000"` give the same signature.

Source: plugin `abstract-favepay-wc-client.php` `generate_api_signature()`; `fpo-guide.html#auth`, `#payload-verification`.

### Pseudo-code

```python
import hmac, hashlib, urllib.parse

def php_urlencode(v):
    if v is True: v = "1"          # PHP http_build_query behaviour
    if v is False: v = "0"
    return urllib.parse.quote_plus(str(v), safe="").replace("~", "%7E")

def fave_sign(fields, secret, exclude=("sign", "country_code")):
    # fields: ordered mapping, in the exact order sent or received
    pairs = [f"{php_urlencode(k)}={php_urlencode(v)}"
             for k, v in fields.items() if k not in exclude and v is not None]
    msg = "&".join(pairs)
    return hmac.new(secret.encode(), msg.encode(), hashlib.sha256).hexdigest()

body = {"omni_reference": "PL-0001", "total_amount_cents": 1000,
        "app_id": "<APP_ID>", "outlet_id": 11637, "format": "web_url",
        "redirect_url": "https://shop.example/return",
        "callback_url": "https://shop.example/webhooks/favepay"}
body["sign"] = fave_sign(body, "<FAVE_SECRET_KEY>")   # sign goes last
```

```js
// Node: PHP-urlencode compatible
const enc = (v) => encodeURIComponent(String(v))
  .replace(/[!'()*~]/g, (c) => '%' + c.charCodeAt(0).toString(16).toUpperCase())
  .replace(/%20/g, '+');
const faveSign = (fields, secret, exclude = ['sign', 'country_code']) =>
  require('crypto').createHmac('sha256', secret).update(
    Object.entries(fields).filter(([k, v]) => !exclude.includes(k) && v !== null)
      .map(([k, v]) => `${enc(k)}=${enc(v)}`).join('&')).digest('hex');
```

`JSON.parse` keeps insertion order for non-numeric keys, which is what webhook verification needs. In Python, `json.loads` returns an ordered `dict`.

## Endpoint Catalogue

All paths are relative to the base URL. `<cc>` is `my` or `sg`.

| Method | Path | Purpose | Guide anchor |
| --- | --- | --- | --- |
| POST | `/api/fpo/v1/<cc>/qr_codes` | Create a single-use QR or hosted payment URL | `#qr-api` |
| POST | `/api/fpo/v1/<cc>/qr_codes/static` | Fetch persistent outlet static QR (planned) | `#static-qr-api` |
| POST | `/api/fpo/v1/<cc>/merchant_scan` | Charge a customer-presented QR or barcode | `#merchant-scan` |
| POST | your `callback_url` | Webhook from Fave with transaction object and `sign` | `#webhook` |
| GET | `/api/fpo/v1/<cc>/transactions` | Get one transaction by `omni_reference` or `receipt_id` | `#txn-api` |
| POST | `/api/fpo/v1/<cc>/transactions/<id>` | Acknowledge a transaction | `#ack-api` |
| POST | `/api/fpo/v1/<cc>/transactions/cancel` | Cancel an unpaid QR | `#cancel-api` |
| GET | `/api/fpo/v1/<cc>/outlets/<outlet_id>/transactions` | List recent outlet transactions | `#history-api` |
| POST | `/api/fpo/v1/<cc>/transactions` | Refund (full or partial), `status=refunded` | `#refund-api` |

Not documented in official docs (checked 2026-10-21): recurring or tokenised payments, payouts, settlement report API, capture or void, customer or card storage.

## Create Payment: Request A QR Code

`POST /api/fpo/v1/<cc>/qr_codes`. Creates a single-use QR or payment URL for an outlet.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `omni_reference` | string | Yes | Your unique reference, format `[prefix]-[id]`, max 28 chars, using the prefix Fave assigned. Duplicates are rejected. |
| `total_amount_cents` | integer | Yes | Positive integer cents. Singapore max `1999900`. |
| `app_id` | string | Yes | Integration App ID from Fave. |
| `outlet_id` | integer | Yes | Fave outlet ID. |
| `format` | string | No | `txt`, `base64` (default), `web_url`, `favepay`, `grab`, `paynow`, `alipay`, `wcp`, `shopeepay`, `atome`. |
| `redirect_url` | string | No | Recommended for `web_url`. Fave appends `?omni_ref=<encrypted>`. |
| `callback_url` | string | No | Per-payment webhook URL; overrides your default callback when enabled for your account; kept for a few hours. |
| `payment_option` | string | No | `both` (default), `app`, or `pwa`. |
| `expiry_time` | integer | No | Unix timestamp for QR expiry. |
| `shopper_details` | object | No | Optional `email`, `phone`, `location`. The plugin also sends `name`, `address_1`, `address_2`, `city`, `postcode`, `state`, `country`. |
| `test` | boolean | No | Sandbox only. Returns a test page URL in `code`. Formats: `base64`, `txt`, `web_url`, `favepay`, `paynow`. |
| `sign` | string | Yes | HMAC-SHA256. |

Response:

```json
{
  "code": "https://omni.myfave.com/web_views/favepay_online/gateway?omni_ref=<token>",
  "format": "web_url",
  "expires_in": 900
}
```

| Field | Notes |
| --- | --- |
| `code` | Depends on `format`: hosted URL for `web_url`, plaintext payload for `txt`, Base64 PNG for `base64`, FavePay payload for `favepay`, wallet QR payload otherwise. In test mode always a test page URL. |
| `format` | Echo of format. |
| `expires_in` | Seconds until expiry. |

Format guide: online checkout use `web_url`; in-store Fave app scan use `txt`, `base64` or `favepay`; wallet-specific use `grab`, `paynow`, `alipay`, `wcp` (SGQR with Fave payload WeChat Pay can detect), `shopeepay`, `atome`. Unsupported formats return an error such as `service_not_supported`.

HTTP code: the UAT checklist expects 200; the WooCommerce plugin treats anything other than 201 as failure. Accept any 2XX with a `code` field. Errors return `error` / `error_description`.

Source: `fpo-guide.html#qr-api`, `#qr-formats`; plugin `includes/regular-checkout.php` `process_payment()`, `get_qr_code_params()`.

### Test mode

- Send `test=true` in sandbox. `code` becomes a test page URL, for example `/web_views/favepay_online/test?omni_ref=...`.
- Buttons on the page simulate success or failure. Ticking "Promo code applied" sends `promo_code=true` and applies promo `fpoms`.
- Wallet-specific formats do not support test mode. The test page is blocked in production. Never send `test` to production.
- A successful simulation creates a real FavePay payment on the sandbox outlet, visible in FaveBiz sandbox Staff Mode > Transactions.

Source: `fpo-guide.html#qr-api` (Use of test parameter), `#uat`.

## Redirect Back To Your Site

Standard integrations: after payment Fave redirects to `<redirect_url>?omni_ref=<encrypted_value>`.

- Decrypt with your private API key (the same key used to sign). Algorithm AES-128-CBC, Base64 encoded.
- Decrypted example: `receipt_id=0871-5110&omni_reference=PL-000023&status=successful&total_amount_cents=1000`.
- Fields: `receipt_id`, `omni_reference`, `status`, `total_amount_cents`.
- Not documented in official docs (checked 2026-10-21): the IV, padding, and key derivation for this AES-128-CBC value. Do not guess; ask Fave, or ignore `omni_ref` and call Get a transaction server-side.
- Magento, WooCommerce, Jocom and EasyStore use different redirect formats.

WooCommerce format (plugin `thankyou_page()` and `get_valid_ipn_response()`): plain GET query parameters `receipt_id`, `omni_reference`, `status`, `total_amount_cents`, `sign`; the plugin verifies `sign` over those four fields in that order. Status values seen: `successful`, `pending_payment`, `payment_processing`, anything else treated as failed.

Never fulfil from the redirect alone.

Source: `fpo-guide.html#qr-redirect-omni-ref`; plugin `regular-checkout.php`, `abstract-favepay-wc-client.php`.

## Webhooks

- Fave POSTs the transaction object (same fields as Get a transaction plus `sign`) as JSON to your callback URL when payment status changes.
- Statuses sent: `successful` (completed and accepted), `rejected` (completed but not successful). Expired unpaid QRs and cancelled QRs can trigger `rejected`.
- Register the URL with Fave at the technical kick-off meeting, or pass `callback_url` per payment.
- Return any HTTP 2XX quickly, then process in the background. (Flow diagrams say "HTTP under 400"; use 2XX.)
- Non-2XX responses are retried at increasing intervals. Handlers must be idempotent. Retry schedule: not documented (checked 2026-10-21).
- After a 2XX, Fave may auto-acknowledge dynamic QR payments. Static QR is never auto-acknowledged; call Acknowledge.
- Webhook source IPs: not documented (checked 2026-10-21).

### Callback body fields

| Field | Description |
| --- | --- |
| `id` | Transaction ID (max 35 chars), prefix shows payment type. |
| `receipt_id` | Fave receipt ID (max 35 chars). |
| `outlet_name` | Outlet display name. |
| `total_amount_cents` | Order total in cents (string). |
| `payment_type` | Category, for example `Credit Card`, `Debit Card`, `DuitNow`. Present when payment-method reporting is enabled. |
| `payment_type_variant` | Variant, for example `Visa`, `MasterCard`, `Touch N'Go eWallet`. |
| `currency` | ISO code, for example `MYR`. |
| `outlet_id` | Fave outlet ID (string). |
| `mid` | Merchant ID. |
| `omni_reference` | Your reference. |
| `status` | `successful` or `rejected`. |
| `status_code` | Numeric status (string). |
| `created_at` | ISO 8601 timestamp. |
| `campaign_credit_amount_cents` | Campaign credit redeemed. |
| `campaign_funded_by` | Who funded campaign credit. |
| `aabig_points_used_amount_cents` | AABig points redeemed. |
| `merchant_cashback_amount_cents` | Merchant cashback redeemed. |
| `fave_credits_amount_cents` | Fave credit redeemed. |
| `promo_code_value_cents` | Promo code discount. |
| `e_card_credits_used_cents` | eCard credits used. |
| `charged_amount_cents` | Card or e-wallet charged amount. |
| `cashback_rate` | Merchant cashback rate. |
| `merchant_cashback_issued_cents` | Cashback the user earned. |
| `promo_code` | Promo code used. |
| `promo_code_cashback_value` | Promo cashback value. |
| `promo_code_cashback_type` | For example `absolute_cashback`. |
| `promo_code_cashback_issued_cents` | Promo cashback earned. |
| `promo_code_cashback_funded_by` | For example `merchant`. |
| `fave_fees_percentage` | Fave fee percentage. |
| `fave_fees_cents` | Total Fave fee. |
| `sst_on_total_fees_cents` | Tax on fees. |
| `merchant_takeback_cents` | Amount the merchant receives. |
| `user_id` | Fave user ID. |
| `fpl_transaction` | `true` if FavePay Later. |
| `fpl_fees_percentage` | FPL fee percentage or `N/A`. |
| `fpl_fees_cents` | FPL fee or `N/A`. |
| `fpl_merchant_cashback_reduction_percentage` | FPL cashback reduction or `N/A`. |
| `sign` | HMAC-SHA256 over all other fields. |

Optional extra fields: `minimum_fee_applied` (with payment-method reporting), `shopper_details` (when supported). Most amounts are strings; promo and FPL fields may be `N/A`.

Note: `charged_amount_cents` can be lower than `total_amount_cents` when credits, cashback or promos are applied. Match your order against `total_amount_cents`.

### Payload verification (worked example, verified)

Build the string from all payload fields in the exact order received, excluding `sign`, URL-encoded with spaces as `+`, then HMAC-SHA256 with the private API key and compare to `sign`.

Docs example key (public example only, not a real key): `d25f1p1zf6ww8eja`.

Example payload (in order): `id=FP_792619`, `receipt_id=0118-7347`, `outlet_name=TTDI Shop`, `total_amount_cents=2100`, `currency=MYR`, `outlet_id=1593`, `mid=1166`, `omni_reference=PL-0001`, `status=successful`, `status_code=2`, `created_at=2021-03-26T18:14:59.698+08:00`, then the remaining fields as in the table above with the values from the guide.

Signing string:

```
id=FP_792619&receipt_id=0118-7347&outlet_name=TTDI+Shop&total_amount_cents=2100&currency=MYR&outlet_id=1593&mid=1166&omni_reference=PL-0001&status=successful&status_code=2&created_at=2021-03-26T18%3A14%3A59.698%2B08%3A00&campaign_credit_amount_cents=0&campaign_funded_by=N%2FA&aabig_points_used_amount_cents=0&merchant_cashback_amount_cents=350&fave_credits_amount_cents=200&promo_code_value_cents=0&e_card_credits_used_cents=0&charged_amount_cents=1550&cashback_rate=0.0&merchant_cashback_issued_cents=0&promo_code=apicbpartner2&promo_code_cashback_value=2.00&promo_code_cashback_type=absolute_cashback&promo_code_cashback_issued_cents=200&promo_code_cashback_funded_by=merchant&fave_fees_percentage=1.0&fave_fees_cents=18&sst_on_total_fees_cents=1&merchant_takeback_cents=1881&user_id=2818771&fpl_transaction=false&fpl_fees_percentage=N%2FA&fpl_fees_cents=N%2FA&fpl_merchant_cashback_reduction_percentage=N%2FA
```

Expected sign: `fc12259055f146e8bf0b587cbd8dbb9283e4db6a525243bc608e5762e43c6b29`. Reproduced on 2026-10-21 with both the Python `fave_sign` and the Node `faveSign` snippets above, run over the parsed docs payload.

Verification handler:

```python
payload = json.loads(raw_body)                 # keeps key order
received = payload.get("sign", "")
expected = fave_sign(payload, FAVE_SECRET_KEY)  # excludes sign
if not hmac.compare_digest(expected, received): return 401
# then: look up local payment by omni_reference, check amount/currency/outlet,
# settle once keyed on receipt_id, return 200
```

Source: `fpo-guide.html#webhook`, `#webhook-accept`, `#payload-verification`; plugin `handle_ipn_callback()`.

## Get A Transaction (Requery)

`GET /api/fpo/v1/<cc>/transactions`, parameters as query string (plugin uses GET query).

| Field | Required | Notes |
| --- | --- | --- |
| `app_id` | Yes | Integration App ID. |
| `omni_reference` | One of | At least one of `omni_reference` or `receipt_id`. |
| `receipt_id` | One of | Use the same `app_id` that created the payment. |
| `outlet_id` | No | Must match the payment's outlet when given. |
| `sign` | Yes | HMAC-SHA256 over the other parameters in sent order. |

Response: the transaction object (same fields as the webhook, without `sign`). For a created but unpaid QR you may get a pending object with many fields `null`. The plugin treats a response without `status` as an expired QR and issues a new `omni_reference` for the next attempt.

Source: `fpo-guide.html#txn-api`; plugin `check_transaction_status()`.

## Acknowledge A Transaction

`POST /api/fpo/v1/<cc>/transactions/<id>`. Use for static QR, or when a webhook was missed and the payment was not auto-acknowledged.

| Field | Required | Notes |
| --- | --- | --- |
| `id` (path) | Yes | Prefixed transaction ID (`FP_`, `RPP_`, `GRAB_`, `PAYNOW_`, `EXT_`). Excluded from sign. |
| `omni_reference` | Yes | Your reference. |
| `app_id` | Yes | Integration App ID. |
| `outlet_id` | Yes | Fave outlet ID. |
| `sign` | Yes | HMAC-SHA256. |

Response: transaction object (`id`, `receipt_id`, `outlet_name`, `total_amount_cents`, `currency`, `outlet_id`, `mid`, `omni_reference`, `status`, `status_code`, `created_at`, `charged_amount_cents`, `merchant_takeback_cents`, `user_id`, `fpl_transaction`).

Source: `fpo-guide.html#ack-api`.

## Cancel A QR Payment

`POST /api/fpo/v1/<cc>/transactions/cancel`. Invalidates an unpaid QR (abandoned order or amount change). Use Refund after success.

| Field | Required | Notes |
| --- | --- | --- |
| `omni_reference` | Yes | Reference used at creation. |
| `app_id` | Yes | Integration App ID. |
| `outlet_id` | No | Must match when given. |
| `sign` | Yes | HMAC-SHA256. |

- 200 `{"cancellation_status": "accepted"}`: QR cancelled; pending or processing payments marked `rejected`. Also returned when no payment record exists (safe to retry).
- 409 `{"cancellation_status": "rejected", "error": "...", "error_description": "...", "omni_reference": "..."}` with `error` one of `QR_PAID` (paid, including acknowledged; query instead), `QR_CANCELLED`, `QR_REFUNDED`.
- Wrong `app_id`: 404 `resource_not_found`. A cancelled QR may send a `rejected` webhook.

Source: `fpo-guide.html#cancel-api`.

## List Transactions

`GET /api/fpo/v1/<cc>/outlets/<outlet_id>/transactions`.

| Field | Required | Notes |
| --- | --- | --- |
| `outlet_id` (path) | Yes | Excluded from sign. |
| `app_id` | Yes | Integration App ID. |
| `limit` | No | Max 1000. |
| `timestamp` | No | Unix timestamp, not earlier than about 31 days ago. |
| `ack` | No | Default `true` (all). `false` returns only unacknowledged. |
| `sign` | Yes | HMAC-SHA256. |

- Neither `limit` nor `timestamp`: last 10 since 24 hours ago. Only `limit`: since 30 days ago.
- Response is a bare JSON array of transaction objects.
- Empty result: 404 with `error: "error"` and description `No records were found that match your query.`

Source: `fpo-guide.html#history-api`.

## Refunds

`POST /api/fpo/v1/<cc>/transactions` with `status=refunded`.

| Field | Required | Notes |
| --- | --- | --- |
| `omni_reference` | Yes | Payment to refund. |
| `app_id` | Yes | Integration App ID. |
| `status` | Yes | Must be `refunded`. |
| `partial_refund_cents` | No | Partial amount in cents; cannot exceed `charged_amount_cents`; not supported for DuitNow or PayNow. |
| `sign` | Yes | HMAC-SHA256. |

- Response: transaction object with `status: "refunded"`, `status_code: "5"`, `refund_amount_cents`, plus fee fields.
- Window: typically same calendar day as the payment; after that 400 `stale_transaction`. Some merchants have a different window.
- Already refunded: HTTP 200 with the transaction object (also named `already_refunded`); treat as success.
- The WooCommerce plugin only does full refunds, expects HTTP 201, and tells merchants that failed refunds need the customer to contact Fave.

Source: `fpo-guide.html#refund-api`, `#errors`; plugin `process_refund()`.

## Merchant Scan

`POST /api/fpo/v1/<cc>/merchant_scan`. POS scans the customer's QR or barcode.

| Field | Required | Notes |
| --- | --- | --- |
| `omni_reference` | Yes | Unique, max 28 chars. |
| `total_amount_cents` | Yes | Positive integer cents. |
| `app_id` | Yes | Integration App ID. |
| `outlet_id` | Yes | Fave outlet ID. |
| `code` | Yes | Customer QR or barcode payload. |
| `callback_url` | No | Per-payment webhook URL. |
| `details` | No | Arbitrary JSON metadata, excluded from sign. |
| `sign` | Yes | HMAC-SHA256 over fields excluding `details`. |

Response: transaction object (`id`, `receipt_id`, `outlet_name`, `total_amount_cents`, `currency`, `outlet_id`, `mid`, `omni_reference`, `status`, `status_code`, `created_at`, `charged_amount_cents`, `user_id`, `fpl_transaction`).

Source: `fpo-guide.html#merchant-scan`.

## Static QR (Planned)

`POST /api/fpo/v1/<cc>/qr_codes/static`. Status in docs: planned for partner integration (target Q3); confirm go-live with Fave.

| Field | Required | Notes |
| --- | --- | --- |
| `outlet_id` | Yes | Must belong to your `app_id`. |
| `app_id` | Yes | Integration App ID. |
| `format` | No | Encoding only: `txt` (default) or `base64`. |
| `sign` | Yes | In the POST body, never the query string. |

Response: `code` (EMVCo / SGQR payload or Base64 PNG), `format`, `supported_apms` (for example `favepay`, `paynow`, `alipay`, `wcp`) for logos. No amount, no `omni_reference`, no expiry. After each payment: webhook, then Acknowledge. If missed: List transactions with `ack=false`, then Acknowledge.

Source: `fpo-guide.html#static-qr-api`, `#flows`.

## Statuses And IDs

| status | status_code |
| --- | --- |
| `pending_payment` | 0 |
| `payment_processing` | 1 |
| `successful` | 2 |
| `disputed` | 3 |
| `rejected` | 4 |
| `refunded` | 5 |

| ID prefix | Payment type |
| --- | --- |
| `FP_` | FavePay / in-app |
| `RPP_` | DuitNow |
| `GRAB_` | Grab |
| `PAYNOW_` | PayNow |
| `EXT_` | External wallet or other |

Payment type values: Fave in-app gives type and variant from the method used (when enabled); DuitNow `payment_type="DuitNow"`, variant is bank name or BIC; Grab `payment_type="grab"`; PayNow from PayNow details; other wallets `payment_type`=provider name.

Source: `fpo-guide.html#ids`, `#payment-types`.

## Errors

| HTTP | error | Meaning |
| --- | --- | --- |
| 400 | `validation` / `stale_transaction` | Invalid params (for example duplicate `omni_reference`), or refund outside window. |
| 401 | `Unauthorized` | Invalid signature, or outlet not linked to this `app_id`. |
| 404 | `resource_not_found` | Unknown `app_id`, or missing transaction or outlet. |
| 404 | `error` | Empty transaction history for the query window. |
| 409 | `QR_PAID` / `QR_CANCELLED` / `QR_REFUNDED` | Cancel rejected. |
| 422 | provider / format codes | Format not supported (for example `service_not_supported`), amount limit, provider rejection. |
| 500 | `internal_error` / `unexpected_error` | Server failure. |

Also seen: `failed_request`, `already_refunded` (success body). Example: `{"error": "validation", "error_description": "Omni reference already exists. Please provide unique omni reference."}`.

Source: `fpo-guide.html#errors`.

## Reference Patterns From The Official Plugin

- `omni_reference` = `<prefix>-<outlet_id><order_id><suffix><site_id>`, where prefix is `WC<COUNTRY>` (sandbox `WCM`), suffix adds one `A` per regenerated QR, and site ID is a random 3-digit number per install. Custom integrations must use the prefix Fave assigns.
- Caches the payment URL for `expires_in` seconds (fallback 360) and reuses it on retry.
- Schedules a status requery at QR expiry, and again 15 minutes later while `pending_payment`.
- Stores `receipt_id` as the transaction ID and skips orders already marked paid (idempotency).
- Fave hosts a promo widget script for product pages: `https://partners.myfave.gdn/js/fpo_promo.js` (staging `fpo_promo_staging.js`).

Source: plugin `includes/functions.php`, `includes/regular-checkout.php`, `public/class-favepay-wc-public.php`.

## Not Documented (checked 2026-10-21)

Rate limits; webhook source IPs; webhook retry schedule and maximum attempts; `omni_ref` AES IV, padding and key handling; signing of booleans and nulls; idempotency headers; settlement or payout timing; public test cards (payments are simulated on the test page instead).
