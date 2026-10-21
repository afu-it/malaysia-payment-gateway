# PayPal Source Map

All paths are on `https://developer.paypal.com` unless a full URL is given. Every page has a Markdown twin at `<path>.md` (for example `https://developer.paypal.com/api/rest/authentication.md`), and every REST API has an OpenAPI schema at `/api/<api>/<version>/schema.json`. Checked 2026-10-21.

## Coverage

- Index used: `/llms.txt` (sections: Get Started 5, Payments 278, Revenue 119, Developer Resources 311, Platforms 209, Braintree 904, Other 2972 pages) and `/sitemap.xml` (4799 URLs).
- In scope (Business, non-platform, Malaysia-relevant): 321 guide pages downloaded as Markdown and searched in full for Malaysia, MYR, eligibility and integration facts; about 45 core pages read end to end (REST basics, auth, requests, responses, idempotency, rate limits, production, webhooks set, checkout quickstart, JS SDK v6 set-up and reference, JS SDK v5 configuration and reference, auth and honor, refunds, uncaptured payments, eligibility, currency and country codes, sandbox and negative testing, subscriptions set, payouts Malaysia pages, error pages).
- API reference: 17 current OpenAPI schemas parsed completely (Orders v2, Payments v2, Webhooks v1, Subscriptions v1, Catalog Products v1, Payment Method Tokens v3, Transaction Search v1, Payouts v1, Disputes v1, Invoicing v2, Shipment Tracking v1, Payment Links and Buttons v1, Payment Experience v1, Pricing v2, Identity v1, Partner Referrals v2, Usage Based Billing v1). These cover the 1,953 per-operation and per-definition reference pages in the sitemap (including the 107 Payments v2 error pages). Orders v2 error page (244 issues) and 107 Payments v2 error pages parsed.
- Left out on purpose: Braintree GraphQL (904 pages, separate product), Platforms/partner pages (209, marketplace onboarding), deprecated APIs (Payments v1, Orders v1, Billing Agreements, Invoicing v1, NVP/SOAP; 795 pages), country-specific methods unavailable to Malaysian buyers or merchants (iDEAL, Bancontact, BLIK, eps, MyBank, Przelewy24, Trustly, Swish, Multibanco, MB WAY, OXXO, Boleto, Satispay, BANCOMAT Pay, Ratepay/PUI, Venmo, Pay Later per-country plugin pages, Fastlane), mobile SDK pages (iOS/Android), Log in with PayPal, AI/agentic commerce pages, reports file specifications (only the overview used).

## REST Basics

- `/api/rest` (get started, client ID/secret, token, sandbox accounts)
- `/api/rest/authentication`
- `/api/rest/requests` (hosts, headers, query params, `PayPal-Request-Id`)
- `/api/rest/responses` (HTTP codes, HATEOAS)
- `/api/rest/reference/idempotency`
- `/api/rest/reference/rate-limiting`
- `/api/rest/production` (go live, app review, checklist)
- `/api/rest/reference/policies-and-guidelines` (Malaysia personal payments rule)
- `/api/rest/current-resources`, `/api/rest/deprecated-resources`

Use for: environments, auth, headers, idempotency, rate limits, go-live, policy limits.

## Orders And Payments

- `/api/orders/v2/schema.json`, `/api/payments/v2/schema.json`
- `/api/rest/integration/orders-api`, `/api/rest/integration/orders-api/api-use-cases/standard`, `/api/rest/integration/orders-api/api-use-cases/advanced`
- `/whats-an-order`
- `/checkout`, `/checkout/integrate`, `/checkout/refund-payment`, `/checkout/delay-capture`, `/checkout/void-authorized-payment`, `/checkout/extend-authorization`
- `/payment-methods/auth-honor`, `/v5/checkout/uncaptured-payments`, `/v5/checkout/handle-funding-failure`

Use for: create/capture/authorize/refund fields, statuses, approval links, PATCH paths, honor period, refund window.

## Webhooks

- `/api/rest/webhooks`, `/api/rest/webhooks/rest` (signature algorithm, postback), `/api/rest/webhooks/simulator`, `/api/rest/webhooks/events-dashboard`, `/api/rest/webhooks/event-names`
- `/api/webhooks/overview`, `/api/webhooks/v1/schema.json`
- `/payment-methods/webhooks`, `/subscriptions/webhooks`

Use for: verification string and headers, retries, event names, Webhooks Management API.

## JavaScript SDK

- `/sdk/js/set-up`, `/sdk/js/reference` (v6)
- `/v5-v6` (migration)
- `/sdk/js/v5/configuration`, `/sdk/js/v5/reference`
- `/guides/low-code-buy-button`, `/payment-links-buttons/overview`

Use for: script URLs, init, eligibility, sessions, callbacks, v5 query params, Malaysia locales.

## Malaysia Eligibility And Currency

- `/api/codes/currency` (MYR footnote, payment receiving preferences)
- `/v5/checkout/fx/currency-codes`, `/v5/checkout/fx/overview`
- `/api/codes/country-region` (`MY`), `/api/codes/locale`
- `/expanded/eligibility` (37 countries, Malaysia absent; Pay Later/Credit/Venmo countries)
- `/v5/save-payment-methods/overview` (card vault needs Expanded Checkout)
- `/grabpay` (Singapore only)
- `/payouts/supported-features`, `/payouts/fees`, `/payouts/faqs`, `/payouts/currency-conversion`

Use for: what a Malaysian merchant can and cannot use.

## Subscriptions

- `/subscriptions/about`, `/subscriptions/integrate`, `/subscriptions/payment-failure-retry`, `/subscriptions/billing-cycle`, `/subscriptions/trial-period`, `/subscriptions/change-price`, `/subscriptions/change-quantity`, `/subscriptions/pause-or-resume`
- `/api/subscriptions/v1/schema.json`, `/api/catalog-products/v1/schema.json`

Use for: product, plan, subscription fields, failure retry, recurring webhooks.

## Errors And Testing

- `/api/rest/integration/orders-api/errors`, `/api/orders/v2/error-messages`, `/api/payments/v2/errors/<code>`, `/api/errors/overview`, `/api/errors/unprocessable-entity`
- `/payment-methods/error-codes` (cancel URL codes)
- `/sandbox-testing/overview`, `/sandbox-testing/accounts`, `/sandbox-testing/card-testing`
- `/negative-testing/overview`, `/negative-testing/request-headers`, `/negative-testing/values`

Use for: error handling, test cards, sandbox accounts, mock responses.

## Reporting, Payouts, Disputes

- `/api/transaction-search/v1/schema.json`, `/reports/overview`, `/reports/reference/t-codes`
- `/api/payments.payouts-batch/v1/schema.json`, `/payouts/overview`, `/api/payouts`, `/payouts/use-payouts/set-up`
- `/api/customer-disputes/v1/schema.json`, `/disputes/overview`, `/disputes/reasons-evidence`, `/api/disputes/test-values`
- `/secure-file-transfer` (SFTP report subnets)

Use for: reconciliation, sending money, chargebacks.

## Account And Pricing (paypal.com)

- `https://www.paypal.com/my/business/open-business-account` (documents needed)
- `https://www.paypal.com/my/business/paypal-business-fees` (pricing, link only)
- `https://www.paypal.com/webapps/mpp/country-worldwide`
- `https://www.paypal.com/my/legalhub/paypal/useragreement-full`
- `https://www.paypal.com/webapps/mpp/ua/xdeveloper-full` (Developer Agreement)

Use for: registration, fees, legal terms.

## Official Code

- `https://github.com/paypal-examples/v6-web-sdk-sample-integration`
- Postman: `/api/rest/postman`

## Not Documented (checked 2026-10-21)

- Query parameters appended to `return_url` for the PayPal wallet flow.
- Subscription approval link `rel` value in the pages read.
- `PAYMENT.SALE.COMPLETED` resource fields for subscriptions.
- Any IP list for webhook senders.
- PayPal-wallet vault (Payment Method Tokens) eligibility by country.
- FPX, DuitNow, Malaysian e-wallets (none exist in PayPal docs).
