# LeanX API Reference

Distilled from the full public LeanX developer docs at `https://docs.leanx.io` (all 43 pages, via `https://docs.leanx.io/llms-full.txt`) plus the official WooCommerce and OpenCart plugin source linked from those docs. Checked 2026-10-21. Page URLs are in `source-map.md`.

LeanX is also written "Lean.x"; the operating company is Leanis Solutions Sdn Bhd (payout recipients see this sender name).

## Environments

| Item | Sandbox | Production |
| --- | --- | --- |
| API base URL | `https://api.leanx.dev` | `https://api.leanx.io` |
| Merchant portal | `https://portal.leanx.dev` | `https://portal.leanx.io` |
| Hosted payment page host (seen in responses) | `https://payment.leanx.dev` | Not documented |
| Onboarding / registration | `https://onboard.leanx.io` | `https://onboard.leanx.io` |

- Every path below is shown without host. Docs examples use the sandbox host; swap the host for production.
- Sandbox requests are "processed as a mock transaction and contained within the sandbox environment".
- HTTPS is mandatory; plain HTTP requests fail. Unauthenticated requests are not processed.
- Legacy hosts used by the official plugins (not in the API docs): sandbox `https://stag-api.leanpay.my/api/v1/`, production `https://api.leanx.io/api/v1/`. Use the hosts above for new work.

### Public sandbox credential (sandbox only)

Published on the Credential page for general connectivity tests. It accepts generic requests and returns generic responses. For transaction-specific responses, use the Sandbox API key from your own Onboarding Dashboard.

| Type | Value |
| --- | --- |
| Base URL | `https://api.leanx.dev` |
| Merchant ID | `LP-44D9FDC8-MM` |
| UUID | `f11bb37b-dc4a-45cf-9607-d68f31e96e7c` |
| Hash Key | `6c7b9ddc3a770db6f760f6550f8ef2164447f2b4c3efd04d45b757ae6e4b09aa3bc006c4d45b6d66f5371af77209521a366ae9fef3ca9fac8d95219898babcf4` |
| Auth Token | `LP-44D9FDC8-MM\|f11bb37b-dc4a-45cf-9607d68f31e96e7c\|c8fac9c6a98f8e739854e26e205f635727d173c90bc9f842a9b0b1a75b0a708785242c3ed6e89b16789991e290367c8e334ab8bc01b8b5e5434636853fb21363` |

The published Auth Token's middle segment is missing one hyphen compared with the UUID (`9607d68f` vs `9607-d68f`). Copy it exactly as published; if it fails, get your own sandbox key.

## Credentials

| Credential | Format | Used for |
| --- | --- | --- |
| Merchant ID | `LP-XXXXXXXX-MM` | Identity; first segment of the Auth Token |
| UUID | UUID v4 string | API key identifier; part of the Auth Token and X-Signature message; echoed in the `token` field of many responses |
| Hash Key | 128 hex characters | Secret for X-Signature HMAC and for decoding/verifying callback JWTs |
| Auth Token | `<MERCHANT_ID>\|<UUID>\|<128 hex>` | `auth-token` request header on every call |

Production values come from Merchant Portal > API page (create/add an API key, enable it, then read the values). The token is static; you do not need to call the validate endpoint before each request.

## Authentication

Every request:

```http
auth-token: <AUTH_TOKEN>
Content-Type: application/json
```

Header name is `auth-token` (lowercase, hyphen). The OpenCart plugin sends exactly `auth-token:<token>`.

## X-Signature request signing (optional, per API key)

Enable in Portal: API > API Key > Edit API Key > X-Signature. When enabled, every API request must carry the signature headers or it is rejected (errors 4001, 4002, 4004, 4006, 4008, 4013, 4015).

String to sign, pipe-joined in this exact order:

```text
HTTP_METHOD|UUID|URL_PATH|TIME_STAMP|AUTH_TOKEN|NONCE
```

| Element | Value |
| --- | --- |
| `HTTP_METHOD` | `GET`, `POST`, `PUT`, `DELETE`, `PATCH` (uppercase, as sent) |
| `UUID` | API key UUID from the portal |
| `URL_PATH` | Endpoint path, for example `/api/v1/merchant/create-bill-page` |
| `TIME_STAMP` | Current Unix time in seconds |
| `AUTH_TOKEN` | Full Auth Token string |
| `NONCE` | Unique random string, typically UUID4 |

Signature: `hex(HMAC_SHA256(key = HASH_KEY, message))`, lowercase hex (PHP `hash_hmac('sha256', ...)`, CryptoJS `.toString(enc.Hex)`).

Headers added to the request:

```http
x-signature: <hex hmac>
x-timestamp: <TIME_STAMP>
x-nonce: <NONCE>
```

Each signature is valid for 5 minutes from the timestamp and tied to that API key.

Pseudo-code:

```text
ts    = floor(now_unix_seconds())
nonce = uuid4()
path  = "/api/v1/merchant/create-bill-page"      # path only
msg   = method + "|" + uuid + "|" + path + "|" + ts + "|" + auth_token + "|" + nonce
sig   = hmac_sha256_hex(key=hash_key_utf8, data=msg_utf8)
headers = { "auth-token": auth_token, "x-signature": sig, "x-timestamp": ts, "x-nonce": nonce }
```

Notes:

- The official Postman script uses `pm.request.url.getPath()`, which returns the path without the query string. The docs do not say whether a query string (for example `?invoice_no=...`) is part of `URL_PATH`. Test both in sandbox for endpoints that take query parameters; start with path only.
- The Hash Key is used as a UTF-8 string key (the PHP and Postman samples pass it directly).
- Keep the server clock NTP-synced; error 4013 means the timestamp expired.

## Response envelope

```json
{
  "response_code": 2000,
  "description": "SUCCESS",
  "app_version": "/api/v1",
  "talk_to_server_before": "2023-11-28T17:06:33.905941",
  "data": { },
  "breakdown_errors": "",
  "token": ""
}
```

- `response_code` 2000 = success. Callbacks use 2100. A documented failure example: `response_code` 1911, `description` `FAILED`, `breakdown_errors` `ACTIVE_COLLECTION_CANT_DELETE`.
- `talk_to_server_before` is the time the API responded.
- `token` often carries the API key UUID (same for collection and payout responses).
- HTTP 200 indicates the request succeeded; failures carry an error code in the body. Check `response_code`, not only HTTP status.

## Endpoint catalogue

All paths prefixed by the base URL. Methods are as documented; see "Documented inconsistencies".

| # | Method | Path | Purpose |
| --- | --- | --- | --- |
| 1 | POST | `/api/v1/merchant/validate` | Validate Auth Token, return merchant id/name |
| 2 | POST | `/api/v1/merchant/overall-balance` | Transaction pool and payout prefund balance |
| 3 | POST | `/api/v1/merchant/list-payment-services` | List payment services (banks, wallets, cards, QR) per payment type |
| 4 | POST | `/api/v1/merchant/create-payment-collection` | Create a Collection |
| 5 | POST | `/api/v1/merchant/collection-list?skip=&limit=` | List Collections |
| 6 | POST | `/api/v1/merchant/payment-collection` | Get one Collection by numeric id |
| 7 | PUT | `/api/v1/merchant/update-payment-collection` | Update Collection title/description |
| 8 | PUT | `/api/v1/merchant/activate-payment-collection` | Re-activate a deactivated Collection |
| 9 | PUT | `/api/v1/merchant/deactivate-payment-collection` | Deactivate a Collection |
| 10 | POST | `/api/v1/merchant/delete-payment-portal` | Delete a Collection (second documented variant: PUT `/api/v1/merchant/delete-payment-collection`) |
| 11 | POST | `/api/v1/merchant/create-bill-page?invoice_no=` | Create bill, get LeanX hosted payment page URL |
| 12 | POST | `/api/v1/merchant/create-bill-silent` | Create bill straight to a chosen payment service (own page, DuitNow QR, card library) |
| 13 | POST | `/api/v1/merchant/bill-list?skip=&limit=` | List bills across collections |
| 14 | POST | `/api/v1/merchant/bill-id` | Get one bill by numeric id |
| 15 | PUT | `/api/v1/merchant/update-bill` | Update payer name/email/phone on a bill |
| 16 | PUT | `/api/v1/merchant/delete-bill` | Delete a bill |
| 17 | POST (GET on one page) | `/api/v1/merchant/manual-checking-transaction?invoice_no=` | Transaction status requery |
| 18 | GET | `/api/v1/merchant/transaction-list?skip=&limit=&start_date=&end_date=&invoice_status=` | Transaction report (max 100 rows per call) |
| 19 | POST | `/api/v1/merchant/list-payout-services` | List payout banks/wallets |
| 20 | POST | `/api/v1/merchant/create-payout-invoice` | Create a payout |
| 21 | POST | `/api/v1/merchant/get-payout-transaction-by-id?_id=` | Payout status by payout invoice no or `external_invoice_ref` |
| 22 | POST | `/api/v1/merchant/payout-transaction-list?skip=&limit=` | Payout transaction list (per Auth Token) |
| 23 | POST | `/api/v1/merchant/check-verification-bank` | Validate recipient bank account (returns account holder name) |
| 24 | POST | `/api/v1/merchant/list-verification-channel` | Bank verification channel list (Deprecated) |
| 25 | (callback) | merchant `callback_url` | Payment callback (JWT) |
| 26 | (callback) | merchant `client_callback_url` | Payout callback (JWT) |

Endpoints used only by the official plugins, not in the API docs (legacy, do not build new work on them): `POST /api/v1/public-merchant/validate`, `POST /api/v1/public-merchant/validate-collection-id` (body `{"uuid": "<collection uuid>"}`), `POST /api/v1/public-merchant/public/collection-payment-portal?invoice_no=` (plugin equivalent of create-bill-page), `POST /api/v1/public-merchant/public/manual-checking-transaction?invoice_no=`, `POST /api/v1/jwt/decode` (body `{"signed": "<jwt>", "secret_key": "<hash key>"}`, server-side decode of callbacks).

## Collections

A Collection groups Bills; every bill needs one. Collections are unique per merchant, can be created in the portal or by API, and one collection can be used indefinitely (no bill limit). Collection UUID format examples: `CL-C0D7F54A90-LNP`, `CL-AUTH-ADA0B0240E-LNP`, `DP-1F6762F9E4-LX`.

### Create Collection: `POST /api/v1/merchant/create-payment-collection`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `title` | String | Yes | Collection title |
| `description` | String | Yes | Collection description |
| `enable_billing_address` | Boolean | Yes | Let payer enter billing address on payment page |

Response `data.collection_uuid`.

### Collection list: `POST /api/v1/merchant/collection-list?skip=0&limit=10`

Query `skip` (start row, required), `limit` (rows per page, required).

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `start_date` | String | Yes | `DD-MM-YYYY` |
| `end_date` | String | Yes | `DD-MM-YYYY` |
| `record_status` | Number | Yes | 1 = ACTIVE |
| `search.search_enable` | Boolean | Yes | Enable search |
| `search.search_key` | String | If search enabled | Search value |
| `search.search_column` | String | If search enabled | Collection property, for example `title` |
| `sort.sort_type` | String | No | `asc` or `desc` |
| `sort.parameter_name` | String | No | Property, for example `created_at` |

Response `data.list`: `draw`, `record_total`, `record_filtered`, `data[]`, `next_page_start`, `next_page_length`, `previous_page_start`, `previous_page_length`. Item fields: `id`, `uuid`, `title`, `description`, `record_status`, `enable_billing_address`, `min_amount`, `fixed_amount`, `created_at`, `updated_at`.

### Get / update / activate / deactivate / delete

| Operation | Body | Notes |
| --- | --- | --- |
| Get (`payment-collection`) | `{"_id": <collection id number>}` | Returns the collection object |
| Update (`update-payment-collection`, PUT) | `{"id": 99, "title": "...", "description": "..."}` | `id` required; `title`, `description` optional |
| Activate (PUT) | `{"_id": 99}` | Only for a deactivated collection; result `record_status` 1 |
| Deactivate (PUT) | `{"_id": 99}` | Result `record_status` 2 |
| Delete | `{"_id": 99}` | Deleted collections become invisible and unusable; result `record_status` 4. An active collection cannot be deleted (`ACTIVE_COLLECTION_CANT_DELETE`), so deactivate first |

## Payment services

### `POST /api/v1/merchant/list-payment-services`

| Field | Type | Req | Values |
| --- | --- | --- | --- |
| `payment_type` | String | Yes | `WEB_PAYMENT` (FPX), `CI_VISA_MASTERCARD` (card), `GLOBAL_CARD_PAYMENT` (card), `DIGITAL_PAYMENT` (e-wallet), `BUY_NOW_PAY_LATER`, `SWITCH_PAYPAL` (PayPal), `GLOBAL_CARD_PAYMENT_STRIPE` (Stripe), `DUITNOW_ONLINE_BANKING` (DuitNow), `QR_PAYMENT` (DuitNow QR) |
| `payment_model_reference_id` | String (examples send number) | Yes | `WEB_PAYMENT`: `1` = B2C, `2` = B2B. Other types: `1` = default |
| `payment_status` | String | Yes | `active` or `inactive` |

B2B lists exist only for `WEB_PAYMENT`. B2C FPX is for personal bank accounts (for example M2U), B2B for business accounts (for example M2U BIZ); rates differ and you can integrate both.

Response `data.list.data[0].<PAYMENT_TYPE>[]` items:

| Field | Example |
| --- | --- |
| `payment_service_id` | `16` (use this in `create-bill-silent`) |
| `payment_model_reference_id` | `1` |
| `payment_model_reference` | `B2C`, `B2B1` |
| `unique_reference` | `AFFIN_BANK_WEBPAY`, `UOB_TESTID_WEBPAY`, `AFFIN_BANK_B2B1_WEBPAY` |
| `payment_service` | `WEB_PAYMENT` |
| `name` | `Affin Bank`, `United Overseas Bank - B2C Test` |
| `record_status_id` / `record_status` | `1` / `ACTIVE` |

IDs shown in docs examples: DuitNow QR `114`; card for the Credit Card library in production `1100`; examples also use `33`, `89`. IDs are environment-specific; you may cache the list but LeanX recommends refreshing daily. The full bank list is not printed in the docs; call the endpoint.

## Create bill: hosted payment page

`POST /api/v1/merchant/create-bill-page` (optional query `invoice_no` = your bill number)

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `collection_uuid` | String | Yes | Collection UUID |
| `amount` | Decimal (10,2) | Yes | Total in MYR ringgit, for example `119.00` |
| `invoice_ref` | String | Yes (quick start) | Merchant invoice reference; the Create Bill page omits it, send it anyway |
| `redirect_url` | String | Yes | Browser return URL |
| `callback_url` | String | No | Server callback URL (set it; it is the only push notification) |
| `full_name` | String | Yes | Payer name |
| `email` | String | Yes | Payer email (only the format is validated) |
| `phone_number` | String | Yes | Payer mobile, for example `0112459822` |

Response `data`:

| Field | Notes |
| --- | --- |
| `collection_uuid` | Echo |
| `redirect_url` | Send the customer here. In live responses this is the LeanX payment page URL (`https://payment.leanx.dev/collection?id=...`); the quick-start sample shows your own return URL instead |
| `old_redirect_url` | Legacy portal payment URL (`https://portal.leanx.dev/pay/collections/...`) |
| `bill_no` | Bill number (equals `invoice_no` query if you sent one) |
| `invoice_ref` | Echo (quick-start sample) |

The customer cannot change the amount during payment.

## Create bill: silent (own page, QR, card)

`POST /api/v1/merchant/create-bill-silent` (quick start also lists optional query `invoice_no`)

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `collection_uuid` | String | Yes | Collection UUID |
| `amount` | Decimal (10,2) | Yes | MYR ringgit |
| `callback_url` | String | Yes | Server callback URL |
| `redirect_url` | String | No | Browser return URL |
| `full_name` | String | Yes | Payer name |
| `email` | String | Yes | Payer email |
| `phone_number` | String | Yes | Payer mobile |
| `payment_service_id` | Number | Yes | From `list-payment-services` |
| `invoice_ref` | String | Yes | Merchant invoice reference |

Response `data` (Create Bill page version):

| Field | Type | Notes |
| --- | --- | --- |
| `bill_no` | String | System bill number |
| `invoice_ref` | String | Echo |
| `payment_url` | String | Where the customer completes payment (bank/wallet/QR page) |
| `metadata` | Array or Object | Empty `[]` for non-QR; for DuitNow QR an object with `qr_data`, `qr_type` |

The quick-start sample instead returns `bill_no`, `transaction_invoice_no` (for example `FPX1700639540AlnAnAy0`) and `redirect_url`. The Credit Card library needs `transaction_invoice_no`, so read both shapes defensively.

### DuitNow QR

Use the silent bill with the DuitNow QR `payment_service_id` (example `114`). Two flows:

1. LeanX-hosted: redirect to `payment_url`; the LeanX page polls status and redirects to your `redirect_url` on success.
2. Self-hosted: render `metadata.qr_data`.
   - `qr_type: "png"`: base64 PNG; render as `data:image/png;base64,<qr_data>` (check whether it already starts with `data:`).
   - `qr_type: "emv"`: raw EMVCo string; generate the QR with any QR library.
   - Poll `manual-checking-transaction` every 3 seconds until final, then redirect.
   - Follow the official DuitNow QR display guidelines (PayNet document linked from the Create Bill page).

### Credit Card library (embedded card form)

```html
<script src="https://sdk.leanx.io/index.umd.min.js"></script>
```

```javascript
const cc = new LeanxCC({ containerId: 'card-container' });
cc.init();                               // render form
await cc.validateCard();                 // rejects with code cc-form-validation
await cc.submit(transactionInvoiceNo);   // validates, submits, redirects to provider
```

- `submit(id)` takes `transaction_invoice_no` from a silent bill created with the card `payment_service_id` (`1100` in production).
- Error codes: `cc-form-validation`, `id-required`, `invalid-id`.
- The browser result is not proof of payment; settle from callback plus requery.

## Payment callback

Sent to `callback_url` "upon every successful transaction". Body:

```json
{ "data": "<JWT>", "response_code": 2100 }
```

The JWT header is `{"alg":"HS256","typ":"JWT"}`. LeanX says it must be "decoded with a hash key" and that the Hash Key acts as the signature proving the callback was meant for you; if decoding with your Hash Key fails, the callback may come from an unknown source.

Decoded payload fields (from the docs sample):

| Field | Example / notes |
| --- | --- |
| `invoice_no` | `BP1701155880uBMwB9q0` (LeanX bill/invoice no) |
| `client_data.merchant_invoice_no` | Your invoice/bill reference |
| `client_data.uuid` | API key UUID |
| `client_data.order_id` | `"None"` unless set (WooCommerce plugin sends `client_data` with the order id) |
| `invoice_status_id` | `2` for SUCCESS |
| `invoice_type_id` / `invoice_type` | `1` / `NORMAL_COLLECTION` |
| `invoice_status` | `SUCCESS` |
| `amount` | String, for example `"456.20"` |
| `payment_model_reference_id` | 1 = B2C |
| `rate_type_reference_id` | `2` |
| `payment_service_id` | `89` |
| `client_redirect_url`, `client_callback_url` | Echo of your URLs |
| `client_transaction_details` | `null` |
| `prefund_collection` | `{ "item": null, "response_code": 2100 }` |
| `description` | `null` |
| `card_token` | `null` (no tokenisation docs) |
| `fpx_buyer_name` | Payer name from FPX |
| `fpx_transaction_id` | FPX transaction id |
| `fpx_debit_auth_code`, `fpx_credit_auth_code` | `"00"` = approved |
| `fpx_debit_status` | `Approved` |
| `transaction_response_time` | `28 November 23 03:18:03 PM` |
| `original_transaction_response_time` | `2023-11-28 15:18:03` |

Delivery rules (FAQ):

- Sent only for successful transactions. Failed or abandoned payments produce no callback; detect them by requery.
- LeanX expects HTTP 200; otherwise it resends up to 3 times. Response body format does not matter.
- The `token` value in the callback equals the one in the original request.

### Verifying the callback

Recommended (derived from the HS256 header and the "decode with hash key" instruction; confirm in sandbox):

```text
body   = parse_json(raw_request_body)
jwt    = body["data"]
claims = jwt_decode(jwt, key = LEANX_HASH_KEY, algorithms = ["HS256"])   # verifies HMAC-SHA256 signature
if fails -> respond 400/401, do not settle
status = POST manual-checking-transaction?invoice_no=<your invoice_ref or claims.invoice_no>
require status.data.transaction_details.invoice_status == "SUCCESS"
require decimal(status.amount) == expected and decimal(claims.amount) == expected
settle idempotently, then respond 200
```

- The JWT carries no `exp`; protect against replay with idempotent settlement keyed on invoice number.
- The official plugins instead POST the JWT and the Hash Key to `/api/v1/jwt/decode`. That sends your secret over the network and is not in the API docs; prefer local verification. If local HS256 verification fails with the Hash Key as a UTF-8 string in sandbox, contact LeanX before falling back.

## Transaction status (requery)

`POST /api/v1/merchant/manual-checking-transaction?invoice_no=<value>` (no body)

`invoice_no` can be the system invoice number or your `invoice_ref`. LeanX says this is recommended over the callback approach for checking status.

Response `data`:

| Field | Notes |
| --- | --- |
| `transaction_details.invoice_no` | LeanX invoice, for example `BP-AD5112621A-LNP` |
| `transaction_details.fpx_invoice_no` | Provider transaction invoice, for example `FPX17116902144iiAlu58` |
| `transaction_details.amount` | String, `"15.00"` |
| `transaction_details.invoice_status` | `SUCCESS`, `PENDING`, `FAILED` (plugins also see `FPX_STOP_CHECK_FROM_PS`, `SENANGPAY_STOP_CHECK_FROM_PS`, treated as failed) |
| `transaction_details.providerTypeReference` | `FPX` |
| `transaction_details.bank_provider` | `Maybank2U` |
| `transaction_details.category_code` | `WEBPAY` |
| `transaction_details.amount_with_fee` | Number, `15.80` |
| `transaction_details.fee` | Number, `0.80` |
| `transaction_details.fee_by_customer` | Boolean |
| `customer_details.name`, `.phone_number`, `.email` | Payer |

The older "Get Bill Transaction Status" page documents the same path as GET and returns `company_name`, `company_number`, `nature_of_business`, `company_details` instead of `customer_details`.

Polling guidance: every 3 seconds for a QR screen; the WooCommerce plugin rechecks pending orders after 5 minutes and cancels after 12 hours.

## Bills management

### Bill list: `POST /api/v1/merchant/bill-list?skip=0&limit=10`

Body as collection list plus `invoice_status` (`success` / `failed` / `pending`, optional). Item fields: `id`, `invoice_no`, `invoice_status_id`, `invoice_status`, `collection_id`, `customer_id`, `full_name`, `email`, `phone_number`, `total`, `total_amount_with_fee`, `transaction_fee`, `quantity`, `price_per_quantity`, `payment_date`, `transaction_invoice_no`, `bill_transaction_type_id`, `payment_record_id`, `checking_counter`, `record_status`, `api_key`, `base_url`, `status`, `shipping_address_id`, `account_id`, `created_at`, `updated_at`, and `customer_bill_payment_record` (`reference_number`, `payment_date`, `payment_method` such as `STANDARD_CHARTERED_WEBPAY`, `payment_mode` such as `WEB_PAYMENT`, `amount`, `net_amount`, `transaction_fee`, `bank_code`, `record_status`, ...).

### Get / update / delete bill

| Operation | Method, path | Body |
| --- | --- | --- |
| Get | POST `/api/v1/merchant/bill-id` | `{"_id": 50}` (numeric bill id) |
| Update | PUT `/api/v1/merchant/update-bill` | `{"id": 50, "full_name": "...", "email": "...", "phone_number": "..."}`; only `id` required |
| Delete | PUT `/api/v1/merchant/delete-bill` | `{"_id": 50}`; result `record_status` 4 |

## Transaction list

`GET /api/v1/merchant/transaction-list?skip=0&limit=10&start_date=01-01-1970&end_date=31-01-2099&invoice_status=SUCCESS` (no body)

| Query | Req | Notes |
| --- | --- | --- |
| `skip` | No | Rows to skip |
| `limit` | No | Rows to return, maximum 100 per call |
| `start_date`, `end_date` | Yes | `DD-MM-YYYY` |
| `invoice_status` | Yes (table) | `SUCCESS` or `PENDING` |

Response `data`: `total-date-sale`, `total-date-count`, `transaction[]` with keys containing spaces: `Invoice No`, `Invoice Status`, `Name`, `Email`, `Phone Number`, `Transaction Invoice No`, `Payment Mode`, `Amount`, `Payment Date`.

## Pool balance

`POST /api/v1/merchant/overall-balance` (no body). Response `data.transaction_balance` (`value`, `pool_name` `TRANSACTION`, `pool_type`, `api_key`, `account_id`, `id`, `record_status`, `created_at`, `updated_at`), `data.prefund_balance` (number), `data.total_transaction_as_of_today`.

## Payouts (DuitNow instant transfer)

Instant payout to Malaysian bank and e-wallet account holders via DuitNow, from your prefund pool. Service runs 24/7 including weekends and public holidays, real time, no approval (except `STATUS_PENDING_APPROVAL`, below). Payout service covers FPX B2C channels only.

### Payout service list: `POST /api/v1/merchant/list-payout-services` (no body)

Items under `data.list.data[0].SWITCH_PAYOUT_WEBPAY[]`: `payout_service_id`, `payment_model_reference_id`, `payment_model_reference` (`B2C`), `unique_reference` (for example `PAYOUT_MBSB_BANK_WEBPAY`, `PAYOUT_AGROBANK_WEBPAY`), `payment_service` (`SWITCH_PAYOUT_WEBPAY`), `name`, `record_status_id`, `record_status`.

### Bank account validation: `POST /api/v1/merchant/check-verification-bank`

Body (the docs label it "Request Query" but show JSON):

| Field | Req | Notes |
| --- | --- | --- |
| `payout_service_id` | Yes | From payout service list |
| `third_party_account_no` | Yes | Recipient account number |

Response `data.Success` (bool), `data.Result.OrgnThirdPartyBankBICCode`, `data.Result.OrgnThirdPartyAccountNo`, `data.Result.CreditorName`. Not a financial request. Let the user pick the bank first. Banks may truncate names (Maybank 20 characters) or add honorifics (Bank Islam "Cik", "Puan", "Encik"); LeanX suggests allowing about 80% name similarity.

Deprecated channel list: `POST /api/v1/merchant/list-verification-channel`, items with `payout_service_id`, `unique_reference`, `name`, `category_code`, `group_code`, `channel_code`, `duitnowServices[]` (`staging_bic_code`, `production_bic_code`).

### Create payout: `POST /api/v1/merchant/create-payout-invoice`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `virtual_pool_reference` | String | Yes | From portal Prefund / Pool Fund page; one per merchant |
| `payout_service_id` | Number | Yes | From payout service list |
| `amount` | Decimal (10,2) | Yes | MYR |
| `recipient_name` | String | Yes | Required (FAQ) |
| `client_callback_url` | String | Yes | Payout callback URL |
| `third_party_account_no` | String | Yes | Recipient account number |
| `external_invoice_ref` | String | No | Your payout id; use it for status queries |
| `recipient_reference` | String | Yes | Transfer reference |

Response `data.status` (`SUCCESS`), `data.invoice_no` (for example `PAYOUT-A9D5:174307-LEANX`).

Keep at least RM2.00 in the pool above the payout amount (to pay RM100, hold RM102); refunded on account deactivation. Insufficient funds fail finally (error 5993 / 10020); create a new request after top-up. By default collection income settles to your bank account; settling into the payout pool can be arranged before go-live.

### Payout status: `POST /api/v1/merchant/get-payout-transaction-by-id?_id=<payout invoice no or external_invoice_ref>`

Prefer `external_invoice_ref`. Response `data`: `payout_invoice_no`, `merchant_invoice_id`, `id`, `payout_account_number`, `transaction_fee`, `switch_transaction_fee`, `account_id`, `record_status`, `switch_invoice_id`, `transaction_invoice_no`, `transaction_status`, `value` (float), `api_key`, `created_at`, `updated_at`; on failure an `error_message` object (`response_code` such as 5223, `error_code` such as `"77"`, `error_message`, `description` `PAYOUT_FAILED`).

| Status | Meaning |
| --- | --- |
| `SUCCESS` | Transferred; final |
| `PENDING` | Sent to LeanX; needs checking |
| `STATUS_PENDING_APPROVAL` | Provider limitation; manual approval by LeanX ops, 30 to 40 minutes |
| `FAILED` | Failed; final |

### Payout transaction list: `POST /api/v1/merchant/payout-transaction-list?skip=0&limit=10`

Body: `start_date`, `end_date` (`DD-MM-YYYY`), `record_status` (1 active, 2 inactive), `search{search_enable, search_key, search_column}`, `sort{parameter_name, sort_type}`; all required. Set `search_enable` to `false` if no `search_key`. Results are scoped to the Auth Token used.

### Payout callback

Sent to `client_callback_url` only for successful payouts, as `{"data": "<HS256 JWT>", "response_code": 2100}`, decoded with the Hash Key. Decoded payload:

- `output.invoice_no`, `output.error_description[]`
- `output.bank_response.Success`, `output.bank_response.Result` (`OrgnFromAccountCurrency`, `OrgnThirdPartyAccountNo`, `OrgnPaymentDescription`, `BizMsgID`, `DisplayName`, `BankReferenceNo`, `OrgnTransactionAmount`, `PaymentStatus` such as `ACSP`, `OrgnThirdPartyBankBICCode`, `OrgnCurrency`, `OrgnRecipientReference`)
- `output.client_data` (`virtual_pool_reference`, `external_invoice_ref`, `recipient_reference`)
- `output.invoice_status_id` (2), `output.invoice_status` (`SUCCESS`), `output.amount` (string), `output.payout_service_id`, `output.client_callback_url`, `output.client_transaction_details`
- `output.payout_payment.item` (`balance`, `virtual_pool_reference`, `record_status_id`, `virtual_pool_collection_type_id`, `virtual_pool_alias`, `virtual_pool_transaction_amount`, `payout_payment_amount`, timestamps), `output.payout_payment.response_code`
- `response_code` 2100

FAQ: `output.amount` and `output.payout_payment.item.payout_payment_amount` represent the same value. Amounts in callbacks are strings; in the status API they are floats.

## Status and record enums

| Enum | Values |
| --- | --- |
| `invoice_status` (bill) | `SUCCESS`, `PENDING`, `FAILED`; plugin-observed `FPX_STOP_CHECK_FROM_PS`, `SENANGPAY_STOP_CHECK_FROM_PS` |
| `invoice_status_id` | 1 = PENDING, 2 = SUCCESS (from samples; other ids not documented) |
| `record_status` | 1 = active, 2 = inactive/deactivated, 4 = deleted (from samples) |
| `payment_model_reference_id` | 1 = B2C / default, 2 = FPX B2B |
| `response_code` | 2000 success, 2100 callback success, 1911 failed (example) |
| Payout `transaction_status` | `SUCCESS`, `PENDING`, `STATUS_PENDING_APPROVAL`, `FAILED` |

## Error codes

| Code | Description |
| --- | --- |
| 1104 | Transaction Charges Problems Occur |
| 1106 | Audit Writing Problems Occur |
| 2111 | Record Not Found |
| 2011 | Pending Approval |
| 2012 | Pending |
| 3333 | The Record Cannot Be Deleted |
| 3614 | The Invoice Number Is Not Available |
| 4000 | General Error |
| 4001 | Missing X-Timestamp header |
| 4002 | Missing X-Signature header |
| 4004 | Missing X-Nonce header |
| 4006 | Missing Secret Key |
| 4008 | Invalid HMAC signature |
| 4013 | Request Timestamp Is Expired |
| 4015 | Invalid Timestamp Format |
| 4125 | Bill Collection Exist |
| 4637 | Not A Valid Product ID |
| 4624 | Not A Valid Account Number |
| 4626 | Bill Invoices Already Exist |
| 4791 | Transaction Block, Please Contact Admin |
| 4794 | Permission Denied To Create Payment |
| 5223 | Bank Account Holder Verification |
| 5788 | Quota Limit Exceeded |
| 5688 | Invalid Token |
| 5699 | Invalid UUID |
| 5993 | Insufficient Pool Balance |
| 6112 | Amount Cannot Be Null |
| 6482 | Recipient Name Is Required |
| 6666 | Invalid JWT Token |
| 7611 | Name Mismatch For Payout Receipient |
| 7651 | Insufficient Product |
| 7777 | JWT Token Expired |
| 8411 | Collection UUID Not Exist |
| 9100 | Service Is Not Available Right Now |
| 10018 | Payment Service Not Active |
| 10019 | Account Number Not Valid |
| 10020 | Insufficient Fund Amount To Transfer |
| 10039 | Transaction Invoice Not Valid |
| 10041 | Payment Service ID Not Active |
| 10064 | Payout Failed |
| 10149 | Duplicate Payout transaction |
| 10150 | Accepted For Processing |
| 10201 | Unauthorized Due To Permission |
| 10289 | No List In The Record |

## Sandbox test data

- Public sandbox credential above (generic responses only).
- Sandbox bank list includes test banks such as `UOB_TESTID_WEBPAY` ("United Overseas Bank - B2C Test").
- Payout test accounts: `1111111111` returns FAILED, `1212121212` returns PENDING; other accounts succeed.
- Test cards and FPX test bank credentials: Not documented in official docs (checked 2026-10-21).

## Not documented (checked 2026-10-21)

- Refund / void / cancel / capture API.
- Recurring billing, subscriptions, card tokenisation (`card_token` field exists but is unexplained).
- Rate limits (only error 5788 Quota Limit Exceeded and the 100-row transaction list cap).
- Callback source IP addresses; webhook signature header (the JWT itself is the proof).
- Bill expiry time, minimum/maximum amounts per method, settlement report API.
- Postman collection (heading exists on the Credential page with no content).

## Documented inconsistencies

1. `manual-checking-transaction` is POST on the quick start and Transaction Status pages, GET on "Get Bill Transaction Status". Plugins use POST. Use POST.
2. Delete Collection page shows both PUT `/api/v1/merchant/delete-payment-collection` (on `api.leanx.io`) and POST `/api/v1/merchant/delete-payment-portal`. Test in sandbox.
3. `create-bill-page`: `invoice_ref` required in the quick start, absent on the Create Bill page. `callback_url` optional here but required for `create-bill-silent`; `redirect_url` the reverse.
4. `create-bill-silent` response: quick start returns `transaction_invoice_no` and `redirect_url`; Create Bill page returns `invoice_ref`, `payment_url`, `metadata`.
5. `create-bill-page` sample response returns your own `redirect_url` in the quick start but the LeanX payment page URL on the Create Bill page.
6. Bill Transaction Status page says statuses are "Active, Pending, Failed"; response values are `SUCCESS` / `PENDING` / `FAILED`.
7. Activate Collection page text says "This will create a bill and return a link", which is a copy-paste error; it activates the collection.
8. `virtual_pool_reference` location named "Pool Fund" page in the quick start and "Prefund" page on Create Payout.
9. FAQ says Hash Key is "used to decrypt callback data". The callback is a signed (not encrypted) HS256 JWT; the payload is readable without the key, so always verify the signature.
10. FAQ says UUID is "used for record-keeping purposes" while the X-Signature page makes it part of the signed message.
