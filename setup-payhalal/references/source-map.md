# Payhalal Source Map

Use this map to pick the right source before answering or coding. All checked 2026-10-21.

## Docs Index

- Developers landing page: `https://payhalal.my/home_v3/developers/` (links to the GitBook docs; its "Quick integration" code is a pseudo-example and does not match the API)
- GitBook index for agents: `https://souqa-fintech.gitbook.io/api-docs/llms.txt`
- GitBook full text: `https://souqa-fintech.gitbook.io/api-docs/llms-full.txt`
- Integration guide home: `https://souqa-fintech.gitbook.io/api-docs/`

Use for: finding every official API page in one fetch.

## Merchant Onboarding

- Introduction: `https://souqa-fintech.gitbook.io/api-docs/payhalal-introduction`
- Registration form: `https://payhalal.my/home_v3/register/`
- Merchant Dashboard login: `https://merchant.payhalal.my/login.php`
- FAQ: `https://payhalal.my/home_v3/faq/`
- Pricing: `https://payhalal.my/home_v3/pricing/`

Use for: Syariah and Malaysia requirements, UAT mode before verification, required documents, payment method list, pricing link, refund responsibility.

## Payment Request

- Payment flow overview: `https://souqa-fintech.gitbook.io/api-docs/payment-flow-overview`
- URI reference: `https://souqa-fintech.gitbook.io/api-docs/uri-reference`
- Request parameters (V1, V2, QR): `https://souqa-fintech.gitbook.io/api-docs/payments/payment-request-parameters`
- PHP example V1 and V2: `https://souqa-fintech.gitbook.io/api-docs/payments/example-payment-request-via-php-for-v1-and-v2`
- V2 JSON response sample: `https://souqa-fintech.gitbook.io/api-docs/payments/example-payment-request-via-php-for-v1-and-v2/sample-json-payment-link-response-for-v2`
- Request errors: `https://souqa-fintech.gitbook.io/api-docs/payments/payment-request-error-response`

Use for: pay URLs per environment, GET/POST support, request fields, V2 limited availability, QR response, minimum amount, error messages.

## Hash And Callback

- Request hash: `https://souqa-fintech.gitbook.io/api-docs/payments/generate-hash-for-payment-request`
- Callback fields: `https://souqa-fintech.gitbook.io/api-docs/payments/payment-response-callback`
- Callback URL and callback hash: `https://souqa-fintech.gitbook.io/api-docs/payments/payment-response-callback/callback-url-to-your-server`

Use for: SHA256 string order, callback field list, statuses, optional Callback URL in Developer Tools, "OK" response.

## Reconciliation

- Single reconcile: `https://souqa-fintech.gitbook.io/api-docs/transaction-reconciliation/single-transaction-reconcile`
- Batch reconcile: `https://souqa-fintech.gitbook.io/api-docs/transaction-reconciliation/batch-transaction-reconcile`
- Transaction list: `https://souqa-fintech.gitbook.io/api-docs/transaction-reconciliation/transaction-list`
- Reconcile hash: `https://souqa-fintech.gitbook.io/api-docs/transaction-reconciliation/formula-of-hash-transaction-reconciliation`

Use for: status requery, UAT vs production reconcile hosts, MD5 hash, date range and paging limits, response fields.

## Official Plugins (second source for real behaviour)

- WooCommerce plugin (Souqa Fintech, v1.0.5): `https://github.com/SouqaFintech/payhalal-woocommerce-plugin`
  - Gateway logic: `https://github.com/SouqaFintech/payhalal-woocommerce-plugin/blob/main/includes/class-payhalal-wc-gateway.php`
  - Dashboard URL setup: `https://github.com/SouqaFintech/payhalal-woocommerce-plugin/blob/main/README.md`
  - Changelog (MYR hash fix, recurring endpoint): `https://github.com/SouqaFintech/payhalal-woocommerce-plugin/blob/main/CHANGELOG.md`
  - Wiki (where keys live): `https://github.com/SouqaFintech/payhalal-woocommerce-plugin/wiki`
- OpenCart 3.x extension: `https://github.com/SouqaFintech/opencart3.x-payhalal-extension/blob/main/upload/catalog/controller/extension/payment/payhalal.php`
- OpenCart 1.x extension: `https://github.com/SouqaFintech/opencart1.x-payhalal-extension`
- PrestaShop module: `https://github.com/SouqaFintech/payhalal-prestashop-plugin`
- Legacy (2018, superseded): `https://github.com/payhalal/woocommerce-plugin`

Use for: callback hash without `transaction_id`, extra statuses `FAIL` and `TIMEOUT`, Return/Success/Cancel/Callback URL setup, test mode key handling, amount formatting, recurring (`seamless/Nomu`) flow.

## Not Covered By Official Sources

Refund API, callback retries and IPs, rate limits, sandbox test cards/banks, settlement timing. Ask `tech_support@payhalal.my`.

## Coverage

Checked 2026-10-21.

- Index used: `https://souqa-fintech.gitbook.io/api-docs/llms.txt` (17 pages; re-fetched 2026-10-21, unchanged) and `https://souqa-fintech.gitbook.io/api-docs/sitemap-pages.xml` (17 URLs, same set).
- Pages enumerated: 17. Pages read: 17 (all, via `llms-full.txt`): Integration Guide, Introduction, Payment Flow Overview, URI Reference, Payments, Payment Request Parameters, Generate Hash, PHP Example V1/V2, Sample V2 JSON, Payment Response (Callback), Callback URL To Your Server, Payment Request Error Response, Transaction Reconciliation, Single Reconcile, Batch Reconcile, Transaction List, Formula of Hash.
- No OpenAPI, Postman collection, PDF guide or official SDK is published. Official plugin source read: WooCommerce v1.0.5 (gateway class, redirect template, README, CHANGELOG, wiki), OpenCart 3.x controller, PrestaShop `validation.php`, legacy 2018 WooCommerce plugin. `SouqaFintech` PHP API repo README is a stub (no code).
- Endpoints catalogued: 13 documented + 3 plugin-only (see `payhalal-api.md` Endpoint Catalogue).
- Left out: nothing in scope. Payment Terminal and Soundbox products have no developer docs.

## Sources For Facts Added 2026-10-21 (completeness top-up)

- Endpoint catalogue rows 1 to 13, V2/QR/reconcile response field tables, error table, enums: the GitBook pages above.
- Plugin-only `check-enabled.php` and recurring request fields: `https://github.com/SouqaFintech/payhalal-woocommerce-plugin/blob/main/includes/class-payhalal-wc-gateway.php`; `payment-cycle` values: `https://github.com/SouqaFintech/payhalal-woocommerce-plugin/blob/main/README.md`.
- Legacy `cart_payment.php` and `number_format($total, 2)` thousand separator: `https://github.com/SouqaFintech/payhalal-prestashop-plugin/blob/main/modules/payhalal/controllers/front/validation.php`.

## Corrections Log

- No existing facts were found wrong in the top-up review. Added a warning that the GitBook PHP samples contain bugs (`$values` vs `$ph`, hardcoded secret, `$ph_app_secret == true`, non constant-time compare, float amount).
