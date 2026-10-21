# SecurePay Source Map

Use this map to pick the right official source before answering or coding. All pages are SecurePay-owned (securepay.my, docs.securepay.my, v2.docs.securepay.my which serves `securepay-4.gitbook.io/console-docs`, and the `securepay` WordPress.org profile). Append `.md` to any GitBook page URL for Markdown.

## Coverage

- Index used: `https://docs.securepay.my/llms.txt` (v1) and `https://v2.docs.securepay.my/llms.txt` (v2). Both sitemaps point to the same GitBook spaces.
- v1: 38 pages enumerated, 38 read. Several are video-only (YouTube embeds) or empty (`guides/api`, `payment-service-agent-psa`, `payment-mode`, `enable-amount-selection`).
- v2: 74 pages enumerated, 74 fetched. 56 merchant/partner/console pages read in full. 18 SecurePay On-Prem pages (administrator and developer guides for self-hosting the platform: Docker Swarm, PostgreSQL/Patroni, HAProxy, monitoring) were skimmed and left out because they describe running SecurePay on your own infrastructure, not accepting payments; their developer guide pages are empty.
- Official plugin source read: SecurePay For WooCommerce 1.0.18 (`https://downloads.wordpress.org/plugin/securepay.1.0.18.zip`), SecurePay For GravityForms 1.0.12 (checksum code only).
- Live public endpoints checked 2026-10-21: `GET /api/public/v1/banks/b2c` and `/b2b` on sandbox and production; `POST /api/v1/auth` on both v2 hosts (401 without credentials).
- No OpenAPI spec, Postman collection or SDK repository was found. `https://github.com/securepay` only hosts `securepay.github.io`.

## Docs Hub and Version Choice

- `https://www.securepay.my/docs/`

Use for: v1 (recommended, production) vs v2 (preview) choice, sandbox-first advice, support email.

## v1 Introduction and Credentials

- `https://docs.securepay.my/master`
- `https://docs.securepay.my/api/introduction`
- `https://docs.securepay.my/api/validate`

Use for: sandbox/production base URLs, API UID, Auth Token, Checksum Token, SecurePay Apps interfaces URL, validate endpoint and responses.

## v1 Payment Creation

- `https://docs.securepay.my/api/merchant/payment`
- `https://docs.securepay.my/api/partner/payment`
- `https://docs.securepay.my/api/partner`
- `https://docs.securepay.my/api/partner/partner-list`

Use for: `POST /v1/payments` fields, request checksum order and test vector, PHP sample, partner_uid rules, B2C/B2B1, bank selection view, cancel/timeout URLs, verified partners.

## v1 Bank Lists

- `https://docs.securepay.my/api/merchant/bank-list`
- `https://docs.securepay.my/api/partner/bank-list`

Use for: public and partner bank list endpoints, response fields, `VerifySign`, online/offline filter, sample codes.

## v1 Payment Session (beta)

- `https://docs.securepay.my/api/payment-session-beta`
- `https://docs.securepay.my/api/payment-session-beta/introduction`
- `https://docs.securepay.my/api/payment-session-beta/payment`

Use for: `POST /apis/payments`, Basic auth, JSON `payment_url`, USD conversion.

## v1 Callback and Redirect

- `https://docs.securepay.my/api/url`

Use for: redirect vs callback semantics, posted fields, `fpx_debit_auth_code`, callback checksum algorithm and test vector.

## v1 Transactions

- `https://docs.securepay.my/api/transactions`
- `https://docs.securepay.my/api/transactions/transactions-status`
- `https://docs.securepay.my/api/transactions/list`
- `https://docs.securepay.my/api/transactions/search`
- `https://docs.securepay.my/api/transactions/transactions-list` (deprecated)

Use for: status requery, pagination, filters, 90-day limit, search by FPX ref/order/exchange number.

## v1 Guides, Settlement, Plugins, Change Log

- `https://docs.securepay.my/guides` and sub-pages: create-account, referral, premium-domain, settlement, fpx-credit-card, payment-form (+5 sub-pages), api
- `https://docs.securepay.my/settlement`
- `https://docs.securepay.my/plugins`, `https://docs.securepay.my/plugins/woocommerce`
- `https://docs.securepay.my/payment-service-agent-psa` (empty)
- `https://docs.securepay.my/change-logs`

Use for: T+1 settlement, payment forms, plugin video, B2B1/payment mode/address changes (14 June 2021), settlement statement download (25 July 2021).

## v2 Getting Started

- `https://securepay-4.gitbook.io/console-docs/readme`
- `https://securepay-4.gitbook.io/console-docs/getting-started/quickstart`
- `https://securepay-4.gitbook.io/console-docs/getting-started/quickstart/account-registration`
- `https://securepay-4.gitbook.io/console-docs/getting-started/quickstart/supporting-documents`

Use for: Console registration steps, password rules, Starter package documents, 2MB limit.

## v2 Console Basics

- `https://securepay-4.gitbook.io/console-docs/basics/reports`
- `https://securepay-4.gitbook.io/console-docs/basics/reports-1` (Virtual Accounts)
- `https://securepay-4.gitbook.io/console-docs/basics/reports-1/credit-reload`
- `https://securepay-4.gitbook.io/console-docs/basics/interfaces`
- `https://securepay-4.gitbook.io/console-docs/basics/payment-form`
- `https://securepay-4.gitbook.io/console-docs/basics/emandate`
- `https://securepay-4.gitbook.io/console-docs/basics/settlements`
- `https://securepay-4.gitbook.io/console-docs/basics/settlements/split-settlement-rules`
- `https://securepay-4.gitbook.io/console-docs/basics/editor` (Supports)
- `https://securepay-4.gitbook.io/console-docs/basics/change-logs` (placeholder template content only)

Use for: interface settings, charge modes, gateways, virtual accounts, credit account, payment forms, eMandate frequencies, settlement cut-off table, split rules, support contacts.

## v2 Developer Tools and Payments

- `https://securepay-4.gitbook.io/console-docs/basics/integrations`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/validate`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/bank-list`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-session`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-session/get-payment-method`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-session/create-payment-session`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-session/closed-wallet-payment`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-records`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-records/status`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-records/list`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-mandate`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-mandate/initiate-emandate`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-mandate/emandate-status`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/payment-mandate/list`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/payments/callback-and-redirect-url`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/certificate`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/settlements`
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/settlements/split-settlement`

Use for: v2 base URLs, `/v1/auth`, Bearer token, bank lists (FPX, direct debit, DuitNow), enabled gateways, payment intents, closed wallet, eMandate fields and status, payment records, v2 callback fields, RSA/PQC keys, split settlement payload.

## v2 Merchant and Partner APIs

- `https://securepay-4.gitbook.io/console-docs/basics/integrations/merchant` and `.../merchant/validate`
- `.../merchant/memberships`, `.../memberships/membership-info`, `.../memberships/membership-members` (+ add-member, list-members, member-info, update-info, transactions), `.../memberships/reload-credit`
- `.../merchant/outlet-info`, `.../merchant/cashier-info` (empty)
- `.../merchant/closed-wallet`, `.../closed-wallet/wallets` (+ list-of-wallets, wallet-info, wallet-transactions, reload-wallet)
- `https://securepay-4.gitbook.io/console-docs/basics/integrations/partner` and `.../partner/api-tokens`, `.../partner/users`, `.../partner/accounts`

Use for: memberships, closed wallet, outlets, partner user/account onboarding, partner tokens.

## Website Policy Pages

- `https://www.securepay.my/settlement-schedule/`
- `https://www.securepay.my/refund/`
- `https://www.securepay.my/contact/`

Use for: settlement timing, refund process, sales/pricing contact.

## Official Plugins

- `https://wordpress.org/plugins/securepay/` (WooCommerce)
- `https://wordpress.org/plugins/securepay-for-gravityforms/`, `securepay-for-wpforms`, `securepay-for-fluentforms`, `securepay-for-givewp`, `securepay-for-paidmembershipspro`, `securepay-for-wpjobster`, `securepay-for-restrictcontentpro`

Use for: real-world field usage (browser form POST to `/v1/payments`, checksum construction, `partner_uid`, `payment_source`), endpoint constants, bank list caching. Do not copy their lack of callback checksum verification.
