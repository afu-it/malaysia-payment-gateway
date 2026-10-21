# Payright API Reference

Use this reference for factual Payright (Malaysia) API guidance. Everything here comes from the official GitBook docs at `https://docs.payright.my/` (all 23 pages, checked 2026-10-21) and the official Postman collection linked from the docs. Where the docs are silent, this file says "Not documented in official docs (checked 2026-10-21)". Verify current portal settings, payment method activation and settlement before claiming production certainty.

## Source URLs

- `https://docs.payright.my/llms.txt` (index) and `https://docs.payright.my/llms-full.txt` (all pages)
- `https://docs.payright.my/introduction`
- `https://docs.payright.my/endpoints`
- `https://docs.payright.my/authentication`
- `https://docs.payright.my/error-codes`
- `https://docs.payright.my/signature-validation`
- `https://docs.payright.my/sandbox`
- `https://docs.payright.my/api/payment-provider`
- `https://docs.payright.my/api/collection/create-collection` (and get, update, get-all)
- `https://docs.payright.my/api/bill/create-bill`
- `https://docs.payright.my/api/bill/get-bill`
- `https://docs.payright.my/api/bill/callback`
- `https://docs.payright.my/api/payout`, `/api/payout-provider`, `/api/payout/create-payout`, `/api/payout/get-payout`, `/api/payout/callback`
- `https://docs.payright.my/api/get-balance`
- `https://docs.payright.my/postman-collection` (Google Drive zip: "Payright API (Sandbox).postman_collection.json" plus a sandbox environment file)

## Model

- **Collection**: a group of bills (for example "Rental"). You need a `collection_uuid` to create a bill.
- **Bill**: one payment request. Creating it returns a hosted `payment_url`. The payer pays by FPX online banking at the bank chosen with `provider_id`.
- **Payment provider**: an FPX bank channel, B2C (retail) or B2B (corporate, listed as `B2B1` in the Postman sample).
- **Payout**: a transfer from your virtual payout pool to a recipient bank account.

Choose the simplest path. For a custom checkout: Collection (once) > list providers > Create Bill > redirect > JWT callback > Get Bill confirm.

## Environments

| Environment | API base URL |
|---|---|
| Production | `https://api.payright.my/api/v1` |
| Sandbox | `https://api.payright-sandbox.my/api/v1` |

Source: `https://docs.payright.my/endpoints`, `https://docs.payright.my/sandbox`. All endpoint pages show the production host; swap the host for sandbox. The Postman environment uses `API_BASE_URL=https://api.payright-sandbox.my/` and `API_VERSION=api/v1/`.

Sandbox payment pages are served from `https://switch.payright-sandbox.my/psapi/api/v1/env/mobile/client/simulator/PaymentPageSimulator?invoice_no=FPX...` (from the Create Bill sample), which is a simulator, not a real bank.

Sandbox note from the docs: "All of the failed bill will not be counted as a payment at dashboard and settlements."

Merchant portal (where keys live): `https://portal.payright.my/` (linked from the payout and balance pages).

Transport: JSON over HTTPS, UTF-8 (`https://docs.payright.my/introduction`). Send `Content-Type: application/json` on POST bodies.

## Authentication

Send the Auth Token in a header named `auth-token`:

```text
auth-token: <AUTH_TOKEN>
```

- The Authentication page words it as "include an Authorization header with the value `auth-token {your-token}`". Every concrete example (cURL, Python, Ruby, the Sample Header block) and the official Postman collection instead send a header literally named `auth-token` whose value is the token. Use the `auth-token` header. If you get 401 or `INVALID_TOKEN`, confirm with Payright support rather than guessing a third format.
- Where to get it: `Accounts > API Keys > View API Keys (magnify icon) > Auth Token`.
- The token itself looks like a JWT (`eyJhbGciOi...`). Treat it as an opaque secret.
- Invalid token error sample (Get Balance page): `{"status":"fail","code":403,"data":{"host":"api.payright-sandbox.my"},"uuid":null,"message":"INVALID_TOKEN","error":null}`.

## Response Envelope

All documented responses use:

| Field | Type | Notes |
|---|---|---|
| `status` | string | `"ok"` or `"fail"` |
| `code` | number | `200` on success for most endpoints; payout endpoints return `2000` on success. Error codes listed below. |
| `data` | object or null | Payload |
| `uuid` | string or null | Request trace id; log it |
| `message` | string or null | Human-readable message, e.g. `"Insufficient pool balance."` |
| `error` | string or null | Error description |

The Error Codes page says that "upon status code is 200 OK", processing errors are reported inside the body. So check `status` and `code` in the body, not only the HTTP status.

## Endpoint Catalogue

All paths are relative to the base URL (`.../api/v1`). Every endpoint needs `auth-token` (and `x-signature` if JWT Verification is on).

| # | Method | Path | Purpose | Docs page |
|---|---|---|---|---|
| 1 | POST | `/merchant/list-payment-provider` | List FPX banks (B2C/B2B) and their `provider_id` | `/api/payment-provider` |
| 2 | POST | `/merchant/collections` | Create collection | `/api/collection/create-collection` |
| 3 | GET | `/merchant/collections/{collection_uuid}` | Get one collection | `/api/collection/get-collection` |
| 4 | POST | `/merchant/collections/{collection_uuid}` | Update collection title/description | `/api/collection/update-collection` |
| 5 | GET | `/merchant/collections?skip=0&limit=10` | List collections (paginated) | `/api/collection/get-all-collections` |
| 6 | POST | `/merchant/bills` | Create bill, get `payment_url` | `/api/bill/create-bill` |
| 7 | GET | `/merchant/bills/{invoice_no}` or `/merchant/bills/{invoice_ref}` | Get bill status | `/api/bill/get-bill` |
| 8 | (inbound) POST to your `callback_url` | Bill callback (JWT) | `/api/bill/callback` |
| 9 | POST | `/merchant/list-payout-providers` | List payout banks and `payout_provider_id` | `/api/payout-provider` |
| 10 | POST | `/merchant/payout` | Create payout | `/api/payout/create-payout` |
| 11 | GET | `/merchant/payout/{reference}` | Get payout by `invoice_no` or `external_invoice_ref` | `/api/payout/get-payout` |
| 12 | (inbound) POST to your `client_callback_url` | Payout callback (JWT) | `/api/payout/callback` |
| 13 | GET | `/merchant/balance` | Collection pool, payout pool, today's total | `/api/get-balance` |

Docs inconsistency: the Payout overview table lists Get Payout as `GET /api/v1/merchant/:invoice_no`, while the Get Payout page and all its examples use `GET /api/v1/merchant/payout/:reference`. Use the Get Payout page form.

Not documented in official docs (checked 2026-10-21): refund, cancel/void bill, delete collection, recurring/tokenisation API, card or e-wallet bill API, settlement report API, transaction list API, webhook registration API, rate limits, IP allowlists, idempotency headers.

## 1. Payment Provider List

`POST /merchant/list-payment-provider` (no body).

Response:

```json
{
  "status": "ok",
  "code": 200,
  "data": {
    "services": [
      { "provider_id": 16, "name": "Affin Bank Berhad", "bank_type": "B2C" },
      { "provider_id": 86, "name": "UOB Bank", "bank_type": "B2B" }
    ]
  },
  "uuid": "408e99d8-dbc5-424d-908c-d2a1bec8316d",
  "message": null,
  "error": null
}
```

| Field | Type | Notes |
|---|---|---|
| `data.services[].provider_id` | integer | Pass as `provider_id` on Create Bill |
| `data.services[].name` | string | Bank display name |
| `data.services[].bank_type` | string | `B2C` (consumer) or B2B (docs page says `B2B`; Postman sandbox sample shows `B2B1`) |

The docs note the list contains both B2C and B2B channels. Filter by `bank_type` for your audience.

Sandbox provider IDs seen in the official Postman sample (sandbox only, may change; always load the live list):

| provider_id | name | bank_type |
|---|---|---|
| 16 | Affin Bank Berhad B2C - Test ID | B2C |
| 17 | Affin Bank | B2C |
| 18 | AGRO Bank | B2C |
| 19 | Alliance Bank (Personal) | B2C |
| 20 | AmBank | B2C |
| 21 | Bank Islam | B2C |
| 22 | Bank Rakyat | B2C |
| 23 | Bank Muamalat | B2C |
| 24 | Bank Simpanan Nasional | B2C |
| 25 | CIMB Clicks | B2C |
| 26 | Hong Leong Bank | B2C |
| 27 | HSBC Bank | B2C |
| 28 | KFH | B2C |
| 29 | Maybank2U | B2C |
| 30 | OCBC Bank | B2C |
| 31 | Public Bank | B2C |
| 32 | RHB Bank | B2C |
| 33 | Standard Chartered Bank | B2C |
| 34 | United Overseas Bank - B2C Test | B2C |
| 35 | UOB Bank | B2C |
| 36 | Maybank2E | B2C |
| 38, 39, 40 | SBI BANK A, B, C (FPX test banks) | B2C |
| 63 to 84, 86 | `<BANK>_B2B1_WEBPAY` corporate channels (Affin, Alliance, Agro, Bank Islam, AmBank, Muamalat, BNP Paribas, CIMB, Citibank Corporate, Deutsche, Hong Leong, HSBC, Bank Rakyat, KFH, Maybank2E, OCBC, Public Bank PBe, PB Enterprise, RHB, SBI A/B/C, UOB) | B2B1 |

Note: payment `provider_id` values and payout `payout_provider_id` values are different ID spaces (Affin is 16/17 for payments and 25 for payouts in the samples).

## 2 to 5. Collections

### Create Collection

`POST /merchant/collections`

| Field | Type | Required | Notes |
|---|---|---|---|
| `title` | string | yes | Collection title |
| `description` | string | yes | Collection description |

Response `data`: `collection_uuid` (e.g. `PyC0D7F54A90rgt`), `title`, `status` (`ACTIVE` in docs; Postman sample shows `"Title"` and `"active"`, so read keys and values case-insensitively).

### Get Collection

`GET /merchant/collections/{collection_uuid}` (path parameter, required).

Response `data`: `title`, `description`, `collection_uuid`, `status`, `created_at` (e.g. `2023-01-18T18:24:00`).

### Update Collection

`POST /merchant/collections/{collection_uuid}` with body `title` (required), `description` (required). Response `data` as Get Collection.

### Get All Collections

`GET /merchant/collections?skip=0&limit=10`

Response `data.list`: `draw`, `record_total`, `record_filtered`, `data[]` (each `title`, `description`, `status`, `collection_uuid`, `created_at`), `next_page_start`, `next_page_length`, `previous_page_start`, `previous_page_length`.

You can also create collections in the portal; the 2022 WooCommerce plugin told merchants to "create new collection at your payright dashboard".

## 6. Create Bill

`POST /merchant/bills`

```json
{
  "collection_uuid": "<COLLECTION_UUID>",
  "amount": 240,
  "invoice_ref": "ORDER-10001",
  "provider_id": 22,
  "redirect_url": "https://merchant.example/payment/return",
  "callback_url": "https://merchant.example/payment/payright/callback",
  "full_name": "Payer Name",
  "email": "payer@example.com",
  "phone_number": "0112459822"
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `collection_uuid` | string | yes | From Create Collection or the portal |
| `amount` | decimal(10,2) | yes | Total in MYR (ringgit, not sen). Example sends `240` and the bill shows `"240.00"` |
| `provider_id` | integer | yes | From Payment Provider List. The payer's bank is chosen before the bill is created |
| `callback_url` | string | yes | Server-to-server callback target |
| `redirect_url` | string | yes | Browser return target |
| `full_name` | string | yes | Payer name |
| `email` | string | yes | Payer email |
| `phone_number` | string | yes | Payer mobile, e.g. `0112459822` |
| `invoice_ref` | string | shown in every example, not in the attribute table | Your own reference. Get Bill accepts it. Error 425 "Invoice already exists" applies to duplicates. Treat as required and unique per attempt |

Response:

```json
{
  "status": "ok",
  "code": 200,
  "data": {
    "invoice_no": "Py4a220e2edcrgt",
    "invoice_ref": "DPL1124519LX",
    "payment_url": "https://switch.payright-sandbox.my/psapi/api/v1/env/mobile/client/simulator/PaymentPageSimulator?invoice_no=FPX1734522325oTfQjC8l"
  },
  "uuid": "716a3267-9fb0-42ab-bb0d-0cfe159c46fb"
}
```

| Field | Type | Notes |
|---|---|---|
| `data.invoice_no` | string | Payright bill number (`Py...rgt`). Store it |
| `data.invoice_ref` | string | Echo of your reference |
| `data.payment_url` | string | Redirect the payer here. Its `invoice_no` query value is the FPX invoice number (`FPX...`) |

Not documented in official docs (checked 2026-10-21): bill expiry, description field, minimum/maximum amount, card or e-wallet channels via API (the pricing page lists e-wallet, card and BNPL as plan payment methods, but the Bill API only documents FPX `provider_id`).

## 7. Get Bill (status requery)

`GET /merchant/bills/{invoice_no}` or `GET /merchant/bills/{invoice_ref}`

The docs warn: "Please ensure that the correct ID is provided to prevent any errors in the API response."

Response:

```json
{
  "status": "ok",
  "code": 200,
  "data": {
    "transaction_details": {
      "invoice_no": "Py4a220e2edcrgt",
      "fpx_invoice_no": "FPX1734522325oTfQjC8l",
      "amount": "240.00",
      "status": "PENDING",
      "fee": null
    },
    "customer_details": {
      "name": "Khairul Mukmin",
      "phone_number": "0112459822",
      "email": "khairul.mukmin@gmail.com"
    }
  },
  "uuid": "408e99d8-dbc5-424d-908c-d2a1bec8316d",
  "message": null,
  "error": null
}
```

| Field | Type | Notes |
|---|---|---|
| `transaction_details.invoice_no` | string | Bill number (`Py...`) |
| `transaction_details.fpx_invoice_no` | string | FPX number (`FPX...`), same value the callback calls `invoice_no` |
| `transaction_details.amount` | string | Two-decimal ringgit string |
| `transaction_details.status` | string | Documented values: `PENDING` (Get Bill sample), `SUCCESS` (callback sample). Other values not documented (checked 2026-10-21); treat anything that is not `SUCCESS` as unpaid |
| `transaction_details.fee` | string or null | Fee, null while pending |
| `customer_details.name`, `.phone_number`, `.email` | string | Payer details |

## 8. Bill Callback

Payright POSTs to `callback_url` "after every successful transaction". The body is JSON:

```json
{"data": "<HS256 JWT>", "response_code": 2100}
```

- `data` is a JWT (header `{"alg":"HS256","typ":"JWT"}`) signed with your **Hash Key** (`Account > API Keys > View API Key > Hash Key`).
- `response_code` sample value is `2100`. Its meaning is not documented (checked 2026-10-21); the payout callback also carries `2100` with `invoice_status: SUCCESS`.

Decoded payload (docs sample):

| Field | Sample | Notes |
|---|---|---|
| `invoice_no` | `FPX1734521578cb7OhtMJ` | FPX invoice number, equals Get Bill `fpx_invoice_no`, NOT the `Py...` bill number |
| `invoice_ref` | `DPL1124599LX` | Your reference. Primary match key |
| `invoice_status` | `SUCCESS` | Only value documented |
| `amount` | `"240.00"` | String, ringgit |
| `fpx_buyer_name` | `Khairul Mukmin` or null | |
| `transaction_response_time` | `18 December 24 07:33:11 PM` | Free-form, not ISO 8601 |
| `invoice_type` | `NORMAL_COLLECTION` | In the sample it equals the collection title |
| `fpx_transaction_id` | `1726547891123` or null | FPX transaction id |
| `fpx_debit_status` | `"00"` or `""` | FPX debit status code; `00` is the success sample |
| `fpx_credit_auth_code` | `00` or null | |
| `fpx_txn_currency` | `MYR` or null | |
| `fpx_buyer_bank` | `Bank Islam Malaysia` or null | |

The decoded sample in the docs is not strictly valid JSON (unquoted `Khairul Mukmin`, `MYR`, `00`). The real JWT example in the docs decodes to valid JSON with `null` for the FPX fields. Parse defensively.

The sample JWT has no `iat` or `exp` claim. Do not require them, but do not skip signature verification.

### Verify the callback (pseudo-code)

```text
body = parse_json(raw_request_body)            # {"data": "...", "response_code": ...}
token = body["data"]
claims = jwt.decode(token, key=PAYRIGHT_HASH_KEY, algorithms=["HS256"])   # raises on bad signature
# reject if decode fails; never use a decode-without-verify helper

payment = db.find_payment(invoice_ref = claims["invoice_ref"])
reject if payment is None
reject if decimal(claims["amount"]) != payment.amount            # compare as decimal, 2dp
if claims["invoice_status"] != "SUCCESS": record and stop
bill = GET /merchant/bills/{payment.invoice_ref}                  # confirm server-side
require bill.data.transaction_details.status == "SUCCESS"
require bill.data.transaction_details.amount == claims["amount"]
mark_paid_once(payment.id, fpx_invoice_no = claims["invoice_no"])  # idempotent
respond 200
```

Node example: `jwt.verify(token, process.env.PAYRIGHT_HASH_KEY, { algorithms: ['HS256'] })` with `jsonwebtoken`. Python: `jwt.decode(token, key, algorithms=["HS256"])` with PyJWT.

Not documented in official docs (checked 2026-10-21): callback retry policy, expected HTTP response from the merchant, callback for failed or cancelled payments (docs only promise callbacks "after every successful transaction"), source IPs, and the query parameters appended to `redirect_url`. Because failures may never trigger a callback, run a scheduled Get Bill requery for pending payments.

## Request Signing (X-Signature, optional)

Source: `https://docs.payright.my/signature-validation`.

- Toggle: `Account > API Keys > Edit > JWT Verification` (enable or disable).
- When enabled, every API request must carry an `x-signature` header.
- The signature is a JWT, HS256, signed with the **Hash Key**, over this payload:

```json
{
  "auth-token": "<AUTH_TOKEN>",
  "http_method": "POST",
  "url_path": "/api/v1/example",
  "iat": 1735936254,
  "exp": 1735936554
}
```

| Claim | Notes |
|---|---|
| `auth-token` | Your Auth Token string |
| `http_method` | Upper-case HTTP method of this request |
| `url_path` | Path including `/api/v1`, e.g. `/api/v1/merchant/bills`. The docs example has no query string; whether to include `?skip=...` for list calls is not documented (checked 2026-10-21) |
| `iat` | Issued-at, Unix seconds |
| `exp` | Expiry, Unix seconds. Docs example uses `iat + 300` (5 minutes); you choose the window |

The page also says `x-signature` is "the HMAC-SHA256 signature". That is the HS256 JWT signature; send the full JWT string, not a bare HMAC hex digest.

Pseudo-code:

```text
now = unix_seconds()
sig = jwt.encode({"auth-token": AUTH_TOKEN, "http_method": "POST",
                  "url_path": "/api/v1/merchant/bills", "iat": now, "exp": now + 300},
                 key=HASH_KEY, algorithm="HS256")
headers = {"auth-token": AUTH_TOKEN, "x-signature": sig, "Content-Type": "application/json"}
```

Sample header block from the docs:

```text
POST /api/v1/merchant/bills HTTP/1.1
Host: api.payright-sandbox.my
Content-Type: application/json
auth-token: <AUTH_TOKEN>
x-signature: <SIGNATURE_JWT>
```

Generate a fresh signature per request; keep server clocks in sync (NTP) because of `exp`.

## 9 to 12. Payouts

Overview: disburse from your virtual pool to verified recipient bank accounts. Workflow: list payout providers > create payout > monitor with Get Payout or callback. The overview recommends confirming recipient bank details before paying out; no bank-account verification endpoint is published (checked 2026-10-21).

### Payout Provider List

`POST /merchant/list-payout-providers` (no body).

Response `data.list`: `draw`, `record_filtered`, `record_total`, `data[]` with `payout_provider_id` (number) and `name` (string). Sample: `39` AEON Bank (M) Berhad, `25` Affin Bank Berhad.

### Create Payout

`POST /merchant/payout`

| Field | Type | Required | Notes |
|---|---|---|---|
| `virtual_pool_reference` | string | yes | Your payout pool reference, e.g. `VA-ABC-1234-PAYOUT` |
| `payout_service_id` | integer | yes | `payout_provider_id` from Payout Provider List |
| `amount` | decimal(10,2) | yes | MYR |
| `client_redirect_url` | string | yes | Redirect after the action |
| `client_callback_url` | string | yes | Payout callback target |
| `third_party_account_no` | string | yes | Recipient bank account number |
| `recipient_name` | string | yes | Recipient name (docs description repeats "bank account number"; the example value is a name) |
| `recipient_reference` | string | yes | Reference or name of the recipient |
| `payment_description` | string | yes | Purpose |
| `external_invoice_ref` | string | yes | Your tracking reference; usable in Get Payout |

Success: `{"status":"ok","code":2000,"data":{"status":"SUCCESS","invoice_no":"po-6DC1:125037-pyt"}, ...}`. `data.status` is `SUCCESS`, `FAILED` or `PENDING`.

Error sample: `{"status":"fail","code":5993,"data":null,"message":"Insufficient pool balance."}`.

Duplicate guard (error 10149): the same `amount`, `account_no` and `recipient_reference` within 3 hours is rejected.

### Get Payout

`GET /merchant/payout/{reference}` where `reference` is `invoice_no` (e.g. `po-6DC1:125037-pyt`) or `external_invoice_ref`. URL-encode the reference (it can contain `:`).

Response `data`: `value` (number, MYR), `external_invoice_ref`, `payout_invoice_no`, `payout_account_number`, `created_at` (ISO 8601), `transaction_status` (`SUCCESS`, `FAILED`, `PENDING`). Success `code` is `2000`.

### Payout Callback

Same format as the bill callback: `{"data": "<HS256 JWT signed with Hash Key>", "response_code": 2100}`. Decoded sample fields:

`invoice_no` (`PAYOUT17405484024Zf3tQCg`), `error_description` (array), `invoice_status_id` (`2`), `invoice_status` (`SUCCESS`), `amount` (`"13.00"`), `payment_switch_charge_rate` (`"0.50"`), `payout_service_id` (`39`), `response_code` (`2100`), `bank_response_success` (`true`), `orgn_from_account_currency` (`MYR`), `orgn_third_party_account_no`, `orgn_payment_description`, `biz_msg_id`, `display_name`, `bank_reference_no`, `orgn_transaction_amount`, `from_account_no`.

Note the callback `invoice_no` (`PAYOUT...`) differs in format from the Create Payout `invoice_no` (`po-...-pyt`) in the samples, and the callback sample carries no `external_invoice_ref`. Confirm with Get Payout before marking a payout final. `invoice_status_id` values other than `2` = SUCCESS are not documented (checked 2026-10-21).

## 13. Get Balance

`GET /merchant/balance`

Response `data`: `collection_balance` (number, collection pool), `payout_balance` (number, payout pool), `total_transaction_today` (number, today's collections).

## Error Codes

HTTP status codes (`https://docs.payright.my/error-codes`): `200` OK, `201` Created, `204` No Content.

Body `code` values:

| Code | Meaning |
|---|---|
| 401 | Unauthorized |
| 403 | Invalid UUID (also seen with message `INVALID_TOKEN`) |
| 404 | Not Found |
| 425 | Invoice already exists |
| 500 | Internal Server Error Occur |
| 2000 | Success (payout endpoints) |
| 2100 | Callback `response_code` on a successful transaction (sample; meaning not stated) |
| 5993 | Insufficient pool balance (payout error sample) |
| 6666 | Invalid JWT Token |
| 7777 | JWT Token Expired |
| 9100 | Service Is Not Available Right Now |
| 10018 | Payment Service Not Active |
| 10041 | Payment Service ID Not Active |
| 10149 | Duplicate Payout Transaction (same amount, account_no, recipient_reference within 3 hours) |
| 10201 | Unauthorized Due To Permission |

6666/7777 relate to the `x-signature` JWT: regenerate with a fresh `iat`/`exp` and the correct Hash Key. 10018/10041 mean the payment service or chosen `provider_id` is not active on your account.

## Amounts

- Unit: ringgit (MYR) with two decimals, `decimal(10,2)`. Not sen. MYR is the only currency documented.
- Responses return amounts as strings (`"240.00"`) on bills and callbacks, and as numbers (`13.0`) on Get Payout and Get Balance.
- Compare using decimal or integer sen arithmetic, never floats.

## Sandbox Test Data

- Base: `https://api.payright-sandbox.my/api/v1`.
- Payment page is a simulator (`.../PaymentPageSimulator`).
- Postman sample banks include "Affin Bank Berhad B2C - Test ID" (16), "United Overseas Bank - B2C Test" (34), and SBI BANK A/B/C (38 to 40).
- Test credentials, test bank logins and outcomes for the simulator: Not documented in official docs (checked 2026-10-21).

## Legacy API (do not use for new work)

The official `PayrightMY` GitHub repos (`payment-php`, `payright-for-woocommerce`, `Payright-For-GravityForms`, last pushed 2022) target an older API: base `https://payright.my/api/v1` (sandbox `https://sandbox.payright.my/api`), `Authorization: Bearer <api key>`, fields such as `collection`, `biller_name`, amount in sen (`intval($amount * 100)`), and an HMAC-SHA256 `signature` field over selected callback fields. The WordPress.org listing for that plugin was closed on 2023-07-24. None of this matches the current GitBook docs. Only use it to recognise and migrate old integrations.
