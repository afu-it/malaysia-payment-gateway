# AhaPay API Reference

Use this reference for factual AhaPay API guidance. AhaPay is a Malaysian Buy Now Pay Later service run by Aharetail Solutions Sdn. Bhd. The docs portal (`https://apidocs.ahapay.my/`) is built by Fingular and mirrors `https://docs.fingular.com/`. All facts below were checked on 2026-10-21. Verify fees, limits, payout schedule and key status with AhaPay when those details matter.

## Source URLs

- `https://apidocs.ahapay.my/docs/api/Introduction`
- `https://apidocs.ahapay.my/docs/api/Flow`
- `https://apidocs.ahapay.my/docs/api/v2/Authentication`
- `https://apidocs.ahapay.my/docs/api/v2/Orders/Create-Default-Order`
- `https://apidocs.ahapay.my/docs/api/v2/Orders/Create-POS-Order`
- `https://apidocs.ahapay.my/docs/api/v2/Orders/Create-QR-POS-Order`
- `https://apidocs.ahapay.my/docs/api/v2/Check-Order-Status`
- `https://apidocs.ahapay.my/docs/api/v2/Refund-Order`
- `https://apidocs.ahapay.my/docs/api/v2/Daily-Limit-Status`
- `https://apidocs.ahapay.my/docs/api/v2/Health-Check`
- `https://apidocs.ahapay.my/docs/api/v2/Webhook`
- `https://apidocs.ahapay.my/docs/api/Callback-Security`
- `https://apidocs.ahapay.my/docs/api/Status%20Codes`
- `https://apidocs.ahapay.my/docs/api/v1/...` (legacy V1 pages)
- `https://apidocs.ahapay.my/openapi.json` (V1 OpenAPI 3.0 spec behind `/swagger`)
- Official WooCommerce plugin source: `https://wordpress.org/plugins/ahapay-buy-now-pay-later/` (v1.1.0)

## Integration Surfaces

- Online checkout (web or mobile app): `POST /v2/orders/default`, redirect to `online_order_payment_link`.
- POS terminal with a screen: `POST /v2/orders/pos`, display `link_qr` for the customer to scan.
- POS terminal with a scanner: customer shows an AhaPay app QR (`client_hash`), POS backend calls `POST /v2/orders/qr_pos`.
- Platform plugins: WooCommerce (official plugin), EasyStore (built-in app). Shopify, Magento, PrestaShop, OpenCart and Wix are listed as "Coming Soon".
- No-code in store: the AhaPay Agent app generates a QR for the amount (merchant FAQ). No API needed.

## Environments

| Environment | API base URL |
| --- | --- |
| Sandbox | `https://api.bnpl-my.dev.mychili.id` |
| Production | `https://api.ahapay.my` |

Source: V2 Authentication page and plugin constants `AHAPAYBUYNOWPAYLATER_SANDBOX_BASE_URL` / `AHAPAYBUYNOWPAYLATER_LIVE_BASE_URL`.

Note: the older WooCommerce docs page once listed `https://api.ahapay.com/` as live; the current plugin source and V2 docs use `https://api.ahapay.my`. Use `https://api.ahapay.my`.

Customer payment page host seen in examples: `https://payment.ahapay.my/?onlineOrderId=...`.

Separate API keys are issued for sandbox and production ("API keys are required for both Sandbox and Production environments"). No public self-serve sandbox signup is documented; keys are "given while onboarding".

## Authentication

Every request carries:

```http
X-ApiKey: <API_KEY>
Content-Type: application/json
```

| Key type | Use | Extra requirement |
| --- | --- | --- |
| Merchant API Key (Trade Outlet API key) | Direct integration by the merchant | Do not send `external_merchant_id` |
| Partner API Key | A platform integrating many merchants | Send `external_merchant_id` (partner's internal merchant ID) on order creation and daily limit calls |

"The system will automatically determine whether it's a trade outlet API key or a partner API key." The same key is also the secret behind the callback `X-Api-Key-Hash` header (see Callbacks).

Documented environment variables (V2 Authentication page): `base_url`, `api_key`, `client_hash` (QR POS only, without the `aha-` prefix). Suggested app env names: `AHAPAY_BASE_URL`, `AHAPAY_API_KEY`, `AHAPAY_EXTERNAL_MERCHANT_ID` (partner only).

Conventions (Introduction page): REST over HTTPS, JSON bodies, standard HTTP methods and status codes. The Introduction mentions idempotency keys "where applicable", but no endpoint page documents an idempotency header. Not documented in official docs (checked 2026-10-21).

## Endpoint Catalogue

| Version | Method | Path | Purpose |
| --- | --- | --- | --- |
| V2 | POST | `/v2/orders/default` | Create online order (web/mobile checkout, Merchant or Partner) |
| V2 | POST | `/v2/orders/pos` | Create POS order, terminal shows `link_qr` |
| V2 | POST | `/v2/orders/qr_pos` | Create QR POS order from the customer's `client_hash` |
| V2 | GET | `/v2/orders/{online_order_id}/status` | Check order status |
| V2 | POST | `/v2/orders/{online_order_id}/refund` | Full refund of an order |
| V2 | GET | `/v2/daily_limit_status` | Whether the agreed daily sales limit is reached |
| V2 | GET | `/v2/health` | API liveness (plugin also uses it to verify the key) |
| V2 | POST (outbound) | your `callback_url` | Webhook on order status change |
| V1 | POST | `/v1/orders` | Legacy create order (online and offline, `pos_terminal` flag) |
| V1 | GET | `/v1/orders/{online_order_id}/status` | Legacy status |
| V1 | POST | `/v1/orders/{online_order_id}/refund` | Legacy full refund |
| V1 | GET | `/v1/health` | Legacy health |

No documented endpoints exist for capture, void, cancel, partial refund, recurring/tokenisation, payouts, or settlement reports (checked 2026-10-21). Payouts are made by AhaPay to the merchant bank account on an agreed schedule.

## Amount Rules

- Type `decimal`, in MYR ringgit (not sen). Examples in docs: `"1000"`, `"288.90"`, `1000`, `1500`.
- `amount` must be greater than 0.
- Only MYR is supported (WooCommerce docs: "Supported currency: Malaysian Ringgit (MYR)"; plugin is unavailable if store currency is not MYR). No `currency` field exists in V2 requests.
- The official plugin rounds and compares amounts to 2 decimal places.
- Customer purchase limits are RM50 to RM3,500, varying by merchant (merchant FAQ). The WooCommerce plugin enforces a default minimum order of 55 MYR (editable `minimum_amount` setting).
- `order_items` should add up to `amount`. The official plugin, when item prices do not sum to the order total (shipping, tax, coupons), sends one summary item instead: `{"category":["general"],"name":"WooCommerce order #<id>","price":<total>,"original_price":<total>,"discount":0,"quantity":1}`.

## Create Default Order (online)

`POST /v2/orders/default`

For online channels, Merchant direct or Partner integration. "The Customer has to be redirected to `online_order_payment_link` URL to proceed with payment via AhaPay."

### Request fields

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `amount` | decimal | Yes | Order amount in MYR |
| `order_items` | array | Yes | List of ordered items (fields below) |
| `external_merchant_id` | string | Partner only | Partner's internal Merchant ID. Required for Partner integration |
| `reference_number` | string | No | Your order reference. Use your local order ID; callbacks echo it |
| `callback_url` | string | No | Webhook URL. Set it; it is the only push notification channel |
| `is_mobile_integration` | boolean | No | `true` when integrating from a mobile app. Plugin sends `false` for web |
| `redirect_urls` | object | No | Browser return URLs (fields below) |

`order_items[]`:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `name` | string | Yes | Product name |
| `quantity` | decimal (V1 and OpenAPI: integer) | Yes | Quantity |
| `price` | decimal | Yes | Price of the product |
| `category` | array of string | Yes | Category or categories |
| `discount` | decimal | Yes | Discount applied to the product price |
| `original_price` | decimal | Yes | Original price before discount |

`redirect_urls`:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `success_redirect_url` | string | No | After successful payment |
| `failed_redirect_url` | string | No | After failed payment |
| `cart_redirect_url` | string | No | Back to the merchant cart page |

### Example request

```json
{
  "amount": 1000,
  "external_merchant_id": "merchant-123",
  "reference_number": "ref-001",
  "callback_url": "https://merchant.com/callback",
  "is_mobile_integration": false,
  "redirect_urls": {
    "success_redirect_url": "https://merchant.com/success",
    "failed_redirect_url": "https://merchant.com/failed",
    "cart_redirect_url": "https://merchant.com/cart"
  },
  "order_items": [
    {
      "name": "Product name",
      "quantity": 1,
      "price": 1000,
      "category": ["electronics"],
      "discount": 0,
      "original_price": 1000
    }
  ]
}
```

Omit `external_merchant_id` with a Merchant key.

### Order response fields (shared by all V2 create calls and status)

| Field | Type | Notes |
| --- | --- | --- |
| `online_order_id` | string | AhaPay order ID. Store it; status and refund use it |
| `order_number` | string | AhaPay order number |
| `online_order_status` | string | Status enum below. New orders return `PAYMENT_IN_PROGRESS` |
| `link` | string | V1 app deeplink per plugin comment; do not use for web redirect |
| `link_qr` | string | QR image as base64 PNG data in V2 examples (starts `iVBORw0KGgo`). V1 called it "QR code link" |
| `initial_amount` | decimal | Order amount |
| `online_order_payment_link` | string | Hosted payment page. Redirect the customer here |
| `online_order_version` | string | `V2` |
| `online_order_type` | string | `DEFAULT` in all examples (other values not documented) |
| `activation_date` | datetime (ISO 8601) | Activation time |
| `reference_number` | string | Echo of your reference |
| `redirect_urls` | object | Echo of your redirect URLs |
| `created` | datetime | Creation time |
| `sent_to_scoring` | boolean | Customer sent to credit scoring |
| `approved` | boolean | Scoring approved |
| `rejected` | boolean | Scoring rejected |

### Response envelope (uncertain)

The V2 doc examples show these fields at the top level. The official plugin v1.1.0 reads them under a `data` object (`$body['data']['online_order_id']`, `$body['data']['online_order_payment_link']`, `$data['data']['online_order_status']`, `$body['data']['success']`), and V1 docs plus `openapi.json` also wrap in `data`. Parse defensively:

```text
obj = json.data if json.data is an object else json
```

### Errors on create

HTTP errors use the status code table below. The plugin reads the message from `error.message` in the body; no other error schema is documented. The plugin treats a 4xx (except 408/409) without an `online_order_id` as a definitive decline and anything else (timeout, 5xx, 408, 409) as unknown, needing reconciliation before retry. It uses a 30 second timeout because shorter timeouts cut off order creation while AhaPay was still creating it.

## Create POS Order

`POST /v2/orders/pos`

For POS terminals that can show a QR code. "The POS terminal has to show `link_qr` QR code, so the Customer can scan it to proceed with AhaPay payment."

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `amount` | decimal | Yes | Order amount |
| `external_merchant_id` | string | Partner only | Required for Partner integration |
| `reference_number` | string | No | Order reference |
| `callback_url` | string | No | Webhook URL |

```json
{
  "amount": 1500,
  "external_merchant_id": "merchant-123",
  "reference_number": "pos-ref-001",
  "callback_url": "https://merchant.com/callback"
}
```

Response: the order response fields above.

## Create QR POS Order

`POST /v2/orders/qr_pos`

For POS terminals that scan QR codes.

Flow:

1. Customer app generates a `client_hash` with an `aha-` prefix, for example `aha-abc123xyz`, and shows it as a QR.
2. Cashier scans it; the POS sends it to the partner/merchant backend.
3. Backend calls Create QR POS Order with `client_hash` WITHOUT the prefix, for example `abc123xyz`.
4. Payment flow starts automatically in the customer's AhaPay app.

Hash lifetime is 10 minutes. Strip `aha-` (or any other prefix) before sending.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `amount` | decimal | Yes | Order amount |
| `client_hash` | string | Yes | Customer identifier, no prefix |
| `external_merchant_id` | string | Partner only | Required for Partner integration |
| `reference_number` | string | No | Order reference |
| `callback_url` | string | No | Webhook URL |

```json
{
  "amount": 2000,
  "client_hash": "abc123xyz",
  "external_merchant_id": "merchant-123",
  "reference_number": "qr-ref-001",
  "callback_url": "https://merchant.com/callback"
}
```

Response: the order response fields above.

## Order Statuses

| Status | Meaning (docs) | Plugin maps to (WooCommerce) |
| --- | --- | --- |
| `PAYMENT_IN_PROGRESS` | Order is being processed | on-hold |
| `PAYMENT_SUCCESSFUL` | Payment is successful | paid (processing/completed) |
| `CANCELLED` | Order is cancelled (plugin readme: cancelled by customer) | cancelled |
| `REFUNDED` | Order is refunded | refunded |
| `UNAVAILABLE` | Order status is unavailable (plugin readme: payment failed) | failed |

Treat `PAYMENT_SUCCESSFUL` as the only paid state. `CANCELLED` and `UNAVAILABLE` are terminal for that order attempt. Reject unknown values. The plugin allows `PAYMENT_SUCCESSFUL` to recover a cancelled or failed order (out-of-order delivery), allows `REFUNDED` out of a paid order, and refuses every other regression from a terminal state.

## Check Order Status

`GET /v2/orders/{online_order_id}/status`

| Param | In | Type | Required | Notes |
| --- | --- | --- | --- | --- |
| `online_order_id` | path | string | Yes | ID returned by a Create Order API |

Response: the order response fields above (see envelope note). "Can be used separately or along with Webhook." The plugin calls it with a 15 second timeout, reads `online_order_status`, and runs the result through the same guards as the callback. Rate limits are not documented (checked 2026-10-21); poll sparingly.

## Refund Order

`POST /v2/orders/{online_order_id}/refund`

| Param | In | Type | Required | Notes |
| --- | --- | --- | --- | --- |
| `online_order_id` | path | string | Yes | ID returned by a Create Order API |

No request body. Full refund only: V1 says "Initiate a full refund of an order", the plugin readme says "to process full refunds", and the plugin refuses partial refunds because "AhaPay refunds a whole order or nothing".

Response example (V2 docs): `{"success": true}`. The plugin checks `data.success === true`. V1 docs show `{"success_redirect_url": true}`, which looks like a doc typo for `success`. OpenAPI: `400 Refund not possible`.

Safe handling (from the plugin):

- Before sending, stamp the order with "refund started at".
- 2xx with `success: true` is the only confirmation.
- Timeout, network error, non-2xx, or a 2xx without `success: true` is an unknown outcome: do NOT resend. Requery status; `REFUNDED` means it went through. Otherwise reconcile in the AhaPay merchant portal first.
- A `REFUNDED` callback may arrive while the refund call is in flight; apply both under one per-order lock.

Merchant FAQ: a customer refund must first be approved by the merchant, then AhaPay returns the money to the customer.

## Daily Limit Status

`GET /v2/daily_limit_status`

Checks whether the daily sales limit agreed with AhaPay (if any) is reached. After the limit is hit, AhaPay rejects all orders until 12am the next day.

| Param | Type | Required | Notes |
| --- | --- | --- | --- |
| `external_merchant_id` | string | Yes | Partner's internal Merchant ID |

The docs label it a "path parameter" but the path has no placeholder; sending it as a query string (`?external_merchant_id=...`) is the likely form. Not confirmed in official docs (checked 2026-10-21); confirm with AhaPay. Merchant-key behaviour without this parameter is not documented.

Response: `{"limit_reached": false}`.

## Health Check

`GET /v2/health` (V1: `GET /v1/health`)

Response: `{"status": "ok"}`. OpenAPI documents `401 Unauthorized`, so the call needs a valid `X-ApiKey`. The plugin uses `GET /v2/health` with a 10 second timeout to verify a key on save: HTTP 200 means verified.

## Callbacks (Webhooks)

When you pass `callback_url` on order creation, AhaPay sends a POST there whenever the order status changes.

- Your endpoint must return HTTP 200 OK.
- Timeout is 65 seconds.
- Request header: `X-Api-Key-Hash: <hash of your API key>`.
- Retry policy, ordering guarantees, and source IPs are not documented (checked 2026-10-21). Design for duplicates and out-of-order delivery.

Example body (V2):

```json
{
  "online_order_id": "ord_abc123",
  "order_number": "ORD-2024-001",
  "created": "2024-01-15T10:30:00Z",
  "reference_number": "ref-001",
  "sent_to_scoring": true,
  "approved": true,
  "rejected": false,
  "initial_amount": 1000,
  "online_order_status": "PAYMENT_SUCCESSFUL",
  "activation_date": "2024-01-15T10:35:00Z"
}
```

| Field | Type | Notes |
| --- | --- | --- |
| `online_order_id` | string | Match to your stored ID |
| `order_number` | string | AhaPay order number |
| `created` | datetime | Order creation |
| `reference_number` | string | Your reference |
| `sent_to_scoring` | boolean | Scoring started |
| `approved` | boolean | Scoring approved |
| `rejected` | boolean | Scoring rejected |
| `initial_amount` | decimal | Order amount |
| `online_order_status` | string | Status enum |
| `activation_date` | datetime | Activation time |

V1 callback schema (`openapi.json` `CallbackRequest`): `online_order_id` (required), `status` (required), `amount` (string), `reference_number`, `created`, `updated`. The plugin reads `online_order_status` first and falls back to `status`.

### Callback verification

Two descriptions exist in the official sources:

1. V2 Webhook page and the official plugin v1.1.0 (current): header `X-Api-Key-Hash`, value = lowercase hex SHA-256 of the API key. Plugin code:

   ```php
   $signature = $request->get_header('x-api-key-hash');
   if (!preg_match('/^([a-f0-9]{64})$/i', $signature)) return false;
   if (!hash_equals(hash('sha256', $this->api_key), $signature)) return false;
   ```

2. Callback-Security page (generic, older): HMAC-SHA256 of the payload with a shared secret, compared to a `Signature` header. Published sample (as shown on the page, the PHP `$` signs are missing in the original):

   ```text
   received_signature = $_SERVER['HTTP_SIGNATURE'];
   calculated_signature = hash_hmac('sha256', payload, $secret_key);
   if (!hash_equals(received_signature, calculated_signature)) throw Exception('Invalid signature');
   ```

Implement method 1, which is what AhaPay's own V2 plugin ships. Optionally also accept a valid HMAC `Signature` header if AhaPay confirms it sends one for your account.

Pseudo-code:

```text
expected = hex(sha256(utf8(AHAPAY_API_KEY)))          # 64 lowercase hex chars
got = header("X-Api-Key-Hash")
if got is missing or not /^[0-9a-fA-F]{64}$/: return 401
if not constant_time_equal(expected, lower(got)): return 401
body = parse_json(raw_body)
status = body.online_order_status or body.status
order = db.find(reference_number = body.reference_number)
if order is null or order.ahapay_online_order_id != body.online_order_id: return 400
fresh = GET /v2/orders/{body.online_order_id}/status   # body is not signed
apply(fresh.online_order_status) idempotently under a per-order lock,
  only if fresh amount == order.amount (2 dp) for PAYMENT_SUCCESSFUL
return 200 {"success": true}
```

Security note: the hash is a static value per key. It proves the sender knows your key but does not sign the body, and the plugin itself notes "The hash authenticates the STORE, not the order". So check order ownership and requery status before settlement. Anyone who sees one callback header could replay it; keep callback logs masked (the plugin masks all but the last 6 characters).

The plugin's callback route: `POST /wp-json/ahapaybuynowpaylater/v1/callback`. On a lock conflict or an in-flight creation it returns 503 so AhaPay can retry; unknown status returns 400; duplicate or ignored callbacks return 200 with an `ignored` reason.

## Redirect vs Callback

- `redirect_urls` only send the browser back. They carry no documented signed parameters. Never fulfil from them.
- The callback (plus status requery) is the settlement path. Use status polling as fallback for missed callbacks.

## Retry and Duplicate Payment Attempts (from the plugin)

- An old `online_order_payment_link` can stay payable after you create a new attempt for the same local order. Keep every `online_order_id` you created for an order with its amount.
- If a second attempt reports `PAYMENT_SUCCESSFUL` after the order is already paid, do not fulfil again; flag it for reconciliation in the AhaPay portal.
- A terminal `CANCELLED` or `UNAVAILABLE` means that link is no longer payable; do not redirect customers back to it.

## HTTP Status Codes

| Code | Meaning |
| --- | --- |
| 200 | OK |
| 400 | Refund not possible (OpenAPI, refund) |
| 401 | Outdated or archived API key, or partner not bound to trade outlet |
| 404 | Trade outlet or partner not found (OpenAPI also: order not found on status) |
| 500 | Invalid API key format, or key was never created |

Error body: plugin reads `error.message`. No error code list is documented (checked 2026-10-21).

## Legacy V1 (only for existing integrations)

`POST /v1/orders` fields: `amount` (decimal, required, > 0), `reference_number`, `callback_url`, `pos_terminal` (bool, offline page only: marks physical POS transactions), `redirect_urls` (same three URLs), `order_items[]` (`name`, `price`, `quantity` integer, `category` string[] required; `discount`, `original_price` optional). OpenAPI also lists `external_merchant_id`.

V1 response (`data` envelope): `amount`, `online_order_status`, `link` (payment link), `link_qr` (QR code link), `reference_number`, `online_order_id`, `callback_url`, `redirect_urls`, `created`, `updated`, `archived` (boolean), `website_url`.

V1 status: `GET /v1/orders/{online_order_id}/status`, same `data` shape. V1 refund: `POST /v1/orders/{online_order_id}/refund`, full refund. V1 health: `GET /v1/health`. The V1 Flow diagram shows the same sequence: create, redirect, status, refund, health, optional callback.

New integrations should use V2.

## Sandbox Test Data

No public test customers, test cards, test `client_hash` values, or simulation steps are documented (checked 2026-10-21). Ask AhaPay for sandbox credentials and a test customer account during onboarding.

## Not Documented (checked 2026-10-21)

- Rate limits and IP allowlists (inbound or outbound).
- Callback retry schedule and source IPs.
- Idempotency key header, despite the Introduction mentioning one.
- Partial refunds, cancel/void, capture, recurring, tokenisation, payouts API, settlement reports.
- Error code catalogue beyond HTTP codes.
- Values of `online_order_type` other than `DEFAULT`.
- Maximum length or charset of `reference_number`.
