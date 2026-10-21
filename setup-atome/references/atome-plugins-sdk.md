# Atome Plugins And App SDKs

Use this reference when a merchant uses an Atome platform plugin instead of the native API, or calls Atome from a native app. All facts come from the Atome Developer Portal (`https://partner.apaylater.com/`), the official plugin downloads, and the official `Atome-FE` GitHub repos, checked 2026-10-21. API facts are in `references/atome-api.md`.

## Platforms With Official Integrations

Developer portal FAQ (`https://partner.apaylater.com/docs/Help/FAQ`): Salesforce, EasyStore, Magento, OpenCart, PrestaShop, Shopify, Wix, WooCommerce. Any other platform uses the native API.

Every plugin asks for the same credentials: **API Key** and **API Password** (the Basic auth username and password from the Atome team; Magento screenshots call them "Merchant Key" and "Merchant Secret"), plus Country/Region and Environment (Test or Production).

## Common Plugin Settings (WooCommerce, Magento, OpenCart, PrestaShop)

| Setting | Required | Meaning |
| --- | --- | --- |
| Country/Region | yes | Website country; drives processing currency (`MY` gives `MYR`). |
| Language | yes (Woo, OpenCart, PrestaShop) | Price divider and checkout tag language. |
| Environment | yes | Test or Production. |
| API Key / API Password | yes | Atome credentials. |
| Maximum/Minimum Amount | no | Hide Atome when the order total is outside the range. |
| Cancel Timeout | no | Default 720 minutes (12 hours); unpaid orders auto-cancel. Woo plugin source allows 10 to 720. |
| Debug | recommended | Full log for debugging. |
| Price Divider | recommended | Shows "or 3 instalments of ..." on listing and product pages. |
| SKU Permission | recommended | Lists products in the Atome app (Woo, OpenCart, PrestaShop, Magento). |

Magento also has New Order Status, Order Status, Order Email Sent By, Clear Cart Without Paying, Sort Order, and Exclude Category.

## Per-Platform Install

| Platform | Install | Config path | Refund |
| --- | --- | --- | --- |
| WooCommerce 4.0+ | Upload `https://gateway.apaylater.com/woocommerce/v4/aaclub_wc_plugin.zip` (3.6 to 4.0: `https://gateway.apaylater.com/woocommerce/aaclub_wc_plugin.zip`) via Plugins, Add New, Upload Plugin. Update with "Update Now" on the plugins page. | WooCommerce, Settings, Payments, Atome, Manage; then enable Atome. | Order, Refund, enter qty/total, "Refund via Atome". |
| Magento 2 | Download the zip for 2.0+, 2.4.4+, or 2.4.8+ from the Integration Process page, run `php atome_magento2_plugin/setup.php <MAGENTO2_ROOT_DIR>` until "Done". Magento 1 zip: `https://gateway.apaylater.com/magento1/atome_magento1_plugin.zip`. | Stores, Configuration, Sales, Payment Methods, Other Payment Methods, Atome; Save Config, then Cache Management and Flush Magento Cache. | Order, Invoices, View, Credit Memo (only the credit memo under the Invoice tab refunds online through Atome). |
| OpenCart 3.0+ | Upload `https://gateway.apaylater.com/opencart3/Atome-OC3.0.ocmod.zip` in Extensions, Installer; install in Extensions, Extensions, Payments (+); then Extensions, Modifications, Refresh. OpenCart 2.3 zip: `https://gateway.apaylater.com/opencart2/Atome-OC2.3.ocmod.zip`. | Payment extension settings. | Sales, Orders, Order History, Atome Payment tab. Partial refunds only in Atome Merchant Center. |
| PrestaShop | Module Catalog search "Atome", or upload: 1.7.0 to 1.7.9 `https://gateway.apaylater.com/prestashop/atome_prestashop17_plugin.zip`, 8.0.0 to 8.2.3 (S3 link on the Integration Process page), 1.6 `https://gateway.apaylater.com/prestashop/atome_prestashop16_plugin.zip`. | Modules, Module manager, Atome, Configure. | Orders, set Status to Refunded, Update Status. Partial refunds only in Atome Merchant Center. |
| Shopify | Install "Atome Pay Later" from `https://apps.shopify.com/atomepaylater`, enter API credentials, Activate, then Activate in Payment settings. | Settings, Payments. | Shopify order Refund button (recommended). Refunds in Atome Merchant Center do not sync back to Shopify. |
| Wix | Settings, Accept Payments, Add Payment Methods, Atome, Add; Set Up Account to Activate; Connect Existing Account; enter API Key and Password; Connect My Account. | Accept Payments. | More Actions, Issue Refund. Partial refunds only in Atome Merchant Center. |
| EasyStore | Settings, Payment, Add Payment Method, search "Atome", enter API Key and Password, Save. | Payment settings. | Not documented in official docs (checked 2026-10-21). |
| Salesforce (SFRA 5.0.1) | Download `https://atome-paylater-fe.s3.amazonaws.com/merchant-plugins/production/salesforce/link_atome_sfra.zip`; add cartridge `int_atome_SFRA` to the site cartridge path; import metadata (rename site folder `Atome_SG` to your site id). | Site Preferences group "Atome Payment Config" (`atomeEnabled`, `AtomeContentOnPLP`, `AtomeContentOnCart`, `AtomeContentOnPDP`); payment method and processor `ATOME_PAYMENT`; service URL, user, password per environment. | Not in default SFRA; call `atomeApis.refundPayment(refundAmount, orderID)` or `atomeApis.cancelPayment(orderID)` from `*/cartridge/scripts/service/atomeApis` in custom code. |

The Salesforce Summary says the cartridge serves SG, MY, and HK and that SGD and MYR are the only currencies it services.

## Plugin Gotchas

- Shopify "The partner merchant is not found": wrong key or password, often a stray space.
- Shopify "The partner merchant already onboarded": one key and password pair works for one Shopify store only.
- Shopify only accepts the store's base (local) currency.
- Shopify reconciliation: Shopify does not share its order id with payment apps. Match on the Shopify **Payment ID** (order Timeline "Information from the gateway", or the Payment ID column in the CSV export). After migrating to the new Shopify app, the Merchant Center and SFTP "E-commerce Platform Order ID" column holds the Payment ID.
- Shopify uninstall: versions after July 2026 deactivate Atome Paylater in Settings, Payments, then Uninstall; older versions (Atome Post Pay) use More actions, Manage, Uninstall.
- Shopify price divider: integrations before 2026-01-01 create a custom app "Atome Price Divider" with scopes `read_orders`, `read_products`, `read_script_tags`, `write_script_tags` and email the `shpat_` admin token to Atome; integrations after 2026-01-01 create the app in the Shopify Dev Dashboard with the same scopes, release, install, and email the Client ID and Secret to Atome.
- SKU Permission (Magento, PrestaShop): generate a store API key with Products permissions (PrestaShop: Advanced Parameters, Web service) and send it to the Atome team; setup takes 1 to 3 working days.
- Customer display: the Customer Payment Process page says Atome shows at checkout only above SGD10, HKD100, or MYR50 in plugin stores. This is plugin display, not the API minimum (RM10.00).
- Callback failures in plugin stores are retried 5 times in 5 hours, then Atome emails you.

## What The Official WooCommerce Plugin Source Shows (v6.9.0)

Useful as a reference implementation; it is official Atome code.

- It calls the **v1** base (`https://api.apaylater.net/v1` staging, `https://api.apaylater.com/v1` production) with the same `/auth`, `/payments`, `/payments/{id}`, `/payments/{id}/refund` paths and the v2-style body (customer, address, items). A native online integration should use the documented v2 base.
- Headers: `Authorization: Basic ...`, `Content-Type: application/json`, plus plugin telemetry headers (`X-Version`, `X-Woocommerce-Version`, `X-PHP-Version`) that are not required by the spec.
- `referenceId` and `merchantReferenceId` are both the WooCommerce order id. `callbackUrl` is `<site>/?wc-api=ATOME&referenceId=<id>`; `paymentResultUrl` is `<site>/?wc-api=RESULT&referenceId=<id>`; `paymentCancelUrl` is the checkout URL.
- Mobile user agents get `appPaymentUrl`; others get `redirectUrl`.
- Callback and result handlers both requery `GET /payments/{id}` and never trust the request; paid orders move only from pending, inside a DB transaction.
- It does not verify `X-Signature` and does not send `refundId`.

## Native App SDKs

From the official READMEs. Use them to open `appPaymentUrl` in the Atome app.

| Platform | Install | Check installed | Open payment URL |
| --- | --- | --- | --- |
| iOS (10+) | CocoaPods `pod 'AtomeSDK'`; add `apaylater` to `LSApplicationQueriesSchemes` (Queried URL Schemes) in Info.plist | `AtomeManager.shared().isAtomeInstalled()` | `AtomeManager.shared().handPaymentURL(url)` (Obj-C `[[AtomeManager sharedManager] handPaymentURL:url options:nil completionHandler:nil]`) |
| Android | `implementation 'io.github.atome-fe:merchant-sdk:1.5.1'`; call `AtomeSDK.init(applicationContext)` | `AtomeSDK.isAtomeInstalled()` | `AtomeSDK.handleUrl(url)` |
| React Native | `npm install react-native-atome-paylater` | `await isAtomeAppInstalled()` | `handlePaymentURL(url)` |
| Flutter | `atome_sdk_flutter: ^0.0.2`; iOS needs the same `apaylater` query scheme | `await AtomeSdkFlutter().isAtomeInstalled()` | `await AtomeSdkFlutter().openAtomeLink(url)` |

Android: if the Atome web page needs the camera inside your WebView, create `TakePictureLauncher.create(this)` and return `takePictureLauncher.onShowFileChooser(...)` from `WebChromeClient.onShowFileChooser`. Any WebView showing Atome must allow DOM storage (`localStorage`, `sessionStorage`).

The SDK only opens the Atome app. Payment status still comes from the callback plus `GET /payments/{referenceId}` on your server.

## Merchant Center

Plugin refund pages say Atome provides a Merchant Center for transactions, refunds (partial refunds for OpenCart, PrestaShop, and Wix stores), and settlement information. Its URL is not published in the public docs (checked 2026-10-21); the Account Manager gives access.
