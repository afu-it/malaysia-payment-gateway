# OxPay API Reference

Use this reference for factual OxPay Payments API guidance. It is distilled from the official OxPay docs site and its OpenAPI file (version 2.0.2), checked 2026-10-21. Verify account-specific items (enabled brands, MYR support, terminal features) with OxPay before production.

## Source URLs

- `https://docs.oxpayfinancial.com/`
- `https://docs.oxpayfinancial.com/getting-started`
- `https://docs.oxpayfinancial.com/api-reference` (Redoc page rendered from the OpenAPI file below)
- `https://docs.oxpayfinancial.com/redocusaurus/oxpay-payments-api-reference.yaml` (OpenAPI 3.0.0, the machine-readable source for every field table here)
- `https://docs.oxpayfinancial.com/security`
- `https://docs.oxpayfinancial.com/credentials`
- `https://docs.oxpayfinancial.com/integrity`
- `https://docs.oxpayfinancial.com/responses/http-status-codes`
- `https://docs.oxpayfinancial.com/responses/operation-status`
- `https://docs.oxpayfinancial.com/responses/reason-codes`

## Scope Note For Malaysia

The public API docs are regional (OxPay SG Pte Ltd). All examples use `SGD`. The documented payment brands are card, Alipay, WeChat Pay, GrabPay, ShopeePay, Atome, Dash, PayNow and UPI. FPX, DuitNow QR, Touch 'n Go eWallet, Boost and Grab PayLater appear on the OxPay Malaysia marketing site but have no request schema in the public API docs (checked 2026-10-21). For a Malaysian merchant, send `currency: "MYR"` with amounts in sen only after OxPay confirms the terminal supports MYR and the brands you need.

## Integration Surfaces

From Getting Started:

1. **Hosted Payment Page (gateway hosted).** Call `POST /payment-page`, redirect the payer to `oxpay_hpp_url`. Payer picks a method on the OxPay page (scan QR, or enter card on the OxPay Secure Payment Page). OxPay sends the result to the merchant. Simplest, keeps you out of PCI scope. The terminal must have the "Ecommerce" feature enabled.
2. **Create Payment (merchant hosted).** Merchant collects payment details and calls `POST /payment`. More UX control. Requires PCI DSS compliance when you handle card data.

Other OxPay products named in the merchant portal guide (OxPay Terminal, eCommPay, OxPay Lite, Click2Pay payment links, Scan2Pay QR, Payout) are portal or app products; only Payout has a public API.

## Environments

| Environment | Base URL | Source |
| --- | --- | --- |
| Live | `https://api.oxpayfinancial.com` | OpenAPI `servers` |
| Sandbox | `https://api.uat.oxpayfinancial.com` | OpenAPI `servers` |

Conflict: Getting Started section 2 says "Merchant proceeds with production testing (note: no UAT environment available)" and section 1.3 says "Set up a live test environment". OxPay provides test cases; you execute them and submit results for approval before go-live. Ask OxPay whether the UAT host is usable for your account. No public test cards or test wallets are published.

## Required Headers

| Header | Required | Applies to | Notes |
| --- | --- | --- | --- |
| `Authorization` | yes | all endpoints | `Bearer <API_KEY>`. Example format `Bearer ak-ec00df37-...`. |
| `Signature` | yes | `POST /payment-page`, `POST /payment`, `POST /void`, `POST /refund`, `POST /payouts` | Lowercase hex HMAC-SHA256 of the request body. See Signature. |
| `User-Agent` | yes | all endpoints | Syntax `<product> / <product-version> <comment>`, example `MyApp/1.0.0 (compatible; OxPay Integration)`. |
| `OxPay-Version` | no | all endpoints | Current version `2.0.1`. Helps OxPay support. |
| `Content-Type` | implied | POST | `application/json` (all request bodies are JSON). |

Responses carry `OxPay-Version` and `Vary` (default `Origin, OxPay-Version`) headers.

## Signature (Integrity)

Source: `https://docs.oxpayfinancial.com/integrity` (Python, Java and PHP samples), `/credentials`, OpenAPI `components.parameters.Signature` and `x-webhooks.*.parameters`.

Algorithm:

1. Take the JSON payload.
2. Remove all line breaks and whitespace in the structure only (not inside string values). Do not change key order. This equals Python `json.dumps(obj, separators=(",", ":"))` on an order-preserving parse, Java Jackson `writeValueAsString(readTree(json))`, PHP `json_encode(json_decode($s), JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES)`.
3. `signature = hex(HMAC_SHA256(key = Sign Key as UTF-8 bytes, message = compact JSON as UTF-8 bytes))`, lowercase hex, 64 chars.
4. Send it in the `Signature` HTTP header. The body you send must be that same compact string.

There is no timestamp, nonce or field concatenation; the whole body is the string to sign.

Webhooks: every webhook (`payment`, `refund`, `void`, `payout`, `compliance_check`) carries a `Signature` header "to make sure the integrity of the payload". OxPay advises merchants to validate it with the Sign Key using the same Integrity method.

Pseudo-code:

```text
SIGN_KEY = env.OXPAY_SIGN_KEY
body = compactJson(payload)            # serialize once, keep key order, no spaces
sig  = hex(hmac_sha256(SIGN_KEY, body)).lower()
POST {base}/payment-page
  Authorization: Bearer {OXPAY_API_KEY}
  Signature: {sig}
  User-Agent: MyShop/1.0.0 (OxPay Integration)
  Content-Type: application/json
  <body bytes exactly as signed>

# webhook
raw = request.raw_body
expected = hex(hmac_sha256(SIGN_KEY, compactJson(parseOrdered(raw))))
if !constant_time_equal(expected, request.headers["Signature"].lower()): reject 401
```

Gotchas:

- The docs samples disagree on non-ASCII: Python `json.dumps` defaults to `ensure_ascii=True` (escapes to `é`), while the PHP sample keeps raw UTF-8 and unescaped slashes. Keep signed fields ASCII where possible, and for webhooks also compare against HMAC of the raw body as received. Confirm the exact canonical form with OxPay if you need non-ASCII names or descriptions.
- PHP `json_decode` without `true` keeps object order; do not sort keys in any language.
- Locally computed check vector (not published by OxPay): key `test-sign-key`, message `{"field_one":"value one","field_three":"value three","field_two":"value two"}` gives `aad79f4c4e2bbc91ea5cd6c7c851fbcdd02abe92de5503b32fe31a7f6b8d1bc6`.
- Whether OxPay signs synchronous API responses is not documented (the header description says "Request/Response Header" but no response header is defined). Do not depend on it.

## Response Envelope

Every response and every payment/refund/void webhook is:

```json
{
  "headers": { "status_code": 0, "status_message": "SUCCESS", "gateway_response_datetime": "20240103101433" },
  "body": { ... }
}
```

- `headers.status_code` (int): see Operation Status.
- `headers.status_message` (string): e.g. `SUCCESS`, `PENDING`, `REJECTED`.
- `headers.gateway_response_datetime` (string): examples use `YYYYMMDDHHmmss` although payment schemas say `date-time`.
- Error body on HTTP 400 (payment endpoints): `body.status` (e.g. `Rejected`) and `body.status_desc`, e.g. `The following fields from Payload's body are invalid: ['merchant_reference_id field not found in payload'].`

Exception: `GET /payment?session_id=` returns the session fields at the root with no headers/body wrapper (per its schema and example).

## Endpoint Catalogue

| # | Method | Path | operationId | Purpose |
| --- | --- | --- | --- | --- |
| 1 | POST | `/payment-page` | payment-page | Get Hosted Payment Page URL |
| 2 | POST | `/payment` | create-payment | Create payment (merchant hosted card, or QR wallet) |
| 3 | POST | `/void` | cancel-payment | Void a completed sale or cancel a pending QR transaction |
| 4 | POST | `/refund` | refund-payment | Refund a successful sale |
| 5 | GET | `/payment/{oxpay_txn_id}` | get-payment-by-id | Query payment by OxPay transaction ID |
| 6 | GET | `/payment?session_id={session_id}` | get-payment-by-session-id | Query hosted session, includes available brands |
| 7 | GET | `/payment?merchant_reference_id={merchant_reference_id}` | get-payment-by-merchant-reference-id | Query payment by your reference |
| 8 | POST | `/payouts` | create-payout | Create payout (NRIC or bank account) |
| 9 | GET | `/payouts/{unique_reference}` | check-payout-status | Payout status |
| W1 | POST | your `notification_url` | payment-notification | Sale result webhook |
| W2 | POST | your `notification_url` | refund-notification | Refund result webhook |
| W3 | POST | your `notification_url` | void-notification | Void/cancel result webhook |
| W4 | POST | your payout `notification_url` | payout-notification | Payout result webhook |
| W5 | POST | your payout `notification_url` | payout-compliance-check-notification | Payout compliance-check failed |

Not documented: capture endpoint (despite `captured` states and a `CapturePaymentSuccessResponse` schema), recurring/tokenisation, settlement report API, payment links API, list/search transactions. Settlement, daily and monthly reports are downloadable in the merchant portal only.

## Shared Field Types

| Field | Type | Rules |
| --- | --- | --- |
| `merchant_id` | string | max 9. Provided on onboarding (MID). |
| `terminal_id` | string | exactly 14. Provided on onboarding (TID). |
| `merchant_reference_id` | string | 1 to 100. Your unique ID, must be unique per terminal. |
| `oxpay_session_id` | string | min 50. From `/payment-page`. |
| `oxpay_txn_id` | string | Generated by OxPay on successful processing. |
| `oxpay_original_txn_id` | string | OxPay ID of the transaction being voided/refunded. |
| `host_txn_id` | string | Acquirer reference per operation. |
| `amount.value` | integer (number, format integer) | min 1, smallest currency unit, no decimal point (SGD 1 = 100; MYR 1 = 100 sen). |
| `amount.currency` | string | 3 chars, ISO 4217 alpha-3. |
| `source.name` / `card_info.name` | string | max 100, payer name. |
| `source.email` | string | max 254. |
| `source.phone` | string | max 16. |
| `card_info.card_number` | string | max 16. |
| `card_info.expiry_month` | string | max 2, e.g. `"12"`. |
| `card_info.expiry_year` | string | max 4, e.g. `"2025"`. |
| `card_info.cvv` | string | max 4. |

## 1. POST /payment-page (Hosted Payment Page)

Terminal must be enabled with the Ecommerce feature. Headers: Authorization, Signature, User-Agent.

Request:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `merchant_id` | string | yes | MID |
| `terminal_id` | string | yes | TID, 14 chars |
| `merchant_reference_id` | string | yes | unique per terminal |
| `description` | string | yes | short description |
| `success_url` | string | yes | browser redirect after success |
| `failure_url` | string | yes | browser redirect after failure |
| `notification_url` | string | yes | webhook URL for status updates |
| `amount` | object | yes | `{ value, currency }` |
| `source` | object | yes | `{ name, email, phone }` all required |
| `capture` | boolean | no | card only; `false` = pre-auth |

Example:

```json
{"merchant_id":"<MID>","terminal_id":"<TID_14_CHARS>","merchant_reference_id":"ORDER-1001","description":"Order 1001","success_url":"https://shop.example/pay/success","failure_url":"https://shop.example/pay/failure","notification_url":"https://shop.example/webhooks/oxpay","amount":{"value":200,"currency":"MYR"},"source":{"name":"Sarah","email":"sarah@example.com","phone":"60123456789"},"capture":true}
```

Response 200: `body.oxpay_hpp_url` (string, max 2048), a URL containing `session_id`. Redirect the browser there. Response 400: `body.status`, `body.status_desc`.

## 2. POST /payment (Create Payment)

Outcome is delivered asynchronously via the payment webhook. Headers: Authorization, Signature, User-Agent.

Request:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `merchant_id` | string | yes | |
| `terminal_id` | string | yes | |
| `merchant_reference_id` | string | yes | |
| `oxpay_session_id` | string | no | session from `/payment-page` when completing a hosted session |
| `success_url` | string | yes | |
| `failure_url` | string | yes | |
| `notification_url` | string | yes | |
| `amount` | object | yes | `{ value, currency }` |
| `payment_method` | object | yes | discriminated by `brand`, see below |

`payment_method` variants (discriminator `brand`):

| brand value | Fields |
| --- | --- |
| `card` (or omitted) | `card_info` (required: `name`, `card_number`, `expiry_month`, `expiry_year`, `cvv`), `capture` (bool, optional, false = pre-auth). `brand` is optional for card. |
| `alipay` | `brand` (req), `payment_channel` (req, `MPQR` or `CPQR`), `source` (req: `name`, `email`, `phone`; plus `consumer_identifier` string exactly 18 chars, required when `CPQR`) |
| `wechat pay` | same as alipay (MPQR or CPQR, `consumer_identifier` for CPQR) |
| `grabpay` | `brand`, `source {name,email,phone}` |
| `shopeepay` | `brand`, `source` |
| `atome` | `brand`, `source` |
| `dash` | `brand`, `source` |
| `PayNow` | `brand`, `source` (channel enum for PayNow is `MPQR` only) |
| `upi` | `brand`, `source` |

MPQR = merchant-presented QR (you display `qr_string`). CPQR = customer-presented QR (you scan the payer's code and pass it as `consumer_identifier`). Brand strings are case and space sensitive (`wechat pay`, `PayNow`).

Card example:

```json
{"merchant_id":"<MID>","terminal_id":"<TID>","merchant_reference_id":"ORDER-1002","success_url":"https://shop.example/pay/success","failure_url":"https://shop.example/pay/failure","notification_url":"https://shop.example/webhooks/oxpay","payment_method":{"card_info":{"name":"Sarah","card_number":"<PAN>","expiry_month":"12","expiry_year":"2028","cvv":"<CVV>"},"capture":true},"amount":{"currency":"MYR","value":200}}
```

Wallet example: `"payment_method":{"brand":"grabpay","source":{"name":"Sarah","email":"sarah@example.com","phone":"60123456789"}}`.

Response 200 body by brand:

| Variant | Fields |
| --- | --- |
| Card | `oxpay_txn_id` (req), `merchant_reference_id` (req), `payment_brand` (opt). Example status_code 0. |
| QR wallets (alipay, wechat pay, grabpay, shopeepay, atome, dash, PayNow, upi) | `payment_brand`, `oxpay_txn_id`, `merchant_reference_id`, `transaction_amount` (number), `transaction_currency`, `host_txn_id`, `qr_string` (MPQR string from acquirer), `payment_expiry_time` (Unix timestamp string, e.g. `"1704775857"`). Example status_code 1 PENDING. |
| CPQR (alipay, wechat pay) | Same IDs plus `transaction_state` (enum `success, requested, pending, rejected, error, expired, voided`). Example status_code 0 with `transaction_state: success`. No `qr_string`. |

A schema also exists for a 3DS host-to-host card variant with `redirect_url`, `request_type` (e.g. `REDIRECT`), `request_data.paRes`, `payment_expiry_time`, but it is not wired to any endpoint response. If you receive `redirect_url`, redirect the payer; confirm with OxPay.

Response 400: `body.status`, `body.status_desc`.

## 3. POST /void (Void / Cancel)

Voids a completed sale or cancels a pending QR transaction. Result via void webhook. Headers: Authorization, Signature, User-Agent.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `merchant_id` | string | yes | |
| `terminal_id` | string | yes | |
| `oxpay_original_txn_id` | string | yes | the sale's `oxpay_txn_id` |
| `merchant_reference_id` | string | yes | |
| `notification_url` | string | yes | |
| `amount` | object | yes | `{ value, currency }` |

Response 200 body: `oxpay_original_txn_id`, `oxpay_txn_id` (new void txn), `merchant_reference_id`, `payment_brand`, `transaction_amount`, `transaction_currency`, `host_txn_id` (card example returns only IDs and `transaction_amount`). Void success means the consumer is not debited and funds are not credited to you.

## 4. POST /refund

Refunds a successful sale. Result via refund webhook. Request fields are identical to `/void` (`merchant_id`, `terminal_id`, `oxpay_original_txn_id`, `merchant_reference_id`, `notification_url`, `amount`, all required). Partial refund rules are not documented; `amount` is sent explicitly.

Response 200 body: `oxpay_original_txn_id`, `oxpay_txn_id` (refund txn), `merchant_reference_id`, `payment_brand`, `refund_amount`, `transaction_currency`, `host_txn_id`.

## 5 and 7. GET /payment/{oxpay_txn_id} and GET /payment?merchant_reference_id=

Headers: Authorization, User-Agent. Response 200 body:

| Field | Required | Notes |
| --- | --- | --- |
| `oxpay_txn_id` | yes | |
| `merchant_reference_id` | yes | |
| `host_txn_id` | yes | |
| `transaction_state` | yes | TransactionState enum |
| `payment_brand` | no | |
| `transaction_amount` | yes | smallest unit |
| `transaction_currency` | yes | |
| `approval_code` | no | card only |

A generic `QueryPaymentResponse` schema also lists `transaction_type` (`SALE`, `VOID`, `REFUND`, `CAPTURE`). Error example: HTTP 400, `status_desc: "The requested transaction id is invalid."`.

## 6. GET /payment?session_id=

Returns session details and the list of brands available. Sessions expire; request a new one via `/payment-page`. Root-level fields: `merchant_id`, `merchant_name`, `terminal_id`, `transaction_id` (null until paid), `expiry_datetime` (e.g. `2024-01-18T05:19:23+0000`), `merchant_reference_id`, `transaction_currency`, `transaction_amount`, `description`, `payer_name`, `payer_email`, `payer_phone`, `payer_return_url`, `payer_error_url`, `payer_notification_url`, `payer_capture` (bool), `payment_brand_list[]` of `{ brand_name, method_name: QR | CARD }`. Expired: HTTP 400 `status_desc: "Transaction has expired."`.

## Webhooks (Payment, Refund, Void)

Delivered as POST JSON to the `notification_url` you sent, with a `Signature` header. Same `{headers, body}` envelope.

Payment webhook body: `merchant_reference_id`, `oxpay_txn_id`, `payment_brand` (absent for card), `transaction_amount`, `transaction_currency`, `transaction_state`, `host_txn_id`, `approval_code` (card).

```json
{"headers":{"status_code":0,"status_message":"SUCCESS","gateway_response_datetime":"20240111071316"},"body":{"merchant_reference_id":"789","oxpay_txn_id":"1000001","host_txn_id":"b8e4c9205a104d97bf7799cc016613bf","transaction_state":"success","payment_brand":"grabpay","transaction_amount":100,"transaction_currency":"SGD"}}
```

Refund webhook body: `merchant_reference_id`, `oxpay_original_txn_id`, `oxpay_txn_id`, `transaction_state` (`refunded`), `payment_brand`, `refund_amount`, `transaction_amount`, `transaction_currency`, `host_txn_id`.

Void webhook body: `merchant_reference_id`, `oxpay_txn_id`, `transaction_state` (`voided`), `payment_brand`, `transaction_amount`, `transaction_currency`, `host_txn_id`.

Not documented (checked 2026-10-21): retry schedule, expected HTTP response code or body, delivery timeout, source IP list, event ID for payment webhooks. Respond 200 quickly after verifying and persisting, and use requery as the safety net.

## Status Codes

HTTP (from `/responses/http-status-codes`): 200 OK processed; 400 malformed or schema violation (also used for Rejected/Error); 401 token not valid; 404 resource not found; 429 rate limit exceeded; 500 internal error; 503 not ready, retry later.

Operation status `headers.status_code` (from `/responses/operation-status`):

| Code | Status | Meaning |
| --- | --- | --- |
| 0 | Success | Accepted and completed by the gateway |
| 1 | Pending | Accepted, waiting for completion (e.g. acquirer callback) |
| 2 | Rejected | Not accepted, incorrect or missing payload data (HTTP 400) |
| 3 | Error | Gateway error while processing (HTTP 400), contact support |

Reason codes (from `/responses/reason-codes`) apply the same 0/1/2/3 to Sale, Refund and Void. Next actions: 0 none; 1 wait for acquirer, status may take a while; 2 read `status_message`/`status_desc` for the bad or missing field; 3 contact OxPay support.

`transaction_state` enum: `success`, `requested`, `authorized`, `pending`, `rejected`, `error`, `expired`, `voided`, `refunded`, `captured`, `capture_submitted`, `capture_rejected`.

Settle a sale as paid only on `transaction_state = success` (or `captured` for a captured pre-auth) with `status_code = 0`. Treat `authorized` as pre-auth only.

`payment_brand` enum: `alipay`, `atome`, `card`, `dash`, `grabpay`, `PayNow`, `shopeepay`, `wechat pay` (plus `upi` in the discriminator and examples). Case varies in examples (`GrabPay`, `paynow`); compare case-insensitively.

## Payouts

`POST /payouts` (headers Authorization, Signature, User-Agent). Outcome via payout webhooks. Examples are Singapore-only (SG nationality, NRIC, SGD, DBS SWIFT).

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `payout_method` | string | yes | `NRIC` or `BANK_ACCOUNT` |
| `unique_reference` | string | yes | max 32, your ID |
| `merchant_id` | string | yes | max 9 |
| `transfer_amount` | integer | yes | min 1, smallest unit |
| `currency` | string | yes | ISO 4217 |
| `notification_url` | string (uri) | yes | |
| `remarks_to_recipient` | string | yes | |
| `recipient` | object | yes | `first_name`, `last_name`, `nationality` (country code), `date_of_birth` (YYYY-MM-DD), `addresses`, `postal_code`, `address_city` (all required) |
| `destination` | object | per method | NRIC: `nric_number`. BANK_ACCOUNT: `bank_account_number`, `bank_code`, `bank_swift_code` |

Response 200: `headers` (`status_code`, `status_message`, `gateway_response_datetime` YYYYMMDDHHmmss) and body `transaction_id` (e.g. `PS0000000027`), `unique_reference`, `transfer_amount`, `transfer_fee` (number, e.g. 4.2), `currency`, `compliance_check_status`, `payout_status`, `reason_code`, `reason_message` (only on non-success). Response 400 body: `unique_reference`, `reason_code`, `reason_message`. Documented reason codes: `DUPLICATE_REFERENCE`, `TRANSACTION_NOT_FOUND`, `COMPLIANCE_CHECK_FAILED`, `INVALID_ACCOUNT`.

`GET /payouts/{unique_reference}` (Authorization, User-Agent): same body fields as above.

Payout webhook: `event_id` (dedupe key, e.g. `evt_...`), `event_type: payout`, `transaction_id`, `unique_reference`, `status` (`COMPLETED` or `REJECTED`), `status_code` (0 Completed, 1 Rejected, 4 Failed), `reason_code`, `reason_message`.

Compliance-check webhook: same fields with `event_type: compliance_check`, `status: FAIL`, `status_code` 4, `reason_code` and `reason_message` required.

## Rate Limits, IPs, Security

- Rate limiting is enforced (Security page; HTTP 429). Numbers are not documented.
- No IP allowlist for API or webhooks is documented.
- All traffic is HTTPS. OxPay is PCI DSS certified; merchant-hosted card capture puts PCI DSS on the merchant.

## Common Errors

Meanings come from the status pages above; the "likely cause" pairings are troubleshooting guidance, not OxPay text.

- `401`: wrong or deleted API key, or missing `Bearer ` prefix.
- `400` status_code 2 with `... field not found in payload`: missing required field; read `status_desc`.
- Signature rejected: body re-serialized after signing, pretty-printed JSON, wrong Sign Key, keys reordered, unicode escaping mismatch.
- `Transaction has expired.`: hosted session expired; create a new `/payment-page` session.
- `The requested transaction id is invalid.`: wrong `oxpay_txn_id` or reference, or wrong environment.
- `/payment-page` fails for a valid payload: check with OxPay that the terminal has the Ecommerce feature (documented prerequisite).
