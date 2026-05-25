# Billplz Reference Notes

Sources read on 2026-05-24:

- Help center home: `https://help.billplz.com/portal/en/home`
- Knowledge base: `https://help.billplz.com/portal/en/kb`
- API reference: `https://www.billplz.com/api#introduction`
- Integration knowledge base: `https://help.billplz.com/portal/en/kb/integration`
- GitHub organization: `https://github.com/billplz`
- Status page: `https://status.billplz.com/`
- Service check page: `https://check.billplz.com/`
- FPX status page: `https://fpxstatus.billplz.com/`
- About: `https://main.billplz.com/about`
- Security: `https://main.billplz.com/security`
- Pricing: `https://main.billplz.com/pricing`
- Collection product page: `https://main.billplz.com/collection`
- Payment Order product page: `https://main.billplz.com/payment-order`
- Payment Form: `https://catalog.billplz.com/payment-form`
- Catalog: `https://catalog.billplz.com/`
- Zakat: `https://zakat.billplz.com/`

## Environment And Authentication

- Production API endpoint: `https://www.billplz.com/api/`
- Sandbox API endpoint: `https://www.billplz-sandbox.com/api/`
- Auth: HTTPS Basic Auth.
- Username: API Secret Key.
- Password: blank. Curl examples use `-u API_SECRET_KEY:`.
- API accepts `application/x-www-form-urlencoded` and JSON.
- Responses are JSON, including errors.
- API requests must use HTTPS.

## API Versions

| Version | Current use | Notes |
| --- | --- | --- |
| V3 | Bills, Collections, Transactions, Payment Methods, FPX Banks | Stable; no new features. |
| V4 | V3-compatible flows plus 2-recipient split rules, receipt delivery, webhook rank, payment gateway list, and card tokenization | Active development. |
| V5 | Payment Order payouts | Requires `epoch` and HMAC-SHA512 `checksum` on each request. |

## Main Bill Collection Flow

1. Customer chooses to pay.
2. Merchant creates Bill by API.
3. Billplz returns Bill URL.
4. Merchant redirects customer to Bill URL.
5. Customer pays through selected option.
6. Billplz sends backend update to `callback_url`.
7. If `redirect_url` exists, Billplz redirects browser back to merchant site.
8. Merchant settles local payment only after verified server-side state.

Important:

- `callback_url` integration is compulsory.
- `redirect_url` is optional and improves user experience.
- `redirect_url` and `callback_url` execution order is not fixed.
- Handlers must be idempotent.
- Billplz API only accepts MYR and does not do currency conversion.
- Amounts use smallest currency unit, for example `100` equals RM 1.00.

## Collections And Bills

- Each Bill must be created inside a Collection.
- Get Collection ID from Collection page in Billplz account.
- Open Collection cannot create a Bill by API.
- Creating a Bill sets its collection to active automatically.
- Default Bill expiry is 30 days.
- Standard plus Open Collections have an account lifetime limit; avoid creating disposable collections for every checkout.

Create Collection endpoint:

```text
POST https://www.billplz.com/api/v3/collections
```

Required field: `title`.

Optional V3 fields include `logo` and single-recipient `split_payment[...]` fields. V4 uses `split_payments[]` for up to two split recipients and removes V3 custom logo upload.

Open Collection endpoints:

```text
POST https://www.billplz.com/api/v3/open_collections
POST https://www.billplz.com/api/v4/open_collections
GET  https://www.billplz.com/api/v4/open_collections/{COLLECTION_ID}
```

Use Open Collections for payment forms/catalog-style payment pages. Do not use them when the app needs to create a Bill by API.

Create Bill endpoint:

```text
POST https://www.billplz.com/api/v3/bills
```

Important fields:

- `collection_id`: Collection ID.
- `description`: shown on bill template.
- `email`: recipient email; required if `mobile` is absent.
- `mobile`: recipient mobile with country code; required if `email` is absent.
- `name`: recipient name.
- `amount`: positive integer in cents.
- `callback_url`: webhook URL called after transaction completes.
- `redirect_url`: optional browser redirect after payment.
- `due_at`: informational due date; does not make an unpaid bill unpayable.
- `deliver`: optional email/SMS delivery flag.
- `reference_1_label`, `reference_1`, `reference_2_label`, `reference_2`: optional merchant references.
- `payment_button`: optional payment button customization in docs.

Response includes:

- `id`
- `collection_id`
- `paid`
- `state`
- `amount`
- `paid_amount`
- `due_at`
- `email`
- `mobile`
- `name`
- `url`
- `redirect_url`
- `callback_url`
- `description`
- `paid_at`

Other Bill endpoints:

```text
GET    https://www.billplz.com/api/v3/bills/{BILL_ID}
DELETE https://www.billplz.com/api/v3/bills/{BILL_ID}
GET    https://www.billplz.com/api/v3/bills/{BILL_ID}/transactions
```

Use callback/redirect signatures for normal settlement instead of polling `GET /bills/{id}`. Only `due` bills can be deleted; paid bills return an error.

## Direct Payment Gateway

Billplz supports direct payment gateway flow:

- Merchant creates Bill with bank/payment gateway code.
- Merchant redirects payer to Bill URL.
- Append `?auto_submit=true` to skip Billplz payment option selection page when valid.
- If inactive or invalid payment gateway is submitted, payer falls back to Billplz payment option selection page.
- Use `Get Payment Gateways` and payment gateway abbreviations/bank codes from API docs for exact supported codes.

Do not hard-code current bank/payment gateway codes without verifying docs or live response.

## Payment Methods And Gateways

Payment Methods endpoints:

```text
GET https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/payment_methods
PUT https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/payment_methods
```

`PUT` enables only the submitted method codes; omitted codes are disabled.

FPX Banks endpoint:

```text
GET https://www.billplz.com/api/v3/fpx_banks
```

Pull on an hourly basis when showing bank availability. The response includes bank code/name and active status.

Payment Gateways endpoint:

```text
GET https://www.billplz.com/api/v4/payment_gateways
```

Use this when exact current FPX, e-wallet, card, or Billplz simulator availability matters. Response rows include gateway `code`, `active`, and `category`.

Webhook rank endpoint:

```text
GET https://www.billplz.com/api/v4/webhook_rank
```

Rank `0.0` is highest priority and `10.0` is lowest. Failed callbacks degrade rank, so callback handlers should verify quickly, persist durably, and return HTTP `200` for accepted notifications.

## X Signature

X Signature is separate from V5 Checksum.

X Signature calculation:

1. Build source string from key-value pairs.
2. Exclude `x_signature`.
3. Sort elements ascending, case-insensitive.
4. Join elements with `|`.
5. Sign final source string with HMAC-SHA256 using X Signature Key.
6. Compare computed signature to `x_signature`.

Nested data prepends parent key to nested elements.

For callback POST payloads, concatenate key plus value with no separator before sorting and joining with `|`. Include empty values exactly as received.

For redirect GET payloads, use `billplz` plus the nested query key plus value, excluding only `billplz[x_signature]`. Example source items look like `billplzid...`, `billplzpaid...`, and `billplzpaid_at...`.

Do not JSON stringify, normalize, or reformat received values before signature calculation.

Use timing-safe comparison when framework supports it.

## Callback And Redirect

Basic callback/redirect:

- Disabled by default for security reasons on newer accounts.
- Older accounts before September 2018 may have Basic Callback/Redirect enabled by default.
- Use X Signature Callback/Redirect instead.

X Signature callback:

- Billplz posts Bill object to `callback_url` on payment completion, success or failure.
- Callback includes `x_signature`.
- Verify HMAC-SHA256 before processing.
- A failed payment returns `paid=false` and state such as `due`.
- Possible Bill states include `due`, `deleted`, and `paid`.
- Optional transaction info requires Extra Payment Completion Information.
- `transaction_status` possible values: `pending`, `completed`, `failed`.
- Callback timeout is 20 seconds.
- Endpoint must respond HTTP `200`.
- Bill callback retries: maximum 5 attempts.
- Retry schedule after failure: second attempt after `15 seconds + random 0-300 seconds`, third/fourth after `15 minutes + random 0-300 seconds`, fifth after `24 hours + random 0-300 seconds` after fourth.
- Account rank is degraded for every unsuccessful callback attempt, lowering callback scheduling priority.

X Signature redirect:

- Browser GET to `redirect_url`.
- Includes `billplz[id]`, `billplz[paid]`, `billplz[paid_at]`, optional `billplz[transaction_id]`, optional `billplz[transaction_status]`, and `billplz[x_signature]`.
- Verify HMAC-SHA256 before trusting redirect.
- Redirect is not guaranteed; users may close browser, disconnect, or app may hang.

## Settlement Strategy

Only mark local payment paid when all are true:

- X Signature valid.
- Local payment exists and maps to Billplz Bill ID.
- Provider is `billplz`.
- Collection ID matches expected Collection.
- Amount matches expected cents.
- `paid=true`.
- State is `paid`.
- If transaction info exists, status is compatible with success, such as `completed`.
- Duplicate callback/redirect replay has no extra side effects.

## V5 Checksum

V5 endpoints require:

- `epoch`: UNIX epoch.
- `checksum`: HMAC-SHA512 using X Signature Key.

Checksum formation:

- Concatenate only values required by the endpoint.
- Use strict order shown in that endpoint's checksum guide.
- Include optional values only when endpoint guide says and parameter is present.
- Digest raw string with HMAC-SHA512 and X Signature Key.

Do not confuse V5 Checksum with X Signature.

Common checksum orders:

| Endpoint | Values, in strict order |
| --- | --- |
| Create Payment Order Collection | `[title, callback_url*, epoch]` |
| Get Payment Order Collection | `[payment_order_collection_id, epoch]` |
| Create Payment Order | `[payment_order_collection_id, bank_account_number, total, epoch]` |
| Get Payment Order | `[payment_order_id, epoch]` |
| Get Payment Order Limit | `[epoch]` |
| Payment Order Callback | `[id, bank_account_number, status, total, reference_id, epoch]` |

`*` means include the optional value only when supplied.

## Payment Order

Payment Order is for transferring money to bank accounts.

Payment Order Collection:

- Create Payment Order Collection before Payment Orders.
- API examples use V5 endpoints.
- Checksum examples include strict ordered values such as `[title, epoch]` for create collection and `[payment_order_collection_id, epoch]` for get collection.

Create Payment Order endpoint:

```text
POST https://www.billplz.com/api/v5/payment_orders
```

Required fields:

- `payment_order_collection_id`
- `bank_code`: SWIFT Bank Code, case-sensitive.
- `bank_account_number`
- `name`
- `description`
- `total`: amount in cents.
- `epoch`
- `checksum`: strict ordered values `[payment_order_collection_id, bank_account_number, total, epoch]`.

Optional fields include:

- `email`
- `notification`
- `recipient_notification`
- `reference_id`: unique reference scoped by Payment Order Collection; useful to prevent unintentional duplicate creation.

Sandbox testing:

- Use `bank_code: DUMMYBANKVERIFIED` for successful payment order result.
- Other bank codes in sandbox result in payment order transaction failure.

Payment Order callback:

- Sent when status changes to `completed` or `refunded`.
- Includes HMAC-SHA512 checksum.
- Payment Order statuses include `processing`, `completed`, `refunded`, and `cancelled`.
- Callback timeout is 20 seconds.
- Endpoint must respond HTTP `200`.
- Retry maximum is 2 attempts; second attempt after one hour.
- Checksum callback order: `[id, bank_account_number, status, total, reference_id, epoch]`.

Payment Order is high impact. Require explicit user confirmation before live payouts.

## Card Tokenization

Billplz V4 card tokenization is available to paid plan members and may require Billplz enablement.

Create card endpoint:

```text
POST https://www.billplz.com/api/v4/cards
```

Required fields: `name`, `email`, `phone`, and `callback_url`.

The response includes a pending card token and `authentication_redirect_url` for 3D Secure authentication. Billplz posts the result to `callback_url`; verify the callback checksum before storing an active token.

Card charge and pre-auth endpoints:

```text
POST https://www.billplz.com/api/v4/bills/{BILL_ID}/charge
POST https://www.billplz.com/api/v4/bills/{BILL_ID}/preauth
POST https://www.billplz.com/api/v4/bills/{BILL_ID}/preauth_capture
DELETE https://www.billplz.com/api/v4/cards/{CARD_ID}
```

Treat tokenized card operations as higher-risk than hosted checkout: keep tokens server-side, test failed status paths, and never mark paid from card redirect/UI state alone.

## OAuth 2.0

- Billplz OAuth lets platform owners connect merchants without manually copying API Secret Key, Collection ID, and X Signature Key.
- OAuth 2.0 is partner-exclusive. Contact Billplz team for details.

## Official GitHub Ecosystem

Billplz GitHub organization includes many plugins/integrations:

- WooCommerce
- GravityForms
- OpenCart
- PrestaShop
- WP e-Commerce
- WHMCS
- GiveWP
- Moodle
- Magento
- Go package
- ASP.NET example
- many older platform-specific plugins

Use GitHub plugins as implementation examples, but verify API/security behavior against current API docs before copying.

## Operational Pages

- `https://status.billplz.com/`: status page.
- `https://check.billplz.com/`: service check.
- `https://fpxstatus.billplz.com/`: FPX status.

Use these for operational troubleshooting only, not payment truth.

## Public Product Pages

Public pages reviewed:

- About
- Security
- Pricing
- Collection
- Payment Order
- Payment Form
- Catalog
- Zakat

Pricing and product availability can change; verify live pages/account dashboard before quoting costs, limits, channel availability, or settlement timing.

## Rate Limits And Errors

Billplz GET endpoints are rate limited cumulatively per IP and account.

| Limit state | Allowance |
| --- | --- |
| Default | 100 requests per 5-minute window |
| Abusive | 10 requests per 5-minute window |

Watch response headers such as `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset`.

Common API error codes:

| Code | Meaning |
| --- | --- |
| `401` | Unauthorized or invalid API key |
| `404` | Not found |
| `422` | Invalid parameter or unprocessable entity |
| `429` | Rate limit exceeded |
| `500` | Internal server error; retry later |
| `503` | Service unavailable or maintenance |

## Common Failure Fixes

- Callback not received: ensure public HTTPS URL, returns 200 within 20 seconds, and firewall allows Billplz.
- Duplicate fulfilment: callback and redirect can arrive in any order and callbacks retry; use idempotency.
- Signature mismatch: ensure `x_signature` excluded, keys sorted case-insensitive, nested keys flattened as docs show, and exact X Signature Key used for same environment.
- Wrong amount: compare local cents to Billplz `amount`/`paid_amount`.
- Paid UI but pending backend: redirect is UX only; wait for verified callback or verified status.
- Sandbox/live issue: keep sandbox and production API Secret Key, Collection ID, X Signature Key, and bill URLs separate.
- Payment Order duplicate payout: use unique `reference_id` and store Payment Order ID before retrying.
