# Payhalal API Reference

Use this reference for factual Payhalal API guidance. The official API docs are the Souqa Fintech GitBook "Payhalal Integration Guide" (Version 1.5). Where the docs are thin or contradict themselves, the official plugins published by Souqa Fintech on GitHub are used as a second source and the conflict is stated. Checked 2026-10-21.

## Source URLs

- Docs index for agents: `https://souqa-fintech.gitbook.io/api-docs/llms.txt`
- Full docs text: `https://souqa-fintech.gitbook.io/api-docs/llms-full.txt`
- `https://souqa-fintech.gitbook.io/api-docs/payment-flow-overview`
- `https://souqa-fintech.gitbook.io/api-docs/uri-reference`
- `https://souqa-fintech.gitbook.io/api-docs/payments/payment-request-parameters`
- `https://souqa-fintech.gitbook.io/api-docs/payments/generate-hash-for-payment-request`
- `https://souqa-fintech.gitbook.io/api-docs/payments/example-payment-request-via-php-for-v1-and-v2`
- `https://souqa-fintech.gitbook.io/api-docs/payments/example-payment-request-via-php-for-v1-and-v2/sample-json-payment-link-response-for-v2`
- `https://souqa-fintech.gitbook.io/api-docs/payments/payment-response-callback`
- `https://souqa-fintech.gitbook.io/api-docs/payments/payment-response-callback/callback-url-to-your-server`
- `https://souqa-fintech.gitbook.io/api-docs/payments/payment-request-error-response`
- `https://souqa-fintech.gitbook.io/api-docs/transaction-reconciliation/single-transaction-reconcile`
- `https://souqa-fintech.gitbook.io/api-docs/transaction-reconciliation/batch-transaction-reconcile`
- `https://souqa-fintech.gitbook.io/api-docs/transaction-reconciliation/transaction-list`
- `https://souqa-fintech.gitbook.io/api-docs/transaction-reconciliation/formula-of-hash-transaction-reconciliation`
- Official WooCommerce plugin (v1.0.5, 2026-05-06): `https://github.com/SouqaFintech/payhalal-woocommerce-plugin`
- Official OpenCart 3.x extension: `https://github.com/SouqaFintech/opencart3.x-payhalal-extension`

## Endpoint Catalogue

Every operation in the official GitBook docs (17 pages, all read), plus the extra endpoints that appear only in official plugin source (marked "plugin-only").

| # | Method | URL | Purpose | Source |
| --- | --- | --- | --- | --- |
| 1 | GET, POST | `https://api.payhalal.my/pay` | V1 production: create payment and redirect the customer to the hosted payment page. | URI Reference, Payment Request Parameters |
| 2 | GET, POST | `https://api-testing.payhalal.my/pay` | V1 UAT: same as 1 with testing keys. | URI Reference |
| 3 | GET, POST | `https://api.payhalal.my/v2/pay.php` | V2: create payment, returns JSON with `payment_link`. Live or testing chosen by the key. Selected merchants only. | URI Reference, Payment Request Parameters |
| 4 | GET, POST | `https://api.payhalal.my/qrCreate` | V1 production QR Code API: returns QR image URL and `transaction_id`. | URI Reference, Payment Request Parameters |
| 5 | GET, POST | `https://api-testing.payhalal.my/qrCreate` | V1 UAT QR Code API. | URI Reference |
| 6 | POST (to you) | your Callback URL | Server callback with payment result, set in Developer Tools. | Callback URL To Your Server |
| 7 | POST (browser) | your Return / Success / Cancel URL | Browser redirect after payment (same fields as callback per the WooCommerce plugin). | WooCommerce README and gateway class |
| 8 | GET | `https://api-merchant.payhalal.my/single_transaction_reconcile.php` | Production single transaction status. | Single Transaction Reconcile |
| 9 | GET | `https://api-merchant-uat.payhalal.my/single_transaction_reconcile.php` | UAT single transaction status. | Single Transaction Reconcile |
| 10 | GET | `https://api-merchant.payhalal.my/batch_transaction_reconcile.php` | Production batch status, comma separated ids. | Batch Transaction Reconcile |
| 11 | GET | `https://api-merchant-uat.payhalal.my/batch_transaction_reconcile.php` | UAT batch status. | Batch Transaction Reconcile |
| 12 | GET | `https://api-merchant.payhalal.my/transaction_list.php` | Production transaction list by date range, 100 per page. | Transaction List |
| 13 | GET | `https://api-merchant-uat.payhalal.my/transaction_list.php` | UAT transaction list. | Transaction List |
| 14 | POST | `https://api-merchant.payhalal.my/seamless/Nomu/index.php` | Plugin-only: recurring payment request (see Recurring). | WooCommerce gateway class |
| 15 | GET (JSON body) | `https://payhalal.my/nomupay/check-enabled.php` | Plugin-only: checks if recurring is enabled for the MID. Body `{"app_key","app_secret"}`, success when `code == "200"`. | WooCommerce gateway class `check_channel()` |
| 16 | GET | `https://payhalal.my/cart_payment.php` | Plugin-only, legacy: PrestaShop module redirect with `amount, currency, product_description, order_id, customer_name, customer_email, customer_phone, hash, key, live`. Do not use for new builds; use 1 or 2. | PrestaShop `validation.php` |

Not in the docs or any official plugin (checked 2026-10-21): refund, cancel, void, capture, payout/disbursement, settlement report, tokenisation, webhook management.

Notes on plugin-only endpoints:

- 15 sends the App Secret to `payhalal.my` in a request body. It is undocumented; only call it if Payhalal tells you to.
- 16 uses PHP `number_format($total, 2)` without a separator argument, which produces `1,200.00` for amounts of RM1,000 and up, and hashes that string. This is the likely origin of the "amount with thousand separator" note on the callback. The current WooCommerce and OpenCart 3.x code sends no separator; follow them.

## Integration Surfaces

- V1 hosted payment page: merchant browser POSTs (or GETs) the signed fields to the pay URL and the customer is redirected to the Payhalal payment page, where they pick a channel (for example FPX) and are sent to that processor.
- V2 payment link: same fields, but the response is JSON containing `payment_link`. Docs: "the V2 API is in testing phase and is only opened to selected merchants." Existing merchants can keep using V1.
- QR Code API: same signed fields, returns a QR image URL and transaction id.
- Official plugins: WooCommerce, OpenCart 1.x and 3.x, PrestaShop.
- Recurring: only in the WooCommerce plugin, not in the API docs (see Recurring below).

## Environments

| Purpose | Production | UAT (sandbox) |
| --- | --- | --- |
| V1 pay | `https://api.payhalal.my/pay` | `https://api-testing.payhalal.my/pay` |
| V2 pay (JSON) | `https://api.payhalal.my/v2/pay.php` | same URL, use the testing key |
| QR create | `https://api.payhalal.my/qrCreate` | `https://api-testing.payhalal.my/qrCreate` |
| Reconcile / list | `https://api-merchant.payhalal.my/` | `https://api-merchant-uat.payhalal.my/` |

- Pay URLs accept both `GET` and `POST`.
- V2: "no need to specify url either production or testing ... You can just used live api_key or testing api_key." The V2 response `payment_link` then points to `api.payhalal.my/pay` or `api-testing.payhalal.my/pay`.
- Docs examples show keys prefixed `live-...` and `testing-...`.

## Auth

There is no API key header or bearer token in the documented API. Each request is authenticated by `app_id` plus a SHA256 `hash` that includes the App Secret. Reconciliation APIs use `merchant_id` (your merchant email) plus an MD5 hash.

## Payment Request Fields (V1, V2 and QR)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `app_id` | String | Yes | Application Id (the App ID/App Key from Developer Tools). |
| `currency` | String | Yes | ISO currency code, for example `MYR`. |
| `amount` | Decimal | Yes | "Amount without thousand separators". Use a two-decimal string, for example `99.99`. |
| `product_description` | String | Yes | Description of the product. Empty gives `product_description is empty.` |
| `order_id` | String | Yes | Your order id. |
| `customer_name` | String | No | Customer first and last name. |
| `customer_email` | String | No | Customer email. |
| `customer_phone` | String | No | Customer phone. |
| `hash` | String | Yes | SHA256 hex, see below. |
| `language` | String | No | Appears in the PHP sample as `en`; not in the parameter table and not part of the hash. |

Amount unit is ringgit with decimals (not sen). Minimum amount error: `{ "error": "Minimum Amount is RM5.00" }`.

The official OpenCart extension formats the request amount with `number_format(total, 2, ".", "")`. The WooCommerce 1.0.3 changelog says it forces `MYR` for callbacks "to prevent hash mismatch issues on multi-currency WooCommerce stores"; send `MYR` unless Payhalal confirms other currencies for your account.

## Request Hash

From "Generate Hash For Payment Request":

```text
sha256(APP_SECRET.AMOUNT.CURRENCY.PRODUCT_DESCRIPTION.ORDER_ID.CUSTOMER_NAME.CUSTOMER_EMAIL.CUSTOMER_PHONE)
```

- `.` is PHP string concatenation: there is no separator in the string.
- Output is lowercase hex (PHP `hash('sha256', ...)`).
- Omitted optional fields are concatenated as empty strings, and must also be sent empty (or omitted) consistently.
- `app_id`, `hash` and `language` are not in the string.

Pseudo-code:

```js
import { createHash } from "node:crypto";

function payhalalRequestHash(secret, f) {
  const s = secret + f.amount + f.currency + f.product_description + f.order_id +
            (f.customer_name ?? "") + (f.customer_email ?? "") + (f.customer_phone ?? "");
  return createHash("sha256").update(s, "utf8").digest("hex");
}
```

V1 send: render an HTML form with `method="post"` and `action` set to the environment V1 URL, one hidden input per field, auto-submitted. The official WooCommerce plugin does exactly this.

## V2 Response (JSON)

```json
{
  "merchant_id": "merchant@example.com",
  "amount": "1.00",
  "currency": "MYR",
  "amount_convert_myr": null,
  "order_id": "test123",
  "payment_link": "https://api.payhalal.my/pay?app_id=...&amount=1.00&currency=MYR&product_description=...&order_id=...&customer_name=&customer_email=&customer_phone=&hash=...",
  "payment_hash": "<sha256 hex>",
  "date_created": "06/12/2022 01:05:38"
}
```

Redirect the customer to `payment_link`.

| Field | Type (from sample) | Notes |
| --- | --- | --- |
| `merchant_id` | String | Merchant login email. |
| `amount` | String, 2 decimals | Echo of request amount. |
| `currency` | String | Echo, for example `MYR`. |
| `amount_convert_myr` | Null or String | Always `null` in samples; meaning not documented (likely the MYR value for non-MYR currency). |
| `order_id` | String | Echo of your order id. |
| `payment_link` | URL | V1 pay URL with all signed fields as query string. Host `api.payhalal.my` for a live key, `api-testing.payhalal.my` for a testing key. |
| `payment_hash` | String, 64 hex | The SHA256 request hash. |
| `date_created` | String `DD/MM/YYYY HH:MM:SS` | Timezone not documented (assume Malaysia time, UTC+8). |

The docs say to make sure you use the correct `app_key` for Live and Testing; the V2 URL is the same for both.

## QR Code Response

```json
{
  "qr_code": { "image_url": "https://api.payhalal.my/qr?id=...&app=...&token=...", "content": "<token>" },
  "transaction_id": 100,
  "amount": "10.00",
  "currency": "MYR",
  "expire": 1560757541
}
```

| Field | Type (from sample) | Notes |
| --- | --- | --- |
| `qr_code.image_url` | URL | QR image to display, `https://api.payhalal.my/qr?id=...&app=...&token=...`. |
| `qr_code.content` | String | The QR token (same as `token` in `image_url`). |
| `transaction_id` | Number | Payhalal transaction id. Store it: the reconcile APIs look up by it. |
| `amount` | String, 2 decimals | Echo. |
| `currency` | String | Echo. |
| `expire` | Number | Undefined in docs; looks like a Unix timestamp (seconds). |

The QR request uses the same fields and hash as V1 (field table order differs, hash order does not). `expire` looks like a Unix timestamp; the docs do not define it. How a QR payment is confirmed (callback or polling) is not documented; assume the same Callback URL and verify with reconciliation.

## Request Errors

JSON `{ "error": "..." }`. No numeric error codes are documented; match on the message.

| Field | Cause | Exact message |
| --- | --- | --- |
| `amount` | not passed | `amount is required` |
| `amount` | below minimum | `Minimum Amount is RM5.00` |
| `currency` | not passed | `currency is required` |
| `product_description` | not passed | `product_description is empty.` |
| `order_id` | not passed | `order id is empty.` |
| `hash` | not passed | `hash is required.` |
| `hash` | does not match | `SHA256 Signature invalid.` |

No error is documented for a missing or wrong `app_id`.

Known bugs in the docs PHP samples (do not copy literally): the request sample builds `$values` but loops over `$ph`; the callback sample overwrites `$secret` with a hardcoded value, checks `$ph_app_secret == true`, and uses `==` instead of a constant-time compare. The sample also sets `amount` as a PHP float `99.99`; send a fixed two-decimal string instead.

## Return, Success, Cancel And Callback URLs

- Configured per app in Merchant Dashboard > Developer Tools (> Edit App). Not sent in the request.
- Docs: "The Callback URL is optional. If no Callback URL is set, PayHalal won't send data to the merchants side."
- The official WooCommerce README configures four URLs: Return URL, Success URL and Callback URL all pointing to the same handler, and Cancel URL to the cart.
- Both the browser return and the server callback arrive as form `POST` (`$_POST`) with the callback fields below. The WooCommerce plugin uses one handler for both and redirects the browser afterward.
- Docs sample comment: "if a callback URL is defined, system must return "OK"". Respond `200` with body `OK` on the server callback after successful processing. Retry behaviour is not documented.

## Callback Fields

| Field | Example | Notes |
| --- | --- | --- |
| `app_id` | `live-XXX` | Check it equals your App ID for this environment. |
| `amount` | `1200.99` | Documented "Amount with thousand separator" (so `1,200.99` is possible). |
| `currency` | `MYR` | |
| `product_description` | | Echo of request. |
| `order_id` | `123` | Your order id. |
| `customer_name`, `customer_email`, `customer_phone` | | Echo of request. |
| `transaction_id` | `FPX123` | Payhalal transaction id; store it. |
| `status` | `SUCCESS` | See statuses. |
| `hash` | | SHA256, see below. |
| `channel` | `CC` | Channel type, for example `FPX` or `CC`. |
| `source` | `RECURRING_NOMU` | Not in docs; read by the WooCommerce plugin for recurring callbacks. |

## Callback Hash (conflict: confirm in UAT)

Docs ("Callback URL To Your Server"):

```text
sha256(APP_SECRET.AMOUNT.CURRENCY.PRODUCT_DESCRIPTION.ORDERID.CUSTOMER_NAME.CUSTOMER_EMAIL.CUSTOMER_PHONE.TRANSACTION_ID.STATUS)
```

Official plugins (WooCommerce v1.0.5 `generate_callback_hash`, OpenCart 3.x `status()` and `callback()`):

```text
sha256(APP_SECRET.AMOUNT.CURRENCY.PRODUCT_DESCRIPTION.ORDER_ID.CUSTOMER_NAME.CUSTOMER_EMAIL.CUSTOMER_PHONE.STATUS)
```

The 2018 payhalal/woocommerce-plugin also omitted `customer_phone`; that plugin is superseded and should not be followed.

Recommended verification, using the posted values exactly as received (not your stored values, because the posted amount may contain a thousand separator):

```js
import { createHash, timingSafeEqual } from "node:crypto";

function verifyPayhalalCallback(secret, p) {
  const base = secret + p.amount + p.currency + p.product_description + p.order_id +
               (p.customer_name ?? "") + (p.customer_email ?? "") + (p.customer_phone ?? "");
  const variants = {
    plugin: base + p.status,                          // what official plugins verify
    docs:   base + (p.transaction_id ?? "") + p.status // what GitBook documents
  };
  const got = Buffer.from(String(p.hash ?? "").toLowerCase(), "utf8");
  for (const [name, s] of Object.entries(variants)) {
    const exp = Buffer.from(createHash("sha256").update(s, "utf8").digest("hex"), "utf8");
    if (exp.length === got.length && timingSafeEqual(exp, got)) return name;
  }
  return null; // reject
}
```

Run one UAT payment, log which variant matched, then pin your code to that one variant. Both variants require the App Secret, so neither is forgeable without it, but pinning keeps the check strict.

After the hash passes, still check:

1. `order_id` exists locally and is pending.
2. `app_id` equals the configured App ID for the environment.
3. `amount` with commas removed equals the stored amount (compare as fixed two-decimal strings or integer sen).
4. `currency` equals the stored currency.
5. `status == "SUCCESS"` before marking paid.
6. Settle once: key on `order_id` and store `transaction_id`.

## Statuses

- Callback docs: `SUCCESS`, `FAILED`, `PENDING`.
- Official plugins branch on `SUCCESS`, `FAIL`, `PENDING`, `TIMEOUT`.
- Reconcile API examples return `FAIL`.
- Rule: only `SUCCESS` is paid. `PENDING` stays pending and is reconciled later. `FAIL`, `FAILED`, `TIMEOUT` and anything unknown are not paid.

## Enums And Codes

No bank code list, channel code list or numeric error code list is published. These are every value that appears in the docs and official plugins.

| Enum | Values seen | Where |
| --- | --- | --- |
| Payment `status` | `SUCCESS`, `FAILED`, `PENDING` (docs); `FAIL`, `TIMEOUT` (plugins, reconcile samples) | Callback table, reconcile samples, WooCommerce and OpenCart 3.x |
| Callback `channel` | `FPX`, `CC` (docs say "i.e. FPX or CC"); full list not documented | Callback table |
| Reconcile `source` | `CC_CIMB`, `FPX`, `ATOME`; full list not documented | Reconcile samples |
| Callback `source` | `RECURRING_NOMU` (plugin-only) | WooCommerce gateway class |
| Reconcile `mode` | `LIVE`, `TEST` | Reconcile samples |
| App ID prefix | `live-...`, `testing-...` | PHP sample, V2 GET samples |
| `payment_cycle` | `MONTHLY`, `WEEKLY`, `DAILY`, `TEST` (plugin-only) | WooCommerce README / gateway class |
| `pro_rate` | `1`, `0` (plugin-only) | WooCommerce gateway class |
| `language` | `en` (sample only) | PHP sample |
| `currency` | `MYR` (only value in any sample) | All samples |

FPX bank selection happens on the Payhalal hosted page; there is no documented field to preselect a bank or channel.

## Reconciliation APIs

Hash for all three ("Formula of Hash"):

```text
Production: md5(merchant_id . app_key . app_secret)
UAT:        md5(merchant_id . app_key_testing . app_secret_testing)
```

`merchant_id` is the merchant login email. The hash does not include the transaction id or dates, so it is a static credential: keep it server-side. Docs label it "String, 32 chars" (MD5 hex).

Parameters are sent with `GET` (docs table titles say "Path Parameters" for reconcile and "Query Parameters" for list; send them as query string parameters).

### Single Transaction Reconcile

`GET https://api-merchant.payhalal.my/single_transaction_reconcile.php` (UAT: `https://api-merchant-uat.payhalal.my/single_transaction_reconcile.php`, only for UAT-mode transactions)

| Field | Type | Required |
| --- | --- | --- |
| `transaction_id` | Number, 20 chars | Yes |
| `merchant_id` | Varchar, 100 chars (email) | Yes |
| `hash` | String, 32 chars | Yes |

### Batch Transaction Reconcile

`GET https://api-merchant.payhalal.my/batch_transaction_reconcile.php` (UAT: `api-merchant-uat`). Same fields; `transaction_id` is comma separated, for example `1000000,20000,30000`. The transaction list error table mentions "Number of transaction cannot be more than 10 only."

### Transaction List

`GET https://api-merchant.payhalal.my/transaction_list.php` (UAT: `api-merchant-uat`)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `merchant_id` | Varchar, 100 | Yes | Merchant email. |
| `sdate` | Date `YYYY-MM-DD` | Yes | Start date. |
| `edate` | Date `YYYY-MM-DD` | Yes | End date; range cannot exceed 3 months. |
| `page_no` | Int | No | Needed when more than 100 transactions; returns the next 100. |
| `hash` | String, 32 | Yes | MD5 formula above. |

### Reconcile Response (array)

```json
[{
  "source": "CC_CIMB", "amount": "12.0000", "currency": "MYR", "status": "FAIL",
  "transaction_id": "101xxxxx", "date_created": "2022-12-02 08:43:36", "mode": "LIVE",
  "customer_name": "...", "customer_email": "...", "customer_phone": "...",
  "customer_order_id": "", "product_description": "test"
}]
```

| Field | Type (from sample) | Notes |
| --- | --- | --- |
| `source` | String | Channel and acquirer, for example `CC_CIMB`, `FPX`, `ATOME`. |
| `amount` | String, 4 decimals | For example `12.0000`. |
| `currency` | String | `MYR`. |
| `status` | String | See Statuses. |
| `transaction_id` | String | Payhalal transaction id. |
| `date_created` | String `YYYY-MM-DD HH:MM:SS` | Timezone not documented. |
| `mode` | String | `LIVE` or `TEST`. |
| `customer_name`, `customer_email`, `customer_phone` | String | Echo of request. |
| `customer_order_id` | String | Empty or `"0"` in every sample. |
| `product_description` | String | Echo of request. |

The response is always a JSON array, also for single reconcile. Unknown ids: behaviour not documented (expect an empty array or an `error` object; handle both).

- `amount` has four decimals here (`12.0000`); normalise before comparing.
- `mode` is `LIVE` or `TEST`.
- `source` examples: `CC_CIMB`, `FPX`, `ATOME`.
- `customer_order_id` is empty or `"0"` in all examples, so do not rely on it to find your order; use the stored `transaction_id`, or match by date, amount and customer in the list API.

Errors: `Transaction ID is empty`, `Merchant ID is empty`, `Hash is empty`, `Start date is empty`, `End date is empty`, `Both date cannot be more than 3 months.`

## Recurring (plugin-only, not in API docs)

The official WooCommerce plugin, for products with a `payment-cycle` attribute (`MONTHLY`, `WEEKLY`, `DAILY`, `TEST`), posts the normal signed fields plus `payment_cycle` and `pro_rate` (`1`/`0`) to `https://api-merchant.payhalal.my/seamless/Nomu/index.php`, with `product_description` set to `WooCommerce|<product_id>`. Recurring callbacks carry `source = RECURRING_NOMU` and `product_id`. The MID must be enabled for recurring by the Payhalal onboarding team. Treat this as undocumented: get written confirmation from Payhalal before building on it.

Recurring request fields (as sent by the WooCommerce plugin, form POST to `seamless/Nomu/index.php`):

| Field | Required | Notes |
| --- | --- | --- |
| `app_id`, `amount`, `currency`, `order_id`, `customer_name`, `customer_email`, `customer_phone` | as V1 | Same meaning as V1. |
| `product_description` | Yes | Plugin sends `WooCommerce|<product_id>`. |
| `payment_cycle` | Yes | `MONTHLY`, `WEEKLY`, `DAILY`, `TEST`. |
| `pro_rate` | Yes | `1` or `0`. |
| `hash` | Yes | Same SHA256 request hash as V1; `payment_cycle` and `pro_rate` are not in the string. |

Recurring callbacks (each renewal): `status = SUCCESS`, `source = RECURRING_NOMU`, `product_id`, `transaction_id`, `channel`, customer fields. The plugin creates a new order per renewal after verifying the same callback hash. Before sending, the plugin calls `check-enabled.php` (Endpoint Catalogue row 15) and shows `Recurring not enabled for your MID.` if it does not return `code` `200`.

## Not Documented (checked 2026-10-21)

- Refund API (the FAQ says refunds are handled by the store; no refund endpoint exists in the docs).
- Callback retry policy, timeout and source IP addresses.
- Rate limits.
- Sandbox test cards or test FPX banks.
- Webhook signature headers. The developers landing page shows a "pseudo-example" with `POST /pay_v2`, `Authorization: Bearer`, JSON body and a `payhalal-signature` header; none of this appears in the GitBook docs or official plugins. Do not use it.

## Common Errors And Fixes

- `SHA256 Signature invalid.`: wrong secret for the environment, amount formatted differently from what was hashed (for example `99.9` vs `99.90`), optional customer field sent but not hashed (or the reverse), or trailing spaces.
- Callback hash mismatch: try the plugin variant (no `transaction_id`); hash the posted amount as received, not a reformatted one.
- No callback received: Callback URL not set in Developer Tools (it is optional and off by default), or set only on the live app and not the UAT app.
- Live key on `api-testing` URL: use matching key and URL.
