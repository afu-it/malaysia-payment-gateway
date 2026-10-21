# SecurePay API Reference (entry point, API v1)

Use this reference for factual SecurePay guidance. It covers API v1 in full and catalogues every v1 and v2 endpoint. API v2 detail lives in `securepay-v2.md`; codes and enums in `securepay-codes.md`. All facts were checked against official SecurePay sources on 2026-10-21.

## Source URLs

- Docs hub: `https://www.securepay.my/docs/`
- v1 docs index: `https://docs.securepay.my/llms.txt`
- v1 introduction: `https://docs.securepay.my/api/introduction`
- v1 validate: `https://docs.securepay.my/api/validate`
- v1 merchant bank list: `https://docs.securepay.my/api/merchant/bank-list`
- v1 merchant FPX payment: `https://docs.securepay.my/api/merchant/payment`
- v1 partner FPX payment: `https://docs.securepay.my/api/partner/payment`
- v1 partner bank list: `https://docs.securepay.my/api/partner/bank-list`
- v1 Payment Session (beta): `https://docs.securepay.my/api/payment-session-beta/introduction`, `https://docs.securepay.my/api/payment-session-beta/payment`
- v1 callback and redirect URL: `https://docs.securepay.my/api/url`
- v1 status: `https://docs.securepay.my/api/transactions/transactions-status`
- v1 list: `https://docs.securepay.my/api/transactions/list`
- v1 search: `https://docs.securepay.my/api/transactions/search`
- v1 change logs: `https://docs.securepay.my/change-logs`
- Official WooCommerce plugin source: `https://wordpress.org/plugins/securepay/` (v1.0.18)

## Which Version

The docs hub (`https://www.securepay.my/docs/`) says:

- API v1 is "Recommended", "the main SecurePay API, in production today. Use this for live integrations."
- API v2 is "New (preview)", "available for teams that want to adopt the new version. Still gaining adoption, use v1 for production today."
- "Sandbox first. Test against the sandbox environment before switching your API key to production."

v1 and v2 use different hosts, credentials and payload shapes. Do not mix them.

## Environments (v1)

| Environment | Base URL |
| --- | --- |
| Sandbox | `https://sandbox.securepay.my/api` |
| Production | `https://securepay.my/api` |

Payment Session (beta) uses the same hosts with the path `/apis/payments`.

Observed 2026-10-21: the sandbox public B2C bank list returned `"mode":"production"` with no test banks, while the sandbox B2B list returned `"mode":"staging"` with test banks `TEST0021` to `TEST0023`. Do not use the `mode` field to detect environment; use your own config.

## Credentials (v1)

Generated per "interface" (API) in SecurePay Apps, for example `https://sandbox.securepay.my/apps/interfaces`. A merchant can generate multiple credential sets.

| Credential | Format example (docs) | Used as |
| --- | --- | --- |
| API UID | UUID, e.g. `2aaa1633-e63f-...` | Basic auth username; last segment of the request checksum string |
| Auth Token | 20 chars, e.g. `ZyUf...` | Basic auth password; `token` field in payment POST |
| Checksum Token | 64 hex chars | HMAC-SHA256 key for request and callback checksums |

Recommended env vars: `SECUREPAY_ENV` (`sandbox`/`production`), `SECUREPAY_BASE_URL`, `SECUREPAY_UID`, `SECUREPAY_AUTH_TOKEN`, `SECUREPAY_CHECKSUM_TOKEN`, optional `SECUREPAY_PARTNER_UID`.

Partners: "Partner may have many merchants. The uid and token are merchant credentials, partner only need to supply their partner_uid." A partner is issued one or more `partner_uid` values.

## Endpoint Catalogue

### API v1 (base `https://securepay.my/api` or sandbox)

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| POST | `/v1/merchants/validate` | Basic uid:auth_token + `checksum_token` field | Validate credentials |
| GET | `/public/v1/banks/b2c` | none | FPX retail (B2C) bank list; `?status=online` or `?status=offline` |
| GET | `/public/v1/banks/b2b` | none | FPX corporate (B2B) bank list; same filter |
| GET | `/v1/banks/b2c` | Basic | Partner FPX B2C bank list |
| GET | `/v1/banks/b2b1` | Basic | Partner FPX B2B1 bank list (merchant must enable the module) |
| POST | `/v1/payments` | `token` + `checksum` fields | Create FPX payment (merchant or partner) |
| POST | `/apis/payments` | Basic uid:auth_token | Create Payment Session (beta), returns JSON `payment_url` |
| GET | `/apis/{slug}/payment_session` | none (customer page) | Hosted Payment Session form (from `payment_url`) |
| GET | `/v1/payments/status/:order_number` | Basic | Status by merchant order number |
| GET | `/v1/payments/status/merchant_reference_number/:merchantrefnumber` | Basic | Status by SecurePay merchant reference number |
| GET | `/v1/transactions/list` | Basic | Paginated transaction list |
| GET | `/v1/transactions/search` | Basic | Search by FPX ref, order number or exchange number |
| GET | `/v1/payments/list` | Basic | Deprecated, "no longer available" |
| GET | `/v1/status/:exchange_number?uid=...` | link | SecurePay status page (`status_url`) |
| GET | `/v1/receipt/:exchange_number.pdf?uid=...` | link | PDF receipt (`receipt_url`) |
| GET | `/v1/retry/:exchange_number?uid=...` | link | Retry a failed payment (`retry_url`) |
| POST | your `callback_url` | checksum | Server-to-server payment status |
| POST | your `redirect_url` | checksum | Browser return with payment status |

### API v2 (base `https://console.securepay.my/api`, staging `https://sandbox.securepay.dev/api`)

Full detail in `securepay-v2.md`.

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/v1/auth` | Exchange client ID/secret (Basic) for `auth_token` |
| GET | `/v1/paynet/:gateway_name/banks[/:bank_type]` | Bank list for `fpx`, `direct_debit`, `duitnow` |
| GET | `/v1/interface/gateways` | Payment methods enabled on the interface |
| POST | `/v1/payment/intents` | Create payment session (FPX, DuitNow) or eMandate (`DirectDebit`) |
| POST | `/v1/payment/intent` | Closed Wallet payment (path is singular in docs) |
| GET | `/v1/payment/status/:identifier[.json]` | Payment record status by order or reference number |
| GET | `/v1/transactions/list` | Payment records list |
| GET | `/v1/payment/mandates/status/:identifier` | eMandate payment record status |
| GET | `/v1/mandates/status/:reference_number[.json]` | Mandate object status |
| GET | `/v1/payment/mandates/transactions/list` | Mandate payment records list |
| GET | `/v1/mandates/transactions/list` | Mandates list |
| POST, GET | `/v1/memberships` | Create / get membership |
| POST, GET | `/v1/memberships/:membership_uuid/members` | Add / list members |
| GET, PUT | `/v1/memberships/:membership_uuid/members/:membership_number` | Member info / update |
| POST | `/v1/memberships/:membership_uuid/members/:membership_number` | Reload member credit |
| GET | `/v1/memberships/:membership_uuid/members/:membership_number/transactions` | Member transactions |
| GET | `/v1/outlets/:uuid` | Outlet info |
| GET | `/v1/wallets` | List closed wallets (`ic_number`) |
| GET | `/v1/wallets/:membership_number` | Wallet info |
| GET | `/v1/wallets/:membership_number/transactions` | Wallet transactions |
| POST | `/v1/wallets/:membership_number/reload_wallet` | Reload wallet |
| POST | `/partner/users` | Partner: onboard user |
| PUT, GET | `/partner/users/:email` | Partner: update / fetch user |
| POST | `/partner/accounts` | Partner: create account |
| PUT, GET | `/partner/accounts/:uuid` | Partner: update / fetch account |

Not documented in official docs (checked 2026-10-21): refund, void, capture, cancel, payout/disbursement API, settlement report API, card tokenisation, webhook registration API. Settlement statements are downloaded from the dashboard.

## Validate Credentials (v1)

`POST /v1/merchants/validate`

```bash
curl -d checksum_token=<CHECKSUM_TOKEN> -u <API_UID>:<AUTH_TOKEN> \
  https://sandbox.securepay.my/api/v1/merchants/validate
```

- 200: `{"credential":true,"api_name":"<interface name>","merchant_name":"<merchant>"}`
- 401: `{"error":"Unauthorized. Invalid or expired token."}`

Use it as a deploy-time health check.

## FPX Bank List (v1)

Public, no auth: `GET /public/v1/banks/b2c`, `GET /public/v1/banks/b2b`, optional `?status=online` or `?status=offline`.

Partner, Basic auth: `GET /v1/banks/b2c`, `GET /v1/banks/b2b1`.

Response:

```json
{"VerifySign":true,"category":"fpx","model":"B2C","mode":"production",
 "fpx_bankList":[{"name":"Affin Bank","code":"ABB0233","status_format1":"online","status_format2":true}]}
```

| Field | Values | Meaning |
| --- | --- | --- |
| `VerifySign` | true/false | "Verify data source only accept true value. false might be bogus, please discard." |
| `category` | `fpx` | Default fpx |
| `model` | `B2C`, `B2B1` | Bank model |
| `mode` | `staging`, `production` | Current mode (see environment note) |
| `fpx_bankList[].name` | string | Bank display name |
| `fpx_bankList[].code` | string | Value for `buyer_bank_code` |
| `fpx_bankList[].status_format1` | `online`/`offline` | Bank availability |
| `fpx_bankList[].status_format2` | true/false | Same, boolean |

Show offline banks as disabled (the official plugin and the docs PHP sample do this). Cache briefly; the plugin caches in a transient. Codes: `securepay-codes.md`.

## Create FPX Payment (v1)

`POST /v1/payments` (sandbox `https://sandbox.securepay.my/api/v1/payments`). Same endpoint for merchant and partner. Send as `application/x-www-form-urlencoded` ("Sending all parameters in request parameter table to the payment URL by using post method").

### Request fields

| Field | Required | Notes |
| --- | --- | --- |
| `order_number` | Yes | Unique merchant order number, e.g. `20200425132755` |
| `buyer_name` | Yes | Full name in one line |
| `buyer_email` | Yes | Valid email, used for status updates |
| `buyer_phone` | Yes | e.g. `+60123121678` |
| `transaction_amount` | Yes | Ringgit, two decimals: `100.20`, `1000.00` |
| `product_description` | Yes | e.g. `Payment for order no. 20200425132755` |
| `checksum` | Yes | HMAC-SHA256 hex, see below |
| `token` | Yes | Auth Token |
| `callback_url` | No | Server-to-server status POST |
| `redirect_url` | No | Browser POST after payment; default is SecurePay status page |
| `redirect_post` | No | `true` = auto redirect to endpoint page |
| `params` | No | Up to 18 custom pairs `reference1_label`/`reference1` ... `reference18_label`/`reference18`; echoed back in callbacks |
| `shipping_address` | No | Object: `contact_name`, `contact_phone_number`, `line1`, `line2`, `postcode`, `city`, `state` |
| `billing_address` | No | Same shape as `shipping_address` |
| `buyer_bank_code` | No | Bank `code` from the bank list; skips SecurePay bank selection |
| `model` | No | `B2C` (default) or `B2B1`; bank code must match the model |
| `fpx_bank_selection` | No | `dropdown` or `grid` for SecurePay's bank selection page |
| `cancel_url` | No | Shows a Cancel button on SecurePay bank selection page |
| `timeout_url` | No | Bank selection page times out within 3 minutes and goes here |
| `partner_uid` | Partner only | Required on the partner page; excluded from the checksum |
| `uid` | See note | Listed as Compulsory only on the partner page |

Notes:

- `uid` field: the merchant page does not list `uid` as a POST field and neither the docs PHP sample nor the official WooCommerce plugin sends it; the partner page lists it as Compulsory. `uid` is always part of the checksum string. Confirm in sandbox whether to send it.
- The official plugin also sends an undocumented `payment_source` (`woocommerce` or `woocommerce_bank_list`). Do not rely on it.
- B2C or B2B1, and other defaults, can also be set on the API settings page in SecurePay Apps.
- The docs do not document the HTTP response of `POST /v1/payments`. The official plugin renders a hidden HTML form and auto-submits it from the customer's browser to `/v1/payments`, so the customer lands on SecurePay's bank selection or the bank page. The docs PHP sample posts with cURL server-side and echoes the returned page. Prefer the browser form POST.

### Request checksum

String to sign (fixed order, ascending key names, including API UID, excluding `partner_uid`):

```text
buyer_email|buyer_name|buyer_phone|callback_url|order_number|product_description|redirect_url|transaction_amount|uid
```

Use the exact values you POST. An empty optional field keeps its empty slot (`||`). Sign with HMAC-SHA256, key = Checksum Token, lowercase hex output.

```text
string   = buyer_email + "|" + buyer_name + "|" + buyer_phone + "|" + callback_url + "|" +
           order_number + "|" + product_description + "|" + redirect_url + "|" +
           transaction_amount + "|" + uid
checksum = hex(HMAC_SHA256(key = CHECKSUM_TOKEN, message = string))
```

```php
$sign = hash_hmac('sha256', $string, $checksum_token);
```

```ruby
OpenSSL::HMAC.hexdigest(OpenSSL::Digest.new('sha256'), checksum_token, string)
```

Documented test vector (docs example values, not real credentials; verified locally with HMAC-SHA256):

```text
key     = 159026b3b7348e2390e5a2e7a1c8466073db239c1e6800b8c27e36946b1f8713
string  = amsyar@gmail.com|AHMAD AMSYAR MOHD ALI|+60123121678||20200425132755|Payment for order no 20200425132755||1540.40|2aaa1633-e63f-4371-9b85-91d936aa56a1
result  = 75b54e403151b1e9b413df8ce5c426ef0dbbc9adcec58b8f5dd5c9c5c6b78844
```

Gotchas:

- The docs code samples contain invisible U+200B characters inside strings. The partner page also shows a different result (`5475e0...`) caused by those characters. Trust `75b54e40...`.
- `transaction_amount` in the string must be byte-identical to the POSTed value (`1540.40`, not `1540.4`).
- The official plugin passes the same URL as both `callback_url` and `redirect_url` and signs both slots with it.

### Minimal server flow (pseudo-code)

```text
order = db.create(order_number=unique(), amount="25.00", status="pending")
fields = {order_number, buyer_name, buyer_email, buyer_phone,
          transaction_amount: order.amount, product_description,
          callback_url: CALLBACK, redirect_url: RETURN, redirect_post: "true",
          token: AUTH_TOKEN}
fields.checksum = hmac_sha256_hex(CHECKSUM_TOKEN, join("|", [buyer_email, buyer_name, buyer_phone,
          callback_url, order_number, product_description, redirect_url, transaction_amount, UID]))
render auto-submitting <form method="post" action="{BASE}/v1/payments"> with fields
```

## Payment Session (beta, v1)

`POST /apis/payments` (sandbox `https://sandbox.securepay.my/api/apis/payments`), HTTP Basic `uid:auth_token`, form-encoded. "The only compulsory parameter is the transaction_amount". No request checksum is documented for this endpoint.

| Field | Required | Notes |
| --- | --- | --- |
| `transaction_amount` | Yes | `100.20` format |
| `order_number` | No | Recommended for tracking |
| `buyer_name`, `buyer_email`, `buyer_phone` | No | Prefill |
| `product_description` | No | Default `Payment for order no. <order_number>` |
| `callback_url`, `redirect_url` | No | As v1 payment |
| `cancel_url`, `timeout_url` | No | As v1 payment |
| `partner_uid` | No | Partner UID |
| `currency` | No | Only `USD` supported; converted to MYR at current rate |

```bash
curl -X POST https://sandbox.securepay.my/api/apis/payments -u <API_UID>:<AUTH_TOKEN> \
  -d 'transaction_amount=200&order_number=231212&cancel_url=https://example.com/cancel'
```

Response (JSON):

```json
{"payment_url":"https://sandbox.securepay.my/api/apis/GSHJD1664332374/payment_session",
 "data":{"buyer_name":null,"buyer_email":null,"buyer_phone":null,"transaction_amount":"200.00",
  "order_number":"231212","product_description":"Payment for order no. 231212","slug":"GSHJD1664332374",
  "callback_url":null,"redirect_url":null,"cancel_url":"https://example.com/cancel","timeout_url":null,
  "currency":"MYR","currency_amount":"200.00","partner_uid":null,"other_params":null,
  "created_at":"2022-09-28T10:32:54.087+08:00","updated_at":"2022-09-28T10:32:54.087+08:00"}}
```

With `currency=USD` and `transaction_amount=200`, the docs show `transaction_amount":"922.15"` (MYR), `currency":"USD"`, `currency_amount":"200.00"`. Expect the callback `transaction_amount` in MYR; store both.

Redirect the customer to `payment_url`.

## Redirect and Callback (v1)

- `redirect_url`: browser to browser. After the bank step the customer is redirected and "Information will be posted (POST method) to the URL". If nil, the customer sees SecurePay's status page.
- `callback_url`: server to server. "SecurePay server sends the transaction status to the merchant callback server... It's very important, if error occur on the redirect URL."
- Both carry the same fields, POSTed form-encoded.
- Callback retry policy, expected HTTP response and source IPs: not documented in official docs (checked 2026-10-21). Return HTTP 200 quickly after durable processing.

### Callback/redirect fields

| Field | Meaning |
| --- | --- |
| `source` | `FPX` or Credit Card |
| `payment_status` | `true` or `false` |
| `currency` | e.g. `MYR` |
| `transaction_amount` | Transaction amount (the Ruby example names it `amount`; accept either) |
| `transaction_amount_received` | Amount received; `0.00` if `payment_status` false |
| `order_number` | Your order number |
| `merchant_reference_number` | SecurePay generated |
| `exchange_number` | SecurePay generated |
| `buyer_name`, `buyer_phone`, `buyer_email` | Buyer |
| `payment_id` | Reference |
| `client_ip` | Customer IP |
| `status_url`, `retry_url`, `receipt_url` | SecurePay links |
| `checksum` | SecurePay's HMAC for verification |
| `transaction_fee` | Fee amount |
| `payment_mode` | `merchant` (deduct from amount), `customer` (customer pays fee), `credit` (deduct from credit account) |
| `payment_method` | e.g. `fpx_online_banking`, FPX credit card |
| `fpx_model` | `B2C` or `B2B1` |
| `fpx_debit_auth_code` | `00` success, `99` pending approval (B2B1), other = unsuccessful |
| `params` | Your `params` echoed back |

Additional keys seen in the official example payload: `fpx_name`, `fpx_bank`, `fpx_transaction_id`, `fpx_status`, `fpx_status_message`, `fpx_transaction_time`, `payment_stage`, `merchant_name`, `merchant_co_number`, `uid`, `created_at`, `created_at_unixtime`, `token`. SecurePay says "Params will keep updating time to time."

B2B1: "there will be code 99, after approval the server will send callback with code 00". Keep the order pending on `99`.

### Callback checksum verification

Official rule: "Compulsory to use function that automatically sort the key and later replace with the value." Steps:

1. Take all received POST parameters.
2. Remove `checksum`.
3. Sort by key name ascending.
4. Join the values with `|` (null/absent values become empty strings).
5. HMAC-SHA256 hex with the Checksum Token.
6. Constant-time compare with the received `checksum`.

Official PHP (from docs):

```php
$data = $request->all();
ksort($data);
$checksum = $data['checksum'];
unset($data['checksum']);
$string = implode('|', $data);
$sign = hash_hmac('sha256', $string, $this->checksum_token);
return ($sign == $checksum); // use hash_equals() in production
```

Hardening that keeps the documented algorithm:

- Read the raw form body and keep values as received strings; do not cast booleans or reformat dates.
- Exclude framework-injected keys (for example a route param or `wc-api` query param) that SecurePay did not send. Verify over POST body keys only.
- If `params` arrives as nested form keys (`params[...]`), behaviour is not documented; test in sandbox.

Documented test vector (docs example values; verified locally):

```text
key    = df8aa0d55308ee762d2a5ce675cb9cfb4f85a59e41723573010e9c3f99f5457b
sorted = amount|buyer_email|buyer_name|buyer_phone|client_ip|created_at|created_at_unixtime|currency|exchange_number|fpx_bank|fpx_debit_auth_code|fpx_model|fpx_name|fpx_status|fpx_status_message|fpx_transaction_id|fpx_transaction_time|merchant_co_number|merchant_name|merchant_reference_number|order_number|params|payment_id|payment_method|payment_mode|payment_stage|payment_status|receipt_url|retry_url|source|status_url|token|transaction_fee|uid
string = 25.64|khairi@localhost.my|MOHD KHAIRI MOHD ADNAN|+600139803112|60.53.196.69|2021-06-14 17:33:48 +0800|1623663228|MYR|MHQWK1623663213|SBI BANK A|00|B2C|N@ME()/ .-&BUYER|true|Transaction Approved|2106141733330972|20210614173333|x111222122y|Muslimah Next Store||20210614173332470||5747|fpx_online_banking|customer|payment_successful|true|https://sandbox.securepay.my/api/v1/receipt/MHQWK1623663213.pdf?uid=c111e3bc3c9c22d572dc||FPX|https://sandbox.securepay.my/api/v1/status/MHQWK1623663213?uid=c111e3bc3c9c22d572dc|6mtetdxEvzj6KJ9htyTy|1.50|c111e3bc3c9c22d572dc
result = 41712f0c155d1799c08e13add760de00b6a39e5a340360569253c360c2a40bbf
```

The official WooCommerce plugin (1.0.18) does not verify this checksum; it trusts `payment_status`/`fpx_status`. Do not copy that.

### Settlement decision

Mark paid only when all hold:

- checksum valid (callback) and status requery agrees;
- `payment_status` is `true` (string `"true"` in form posts);
- `fpx_debit_auth_code` is `00` when present;
- `transaction_amount_received` equals the expected amount and `currency` is `MYR`;
- `order_number` maps to a pending local order for this environment and interface;
- the order is not already settled (idempotent).

## Payment Status (v1)

| Lookup | Path |
| --- | --- |
| By your order number | `GET /v1/payments/status/:order_number` |
| By SecurePay merchant reference number | `GET /v1/payments/status/merchant_reference_number/:merchantrefnumber` |

```bash
curl -u <API_UID>:<AUTH_TOKEN> https://sandbox.securepay.my/api/v1/payments/status/20200927045002577
```

Response fields: `source`, `payment_status` (true success / false failed), `currency`, `transaction_amount` (RM), `transaction_amount_received`, `order_number`, `merchant_reference_number`, `exchange_number`, `buyer_name`, `buyer_phone`, `buyer_email`, `interface_name` (API name), `interface_uid` (API UID), `payment_id`, `client_ip`, `status_url`, `retry_url`, `receipt_url`, `created_at` (ISO8601), `created_at_unixtime`.

Behaviour when one `order_number` has several attempts is not documented; keep order numbers unique per attempt.

## Transactions List (v1)

`GET /v1/transactions/list`, Basic auth.

| Param | Notes |
| --- | --- |
| `per_page` | Default 10, max 100 |
| `page` | Page number |
| `payment_status` | `true` (default), `false`, `all` |
| `start_date`, `end_date` | `YYYY-MM-DD`; both required for a range; same date for one day |

Limits: data search limit 3 months (90 days). Response: `next_page`, `current_page`, `per_page`, `total_page`, `prev_page`, `total_entries`, `transactions[]`. Each transaction adds to the status fields: `fpx_debit_auth_code`, `payment_stage`, `fpx_model`, `merchant_name`, `merchant_co_number`, `fpx_transaction_time`, `payment_method`, `payment_mode` (`merchant`, `buyer`/`customer`, `merchant_credit`), `fpx_bank`, `fpx_name`, `fpx_transaction_id`, `fpx_status`, `fpx_status_message`, `transaction_fee`, `uid`.

Deprecated: `GET /v1/payments/list` ("no longer available").

## Transactions Search (v1)

`GET /v1/transactions/search?search_reference=<type>&reference_number=<value>`, Basic auth. Any credential set of the merchant works.

| `search_reference` | Searches |
| --- | --- |
| `fpx_ref_number` | FPX reference number |
| `order_number` | Your order number |
| `exchange_number` | SecurePay exchange number |

Returns a JSON array of transactions (same fields as list). Example failed record: `fpx_debit_auth_code":"76"`, `fpx_status_message":"Transaction Not Found"`, `payment_stage":"payment_unsuccessful"`, `retry_url` set.

## Errors

| Case | Response |
| --- | --- |
| v1 invalid/expired credentials | 401 `{"error":"Unauthorized. Invalid or expired token."}` |
| v2 Basic auth failure | `HTTP Basic: Access denied.` (docs); live 401 `{"error":"Access denied. Please check your credentials and try again.","code":401}` (observed) |
| v2 bad bearer | 401 `{"error":{"status":401,"message":"Unauthorized"}}` |
| v2 record not found | 404 `{"error":{"status":404,"message":"Record not found"}}` |
| v2 empty list | `{"msg":"No payment records found!"}` |
| v2 bad gateway name | `{"errors":"Invalid gateway name"}` |

A full v1 error code catalogue is not documented. FPX debit auth codes: `securepay-codes.md`.

## Refunds

No refund API is documented. SecurePay's Refund & Cancellation Policy (`https://www.securepay.my/refund/`) handles refunds manually: apply to `support@securepay.my` with the transaction number and receipt; processed within 7 working days of a valid claim, net of gateway charges. Track refunds as a manual state in your system.

## Recurring

v1 has no recurring API. v2 offers eMandate (FPX direct debit) through `POST /v1/payment/intents` with `payment_method=DirectDebit`; see `securepay-v2.md`.

## Sandbox

- Sandbox host `https://sandbox.securepay.my` (v1), dashboard at `https://sandbox.securepay.my/apps/`.
- FPX sandbox test banks in the docs: `SBI Bank A` (`TEST0021`), `SBI Bank B` (`TEST0022`), `SBI Bank C` (`TEST0023`). On 2026-10-21 the public sandbox B2B list included them but the public sandbox B2C list did not, so use whatever the sandbox bank selection page shows.
- Test cards and test bank login values: not documented in official docs (checked 2026-10-21).

## Rate Limits, IPs, Timeouts

- Rate limits: not documented.
- Callback source IPs / allowlist: not documented.
- Bank selection page with `timeout_url` times out within 3 minutes.
- List API search window: 90 days.
