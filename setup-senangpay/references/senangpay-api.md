# senangPay API Reference

Use this reference for factual senangPay API guidance. Verify current dashboard settings, package capability, payment method availability, pricing, settlement timing, and support approval when those details matter.

## Source URLs

- `https://guide.senangpay.com/developer-tools`
- `https://guide.senangpay.com/manual-integration-api`
- `https://guide.senangpay.com/direct-api`
- `https://guide.senangpay.com/merchant-id-and-secret-key`
- `https://guide.senangpay.com/return-url`
- `https://guide.senangpay.com/what-is-callback-url-and-how-to-set-it`
- `https://guide.senangpay.com/query-order-status`
- `https://guide.senangpay.com/query-transaction-status`
- `https://guide.senangpay.com/refund-api`
- `https://guide.senangpay.com/testing-sandbox`
- `https://senangpay.com/social-selling/`
- `https://senangpay.com/online-store/`
- `https://senangpay.com/enterprise-solutions/`
- `https://senangpay.com/pricing/`

## Integration Surfaces

senangPay supports multiple payment surfaces:

- Hosted/manual payment form for shopping cart integrations.
- Payment forms for social selling: universal form, unique form, and product collection form.
- Shopping cart plugins and e-commerce platform integrations.
- Direct API with JavaScript Web SDK for Enterprise and authorised merchants.
- Recurring payments for instalments and subscriptions.
- Mass payments, Payout API, tokenisation, split settlement, IPP, and refund API.

Choose the simplest integration that satisfies the merchant requirement. Hosted/manual checkout is the normal custom integration path. Direct API is advanced and restricted.

## Hosted / Manual Integration API

Use this flow when a merchant has a custom shopping cart and wants to redirect customers to senangPay hosted checkout.

### Dashboard Setup

In the senangPay dashboard:

1. Go to `Settings > Profile`.
2. Find `Shopping Cart Integration Link`.
3. Copy `Merchant ID` and `Secret Key`.
4. Configure Return URL.
5. Configure Callback URL.
6. Select Hash Type. SHA256 is recommended; MD5 is legacy and not recommended.

### Payment URLs

Production:

```text
https://app.senangpay.my/payment/{merchant_id}
```

Sandbox:

```text
https://sandbox.senangpay.my/payment/{merchant_id}
```

Parameters can be sent by GET or POST.

### Request Parameters

| Parameter | Required | Notes |
| --- | --- | --- |
| `detail` | Yes | Payment description, max 500 characters. Underscores are converted to spaces. Allowed characters are letters, digits, dot, comma, dash, and underscore. |
| `amount` | Yes | Decimal amount with two decimal places, for example `25.50`. |
| `order_id` | Yes | Merchant order reference, max 100 characters. Only letters, digits, and dash. |
| `hash` | Yes | Integrity hash generated from Secret Key, detail, amount, and order ID. |
| `name` | Yes in docs | Prefills customer name. Not part of request hash. Customer may overwrite. |
| `email` | Yes in docs | Prefills customer email. Not part of request hash. Customer may overwrite. |
| `phone` | Yes in docs | Prefills customer phone. Not part of request hash. Customer may overwrite. |
| `timeout` | No | Payment timeout in seconds. Minimum is 60 seconds. Omit for no timeout. |

### Request Hash

Recommended SHA256 HMAC source sequence:

```text
{secret_key}{detail}{amount}{order_id}
```

HMAC key:

```text
{secret_key}
```

Legacy MD5 uses the same source string but should not be used unless a legacy system requires it.

### Return and Callback Parameters

senangPay sends these default fields back to merchant systems:

| Parameter | Meaning |
| --- | --- |
| `status_id` | `1` successful, `0` failed, `2` pending authorization. |
| `order_id` | Merchant order ID sent to senangPay. |
| `msg` | Status message, may contain underscores. |
| `transaction_id` | senangPay transaction ID. |
| `hash` | Integrity hash from senangPay. |

Verification hash source sequence:

```text
{secret_key}{status_id}{order_id}{transaction_id}{msg}
```

Use the account's configured Hash Type. Prefer SHA256 HMAC. Treat mismatched hashes as tampered data and do not settle.

## Return URL

Return URL is browser-facing. Use it to show the customer a result page, not to unlock paid access.

Return URL parameters can be customized only when the receiving system needs non-default field names. If the default parameters work, leave Return URL Parameters empty. senangPay says WooCommerce and OpenCart plugin users should leave the field empty.

Supported placeholders include:

- `[NAME]`
- `[EMAIL]`
- `[PHONE]`
- `[AMOUNT]`
- `[TXN_STATUS]`
- `[ORDER_ID]`
- `[TXN_REF]`
- `[MSG]`
- `[HASH]`
- `[TXN_TYPE]`

Return URL parameter customization also applies to Callback URL. For custom parameter hashes, URL encode values and calculate the hash over the configured query string appended to the merchant Secret Key. MD5 is deprecated; SHA256 is recommended.

## Callback URL

Callback URL is an alternative/backend notification for shopping cart integrations and is recommended for data integrity.

Key behavior:

- senangPay sends callback data by HTTP POST.
- Callback sends the same parameters as Return URL unless custom Return URL Parameters are configured.
- Callback URL must output exactly `OK` without HTML tags when successful.
- Avoid HTTP redirects; 302 can cause callback failure.
- Callback URL must not depend on sessions.
- Non-secure HTTP callback ports may need whitelisting by senangPay support.
- Multiple callbacks can arrive for the same transaction.
- If a transaction is pending, callbacks can start 5 minutes after transaction start.
- The last callback is 1 hour after transaction start.
- A failed status can be followed by a successful status because earlier bank queries may not be final.
- The latest callback data is considered the most up-to-date transaction data.

Implementation rule: record callback events durably, verify hash, process idempotently, then return `OK` only after the durable path succeeds.

## Query APIs

Use query APIs to re-check status from the backend before settlement or when callback/return ordering is unclear.

### Query Order Status

Endpoint:

```text
GET https://app.senangpay.my/apiv1/query_order_status
```

Parameters:

- `merchant_id`
- `order_id`
- `hash`

Hash source sequence:

```text
{merchant_id}{secret_key}{order_id}
```

For SHA256, use HMAC-SHA256 with Secret Key as the HMAC key. Docs also show MD5 for legacy hash type.

### Query Transaction Status

Endpoint:

```text
GET https://app.senangpay.my/apiv1/query_transaction_status
```

Parameters:

- `merchant_id`
- `transaction_reference`
- `hash`

Hash source sequence:

```text
{merchant_id}{secret_key}{transaction_reference}
```

For SHA256, use HMAC-SHA256 with Secret Key as the HMAC key. Docs also show MD5 for legacy hash type.

## Direct API

Direct API is an advanced flow for Enterprise package users and authorised merchants only. Merchants must contact senangPay and whitelist the server outgoing IP address.

Direct API has two parts:

1. Server creates a Client Session.
2. Browser uses JavaScript Web SDK with the returned UUID.

### Client Session Endpoint

Sandbox:

```text
POST https://api.sandbox.senangpay.my/payment/v3
```

Production:

```text
POST https://api.senangpay.my/payment/v3
```

Authentication:

- Basic Auth username: Merchant ID.
- Basic Auth password: blank.

Parameters include:

| Parameter | Required | Notes |
| --- | --- | --- |
| `payment_method` | Yes | Values include `fpx`, `boost`, `tng`, `grabpay`, `grabPl`, `shopeepay`, `shopback`, `cc`, `jcb_direct`. |
| `fpx_bank_code` | Conditional | Required when `payment_method` is `fpx`. |
| `shopeepay_platform` | Conditional | Required when `payment_method` is `shopeepay`; `desktop` or `mobile`. |
| `filter_card` | No | Optional array for card filters. |
| `customer_name` | Yes | Customer name. |
| `customer_email` | Yes | Customer email. |
| `customer_phone` | Yes | Customer phone. |
| `order_id` | Yes | Unique merchant transaction reference. |
| `amount` | Yes | Amount multiplied by 100. Example `200` means RM 2.00. |
| `detail` | Yes | Payment description. |
| `hash` | Yes | SHA256 hash as documented. |

Hash source sequence:

```text
{secret_key}{customer_name}{customer_email}{customer_phone}{order_id}{amount}{detail}{payment_method}
```

Response includes:

- `result`: `1` successful, `0` failed.
- `msg`: description.
- `uuid`: client token for JavaScript Web SDK.

The UUID is valid for only 60 seconds.

### JavaScript Web SDK

Include jQuery and senangPay SDK assets.

Production JS:

```text
https://senangpay.my/sdk/senangpay_v1.0.1.min.js
```

Sandbox JS:

```text
https://senangpay.my/sdk/senangpay_v1.0.1_sandbox.min.js
```

CSS:

```text
https://senangpay.my/sdk/senangpay_v1.0.1.min.css
```

HTML target:

```html
<div id="senangpay_payment"></div>
```

JavaScript:

```js
const senangpay = new Senangpay(merchantid);
senangpay.pay(uuid);
```

Do not generate hashes or expose Secret Key in browser code.

### FPX Bank List API

Production:

```text
GET https://api.senangpay.my/fpx_bank_list
```

Sandbox:

```text
GET https://api.sandbox.senangpay.my/fpx_bank_list
```

Authentication uses Basic Auth with Merchant ID as username and blank password.

Response contains `fpx_bank_list` entries with:

- `fpx_bank_code`
- `name`
- `status`: active or inactive
- `type`: b2b or b2c
- `logo`

Do not show inactive banks as selectable.

## Refund API

Base URLs:

```text
https://api.senangpay.my
https://api.sandbox.senangpay.my
```

Get token endpoint:

```text
/merchant/get_token
```

The guide shows merchant ID in the `Authorization` header for token generation.

Create refund endpoint:

```text
POST /merchant_service/create_refund_request
```

Use token header:

```text
Authorization: Bearer <token>
```

JSON body:

```json
{
  "transaction_reference": "1729057564864250",
  "refund_amount": "2"
}
```

Successful response creates a refund with fields such as `refund_reference_no`, `amount`, `type`, `status`, `message`, and `approval_code`.

Documented invalid responses include:

- Transaction does not exist for merchant.
- Refund request for transaction reference already exists.
- Partial refund is not supported.
- Requested refund amount exceeds transaction amount.
- Transaction date exceeds allowed refund window: 180 days.
- Authentication failed.

Use local refund idempotency. Do not retry blindly when the provider reports duplicate or unsupported refund state.

## Product and Package Caveats

Public pages describe these products:

- Social selling: online payment form, digital payments, mobile/tap-to-phone payments.
- Online store: shopping cart plugins, e-commerce integrations, Direct API, recurring payments, mass payments, IPP.
- Enterprise: Payout API, tokenisation, split payments, custom rates, advanced technical functions.

Feature availability can depend on package, add-ons, channel activation, dashboard state, and senangPay approval. Direct API is Enterprise and authorised only. Always verify current package and dashboard before promising availability.

## Settlement Checklist

Before marking a local payment as paid:

1. Local payment exists and is pending or in a transition-safe state.
2. Hash verifies with the configured Hash Type and exact source sequence.
3. `order_id` maps to exactly one local payment.
4. Amount matches expected local amount with the right unit for the active flow.
5. Status is successful and final enough for the business rule.
6. Provider transaction ID/reference is stored.
7. Environment matches local config: sandbox vs production.
8. Callback/query event is recorded durably.
9. Idempotency guard prevents duplicate fulfilment.
10. Return URL was not used as the sole authority.
