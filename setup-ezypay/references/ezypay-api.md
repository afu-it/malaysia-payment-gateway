# Ezypay API Reference (entry point)

Distilled from the official Ezypay developer docs at `https://developer.ezypay.com` (guides under `/docs`, API reference under `/reference`, OpenAPI `billing` v2.1.0 embedded in each reference page). Checked 2026-10-21. Detail lives in:

- `ezypay-billing.md`: request and response field tables for every payment-related operation.
- `ezypay-webhooks.md`: events, payloads, signature verification, retries.
- `ezypay-payment-methods.md`: Payment Capture Page, mobile, Apple Pay, Google Pay, payloads, test data, Malaysia failure codes.

## What Ezypay is

Ezypay is a subscription billing platform for membership businesses (mainly gyms and studios), sold through software partners or direct API integration. It vaults payment methods, creates invoices (on demand or from subscriptions), collects, retries failed payments, and settles net of fees to the merchant. It is not a redirect-per-payment gateway: you store a payment method token once, then bill it.

Two partnership types (`/docs/integrating-with-ezypay`):

- Partner (software provider): one partner `client_id`/`client_secret` reused for all its merchants, plus per-merchant credentials.
- Integrated merchant: processes payments in its own platform; same credential shape.

Two billing models:

- On-demand: you call Create Invoice whenever you want to charge. Ezypay keeps no schedule.
- Subscription: you create a plan and a subscription; Ezypay generates and bills invoices on schedule.

Plus Checkout (hosted one-off card payment page), Pay Now (hosted recovery page for failed invoices), Tap to Pay (AU and NZ only) and PromptPay (Thailand only).

## Malaysia scope

From `/docs/supported-payment-methods` (checked 2026-10-21):

| Item | Malaysia value |
| --- | --- |
| Bank direct debit | No |
| Cards | Visa, Mastercard |
| Apple Pay, Google Pay | Listed, marked "currently under beta testing" |
| PayTo | No (Australia only) |
| Wallet | No (GCash is Philippines only) |
| Card network tokenisation | Yes (Visa and Mastercard), automatic |
| Country code / currency | `MY` / `MYR`, decimal values supported |
| 3D Secure | Not supported. CVV not required |
| Cross-border currency | Not supported. A Malaysian merchant must charge in `MYR` |
| Result timing | Cards return results in near real time |
| Settlement holding period | T+3 business days (all countries except KR, ID, TH) |
| Settlement frequency | Weekly or monthly (daily is AU and NZ only) |
| Failed payment code guide | `http://www.ezypay.com/failed-payment-response-guide/my/mastercard-visa` |

FPX, DuitNow QR, Malaysian e-wallets and BNPL are not offered by Ezypay in Malaysia. Do not build them.

## Environments

| Purpose | Sandbox | Production |
| --- | --- | --- |
| Identity (tokens) | `https://identity-sandbox.ezypay.com` | `https://identity.ezypay.com` |
| Vault (bank and QR token APIs, legacy HPP) | `https://vault-sandbox.ezypay.com` | `https://vault.ezypay.com` |
| Billing API | `https://api-sandbox.ezypay.com` | `https://api-global.ezypay.com` |
| Hosted pages (Payment Capture Page) | `https://hosted-global-sandbox.ezypay.com` | `https://hosted-global.ezypay.com` |
| Checkout pages (returned by API) | `https://checkout-sandbox.ezypay.com/<hash>` (example) | URL returned in `checkoutUrl` |
| NPP (PayTo mandate types, AU only) | `https://api-gateway-2-sandbox.ezypay.com/v2/npp` | Not documented in official docs (checked 2026-10-21) |

Server prefixes used in the OpenAPI specs:

- Billing: `https://api-sandbox.ezypay.com/v2` + `/billing/...` (for example `https://api-sandbox.ezypay.com/v2/billing/invoices`).
- Partner billing: `https://api-sandbox.ezypay.com/v2` + `/partnerinvoices`, `/partnersubscriptions`, `/merchantconnections`, `/retrypartnerinvoices/...`.
- Files: `https://api-sandbox.ezypay.com/v2/files/{fileId}`.
- Webhooks: `https://api-sandbox.ezypay.com/v2/webhooks` + `/`, `/{webhookId}`, `/logs`.
- Events: `https://api-sandbox.ezypay.com/v2/events/{eventId}/resendwebhook`.
- Vault: `https://vault-sandbox.ezypay.com/v2/vault/paymentmethodtokens/bank` and `/qrpayment`.

Sandbox is a scaled-down replica. Credentials are separate per environment. Inactive sandbox accounts are disabled 8 weeks after last activity. Sandbox actions send no customer notifications. Styling profiles are per environment.

## Authentication

OAuth 2.0 password grant. Partner credentials (`client_id`, `client_secret`, scope) plus merchant credentials (username, password, merchant ID). Credentials are issued after onboarding; merchant username and password arrive by Welcome Email.

```bash
curl -X POST https://identity-sandbox.ezypay.com/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password&username=<MERCHANT_USERNAME>&password=<MERCHANT_PASSWORD>&client_id=<CLIENT_ID>&client_secret=<CLIENT_SECRET>&scope=integrator%20billing_profile%20create_payment_method%20offline_access'
```

Response:

```json
{ "token_type": "Bearer", "expires_in": 3600, "access_token": "<ACCESS_TOKEN>",
  "scope": "integrator offline_access billing_profile create_payment_method",
  "refresh_token": "<REFRESH_TOKEN>" }
```

Refresh (refresh token is disabled if unused for 7 days; Ezypay prefers re-creating a token after expiry):

```bash
curl -X POST https://identity-sandbox.ezypay.com/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=refresh_token&client_id=<CLIENT_ID>&client_secret=<CLIENT_SECRET>&refresh_token=<REFRESH_TOKEN>'
```

- Access tokens expire after 60 minutes. Reuse a token until it expires; do not fetch one per call.
- Rate limit on token requests: 5,000 per 5 minutes, else `429 Too Many Requests` and possible temporary block.
- Hard limit of 4 requests per second per individual merchant account, enforced by Ezypay's identity provider against brute force.
- Multi-split payouts need the extra scope `partner`.
- Hosted page token (reference page "Get A Hosted Page Token", `POST https://identity-sandbox.ezypay.com/token`): same fields with scope `integrator hosted_payment`, documented as a limited-scope token for embedding hosted pages, valid 60 minutes. The OpenAPI block types the body as JSON while the auth guide sends form-encoded; the identity endpoint examples are form-encoded. The Payment Capture Page guide links its `token` parameter to the normal auth token. Confirm with Ezypay which scope your account allows in the iframe; prefer the limited scope token when it works because the URL reaches the browser.
- Bad credentials return:

```json
{ "type": "authentication_error", "code": 400,
  "message": "400 - {\"error\":\"invalid_grant\",\"error_description\":\"The credentials provided were invalid.\"}" }
```

Every billing call:

```text
Authorization: Bearer <ACCESS_TOKEN>
merchant: <MERCHANT_ID>
Content-Type: application/json
```

Header spelling seen in the reference: `merchant` on most operations, `Merchant` on checkout and retrieve merchant, `merchantId` on Create A Terminal Invoice. HTTP header names are case-insensitive. The `customerId` path value is case-sensitive; do not upper-case it.

Best practice from the docs: give merchants a secure form in your portal to enter their own username, password and merchant ID instead of developers handling them.

## End-to-end server flow (on-demand card billing, Malaysia)

1. Get or reuse a bearer token for the merchant.
2. `POST /v2/billing/customers` with `firstName`, `lastName`, `email` (plus `mobilePhone`, `referenceCode`). Store `id` and `number` (`EZYxxxxxx`). Ezypay does not dedupe customers.
3. Show Ezypay fees to the customer if they are customer-paid (`GET /v2/billing/merchant/pricing/fees`) and show Ezypay terms (mandatory).
4. Render the Payment Capture Page iframe:
   `https://hosted-global-sandbox.ezypay.com/paymentmethod/embed?token=<TOKEN>&customerId=<CUSTOMER_ID>&submitbutton=true&paymentMethods=VISA,MASTERCARD` with `allow="payment"`.
5. In a `message` listener (check `event.origin` is the hosted-global host), on `type == "success"` read `paymentMethodToken`; if `link_status` is false call `POST /v2/billing/customers/{customerId}/paymentmethods` to link it. Send the token to your server; the server should confirm with `GET /v2/billing/customers/{customerId}/paymentmethods/{paymentMethodToken}` that it exists, is `valid`, and belongs to that customer.
6. Create a local payment row `pending` with your own id.
7. `POST /v2/billing/invoices`:

```json
{ "customerId": "<CUSTOMER_ID>", "paymentMethodToken": "<PM_TOKEN>",
  "externalInvoiceId": "<YOUR_PAYMENT_ID>", "processingModel": "cardonfile",
  "items": [ { "amount": { "currency": "MYR", "value": 120.00 }, "description": "Monthly membership" } ] }
```

   Response status is `PROCESSING` (or `UNPAID` if `scheduledPaymentDate` is set). Store invoice `id` and `documentNumber`. Do not fulfil yet.
8. Receive `invoice_paid` or `invoice_past_due` on your webhook. Verify `X-Ezypay-Signature`, dedupe `requestId`, check `merchantId`, then `GET /v2/billing/invoices/{invoiceId}`.
9. Settle once when status is `PAID`, invoice id and `externalInvoiceId` match, currency is `MYR`, and your `on_demand_payment` item value matches the expected amount (fees are separate items).
10. On `PAST_DUE`: read `failedPaymentReason.code`, `paymentMethodInvalid`, `manualRetryPossible`, `payNowUrl`, `scheduledPaymentDate`; notify the customer; replace invalid methods; retry or resolve.
11. Reconcile with `transaction_settled` webhooks and settlement reports.

Subscription variant: create a plan once (`POST /v2/billing/plans`, a "dummy" plan is fine), then `POST /v2/billing/subscriptions`; listen for `invoice_created`, `invoice_paid`, `invoice_past_due`, `subscription_payment_stopped`, `subscription_payment_reactivate`.

Checkout variant: `POST /v2/billing/checkout` with `amount`, `description`, `customerId`, `successUrl`, `cancelUrl`; redirect to `checkoutUrl`. `successUrl` receives `documentNumber`, `invoiceId`, `customerId`, `paymentMethodToken` as query parameters: treat as hints and confirm with the webhook or Retrieve Invoice (filter `checkoutId` on List Invoices). Checkout failures do not enter automatic recovery and stay `past_due`. Disable unused sessions (`DELETE /v2/billing/checkout/{id}`); sessions auto-disable after 3 failed attempts by default. Checkout only accepts card transactions.

## Signature and verification summary

Only webhooks are signed. There is no request signing for API calls (bearer token only) and no signed redirect.

```text
signature = lowercase_hex( HMAC_SHA1( key = clientKey, message = raw HTTP request body bytes ) )
compare with header X-Ezypay-Signature using constant-time comparison
```

Official test vector: client key `key` and the payload in `ezypay-webhooks.md` give `6354ecd501ca4c87da2b42872949c7fa02fefd89` (re-computed and matched on 2026-10-21). Full code in `ezypay-webhooks.md`.

## Amount and currency rules

- Amount object everywhere: `{ "currency": "MYR", "value": <number>, "type": null }`. `value` is a decimal in major units (RM 24.50 is `24.50`), must be greater than zero. `type` accepts `percentage` or `fixed_amount` where documented; leave it out for normal charges.
- `MYR` supports decimals. TWD, KRW and IDR do not (trailing `.00` is accepted).
- Wrong currency: `{"type":"api_error","code":"[currency] ","message":"[Value specified is invalid.] "}`. Decimal on a no-decimal currency: `code` `[value] `. Currency not enabled for checkout: `currency_not_accepted_by_merchant`.
- Amounts are gross, tax inclusive. `tax.rate` is a percentage from 0 to 99.99; if omitted the merchant default applies.
- Invoice `amount` = sum of items, including Ezypay fee items when fees are customer-paid. Item `type` values: `on_demand_payment`, `subscription_payment`, `setup_payment`, `addon_payment`, `transaction_fee`, `load_fee`, `failed_payment_fee`, `partner_on_demand`.
- An invoice over the merchant's maximum debit value is rejected with `invoice_exceeds_merchant_maximum_debit_value`.
- Avoid special characters such as `&` in request bodies, especially `accountingCode`; use `_` instead of spaces.

## Statuses

Invoice (`status`): `UNPAID` (scheduled or not sent), `PENDING`, `PROCESSING`, `PAID`, `PAST_DUE`, `REFUNDED`, `PARTIALLY_REFUNDED`, `WRITTEN_OFF`, `PENDING_QR_PAYMENT` (PromptPay), `PENDING_TAP_PAYMENT` / `PENDING_TERMINAL_PAYMENT`, `VOIDED`, `FAILED` (terminal invoices). List Invoices filters accept `PAID`, `PENDING`, `PROCESSING`, `PAST_DUE`, `UNPAID`, `REFUNDED`, `PARTIALLY_REFUNDED`, `WRITTEN_OFF`.

Transitions that matter:

- Create without schedule: `PROCESSING` then `PAID` (`invoice_paid`) or `PAST_DUE` (`invoice_past_due`).
- Retry: `PAST_DUE` to `PROCESSING`.
- Refund: `PAID` to `PROCESSING` (credit note `PROCESSING`), then `REFUNDED` / `PARTIALLY_REFUNDED` (credit note `PAID`) or back to `PAID` (credit note `FAILED`).
- Chargeback: `PAID` to `PAST_DUE` with credit note `PAID` reason `CHARGEBACK`; if won, `PAST_DUE` to `PAID` and credit note `FAILED`.
- Write off: `PAST_DUE` or `UNPAID` to `WRITTEN_OFF`. Record external payment: `PAST_DUE` to `PAID` with no `invoice_paid` webhook.

Subscription: `ACTIVE`, `FUTURE`, `PENDING`, `PAST_DUE`, `PENDING_CANCELLATION`, `CANCELLED`, `COMPLETED`.
Credit note: `PROCESSING`, `PAID`, `FAILED`; `reason` `REFUND` or `CHARGEBACK`.
Transaction: `success`, `processing`, `failed`, `settled`.
Plan: `ACTIVE`, `INACTIVE`. Partner invoice: `PROCESSING`, `PAID`, `PAST_DUE` / `FAILED`.
Merchant connection: `pending`, `active`, `cancelled`.
Terminal registration: device `INACTIVE` to `REGISTERED`; webhook `status` `SUCCESS`.

## Endpoint catalogue

All 79 documented operations plus the identity and sandbox-only endpoints. Base `https://api-sandbox.ezypay.com/v2` unless stated.

### Identity

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `https://identity-sandbox.ezypay.com/token` | Access token (`grant_type=password` or `refresh_token`); also the hosted page token (scope `integrator hosted_payment`) |

### Customers

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/billing/customers` | Create a customer |
| GET | `/billing/customers` | List customers |
| GET | `/billing/customers/{customerId}` | Retrieve a customer |
| PUT | `/billing/customers/{customerId}` | Update a customer |

### Payment methods

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `https://vault-sandbox.ezypay.com/v2/vault/paymentmethodtokens/bank` | Create a bank token (AU and NZ direct debit; not Malaysia) |
| POST | `https://vault-sandbox.ezypay.com/v2/vault/paymentmethodtokens/qrpayment` | Create a QR token (PromptPay, Thailand) |
| POST | `/billing/customers/{customerId}/paymentmethods` | Link a token to a customer |
| GET | `/billing/customers/{customerId}/paymentmethods` | List a customer's payment methods |
| GET | `/billing/customers/{customerId}/paymentmethods/{paymentMethodToken}` | Retrieve a payment method |
| GET | `/billing/customers/{customerId}/paymentmethods/primary` | Retrieve primary payment method |
| PUT | `/billing/customers/{customerId}/paymentmethods/{paymentMethodToken}/primary` | Set primary payment method |
| PUT | `/billing/customers/{customerId}/paymentmethods/{paymentMethodToken}/new` | Replace a payment method (moves invoices and subscriptions) |
| DELETE | `/billing/customers/{customerId}/paymentmethods/{paymentMethodToken}` | Delete a payment method |
| POST | `/billing/styles` | Create a Payment Capture Page styling |
| GET | `/billing/styles/{styleId}` | Retrieve a styling |

There is no public card tokenisation API: cards are captured only through the Payment Capture Page, Checkout or Pay Now.

### Invoices (on-demand) and recovery

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/billing/invoices` | Create an invoice (charges immediately unless scheduled) |
| GET | `/billing/invoices` | List invoices (filters include `externalInvoiceId`, `checkoutId`, `status`) |
| GET | `/billing/invoices/{invoiceId}` | Retrieve an invoice (status requery) |
| PUT | `/billing/invoices/{invoiceId}` | Update an invoice (unpaid: all fields; past due: token, schedule, description, discounts) |
| POST | `/billing/invoices/{invoiceId}/retrypayment` | Retry a `PAST_DUE` invoice |
| POST | `/billing/invoices/{invoiceId}/writeoff` | Write off a `PAST_DUE` or `UNPAID` invoice |
| POST | `/billing/invoices/{invoiceId}/recordpayment` | Record an external payment on a `PAST_DUE` invoice |
| PUT | `/billing/invoices/{invoiceId}/refund` | Refund an invoice (full, partial, per item) |
| PUT | `/billing/invoices/{invoiceId}/cancel` | Cancel a PromptPay QR payment (`PENDING_QR_PAYMENT` only) |

### Checkout

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/billing/checkout` | Create a checkout session, returns `checkoutUrl` and `id` |
| DELETE | `/billing/checkout/{id}` | Disable a checkout session (irreversible; path value is the hash from the URL) |

### Plans, subscriptions, future invoices

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/billing/plans` | Create a plan |
| GET | `/billing/plans` | List plans |
| GET | `/billing/plans/{planId}` | Retrieve a plan |
| PUT | `/billing/plans/{planId}` | Update a plan (affects only new subscriptions) |
| POST | `/billing/subscriptions` | Create a subscription |
| POST | `/billing/subscriptions/preview` | Preview a subscription |
| GET | `/billing/subscriptions` | List subscriptions |
| GET | `/billing/subscriptions/{subscriptionId}` | Retrieve a subscription |
| PUT | `/billing/subscriptions/{subscriptionId}` | Update a subscription |
| PUT | `/billing/subscriptions/{subscriptionId}/activate` | Activate a `PENDING` subscription |
| PUT | `/billing/subscriptions/{subscriptionId}/cancel` | Cancel a subscription (optional future date) |
| POST | `/billing/subscriptions/{subscriptionId}/undo-cancel` | Undo a `PENDING_CANCELLATION` |
| PUT | `/billing/subscriptions/{subscriptionId}/paymentmethod/{paymentMethodToken}` | Change a subscription's payment method |
| GET | `/billing/futureinvoices` | List future invoices |
| PUT | `/billing/futureinvoices` | Update a future invoice |
| DELETE | `/billing/futureinvoices` | Delete a future invoice (skip or pause a cycle) |
| POST | `/billing/futureinvoices/recordpayment` | Record an external payment for a future invoice |

### Transactions, credit notes, merchant, settlement, reports, files

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/billing/transactions` | List transactions |
| GET | `/billing/transactions/{transactionId}` | Retrieve a transaction |
| GET | `/billing/creditnotes` | List credit notes (needs `customerId`, `subscriptionId` or `invoiceId`) |
| GET | `/billing/creditnotes/{creditNoteId}` | Retrieve a credit note |
| GET | `/billing/merchant` | Retrieve the merchant |
| GET | `/billing/merchant/pricing/fees` | Customer-paid fee schedule |
| GET | `/billing/merchant/nextSettlement/{merchantId}` | Amount collected for the next settlement |
| GET | `/billing/settlements` | List settlements |
| POST | `/billing/settlements/{settlementNumber}/file` | Generate a settlement document (returns `fileId`) |
| POST | `/billing/reports/merchanttransactions` | Merchant transactions CSV (max 90 days) |
| POST | `/billing/reports/outstandingcollectionsreport/generate` | Outstanding collections report (PDF or CSV) |
| GET | `/files/{fileId}` | Get a generated file (returns download `url`) |

### Webhooks and events

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/webhooks/` | Create a webhook (`url`, `eventTypes`, `clientKey`) |
| GET | `/webhooks/` | List webhooks |
| GET | `/webhooks/{webhookId}` | Retrieve a webhook |
| PUT | `/webhooks/{webhookId}` | Update a webhook (overwrites url, events, key) |
| DELETE | `/webhooks/{webhookId}` | Delete a webhook |
| GET | `/webhooks/logs` | List webhook delivery attempts |
| POST | `/events/{eventId}/resendwebhook` | Resend an event to all related webhooks |

### Partner billing (multi-split payouts)

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/merchantconnections` | List merchant connections |
| GET | `/merchantconnections/{merchantConnectionId}` | Retrieve a merchant connection |
| POST | `/partnerinvoices` | Create a partner invoice (merchant to merchant) |
| GET | `/partnerinvoices` | List partner invoices |
| POST | `/retrypartnerinvoices/{partnerInvoiceId}` | Retry a past due partner invoice |
| POST | `/partnersubscriptions` | Create a partner subscription |
| GET | `/partnersubscriptions` | List partner subscriptions |
| PUT | `/partnersubscriptions/{partnerSubscriptionId}/cancel` | Cancel a partner subscription |
| POST | `/partnersubscriptions/{partnerSubscriptionId}/addons/{partnerSubscriptionAddOnCode}/recordusage` | Record add-on usage |
| GET | `/partnersubscriptions/{partnerSubscriptionId}/addons/{partnerSubscriptionAddOnCode}` | List add-on usage |

### Tap to Pay (AU and NZ only)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/billing/terminal/register` | Register a terminal device, returns 6-digit activation code (expires in 24 hours) |
| POST | `/billing/terminal/invoices` | Create a terminal invoice (max 1,500 per invoice) |
| GET | `/billing/terminal/invoices` | List terminal invoices |
| GET | `/billing/terminal/invoices/{invoiceId}` | Retrieve a terminal invoice |

### PayTo (Australia only)

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `https://api-gateway-2-sandbox.ezypay.com/v2/npp/mandatetype` | Get a mandate type |
| GET | `https://api-gateway-2-sandbox.ezypay.com/v2/npp/mandatetype/list` | List mandate types |
| POST | `https://api-sandbox.ezypay.com/v2/npp/mandate/status/{paymentMethodToken}/mock` | Sandbox only: set mandate status (`mandateStatus=ACTV` or `CNCD`, form-encoded) |

## Pagination

List endpoints use `limit` (max 100) and `cursor` (offset: `limit=10&cursor=10` returns records 11 to 20). Responses carry `paging` with `nextUrl`, `nextCursor`, `limit`, `totalCount`. Defaults vary by endpoint (10 for invoices, subscriptions and future invoices; 100 elsewhere). List Invoices returns the unfiltered list if a filter value matches nothing, so never assume a filtered result belongs to your filter: check each row.

## Errors

Error body: `{ "type": "<type>", "code": "<code>", "message": "<text>" }`. Types seen: `api_error`, `invalid_request_error`, `authentication_error`.

| Code | Where | Meaning |
| --- | --- | --- |
| `customer_not_found` | customer, invoice, payment method calls | `customerId` not found |
| `invoice_not_found` | invoice calls | `invoiceId` not found |
| `invoiceItem_not_found` | update invoice, refund | item id not found |
| `invoice_creation_failed` | create invoice | creation failed |
| `invoice_exceeds_merchant_maximum_debit_value` | create invoice | amount (excluding fees) over merchant limit |
| `payment_method_token_not_found` | invoice, subscription, payment method calls | token not found |
| `payment_method_not_valid` | refund, retry | method invalid or deleted |
| `payment_method_not_accepted` | create QR token | method not accepted for the merchant |
| `unauthorized_payment_method` | replace payment method | "This merchant is not authorized to use this payment method" |
| `INVALID_PRIMARY_METHOD` | set primary | QR payment methods cannot be primary |
| `invalid_invoice_status_cannot_be_retried` | retry | only `past_due` can be retried |
| `manual_retry_not_allowed_for_invoice` | retry | manual retry blocked (see `manualRetryPossible`) |
| `invoice_cannot_be_written_off` | write off | wrong status |
| `invalid_invoice_status_cannot_be_refunded` | refund | only `PAID` or `PARTIALLY_REFUNDED` |
| `refund_amount_exceed_invoice_amount` | refund | amount above remaining refundable |
| `refund_item_amount_exceed_invoice_item_amount` | refund | item amount too high |
| `refund_partial_fee_item_not_allowed` | refund | fee items refund in full only |
| `refund_not_allowed` | refund | PromptPay cannot be refunded |
| `discountAmount_cannot_more_than_item_amount` | update invoice | discount too high |
| `plan_not_found`, `subscription_not_found` | plan, subscription calls | id not found |
| `subscription_must_be_pending` | activate | not pending |
| `subscription_must_be_active` | update | not active |
| `subscription_cannot_be_cancelled` | cancel | only pending, future, active or past due can be cancelled |
| `end_date_must_be_future_date`, `start_date_and_end_date_not_within_the_billing_interval_range` | subscriptions | date rules |
| `checkout_session_not_found`, `checkout_session_disabled`, `new_checkout_creation_disabled` | checkout | session state or merchant setting |
| `currency_not_accepted_by_merchant` | checkout | currency mismatch |
| `invalid_merchant_id` | any | bad `merchant` header |
| `duplicate_webhook_url` | create or update webhook | same URL already registered for this merchant |
| `webhook_not_found` | webhook calls | id not found |
| `qr_cancel_failed` | cancel QR | cancellation failed |
| `App_device_Id_Is_required`, `Device_Name_Is_required` | terminal register | missing field |
| `wallet_payment_method_error` | PCP wallet linking (GCash) | linking failed |

Payment failure reasons on invoices and transactions use `failedPaymentReason.code` (for example `insufficient_funds`, `limit_exceeded`, `expired_card`, `lost_or_stolen_card`, `invalid_payment_method`, `customer_to_contact_bank`, `declined`, `authority_withdrawn`, `suspected_fraud`, `bank_system_error`); see `ezypay-payment-methods.md`. Ezypay says to build logic on `failedPaymentReason.code`, not on `paymentProviderResponse`.

## Rate limits, IPs, idempotency

- Token endpoint: 5,000 requests per 5 minutes; 4 requests per second per merchant account.
- Other API rate limits: Not documented in official docs (checked 2026-10-21).
- Webhook source IP addresses: Not documented in official docs (checked 2026-10-21). Rely on the HMAC signature.
- Idempotency: no idempotency header is documented. Use `externalInvoiceId` (unique per merchant) on invoices and terminal invoices, and check List Invoices by `externalInvoiceId` before re-sending after a timeout.
- Wait 1 to 2 seconds after creating a customer before adding a payment method (reference "Customer" section note on asynchronous calls).
- All requests must use HTTPS.
