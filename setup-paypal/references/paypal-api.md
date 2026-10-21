# PayPal REST API Reference (Malaysia scope)

Entry point for factual PayPal guidance. Distilled from PayPal's official developer docs and the official OpenAPI schemas published at `https://developer.paypal.com/api/<api>/<version>/schema.json` (checked 2026-10-21). Detail lives in topic files:

- `paypal-orders.md`: Orders v2 and Payments v2 field tables (create, capture, authorize, void, reauthorize, refund), statuses, PATCH paths.
- `paypal-webhooks.md`: webhook signature verification (self and postback), headers, event names, Webhooks Management API.
- `paypal-checkout-js.md`: JavaScript SDK v6 and v5 (script URLs, init options, callbacks, query params).
- `paypal-subscriptions.md`: Catalog Products, Plans, Subscriptions, recurring webhooks, failure retry.
- `paypal-errors-testing.md`: error format, common error codes, sandbox accounts, test cards, negative testing.
- `paypal-reporting-payouts.md`: Transaction Search, Payouts (with Malaysia MYR rules), disputes, other APIs.
- `account-setup.md`: account, credentials, env vars, go-live, Malaysia account notes.
- `source-map.md`: every page and schema used.

## Malaysia Scope Summary

- MYR (`MYR`, 2 decimals) is a supported REST currency, but PayPal footnotes it as "supported as a payment currency and a currency balance for in-country PayPal accounts only". Source: `/api/codes/currency`. The FX page says the same for buyer and holding currency and adds that if the settlement account is outside Malaysia PayPal converts into the account's primary currency with a spread or fee. Source: `/v5/checkout/fx/currency-codes`.
- Malaysia country code `MY`; JS SDK v5 locales for MY are `ms-MY` and `en-MY`. Sources: `/api/codes/country-region`, `/sdk/js/v5/configuration`.
- PayPal standard checkout (PayPal wallet button, plus the PayPal-hosted "Debit or Credit Card" / guest flow where eligible) "works everywhere PayPal is available". Source: `/checkout/integrate`.
- Expanded (advanced) Checkout card fields are listed for 37 countries; Malaysia is not in that list. Saving cards to the vault requires Expanded Checkout approval, so card vaulting is not available to a Malaysia merchant per that list. Sources: `/expanded/eligibility`, `/v5/save-payment-methods/overview`.
- Pay Later, PayPal Credit and Venmo are listed only for other countries (not Malaysia). Source: `/expanded/eligibility`, `/sdk/js/set-up`.
- GrabPay via PayPal is listed for Singapore buyers in `SGD` only. Source: `/grabpay`.
- No FPX, DuitNow QR, Touch 'n Go or other Malaysian local methods appear anywhere in the PayPal developer docs (searched 2026-10-21). Do not promise them.
- Policy: "Do not use PayPal API operation calls to provide personal payments in India, Mexico, Malaysia, Singapore, and Taiwan." Source: `/api/rest/reference/policies-and-guidelines`.
- Payouts from Malaysia: Malaysian users can send MYR only to other Malaysian users; they cannot send payments that need exchange to or from MYR, or send MYR balances to non-Malaysian users. Source: `/payouts/supported-features`.

## Environments

| Item | Sandbox | Live |
| --- | --- | --- |
| REST API base | `https://api-m.sandbox.paypal.com` | `https://api-m.paypal.com` |
| JS SDK v6 script | `https://www.sandbox.paypal.com/web-sdk/v6/core` | `https://www.paypal.com/web-sdk/v6/core` |
| JS SDK v5 script | `https://www.paypal.com/sdk/js?client-id=<SANDBOX_CLIENT_ID>` | `https://www.paypal.com/sdk/js?client-id=<LIVE_CLIENT_ID>` |
| Buyer approval page | `https://www.sandbox.paypal.com/checkoutnow?token=<ORDER_ID>` | returned in `links` (`rel: approve` or `payer-action`) |
| Dashboard | Developer Dashboard, Sandbox toggle | Developer Dashboard, Live toggle |

Sources: `/api/rest/requests`, `/api/rest/production`, `/sdk/js/set-up`, `/sdk/js/v5/configuration`. Sandbox and live credentials are different; the sandbox app ID is not accepted by production. Note: PayPal's postback sample uses the older host `https://api.sandbox.paypal.com`; use `api-m` hosts for new code.

## Authentication

1. Get client ID and secret: Developer Dashboard > Apps & Credentials > REST API apps (Default Application or Create App). Source: `/api/rest`.
2. Exchange for an access token:

```bash
curl -X POST "https://api-m.sandbox.paypal.com/v1/oauth2/token" \
  -u "<CLIENT_ID>:<CLIENT_SECRET>" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials"
```

Response fields: `scope`, `access_token`, `token_type` (`Bearer`), `app_id`, `expires_in` (seconds), `nonce`. Call with `Authorization: Bearer <ACCESS_TOKEN>`. When the token expires, call `/v1/oauth2/token` again. Cache tokens; PayPal says not to generate one per transaction. Sources: `/api/rest/authentication`, `/api/rest/reference/rate-limiting`.

Client token (only for Fastlane, JS SDK v6): same endpoint with `-d 'response_type=client_token'` and `-d 'domains[]=<URL1>,<URL2>'`; `expires_in` 32400 in the sample, while the SDK page says the client token expires after 15 minutes. Fastlane is not relevant for Malaysia. Source: `/sdk/js/set-up`.

## Common Request Headers

| Header | Use |
| --- | --- |
| `Authorization: Bearer <token>` | All API calls. |
| `Content-Type: application/json` | Calls with a body. |
| `PayPal-Request-Id` | Idempotency key on POST. Orders v2 docs: stored 6 hours by default (up to 72 hours via account manager). Refund example in `/api/rest/requests`: retry with same ID for up to 45 days. Subscriptions and Catalog: 72 hours. Payment Tokens: 3 hours. Payouts: 30 days. Use a UUID (38 single-byte character limit). Must be unique per request and per call type. Two simultaneous requests with the same ID: first processed, second might fail. |
| `Prefer` | `return=minimal` (default: `id`, `status`, HATEOAS links) or `return=representation` (full resource). |
| `PayPal-Client-Metadata-Id` | Risk correlation GUID (FraudNet). Optional. |
| `PayPal-Partner-Attribution-Id` | BN code; partners only. |
| `PayPal-Auth-Assertion` | JWT to act on behalf of a merchant; partners only. |
| `PayPal-Mock-Response` | Sandbox negative testing: `{"mock_application_codes":"INSTRUMENT_DECLINED"}`. |

Sources: `/api/rest/requests`, `/api/rest/reference/idempotency`, Orders v2 schema, `/api/rest/integration/orders-api/api-use-cases/advanced`, other API schemas.

## Amount Rules

- Amounts are JSON strings in a `money` object: `{"currency_code":"MYR","value":"10.00"}`. Not sen, not integers. Pattern `^((-?[0-9]+)|(-?([0-9]+)?[.][0-9]+))$`, max 32 chars.
- `HUF`, `JPY`, `TWD` do not support decimals (error if sent). MYR has no such footnote, so use 2 decimals.
- If `amount.breakdown` is sent, `amount.value` equals `item_total` + `tax_total` + `shipping` + `handling` + `insurance` - `shipping_discount` - `discount`, and must be positive. `item_total` is required when items carry `unit_amount` and must equal the sum of `unit_amount * quantity`. Compute these server-side.
- A subscription plan allows only one `currency_code`.

Sources: Orders v2 schema `money`, `/api/codes/currency`, `/subscriptions/integrate`.

## Default Flow For A Malaysian Website (Orders v2, intent CAPTURE)

1. Create a local pending payment (amount, `MYR`, your order ID).
2. Server: `POST /v2/checkout/orders` with `intent: CAPTURE`, one purchase unit with `amount` (`MYR`), `custom_id` and/or `invoice_id` = your order reference, `PayPal-Request-Id` header. Store the PayPal order `id`.
3. Browser: JS SDK button (`createOrder` returns the server order ID) or a server redirect to the `rel: approve` / `rel: payer-action` link. For redirect flows set `payment_source.paypal.experience_context.return_url` and `cancel_url`; without the JS SDK, missing `return_url` gives the payer an error page after approval (schema note on `order.links`).
4. On approval (`onApprove` or browser arriving at `return_url`), server: `POST /v2/checkout/orders/{id}/capture` with a `PayPal-Request-Id`.
5. Read `purchase_units[0].payments.captures[0].status`. Order `status: COMPLETED` alone is not enough: the schema says a completed order "can indicate a payment was authorized, an authorized payment was captured, or a payment was declined". Fulfil only when capture `status` is `COMPLETED` and amount, currency, `custom_id`/`invoice_id` match.
6. If capture `status` is `PENDING` (see `status_details.reason`), do not fulfil; wait for `PAYMENT.CAPTURE.COMPLETED` or `PAYMENT.CAPTURE.DENIED` webhook.
7. Webhooks: subscribe at least `CHECKOUT.ORDER.APPROVED`, `PAYMENT.CAPTURE.COMPLETED`, `PAYMENT.CAPTURE.PENDING`, `PAYMENT.CAPTURE.DENIED`, `PAYMENT.CAPTURE.REFUNDED`, `PAYMENT.CAPTURE.REVERSED`, `CHECKOUT.PAYMENT-APPROVAL.REVERSED`. Verify every webhook (see `paypal-webhooks.md`), then re-fetch `GET /v2/checkout/orders/{id}` or `GET /v2/payments/captures/{capture_id}` before settling.
8. Store the capture ID; refunds use it.

Sources: `/checkout/integrate`, `/api/rest/integration/orders-api`, `/payment-methods/webhooks`, Orders v2 schema.

Approval window: after create, the caller has 6 hours by default (account manager can change to 24/48/72) to redirect the payer, then 6 hours for approval and authorize/capture. Source: Orders v2 schema `order.links`.

Return URL query parameters: the docs read do not document which query parameters PayPal appends to `return_url` for the PayPal wallet flow (checked 2026-10-21). Put your own reference in `return_url` and look up the stored PayPal order ID. The cancel URL may carry `errorcode=<code>&token=<id>` for alternative payment methods (codes in `paypal-errors-testing.md`).

## Status Values

| Object | Values |
| --- | --- |
| Order `status` | `CREATED`, `SAVED`, `APPROVED`, `VOIDED`, `COMPLETED`, `PAYER_ACTION_REQUIRED` |
| Capture `status` | `COMPLETED`, `DECLINED`, `PARTIALLY_REFUNDED`, `PENDING`, `REFUNDED`, `FAILED` |
| Capture `status_details.reason` | `BUYER_COMPLAINT`, `CHARGEBACK`, `ECHECK`, `INTERNATIONAL_WITHDRAWAL`, `OTHER`, `PENDING_REVIEW`, `RECEIVING_PREFERENCE_MANDATES_MANUAL_ACTION`, `REFUNDED`, `TRANSACTION_APPROVED_AWAITING_FUNDING`, `UNILATERAL`, `VERIFICATION_REQUIRED`, `DECLINED_BY_RISK_FRAUD_FILTERS` |
| Authorization `status` | `CREATED`, `CAPTURED`, `DENIED`, `PARTIALLY_CAPTURED`, `VOIDED`, `PENDING` (reason `PENDING_REVIEW`, `DECLINED_BY_RISK_FRAUD_FILTERS`) |
| Refund `status` | `CANCELLED`, `FAILED`, `PENDING`, `COMPLETED` (reason `ECHECK`) |
| Subscription `status` | `APPROVAL_PENDING`, `APPROVED`, `ACTIVE`, `SUSPENDED`, `CANCELLED`, `EXPIRED` |
| Webhook verify `verification_status` | `SUCCESS`, `FAILURE` |

Sources: Orders v2, Payments v2, Subscriptions v1, Webhooks v1 schemas.

`RECEIVING_PREFERENCE_MANDATES_MANUAL_ACTION` matters for MYR: payments in a currency the account does not hold stay pending until accepted manually unless Payment Receiving Preferences are set (Account Settings > Payment preferences > Block payments). Source: `/api/codes/currency`.

## Webhook Verification (summary)

Self-verify: build `<paypal-transmission-id>|<paypal-transmission-time>|<webhook_id>|<crc32 of raw body as unsigned decimal>`, then verify the base64 `paypal-transmission-sig` with SHA256 RSA using the public key from the certificate at `paypal-cert-url`. Or postback to `POST /v1/notifications/verify-webhook-signature` and require `verification_status: SUCCESS`. Full detail and pseudo-code: `paypal-webhooks.md`. Delivery retries: up to 25 times over 3 days on non-2xx.

## Status Requery

- `GET /v2/checkout/orders/{id}` (optional `fields=payment_source`).
- `GET /v2/payments/captures/{capture_id}`, `GET /v2/payments/authorizations/{authorization_id}`, `GET /v2/payments/refunds/{refund_id}`.
- `GET /v1/reporting/transactions` for reconciliation (transactions can take up to 3 hours to appear; max 31-day range).
- PayPal does not publish a rate limit; it may return `429 RATE_LIMIT_REACHED` for abusive traffic and tells you not to poll but to use webhooks. Source: `/api/rest/reference/rate-limiting`.

## Refunds, Voids, Authorizations

- Refund: `POST /v2/payments/captures/{capture_id}/refund`; empty body = full refund, `amount` = partial. Optional `invoice_id`, `custom_id`, `note_to_payer`. Use `PayPal-Request-Id`. Guide: process within 180 days of capture; processed refunds cannot be cancelled. Use void, not refund, for an uncaptured authorization.
- Authorize flow (`intent: AUTHORIZE`): `POST /v2/checkout/orders/{id}/authorize`, then `POST /v2/payments/authorizations/{id}/capture`. Authorization valid 29 days, 3-day honor period; reauthorize (`/reauthorize`) from day 4 to 29 creates a new authorization ID; capture up to 115% or USD 75 over the authorized amount, whichever is less; partial capture of PayPal authorizations must be enabled on the account. Void with `/void` (not possible after full capture).

Sources: `/checkout/refund-payment`, `/payment-methods/auth-honor`, Payments v2 schema.

## Recurring

PayPal Subscriptions API (`/v1/billing/...`) with Catalog Products. See `paypal-subscriptions.md`. Card vaulting needs Expanded Checkout (not listed for Malaysia). PayPal-wallet vaulting via Payment Method Tokens v3 has no documented country list in the pages read; confirm eligibility with PayPal before building on it.

## Endpoint Catalogue

Every operation in the 17 current REST API schemas, plus OAuth. Paths are relative to the REST API base. Field tables for payment operations are in the topic files.

### OAuth 2.0

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v1/oauth2/token` | Get access token (`grant_type=client_credentials`) or client token (`response_type=client_token`) |

### Orders (v2, spec 2.36)

Reference: `https://developer.paypal.com/api/orders/v2/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v2/checkout/orders` | Create order |
| `GET` | `/v2/checkout/orders/{id}` | Show order details |
| `PATCH` | `/v2/checkout/orders/{id}` | Update order |
| `POST` | `/v2/checkout/orders/{id}/confirm-payment-source` | Confirm the Order |
| `POST` | `/v2/checkout/orders/{id}/authorize` | Authorize payment for order |
| `POST` | `/v2/checkout/orders/{id}/capture` | Capture payment for order |
| `POST` | `/v2/checkout/orders/{id}/track` | Add tracking information for an Order |
| `PATCH` | `/v2/checkout/orders/{id}/trackers/{tracker_id}` | Update or cancel tracking information for an order |
| `POST` | `/v2/checkout/orders/order-update-callback` | Receive updated order information via callback URL |
| `GET` | `/v2/checkout/callback/certs/{cert_id}` | Get public callback verification certificate |

### Payments (v2, spec 2.12)

Reference: `https://developer.paypal.com/api/payments/v2/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/v2/payments/authorizations/{authorization_id}` | Show details for authorized payment |
| `POST` | `/v2/payments/authorizations/{authorization_id}/capture` | Capture authorized payment |
| `POST` | `/v2/payments/authorizations/{authorization_id}/reauthorize` | Reauthorize authorized payment |
| `POST` | `/v2/payments/authorizations/{authorization_id}/void` | Void authorized payment |
| `GET` | `/v2/payments/captures/{capture_id}` | Show captured payment details |
| `POST` | `/v2/payments/captures/{capture_id}/refund` | Refund captured payment |
| `POST` | `/v2/payments/find-eligible-methods` | Find a list of eligible payment methods |
| `GET` | `/v2/payments/refunds/{refund_id}` | Show refund details |

### Webhooks Management (v1, spec 1.11)

Reference: `https://developer.paypal.com/api/webhooks/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v1/notifications/webhooks` | Create webhook |
| `GET` | `/v1/notifications/webhooks` | List webhooks |
| `GET` | `/v1/notifications/webhooks/{webhook_id}` | Show webhook details |
| `PATCH` | `/v1/notifications/webhooks/{webhook_id}` | Update webhook |
| `DELETE` | `/v1/notifications/webhooks/{webhook_id}` | Delete webhook |
| `GET` | `/v1/notifications/webhooks/{webhook_id}/event-types` | List event subscriptions for webhook |
| `POST` | `/v1/notifications/webhooks-lookup` | Create webhook lookup |
| `GET` | `/v1/notifications/webhooks-lookup` | List webhook lookups |
| `GET` | `/v1/notifications/webhooks-lookup/{webhook_lookup_id}` | Show webhook lookup details |
| `DELETE` | `/v1/notifications/webhooks-lookup/{webhook_lookup_id}` | Delete webhook lookup |
| `POST` | `/v1/notifications/verify-webhook-signature` | Verify webhook signature |
| `GET` | `/v1/notifications/webhooks-event-types` | List available events |
| `GET` | `/v1/notifications/webhooks-events` | List event notifications |
| `GET` | `/v1/notifications/webhooks-events/{event_id}` | Show event notification details |
| `POST` | `/v1/notifications/webhooks-events/{event_id}/resend` | Resend event notification |
| `POST` | `/v1/notifications/simulate-event` | Simulate webhook event |

### Subscriptions (v1, spec 1.11)

Reference: `https://developer.paypal.com/api/subscriptions/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v1/billing/plans` | Create plan |
| `GET` | `/v1/billing/plans` | List plans |
| `GET` | `/v1/billing/plans/{id}` | Show plan details |
| `PATCH` | `/v1/billing/plans/{id}` | Update plan |
| `POST` | `/v1/billing/plans/{id}/activate` | Activate plan |
| `POST` | `/v1/billing/plans/{id}/deactivate` | Deactivate plan |
| `POST` | `/v1/billing/plans/{id}/update-pricing-schemes` | Update pricing |
| `POST` | `/v1/billing/subscriptions` | Create subscription |
| `GET` | `/v1/billing/subscriptions` | List subscriptions |
| `GET` | `/v1/billing/subscriptions/{id}` | Show subscription details |
| `PATCH` | `/v1/billing/subscriptions/{id}` | Update subscription |
| `POST` | `/v1/billing/subscriptions/{id}/revise` | Revise plan or quantity of subscription |
| `POST` | `/v1/billing/subscriptions/{id}/suspend` | Suspend subscription |
| `POST` | `/v1/billing/subscriptions/{id}/cancel` | Cancel subscription |
| `POST` | `/v1/billing/subscriptions/{id}/activate` | Activate subscription |
| `POST` | `/v1/billing/subscriptions/{id}/capture` | Capture authorized payment on subscription |
| `GET` | `/v1/billing/subscriptions/{id}/transactions` | List transactions for subscription |

### Catalog Products (v1, spec 1.0)

Reference: `https://developer.paypal.com/api/catalog-products/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v1/catalogs/products` | Create product |
| `GET` | `/v1/catalogs/products` | List products |
| `GET` | `/v1/catalogs/products/{product_id}` | Show product details |
| `PATCH` | `/v1/catalogs/products/{product_id}` | Update product |

### Payment Method Tokens (v3, spec 3.5)

Reference: `https://developer.paypal.com/api/payment-tokens/v3/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v3/vault/payment-tokens` | Create payment token for a given payment source |
| `GET` | `/v3/vault/payment-tokens` | List all payment tokens |
| `GET` | `/v3/vault/payment-tokens/{id}` | Retrieve a payment token |
| `DELETE` | `/v3/vault/payment-tokens/{id}` | Delete payment token |
| `POST` | `/v3/vault/setup-tokens` | Create a setup token |
| `GET` | `/v3/vault/setup-tokens/{id}` | Retrieve a setup token |

### Transaction Search (v1, spec 1.10)

Reference: `https://developer.paypal.com/api/transaction-search/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/v1/reporting/transactions` | List transactions |
| `GET` | `/v1/reporting/balances` | List all balances |
| `GET` | `/v1/reporting/get-balance-net-summary` | List all balance and net activity summary |
| `GET` | `/v1/reporting/get-daily-summary` | List all daily summary |

### Payouts (v1, spec 1.9)

Reference: `https://developer.paypal.com/api/payments.payouts-batch/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v1/payments/payouts` | Create batch payout |
| `GET` | `/v1/payments/payouts/{id}` | Show payout batch details |
| `GET` | `/v1/payments/payouts-item/{payout_item_id}` | Show payout item details |
| `POST` | `/v1/payments/payouts-item/{payout_item_id}/cancel` | Cancel unclaimed payout item |

### Disputes (v1, spec 1.12)

Reference: `https://developer.paypal.com/api/customer-disputes/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/v1/customer/disputes` | List disputes |
| `GET` | `/v1/customer/disputes/{id}` | Show dispute details |
| `PATCH` | `/v1/customer/disputes/{id}` | Partially update dispute |
| `POST` | `/v1/customer/disputes/{id}/provide-evidence` | Provide evidence |
| `POST` | `/v1/customer/disputes/{id}/appeal` | Appeal dispute |
| `POST` | `/v1/customer/disputes/{id}/accept-claim` | Accept claim |
| `POST` | `/v1/customer/disputes/{id}/adjudicate` | Settle dispute |
| `POST` | `/v1/customer/disputes/{id}/require-evidence` | Update dispute status |
| `POST` | `/v1/customer/disputes/{id}/escalate` | Escalate dispute to claim |
| `POST` | `/v1/customer/disputes/{id}/send-message` | Send message about dispute to other party |
| `POST` | `/v1/customer/disputes/{id}/make-offer` | Make offer to resolve dispute |
| `POST` | `/v1/customer/disputes/{id}/accept-offer` | Accept offer to resolve dispute |
| `POST` | `/v1/customer/disputes/{id}/deny-offer` | Deny offer to resolve dispute |
| `POST` | `/v1/customer/disputes/{id}/acknowledge-return-item` | Acknowledge returned item |
| `POST` | `/v1/customer/disputes/{id}/provide-supporting-info` | Provide supporting information for dispute |

### Invoices (v2, spec 2.12.0)

Reference: `https://developer.paypal.com/api/invoicing/v2/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/v2/invoicing/invoices` | List invoices |
| `POST` | `/v2/invoicing/invoices` | Create draft invoice |
| `POST` | `/v2/invoicing/invoices/{invoice_id}/send` | Send invoice |
| `POST` | `/v2/invoicing/invoices/{invoice_id}/remind` | Send invoice reminder |
| `POST` | `/v2/invoicing/invoices/{invoice_id}/cancel` | Cancel sent invoice |
| `POST` | `/v2/invoicing/invoices/{invoice_id}/payments` | Record payment for invoice |
| `DELETE` | `/v2/invoicing/invoices/{invoice_id}/payments/{transaction_id}` | Delete external payment |
| `POST` | `/v2/invoicing/invoices/{invoice_id}/refunds` | Record refund for invoice |
| `DELETE` | `/v2/invoicing/invoices/{invoice_id}/refunds/{transaction_id}` | Delete external refund |
| `POST` | `/v2/invoicing/invoices/{invoice_id}/generate-qr-code` | Generate QR code |
| `POST` | `/v2/invoicing/generate-next-invoice-number` | Generate invoice number |
| `GET` | `/v2/invoicing/invoices/{invoice_id}` | Show invoice details |
| `PUT` | `/v2/invoicing/invoices/{invoice_id}` | Fully update invoice |
| `DELETE` | `/v2/invoicing/invoices/{invoice_id}` | Delete invoice |
| `POST` | `/v2/invoicing/search-invoices` | Search for invoices |
| `GET` | `/v2/invoicing/templates` | List templates |
| `POST` | `/v2/invoicing/templates` | Create template |
| `GET` | `/v2/invoicing/templates/{template_id}` | Show template details |
| `PUT` | `/v2/invoicing/templates/{template_id}` | Fully update template |
| `DELETE` | `/v2/invoicing/templates/{template_id}` | Delete template |
| `GET` | `/v2/invoicing/invoices/{id}/conditional-rules/{conditional_rule_id}` | Retrieve the conditional rule details for an Invoice |
| `PUT` | `/v2/invoicing/invoices/{id}/conditional-rules/{conditional_rule_id}` | Fully update conditional rule |
| `DELETE` | `/v2/invoicing/invoices/{id}/conditional-rules/{conditional_rule_id}` | Delete the conditional rule details from the Invoice |
| `GET` | `/v2/invoicing/invoices/{id}/conditional-rules` | Retrieves all conditional rules for a given Invoice |
| `POST` | `/v2/invoicing/invoices/{id}/conditional-rules` | Create conditional rules |
| `POST` | `/v2/invoicing/recurring-invoices` | Create a recurring invoice series |
| `GET` | `/v2/invoicing/recurring-invoices/{id}` | Get recurring invoice series details |
| `PUT` | `/v2/invoicing/recurring-invoices/{id}` | Update recurring invoice series details |
| `DELETE` | `/v2/invoicing/recurring-invoices/{id}` | Delete recurring invoice series |
| `POST` | `/v2/invoicing/recurring-invoices/{id}/activate` | Activate recurring invoice series |
| `POST` | `/v2/invoicing/recurring-invoices/{id}/cancel` | Cancel an active recurring invoice series |
| `POST` | `/v2/invoicing/setup-reminders` | Setup auto reminder configuration |
| `GET` | `/v2/invoicing/reminders` | Get all invoice auto reminder configuration |
| `GET` | `/v2/invoicing/reminders/{id}` | Retrieve an invoice auto reminder configuration |
| `PUT` | `/v2/invoicing/reminders/{id}` | Update invoice auto reminder configuration |
| `POST` | `/v2/invoicing/reminders/{id}/suspend` | Suspend invoice auto reminder configuration |
| `POST` | `/v2/invoicing/reminders/{id}/resume` | Resume invoice auto reminder configuration |
| `POST` | `/v2/invoicing/invoices/{id}/cancel-reminders` | Cancel auto reminders for an invoice |
| `POST` | `/v2/invoicing/search-recurring-invoices` | Search recurring invoices series |

### Shipment Tracking (v1, spec 1.10)

Reference: `https://developer.paypal.com/api/tracking/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v1/shipping/trackers-batch` | Add tracking information for multiple PayPal transactions |
| `POST` | `/v1/shipping/trackers` | Add tracking information for PayPal transaction |
| `GET` | `/v1/shipping/trackers` | List tracking information |
| `PUT` | `/v1/shipping/trackers/{id}` | Update or cancel tracking information for PayPal transaction |
| `GET` | `/v1/shipping/trackers/{id}` | Show tracking information |

### Payment Links and Buttons (v1, spec 1.8)

Reference: `https://developer.paypal.com/api/payment-links-buttons/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v1/checkout/payment-resources` | Creates a payment resource for a single-item purchase |
| `GET` | `/v1/checkout/payment-resources` | List payment resources |
| `GET` | `/v1/checkout/payment-resources/{id}` | Retrieve a payment resource |
| `PUT` | `/v1/checkout/payment-resources/{id}` | Replace a payment resource |
| `DELETE` | `/v1/checkout/payment-resources/{id}` | Delete a payment resource |

### Payment Experience Web Profiles (v1, spec 1.3)

Reference: `https://developer.paypal.com/api/payment-experience/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v1/payment-experience/web-profiles` | Create web experience profile |
| `GET` | `/v1/payment-experience/web-profiles` | List web experience profiles |
| `PUT` | `/v1/payment-experience/web-profiles/{id}` | Update web experience profile |
| `PATCH` | `/v1/payment-experience/web-profiles/{id}` | Partially update web experience profile |
| `GET` | `/v1/payment-experience/web-profiles/{id}` | Show web experience profile details by ID |
| `DELETE` | `/v1/payment-experience/web-profiles/{id}` | Delete web experience profile |

### Currency Exchange (v2, spec 2.17)

Reference: `https://developer.paypal.com/api/pricing/v2/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v2/pricing/quote-exchange-rates` | Create Exchange Rate Quote |

### Identity (v1, spec 1.0)

Reference: `https://developer.paypal.com/api/identity/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/v1/identity/openidconnect/userinfo` | Show user profile details |

### Partner Referrals (v2, spec 2.5)

Reference: `https://developer.paypal.com/api/partner-referrals/v2/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v2/customer/partner-referrals` | Create partner referral |
| `GET` | `/v2/customer/partner-referrals/{partner_referral_id}` | Show referral data |

### Usage Based Billing API (v1, spec 1.0.0)

Reference: `https://developer.paypal.com/api/usage-based-billing/v1/` (schema: `.../schema.json`)

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/billing-entities` | List all billing entities |
| `POST` | `/billing-entities` | Create a billing entity |
| `GET` | `/billing-entities/{billing_entity_code}` | Get a billing entity by code |
| `PUT` | `/billing-entities/{billing_entity_code}` | Update a billing entity by code |
| `POST` | `/taxes` | Create a tax |
| `GET` | `/taxes` | List all taxes |
| `GET` | `/taxes/{tax_code}` | Retrieve a Tax |
| `PUT` | `/taxes/{tax_code}` | Update a tax |
| `DELETE` | `/taxes/{tax_code}` | Delete a tax |
| `POST` | `/activate` | Activate Usage Based Billing |
| `GET` | `/invoices` | Get all invoices |
| `GET` | `/invoices/{invoice_id}` | Get a single invoice by ID |
| `POST` | `/invoices/{invoice_id}/retry_payment` | Retry payment for a specific invoice |
| `GET` | `/metrics` | Get all metrics |
| `POST` | `/metrics` | Create a new metric |
| `GET` | `/metrics/{metric_code}` | Get a metric by code |
| `PUT` | `/metrics/{metric_code}` | Update a metric by Code |
| `DELETE` | `/metrics/{metric_code}` | Delete a metric by Code |
| `GET` | `/plans` | Get all plans |
| `POST` | `/plans` | Create a new plan |
| `GET` | `/plans/{plan_code}` | Get a plan by Code |
| `PUT` | `/plans/{plan_code}` | Update a plan by Code |
| `DELETE` | `/plans/{plan_code}` | Delete a plan by Code |
| `GET` | `/customers` | Get all customers |
| `POST` | `/customers` | Create a new customer |
| `GET` | `/customers/{external_customer_id}` | Get a customer by ID |
| `PUT` | `/customers/{external_customer_id}` | Update a customer by ID |
| `DELETE` | `/customers/{external_customer_id}` | Delete a customer by ID |
| `GET` | `/customers/{external_customer_id}/analytics/overdue-balance` | Retrieve customer analytics for overdue balance |
| `GET` | `/customers/{external_customer_id}/analytics/gross-revenue` | Retrieve customer gross revenue analytics |
| `GET` | `/customers/{external_customer_id}/current_usage` | Get current usage for a customer |
| `GET` | `/customers/{external_customer_id}/past_usage` | Get past usage for a customer |
| `GET` | `/customers/{external_customer_id}/portal_url` | Get an embeddable URL to the customer portal |
| `GET` | `/subscriptions` | Get all subscriptions |
| `POST` | `/subscriptions` | Create a new subscription |
| `GET` | `/subscriptions/{external_subscription_id}` | Get a subscription by ID |
| `PUT` | `/subscriptions/{external_subscription_id}` | Update a subscription |
| `GET` | `/subscriptions/{external_subscription_id}/lifetime-usage` | Retrieve lifetime usage for a specific subscription |
| `POST` | `/subscriptions/{external_subscription_id}/cancel` | Cancel a subscription |
| `GET` | `/events` | Query usage events |
| `POST` | `/events` | Create a new usage event |
| `POST` | `/events/batch` | Create multiple usage events in batch |
| `GET` | `/wallets` | Get all wallets |
| `POST` | `/wallets` | Create a new wallet |
| `GET` | `/wallets/{wallet_id}` | Retrieve a wallet by ID |
| `PUT` | `/wallets/{wallet_id}` | Update a wallet |
| `DELETE` | `/wallets/{wallet_id}` | Terminate wallet |
| `POST` | `/wallets/{wallet_id}/wallet-transactions` | Create Wallet transaction |
| `GET` | `/wallets/{wallet_id}/wallet-transactions` | Get wallet transactions |
| `GET` | `/wallet-transactions/{wallet_transaction_id}` | Get Wallet transaction by id |
| `GET` | `/credit-notes` | Get all credit notes |
| `POST` | `/credit-notes` | Create a new credit note |
| `GET` | `/credit-notes/{credit_note_id}` | Get a single credit note by ID |
| `POST` | `/credit-notes/{credit_note_id}/void` | Void a credit note |
| `GET` | `/subscriptions/{external_subscription_id}/alerts` | Retrieve subscription alerts |
| `POST` | `/subscriptions/{external_subscription_id}/alerts` | Create subscription alert |
| `GET` | `/subscriptions/{external_subscription_id}/alerts/{alert_code}` | Retrieve specific subscription alert |
| `PUT` | `/subscriptions/{external_subscription_id}/alerts/{alert_code}` | Modify subscription alert |
| `DELETE` | `/subscriptions/{external_subscription_id}/alerts/{alert_code}` | Remove subscription alert |
| `GET` | `/features` | List all features |
| `POST` | `/features` | Create a feature |
| `GET` | `/features/{feature_code}` | Retrieve a feature |
| `PUT` | `/features/{feature_code}` | Update a feature |
| `DELETE` | `/features/{feature_code}` | Delete a feature |
| `DELETE` | `/features/{feature_code}/privileges/{privilege_code}` | Delete a feature privilege |
| `GET` | `/plans/{plan_code}/entitlements` | List all entitlements |
| `POST` | `/plans/{plan_code}/entitlements` | Create an entitlement |
| `PATCH` | `/plans/{plan_code}/entitlements` | Partial update of entitlements |
| `GET` | `/plans/{plan_code}/entitlements/{feature_code}` | Retrieve an entitlement |
| `DELETE` | `/plans/{plan_code}/entitlements/{feature_code}` | Delete an entitlement |
| `DELETE` | `/plans/{plan_code}/entitlements/{feature_code}/privileges/{privilege_code}` | Remove a privilege from an entitlement |
| `GET` | `/subscriptions/{external_subscription_id}/entitlements` | List all subscription entitlements |
| `PATCH` | `/subscriptions/{external_subscription_id}/entitlements` | Update subscription entitlements |
| `DELETE` | `/subscriptions/{external_subscription_id}/entitlements/{feature_code}` | Remove an entitlement from a subscription |
| `DELETE` | `/subscriptions/{external_subscription_id}/entitlements/{feature_code}/privileges/{privilege_code}` | Remove a privilege from a subscription entitlement override |

Usage Based Billing paths are published in its schema without a version prefix (as shown). It is a separate metered-billing product; confirm availability for a Malaysian account with PayPal before use.

Global-only or partner-only areas in this catalogue (Partner Referrals, Payment Experience web profiles, Identity, Pricing quotes, Usage Based Billing, Shipment Tracking, Invoicing) are listed for completeness; field tables are in the official schemas, not copied here. Deprecated APIs (Payments v1, Orders v1, Billing Agreements v1, Invoicing v1, NVP/SOAP) are out of scope: see `/api/rest/deprecated-resources`.
