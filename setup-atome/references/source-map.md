# Atome Source Map

Use this map to pick the right official source before answering or coding. All pages checked 2026-10-21.

## Product And Market Overview

- Malaysia merchant page: `https://www.atome.my/en-my/business`
- Malaysia home: `https://www.atome.my/`

Use for: BNPL positioning, sales contact, retailer email, and the fact that pricing is not published.

## API Reference (Online, v2)

- Redoc page: `https://doc.apaylater.com/v2/`
- OpenAPI spec: `https://doc.apaylater.com/v2/swagger.yaml`

Use for: base URLs, Basic auth, `POST /auth`, `POST /payments`, `GET /payments/{referenceId}`, `POST /payments/{referenceId}/cancel`, `POST /payments/{referenceId}/refund`, callback body, `X-Signature` HMAC-SHA256, payment states, amount units and minimums, error codes, IP allowlist, FAQ on expiry, refunds, and failure causes. This is the primary source for every API fact in this skill.

## Checkout Flow Diagram

- `https://aaclub-sg-prod.s3.amazonaws.com/public/open-api/atome_open_api_flowchart.jpg`

Use for: the 10-step sequence between merchant site, merchant server, Atome site/app, and Atome server.

## Offline / POS API (v1)

- `https://doc.apaylater.com/v1/`

Use for: in-store barcode integration only. Not covered by this skill.

## Developer Portal

- Home: `https://partner.apaylater.com/`
- FAQ: `https://partner.apaylater.com/docs/Help/FAQ`
- Customer payment process: `https://partner.apaylater.com/docs/Help/CustomerPaymentProcess`
- Sitemap: `https://partner.apaylater.com/sitemap.xml`

Use for: callback retry policy (5 times in 5 hours), test vs production endpoints, app vs browser payment by country (MY is app only), plugin display threshold.

## Platform Plugins

All under `https://partner.apaylater.com/docs/`:

- WooCommerce: `WooCommerce/Overview`, `WooCommerce/PluginIntegration`, `WooCommerce/PluginUpdate`, `WooCommerce/RefundProcess`, `WooCommerce/FAQ`
- Shopify: `Shopify/Overview`, `Shopify/InstallAtomePlugin`, `Shopify/OrderReconciliation`, `Shopify/PriceDividerSetup`, `Shopify/PriceDividerSetupAfter2026`, `Shopify/RefundProcess`, `Shopify/UninstallPlugin`, `Shopify/FAQ`
- Magento: `Magento/Overview`, `Magento/IntegrationProcess`, `Magento/RefundProcess`, `Magento/SKUPermission`, `Magento/FAQ`
- PrestaShop: `Prestashop/Overview`, `Prestashop/IntegrationProcess`, `Prestashop/RefundProcess`, `Prestashop/SKUPermission`, `Prestashop/FAQ`
- OpenCart: `Opencart/Overview`, `Opencart/MerchantConfiguration`, `Opencart/RefundProcess`, `Opencart/FAQ`
- EasyStore: `EasyStore/Overview`, `EasyStore/MerchantConfiguration`, `EasyStore/FAQ`
- Wix: `Wix/Overview`, `Wix/MerchantConfiguration`, `Wix/RefundProcess`, `Wix/FAQ`
- Salesforce: `Salesforce/Summary`, `Salesforce/ComponentOverview`, `Salesforce/ImplementationGuide`, `Salesforce/Testing`, `Salesforce/UserGuide`, `Salesforce/OperationsMaintenance`

Use for: everything in `references/atome-plugins-sdk.md`: plugin downloads, install and config paths, settings, plugin refunds, Shopify reconciliation by Payment ID, price divider, SKU permission, Salesforce cartridge and `atomeApis` helpers, plugin error messages.

## Official Plugin Source

- WooCommerce plugin v6.9.0: `https://gateway.apaylater.com/woocommerce/v4/aaclub_wc_plugin.zip` (files `includes/Payment/SubmitPayment.php`, `Callback.php`, `Result.php`, `Refund.php`, `Auth.php`, `includes/Enums/Atome.php`, `includes/Config/SyncConfig.php`, `i18n/locale-info.php`)

Use for: reference implementation of create, callback, result and refund handling; v1 base URL used by the plugin; unauthenticated `GET /variables/{COUNTRY}` helper; MY minimum spend 10; cancel timeout 10 to 720 minutes; the fact that the plugin neither verifies `X-Signature` nor sends `refundId`.

## Mobile SDKs

- iOS: `https://github.com/Atome-FE/AtomeMerchantDemo-iOS` (README)
- Android: `https://github.com/Atome-FE/AtomeMerchantDemo-Android` (README)
- React Native: `https://github.com/Atome-FE/AtomeMerchantDemo-RN` (README)
- Flutter: `https://github.com/Atome-FE/AtomeMerchantDemo-Flutter` (README)

Use for: SDK package names (`AtomeSDK` pod, `io.github.atome-fe:merchant-sdk:1.5.1`, `react-native-atome-paylater`, `atome_sdk_flutter`), `isAtomeInstalled` and open-URL method names, `apaylater` iOS query scheme, Android `TakePictureLauncher`.

## Offline / POS Spec (v1)

- `https://doc.apaylater.com/v1/swagger.yaml`

Use for: the endpoint catalogue entry for `POST /payments/{referenceId}/barcode_pay` and `BARCODE_INVALID`. Offline QR and barcode flows are otherwise out of scope.

## Coverage

Checked 2026-10-21. Indexes used: `https://doc.apaylater.com/v2/swagger.yaml`, `https://doc.apaylater.com/v1/swagger.yaml`, and `https://partner.apaylater.com/sitemap.xml`. No `llms.txt` exists (`doc.apaylater.com/llms.txt` is 404; `partner.apaylater.com/llms.txt` and `llms-full.txt` return the site HTML shell, not an index).

| Source | Enumerated | Read | Notes |
| --- | --- | --- | --- |
| v2 OpenAPI spec (online) | 1 file: 6 operations, 9 schemas, FAQ (11 Qs), Error code table (26 codes), Postman tag | 1 (whole file) | Primary source. Every operation and schema is in `atome-api.md`. |
| v1 OpenAPI spec (offline) | 1 file: 7 operations | 1 | Only `barcode_pay` differs from v2; listed in the catalogue, otherwise out of scope. |
| Developer portal sitemap | 45 URLs | 45 | 42 docs pages plus home, `/search`, and a Docusaurus `/markdown-page` placeholder (no content). |
| Official WooCommerce plugin source | 1 zip | Payment, Enums, Config, i18n files | Reference implementation. |
| Official SDK repos (Atome-FE) | 4 | 4 READMEs | |
| Flow diagram | 1 | 1 | |
| Malaysia merchant site | 2 pages | 2 | Sales contact; no pricing. |

Not read or left out, and why:

- Postman screenshots Google Doc linked from the spec: third-party host, screenshots only.
- Singapore and Thailand callback IP lists in the spec: not relevant for a Malaysian merchant (only MY IPs are copied).
- v1 Dynamic QR and Barcode POS flow diagrams: offline in-store only.
- Plugin screenshots: images only; the text around them is captured.

## Corrections Made In The 2026-10-21 Top-Up

- No existing statement was found wrong. Added: endpoint catalogue (including v1 `barcode_pay` and the plugin-only `/variables` helper), nested request field tables, create, refund, and cancel response examples, `paymentTransaction` and `refundTransactions` tables, callback field table, enums table, the full verbatim error table (added `BARCODE_INVALID` and exact messages), plugin and SDK reference file, and WooCommerce plugin source findings.
- Clarified: the official WooCommerce plugin calls the `/v1` base, while the spec documents `/v2` for online. Native online integrations should use `/v2`.

## Gaps (ask the Atome Account Manager)

- Callback secret issuance and `X-Signature` encoding.
- Test shopper accounts.
- Maximum amount, MDR, settlement schedule, merchant portal URL.
- Rate limits.
