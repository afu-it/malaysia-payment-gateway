# KineticPay API Reference

Distilled from the official KineticPay API docs (`https://docs.kineticpay.my/`) and the source of KineticPay's official WordPress plugins (WooCommerce 3.0, GiveWP 1.0.3, Gravity Forms 1.0.3, Forminator 1.0.2, all with homepage `https://kineticpay.my/`). Checked 2026-10-21.

Important: the public docs site documents only API v2 utility endpoints (`/ping`, `/bank`, `/company`). The payment flow (create bill, requery) is not on the docs site; it is taken from the official plugin source, which all four plugins implement the same way. Where a field or behaviour is not visible in either source, this file says so.

## Source URLs

- Docs home: `https://docs.kineticpay.my/`
- Quickstart: `https://docs.kineticpay.my/quickstart`
- Authentication: `https://docs.kineticpay.my/authentication`
- Pagination: `https://docs.kineticpay.my/pagination`
- Errors: `https://docs.kineticpay.my/errors`
- Sandbox: `https://docs.kineticpay.my/sandbox`
- FPX Banks: `https://docs.kineticpay.my/banks`
- Companies: `https://docs.kineticpay.my/companies`
- WooCommerce plugin: `https://wordpress.org/plugins/kineticpay-for-woocommerce/` (files `src/kineticpay.php`, `index.php`)
- GiveWP plugin: `https://wordpress.org/plugins/kineticpay-for-givewp/` (file `includes/Kineticpay_WPConnect.php`, `includes/give-kineticpay-gateway.php`)
- Gravity Forms plugin: `https://wordpress.org/plugins/kineticpay-for-gravityforms/` (file `helpers/kineticpay_wpconnect.php`, `class-gf-kineticpay.php`)
- Forminator plugin: `https://wordpress.org/plugins/kineticpay-for-forminator/` (file `classes/kineticpay-gateway.php`, `classes/public.php`)

## Environments

| Item | Value | Source |
|---|---|---|
| Payment host (bill create, requery) | `https://manage.kineticpay.my` | all plugins |
| API v2 base | `https://manage.kineticpay.my/api/v2` | Quickstart |
| Dashboard | `https://manage.kineticpay.my` | docs home |
| Register | `https://manage.kineticpay.my/register` | docs home |
| API key page | `https://manage.kineticpay.my/api-key` | docs home |
| Sandbox | Same endpoints. Sandbox is a toggle on the Collection page in the dashboard; "all the API endpoints and functionalities remain the same as in the live environment". | Sandbox |

There is no separate sandbox host. Which mode a bill runs in depends on the Collection's sandbox toggle, so the same `merchant_key` can be sandbox or live depending on that toggle. Note: the docs' cURL examples use `http://www.kineticpay.test/api/v2/ping`, a local dev host; use the documented base URL instead.

## Authentication

Two credentials exist:

| Credential | Used by | How sent | Where to get it |
|---|---|---|---|
| Merchant key (per Collection) | `/payment/create`, `/payment/status` | `merchant_key` in JSON body (create) or query string (status) | Dashboard Collection page. "Key will be generated automatically after you fill up Collection Name and your website url." (WooCommerce readme FAQ) |
| API key | API v2 (`/api/v2/*`) | `Authorization: Bearer <API_KEY>` | Dashboard > API Key (`https://manage.kineticpay.my/api-key`) |

API v2 requests must also send `Content-Type: application/json` and `Accept: application/json` (Quickstart). An invalid or missing bearer key returns HTTP 401 `{"message":"Unauthenticated."}` (observed 2026-10-21). Docs: "Always keep api key safe and reset it if you suspect it has been compromised."

## Endpoint Catalogue

| Method | Path | Auth | Purpose | Source |
|---|---|---|---|---|
| POST | `https://manage.kineticpay.my/payment/create` | merchant_key in body | Create an FPX bill; returns auto-submit HTML to send the buyer to the bank | plugins |
| GET | `https://manage.kineticpay.my/payment/status?merchant_key=&invoice=` | merchant_key in query | Requery payment status by invoice | plugins |
| GET | `/api/v2/ping` | Bearer | Connectivity test, returns `{"message":"pong"}` | Quickstart |
| GET | `/api/v2/bank` | Bearer | List FPX banks (`name`, `display`, `code`) | FPX Banks |
| GET | `/api/v2/company` | Bearer | List approved companies on the account | Companies |

Listed in the docs navigation but with no published page (nav link is `/#`, checked 2026-10-21): Bills, Collections, Collections Form, Transactions, Settlements, Points. Their endpoints, fields and behaviour are not documented in official docs. Do not invent paths for them.

Not documented anywhere official (checked 2026-10-21): refunds, cancel/void, capture, recurring/tokenisation, payouts, webhooks with signatures, settlement report API, card or e-wallet payment fields.

## Create Bill: `POST /payment/create`

Send JSON with `Content-Type: application/json` (WooCommerce and GiveWP do this with cURL). The Gravity Forms and Forminator plugins send the same fields form-encoded via `wp_remote_post` with an array body, so form-encoded is also accepted in practice; prefer JSON.

### Request fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `merchant_key` | string | yes | Collection merchant key. Server-side only. |
| `invoice` | string | yes | Your unique reference. Used again for requery. Plugins build it as an uppercase 5-char site host prefix + local id + suffix (`KNWC` WooCommerce, `KNGV` GiveWP), or `1786` + id (Forminator). Max length and charset not documented. |
| `amount` | string/number | yes | Ringgit with 2 decimals, for example `"12.50"`. GiveWP formats with `number_format($amount, 2, '.', '')`; WooCommerce sends the order total. Not sen. Currency is MYR (no currency field). Min/max not documented. |
| `description` | string | yes in all plugins | Free text. Plugins use `"Payment for Order No <id>, Buyer Name <name>, Email <email>, Phone No. <phone>"`. Max length not documented. |
| `bank` | string | yes in all plugins | FPX bank code chosen by the buyer, for example `MB2U0227`. See bank codes below. |
| `callback_success` | URL | yes in all plugins | Where KineticPay sends the buyer/data on success. |
| `callback_error` | URL | yes in all plugins | Where KineticPay sends the buyer/data on failure. Plugins default it to the success URL. |
| `callback_status` | URL | yes in all plugins | Status callback URL. Whether it is called browser-side or server-to-server is not documented. |

Buyer name, email and phone are not separate fields; plugins only put them inside `description`. The WooCommerce plugin refuses checkout if name, phone or email is empty.

### Response

| Shape | Meaning | Handling in plugins |
|---|---|---|
| `{"html": "<form ...><input ...>...<script ...>"}` | Bill created. `html` is a self-submitting form that sends the buyer to the FPX bank flow. | Echo `html` to the browser and exit. Forminator whitelists only `form[action,method,id]`, `input[type,value,name]`, `script[src]`. |
| `{"error": {"field": ["message", ...]}}` | Validation error | Show each `error[field][0]`. |
| `{"error": "message"}` | Error as a string | Show it. |
| Other / `[ "message" ]` | Unknown failure | Plugins show `response[0]` or a generic decline message. |

No bill id or payment URL is returned besides `html`; the `invoice` you sent is the key for requery. Documented HTTP status codes for this endpoint: not documented.

## Callback / Return

All three callback URLs are yours. What KineticPay sends to them is not documented on the docs site. From the WooCommerce plugin (`src/kineticpay.php`, `check_kineticpay_callback`):

- The callback request carries POST field `encoded_transaction_data`, a JSON string.
- The plugin reads `record.transaction_status_code`; `"00"` is treated as success, any other value as failure, missing as "failed without code".

```
encoded_transaction_data = {"record": {"transaction_status_code": "00", ...}, ...}
```

Other keys in that JSON are not documented. There is no signature, checksum or HMAC on this payload in any official source. Anyone can POST `{"record":{"transaction_status_code":"00"}}` to your callback URL. The GiveWP, Gravity Forms and Forminator plugins ignore the callback body and requery `/payment/status` instead. Follow that pattern.

### Verification method

There is no signature to verify. Verification is: on any callback, requery `/payment/status` server-side with your `merchant_key` and the invoice you stored, then settle only from that response.

```
on callback(request):
    invoice = lookup_invoice_from_your_own_url_param(request)   # e.g. /kineticpay/return?ref=<local id>
    payment = db.find_pending(invoice) or return 200
    r = GET https://manage.kineticpay.my/payment/status
            ?merchant_key=<MERCHANT_KEY>&invoice=urlencode(invoice)
    body = json(r)
    if body.error: log redacted, leave pending, return
    if body.code == "00" and amounts_match(body, payment):
        db.transaction: if payment.status == "pending": mark paid, store body.id, fulfil once
    elif body.code is set and != "00":
        mark failed (or keep pending if your policy waits)
    redirect buyer to a result page that reads local status
```

Put your own local reference in the callback URL query (all plugins do, for example `?wc-api=kineticpay&verify=<order_id>` or `?kineticpay=success&paymentid=<id>`). Never trust that parameter for anything but lookup.

## Requery: `GET /payment/status`

Query parameters:

| Param | Required | Notes |
|---|---|---|
| `merchant_key` | yes | Same Collection key used at create. |
| `invoice` | yes | The exact invoice string sent at create. |

Response fields seen in plugin code:

| Field | Type | Notes |
|---|---|---|
| `code` | string | `"00"` = paid. Other values = not paid; the full code list is not documented. |
| `id` | string/number | Transaction id; plugins store it as "Transaction ID". |
| `error` | object | On validation failure, for example `{"error":{"merchant_key":["The selected merchant key is invalid."],"invoice":["The selected invoice is invalid."]}}` (observed 2026-10-21 with HTTP 200). |

The Gravity Forms plugin also references `invoice` and `amount` from the payment data, which suggests the status response may carry them, but this is not confirmed in any source. Log the full response in sandbox and confirm before relying on an amount field. If no amount comes back, match on invoice and trust your stored amount (the amount was set server-side at create).

The merchant key travels in the URL. Redact it from access logs, proxy logs and HTTP client debug output.

## Status Codes

| Value | Where | Meaning |
|---|---|---|
| `"00"` | `code` (requery), `record.transaction_status_code` (WooCommerce callback) | Successful payment |
| any other | same | Not successful. Specific codes not documented. |

## API v2 Endpoints

### `GET /api/v2/ping`

Response: `{"message": "pong"}`.

### `GET /api/v2/bank`

| Property | Type | Description |
|---|---|---|
| `name` | string | Bank name |
| `display` | string | Display name or alias |
| `code` | string | Bank code to send as `bank` |

Response: `{"data": [{"name": "Affin Bank Berhad", "display": "Affin Bank", "code": "ABB0233"}, {"name": "Bank Pertanian Malaysia Berhad (AgroBank)", "display": "AGRONet", "code": "AGRO01"}, ...]}`.

### `GET /api/v2/company`

| Property | Type | Description |
|---|---|---|
| `id` | integer | Company id |
| `name` | string | Company name |
| `ssm_no` | string | SSM number |
| `ssm_doc` | string | URL to SSM document |
| `address` | string | Address |
| `phone` | string | Contact phone |
| `office` | string | Office phone |

Returns only companies approved by the KineticPay team.

### Pagination (v2 list endpoints)

- `perPage`: results per page, default 20, max 40.
- `page`: page number.
- Response `meta.pagination`: `total`, `count`, `per_page`, `current_page`, `total_pages`.

## FPX Bank Codes

Codes hard-coded in all four official plugins (19 banks). Prefer the live list from `GET /api/v2/bank`, which also includes AgroBank `AGRO01`.

| Code | Bank |
|---|---|
| `ABMB0212` | Alliance Bank Malaysia Berhad |
| `ABB0233` | Affin Bank Berhad |
| `AMBB0209` | AmBank (M) Berhad |
| `BCBB0235` | CIMB Bank Berhad |
| `BIMB0340` | Bank Islam Malaysia Berhad |
| `BKRM0602` | Bank Kerjasama Rakyat Malaysia Berhad |
| `BMMB0341` | Bank Muamalat (Malaysia) Berhad |
| `BSN0601` | Bank Simpanan Nasional |
| `CIT0219` | Citibank Berhad |
| `HLB0224` | Hong Leong Bank Berhad |
| `HSBC0223` | HSBC Bank Malaysia Berhad |
| `KFH0346` | Kuwait Finance House |
| `MB2U0227` | Maybank2u / Malayan Banking Berhad |
| `MBB0228` | Maybank2E / Malayan Banking Berhad E (corporate) |
| `OCBC0229` | OCBC Bank (Malaysia) Berhad |
| `PBB0233` | Public Bank Berhad |
| `RHB0218` | RHB Bank Berhad |
| `SCB0216` | Standard Chartered Bank (Malaysia) Berhad |
| `UOB0226` | United Overseas Bank (Malaysia) Berhad |

The WooCommerce plugin also has commented-out codes (`ABB0234`, `BPMBMYKL`, `HBMBMYKL`, `MBBEMYKL`, `MBB0227`, `RJHIMYKL`, `RHBBMYKL`, `UOB0229`); do not use them.

## Rate Limits

- 60 requests per minute, all endpoints counted (Quickstart).
- Headers `X-RateLimit-Limit`, `X-RateLimit-Remaining` on each response.
- HTTP 429 when exceeded. Back off; do not poll aggressively.
- KineticPay says the limit "will be priced to fair usage when we have announced a paid API model".

## Errors (documented HTTP codes)

| Code | Meaning (from docs) |
|---|---|
| 200 | Success. Note: `/payment/status` validation errors were also observed with 200, so always check for an `error` key. |
| 401 | Auth failed: wrong API key, API not enabled, or IP restriction. |
| 403 | Docs label it "Unprocessable Entity": incomplete data in POST/PATCH. |
| 404 | Resource not found. |
| 429 | Rate limit exceeded. |
| 500, 502, 503, 504 | Server error; handle with retry and backoff. |

IP allowlisting: the 401 description mentions "IP address restrictions", but how to configure them is not documented.

## Plugin Patterns To Avoid

These appear in the official plugin source. Do not copy them:

- WooCommerce marks the order paid from the unsigned `encoded_transaction_data` callback body (forgeable).
- WooCommerce requery code uses `$order_id` without the invoice prefix in its cron, so it queries the wrong invoice.
- A 3-second WP cron requery of the latest order; wasteful against a 60/min limit.
- Gravity Forms compares against an undefined `$data` variable after requery.
- The Gravity Forms `success_action` uses an undefined `$order_id` when building the status URL.
- `description` in WooCommerce puts phone in the Email slot.

## Production Checklist

1. Collection sandbox toggle off for live.
2. `KINETICPAY_MERCHANT_KEY` in server env; API key only if v2 is used.
3. Callback routes requery and never trust the body.
4. Invoice unique, stored before redirect, unique index in DB.
5. Requery job with backoff for pending payments.
6. Logs redact `merchant_key` from URLs.
