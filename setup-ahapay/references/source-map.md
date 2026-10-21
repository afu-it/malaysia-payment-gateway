# AhaPay Source Map

Use this map to pick the right source before answering or coding. All pages checked on 2026-10-21. The developer portal at `https://apidocs.ahapay.my/` is a Docusaurus site whose canonical URLs point to `https://docs.fingular.com/`; both serve the same pages.

## Coverage

- Index used: `https://apidocs.ahapay.my/sitemap.xml` (39 URLs). `/llms.txt`, `/llms-full.txt` and `/robots.txt` return 404.
- OpenAPI: `/swagger` loads `https://apidocs.ahapay.my/openapi.json` (OpenAPI 3.0.0, V1 only, 4 paths, 6 schemas). Read in full.
- Enumerated: 39 sitemap URLs. Read: 39. The 13 WooCommerce subpages repeat sections of the single WooCommerce page; all were compared. The Flow page content is an SVG sequence diagram, read from its image file.
- Official plugin source read: WordPress.org `ahapay-buy-now-pay-later` v1.1.0 (author profile `ahapay`, last updated 2026-09-17): `payment-gateway.php`, `includes/class-ahapay-gateway.php`, `includes/class-ahapay-callback-handler.php`, `includes/class-ahapay-status-handler.php`, `readme.txt`.
- Supporting pages read: `https://ahapay.my/merchants` (merchant FAQ), `https://ahapay.my/sitemap.xml`. `https://merchant-help.ahapay.my/hc/en-my` returned 403 to automated fetch; `https://ahapay.my/refund-policy` renders client-side only.
- Searched, not found: GitHub repos, npm packages, Postman collection, PDF guide.
- Endpoints documented: 12 (7 V2, 4 V1, 1 outbound callback). All are in the catalogue in `ahapay-api.md`.
- Left out: product-label CSS, block-editor, webpack/packaging and design-pattern sections of the WooCommerce docs (plugin build detail, not integration facts).

## Overview And Flow

- Portal home: `https://apidocs.ahapay.my/`
- Introduction: `https://apidocs.ahapay.my/docs/api/Introduction`
- Flow (sequence diagram): `https://apidocs.ahapay.my/docs/api/Flow`

Use for: REST/JSON conventions, X-ApiKey on every request, create then redirect then status/callback sequence, refund and health in the flow.

## Authentication And Environments

- V2 Authentication: `https://apidocs.ahapay.my/docs/api/v2/Authentication`
- V1 Authentication: `https://apidocs.ahapay.my/docs/api/v1/Authentication`
- Plugin constants: `payment-gateway.php` (`AHAPAYBUYNOWPAYLATER_LIVE_BASE_URL`, `AHAPAYBUYNOWPAYLATER_SANDBOX_BASE_URL`)

Use for: sandbox and production base URLs, `X-ApiKey`, Merchant vs Partner key, `external_merchant_id`, `client_hash` variable.

## Order Creation

- Create Default Order: `https://apidocs.ahapay.my/docs/api/v2/Orders/Create-Default-Order`
- Create POS Order: `https://apidocs.ahapay.my/docs/api/v2/Orders/Create-POS-Order`
- Create QR POS Order: `https://apidocs.ahapay.my/docs/api/v2/Orders/Create-QR-POS-Order`
- V1 Create Order Online: `https://apidocs.ahapay.my/docs/api/v1/Create%20Order/Online`
- V1 Create Order Offline: `https://apidocs.ahapay.my/docs/api/v1/Create%20Order/Offline`
- OpenAPI `CreateOrderRequest` / `CreateOrderResponse`: `https://apidocs.ahapay.my/openapi.json`
- Plugin `process_payment()` request body and response parsing: `includes/class-ahapay-gateway.php`

Use for: request field tables, `order_items`, `redirect_urls`, `is_mobile_integration`, `pos_terminal`, `client_hash` rules (no `aha-` prefix, 10 minute lifetime), response fields, `online_order_payment_link` vs `link`, the `data` envelope, timeouts, and decline vs unknown outcome handling.

## Status

- V2 Check Order Status: `https://apidocs.ahapay.my/docs/api/v2/Check-Order-Status`
- V1 Check Order Status: `https://apidocs.ahapay.my/docs/api/v1/Check%20Order%20Status`
- Plugin status poll: `payment-gateway.php` (`/v2/orders/{id}/status` call)
- Plugin status mapping: `includes/class-ahapay-status-handler.php`

Use for: status enum, status requery, mapping statuses to order states, terminal-state rules.

## Callbacks And Verification

- V2 Webhook: `https://apidocs.ahapay.my/docs/api/v2/Webhook`
- Callback-Security: `https://apidocs.ahapay.my/docs/api/Callback-Security`
- OpenAPI `CallbackRequest`: `https://apidocs.ahapay.my/openapi.json`
- Plugin callback handler: `includes/class-ahapay-callback-handler.php`

Use for: `callback_url`, 200 OK and 65 second timeout, `X-Api-Key-Hash` = SHA-256 hex of the API key, the older HMAC `Signature` description, callback body fields, `online_order_status` vs `status`, ownership checks, duplicate and out-of-order handling, response codes.

## Refunds

- V2 Refund Order: `https://apidocs.ahapay.my/docs/api/v2/Refund-Order`
- V1 Refund Order: `https://apidocs.ahapay.my/docs/api/v1/Refund%20Order`
- Plugin `process_refund()`: `includes/class-ahapay-gateway.php`
- Merchant FAQ refunds: `https://ahapay.my/merchants`

Use for: full-refund-only rule, response `success`, unknown-outcome handling, merchant approval step.

## Limits, Health, Status Codes

- Daily Limit Status: `https://apidocs.ahapay.my/docs/api/v2/Daily-Limit-Status`
- V2 Health Check: `https://apidocs.ahapay.my/docs/api/v2/Health-Check`
- V1 Health Check: `https://apidocs.ahapay.my/docs/api/v1/Health%20Check`
- Status Codes: `https://apidocs.ahapay.my/docs/api/Status%20Codes`

Use for: daily sales limit reset at 12am, health response and key verification, HTTP 200/401/404/500 meanings.

## Platform Plugins

- WooCommerce (full manual): `https://apidocs.ahapay.my/docs/platforms/WooCommerce/`
- WooCommerce subpages (same content split): `.../WooCommerce/Overview`, `Plugin-Architecture`, `Installation-and-Setup`, `Core-Components`, `Code-Structure-Analysis`, `Configuration-Guide`, `Features-and-Functionality`, `Security-Features`, `Quick-Reference`, `API-Integration`, `Troubleshooting`, `Packaging-the-Plugin-for-Distribution`, `Conclusion`
- WordPress.org plugin: `https://wordpress.org/plugins/ahapay-buy-now-pay-later/`
- EasyStore: `https://apidocs.ahapay.my/docs/platforms/EasyStore/`, `.../EasyStore/Overview`, `.../EasyStore/Troubleshooting`
- Coming soon (Shopify, Magento, PrestaShop, OpenCart, Wix): `https://apidocs.ahapay.my/docs/platforms/coming-soon`

Use for: plugin install and settings, callback route `/wp-json/ahapaybuynowpaylater/v1/callback`, 55 MYR default minimum, MYR-only rule, EasyStore Merchant ID and API key setup, troubleshooting.

Note: the WooCommerce docs page section "10.3 API Integration" shows `/transactions` and `/refunds` endpoints and a `currency`/`merchant_id` request. These do not match the V2 API or the shipped plugin code; ignore them.

## Merchant Onboarding, Fees, Payout

- Merchant page and FAQ: `https://ahapay.my/merchants`
- Merchant portal: `https://portal.ahapay.my/login`
- Merchant help center: `https://merchant-help.ahapay.my/hc/en-my`
- Refund policy: `https://ahapay.my/refund-policy`

Use for: eligibility, documents, signup, MDR and SST (link only), transaction limits, payout schedule, dashboard scope, support contacts.

## Not Documented (checked 2026-10-21)

Rate limits, IP allowlists, callback retry policy and source IPs, idempotency header, error code list, partial refund, cancel/void/capture, recurring/tokenisation, payout and settlement APIs, sandbox test data, and the exact location (query vs path) of `external_merchant_id` on Daily Limit Status.
