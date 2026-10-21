# ShopeePay Payment Gateway (Hosted Checkout Session)

A separate ShopeePay product: a ShopeePay-hosted checkout page that can offer several local methods (wallet, SPayLater, cards, and in Malaysia online banking and DuitNow QR) through one integration. Source: `https://product.shopeepay.com/integration/api/shopeepay-gateway` and `https://product.shopeepay.com/products/online-payments/payment-gateway` (checked 2026-10-21). Which methods your merchant account can actually use depends on the commercial agreement.

## Environments

| Environment | Host pattern | Malaysia |
| --- | --- | --- |
| Sandbox | `api.gw.uat.airpay.<region code>` | `https://api.gw.uat.airpay.com.my` |
| Production | `api.gw.airpay.<region code>` | `https://api.gw.airpay.com.my` |

Different from the wallet API hosts (`api[.uat].wallet.airpay.com.my`). Credentials: Client ID and Secret Key issued after the commercial agreement.

## Auth And Signature

Same protocol as the wallet APIs: HTTPS, UTF-8, UNIX seconds, TLS 1.2/1.3, HMAC-SHA256 Base64 signature over the request body with the secret key, no SDK. The page's header table names the values `client_id` and `signature`, and its header example uses:

```text
X-Airpay-ClientId: <CLIENT_ID>
X-Airpay-Req-H: <BASE64_HMAC_SHA256_OF_BODY>
```

Use the `X-Airpay-*` names. Not documented (checked 2026-10-21): what to sign for the bodiless `GET /v1/checkout/{checkout_id}` and `POST /v1/checkout/cancel/{checkout_id}` calls (likely the empty string; confirm with ShopeePay), and whether the protocol table's "POST only" rule overrides the documented `GET`.

The same documented test vector applies (see `shopeepay-gateway-api.md`).

## Flow

1. Customer clicks Checkout; backend calls Create Checkout Session.
2. Redirect to `checkout_url`. If `allowed_payment_method` contains only ShopeePay, SPayLater, or Seabank Direct Debit, the customer may open the app directly.
3. Customer confirms amount, picks a method, applies promotions, authorises.
4. ShopeePay shows the result and, on success, calls the merchant callback URL.
5. Customer returns to `return_url`. Docs: the redirect "should never be used as an indication of payment success"; rely on server responses.
6. Merchant may poll Get Checkout Status: `Successful` = paid; `Expired` or `Cancelled` = failed; `Active` = still processing (backoff 5 s steps to 100 s, then every 5 minutes up to 24 hours), or call Cancel Checkout.
7. Refund successful payments after confirming with the customer.

## Create Checkout Session

`POST /v1/checkout`

Request:

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `reference_id` | string | Required | Merchant reference, max 64 |
| `merchant_ext_id` | string | Required | |
| `store_ext_id` | string | Required | |
| `amount` | int64 | Required | x100 integer. Must equal sum(`items.price` x `items.quantity`) + fee - discount when items are sent |
| `currency` | string | Required | `MYR` for Malaysia |
| `return_url` | string | Required | HTTP or URL scheme |
| `validity_period` | uint32 | Optional | Seconds; default 1200; max 86400 (1 day) |
| `locale` | string | Optional | Malaysia: `ms`, `en`, `zh-CN` |
| `allowed_payment_method` | array of string | Optional | `spp_wallet`, `spay_later`, `card` (international cards), `online_banking` (MY only), `duitnow_qr` (MY national QR). Other markets: `bank_transfer[.bri/.seabank/.bni/.others]` (ID, VN), `maribank_direct_debit` (PH), `qris`, `promptpay_qr`, `viet_qr`, `qrph` |
| `customer_tokenization` | object | Optional | `enabled` (boolean, default false) to tokenize the method for future payments; `payment_method` (array, subset of `allowed_payment_method`; only `card` accepted for now) |
| `customer` | object | Optional | `identifier` (optional, unique in merchant system), `name`, `email`, `phone_number`, `postal_code` (all marked Required inside the object) |
| `items` | array | Optional | `name` (Req), `quantity` (Req, 1+), `price` (Req, x100; negative for discount), `image_url` (HTTPS recommended), `category` (`fee` for an extra fee with quantity 1, `discount` for a negative-price line with quantity 1) |

Response (HTTP 200):

| Field | Type | Notes |
| --- | --- | --- |
| `reference_id` | string | Echo |
| `checkout_id` | string | Session id, e.g. `SPP-...`; store it |
| `checkout_url` | string | Redirect the customer here |
| `created_at` | date-time | ISO 8601 with offset |
| `expires_at` | date-time | |

Errors (HTTP status / error code):

| HTTP | Code | Meaning |
| --- | --- | --- |
| 400 | `invalid_parameter` | Missing or badly formatted parameters |
| 400 | `invalid_mandatory_parameter` | Missing or badly formatted mandatory parameters |
| 400 | `payment_method_unsupported` | Method not supported by the gateway |
| 400 | `invalid_total_amount` | Items + fee - discount does not equal amount |
| 400 | `invalid_amount` | Too large, too small, or bad format |
| 401 | `Unauthorized` | Invalid client key |
| 403 | `feature_not_allowed` | No checkout API access, no channel enabled, or maintenance |
| 404 | `invalid_merchant` / `invalid_store` | Merchant or store missing or abnormal |
| 409 | `duplicate_reference_id` | Reference already used |
| 500 | `general_error` | Other technical error |

The JSON shape of the error body is not documented (checked 2026-10-21).

## Get Checkout Status

`GET /v1/checkout/{checkout_id}`

Response: `checkout_id`; `status` (`Active`, `Expired`, `Cancelled`, `Successful`, `Settled` = amount escrowed to merchant; sample returns lowercase `active`, so compare case-insensitively); `payment_method` (method used, same enum as `allowed_payment_method`); `created_at`; `updated_at`; `checkout_details` {`reference_id`, `merchant_ext_id`, `store_ext_id`, `amount`, `currency`, `return_url`, `expiry_time`, `locale`, `allowed_payment_method`, `customer` {`name`,`email`,`phone_number`,`postal_code`}, `items` {`name`,`quantity`,`price`,`image_url`,`category`}}.

Errors: 401 `Unauthorized`; 403 `feature_not_allowed` (checkout not under this merchant); 404 `invalid_checkout_id`; 505 `general_error`.

Settle only when status is `Successful` (or `Settled`) and `checkout_details.reference_id`, `amount`, `currency`, `merchant_ext_id`, `store_ext_id` match the local order.

## Cancel Checkout

`POST /v1/checkout/cancel/{checkout_id}`. After success the checkout URL can no longer be paid. Response: `checkout_id`, `created_at`, `updated_at`, `checkout_details` (same as above).

## Create Refund

`POST /v1/refund` (docs write `v1/refund`)

Full or multiple partial refunds up to the original amount; wait for each refund to complete before the next.

Request: `amount` (int64, Required, x100), `original_checkout_id` (string, Required), `refund_reference_id` (string, Required, max 64, unique).

Response: `refund_id`, `original_checkout_id`, `refund_reference_id`, `amount`, `created_at`, `updated_at`, `status`.

Status values: field description says `pending`, `succeeded`, `failed`, while every sample returns `successful`. Treat `succeeded` and `successful` as success.

## Get Refund Status

Path `v1/refund/{refund_id}` (HTTP method not stated; Get Checkout Status uses GET).

Response: `refund_id`, `amount`, `status` (`pending`, `succeeded`/`successful`, `failed`), `created_at`, `updated_at`, `refund_session_details` {`refund_reference_id`, `original_checkout_id`}.

Errors: 401 `Unauthorized`; 403 `feature_not_allowed`; 404 `invalid_refund_id`; 505 `general_error`.

## Webhook (Notify Transaction Status)

ShopeePay calls the callback URL registered during onboarding for final checkout status and refunds. Each callback carries an HMAC signature in a request header generated with the merchant's secret key (header name not stated on this page; the wallet flow uses `X-Airpay-Req-H`). Verify over the raw body before parsing.

Event types: `checkout.successful`, `checkout.expired`, `checkout.cancelled`, `refund.successful`, `refund.failed`.

Envelope (from the documented refund sample):

```json
{
  "event_type": "refund.successful",
  "event_id": "unique_identifier_of_the_webhook_event",
  "timestamp": "2025-05-15T19:00:00+07:00",
  "created_at": "2025-05-15T18:55:00+07:00",
  "updated_at": "2025-05-15T19:00:00+07:00",
  "data": {
    "refund_id": "...",
    "amount": "10000",
    "currency": "IDR",
    "status": "successful",
    "failure_reason": "",
    "refund_session_details": {
      "refund_reference_id": "unique-transaction-id-12345",
      "original_checkout_id": "...",
      "amount": 10000,
      "currency": "IDR"
    }
  }
}
```

Note `data.amount` is a string in the sample while the nested amount is an integer; parse both. No `checkout.*` payload sample is published, and the expected acknowledgement body and retry policy are not documented for this product (checked 2026-10-21). Deduplicate on `event_id`, then confirm with Get Checkout Status before settling.
