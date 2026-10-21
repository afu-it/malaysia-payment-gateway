# SecurePay API v2 (SecurePay Console) Reference

API v2 is marked "New (preview)" on `https://www.securepay.my/docs/`; SecurePay says to use v1 for production today. Use this file only when the merchant's account and credentials are on SecurePay Console. Checked 2026-10-21 against `https://v2.docs.securepay.my/` (GitBook `https://securepay-4.gitbook.io/console-docs/`).

Many v2 pages show request fields but no response body, and some sample commands disagree with the endpoint heading. Those gaps are flagged inline. Test every call in staging before relying on it.

## Base URLs

| Environment | Base URL |
| --- | --- |
| Production | `https://console.securepay.my/api` |
| Staging (sandbox) | `https://sandbox.securepay.dev/api` |

"Partners will have different base URL." Registration: `https://console.securepay.my/users/sign_up` (observed redirecting to `https://securepay.my/apps/users/sign_up` on 2026-10-21).

## Authentication

Credentials per interface: Client ID (32 hex chars) and Client Secret. An organisation can generate multiple credentials.

`POST {base_url}/v1/auth` with HTTP Basic `client_id:client_secret`:

```bash
curl -X POST "https://sandbox.securepay.dev/api/v1/auth" -u "<CLIENT_ID>:<CLIENT_SECRET>"
```

200:

```json
{"auth_token":"<JWT>","expired_at":"2025-09-19 10:38:00","expired_at_epoch":1758278280,
 "interface":{"name":"console","uid":"01983fcf-df31-7a3b-b61a-a2b27ac5e1bf"}}
```

Failure: `HTTP Basic: Access denied.` (docs). Observed 2026-10-21 without credentials: 401 `{"error":"Access denied. Please check your credentials and try again.","code":401}`.

Send `Authorization: Bearer <auth_token>` on payment calls. Cache the token until shortly before `expired_at_epoch` and re-authenticate after a 401. Token lifetime is not stated beyond the `expired_at` fields.

Env vars: `SECUREPAY_V2_BASE_URL`, `SECUREPAY_CLIENT_ID`, `SECUREPAY_CLIENT_SECRET`.

## Interface Settings (dashboard)

Configured per interface in the Console (Interfaces page): Name, Status (only Active interfaces process payments), Description, Charge Mode (`Merchant`, `Credit`, `Customer`), Virtual account bank, Split Settlement rule, Redirect URL, Cancel URL, Callback URL, Timeout URL, and gateways:

- FPX: Gateway, Charge Mode (`Merchant`, `Credit`, `Customer`, `Inherit`), Bank View, Bank Type (B2C retail or B2B corporate).
- DuitNow: Gateway (participating bank), Charge Mode, Bank View (`List`), Bank Type (typically Retail).

## Get Enabled Payment Methods

`GET {base_url}/v1/interface/gateways` (Bearer).

```json
{"gateways":[
  {"gateway_uuid":"52c75ce3-...","gateway_type":"Fpx","enabled":true,"bank_type":"retail","bank_view":"list"},
  {"gateway_uuid":"9a82fe64-...","gateway_type":"Duitnow","enabled":true,"bank_type":"retail","bank_view":"list"}]}
```

`gateway_type` is the method usable in a payment session. 401: `{"error":{"status":401,"message":"Unauthorized"}}`.

## Bank List

`GET {base_url}/v1/paynet/:gateway_name/banks` or `/v1/paynet/:gateway_name/banks/:bank_type` (Bearer).

| `gateway_name` | `bank_type` |
| --- | --- |
| `fpx` | `b2c` personal, `b2b1` corporate |
| `direct_debit` | `b2c`, `b2b1` |
| `duitnow` | `retail`, `corporate` |

FPX / direct debit response: `{"banks":{"retail":[{name, code, status_format1, status_format2}],"corporate":[...]}}`.

DuitNow response: `{"retail":[{code, name, active, browser, iosApplicationId, androidApplicationId, redirectUrl}], "corporate":[...]}`. DuitNow codes are SWIFT-style (e.g. `MBBEMYKL`, `CIBBMYKL`); the docs sample is staging data, so always fetch live.

Invalid gateway: `{"errors":"Invalid gateway name"}`.

## Create Payment Session (FPX, DuitNow)

`POST {base_url}/v1/payment/intents` (Bearer). Returns a URL to SecurePay's hosted payment page. The page text names it both `payment_url` and `checkout_url`; the response body is not shown in the docs. Log the full response in staging and read the URL key from it.

| Field | Required | Notes |
| --- | --- | --- |
| `buyer_name` | Yes | One line |
| `buyer_email` | Yes | Valid email |
| `buyer_phone` | Yes | e.g. `+60123121678` |
| `amount` | Yes | Cents: `154040` = RM1,540.40 |
| `description` | Yes | e.g. `Payment for order no. 123` |
| `order_number` | No | Your unique reference (strongly recommended) |
| `amount_currency` | No | Default `MYR` |
| `callback_url` | No | Server-to-server status POST |
| `redirect_url` | No | Browser POST after payment |
| `redirect` | No | `true` to show SecurePay payment page |
| `cancel_url` | No | Cancel button on SecurePay page |
| `timeout_url` | No | Page times out within 3 minutes |
| `payment_method` | No | e.g. `Fpx` (from `gateway_type`) |
| `billing_address` | No | Object `line_1`, `line_2`, `city` |
| `custom_fields` | No | Free-form object, e.g. `{"customer_id":"123"}` |
| `split_settlement` | No | Array, see Split Settlement |

No request checksum is documented for v2 intents; Bearer auth protects the call. The Certificate page describes RSA/PQC signing keys per interface but does not document how requests are signed (see Certificate).

## Closed Wallet Payment

`POST {base_url}/v1/payment/intent` (singular in docs). Same fields as above, with `amount` documented as decimal (`1540.40`), which conflicts with the cents format on the intents page; confirm in staging. Extra fields:

| Field | Required | Notes |
| --- | --- | --- |
| `payment_method` | Yes | `ClosedWallet` |
| `cashier_id` | No | Cashier responsible |
| `outlet_id` | No | Outlet where paid |
| `card_payment` | No | `true` if physical card |

User authorises with registered phone number and pincode. Flow: merchant creates intent, user opens Closed Wallet checkout, authenticates, confirms, balance is deducted, user is redirected to receipt.

## eMandate (FPX Direct Debit)

`POST {base_url}/v1/payment/intents` with `payment_method=DirectDebit`. Same base fields as a payment session (`amount` in cents, `buyer_*`, `description`, URLs, `redirect`, `billing_address`, `custom_fields`) plus:

| Field | Required | Values |
| --- | --- | --- |
| `payment_method` | Yes | `DirectDebit` |
| `application_type` | Yes | `01` (enrollment) |
| `id_type` | Yes | `1` new IC, `2` old IC, `3` passport, `4` business registration, `5` others |
| `id_number` | Yes | ID for the chosen type |
| `maximum_frequency` | Yes | Max debits; `999` = no limit |
| `mode_frequency` | Yes | `DL` daily, `WK` weekly, `MT` monthly, `YR` yearly |
| `purpose` | Yes | Mandate purpose |

Mandate statuses: `pending`, `active`, `failed`, `revoked`. How to trigger subsequent debits on an active mandate via API is not documented (checked 2026-10-21).

### Mandate status and lists

| Method | Path | Notes |
| --- | --- | --- |
| GET | `/v1/payment/mandates/status/:identifier` | Enrollment payment record; identifier = order or reference number. The sample curl uses `/manage/payments/mandate/:identifier` instead |
| GET | `/v1/mandates/status/:reference_number` (or `.json`) | Mandate object: `uuid`, `reference_number`, `slug`, `name`, `phone_number`, `email`, `id_type` (`new_ic`), `application_type` (`enrollment`), `purpose`, `maximum_frequency`, `mode_frequency` (`monthly`), `effective_date`, `expiry_date`, `amount{cents,currency_iso}`, `status`, timestamps |
| GET | `/v1/payment/mandates/transactions/list` | Mandate payment records; `status`, `start_date`, `end_date` |
| GET | `/v1/mandates/transactions/list` | Mandates; same filters |

Enrollment record `metadata` includes `id_type`, `purpose`, `bank_code`, `bank_type`, `id_number`, `charge_mode`, `expiry_date`, `account_number`, `effective_date`, `mode_frequency`, `application_type`, `maximum_frequency`, `reason`.

## Payment Record Status

`GET {base_url}/v1/payment/status/:identifier.json` (or send `Accept: application/json`; default response is HTML). Identifier = `order_number` or `reference_number`. The sample curl omits the Bearer header; send it anyway.

Response (record):

| Field | Notes |
| --- | --- |
| `id`, `uuid` | Record IDs |
| `payment_intent_id`, `account_id`, `gateway_id`, `interface_id`, `tenant_id` | Internal IDs |
| `reference_number` | SecurePay reference, e.g. `PYMTRNYZNS1758176376` |
| `order_number` | Your order number |
| `transaction_number` | e.g. `PYMTINTKOLVP1758176376` (FPX seller exchange order no) |
| `amount_cents`, `amount_currency` | Amount in cents |
| `fee_amount_cents`, `fee_amount_currency` | Fee |
| `net_amount_cents`, `net_amount_currency` | Net (can be negative) |
| `partner_uuid`, `tax_id_number`, `initiated_by` | Optional |
| `request_payload` | Raw FPX AR/AD message fields (`fpx_*`) |
| `response_payload` | Raw response |
| `metadata` | `bank_code`, `bank_type`, `charge_mode` |
| `failure_reason` | Reason if failed |
| `environment` | `sandbox` or production |
| `status` | Observed `failed`, `successful`; the Payment Records page lists `pending`, `succeeded`, `failed` |
| `discarded_at`, `created_at`, `updated_at` | Timestamps |

The status page's field table repeats the v1 fields (`payment_status`, `transaction_amount`, ...), which do not match the sample. Code against the sample and confirm in staging. 404: `{"error":{"status":404,"message":"Record not found"}}`.

## Payment Records List

`GET {base_url}/v1/transactions/list` (Bearer). Filters: `status`, `start_date`, `end_date` (same date for daily). Returns an array of records. Empty: `{"msg":"No payment records found!"}`. Pagination and date format are not documented (examples say "18 September 2025").

## Callback and Redirect (v2)

Same semantics as v1: `redirect_url` is a browser POST, `callback_url` is server to server and "very important" if the redirect fails. Documented fields:

| Field | Sample |
| --- | --- |
| `gateway_type` (spelled `gatweway_type` in docs) | `FPX` |
| `payment` | Object: `amount` (`"3.00"`), `status` (`successful`), `net_amount`, `environment` (`sandbox`), `intent_uuid`, `record_uuid`, `gateway_type` (`fpx`), `gateway_uuid`, `reference_number` |
| `currency` | `MYR` |
| `order_number` | Your order number |
| `seller_order_number` | Sample key `fpx_seller_order_no` |
| `exchange_number` | Sample key `fpx_seller_ex_order_no` |
| `buyer_name` | Sample key `fpx_buyer_name` |
| `fpx_debit_auth_code` | `00` success, `99` pending approval (B2B1), other unsuccessful |

The table header names and the sample keys differ; accept both. Content type (form vs JSON) is not stated.

Checksum: the v2 page copies the v1 rule ("sort the key and later replace with the value", HMAC-SHA256 with token, "the string also need to include uid"). It does not say which secret is the key for v2 credentials or how to flatten the nested `payment` object. Therefore for v2: verify a checksum if one is present using the v1 method and a key confirmed with SecurePay, and always settle from an authenticated `GET /v1/payment/status/:identifier.json` that matches `order_number`, `amount_cents`, `amount_currency` and `status`.

## Split Settlement

Per-payment `split_settlement` array on the payment session overrides the interface's rule:

```json
"split_settlement": [
  {"value": 10, "mode": "percentage", "account_number": "20251208164108-000000-0001"},
  {"value": 1,  "mode": "fixed",      "account_number": "20251217044106-000077-0003"}
]
```

Rules: values grouped by mode; fixed sum must be <= payment amount; percentage sum must be <= 100; recipient account must be valid and active; invalid configs are rejected. The unit of `fixed` `value` (ringgit or cents) is not stated. Dashboard rules: Payout > Split Rules > New Rules, then Interface > Show Interface > Edit Split Settlement.

## Certificate (RSA / PQC keys)

Each interface can have cryptographic keys: generate a new key pair or import a public key (PEM). Algorithms: RSA (production-ready) and PQC KAZ-SIGN (experimental). Private key is shown once. PQC flow: generate KAZ-SIGN key pair, generate CSR, submit to SecurePay CA, optionally verify CA signature, submit certificate to SecurePay. The docs say "all API requests sent to SecurePay are cryptographically signed", but the signature header, string-to-sign and when it is mandatory are not documented (checked 2026-10-21). Ask SecurePay before enabling.

## Memberships and Closed Wallet (merchant)

All require `auth_token` (Bearer).

| Method | Path | Fields |
| --- | --- | --- |
| POST | `/v1/memberships` | Create membership (fields not documented) |
| GET | `/v1/memberships` | Returns `account`, `uuid`, `created_at`, `updated_at`, `tiers_count`, `members_count` |
| POST | `/v1/memberships/:membership_uuid/members` | `email` (existing user, required), `membership_number`, `card_number`, `pincode` (6 digits, required) |
| GET | `/v1/memberships/:membership_uuid/members` | List members |
| GET | `/v1/memberships/:membership_uuid/members/:membership_number` | Member info |
| PUT | `/v1/memberships/:membership_uuid/members/:membership_number` | `status` (`active`/`inactive`), `card_number` |
| GET | `/v1/memberships/:membership_uuid/members/:membership_number/transactions` | Member transactions |
| POST | `/v1/memberships/:membership_uuid/members/:membership_number` | Reload credit: `amount` (required), `redirect` |
| GET | `/v1/outlets/:uuid` | Outlet info |
| GET | `/v1/wallets` | `ic_number` param; wallet credentials Basic `ic_number:phone_number`. Returns `merchant_name`, `balance.cents`, `balance.currency_iso`, `uuid`, `status`, `membership_number`, `card_number`, `points`, `joined_at`, `created_at`, `updated_at` |
| GET | `/v1/wallets/:membership_number` | Wallet info (adds `tier`) |
| GET | `/v1/wallets/:membership_number/transactions` | `uuid`, `amount.cents`, `amount.currency_iso`, `status`, `description`, `created_at`, `merchant_name` |
| POST | `/v1/wallets/:membership_number/reload_wallet` | `membership_number`, `amount`; with `redirect=true` the payment page is shown |

Cashier Info page is empty in the docs.

## Partner API

Partner credentials: Token and Secret (64 hex each), multiple allowed. How to send them (header names or Basic) is not documented (checked 2026-10-21).

| Method | Path | Fields |
| --- | --- | --- |
| POST | `/partner/users` | `fullname`, `username`, `mobile_number`, `email`, `password`, `id_number` (all required), `bank_attributes` {`code`, `account_number`, `account_holder_name`, `branch`} optional |
| PUT | `/partner/users/:email` | `email` (required), `fullname`, `mobile_number`, `id_number` |
| GET | `/partner/users/:email` | Fetch user |
| POST | `/partner/accounts` | `user_uuid` (required), `name`, `identity_number`; type `Personal` or `Orgnisation` (sic) |
| PUT | `/partner/accounts/:uuid` | `name`, `identity_number` |
| GET | `/partner/accounts/:uuid` | Fetch account |

## Virtual Accounts and Credit (dashboard concepts)

- Virtual account types: Main (settlement), Credit, Default (virtual), Commission. Transfer Mode `Auto` or `Manual`. Negative settlement amounts are held.
- Credit Account: prepaid balance for transaction, DirectDebit and settlement processing fees when Charge Mode is Credit. Keep a buffer; reload in the Merchant Portal.
