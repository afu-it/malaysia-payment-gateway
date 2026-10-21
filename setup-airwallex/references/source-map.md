# Airwallex Source Map

Every official page used, grouped by topic. All URLs are on `https://www.airwallex.com/docs/`; append `.md` to any page URL to get the Markdown version Airwallex serves for agents. Checked 2026-10-21.

## Indexes

- `https://www.airwallex.com/docs/llms.txt` (docs index for agents, 12 product sections)
- `https://www.airwallex.com/docs/sitemap.xml` (1,476 docs URLs)
- `https://www.airwallex.com/llms.txt` (marketing site index; no API pages)
- No `llms-full.txt` (404) and no public OpenAPI file was found; the API reference pages themselves are served as Markdown.

Use for: enumerating pages, finding a page's `.md` form.

## Coverage

| Area | Enumerated | Treatment |
| --- | --- | --- |
| API reference resource pages (`/docs/api/...`) | 115 | All 115 fetched and parsed; every listed operation (470) is in the Endpoint catalogue in `airwallex-api.md` |
| API endpoint detail pages for payment acceptance (payments, checkout, webhooks, authentication, finance, payment simulation) | 105 | All 105 fetched; request, query and response fields extracted into the topic files |
| Payments product docs (`/docs/payments/...`) | 481 | All 481 downloaded. About 60 core pages read in full (get started, Payment Intents, HPP, Checkout Session, Drop-in, Native API, 3DS, holds, refunds, cancellations, statuses, webhooks, currencies, testing, errors, settlement, recurring, onboarding requirements, cards, Apple Pay, every Malaysian method page and sub-page, WooCommerce). The rest scanned by heading/grep and left out as below |
| Developer tools (`/docs/developer-tools/...`) | 90 | All downloaded. Webhooks (overview, listen, online payments events, re-trigger), API keys, scopes, rate limits, IP whitelisting, sandbox (overview, settings, payments simulation), server SDK, AI/MCP, CLI read. Non-payment webhook event pages, partner OAuth, open banking and non-payment simulations left out |
| Airwallex.js reference (`/docs/js/...`) | 27 | Read: getting started, hosted payment page, drop-in, card. Payouts, onboarding, risk, SCA, billing elements left out |
| Payouts, connected accounts, BaaS, global treasury, issuing, payments for platforms, billing, transactional FX, spend, accounts, partner API | 762 | Out of scope (not payment acceptance for a single Malaysian merchant). Their API operations are still listed in the Endpoint catalogue |

Left out of the distilled guidance, and why:

- Payment methods for other markets (about 200 pages: EU, US, LATAM, Korea, Japan, China, Indonesia, Thailand, Philippines, and so on): not usable for MYR shoppers.
- POS / terminals and SoftPOS: in-person, not online.
- Plugin pages for platforms other than WooCommerce (Shopify, Magento, BigCommerce, PrestaShop, SFCC, SHOPLINE, partner orchestrators): no-code setups; linked from `account-setup.md`.
- Legacy embedded elements, Korean local card elements, mobile SDK step pages: covered by the newer Airwallex.js pages; mobile SDK is linked only.
- Billing (subscriptions, invoices), Payments for Platforms, funds splits, customs declarations, MCP/ACC currency conversion, network tokenization, card account updater: advanced or non-Malaysian features; endpoints are in the catalogue.

## API basics

- `https://www.airwallex.com/docs/api/introduction`
- `https://www.airwallex.com/docs/api/errors`
- `https://www.airwallex.com/docs/api/versioning`
- `https://www.airwallex.com/docs/api/data_types`
- `https://www.airwallex.com/docs/api/changelog`
- `https://www.airwallex.com/docs/api/authentication/api_access_token/login`

Use for: base URLs, auth, versioning header, error shape.

## Developer setup

- `https://www.airwallex.com/docs/developer-tools/api/manage-api-keys`
- `https://www.airwallex.com/docs/developer-tools/api/api-key-best-practices`
- `https://www.airwallex.com/docs/developer-tools/api/api-key-scopes`
- `https://www.airwallex.com/docs/developer-tools/api/rate-limits`
- `https://www.airwallex.com/docs/developer-tools/api/ip-whitelisting`
- `https://www.airwallex.com/docs/developer-tools/api/quickstart-with-postman`
- `https://www.airwallex.com/docs/developer-tools/sdks/server-side-sdks-(beta)`
- `https://www.airwallex.com/docs/developer-tools/sdks/airwallex.js`
- `https://www.airwallex.com/docs/developer-tools/sdks/airwallex.js/error-codes`
- `https://www.airwallex.com/docs/developer-tools/security/securing-connections-to-airwallex`
- `https://www.airwallex.com/docs/developer-tools/ai/developer-connector`
- `https://www.airwallex.com/docs/developer-tools/ai/agentos`
- `https://www.airwallex.com/docs/developer-tools/cli`

Use for: keys, scopes, `x-login-as`, rate limits, IP whitelist, Node SDK, MCP endpoints.

## Getting started and Payment Intents

- `https://www.airwallex.com/docs/payments/get-started/get-started-with-payments`
- `https://www.airwallex.com/docs/payments/get-started/quickstart`
- `https://www.airwallex.com/docs/payments/get-started/using-payments-intent-api`
- `https://www.airwallex.com/docs/payments/get-started/payment-method-activation`
- `https://www.airwallex.com/docs/payments/reference/payment-statuses`
- `https://www.airwallex.com/docs/payments/reference/supported-currencies`
- `https://www.airwallex.com/docs/api/payments/payment_intents` (and `/create`, `/retrieve`, `/list`, `/update`, `/confirm`, `/confirm_continue`, `/capture`, `/cancel`, `/increment_authorization`)
- `https://www.airwallex.com/docs/api/payments/payment_attempts` (and `/retrieve`, `/list`)

Use for: create/confirm flow, `request_id`, `client_secret` lifetime, amounts in major units, statuses.

## Checkout surfaces

- `https://www.airwallex.com/docs/payments/integration-options/web-checkout`
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/hosted-payment-page`
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/checkout-session/quickstart`
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/drop-in-element` and `/guest-user-checkout`
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/embedded-elements`
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/native-api` and `/guest-user-checkout`
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/native-api/3d-secure-authentication` (and the two version-specific pages)
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/customize-appearance`
- `https://www.airwallex.com/docs/payments/integration-options/invoices-and-payment-links/payment-links`
- `https://www.airwallex.com/docs/js/getting-started`, `https://www.airwallex.com/docs/js/payments/hosted-payment-page`, `https://www.airwallex.com/docs/js/payments/dropin`, `https://www.airwallex.com/docs/js/payments/card`
- `https://www.airwallex.com/docs/api/checkout/checkout_sessions` (and `/create`, `/retrieve`, `/list`, `/update`, `/cancel`)
- `https://www.airwallex.com/docs/api/payments/payment_links` (and sub-pages)

Use for: HPP props, Drop-in options and events, Checkout Session fields, PCI levels, 3DS handling.

## Malaysian payment methods

- `https://www.airwallex.com/docs/payments/payment-methods`
- `https://www.airwallex.com/docs/payments/payment-methods/apac/fpx` and `/desktopmobile-website-browser`
- `https://www.airwallex.com/docs/payments/payment-methods/apac/duitnow` and `/desktopmobile-website-browser`
- `https://www.airwallex.com/docs/payments/payment-methods/apac/touch-'n-go` and its QR, mobile web, mobile app, recurring pages
- `https://www.airwallex.com/docs/payments/payment-methods/apac/boost` and sub-page
- `https://www.airwallex.com/docs/payments/payment-methods/apac/grabpay` and sub-page
- `https://www.airwallex.com/docs/payments/payment-methods/apac/shopeepay-beta` and sub-pages
- `https://www.airwallex.com/docs/payments/payment-methods/apac/spaylater-beta`
- `https://www.airwallex.com/docs/payments/payment-methods/apac/atome` and sub-page
- `https://www.airwallex.com/docs/payments/payment-methods/apac/7-eleven-malaysia` and sub-page
- `https://www.airwallex.com/docs/payments/payment-methods/global/cards`
- `https://www.airwallex.com/docs/payments/payment-methods/global/apple-pay` and `/enable-apple-pay`
- `https://www.airwallex.com/docs/payments/payment-methods/global/google-paytm`
- `https://www.airwallex.com/docs/payments/payment-methods/payment-method-onboarding-requirement`
- `https://www.airwallex.com/docs/api/payments/config` (payment method types, banks, Apple Pay domains, reserve plan, convertible currencies)
- `https://www.airwallex.com/docs/api/payments/reference_data` (BIN lookup)

Use for: limits, timeouts, settlement, refund windows, confirm payloads, FPX bank codes.

## Webhooks

- `https://www.airwallex.com/docs/developer-tools/webhooks/webhooks-overview`
- `https://www.airwallex.com/docs/developer-tools/webhooks/listen-for-webhook-events`
- `https://www.airwallex.com/docs/developer-tools/webhooks/listen-for-webhook-events/online-payments`
- `https://www.airwallex.com/docs/developer-tools/webhooks/view-and-re-trigger-webhook-events`
- `https://www.airwallex.com/docs/payments/reference/payments-webhooks`
- `https://www.airwallex.com/docs/api/webhooks/webhooks` (and `/create`, `/retrieve`, `/list`, `/update`, `/delete`)

Use for: `x-timestamp` + `x-signature` HMAC-SHA256 verification, retries, event names, source IPs.

## Refunds, cancellations, disputes

- `https://www.airwallex.com/docs/payments/payment-operations/manage-payments/refunds`
- `https://www.airwallex.com/docs/payments/payment-operations/manage-payments/cancellations`
- `https://www.airwallex.com/docs/payments/payment-operations/manage-payments/place-a-hold-on-a-payment-method`
- `https://www.airwallex.com/docs/payments/payment-operations/disputes` and `/handle-disputes-using-our-apis`
- `https://www.airwallex.com/docs/api/payments/refunds` (and sub-pages)
- `https://www.airwallex.com/docs/api/payments/payment_disputes` (and sub-pages)

Use for: refund rules and statuses, cancel vs refund, hold windows, dispute stages.

## Recurring and saved methods

- `https://www.airwallex.com/docs/payments/about-airwallex-payments/recurring-payments`
- `https://www.airwallex.com/docs/payments/about-airwallex-payments/cof-payments`
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/save-and-reuse-payment-details`
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/native-api/registered-user-checkout`
- `https://www.airwallex.com/docs/api/payments/customers`, `.../payment_consents`, `.../payment_methods` (and sub-pages)

Use for: Customer, Payment Consent, CIT/MIT, Touch 'n Go recurring.

## Settlement and reporting

- `https://www.airwallex.com/docs/payments/about-airwallex-payments/settle-into-your-airwallex-balance`
- `https://www.airwallex.com/docs/payments/about-airwallex-payments/reserve-plan`
- `https://www.airwallex.com/docs/payments/about-airwallex-payments/fees-and-pricing-models`
- `https://www.airwallex.com/docs/payments/payment-operations/reporting` and `/settlement-report`
- `https://www.airwallex.com/docs/api/finance/settlements`, `.../finance/financial_transactions`, `.../finance/financial_reports`, `.../payments/settlement_records`

Use for: T+N, reserves, net vs gross, settlement and reconciliation APIs.

## Sandbox, testing, errors

- `https://www.airwallex.com/docs/developer-tools/sandbox-environment`
- `https://www.airwallex.com/docs/developer-tools/sandbox-environment/sandbox-settings`
- `https://www.airwallex.com/docs/developer-tools/sandbox-environment/payments`
- `https://www.airwallex.com/docs/developer-tools/sandbox-environment/payments/simulate-shopper-action`
- `https://www.airwallex.com/docs/developer-tools/sandbox-environment/payments/simulate-payment-disputes`
- `https://www.airwallex.com/docs/payments/test-and-go-live/test-card-numbers`
- `https://www.airwallex.com/docs/payments/troubleshooting/error-response-codes`
- `https://www.airwallex.com/docs/payments/troubleshooting/response-codes` (issuer, AVS, CVC, merchant advice code pages)
- `https://www.airwallex.com/docs/api/simulation/payments` (and sub-pages)

Use for: sandbox defaults, LPM simulation, test cards, error handling.

## Account and plugins

- `https://www.airwallex.com/my` and `https://www.airwallex.com/my/pricing`
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/plugins`
- `https://www.airwallex.com/docs/payments/integration-options/web-checkout/plugins/built-by-airwallex/woocommerce/install-the-woocommerce-plugin`

Use for: registration, Malaysian entity, pricing link, plugin setup.
