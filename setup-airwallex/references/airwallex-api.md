# Airwallex API Reference (entry point)

Distilled from the official Airwallex docs, checked 2026-10-21. Scope: online payment acceptance for a Malaysian merchant (MYR). Field tables for each payment operation are in the topic files listed below. Every endpoint Airwallex documents is listed in the Endpoint catalogue at the end of this file.

Topic files:

- `airwallex-payment-intents.md`: Payment Intents, Payment Attempts, `next_action`, 3DS, capture.
- `airwallex-checkout.md`: Hosted Payment Page, Drop-in, Checkout Sessions, Payment Links.
- `airwallex-malaysia-methods.md`: FPX, DuitNow, Touch 'n Go, Boost, GrabPay, ShopeePay, Atome, 7-Eleven, cards, wallets, config APIs.
- `airwallex-webhooks.md`: signature verification, events, IPs, Webhook API.
- `airwallex-refunds-disputes.md`: refunds, cancellations, disputes.
- `airwallex-recurring.md`: Customers, Payment Consents, Payment Methods.
- `airwallex-settlement.md`: settlements and financial reporting APIs.
- `airwallex-testing-errors.md`: sandbox, test cards, simulation, error codes, login API.

## Source URLs

- API introduction: `https://www.airwallex.com/docs/api/introduction`
- Errors: `https://www.airwallex.com/docs/api/errors`
- Versioning: `https://www.airwallex.com/docs/api/versioning`
- Obtain access token: `https://www.airwallex.com/docs/api/authentication/api_access_token/login`
- Payment Intents guide: `https://www.airwallex.com/docs/payments/get-started/using-payments-intent-api`
- Quickstart: `https://www.airwallex.com/docs/payments/get-started/quickstart`
- Payment statuses: `https://www.airwallex.com/docs/payments/reference/payment-statuses`
- Listen for webhook events: `https://www.airwallex.com/docs/developer-tools/webhooks/listen-for-webhook-events`
- Rate limits: `https://www.airwallex.com/docs/developer-tools/api/rate-limits`
- IP whitelisting: `https://www.airwallex.com/docs/developer-tools/api/ip-whitelisting`
- Manage API keys: `https://www.airwallex.com/docs/developer-tools/api/manage-api-keys`
- Supported currencies: `https://www.airwallex.com/docs/payments/reference/supported-currencies`
- Docs index for agents: `https://www.airwallex.com/docs/llms.txt` (every page is also served as Markdown by adding `.md` to its URL)

## Environments

| Environment | API base | Web app | Airwallex.js `env` |
| --- | --- | --- | --- |
| Sandbox | `https://api.sandbox.airwallex.com` | `https://www.sandbox.airwallex.com/app/` | `'sandbox'` per the JS reference (guides and examples also use `'demo'`) |
| Production | `https://api.airwallex.com` | `https://www.airwallex.com/app/` | `'prod'` (default) |

- Paths start with `/api/v1/`. Payment acceptance paths use `/api/v1/pa/`.
- Credentials are environment-specific. Sandbox keys do not work in production.
- Sandbox signup: `https://www.sandbox.airwallex.com/global/signup`.
- The URL version stays `v1`. Contract changes use date versions (latest `2026-08-21`). Your account's version applies by default; override per call with header `x-api-version: YYYY-MM-DD`. Webhook payload shape follows the version chosen on the webhook subscription.
- Airwallex treats new response fields as backwards compatible. Do not fail on unknown fields.

## Authentication

1. `POST /api/v1/authentication/login` with headers `x-client-id: <CLIENT_ID>` and `x-api-key: <API_KEY>` (plus optional `x-login-as: <ACCOUNT_ID>` when a scoped key covers several accounts).
2. Response `201`: `{ "token": "...", "expires_at": "ISO8601" }`. Token lasts 30 minutes. Reuse it until `expires_at`.
3. Send `Authorization: Bearer <token>` on every other call. `x-client-id` and `x-api-key` are only for login.
4. Login errors: `401` `credentials_invalid` or `credentials_expired`. Login is limited to 100 requests per minute per API key, so never log in per request.
5. A `403` with a valid token means the key's scope does not cover the endpoint or account, or the caller IP is not whitelisted (`"Access denied: IP address not whitelisted."`).

```js
// Token cache sketch (server only)
let cached = null;
async function getToken() {
  if (cached && Date.parse(cached.expires_at) - Date.now() > 60_000) return cached.token;
  const r = await fetch(`${AIRWALLEX_BASE}/api/v1/authentication/login`, {
    method: 'POST',
    headers: { 'x-client-id': process.env.AIRWALLEX_CLIENT_ID, 'x-api-key': process.env.AIRWALLEX_API_KEY },
  });
  if (r.status !== 201) throw new Error(`airwallex login ${r.status}`);
  cached = await r.json();
  return cached.token;
}
```

Official Node SDK (beta): `npm install @airwallex/node-sdk@beta` (`new Airwallex({ clientId, apiKey, env: 'demo' | 'prod' })`), supports API versions from `2025-02-14`. Source: `https://www.airwallex.com/docs/developer-tools/sdks/server-side-sdks-(beta)`.

## Amounts, currency, ids

- `amount` is a JSON number in major units: `10.99` means 10.99, not 1,099 minor units. MYR has 2 decimal places.
- `currency` is ISO 4217 (`MYR`). Local Malaysian methods process MYR only.
- `request_id`: required on every create/confirm/capture/cancel/refund POST; unique per operation, max 64 chars; Airwallex documents it as the idempotency key (v4 UUID recommended). The Payment Intents guide says a retried create with the same `request_id` returns the original response instead of a duplicate. The error list also has `duplicate_request` ("The request ID `$id` has been used previously"), so never reuse a `request_id` for a different operation.
- `merchant_order_id`: your order reference, required on create, max 64 chars.
- `metadata`: up to 50 keys, key names up to 50 chars, values up to 500 chars.
- `descriptor`: max 32 chars (create).
- Id prefixes seen in docs: `int_` Payment Intent, `att_` Payment Attempt, `rfd_` Refund, `cus_` Customer, `cst_` Payment Consent, `evt_` webhook event, `dst_` dispute.
- Payment Intents can be queried and operated for up to 2 years after creation.

## Choose an integration

| Surface | Frontend code | PCI (cards) | Notes |
| --- | --- | --- | --- |
| Hosted Payment Page via Airwallex.js `redirectToCheckout()` | Small | SAQ A | Needs an intent `id` + `client_secret` |
| Hosted page via Checkout Session (`POST /api/v1/checkout/checkout_sessions/create`) | None (redirect to `url`) | SAQ A | Marked early access; needs Billing prices or inline `price` |
| Drop-in Element | Medium | SAQ A | Full multi-method UI on your page |
| Embedded Elements (card, split card, Apple Pay, Google Pay) | More | SAQ A | Layout control |
| Native API (server confirm with raw card data) | Custom | PCI-DSS ROC + Airwallex review | Local methods via Native API do not need PCI; device fingerprinting required |
| Payment Links / plugins | None | n/a | WooCommerce, Shopify, Magento, BigCommerce and more |

Source: `https://www.airwallex.com/docs/payments/integration-options/web-checkout`.

## End-to-end flow (recommended)

1. Create a local order/payment row `pending` with your own `merchant_order_id`.
2. `POST /api/v1/pa/payment_intents/create` with `request_id`, `amount`, `currency: "MYR"`, `merchant_order_id`, `return_url` (HTTPS page carrying a non-guessable order reference), optional `customer` or `customer_id`, `metadata`.
3. Store the returned `id` (`int_...`). Return `id`, `client_secret`, `currency` to the browser.
4. Browser: `AirwallexComponentsSDK.init({ env, enabledElements: ['payments'] })` then `payments.redirectToCheckout({ intent_id, client_secret, currency, successUrl, ... })` or mount `createElement('dropIn', { intent_id, client_secret, currency })`.
   Server-only alternative for local methods: `POST /api/v1/pa/payment_intents/{id}/confirm` with `payment_method` (for example `{ "type": "fpx", "fpx": { "bank_name": "maybank2u" } }`) and redirect the shopper to `next_action.url`, or render `next_action.qrcode`.
5. Shopper returns to `successUrl` or `return_url`. Show "processing" and poll your own backend; do not fulfil here.
6. Webhook `payment_intent.succeeded` arrives. Verify signature (below), dedupe by event `id`, call `GET /api/v1/pa/payment_intents/{id}`, check `status == "SUCCEEDED"`, `amount`, `currency`, `merchant_order_id`, then mark paid in one transaction.
7. If no webhook after a reasonable wait, retrieve the intent as a fallback (Airwallex method guides suggest polling after the shopper returns).
8. Refund with `POST /api/v1/pa/refunds/create`; track `refund.*` events.

`client_secret` lasts 60 minutes from intent creation and cannot be extended. If it expires, create a new intent. If the cart changes before payment, call `POST /api/v1/pa/payment_intents/{id}/update` instead of creating a new intent.

## Redirect vs webhook

- `return_url` (intent create/confirm) and `successUrl` / `cancelUrl` (HPP) are browser redirects. They prove nothing.
- Drop-in `success` event and HPP redirect are client-side signals. Use them only to show status.
- Webhooks are the source of truth; retrieve is the fallback.

## Webhook signature (summary)

Full detail in `airwallex-webhooks.md`.

- Headers: `x-timestamp` (Unix time in milliseconds, as a string) and `x-signature` (hex digest).
- String to sign: `value_to_digest = x-timestamp + raw_request_body` (timestamp first, no separator, body exactly as received).
- Algorithm: `HMAC-SHA256(key = webhook secret of that notification URL, message = value_to_digest)`, hex digest.
- Compare to `x-signature` (use a constant-time compare), then check `now - x-timestamp` is within your tolerance, then parse JSON.
- Respond `200` immediately. Anything else or a timeout is retried with exponential back-off for about three days.

```text
ts      = header("x-timestamp")          // e.g. "1357872222592"
sig     = header("x-signature")
raw     = request body bytes as received (UTF-8 string)
expect  = hex(hmac_sha256(WEBHOOK_SECRET, ts + raw))
if !constant_time_equal(expect, sig): return 400
if abs(now_ms() - int(ts)) > TOLERANCE_MS: return 400
event = json_parse(raw)                   // { id, name, account_id, data: { object }, created_at, version }
if seen(event.id): return 200
enqueue(event); return 200
```

## Payment Intent status values

| Status | Meaning | Action |
| --- | --- | --- |
| `REQUIRES_PAYMENT_METHOD` | Created, or the last attempt failed | Collect a method and confirm (SDK does this) |
| `REQUIRES_CUSTOMER_ACTION` | 3DS, redirect, or QR scan pending | Handle `next_action` |
| `REQUIRES_CAPTURE` | Authorized with `auto_capture: false` | Capture within the hold window |
| `PENDING` | Async processing (for example direct debits) | Wait for `payment_intent.succeeded` |
| `PENDING_REVIEW` | Authorized, under risk review | Wait for `payment_intent.succeeded` |
| `SUCCEEDED` | Captured; funds will settle to the Wallet | Fulfil |
| `CANCELLED` | Cancelled or authorization expired | Do not fulfil; new intent to retry |

The API reference says this list is not exhaustive; treat unknown statuses as not paid. Payment Attempt statuses: `RECEIVED`, `AUTHENTICATION_REDIRECTED`, `PENDING_AUTHORIZATION`, `AUTHORIZED`, `PENDING_REVIEW`, `CAPTURE_REQUESTED`, `EXPIRED`, `CANCELLED`, `FAILED`, `SETTLED`, `PAID`. Attempt `failure_code`: `authentication_failed`, `capture_failed`, `authorization_failed`, `provider_unavailable`, `system_unavailable`, `fraud_rejected`.

## Requery / status API

- `GET /api/v1/pa/payment_intents/{id}`: authoritative status, `latest_payment_attempt`, `next_action`.
- `GET /api/v1/pa/payment_intents?merchant_order_id=...`: find by your order id (also `status`, `currency`, `from_created_at`, `to_created_at`, `page_num`, `page_size`).
- `GET /api/v1/pa/payment_attempts/{id}` and `GET /api/v1/pa/payment_attempts?payment_intent_id=...`.
- Checkout Sessions: `GET /api/v1/checkout/checkout_sessions/{id}` returns `status` (`COMPLETED`) and `payment_intent_id`.

## Refunds (summary)

`POST /api/v1/pa/refunds/create` with `request_id`, one of `payment_intent_id` or `payment_attempt_id`, optional `amount` (defaults to remaining captured amount) and `reason`. Statuses `RECEIVED`, `ACCEPTED`, `SETTLED`, `FAILED`. Refunds draw on refundable funds in the same settlement currency and fail if neither unsettled funds nor available balance alone can cover them. Detail in `airwallex-refunds-disputes.md`.

## Recurring / tokenisation (summary)

Customer (`cus_`) + Payment Consent (`cst_`) with `next_triggered_by` (`merchant` or `customer`) and `merchant_trigger_reason` (`scheduled`, `unscheduled`, `installments`), verified once, then confirm later intents with `payment_consent_id`. In Malaysia, Touch 'n Go supports recurring; FPX, DuitNow, Boost, GrabPay, ShopeePay do not. Detail in `airwallex-recurring.md`.

## Errors

Error body: `code`, `source` (field, for `validation_error`), `message`, `trace_id`, `details`; card declines add `provider_original_response_code`. HTTP: 200/201 ok, 400 error, 401 bad or expired token, 403 forbidden (scope or IP), 404 not found, 409 conflict, 429 rate limited, 500 server error. Full code list in `airwallex-testing-errors.md`.

## Rate limits

| Environment | Global rate | Per endpoint | Concurrency |
| --- | --- | --- | --- |
| Production | 100/s | 20/s | 50 |
| Sandbox | 20/s | 10/s | 10 |

Auth endpoint: 100 per minute per API key. On `429` (`too_many_requests`) retry with exponential back-off and jitter (1s, 2s, 4s, ... up to about 16s). Do not load test in sandbox.

## IP allowlists

- Your outbound IPs (API key IP whitelist): IPv4 only, single IP or CIDR `/24` to `/32`, no private ranges. Set in **Settings > Developer > API keys > IP whitelist**.
- Airwallex inbound webhook IPs: see `airwallex-webhooks.md`.

## Known doc inconsistencies (verify in sandbox)

- JS `env`: the Airwallex.js reference lists `'sandbox' | 'prod'`; quickstart and Drop-in examples use `env: 'demo'`.
- ShopeePay: API type is `shopee_pay` (confirm reference, HPP `methods`); the ShopeePay guide examples use `"type": "shopeepay"` with a `flow` field that the API reference does not list.
- GrabPay: guide examples send `grabpay.flow`; the confirm reference lists only `grabpay.shopper_name`.
- Banks list: API reference field is `items[].bank_name`; the FPX guide example response shows `name`.
- Checkout Session create: API reference marks `account_id` required; the quickstart example omits it.
- "Get started" mentions `payment_intent.failed`; the event list documents `payment_intent.payment_failed`.
- DuitNow page lists "Available for businesses registered in HK, AU, SG, UK, EU, US" while also listing MY settlement in MYR; confirm availability for a Malaysian entity with Airwallex.

## Endpoint catalogue

Generated from every resource page in the official API reference (`https://www.airwallex.com/docs/api/...`, 115 resource pages, 470 operations, checked 2026-10-21). Payment acceptance groups come first. Groups marked "not payment acceptance" are FX, payouts, issuing, treasury, billing and platform products, listed only so an agent knows they exist; they are out of scope for this skill.

### Authentication

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/authentication/login` | Obtain Access Token | API Access Token |
| POST | `/api/v1/authentication/authorize` | Authorize an Account | Embedded Components |

### Payment acceptance (pa)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| GET | `/api/v1/pa/config/reserve_plan` | Retrieve the Current Reserve Plan | Config |
| GET | `/api/v1/pa/config/payment_method_types` | List all Available Payment Method Types | Config |
| GET | `/api/v1/pa/config/banks` | List all Available Bank Names | Config |
| GET | `/api/v1/pa/config/applepay/registered_domains` | List all Apple Pay Domains | Config |
| POST | `/api/v1/pa/config/applepay/registered_domains/add_items` | Add Apple Pay Domains | Config |
| POST | `/api/v1/pa/config/applepay/registered_domains/remove_items` | Remove Apple Pay Domains | Config |
| GET | `/api/v1/pa/config/convertible_shopper_currencies` | List all Convertible Shopper Currencies | Config |
| POST | `/api/v1/pa/conversion_quotes/create` | Create a Conversion Quote | Conversion Quotes |
| GET | `/api/v1/pa/conversion_quotes/{id}` | Retrieve a Conversion Quote | Conversion Quotes |
| GET | `/api/v1/pa/customer_balance_transactions/{id}` | Retrieve a Customer Balance Transaction | Customer Balance Transactions |
| POST | `/api/v1/pa/customers/create` | Create a Customer | Customers |
| GET | `/api/v1/pa/customers/{id}` | Retrieve a Customer | Customers |
| GET | `/api/v1/pa/customers` | List all Customers | Customers |
| POST | `/api/v1/pa/customers/{id}/update` | Update a Customer | Customers |
| GET | `/api/v1/pa/customers/{id}/generate_client_secret` | Generate a Client Secret for a Customer | Customers |
| POST | `/api/v1/pa/customs_declarations/create` | Create a Customs Declaration | Customs Declarations |
| GET | `/api/v1/pa/customs_declarations/{id}` | Retrieve a Customs Declaration | Customs Declarations |
| POST | `/api/v1/pa/customs_declarations/{id}/update` | Update a Customs Declaration | Customs Declarations |
| POST | `/api/v1/pa/customs_declarations/{id}/redeclare` | Redeclare a Customs Declaration | Customs Declarations |
| POST | `/api/v1/pa/funds_split_reversals/create` | Create a Funds Split Reversal | Funds Split Reversals |
| GET | `/api/v1/pa/funds_split_reversals/{id}` | Retrieve a Funds Split Reversal | Funds Split Reversals |
| GET | `/api/v1/pa/funds_split_reversals` | List all Funds Split Reversals | Funds Split Reversals |
| POST | `/api/v1/pa/funds_splits/create` | Create a Funds Split | Funds Splits |
| GET | `/api/v1/pa/funds_splits/{id}` | Retrieve a Funds Split | Funds Splits |
| GET | `/api/v1/pa/funds_splits` | List all Funds Splits | Funds Splits |
| POST | `/api/v1/pa/funds_splits/{id}/release` | Release a Funds Split | Funds Splits |
| GET | `/api/v1/pa/payment_attempts/{id}` | Retrieve a Payment Attempt | Payment Attempts |
| GET | `/api/v1/pa/payment_attempts` | List all Payment Attempts | Payment Attempts |
| POST | `/api/v1/pa/payment_consents/create` | Create a Payment Consent | Payment Consents |
| GET | `/api/v1/pa/payment_consents/{id}` | Retrieve a Payment Consent | Payment Consents |
| GET | `/api/v1/pa/payment_consents` | List all Payment Consents | Payment Consents |
| POST | `/api/v1/pa/payment_consents/{id}/update` | Update a Payment Consent | Payment Consents |
| POST | `/api/v1/pa/payment_consents/{id}/verify` | Verify a Payment Consent | Payment Consents |
| POST | `/api/v1/pa/payment_consents/{id}/verify_continue` | Continue To Verify a Payment Consent | Payment Consents |
| POST | `/api/v1/pa/payment_consents/{id}/disable` | Disable a Payment Consent | Payment Consents |
| GET | `/api/v1/pa/payment_disputes/{id}` | Retrieve a Payment Dispute | Payment Disputes |
| GET | `/api/v1/pa/payment_disputes` | List all Payment Disputes | Payment Disputes |
| POST | `/api/v1/pa/payment_disputes/{id}/update` | Update a Payment Dispute | Payment Disputes |
| POST | `/api/v1/pa/payment_disputes/{id}/accept` | Accept a Payment Dispute | Payment Disputes |
| POST | `/api/v1/pa/payment_disputes/{id}/challenge` | Challenge a Payment Dispute | Payment Disputes |
| GET | `/api/v1/pa/payment_disputes/{id}/related_payment_intents` | List all Related Payment Intents | Payment Disputes |
| POST | `/api/v1/pa/payment_intents/create` | Create a Payment Intent | Payment Intents |
| GET | `/api/v1/pa/payment_intents/{id}` | Retrieve a Payment Intent | Payment Intents |
| GET | `/api/v1/pa/payment_intents` | List all Payment Intents | Payment Intents |
| POST | `/api/v1/pa/payment_intents/{id}/update` | Update a Payment Intent | Payment Intents |
| POST | `/api/v1/pa/payment_intents/{id}/confirm` | Confirm a Payment Intent | Payment Intents |
| POST | `/api/v1/pa/payment_intents/{id}/capture` | Capture a Payment Intent | Payment Intents |
| POST | `/api/v1/pa/payment_intents/{id}/confirm_continue` | Continue To Confirm a Payment Intent | Payment Intents |
| POST | `/api/v1/pa/payment_intents/{id}/increment_authorization` | Increment Authorization for a Payment Intent | Payment Intents |
| POST | `/api/v1/pa/payment_intents/{id}/cancel` | Cancel a Payment Intent | Payment Intents |
| POST | `/api/v1/pa/payment_links/create` | Create a Payment Link | Payment Links |
| GET | `/api/v1/pa/payment_links/{id}` | Retrieve a Payment Link | Payment Links |
| GET | `/api/v1/pa/payment_links` | List all Payment Links | Payment Links |
| POST | `/api/v1/pa/payment_links/{id}/update` | Update a Payment Link | Payment Links |
| POST | `/api/v1/pa/payment_links/{id}/notify_shopper` | Send a Payment Link | Payment Links |
| POST | `/api/v1/pa/payment_links/{id}/activate` | Activate a Payment Link | Payment Links |
| POST | `/api/v1/pa/payment_links/{id}/deactivate` | Deactivate a Payment Link | Payment Links |
| POST | `/api/v1/pa/payment_methods/create` | Create a Payment Method | Payment Methods |
| GET | `/api/v1/pa/payment_methods/{id}` | Retrieve a Payment Method | Payment Methods |
| GET | `/api/v1/pa/payment_methods` | List all Payment Methods | Payment Methods |
| POST | `/api/v1/pa/payment_methods/{id}/update` | Update a Payment Method | Payment Methods |
| POST | `/api/v1/pa/payment_methods/{id}/disable` | Disable a Payment Method | Payment Methods |
| GET | `/api/v1/pa/reference/bin/lookup` | Retrieve the BIN Info | Reference Data |
| POST | `/api/v1/pa/refunds/create` | Create a Refund | Refunds |
| GET | `/api/v1/pa/refunds/{id}` | Retrieve a Refund | Refunds |
| GET | `/api/v1/pa/refunds` | List all Refunds | Refunds |
| GET | `/api/v1/pa/settlement_records` | List all Settlement Records | Settlement Records |
| POST | `/api/v1/pa/pos/terminals/create` | Create a Terminal | Terminals |
| GET | `/api/v1/pa/pos/terminals/{id}` | Retrieve a Terminal | Terminals |
| GET | `/api/v1/pa/pos/terminals` | List all Terminals | Terminals |
| POST | `/api/v1/pa/pos/terminals/{id}/update` | Update a Terminal | Terminals |
| POST | `/api/v1/pa/pos/terminals/{id}/activate` | Activate a Terminal | Terminals |
| POST | `/api/v1/pa/pos/terminals/{id}/process_payment_intent` | Process a Payment Intent in a Terminal | Terminals |
| POST | `/api/v1/pa/pos/terminals/{id}/reset_password` | Reset a Terminal Password | Terminals |
| POST | `/api/v1/pa/pos/terminals/{id}/deactivate` | Deactivate a Terminal | Terminals |
| POST | `/api/v1/pa/pos/terminals/{id}/terminate` | Terminate a Terminal | Terminals |
| POST | `/api/v1/pa/pos/terminals/{id}/cancel_current_operation` | Cancel Current Operation on a Terminal | Terminals |

### Checkout Sessions

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/checkout/checkout_sessions/create` | Create a Checkout Session | Checkout Sessions |
| GET | `/api/v1/checkout/checkout_sessions/{id}` | Retrieve a Checkout Session | Checkout Sessions |
| GET | `/api/v1/checkout/checkout_sessions` | Get list of Checkout Sessions | Checkout Sessions |
| POST | `/api/v1/checkout/checkout_sessions/{id}/update` | Update a Checkout Session | Checkout Sessions |
| POST | `/api/v1/checkout/checkout_sessions/{id}/cancel` | Cancel a Checkout Session | Checkout Sessions |

### Webhook management

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/webhooks/create` | Create a Webhook | Webhooks |
| GET | `/api/v1/webhooks/{id}` | Retrieve a Webhook | Webhooks |
| GET | `/api/v1/webhooks` | List all Webhooks | Webhooks |
| POST | `/api/v1/webhooks/{id}/update` | Update a Webhook | Webhooks |
| POST | `/api/v1/webhooks/{id}/delete` | Delete a Webhook | Webhooks |

### Finance (settlements, financial transactions, reports)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/finance/financial_reports/create` | Create a Financial Report | Financial Reports |
| GET | `/api/v1/finance/financial_reports/{id}` | Retrieve a Financial Report | Financial Reports |
| GET | `/api/v1/finance/financial_reports` | List all Financial Reports | Financial Reports |
| GET | `/api/v1/finance/financial_reports/{id}/content` | Retrieve a Contents of a Financial Report | Financial Reports |
| GET | `/api/v1/financial_transactions/{id}` | Retrieve a Financial Transaction | Financial Transactions |
| GET | `/api/v1/financial_transactions` | List all Financial Transactions | Financial Transactions |
| GET | `/api/v1/pa/financial/settlements/{id}` | Retrieve a Settlement | Settlements |
| GET | `/api/v1/pa/financial/settlements` | List all Settlements | Settlements |
| GET | `/api/v1/pa/financial/settlements/{id}/report` | Retrieve a Settlement Report | Settlements |

### Simulation (sandbox only)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/simulation/account/amendments/{id}/approve` | Approve an Account Amendment | Accounts |
| POST | `/api/v1/simulation/account/amendments/{id}/reject` | Reject an Account Amendment | Accounts |
| POST | `/api/v1/simulation/accounts/{account_id}/update_status` | Update Status of Connected Account | Accounts |
| POST | `/api/v1/simulation/billing/payment_sources/{id}/fail_next_autocharge` | Fail Next Autocharge | Billing |
| POST | `/api/v1/simulation/deposit/create` | Create a Global Account Deposit | Deposits |
| POST | `/api/v1/simulation/deposits/{deposit_id}/reverse` | Reverse a Direct Debit Deposit | Deposits |
| POST | `/api/v1/simulation/deposits/{deposit_id}/settle` | Settle a Direct Debit Deposit | Deposits |
| POST | `/api/v1/simulation/deposits/{deposit_id}/reject` | Reject a Direct Debit Deposit | Deposits |
| POST | `/api/v1/simulation/issuing/create` | Create a Transaction for the Provided Card | Issuing |
| POST | `/api/v1/simulation/issuing/card_transaction_lifecycles/{card_transaction_lifecycle_id}/capture` | Capture the Card Transaction Lifecycle | Issuing |
| POST | `/api/v1/simulation/issuing/refund` | Refund a Transaction for the Provided Card | Issuing |
| POST | `/api/v1/simulation/issuing/card_transaction_lifecycles/{card_transaction_lifecycle_id}/reverse` | Reverse the Card Transaction Lifecycle with the Provided Id | Issuing |
| POST | `/api/v1/simulation/issuing/cardholders/{id}/pass_review` | Pass Review Stage of a Cardholder | Issuing |
| POST | `/api/v1/simulation/issuing/threeds/notify` | Send Threeds Notification for the Provided Card | Issuing |
| POST | `/api/v1/simulation/linked_accounts/{linked_account_id}/mandate/accept` | Accept a Mandate | Linked Accounts |
| POST | `/api/v1/simulation/linked_accounts/{linked_account_id}/mandate/reject` | Reject a Mandate | Linked Accounts |
| POST | `/api/v1/simulation/linked_accounts/{linked_account_id}/mandate/cancel` | Cancel a Mandate | Linked Accounts |
| POST | `/api/v1/simulation/linked_accounts/{linked_account_id}/fail_microdeposits` | Fail a Micro Deposits Verification | Linked Accounts |
| POST | `/api/v1/simulation/accounts/{account_id}/offboardings/{id}/cancel` | Cancel an Offboarding | Offboardings |
| POST | `/api/v1/simulation/accounts/{account_id}/offboardings/{id}/complete` | Complete an Offboarding | Offboardings |
| POST | `/api/v1/simulation/pa/shopper_actions/{action}` | Simulate a Shopper Action | Payment Acceptance |
| POST | `/api/v1/simulation/pa/payment_disputes/create` | Create a Payment Dispute | Payment Acceptance |
| POST | `/api/v1/simulation/pa/payment_disputes/{id}/resolve` | Resolve a Payment Dispute | Payment Acceptance |
| POST | `/api/v1/simulation/pa/payment_disputes/{id}/escalate` | Escalate a Payment Dispute | Payment Acceptance |
| POST | `/api/v1/simulation/rfis/create` | Create an RFI | Request for Information (RFI) |
| POST | `/api/v1/simulation/rfis/{id}/follow_up` | Follow-up RFI | Request for Information (RFI) |
| POST | `/api/v1/simulation/rfis/{id}/close` | Close an RFI | Request for Information (RFI) |
| POST | `/api/v1/simulation/pa/pos/terminals/generate_activation_code` | Generate an Activation Code | Terminal Simulation |
| POST | `/api/v1/simulation/pa/pos/terminals/turn_on` | Turn on a Terminal | Terminal Simulation |
| POST | `/api/v1/simulation/pa/pos/terminals/turn_off` | Turn Off a Terminal | Terminal Simulation |
| POST | `/api/v1/simulation/pa/pos/terminals/confirm_payment_intent` | Confirm Payment Using a Scenario | Terminal Simulation |
| GET | `/api/v1/simulation/pa/pos/terminals/payment_scenarios` | List all Available Payment Scenarios | Terminal Simulation |
| POST | `/api/v1/simulation/transfers/{id}/transition` | Transition Transfer Status | Transfers |

### Account capabilities (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| GET | `/api/v1/account_capabilities/{id}` | Retrieve an Account Capability | Account Capability |
| POST | `/api/v1/account_capabilities/{id}/enable` | Enable an Account Capability | Account Capability |
| GET | `/api/v1/account_capabilities/funding_limits` | List all Funding Limits | Account Capability |

### Accounting (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/accounting/accounting_field_values/create` | Create an Accounting Field Value | Accounting Field Values |
| GET | `/api/v1/accounting/accounting_field_values/{id}` | Retrieve an Accounting Field Value | Accounting Field Values |
| GET | `/api/v1/accounting/accounting_field_values` | List all Accounting Field Values | Accounting Field Values |
| POST | `/api/v1/accounting/accounting_field_values/{id}/update` | Update an Accounting Field Value | Accounting Field Values |
| POST | `/api/v1/accounting/accounting_fields/create` | Create an Accounting Field | Accounting Fields |
| GET | `/api/v1/accounting/accounting_fields/{id}` | Retrieve an Accounting Field | Accounting Fields |
| GET | `/api/v1/accounting/accounting_fields` | List all Accounting Fields | Accounting Fields |
| POST | `/api/v1/accounting/accounting_fields/{id}/update` | Update an Accounting Field | Accounting Fields |
| POST | `/api/v1/accounting/gl_accounts/create` | Create a General Ledger Account | General Ledger Accounts |
| GET | `/api/v1/accounting/gl_accounts/{id}` | Retrieve a General Ledger Account | General Ledger Accounts |
| GET | `/api/v1/accounting/gl_accounts` | List all General Ledger Accounts | General Ledger Accounts |
| POST | `/api/v1/accounting/gl_accounts/{id}/update` | Update a General Ledger Account | General Ledger Accounts |
| POST | `/api/v1/accounting/tax_codes/create` | Create a Tax Code | Tax Codes |
| GET | `/api/v1/accounting/tax_codes/{id}` | Retrieve a Tax Code | Tax Codes |
| GET | `/api/v1/accounting/tax_codes` | List all Tax Codes | Tax Codes |
| POST | `/api/v1/accounting/tax_codes/{id}/update` | Update a Tax Code | Tax Codes |

### Auto payments (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/auto_payments/create` | Create an Auto Payment | Auto Payments |
| GET | `/api/v1/auto_payments/{id}` | Retrieve an Auto Payment | Auto Payments |
| GET | `/api/v1/auto_payments` | List all Auto Payments | Auto Payments |
| POST | `/api/v1/auto_payments/{id}/update` | Update an Auto Payment | Auto Payments |
| POST | `/api/v1/auto_payments/{id}/enable` | Enable an Auto Payment | Auto Payments |
| POST | `/api/v1/auto_payments/{id}/disable` | Disable an Auto Payment | Auto Payments |
| GET | `/api/v1/auto_payments/task` | List all Auto Payment Tasks | Auto Payments |

### Billing (subscriptions, invoices) (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/billing/billing_checkouts/create` | Create a Billing Checkout | Billing Checkouts |
| GET | `/api/v1/billing/billing_checkouts/{id}` | Retrieve a Billing Checkout | Billing Checkouts |
| GET | `/api/v1/billing/billing_checkouts` | List all Billing Checkouts | Billing Checkouts |
| POST | `/api/v1/billing/billing_checkouts/{id}/update` | Update a Billing Checkout | Billing Checkouts |
| POST | `/api/v1/billing/billing_checkouts/{id}/cancel` | Cancel a Billing Checkout | Billing Checkouts |
| POST | `/api/v1/billing/billing_customers/create` | Create a Billing Customer | Billing Customers |
| GET | `/api/v1/billing/billing_customers/{id}` | Retrieve a Billing Customer | Billing Customers |
| GET | `/api/v1/billing/billing_customers/{id}/bank_transfer_instructions` | Retrieve Bank Transfer Instructions of a Billing Customer | Billing Customers |
| GET | `/api/v1/billing/billing_customers` | List all Billing Customers | Billing Customers |
| POST | `/api/v1/billing/billing_customers/{id}/update` | Update a Billing Customer | Billing Customers |
| GET | `/api/v1/billing/billing_transactions/{id}` | Retrieve a Billing Transaction | Billing Transactions |
| GET | `/api/v1/billing/billing_transactions` | List all Billing Transactions | Billing Transactions |
| POST | `/api/v1/billing/coupons/create` | Create a Coupon | Coupons |
| GET | `/api/v1/billing/coupons/{id}` | Retrieve a Coupon | Coupons |
| GET | `/api/v1/billing/coupons` | List all Coupons | Coupons |
| POST | `/api/v1/billing/coupons/{id}/update` | Update a Coupon | Coupons |
| POST | `/api/v1/billing/credit_notes/create` | Create a Credit Note | Credit Notes |
| GET | `/api/v1/billing/credit_notes/{id}` | Retrieve a Credit Note | Credit Notes |
| GET | `/api/v1/billing/credit_notes` | List all Credit Notes | Credit Notes |
| POST | `/api/v1/billing/credit_notes/{id}/update` | Update a Credit Note | Credit Notes |
| POST | `/api/v1/billing/credit_notes/preview` | Preview a Credit Note | Credit Notes |
| POST | `/api/v1/billing/credit_notes/{id}/finalize` | Finalize a Credit Note | Credit Notes |
| POST | `/api/v1/billing/credit_notes/{id}/delete` | Delete a Credit Note | Credit Notes |
| POST | `/api/v1/billing/credit_notes/{id}/void` | Void a Credit Note | Credit Notes |
| POST | `/api/v1/billing/credit_notes/{id}/add_line_items` | Create Credit Note Line Items and Add Them to a Credit Note | Credit Notes |
| GET | `/api/v1/billing/credit_notes/{id}/line_items/{line_item_id}` | Retrieve a Credit Note Line Item | Credit Notes |
| GET | `/api/v1/billing/credit_notes/{id}/line_items` | List all Credit Note Line Items | Credit Notes |
| POST | `/api/v1/billing/credit_notes/{id}/update_line_items` | Update Credit Note Line Items in a Credit Note | Credit Notes |
| POST | `/api/v1/billing/credit_notes/{id}/delete_line_items` | Delete Credit Note Line Items From a Credit Note | Credit Notes |
| GET | `/api/v1/billing/e_invoicing_documents/{id}` | Retrieve an E-Invoicing Document | E-Invoicing Documents |
| GET | `/api/v1/billing/e_invoicing_documents` | Get list of E-Invoicing Documents | E-Invoicing Documents |
| POST | `/api/v1/billing/invoices/create` | Create an Invoice | Invoices |
| GET | `/api/v1/billing/invoices/{id}` | Retrieve an Invoice | Invoices |
| GET | `/api/v1/billing/invoices` | List all Invoices | Invoices |
| POST | `/api/v1/billing/invoices/{id}/update` | Update an Invoice | Invoices |
| POST | `/api/v1/billing/invoices/preview` | Preview an Invoice | Invoices |
| POST | `/api/v1/billing/invoices/{id}/finalize` | Finalize an Invoice | Invoices |
| POST | `/api/v1/billing/invoices/{id}/mark_as_paid` | Mark an Invoice As Paid | Invoices |
| POST | `/api/v1/billing/invoices/{id}/pay` | Pay an Invoice | Invoices |
| POST | `/api/v1/billing/invoices/{id}/delete` | Delete a Draft Invoice | Invoices |
| POST | `/api/v1/billing/invoices/{id}/void` | Void an Invoice | Invoices |
| POST | `/api/v1/billing/invoices/{id}/add_line_items` | Create Invoice Line Items and Add Them to an Invoice | Invoices |
| GET | `/api/v1/billing/invoices/{id}/line_items/{item_id}` | Retrieve an Invoice Line Item | Invoices |
| GET | `/api/v1/billing/invoices/{id}/line_items` | List all Invoice Line Items | Invoices |
| POST | `/api/v1/billing/invoices/{id}/update_line_items` | Update Invoice Line Items in an Invoice | Invoices |
| POST | `/api/v1/billing/invoices/{id}/delete_line_items` | Delete Invoice Line Items Within an Invoice | Invoices |
| GET | `/api/v1/billing/invoices/{id}/line_item_groups/{group_id}` | Retrieve an Invoice Line Item Group | Invoices |
| GET | `/api/v1/billing/invoices/{id}/line_item_groups` | List all Invoice Line Item Groups | Invoices |
| POST | `/api/v1/billing/meters/create` | Create a Meter | Meters |
| GET | `/api/v1/billing/meters/{id}` | Retrieve a Meter | Meters |
| GET | `/api/v1/billing/meters` | List all Meters | Meters |
| POST | `/api/v1/billing/meters/{id}/update` | Update a Meter | Meters |
| POST | `/api/v1/billing/meters/{id}/restore` | Restore a Meter | Meters |
| POST | `/api/v1/billing/meters/{id}/archive` | Archive a Meter | Meters |
| GET | `/api/v1/billing/meters/{id}/summaries` | Retrieve Summaries of a Meter | Meters |
| POST | `/api/v1/billing/payment_sources/create` | Create a Payment Source | Payment Sources |
| GET | `/api/v1/billing/payment_sources/{id}` | Retrieve a Payment Source | Payment Sources |
| GET | `/api/v1/billing/payment_sources` | List all Payment Sources | Payment Sources |
| POST | `/api/v1/billing/prices/create` | Create a Price | Prices |
| GET | `/api/v1/billing/prices/{id}` | Retrieve a Price | Prices |
| GET | `/api/v1/billing/prices` | List all Prices | Prices |
| POST | `/api/v1/billing/prices/{id}/update` | Update a Price | Prices |
| POST | `/api/v1/billing/products/create` | Create a Product | Products |
| GET | `/api/v1/billing/products/{id}` | Retrieve a Product | Products |
| GET | `/api/v1/billing/products` | List all Products | Products |
| POST | `/api/v1/billing/products/{id}/update` | Update a Product | Products |
| POST | `/api/v1/billing/rate_cards/create` | Create a Rate Card | Rate Cards |
| GET | `/api/v1/billing/rate_cards/{id}` | Retrieve a Rate Card | Rate Cards |
| GET | `/api/v1/billing/rate_cards` | List all Rate Cards | Rate Cards |
| POST | `/api/v1/billing/rate_cards/{id}/update` | Update a Rate Card | Rate Cards |
| POST | `/api/v1/billing/rate_cards/{id}/versions/create` | Create a Rate Card Version | Rate Cards |
| GET | `/api/v1/billing/rate_cards/{id}/versions/{version}` | Retrieve a Rate Card Version | Rate Cards |
| GET | `/api/v1/billing/rate_cards/{id}/versions` | List all Rate Card Versions | Rate Cards |
| GET | `/api/v1/billing/rate_cards/{id}/versions/{version}/items/{item_id}` | Retrieve a Rate Card Item | Rate Cards |
| GET | `/api/v1/billing/rate_cards/{id}/versions/{version}/items` | List all Rate Card Items | Rate Cards |
| POST | `/api/v1/billing/subscriptions/create` | Create a Subscription | Subscriptions |
| GET | `/api/v1/billing/subscriptions/{id}` | Retrieve a Subscription | Subscriptions |
| GET | `/api/v1/billing/subscriptions` | List all Subscriptions | Subscriptions |
| POST | `/api/v1/billing/subscriptions/{id}/update` | Update a Subscription | Subscriptions |
| POST | `/api/v1/billing/subscriptions/{id}/cancel` | Cancel a Subscription | Subscriptions |
| GET | `/api/v1/billing/subscriptions/{id}/items/{item_id}` | Retrieve a Subscription Item | Subscriptions |
| GET | `/api/v1/billing/subscriptions/{id}/items` | List all Subscription Items | Subscriptions |
| POST | `/api/v1/billing/usage_events/ingest` | Ingest a Usage Event | Usage Events |
| POST | `/api/v1/billing/usage_events/batch_ingest` | Batch Ingest Usage Events | Usage Events |
| POST | `/api/v1/billing/usage_events/void` | Void a Usage Event | Usage Events |

### Confirmation letter (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/confirmation_letters/create` | Create a Confirmation Letter | Confirmation Letter |

### Core resources (balances, deposits, global accounts, linked accounts, direct debits) (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| GET | `/api/v1/balances/current` | List all Current Balances | Balances |
| GET | `/api/v1/balances/history` | Retrieve the Balance History | Balances |
| POST | `/api/v1/deposits/create` | Create a Deposit Via Direct Debit | Deposits |
| GET | `/api/v1/deposits/{id}` | Retrieve a Deposit | Deposits |
| GET | `/api/v1/deposits` | List all Deposits | Deposits |
| GET | `/api/v1/direct_debits/{transaction_id}` | Retrieve a Direct Debit | Direct Debits |
| GET | `/api/v1/direct_debits` | List all Direct Debits | Direct Debits |
| POST | `/api/v1/direct_debits/{transaction_id}/cancel` | Cancel a Direct Debit | Direct Debits |
| POST | `/api/v1/global_accounts/create` | Create a Global Account | Global Accounts |
| GET | `/api/v1/global_accounts/{id}` | Retrieve a Global Account | Global Accounts |
| GET | `/api/v1/global_accounts` | List all Global Accounts | Global Accounts |
| POST | `/api/v1/global_accounts/{id}/update` | Update an Existing Global Account | Global Accounts |
| POST | `/api/v1/global_accounts/{id}/generate_statement_letter` | Generate Global Account Statement | Global Accounts |
| POST | `/api/v1/global_accounts/{id}/close` | Close a Global Account | Global Accounts |
| POST | `/api/v1/global_accounts/{global_account_id}/aliases/create` | Create an Alias for a Global Account | Global Accounts |
| GET | `/api/v1/global_accounts/{global_account_id}/aliases/{id}` | Retrieve an Alias | Global Accounts |
| GET | `/api/v1/global_accounts/{global_account_id}/aliases` | List all Aliases | Global Accounts |
| POST | `/api/v1/global_accounts/{global_account_id}/aliases/{id}/initiate_port` | Initiate an Alias Port for a Global Account | Global Accounts |
| POST | `/api/v1/global_accounts/{global_account_id}/aliases/{id}/submit_verification_code` | Submit Alias Verification Code | Global Accounts |
| POST | `/api/v1/global_accounts/{global_account_id}/aliases/{id}/request_new_verification_code` | Request New Alias Verification Code | Global Accounts |
| POST | `/api/v1/global_accounts/{global_account_id}/aliases/{id}/cancel` | Cancel an Alias for a Global Account | Global Accounts |
| GET | `/api/v1/global_accounts/{global_account_id}/mandates/{id}` | Retrieve a Direct Debit Payout Mandate | Global Accounts |
| GET | `/api/v1/global_accounts/{global_account_id}/mandates` | List all Direct Debit Payout Mandates | Global Accounts |
| POST | `/api/v1/global_accounts/{global_account_id}/mandates/{id}/cancel` | Cancel a Direct Debit Payout Mandate | Global Accounts |
| GET | `/api/v1/global_accounts/{id}/transactions` | List all Global Account Transactions | Global Accounts |
| POST | `/api/v1/linked_accounts/create` | Create a Linked Account | Linked Accounts |
| GET | `/api/v1/linked_accounts/{id}` | Retrieve a Linked Account | Linked Accounts |
| GET | `/api/v1/linked_accounts` | List all Linked Accounts | Linked Accounts |
| POST | `/api/v1/linked_accounts/auth` | Generate a Linked Account Authentication | Linked Accounts |
| POST | `/api/v1/linked_accounts/{id}/auth` | Refresh a Linked Account Authentication | Linked Accounts |
| POST | `/api/v1/linked_accounts/{id}/verify_microdeposits` | Verify Linked Account with Micro-deposits | Linked Accounts |
| GET | `/api/v1/linked_accounts/{id}/balances` | Check Available Balances | Linked Accounts |
| POST | `/api/v1/linked_accounts/{id}/suspend` | Suspend a Linked Account | Linked Accounts |
| POST | `/api/v1/linked_accounts/{id}/confirm` | Confirm a Linked Account Among Candidate Accounts | Linked Accounts |
| POST | `/api/v1/linked_accounts/{id}/complete_auth` | Complete the Authentication Refresh | Linked Accounts |
| GET | `/api/v1/linked_accounts/{id}/mandate` | Retrieve a Direct Debit Deposit Mandate | Linked Accounts |
| POST | `/api/v1/linked_accounts/{id}/mandate` | Update a Direct Debit Deposit Mandate | Linked Accounts |

### Issuing (cards) (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| GET | `/api/v1/issuing/authorizations/{id}` | Retrieve an Authorization | Authorizations |
| GET | `/api/v1/issuing/authorizations` | List all Authorizations | Authorizations |
| GET | `/api/v1/issuing/card_transaction_events/{id}` | Retrieve a Card Transaction Event | Card Transaction Events |
| GET | `/api/v1/issuing/card_transaction_events` | List all Card Transaction Events | Card Transaction Events |
| GET | `/api/v1/issuing/lifecycles/{id}` | Retrieve a Card Transaction Lifecycle | Card Transaction Lifecycles |
| GET | `/api/v1/issuing/lifecycles` | List all Card Transaction Lifecycles | Card Transaction Lifecycles |
| GET | `/api/v1/issuing/card_transactions/{id}` | Retrieve a Card Transaction | Card Transactions |
| GET | `/api/v1/issuing/card_transactions` | List all Card Transactions | Card Transactions |
| POST | `/api/v1/issuing/cardholders/create` | Create a Cardholder | Cardholders |
| GET | `/api/v1/issuing/cardholders/{id}` | Retrieve the Cardholder Details | Cardholders |
| GET | `/api/v1/issuing/cardholders` | List all Cardholders | Cardholders |
| POST | `/api/v1/issuing/cardholders/{id}/update` | Update a Cardholder | Cardholders |
| POST | `/api/v1/issuing/cardholders/{id}/delete` | Delete a Cardholder | Cardholders |
| POST | `/api/v1/issuing/cards/create` | Create a Card | Cards |
| GET | `/api/v1/issuing/cards/{id}` | Retrieve a Card | Cards |
| GET | `/api/v1/issuing/cards` | List all Cards | Cards |
| POST | `/api/v1/issuing/cards/{id}/update` | Update a Card | Cards |
| POST | `/api/v1/issuing/cards/{id}/activate` | Activate a Card | Cards |
| POST | `/api/v1/issuing/cards/{id}/provision_digital_token` | Provision a Card to a Digital Wallet | Cards |
| GET | `/api/v1/issuing/cards/{id}/details` | List all Sensitive Card Details | Cards |
| GET | `/api/v1/issuing/cards/{id}/limits` | List all Card Remaining Limits | Cards |
| GET | `/api/v1/issuing/config` | Retrieve the Issuing Config | Config |
| POST | `/api/v1/issuing/config/update` | Update the Issuing Config | Config |
| GET | `/api/v1/issuing/digital_wallet_tokens/{id}` | Retrieve a Digital Wallet Token | Digital Wallet Tokens |
| GET | `/api/v1/issuing/digital_wallet_tokens` | List all Digital Wallet Tokens | Digital Wallet Tokens |
| POST | `/api/v1/issuing/digital_wallet_tokens/{id}/lifecycle` | Update the Digital Wallet Token Status | Digital Wallet Tokens |
| GET | `/api/v1/issuing/merchant_brands/{id}` | Retrieve a Merchant Brand | Merchant Brands |
| GET | `/api/v1/issuing/merchant_brands` | List all Merchant Brands | Merchant Brands |
| POST | `/api/v1/issuing/transaction_disputes/create` | Create a Transaction Dispute | Transaction Disputes |
| GET | `/api/v1/issuing/transaction_disputes/{id}` | Retrieve a Transaction Dispute | Transaction Disputes |
| GET | `/api/v1/issuing/transaction_disputes` | List all Transaction Disputes | Transaction Disputes |
| POST | `/api/v1/issuing/transaction_disputes/{id}/update` | Update a Transaction Dispute | Transaction Disputes |
| POST | `/api/v1/issuing/transaction_disputes/{id}/submit` | Submit a Transaction Dispute | Transaction Disputes |
| POST | `/api/v1/issuing/transaction_disputes/{id}/cancel` | Cancel a Transaction Dispute | Transaction Disputes |
| GET | `/api/v1/issuing/transactions/{id}` | Retrieve a Transaction | Transactions |
| GET | `/api/v1/issuing/transactions` | List all Transactions | Transactions |

### Lending (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/lending/lenders/{lender_id}/borrowers/create` | Create a Borrower | Borrower |
| GET | `/api/v1/lending/lenders/{lender_id}/borrowers` | List all Borrowers for This Lender | Borrower |
| GET | `/api/v1/lending/lenders/{lender_id}/borrowers/{borrower_id}` | List all Borrower Details | Borrower |
| POST | `/api/v1/lending/lenders/{lender_id}/borrowers/{borrower_id}/update` | Update Borrower Settings | Borrower |
| GET | `/api/v1/lending/lenders/{lender_id}` | List all Lender Details | Lender |

### Payouts (transfers, beneficiaries) (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/batch_transfers/create` | Create a Batch Transfer | Batch Transfers |
| GET | `/api/v1/batch_transfers/{id}` | Retrieve a Batch Transfer | Batch Transfers |
| GET | `/api/v1/batch_transfers` | List all Batch Transfers | Batch Transfers |
| POST | `/api/v1/batch_transfers/{id}/add_items` | Add Items To a Batch | Batch Transfers |
| POST | `/api/v1/batch_transfers/{id}/quote` | Quote a Batch Transfer | Batch Transfers |
| POST | `/api/v1/batch_transfers/{id}/submit` | Submit a Batch Transfer | Batch Transfers |
| POST | `/api/v1/batch_transfers/{id}/delete` | Delete a Batch Transfer | Batch Transfers |
| POST | `/api/v1/batch_transfers/{id}/delete_items` | Delete Items Within a Batch | Batch Transfers |
| GET | `/api/v1/batch_transfers/{id}/items` | List all Items Within a Batch | Batch Transfers |
| POST | `/api/v1/beneficiaries/create` | Create a Beneficiary | Beneficiaries |
| GET | `/api/v1/beneficiaries/{id}` | Retrieve a Beneficiary | Beneficiaries |
| GET | `/api/v1/beneficiaries` | List all Beneficiaries | Beneficiaries |
| POST | `/api/v1/beneficiaries/{id}/update` | Update an Existing Beneficiary | Beneficiaries |
| POST | `/api/v1/beneficiaries/validate` | Validate a Beneficiary | Beneficiaries |
| POST | `/api/v1/beneficiaries/verify_account` | Verify a Beneficiary Account | Beneficiaries |
| POST | `/api/v1/beneficiaries/{id}/delete` | Delete an Existing Beneficiary | Beneficiaries |
| POST | `/api/v1/beneficiary_api_schemas/generate` | Retrieve the API Schema | Beneficiaries |
| POST | `/api/v1/beneficiary_form_schemas/generate` | Retrieve the Form Schema | Beneficiaries |
| GET | `/api/v1/beneficiary_form_schemas/supported_financial_institutions` | List all Supported Financial Institutions | Beneficiaries |
| POST | `/api/v1/payers/create` | Create a Payer | Payers |
| GET | `/api/v1/payers/{id}` | Retrieve a Payer | Payers |
| GET | `/api/v1/payers` | List all Payers | Payers |
| POST | `/api/v1/payers/{id}/update` | Update an Existing Payer | Payers |
| POST | `/api/v1/payers/validate` | Validate a Payer | Payers |
| POST | `/api/v1/payers/{id}/delete` | Delete an Existing Payer | Payers |
| POST | `/api/v1/transfers/create` | Create a Transfer | Transfers |
| GET | `/api/v1/transfers/{id}` | Retrieve a Transfer | Transfers |
| GET | `/api/v1/transfers` | List all Transfers | Transfers |
| POST | `/api/v1/transfers/{id}/confirm_funding` | Confirm Funding for a Transfer | Transfers |
| POST | `/api/v1/transfers/validate` | Validate a Transfer | Transfers |
| POST | `/api/v1/transfers/{id}/cancel` | Cancel a Transfer | Transfers |

### Reconciliation (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| GET | `/api/v1/tc/balances` | List all Balances | Reconciliation |
| POST | `/api/v1/tc/balances/requests` | Retrieve the Balances in Batch | Reconciliation |
| GET | `/api/v1/tc/transactions` | List all Transactions | Reconciliation |
| POST | `/api/v1/tc/transactions/requests` | Retrieve the Transactions in Batch | Reconciliation |

### Reserves (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/reserves/create` | Create a Reserve | Reserves |
| POST | `/api/v1/reserves/unreserve` | Create an Unreserve | Reserves |

### Risk (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/risk/issuing/fraud_feedback` | Fraud Feedback (Issuing) | Fraud Feedback (Issuing) |
| GET | `/api/v1/rfis/{id}` | Retrieve an RFI | Request for Information (RFI) |
| GET | `/api/v1/rfis` | List all RFIs | Request for Information (RFI) |
| POST | `/api/v1/rfis/{id}/respond` | Respond to an RFI | Request for Information (RFI) |
| GET | `/api/v1/risk/risk_reports/{id}` | Retrieve a risk report | Risk Reports |
| POST | `/api/v1/risk/risk_reports/generate` | Generate a risk report | Risk Reports |
| POST | `/api/v1/sellers/create` | Create a Seller | Sellers |
| GET | `/api/v1/sellers/{id}` | Retrieve a Seller | Sellers |
| POST | `/api/v1/sellers/{id}/deactivate` | Deactivate a Seller | Sellers |
| GET | `/api/v1/risk/watchlist/{item_id}` | Retrieve an Item | Watchlist |
| GET | `/api/v1/risk/watchlist` | List all Items | Watchlist |
| POST | `/api/v1/risk/watchlist/{item_id}/update` | Update an Item | Watchlist |
| POST | `/api/v1/risk/watchlist/create` | Add an Item to the Watchlist | Watchlist |

### Scale / Payments for Platforms (connected accounts) (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/accounts/create` | Create an Account | Accounts |
| GET | `/api/v1/accounts/{id}` | Retrieve an Account | Accounts |
| GET | `/api/v1/account` | Retrieve Account Details | Accounts |
| GET | `/api/v1/accounts` | List all Connected Accounts | Accounts |
| POST | `/api/v1/accounts/{id}/update` | Update a Connected Account | Accounts |
| POST | `/api/v1/accounts/{id}/submit` | Submit an Account for Activation | Accounts |
| POST | `/api/v1/accounts/{id}/terms_and_conditions/agree` | Agree To Terms and Conditions | Accounts |
| POST | `/api/v1/accounts/{id}/suspend` | Suspend an Account | Accounts |
| POST | `/api/v1/accounts/{id}/reactivate` | Reactivate an Account | Accounts |
| POST | `/api/v1/account/amendments/create` | Create an Account Amendment | Accounts |
| GET | `/api/v1/account/amendments/{amendment_id}` | Retrieve an Account Amendment | Accounts |
| GET | `/api/v1/account/wallet_info` | Retrieve Account Wallet Information | Accounts |
| POST | `/api/v1/charges/create` | Create a Charge | Charges |
| GET | `/api/v1/charges/{id}` | Retrieve a Charge | Charges |
| GET | `/api/v1/charges` | List all Charges | Charges |
| POST | `/api/v1/connected_account_transfers/create` | Create a Connected Account Transfer | Connected Account Transfers |
| GET | `/api/v1/connected_account_transfers/{id}` | Retrieve a Connected Account Transfer | Connected Account Transfers |
| GET | `/api/v1/connected_account_transfers` | List all Connected Account Transfers | Connected Account Transfers |
| POST | `/api/v1/hosted_flows/create` | Create a Flow | Hosted Flows |
| GET | `/api/v1/hosted_flows/{id}` | Retrieve a Flow | Hosted Flows |
| POST | `/api/v1/hosted_flows/{id}/authorize` | Authorize a Flow | Hosted Flows |
| POST | `/api/v1/accounts/invitation_links/create` | Create an Account Invitation Link | Invitation Links |
| GET | `/api/v1/accounts/invitation_links/{id}` | Retrieve an Account Invitation Link | Invitation Links |
| POST | `/api/v1/accounts/{account_id}/offboardings/create` | Create an Offboarding | Offboardings |
| GET | `/api/v1/accounts/{account_id}/offboardings/{id}` | Retrieve an Offboarding | Offboardings |
| GET | `/api/v1/accounts/{account_id}/offboardings` | List all Offboardings | Offboardings |
| GET | `/api/v1/platform_liquidity_programs/{id}` | Retrieve a Program | Platform Liquidity Programs |
| POST | `/api/v1/platform_liquidity_programs/{id}/deposit` | Deposit Funds Into a Program | Platform Liquidity Programs |
| POST | `/api/v1/platform_liquidity_programs/{id}/withdraw` | Withdraw Funds From a Program | Platform Liquidity Programs |
| POST | `/api/v1/platform_liquidity_programs/{id}/program_spending_accounts/create` | Create a Program Spending Account | Platform Liquidity Programs |
| GET | `/api/v1/platform_liquidity_programs/{id}/program_spending_accounts/{spending_account_id}` | Retrieve a Program Spending Account | Platform Liquidity Programs |
| GET | `/api/v1/platform_liquidity_programs/{id}/program_spending_accounts` | List all Program Spending Accounts | Platform Liquidity Programs |
| POST | `/api/v1/platform_liquidity_programs/{id}/program_spending_accounts/{spending_account_id}/adjust_limits` | Adjust Program Spending Account Limits | Platform Liquidity Programs |
| GET | `/api/v1/platform_liquidity_programs/{id}/transactions` | List all Program Transactions | Platform Liquidity Programs |
| POST | `/api/v1/platform_reports/create` | Create a Platform Report | Platform Reports |
| GET | `/api/v1/platform_reports/{id}` | Retrieve a Platform Report | Platform Reports |
| GET | `/api/v1/psp_settlement_deposits` | List all PSP Settlement Deposits | PSP Settlement Deposits |
| POST | `/api/v1/psp_settlement_intents/create` | Create a PSP Settlement Intent | PSP Settlement Intents |
| GET | `/api/v1/psp_settlement_intents/{id}` | Retrieve a PSP Settlement Intent | PSP Settlement Intents |
| GET | `/api/v1/psp_settlement_intents` | List all PSP Settlement Intents | PSP Settlement Intents |
| POST | `/api/v1/psp_settlement_intents/{id}/update` | Update a PSP Settlement Intent | PSP Settlement Intents |
| POST | `/api/v1/psp_settlement_intents/{id}/submit` | Submit a PSP Settlement Intent | PSP Settlement Intents |
| POST | `/api/v1/psp_settlement_intents/{id}/cancel` | Cancel a PSP Settlement Intent | PSP Settlement Intents |
| GET | `/api/v1/psp_settlement_splits/{id}` | Retrieve a PSP Settlement Split | PSP Settlement Splits |
| GET | `/api/v1/psp_settlement_splits` | List all PSP Settlement Splits | PSP Settlement Splits |
| POST | `/api/v1/psp_settlement_intents/{psp_settlement_intent_id}/split` | Split a PSP Settlement Intent | PSP Settlement Splits |
| POST | `/api/v1/psp_settlement_splits/{id}/release` | Release a PSP Settlement Split | PSP Settlement Splits |
| POST | `/api/v1/psp_settlement_splits/{id}/cancel` | Cancel a PSP Settlement Split | PSP Settlement Splits |

### Scale settlements (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/settlement_reports/prepare` | Create a Batch Report | Scale Settlements |
| GET | `/api/v1/settlement_reports/{settlement_report_id}` | Retrieve a Settlement Report | Scale Settlements |

### Security (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| GET | `/api/v1/audit_log/security_audit_logs` | Security Audit Logs | Activity Log |

### Spend (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/spend/bills/create` | Create a Bill | Bills |
| GET | `/api/v1/spend/bills/{id}` | Retrieve a Bill | Bills |
| GET | `/api/v1/spend/bills` | List all Bills | Bills |
| POST | `/api/v1/spend/bills/{id}/mark_as_paid` | Mark Bill As Paid | Bills |
| POST | `/api/v1/spend/bills/{id}/sync` | Update Bill Sync Status | Bills |
| POST | `/api/v1/spend/expenses/{id}/add_line_items` | Add Expense Line Items | Expenses |
| POST | `/api/v1/spend/expenses/{id}/delete_line_items` | Delete Expense Line Items | Expenses |
| POST | `/api/v1/spend/expenses/{id}/update` | Update an Expense | Expenses |
| POST | `/api/v1/spend/expenses/{id}/update_line_items` | Update Expense Line Items | Expenses |
| GET | `/api/v1/spend/expenses/{id}` | Retrieve an Expense | Expenses |
| GET | `/api/v1/spend/expenses` | List all Expenses | Expenses |
| POST | `/api/v1/spend/expenses/{id}/sync` | Update Expense Sync Status | Expenses |
| POST | `/api/v1/spend/purchase_orders/create` | Create a Purchase Order | Purchase Orders |
| GET | `/api/v1/spend/purchase_orders/{id}` | Retrieve a Purchase Order | Purchase Orders |
| GET | `/api/v1/spend/purchase_orders` | List all Purchase Orders | Purchase Orders |
| POST | `/api/v1/spend/purchase_orders/{id}/sync` | Update Purchase Order Sync Status | Purchase Orders |
| GET | `/api/v1/spend/reimbursement_reports/{id}/reimbursements/{reimbursement_id}` | Retrieve a Reimbursement | Reimbursements |
| GET | `/api/v1/spend/reimbursement_reports/{id}` | Retrieve a Reimbursement Report | Reimbursements |
| GET | `/api/v1/spend/reimbursement_reports/{id}/transfers/{transfer_id}` | Retrieve a Reimbursement Report Transfer | Reimbursements |
| GET | `/api/v1/spend/reimbursement_reports/{id}/reimbursements` | List all Reimbursements | Reimbursements |
| GET | `/api/v1/spend/reimbursement_reports` | List all Reimbursement Reports | Reimbursements |
| POST | `/api/v1/spend/reimbursement_reports/{id}/sync` | Update Reimbursement Report Sync Status | Reimbursements |
| POST | `/api/v1/spend/reimbursement_reports/{id}/transfers/{transfer_id}/sync` | Update Reimbursement Report Transfer Sync Status | Reimbursements |
| POST | `/api/v1/spend/reimbursement_reports/{id}/mark_as_paid` | Mark Reimbursement Report As Paid | Reimbursements |
| POST | `/api/v1/spend/vendors/create` | Create a Vendor | Vendors |
| GET | `/api/v1/spend/vendors/{id}` | Retrieve a Vendor | Vendors |
| GET | `/api/v1/spend/vendors` | List all Vendors | Vendors |
| POST | `/api/v1/spend/vendors/{id}/sync` | Update Vendor Sync Status | Vendors |

### Supporting services (files, reference data, connected stores) (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| GET | `/api/v1/ecosystem/connected_stores/{id}` | Retrieve a Connected Store | Connected Stores |
| GET | `/api/v1/ecosystem/connected_stores` | List all Connected Stores | Connected Stores |
| POST | `/api/v1/files/upload` | Upload a File | File Service |
| POST | `/api/v1/files/download_links` | List all File Download Links | File Service |
| GET | `/api/v1/reference/industry_categories` | Industry Categories | Reference Data |
| GET | `/api/v1/reference/invalid_conversion_dates` | Invalid Conversion Dates | Reference Data |
| GET | `/api/v1/reference/supported_currencies` | Supported Currencies | Reference Data |
| GET | `/api/v1/reference/order_items/expiring_quota` | Retrieve the Expiring Transfer or Deposit Limit | Reference Data |
| GET | `/api/v1/reference/order_items/quota` | Retrieve the Transfer or Deposit Limit | Reference Data |
| POST | `/api/v1/reference/order_items/upload` | Increase Transfer or Deposit Limit | Reference Data |

### Transactional FX (rates, quotes, conversions) (not payment acceptance; listed for completeness)

| Method | Path | Operation | Resource |
| --- | --- | --- | --- |
| POST | `/api/v1/fx/conversions/create` | Create a Conversion | Conversion |
| GET | `/api/v1/fx/conversions/{conversion_id}` | Retrieve a Conversion | Conversion |
| GET | `/api/v1/fx/conversions` | List all Conversions | Conversion |
| POST | `/api/v1/fx/conversion_amendments/create` | Create an Amendment | Conversion Amendments |
| GET | `/api/v1/fx/conversion_amendments/{conversion_amendment_id}` | Retrieve an Amendment | Conversion Amendments |
| GET | `/api/v1/fx/conversion_amendments` | List all Amendments | Conversion Amendments |
| POST | `/api/v1/fx/conversion_amendments/quote` | Create an Amendment Quote | Conversion Amendments |
| POST | `/api/v1/fx/quotes/create` | Create a Quote | Quotes |
| GET | `/api/v1/fx/quotes/{quote_id}` | Retrieve a Quote | Quotes |
| GET | `/api/v1/fx/rates/current` | Retrieve a Current Rate | Rates |

