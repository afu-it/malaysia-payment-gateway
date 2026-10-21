# Fasspay Source Map

Use this map to pick the right source before answering or coding. All checked 2026-10-21.

## Coverage

- Online e-commerce API: Fasspay publishes no public API document. Searched `fasspay.com` (sitemap, product, support pages), `api.fasspay.com` (404), `developer(s).fasspay.com` and `docs.fasspay.com` (no host), GitHub (no official repo; `github.com/softspace` has no public repos), npm and Packagist (nothing official). The official source found is the WordPress.org plugin by Fasspay. Read all 11 code files of plugin v1.0.14 in full (`fasspay-for-woocommerce.php`, `includes/class-fasspay-api.php`, `class-fasspay-gateway.php`, `class-fasspay-webhook-handler.php`, `class-fasspay-admin.php`, `class-fasspay-logger.php`, `templates/payment-form.php`, `assets/js/payment.js`, `assets/js/admin.js`, `uninstall.php`, `readme.txt`).
- Soft Space Developer Portal: enumerated 64 pages from the portal's navigation data (`/guides`, `/reference` sidebars), read 64 of 64. The site has no `llms.txt`, `sitemap.xml` or OpenAPI file (all routes return the SPA shell). Expandable "Show More" field tables were read from the embedded page data.
- Fasspay site pages read: home, products and services, payments, about, regulated partners, contact, pricing, getting started, FAQs, merchant guide, SME tier.
- Left out: merchant guide PDFs for devices (end-user device manuals, not integration docs), e-wallet issuing and money lending (not payment acceptance).

## Online Checkout (e-commerce API)

- Plugin page: `https://wordpress.org/plugins/fasspay-for-woocommerce/`
- Plugin source: `https://downloads.wordpress.org/plugin/fasspay-for-woocommerce.1.0.14.zip`
- Plugin info API: `https://api.wordpress.org/plugins/info/1.2/?action=plugin_information&request[slug]=fasspay-for-woocommerce`

Use for: base URLs, endpoint paths, `messages` envelope, HMAC signing headers, create-payment fields, amount in sen, callback verification, statusId and transactionMethod codes, void and capture, white label, test connection, settlement logic, UAT credentials.

## Product And Company

- `https://fasspay.com`
- `https://fasspay.com/productsandservices`
- `https://fasspay.com/payments`
- `https://fasspay.com/about-us`
- `https://fasspay.com/regulated-partners`
- `https://fasspay.com/sitemap.xml`

Use for: product scope, online "Payment Gateway" offering, company identity.

## Account, Pricing, Support

- `https://fasspay.com/support/contact-us`
- `https://fasspay.com/support/pricing`
- `https://fasspay.com/support/getting-started`
- `https://fasspay.com/support/faqs`
- `https://fasspay.com/support/merchant-guide`
- `https://fasspay.com/business-solutions/sme`

Use for: onboarding contact, pricing links, mPOS settlement note, device guides.

## Developer Portal Index

- `https://developers.softspace.com.my/`
- `https://developers.softspace.com.my/guides`
- `https://developers.softspace.com.my/reference`
- `https://developers.softspace.com.my/support/contact-us`

Use for: entry points to Fasstap and Smart Terminal docs.

## Fasstap SDK

- `/guides/fasstap/introduction/overview`
- `/guides/fasstap/development/sdk-integration/android/latest/overview`
- `/guides/fasstap/development/sdk-integration/android/latest/sdk-components`
- `/guides/fasstap/development/sdk-integration/android/latest/sdk-integration`
- `/guides/fasstap/development/sdk-integration/android/latest/status-code`
- `/guides/fasstap/development/sdk-integration/android/latest/transaction-event`
- `/guides/fasstap/development/sdk-integration/releases/android/latest/changelog`
- `/reference/fasstap/sdk-integration/android/latest/sdk-api`
- `/reference/fasstap/sdk-integration/android/latest/web-api`
- Index pages: `/guides/fasstap`, `/guides/fasstap/development/sdk-integration`, `/reference/fasstap`, `/reference/fasstap/sdk-integration`

Use for: SoftPOS requirements, setup, init, transaction API, outcome fields, result and UI event codes, status and error codes, provisioning Web API.

## Fasstap App-to-App

- `/guides/fasstap/development/app-to-app-integration/android/latest/overview`
- `/guides/fasstap/development/app-to-app-integration/android/latest/app-to-app-integration`
- `/guides/fasstap/development/app-to-app-integration/android/latest/integration-components`
- `/guides/fasstap/development/app-to-app-integration/android/latest/status-code`
- `/guides/fasstap/development/app-to-app-integration/module-flows/android/latest/` + `credit-settlement`, `get-token`, `get-transaction-status`, `installment-payment`, `payment`, `qr-check-status`, `qr-payment`, `qr-refund`, `quick-chip`, `quick-chip-completion`, `recurring-payment`, `refund`, `registration`, `sales-history`, `void-payment`
- `/guides/fasstap/development/app-to-app-integration/releases/android/latest/changelog`
- `/reference/fasstap/app-to-app-integration/android/latest/sdk-api`
- Index pages: `/guides/fasstap/development/app-to-app-integration`, `/guides/fasstap/development/app-to-app-integration/module-flows`

Use for: intent-based handoff, ActionType values, request and response VO fields, module flows, status codes.

## Smart Terminal

- `/guides/smart-terminal/introduction/android/latest/overview`
- `/guides/smart-terminal/development`, `/guides/smart-terminal/development/android/latest/sdk-components`, `/guides/smart-terminal/development/android/latest/response-codes`
- `/guides/smart-terminal/development/sdk-integrations/android/latest/` + `sdk-setup`, `sdk-initialization`, `terminal-connection`, `creating-and-sending-request`, `response-handling`
- `/guides/smart-terminal/development/sdk-module-flows/android/latest/` + `sale`, `qr-payment`, `qr-inquiry`, `void`, `cancel`, `transaction-status`, `settlement-info`, `settlement`, `send-ereceipt`, `membership-info`, `top-up`, `card-refund`, `refund-qr`, `bill-payment`
- `/guides/smart-terminal/releases/android/latest/changelog`
- `/reference/smart-terminal/android/latest/sdk-api`
- Index pages: `/guides/smart-terminal`, `/guides/smart-terminal/development/sdk-integrations`, `/guides/smart-terminal/development/sdk-module-flows`

Use for: USB setup, codec and reader API, ReaderTag keys, request objects, module flows, response codes.

All developer portal paths are relative to `https://developers.softspace.com.my`.
