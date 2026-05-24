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

Create Bill endpoint:

```text
POST https://www.billplz.com/api/v3/bills
```

Important fields:

- `collection_id`: Collection ID.
- `description`: shown on bill template.
- `email`: recipient email.
- `name`: recipient name.
- `amount`: positive integer in cents.
- `callback_url`: webhook URL called after transaction completes.
- `redirect_url`: optional browser redirect after payment.
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

## Direct Payment Gateway

Billplz supports direct payment gateway flow:

- Merchant creates Bill with bank/payment gateway code.
- Merchant redirects payer to Bill URL.
- Append `?auto_submit=true` to skip Billplz payment option selection page when valid.
- If inactive or invalid payment gateway is submitted, payer falls back to Billplz payment option selection page.
- Use `Get Payment Gateways` and payment gateway abbreviations/bank codes from API docs for exact supported codes.

Do not hard-code current bank/payment gateway codes without verifying docs or live response.

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

## Common Failure Fixes

- Callback not received: ensure public HTTPS URL, returns 200 within 20 seconds, and firewall allows Billplz.
- Duplicate fulfilment: callback and redirect can arrive in any order and callbacks retry; use idempotency.
- Signature mismatch: ensure `x_signature` excluded, keys sorted case-insensitive, nested keys flattened as docs show, and exact X Signature Key used for same environment.
- Wrong amount: compare local cents to Billplz `amount`/`paid_amount`.
- Paid UI but pending backend: redirect is UX only; wait for verified callback or verified status.
- Sandbox/live issue: keep sandbox and production API Secret Key, Collection ID, X Signature Key, and bill URLs separate.
- Payment Order duplicate payout: use unique `reference_id` and store Payment Order ID before retrying.
