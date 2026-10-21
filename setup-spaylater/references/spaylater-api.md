# SPayLater API Reference (via ShopeePay APIs)

Use this reference for factual SPayLater integration guidance. SPayLater has no separate merchant API: an online merchant accepts SPayLater through ShopeePay's APIs, where SPayLater is a payment method (`spay_later`) and a set of payment channels (4 to 12). All facts below come from `https://product.shopeepay.com` (checked 2026-10-21) and the official "Checkout with ShopeePay" WooCommerce plugin v1.0.0 published by ShopeePay on WordPress.org.

Related files:

- `spaylater-tokenized.md`: Account Linking, Link & Pay (SPayLater instalment options), Subscription, Auth & Capture.
- `spaylater-widget.md`: SPayLater promotional messaging widget (Web, Android, iOS, Shopify app).
- `account-setup.md`: onboarding, credentials, env vars, settlement.

## Source URLs

- `https://product.shopeepay.com/integration/get-started`
- `https://product.shopeepay.com/integration/api/shopeepay-gateway`
- `https://product.shopeepay.com/integration/api/checkout-with-shopeepay`
- `https://product.shopeepay.com/integration/api/check-transaction-status`
- `https://product.shopeepay.com/integration/api/notify-transaction-status`
- `https://product.shopeepay.com/integration/api/refund-a-payment`
- `https://product.shopeepay.com/integration/api/invalidate-order`
- `https://product.shopeepay.com/integration/api/merchant-presented-mode`
- `https://product.shopeepay.com/integration/api/customer-presented-mode`
- `https://product.shopeepay.com/integration/api/invalidate-qr`
- `https://product.shopeepay.com/integration/common-objects`
- `https://product.shopeepay.com/products/functionalities`
- `https://wordpress.org/plugins/checkout-with-shopeepay/` (plugin source: `includes/class-shopeepay-api.php`, `class-shopeepay-webhook.php`, `class-shopeepay-countries.php`)

## Which Integration To Choose For SPayLater

| Surface | API family | How SPayLater is selected | Notes |
| --- | --- | --- | --- |
| ShopeePay Payment Gateway (hosted checkout) | Gateway v1, host `api.gw...` | `allowed_payment_method: ["spay_later"]` | Hosted page lists methods; SPayLater-only may open the Shopee/ShopeePay app directly. Status by `checkout_id`. |
| Checkout with ShopeePay (redirect/app jump) | Wallet v3, host `api.wallet...` | `preferred_payment_method_type: "spay_later"` | Used by the official WooCommerce plugin. Customer picks tenure in the Shopee/ShopeePay app. |
| Link & Pay (API-based) | Wallet v3/v4 | `payment_method_type: "spay_later"` + `payment_option_reference` of a tenure | Merchant shows SPayLater tenures itself. See `spaylater-tokenized.md`. |
| MPM dynamic QR (in person) | Wallet v3 | `preferred_payment_method_type: "spay_later"` | |
| CPM (merchant scans customer code) | Wallet v3 | Customer chooses in app | SPayLater often returns "Processing" first. |

For a custom Malaysian web checkout that wants SPayLater, the default is Checkout with ShopeePay (v3) or the Payment Gateway (v1). Ask the ShopeePay integration team which product your agreement enables.

## Environments (Malaysia)

Region code for Malaysia is `com.my`.

| API family | Staging / Sandbox | Production |
| --- | --- | --- |
| Wallet APIs (`/v3/merchant-host/...`, `/v4/...`) | `https://api.uat.wallet.airpay.com.my` | `https://api.wallet.airpay.com.my` |
| Payment Gateway (`/v1/checkout`, `/v1/refund`) | `https://api.gw.uat.airpay.com.my` | `https://api.gw.airpay.com.my` |

Other regions: ID `co.id`, PH `com.ph`, SG `sg`, TH `co.th`, VN `vn`. The WooCommerce plugin hardcodes the same wallet hosts.

Credentials (Client ID, Secret Key) are issued by ShopeePay after NDA and commercial agreement. No public self-serve sandbox account or public test credentials are documented (checked 2026-10-21).

## API Protocol Rules

- HTTPS only, TLS 1.2 or 1.3.
- Submit mode: POST (Gateway status lookups use GET, see below).
- Dates: UNIX seconds (v3). Gateway v1 uses ISO 8601 strings with offset.
- Encoding: UTF-8. Body: JSON.
- No SDK: ShopeePay says to integrate directly with endpoints, without an SDK.
- All calls from the merchant server; never put the secret key in a frontend.
- Response fields may come in any order, new fields may be added without notice, and empty values default to: Integer `0`, String `""` (or `"0"` for numeric strings like amount or timestamp), Object `null`, Array `[]`, Boolean `false`.
- URL query parameters may come in any order, new ones may be added, and they are case-sensitive.
- Backward-compatible changes (new endpoints, new optional request fields, new response fields, length changes, field order) can ship without notice. Parse leniently.

## Authentication And Signature

Request headers on every call:

```text
Content-Type: application/json
X-Airpay-ClientId: <CLIENT_ID>
X-Airpay-Req-H: <signature>
```

Algorithm: HMAC-SHA256 over the exact raw request body bytes with the shared Secret Key, then standard Base64 of the raw 32-byte digest (not of the hex string).

```text
signature = base64( hmac_sha256( key = SECRET_KEY, message = raw_json_body_bytes ) )
```

Rules:

- Sign the exact string you send. Serialize once, sign that string, send that same string. Any whitespace or key-order change breaks the signature.
- Response validation: sign the raw response body with the same secret and compare with the signature header on the response. If different, do not trust the response. The docs do not name the response header; the WooCommerce plugin does not check response signatures. Treat `X-Airpay-Req-H` on the response as the likely header and confirm with ShopeePay.
- Callback validation: callbacks carry an HMAC signature in the header, made with your secret key. The official plugin reads header `X-Airpay-Req-H` and checks `hash_equals(base64(hmac_sha256(raw_body, secret)), header)` before parsing JSON.
- Use constant-time comparison.
- GET requests (Gateway v1 status lookups) have no body. The docs say the body is signed "if the request has a body" and do not say what to sign for a bodyless GET. Not documented in official docs (checked 2026-10-21); confirm with ShopeePay (signing an empty string is the natural reading, unverified).

Published test vector (docs example secret, not a real key):

```text
secret = pz148x0gXyPCLHxnlhEydNLg55jni91i
body   = {"request_id": "","store_ext_id": "externalstore","merchant_ext_id": "externalmerchant","amount": 1000,"terminal_id": "terminal","convenience_fee_percentage": 0,"convenience_fee_fixed": 0,"convenience_fee_indicator": "","additional_info": "","currency": "IDR","qr_validity_period": 1000,"payment_reference_id": "testreference"}
digest (hex) = 5f5532bf23018674788770f43743522dff1af5782243fab06b1e8677a7391d4c
signature    = X1UyvyMBhnR4h3D0N0NSLf8a9XgiQ/qwax6Gd6c5HUw=
```

(Verified locally: HMAC-SHA256 of that exact body with that secret gives the digest above.)

Node.js:

```js
import crypto from "node:crypto";
const sign = (raw, secret) => crypto.createHmac("sha256", secret).update(raw, "utf8").digest("base64");
const body = JSON.stringify(payload);            // send exactly this string
headers["X-Airpay-ClientId"] = process.env.SHOPEEPAY_CLIENT_ID;
headers["X-Airpay-Req-H"] = sign(body, process.env.SHOPEEPAY_SECRET_KEY);
// callback: verify on the raw body before JSON.parse
const ok = crypto.timingSafeEqual(Buffer.from(sign(rawBody, secret)), Buffer.from(headerValue || ""));
```

(`timingSafeEqual` throws on length mismatch; check lengths first.)

## Amount Rules

- All amounts are integers inflated by 100, no decimal point. MYR: RM 12.34 is `1234`. Smallest unit `1` = RM 0.01.
- For IDR and VND the smallest amount is `100` (= 1 unit).
- Gateway v1: `amount` must equal `sum(items.price x items.quantity) + fee - discount` when items are passed, else `invalid_total_amount`.
- Gateway v1 callback samples show `data.amount` as a string (`"10000"`); v3 callbacks use a number. Compare as integers.
- Minimum and maximum SPayLater amounts are not documented (checked 2026-10-21). v3 errcodes `15` (exceeded payment limit) and `16` (did not meet payment limit, MPM) exist.

## Endpoint Catalogue

All paths are relative to the host of their API family.

| # | Method | Path | Purpose | Family |
| --- | --- | --- | --- | --- |
| 1 | POST | `/v1/checkout` | Create checkout session (hosted page) | Gateway |
| 2 | GET | `/v1/checkout/{checkout_id}` | Get checkout status | Gateway |
| 3 | POST | `/v1/checkout/cancel/{checkout_id}` | Cancel checkout | Gateway |
| 4 | POST | `/v1/refund` | Create refund | Gateway |
| 5 | GET | `/v1/refund/{refund_id}` | Get refund status | Gateway |
| 6 | Callback | merchant callback URL | Gateway events `checkout.*`, `refund.*` | Gateway |
| 7 | POST | `/v3/merchant-host/order/create` | Create Checkout Order (Checkout with ShopeePay) | Wallet |
| 8 | POST | `/v3/merchant-host/order/invalidate` | Invalidate checkout order | Wallet |
| 9 | POST | `/v3/merchant-host/qr/create` | Create dynamic QR (MPM) | Wallet |
| 10 | POST | `/v3/merchant-host/qr/invalidate` | Invalidate MPM QR | Wallet |
| 11 | POST | `/v3/merchant-host/transaction/payment/create` | Create CPM payment | Wallet |
| 12 | POST | `/v3/merchant-host/transaction/check` | Check transaction status (all types) | Wallet |
| 13 | POST | `/v3/merchant-host/transaction/refund/create-new` | Refund a payment | Wallet |
| 14 | Callback | merchant callback URL | Notify Transaction Status | Wallet |
| 15 | POST | `/v3/merchant-host/account/link` | Initiate account linking | Wallet |
| 16 | POST | `/v3/merchant-host/access-token/get` | Get access token | Wallet |
| 17 | POST | `/v3/merchant-host/account/unlink` | Unlink account | Wallet |
| 18 | Callback | merchant callback URL | Notify account link/unlink status | Wallet |
| 19 | POST | `/v3/merchant-host/user-info/get` | Get user info (incl. `spaylater_info`) | Wallet |
| 20 | POST | `/v3/merchant-host/coin/potential-redemption` | Get coin redemption info | Wallet |
| 21 | POST | `/v4/merchant-host/payment-methods/get` | Get payment methods (incl. SPayLater tenures) | Wallet |
| 22 | POST | `/v3/merchant-host/transaction/payment/direct` | Direct Payment (Link & Pay and Subscription) | Wallet |
| 23 | POST | `/v3/merchant-host/transaction/auth/create` | Create auth | Wallet |
| 24 | POST | `/v3/merchant-host/transaction/auth/capture` | Capture | Wallet |
| 25 | POST | `/v3/merchant-host/transaction/auth/reverse` | Reverse auth | Wallet |

Rows 15 to 25 are detailed in `spaylater-tokenized.md`. Gateway paths `v1/refund` and `v1/refund/{refund_id}` appear without a leading slash in the docs; use `/v1/...`.

Rate limits: not documented in official docs (checked 2026-10-21). IP allowlist for callbacks: not documented. ShopeePay says not to apply domain whitelisting or IP restriction to `redirect_url_http` / `redirect_url_web`.

## A. Payment Gateway v1 (Hosted Checkout)

### A1. Create Checkout Session: `POST /v1/checkout`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `reference_id` | string | Yes | Merchant transaction ID, max 64. Duplicate gives 409 `duplicate_reference_id`. |
| `merchant_ext_id` | string | Yes | Merchant ID in merchant system (as registered with ShopeePay). |
| `store_ext_id` | string | Yes | Store ID. |
| `amount` | int64 | Yes | x100. Must equal items total + fee - discount. |
| `currency` | string | Yes | `MYR` for Malaysia. |
| `return_url` | string | Yes | HTTP link or URL scheme; browser comes back here after payment or cancel. |
| `validity_period` | uint32 | No | Seconds. Default 1200 (20 min). Max 86400. |
| `locale` | string | No | MY: `ms`, `en`, `zh-CN`. |
| `allowed_payment_method` | array of string | No | `spp_wallet`, `spay_later` (all markets), `card` (international cards, "all markets"), `online_banking` (MY only), `duitnow_qr` (MY national QR). Others: `bank_transfer[.bri/.seabank/.bni/.others]` ID/VN, `maribank_direct_debit` PH, `qris`, `promptpay_qr`, `viet_qr`, `qrph`. |
| `customer_tokenization.enabled` | boolean | No | Tokenize the method for future use. Default false. |
| `customer_tokenization.payment_method` | array | No | Subset of `allowed_payment_method`; only `card` accepted now. |
| `customer.identifier` | string | No | Unique customer ID in merchant system. |
| `customer.name` | string | Yes (if customer sent) | |
| `customer.email` | string | Yes (if customer sent) | |
| `customer.phone_number` | string | Yes (if customer sent) | |
| `customer.postal_code` | string | Yes (if customer sent) | |
| `items[].name` | string | Yes | |
| `items[].quantity` | int64 | Yes | 1 or more. |
| `items[].price` | int64 | Yes | x100. Negative only for discount rows. |
| `items[].image_url` | string | No | Public HTTPS URL recommended. |
| `items[].category` | string | No | `fee` (positive, qty 1) or `discount` (negative, qty 1). |

Response: `reference_id`, `checkout_id` (e.g. `SPP-...`), `checkout_url` (redirect the customer here), `created_at`, `expires_at` (ISO 8601).

HTTP errors:

| HTTP | error code | Meaning |
| --- | --- | --- |
| 200 | | Success |
| 400 | `invalid_parameter` | Missing or badly formatted parameter |
| 400 | `invalid_mandatory_parameter` | Missing or badly formatted mandatory parameter |
| 400 | `payment_method_unsupported` | Method not supported by the gateway |
| 400 | `invalid_total_amount` | Items total + fee - discount does not match `amount` |
| 400 | `invalid_amount` | Amount too large, too small, or bad format |
| 401 | `Unauthorized` | Invalid client key |
| 403 | `feature_not_allowed` | No checkout API access, no channel enabled, or maintenance |
| 404 | `invalid_merchant` / `invalid_store` | Merchant or store missing or abnormal |
| 409 | `duplicate_reference_id` | Reference already used |
| 500 | `general_error` | Other technical error |

Sample request (from docs, with SPayLater only):

```json
{"reference_id":"webhost165","merchant_ext_id":"testxxxpg","store_ext_id":"testxxx","amount":100000,"currency":"IDR","return_url":"https://www.google.com","validity_period":7200,"locale":"en","allowed_payment_method":["spay_later"],"items":[{"name":"item1","quantity":1,"price":100000},{"name":"shipping","quantity":1,"price":100,"category":"fee"},{"name":"discount","quantity":1,"price":-100,"category":"discount"}],"customer":{"name":"john cena","postal_code":"12345","phone_number":"00810029200006","email":"test@test.com"}}
```

### A2. Get Checkout Status: `GET /v1/checkout/{checkout_id}`

Response: `checkout_id`, `status`, `payment_method` (one of the `allowed_payment_method` values, e.g. `spay_later`), `created_at`, `updated_at`, `checkout_details` {`reference_id`, `merchant_ext_id`, `store_ext_id`, `amount`, `currency`, `return_url`, `expiry_time`, `locale`, `allowed_payment_method`, `customer` {name, email, phone_number, postal_code}, `items` [{name, quantity, price, image_url, category}]}.

`status` values: `Active` (created, still processing), `Expired`, `Cancelled`, `Successful` (paid), `Settled` (amount escrowed to merchant). Docs list them capitalized; samples return lowercase (`"active"`). Compare case-insensitively.

Merchant action: `Successful` (or `Settled`) = paid; `Expired` or `Cancelled` = failed; `Active` = poll (schedule below) or cancel.

Errors: 401 `Unauthorized`; 403 `feature_not_allowed` (checkout not under this merchant); 404 `invalid_checkout_id`; 505 `general_error`.

### A3. Cancel Checkout: `POST /v1/checkout/cancel/{checkout_id}`

After success the `checkout_url` can no longer be paid. Response: `checkout_id`, `created_at`, `updated_at`, `checkout_details` (same shape as A2).

### A4. Create Refund: `POST /v1/refund`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `amount` | int64 | Yes | x100 refund amount. |
| `original_checkout_id` | string | Yes | `checkout_id` of the paid session. |
| `refund_reference_id` | string | Yes | Merchant refund ID, max 64. |

Response: `refund_id`, `original_checkout_id`, `refund_reference_id`, `amount`, `created_at`, `updated_at`, `status`. Full and multiple partial refunds allowed up to the original amount; wait for the previous partial refund to finish before the next.

### A5. Get Refund Status: `GET /v1/refund/{refund_id}`

Response: `refund_id`, `amount`, `status`, `created_at`, `updated_at`, `refund_session_details` {`refund_reference_id`, `original_checkout_id`}. Errors: 401 `Unauthorized`, 403 `feature_not_allowed`, 404 `invalid_refund_id`, 505 `general_error`.

Refund `status`: docs text says `pending`, `succeeded`, `failed`; samples return `successful`. Accept both `succeeded` and `successful` as success.

### A6. Gateway Callbacks

Callback URL is given to ShopeePay during onboarding. Header carries the HMAC signature (verify as above). Event types: `checkout.successful`, `checkout.expired`, `checkout.cancelled`, `refund.successful`, `refund.failed`.

Checkout callback body: `event_type`, `event_id` (dedupe key), `timestamp`, `created_at`, `updated_at`, `data` {`checkout_id`, `amount` (string), `currency`, `status`, `checkout_details` {reference_id, merchant_ext_id, store_ext_id, currency, return_url, expiry_time, locale, customer, items, `email_address_collection.enabled`, `phone_number_collection.enabled`}}.

Refund callback body: same envelope, `data` {`refund_id`, `amount`, `currency`, `status`, `failure_reason`, `refund_session_details` {refund_reference_id, original_checkout_id, amount, currency}}.

The required acknowledgement body and retry policy for Gateway callbacks are not documented (checked 2026-10-21). Return HTTP 200 quickly and confirm with ShopeePay.

## B. Checkout with ShopeePay (v3)

### B1. Create Checkout Order: `POST /v3/merchant-host/order/create`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Yes | Unique per request, max 64. |
| `payment_reference_id` | string | Yes | Merchant transaction ID, max 64. Unique. |
| `merchant_ext_id` | string | Yes | |
| `store_ext_id` | string | Yes | |
| `amount` | int64 | Yes | x100. |
| `currency` | string | Yes | ISO 4217, `MYR`. |
| `return_url` | string | Yes | Where the customer returns after payment or cancel. |
| `platform_type` | string | Yes | `app`, `pc`, or `mweb`. |
| `validity_period` | uint32 | No | Seconds. Default 1200. Max 3600 in Malaysia. Do not send with `expiry_time`. |
| `expiry_time` | uint32 | No | UNIX timestamp. Same limits. Do not send with `validity_period`. |
| `promo_ids` | string | No | Comma separated, up to 20, total 100 chars or less. |
| `additional_info` | JSON string | No | `field1`, `field2`, `field3` (remarks shown in the ShopeePay clearing order report), `order_detail` {`shipping_address` {region, state, city, address, postal_code}, `item_info_list` [{id, name, quantity, currency, amount, category, brand, photo_url}]}, `merchant_user_account_info.user_identifier_info` {user_id, phone_number, email, username}. |
| `phone` | string | No | Customer phone with country code, no plus, e.g. `60123456789`. |
| `preferred_payment_method_type` | string | No | `spay_later` to preselect SPayLater on the payment page, `spp_wallet` for wallet. `maribank_direct_debit` PH only. |

Note `additional_info` is a JSON-encoded string inside the JSON body, not a nested object.

Response: `request_id`, `errcode`, `debug_msg`, `redirect_url_http` (universal URL to the ShopeePay payment page; opens the Shopee/ShopeePay app if installed, else web login by phone), `qr_content`, `qr_url` (QR image, valid 5 minutes; PC web flows).

errcode: `-2` server dropped connection; `-1` server error, use Check Transaction Status; `0` success; `1` invalid or missing parameter; `2` permission denied; `4` not found; `11` duplicated request; `15` amount exceeded payment limit; `50` body is not valid JSON; `627` too many promo IDs; `1800` server error; `1908` scheduled maintenance.

The sample `redirect_url_http` embeds a Base64 `return_url` that ShopeePay later opens with query parameters such as `amount`, `client_id`, `reference_id`, `result_code`, `signature`. These return parameters are not documented (checked 2026-10-21). Never settle from them; ShopeePay says the redirect must not be used as a success signal.

### B2. Invalidate Order: `POST /v3/merchant-host/order/invalidate`

Fields: `request_id` (Yes, max 64), `payment_reference_id` (Yes), `merchant_ext_id` (Yes), `store_ext_id` (Yes). Response: `request_id`, `errcode`, `debug_msg`. After success the `redirect_url_http` can no longer be paid. errcode: `-2`, `-1`, `0`, `1`, `2`, `4`, `1800` server error, `1901` cannot be invalidated.

Use it when the merchant order is cancelled or when polling gives up. The WooCommerce plugin calls it when a Woo order is cancelled.

## C. Check Transaction Status: `POST /v3/merchant-host/transaction/check`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Yes | Max 64. |
| `reference_id` | string | Yes | `payment_reference_id`, `capture_reference_id`, or `refund_reference_id`. |
| `transaction_type` | uint32 | Yes | `13` payment, `15` refund, `1000`/`1001`/`1002` auth flows. |
| `merchant_ext_id` | string | Yes | |
| `store_ext_id` | string | Yes | |
| `amount` | int64 | Yes | Must equal the original transaction amount. |

Response: `request_id`, `errcode`, `debug_msg`, `payment_method` (Point of Initiation; do not hardcode as a validation default), `transaction` {`reference_id`, `amount`, `transaction_sn`, `status`, `transaction_type`, `create_time`, `update_time`, `user_id_hash`, `merchant_ext_id`, `store_ext_id`, `terminal_id`, `promo_id_applied`, `payment_channel`, `payment_method_type`}, `co_funding` (TH/VN only).

errcode: `-2`, `-1`, `0`, `1`, `2`, `4` not found, `5` processing.

Decision: `transaction.status` `3` = success; `4`, `6`, `7` = failed/expired/invalidated; `1`, `2` or errcode `5` = keep polling. `payment_channel` 4 to 12 tells you SPayLater and the tenure.

Polling schedule (all v3 flows and Gateway `Active`): retry every 5 s increasing (5, 10, 15 s...) up to 100 s total; then every 5 minutes up to 24 hours. Or invalidate/cancel to stop. ShopeePay also says that if the customer shows proof of success in their app, the merchant may accept it and reconcile at T+1.

## D. Notify Transaction Status (v3 callback)

Merchant registers one callback URL during onboarding. Each callback has an HMAC signature in the header (`X-Airpay-Req-H` per the official plugin). ShopeePay requires the merchant to check that `amount` and `payment_reference_id` match the original request before marking paid, and strongly recommends also implementing Check Transaction Status.

Common fields: `amount` (int64 x100), `transaction_sn`, `payment_method` (Point of Initiation, e.g. `16` Checkout, `128` tokenized), `user_id_hash`, `merchant_ext_id`, `store_ext_id`, `terminal_id` (MPM/CPM only), `payment_channel`, `payment_method_type` (Link & Pay API-based only), `co_funding` {promotion_ext_name, redeemed_promotion_amount, redeemed_promotion_type, co_funding_cost_amount} (TH/VN only).

Malaysia (also ID, PH, SG) extra fields:

- CPM, MPM, Checkout: `payment_reference_id`, `payment_status` (`1` = success; see Payment Status table).
- Link & Pay, Subscription: `reference_id`, `transaction_type`, `transaction_status` (`3` = success).

TH and VN use `reference_id`, `transaction_type`, `transaction_status` for every flow.

Acknowledge with JSON `{"errcode":0,"debug_msg":"success"}`. In Malaysia ShopeePay sends each notification only once (no retry). Only TH and VN retry (2 more times, 5 minutes apart). So a Malaysian integration must poll Check Transaction Status when a callback does not arrive.

Sample Checkout callback:

```json
{"amount":10000,"transaction_sn":"019703251690639893","payment_method":2,"user_id_hash":"15e4...8d76","merchant_ext_id":"external-merchant","store_ext_id":"external-store","terminal_id":"A88","payment_reference_id":"ref-must-be-unique","payment_status":1,"payment_channel":1}
```

## E. Refund a Payment (v3): `POST /v3/merchant-host/transaction/refund/create-new`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Yes | Max 64. |
| `reference_id` | string | Yes | Original `payment_reference_id`. |
| `transaction_type` | uint32 | Yes | Payment type, normally `13`. |
| `refund_reference_id` | string | Yes | Merchant refund ID, max 64. |
| `merchant_ext_id` | string | Yes | |
| `store_ext_id` | string | Yes | |
| `amount` | int64 | Yes (listed) | x100. Docs also say a full refund without `amount` is processed; the plugin omits it for full refunds. |

Response: `request_id`, `errcode`, `debug_msg`, `transaction` (sample key is `transaction_list`) {`reference_id`, `amount`, `transaction_sn`, `status`, `transaction_type` (15), `create_time`, `update_time`, `user_id_hash`, `merchant_ext_id`, `store_ext_id`, `terminal_id`, `payment_channel`, `payment_method_type`}, `co_funding`, `additional_info`.

Rules: full or multiple partial refunds up to the paid amount; wait for each partial refund to finish; same payload is idempotent (only acts once); refund window 365 days from payment on this page (the Product Functionalities page says 180 days, so confirm); refund block period 12 am to 5 am local time by default.

errcode: `-2`, `-1`, `0`, `1`, `2`, `4`, `5` processing (requery with `transaction_type` 15), `11` duplicated, `42` insufficient balance, `304` past refund validity, `308` non-refundable voucher, `309` not enough fee balance, `320` another refund ongoing, `321` insufficient balance, `401` amount exceeds transaction, `601` refund rules not met (not allowed for store, already refunded, different merchant/store, block period), `602` payment not successful, `1302` insufficient balance, `1906` transaction failed, `1908` maintenance, `300226` cross-border merchant.

## F. In-Person Flows (SPayLater-relevant notes)

MPM `POST /v3/merchant-host/qr/create`: `request_id` (Yes), `amount` (Yes), `currency` (Yes), `merchant_ext_id` (Yes), `store_ext_id` (Yes), `payment_reference_id` (Yes, max 64 in MY), `qr_validity_period` or `expiry_time` (No, default 1200 s, MY max 3600, not both), `terminal_id`, `convenience_fee_indicator` (`01` customer input, `02` fixed, `03` percentage; ID/MY), `convenience_fee_fixed`, `convenience_fee_percentage` (MY), `promo_ids`, `additional_info`, `preferred_payment_method_type` (`spay_later`). Response: `request_id`, `errcode`, `debug_msg`, `store_name`, `qr_content`, `qr_url` (5 minutes). errcode adds `15`, `16` (below limit), `50`, `627`, `1800`, `1908`.

Invalidate MPM QR `POST /v3/merchant-host/qr/invalidate`: same fields as B2; errcode `1901` QR cannot be invalidated.

CPM `POST /v3/merchant-host/transaction/payment/create`: `request_id`, `amount`, `merchant_ext_id`, `store_ext_id`, `payment_code` (scanned; proprietary codes start with `918`, 20 digits), `payment_reference_id`, `currency` (all Yes); `validity_period`/`expiry_time` (default 120 s, 30 s to 5 days), `terminal_id`, `promo_ids`, `additional_info` (field1..3). Response `transaction_list` (named `transaction` in TH/VN). "Processing" is common when the customer picks SPayLater, so poll. errcode includes `7`/`1905` expired code, `126` transaction limit, `301` invalid code, `303` paying own account, `603` ineligible channel, `604` ineligible voucher, `630` SKU mismatch, `702` cannot redeem coins, `1102` SPL credit limit reached, `1105` authentication failed, `1106` duplicated payment.

## G. Common Objects (enums)

Transaction Types: Payment `13`, Refund `15`, Payment Authorised `1000`, Payment Captured `1001`, Authorisation Reversed `1002`, Direct Payment `13`, Capture Refund `15`, Direct Payment Refund `15`.

Transaction Status: Initial `1`, Processing `2`, Successful `3`, Failed `4`, Expired `6`, Invalidated `7`.

Payment Status (v3 callback `payment_status`): Successful `1`, Not Found `2`, Refunded `3`, Cancelled `4`, Processing `5` (requery), Failed `6`.

Payment Channel (`payment_channel`):

| Value | Channel |
| --- | --- |
| 0 | No available payment channel (e.g. off-us) |
| 1 | ShopeePay Wallet Balance |
| 2 | Credit / Debit Card |
| 3 | Linked Bank Account |
| 4 | SPayLater Buy Now Pay Later |
| 5 | SPayLater 2 months instalment |
| 6 | SPayLater 3 months instalment |
| 7 | SPayLater 6 months instalment |
| 8 | SPayLater 12 months instalment |
| 9 | SPayLater 18 months instalment |
| 10 | SPayLater 24 months instalment |
| 11 | SPayLater 4 months instalment |
| 12 | SPayLater 5 months instalment |

Point of Initiation (`payment_method`): MPM static `1`, MPM dynamic `2`, CPM `4`, Checkout with ShopeePay `16`, Tokenized Payment `128`, Cross Border `256`, Account Linking `2048`, Handphone Loan `4096`.

Payment Method Type (`payment_method_type`): `ewallet`, `spay_later`, `giro_bank` (linked bank), `card`.

Promotion Type (v1): Coins Back `1`, Cashback `2`, Discount `3`.

Promotion Type V2 (`promo_type`): `DISCOUNT`, `COIN_REDEMPTION`, `CASHBACK`, `COINBACK`, `FEE_WAIVER`, `INTEREST_WAIVER` (SPayLater instalment interest waiver), `REWARD`. (A Link & Pay sample shows `COINS_REDEMPTION`; accept both spellings.)

## Settlement Logic Checklist

1. Store local order: `payment_reference_id` (or `reference_id` / `checkout_id`), expected amount x100, currency, environment, status `pending`.
2. Create the order/session server-side; redirect to `redirect_url_http` or `checkout_url`.
3. On callback: verify `X-Airpay-Req-H` over raw body; parse; look up by reference; reject if `merchant_ext_id`, `store_ext_id`, amount or currency differ; dedupe (`event_id` for Gateway, reference + `transaction_sn` for v3); mark paid only on `payment_status` 1 / `transaction_status` 3 / `checkout.successful`; reply `errcode 0`.
4. On return URL: do not trust query params; call Check Transaction Status or Get Checkout Status.
5. Missing callback (Malaysia: no retries): poll on the documented schedule, then invalidate/cancel.
6. Record `payment_channel` to know which SPayLater tenure was used. The ShopeePay Malaysia merchant page says ShopeePay handles all credit risk and settles in full, so tenure does not change the amount you expect; fees follow your agreement.
