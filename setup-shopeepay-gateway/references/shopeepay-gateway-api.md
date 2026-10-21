# ShopeePay Merchant API Reference

Entry point for direct ShopeePay merchant integration (ShopeePay Malaysia Sdn Bhd). Distilled from the official docs at `https://product.shopeepay.com` (checked 2026-10-21) and the official "Checkout with ShopeePay" WooCommerce plugin 1.0.0 source (published by shopeepay on wordpress.org, linked from the official setup guideline). The docs cover six markets; this file keeps Malaysia values and notes other markets only where they change behaviour.

Related files:

- `shopeepay-gateway-hosted-gateway.md`: ShopeePay Payment Gateway (`/v1/checkout`) on the `api.gw.*` hosts.
- `shopeepay-gateway-tokenized.md`: Account Linking, Link & Pay, Subscription, Auth & Capture.

## Source URLs

- `https://product.shopeepay.com/integration/get-started`
- `https://product.shopeepay.com/integration/api/checkout-with-shopeepay`
- `https://product.shopeepay.com/integration/api/merchant-presented-mode`
- `https://product.shopeepay.com/integration/api/customer-presented-mode`
- `https://product.shopeepay.com/integration/api/invalidate-qr`
- `https://product.shopeepay.com/integration/api/invalidate-order`
- `https://product.shopeepay.com/integration/api/refund-a-payment`
- `https://product.shopeepay.com/integration/api/notify-transaction-status`
- `https://product.shopeepay.com/integration/api/check-transaction-status`
- `https://product.shopeepay.com/integration/common-objects`
- `https://wordpress.org/plugins/checkout-with-shopeepay/` (official plugin source)

## Products

| Product | Channel | Main call | Reference |
| --- | --- | --- | --- |
| Checkout with ShopeePay | Online (app, mweb, pc) redirect / jump app | `/v3/merchant-host/order/create` | this file |
| ShopeePay Payment Gateway | Online hosted page, multiple methods | `/v1/checkout` (gw host) | hosted-gateway file |
| Merchant Presented Mode (MPM) | In-person or screen QR, customer scans | `/v3/merchant-host/qr/create` | this file |
| Customer Presented Mode (CPM) | In-person, merchant scans customer code | `/v3/merchant-host/transaction/payment/create` | this file |
| Account Linking + Link & Pay | Tokenized one-click | `/v3/merchant-host/account/link`, `/v3/merchant-host/transaction/payment/direct` | tokenized file |
| Subscription | Tokenized recurring | `/v3/merchant-host/transaction/payment/direct` | tokenized file |
| Auth & Capture | Tokenized hold then capture | `/v3/merchant-host/transaction/auth/*` | tokenized file |

Static MPM QR codes are issued by ShopeePay offline or through the Shopee Partner app; no API creates them (product page, merchant-presented-mode).

## Environments

Access nodes are per country; Malaysia region code is `com.my`.

| Environment | Wallet API host (Malaysia) |
| --- | --- |
| Staging (UAT) | `https://api.uat.wallet.airpay.com.my` |
| Production | `https://api.wallet.airpay.com.my` |

Pattern: `api.uat.wallet.airpay.<region code>` / `api.wallet.airpay.<region code>`. Region codes: ID `co.id`, MY `com.my`, PH `com.ph`, SG `sg`, TH `co.th`, VN `vn`. The official plugin builds exactly `https://api.uat.wallet.airpay.com.my` and `https://api.wallet.airpay.com.my`.

The hosted ShopeePay Payment Gateway uses different hosts (`api.gw.uat.airpay.com.my` / `api.gw.airpay.com.my`); see that file.

## Protocol Rules

| Component | Rule |
| --- | --- |
| Transfer | HTTPS, TLS 1.2 or 1.3 |
| Method | `POST` for all wallet APIs |
| Body | JSON, UTF-8 |
| Dates | UNIX seconds |
| Signature | HMAC, SHA-256, Base64 |
| SDK | None; integrate directly with ShopeePay endpoints |

Response handling rules from the docs:

- Fields may come in any order; new fields may be added without notice; do not assume field count.
- Empty values: integer `0`, string `""` (or `"0"` for numeric strings like amount/timestamp), object `null`, array `[]`, boolean `false`.
- URL query parameters may come in any order, may gain new keys, and are case-sensitive.
- Non-breaking changes ShopeePay may ship without notice: new endpoints or callback types, adding or removing optional request fields, new response fields, changed max length of response fields, field order changes, increased max length of request fields.

## Authentication And Signature

Headers on every request:

```text
Content-Type: application/json
X-Airpay-ClientId: <CLIENT_ID>
X-Airpay-Req-H: <BASE64_HMAC_SHA256_OF_BODY>
```

Algorithm (Get Started, Security Specifications):

```text
signature = Base64( HMAC_SHA256( key = SECRET_KEY, message = exact_request_body_bytes ) )
```

- Sign the exact JSON string you put on the wire. Serialize once, sign, send the same bytes.
- Docs JS sample strips trailing newlines: `CryptoJS.enc.Base64.stringify(CryptoJS.HmacSHA256(body, secret)).replace(/\n+$/, '')`.
- Response validation: sign the response body with the same secret and compare with "the signature in the header of the API response"; if different, do not trust the response. The response header name is not stated in the docs (checked 2026-10-21); confirm with ShopeePay (likely the same `X-Airpay-Req-H`, unverified). The official plugin does not verify response signatures.

Documented test vector (published sample secret from the docs, not a real key):

```text
secret: pz148x0gXyPCLHxnlhEydNLg55jni91i
body:   {"request_id": "","store_ext_id": "externalstore","merchant_ext_id": "externalmerchant","amount": 1000,"terminal_id": "terminal","convenience_fee_percentage": 0,"convenience_fee_fixed": 0,"convenience_fee_indicator": "","additional_info": "","currency": "IDR","qr_validity_period": 1000,"payment_reference_id": "testreference"}
HMAC hex: 5f5532bf23018674788770f43743522dff1af5782243fab06b1e8677a7391d4c
Base64:   X1UyvyMBhnR4h3D0N0NSLf8a9XgiQ/qwax6Gd6c5HUw=
```

(Re-computed and confirmed 2026-10-21. The docs label the hex step "Hash the JSON content by using the SHA256 algorithm"; it is the HMAC-SHA256 digest, not plain SHA-256.)

Pseudo-code:

```js
import crypto from "node:crypto";
const body = JSON.stringify(payload);            // serialize once
const sig = crypto.createHmac("sha256", process.env.SHOPEEPAY_SECRET_KEY)
                  .update(body, "utf8").digest("base64");
await fetch(`${HOST}/v3/merchant-host/order/create`, {
  method: "POST",
  headers: { "Content-Type": "application/json",
             "X-Airpay-ClientId": process.env.SHOPEEPAY_CLIENT_ID,
             "X-Airpay-Req-H": sig },
  body,                                          // same bytes that were signed
});
```

PHP equivalent from the official plugin: `base64_encode(hash_hmac('sha256', $json_body, $client_secret, true))`.

## Amount Rules

- `amount` is an integer "inflated by a factor of 100", smallest currency unit, no decimal point. RM 10.00 = `1000`. Smallest value `1` = RM 0.01.
- Currency `MYR` (ISO 4217). Docs list IDR, MYR, PHP, SGD, THB, VND.
- For IDR/VND the smallest amount is `100` (= 1 unit), i.e. still x100. The official plugin uses `round(total * 100)` for every market.

## Endpoint Catalogue

Wallet host (`api[.uat].wallet.airpay.com.my`), all `POST`:

| Path | Purpose | Reference |
| --- | --- | --- |
| `/v3/merchant-host/order/create` | Create Checkout Order (Checkout with ShopeePay) | this file |
| `/v3/merchant-host/order/invalidate` | Invalidate Checkout order | this file |
| `/v3/merchant-host/qr/create` | Create Dynamic QR for MPM | this file |
| `/v3/merchant-host/qr/invalidate` | Invalidate MPM QR | this file |
| `/v3/merchant-host/transaction/payment/create` | Create CPM Payment | this file |
| `/v3/merchant-host/transaction/refund/create-new` | Refund a payment | this file |
| `/v3/merchant-host/transaction/check` | Check Transaction Status (payment, capture, refund) | this file |
| `/v3/merchant-host/account/link` | Initiate Account Linking | tokenized |
| `/v3/merchant-host/access-token/get` | Get Access Token | tokenized |
| `/v3/merchant-host/account/unlink` | Account Unlinking | tokenized |
| `/v3/merchant-host/user-info/get` | Get User Info | tokenized |
| `/v3/merchant-host/coin/potential-redemption` | Get Coin Redemption Info | tokenized |
| `/v4/merchant-host/payment-methods/get` | Get Payment Methods (Link & Pay API-based) | tokenized |
| `/v3/merchant-host/transaction/payment/direct` | Direct Payment (Link & Pay, Subscription) | tokenized |
| `/v3/merchant-host/transaction/auth/create` | Create Auth | tokenized |
| `/v3/merchant-host/transaction/auth/capture` | Capture | tokenized |
| `/v3/merchant-host/transaction/auth/reverse` | Reverse Auth | tokenized |
| Merchant callback URL (registered at onboarding) | Notify Transaction Status | this file |
| Merchant callback URL (registered at onboarding) | Notify Account Link Status | tokenized |
| `/v3/merchant-host/qr/download?qr=...` | QR image download link returned as `qr_url` (valid 5 minutes) | this file |

Hosted gateway host (`api.gw[.uat].airpay.com.my`):

| Method and path | Purpose |
| --- | --- |
| `POST /v1/checkout` | Create Checkout Session |
| `GET /v1/checkout/{checkout_id}` | Get Checkout Status |
| `POST /v1/checkout/cancel/{checkout_id}` | Cancel Checkout |
| `POST /v1/refund` | Create Refund |
| `v1/refund/{refund_id}` (method not stated) | Get Refund Status |
| Merchant callback URL | Webhook events `checkout.*`, `refund.*` |

## Checkout With ShopeePay

Flow: server calls Create Checkout Order; redirect the customer to `redirect_url_http` (opens Shopee/ShopeePay app or ShopeePay web); customer pays; ShopeePay sends Notify Transaction Status; customer returns to `return_url`; merchant may requery with Check Transaction Status.

### Create Checkout Order

`POST /v3/merchant-host/order/create`

Request:

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Required | Unique per request, max 64 chars |
| `payment_reference_id` | string | Required | Merchant order reference, unique, max 64 chars |
| `merchant_ext_id` | string | Required | Merchant id in merchant system (issued at onboarding) |
| `store_ext_id` | string | Required | Store id in merchant system |
| `amount` | int64 | Required | x100 integer |
| `currency` | string | Required | `MYR` |
| `return_url` | string | Required | HTTP link or URL scheme; customer returns here after payment or cancel |
| `platform_type` | string | Required | `app`, `pc`, or `mweb` |
| `validity_period` | uint32 | Optional | Seconds; default 1200; Malaysia max 3600. Error if sent with `expiry_time` |
| `expiry_time` | uint32 | Optional | Unix timestamp; same limits; mutually exclusive with `validity_period` |
| `promo_ids` | string | Optional | Comma separated, up to 20, 100 chars or less |
| `phone` | string | Optional | Customer phone with country code, no plus (e.g. `60123456789`) |
| `preferred_payment_method_type` | string | Optional | `spp_wallet` (ShopeePay) or `spay_later` (SPayLater); `maribank_direct_debit` PH only |
| `additional_info` | JSON string | Optional | See below |

`additional_info` (JSON-encoded string): `field1`, `field2`, `field3` (payment remarks shown in the ShopeePay Clearing Order Report, for reconciliation); `order_detail.shipping_address` {`region`,`state`,`city`,`address`,`postal_code`}; `order_detail.item_info_list[]` {`id`,`name`,`quantity`,`currency`,`amount`,`category`,`brand`,`photo_url`}; `merchant_user_account_info.user_identifier_info` {`user_id`,`phone_number`,`email`,`username`}. All optional here.

Response:

| Field | Type | Notes |
| --- | --- | --- |
| `request_id` | string | Echo |
| `errcode` | int32 | 0 = success |
| `debug_msg` | string | Detail |
| `redirect_url_http` | string | Universal URL to the ShopeePay payment page. Do not apply domain whitelisting or IP restriction to it |
| `qr_content` | string | QR string (returned in some markets, e.g. PH sample) |
| `qr_url` | string | QR image URL, valid 5 minutes |

Errcodes: -2, -1 (server error; requery), 0, 1, 2, 4, 11, 15, 50, 627 (promo ID exceeded limit), 1800, 1908.

Return URL: the docs sample `redirect_url_http` carries a base64 `return_url` that decodes to the merchant URL with `amount`, `client_id`, `reference_id`, `result_code`, `signature` query keys. These return parameters are not documented as fields (checked 2026-10-21). Do not settle from them; requery or wait for the callback. The official plugin ignores them and calls Check Transaction Status on return.

### Invalidate Order

`POST /v3/merchant-host/order/invalidate`. Use when the merchant order is closed or cancelled; after success `redirect_url_http` can no longer be paid.

Request: `request_id` (Required), `payment_reference_id` (Required), `merchant_ext_id` (Required), `store_ext_id` (Required). Response: `request_id`, `errcode`, `debug_msg`. Errcodes: -2, -1, 0, 1, 2, 4, 1800, 1901 (cannot be invalidated).

## Merchant Presented Mode (Dynamic QR)

### Create Dynamic QR

`POST /v3/merchant-host/qr/create`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Required | Max 64 |
| `amount` | int64 | Required | x100 |
| `currency` | string | Required | `MYR` |
| `merchant_ext_id` | string | Required | |
| `store_ext_id` | string | Required | |
| `payment_reference_id` | string | Required | Max 64 (MY/SG/TH/VN); 25 for PH |
| `qr_validity_period` | uint32 | Optional | Seconds; default 1200; Malaysia max 3600. Not with `expiry_time` |
| `expiry_time` | uint32 | Optional | Unix timestamp; same limits |
| `terminal_id` | string | Optional | Store terminal id, returned on the transaction |
| `convenience_fee_indicator` | string | Optional | `01` customer inputs fee, `02` fixed (`convenience_fee_fixed`), `03` percentage (`convenience_fee_percentage`). ID and MY only |
| `convenience_fee_fixed` | int64 | Conditional | Required when indicator `02`. MY only |
| `convenience_fee_percentage` | uint32 | Conditional | Required when indicator `03`. MY only |
| `promo_ids` | string | Optional | As above |
| `additional_info` | JSON string | Optional | Same structure as Checkout |
| `preferred_payment_method_type` | string | Optional | `spp_wallet` or `spay_later` |

Response: `request_id`, `errcode`, `debug_msg`, `store_name`, `qr_content` (plain QR string to render), `qr_url` (image URL, valid 5 minutes).

Errcodes: -2, -1 (requery), 0, 1, 2, 4, 11, 15, 16 (amount below limit), 50, 627, 1800, 1908.

### Invalidate QR

`POST /v3/merchant-host/qr/invalidate`. Same request and response fields as Invalidate Order (`request_id`, `payment_reference_id`, `merchant_ext_id`, `store_ext_id`). Errcodes: -2, -1, 0, 1, 2, 4, 1800, 1901 (QR cannot be invalidated).

## Customer Presented Mode

### Create CPM Payment

`POST /v3/merchant-host/transaction/payment/create`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Required | Max 64 |
| `amount` | uint64 | Required | x100 |
| `merchant_ext_id` | string | Required | |
| `store_ext_id` | string | Required | |
| `payment_code` | string | Required | Scanned QR/barcode content. Without a national standard it starts with `918` and has 20 digits |
| `payment_reference_id` | string | Required | Max 64 |
| `currency` | string | Required | `MYR` |
| `validity_period` | uint32 | Optional | Default 120 s; min 30 s; max 5 days. Not with `expiry_time` |
| `expiry_time` | uint32 | Optional | Unix timestamp; min 30 s ahead; max 5 days |
| `terminal_id` | string | Optional | POS terminal |
| `promo_ids` | string | Optional | |
| `additional_info` | string | Optional | JSON with up to 3 fields: `{"field1":"..","field2":"..","field3":".."}` |

Response: `request_id`, `errcode`, `debug_msg`, `transaction_list` (named `transaction` in TH/VN; empty on failure) with `reference_id`, `amount`, `create_time`, `update_time`, `transaction_sn`, `status`, `transaction_type`, `merchant_ext_id`, `store_ext_id`, `terminal_id`, `user_id_hash`, `promo_id_applied`, `payment_channel`; `co_funding` (TH/VN only).

A "Processing" result usually means the customer must pick a method (often SPayLater); requery.

CPM errcodes: -2, -1 (requery), 0, 1, 2, 4, 5 (processing, requery), 7 (expired code), 9, 11, 14, 15, 24, 27, 42, 43, 50, 126, 140, 301 (invalid code), 303 (paying self), 603, 604, 630, 702, 1001, 1100, 1101, 1102, 1104, 1105, 1106, 1200, 1500, 1600, 1800, 1905 (expired code), 1908. Meanings are in the Error Codes table.

## Notify Transaction Status (Callback)

ShopeePay POSTs to the callback URL the merchant supplies during onboarding (not a request field). Docs: the callback has an HMAC signature in the request header, generated with the merchant's ShopeePay-issued secret key; the amount and `payment_reference_id` must be validated against the original request before treating the payment as successful; merchants are highly recommended to also implement Check Transaction Status.

Header name: not stated on the notify page. The official plugin reads `X-Airpay-Req-H` and checks `hash_equals(base64(hmac_sha256(raw_body, secret)), header)`. Use that, and confirm with ShopeePay during UAT.

Common fields:

| Field | Type | Notes |
| --- | --- | --- |
| `amount` | int64 | x100 |
| `transaction_sn` | string | ShopeePay transaction id |
| `payment_method` | uint32 | Product flow (Point of Initiation). Do not use as a default validation value |
| `user_id_hash` | string | Customer id |
| `merchant_ext_id` | string | |
| `store_ext_id` | string | |
| `terminal_id` | string | Present if sent in MPM/CPM request |
| `payment_channel` | uint32 | Source of funds |
| `payment_method_type` | string | Present for Link & Pay (API-based) |
| `co_funding` | object | TH/VN only |

Malaysia (also ID, PH, SG) additional fields:

| Flow | Field | Notes |
| --- | --- | --- |
| CPM, MPM, Checkout | `payment_reference_id` | Merchant reference |
| CPM, MPM, Checkout | `payment_status` | Success = `1` (Payment Status enum) |
| Link & Pay, Subscription | `reference_id` | `payment_reference_id` or `capture_reference_id` |
| Link & Pay, Subscription | `transaction_type` | Transaction Types enum |
| Link & Pay, Subscription | `transaction_status` | Success = `3` |

TH/VN use `reference_id`, `transaction_type`, `transaction_status` for every flow.

Samples:

```json
{"amount":10000,"transaction_sn":"019703251690639893","payment_method":2,"user_id_hash":"15e4...8d76","merchant_ext_id":"external-merchant","store_ext_id":"external-store","terminal_id":"A88","payment_reference_id":"ref-must-be-unique","payment_status":1,"payment_channel":1}
```

```json
{"amount":10000,"transaction_sn":"019703251690639893","payment_method":128,"user_id_hash":"15e4...8d76","merchant_ext_id":"external-merchant","store_ext_id":"external-store","reference_id":"ref-must-be-unique","transaction_type":13,"transaction_status":3,"payment_channel":1}
```

Merchant response: `{"errcode":0,"debug_msg":"success"}` (`errcode` required, `debug_msg` optional). In Malaysia, Indonesia, Philippines, Singapore ShopeePay sends the notification only once; in TH/VN it retries up to 2 times at 5-minute intervals if `errcode` is not 0. Docs describe notifications as updates for successful payments, so do not wait for a failure callback.

Handler outline:

```text
raw = read raw body bytes
if !ct_equal(base64(hmac_sha256(SECRET, raw)), header["X-Airpay-Req-H"]): reply errcode -1; stop
data = json(raw)
ref = data.payment_reference_id ?? data.reference_id
order = find pending order by ref; if paid already: reply errcode 0
if data.amount != order.amount_sen: log; reply non-zero; stop
success = data.payment_status == 1 || data.transaction_status == 3
if success: mark paid with data.transaction_sn (idempotent)
reply {"errcode":0,"debug_msg":"success"}
```

## Check Transaction Status

`POST /v3/merchant-host/transaction/check`. Works for all transaction types.

Request:

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Required | Max 64 |
| `reference_id` | string | Required | `payment_reference_id`, `capture_reference_id`, or `refund_reference_id` |
| `transaction_type` | uint32 | Required | 13 payment, 15 refund, 1000/1001/1002 auth flows |
| `merchant_ext_id` | string | Required | |
| `store_ext_id` | string | Required | |
| `amount` | int64 | Required | Must match the original transaction amount |

Response: `request_id`, `errcode`, `debug_msg`, `payment_method` (product flow), `transaction` {`reference_id`, `amount`, `transaction_sn`, `status`, `transaction_type`, `create_time`, `update_time`, `user_id_hash`, `merchant_ext_id`, `store_ext_id`, `terminal_id`, `promo_id_applied`, `payment_channel`, `payment_method_type`} (empty if not found), `co_funding` (TH/VN).

Errcodes: -2, -1, 0, 1, 2, 4 (not found), 5 (processing).

Decide from `transaction.status` (3 successful, 4 failed, 6 expired, 7 invalidated, 1/2 keep polling). The official plugin uses errcode 4 on a dummy reference as a credentials check.

### Retry And Requery Policy (all flows)

- Success response: mark successful. Failed: mark failed.
- Processing or no response: retry every 5 s incrementally (5, 10, 15 ... s) up to 100 s; then every 5 minutes up to 24 hours.
- For Checkout and MPM, the merchant may instead call Invalidate to terminate.
- If the customer shows proof of success in their app, the merchant may accept it and rely on T+1 reconciliation to settle discrepancies.

## Refund A Payment

`POST /v3/merchant-host/transaction/refund/create-new`

- Full or multiple partial refunds up to the original amount; wait for each partial refund to succeed before the next.
- Refund window: 365 days from payment.
- Idempotent: identical payloads only trigger one refund.
- Results: success, or failed (transaction found but not eligible).

Request:

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Required | Max 64 |
| `reference_id` | string | Required | Original `payment_reference_id` |
| `transaction_type` | uint32 | Required | Payment type being refunded (13) |
| `refund_reference_id` | string | Required | Unique, max 64 |
| `merchant_ext_id` | string | Required | |
| `store_ext_id` | string | Required | |
| `amount` | int64 | Required | x100. Docs also say a full-refund merchant that omits it is processed as usual; the plugin omits it when zero |

Response: `request_id`, `errcode`, `debug_msg`, `transaction` (sample uses key `transaction_list`) {`reference_id`, `amount`, `transaction_sn`, `status`, `transaction_type` = 15, `create_time`, `update_time`, `user_id_hash`, `merchant_ext_id`, `store_ext_id`, `terminal_id`, `payment_channel`, `payment_method_type`}, `co_funding`, `additional_info`. Parse both key names.

Refund errcodes: -2, -1, 0, 1, 2, 4, 5 (processing; requery with type 15), 11, 42, 304 (past validity), 308 (non-refundable voucher), 309 (merchant fee balance low), 320 (another refund ongoing), 321, 401 (exceeds amount), 601 (refund rules: not allowed for store, already refunded, other merchant's payment, block window 12am to 5am local by default; not exhaustive), 602 (payment not successful), 1302, 1906, 1908, 300226 (cross-border merchant).

## Common Objects

Transaction Types: Payment 13, Refund 15, Payment Authorised 1000, Payment Captured 1001, Authorisation Reversed 1002, Direct Payment 13, Capture Refund 15, Direct Payment Refund 15.

Transaction Status: 1 Initial, 2 Processing, 3 Successful, 4 Failed, 6 Expired, 7 Invalidated.

Payment Status: 1 Successful, 2 Not Found, 3 Refunded, 4 Cancelled, 5 Processing (requery), 6 Failed.

Payment Channel: 0 none (off-us), 1 ShopeePay wallet balance, 2 card, 3 linked bank, 4 SPayLater BNPL, 5 SPayLater 2 months, 6 3 months, 7 6 months, 8 12 months, 9 18 months, 10 24 months, 11 4 months, 12 5 months.

Point of Initiation (`payment_method`): 1 MPM static, 2 MPM dynamic, 4 CPM, 16 Checkout with ShopeePay, 128 Tokenized payment, 256 Cross border, 2048 Account linking, 4096 Handphone loan.

Payment Method Type (`payment_method_type`): `ewallet`, `spay_later`, `giro_bank` (linked bank), `card`.

Promotion Type (`redeemed_promotion_type`): 1 Coins back, 2 Cashback, 3 Discount.

Promotion Type V2 (`promo_type`): `DISCOUNT`, `COIN_REDEMPTION`, `CASHBACK`, `COINBACK`, `FEE_WAIVER`, `INTEREST_WAIVER`, `REWARD`.

`co_funding` (TH/VN only): `promotion_ext_name`, `redeemed_promotion_amount`, `redeemed_promotion_type`, `co_funding_cost_amount`.

## Error Codes (wallet APIs)

Read `debug_msg` for detail; merchants may show it to staff.

| errcode | Meaning |
| --- | --- |
| -2 | Server dropped the connection (unknown; requery) |
| -1 | Server error (unknown; requery) |
| 0 | Success |
| 1 | Invalid or missing mandatory parameter |
| 2 | Permission denied, often invalid status |
| 4 | Not found (merchant/store/user/transaction) |
| 5 | Processing; requery |
| 7 | Expired payment code or QR |
| 9 | Customer account banned |
| 11 | Duplicated request |
| 14 | Customer account deleted |
| 15 | Amount exceeds payment limit |
| 16 | Amount below payment limit |
| 24 | Customer account frozen |
| 27 | Customer account not activated |
| 42, 43 | Insufficient balance |
| 50 | Body is not valid JSON |
| 105 | Invalid auth code |
| 126 | Customer transaction limit reached |
| 140 | Customer wallet limit reached |
| 150 | Active linking count threshold reached |
| 301 | Invalid payment code or QR |
| 303 | Cannot pay own account |
| 304, 308, 309, 320, 321, 401, 601, 602, 1302, 1906, 300226 | Refund errors (see Refund) |
| 305 | Invalid merchant |
| 603, 1104, 1907 | Ineligible payment channel |
| 604 | Ineligible voucher |
| 627 | Promo ID exceeded limit |
| 630 | SKU ID not matched |
| 702, 705 | Unable to redeem coins / coin redemption disabled |
| 907 | Unable to verify customer |
| 1001 | Customer not allowed to transact |
| 1100, 1101, 1200, 1201, 1500, 1600, 1800, 1801 | ShopeePay internal error |
| 1102 | SPayLater credit limit reached |
| 1103 | Customer must link bank account |
| 1105 | Authentication failed |
| 1106 | Duplicated payment |
| 1601 | Invalid promotion |
| 1901 | QR or order cannot be invalidated |
| 1905 | Expired payment code or QR |
| 1908 | Scheduled maintenance |
| 300026 | Invalid or expired QR (account link) |
| 300059 | Bank account not linked |
| 300088 | Customer must complete verification |
| 300210 | ShopeePay scam warning |

## Not Documented (checked 2026-10-21)

- Public sandbox test wallets, test cards, or test data: none; UAT credentials come from onboarding.
- Rate limits: not documented.
- Callback source IP ranges or IP allowlist: not documented.
- Response signature header name: not documented.
- Callback signature header name: only in the official plugin source (`X-Airpay-Req-H`).
- Settlement report or payout API: none; docs only mention the "ShopeePay Clearing Order Report" (fed by `field1`-`field3`) and T+1 reconciliation.
- Pricing: not published.
