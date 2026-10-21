# Kiple Source Map

Use this map to pick the right source before answering or coding. All sources checked 2026-10-21.

## Coverage

- Kiple publishes no public developer docs. Checked: `kiple.com` (403 to plain fetch; `robots.txt` points to `wp-sitemap.xml`, also 403), `kiple.com/llms.txt` (redirects to home), `developer.kiple.com`, `developers.kiple.com`, `docs.kiple.com`, `api.kiple.com`, `dev.kiplepay.com` (none resolve), `kiplepay.com/api` (404), WordPress.org plugin directory (no Kiple plugin), npm (none), Packagist.
- Official source found instead: GitHub organisation `https://github.com/m2c`, display name "Kiplepay Sdn Bhd", 12 public repos. Code lives on version branches; `master` holds only a readme.
- Enumerated payment-relevant repos: 10 (woocommerce, opencart, magento, prestashop, whmcsv, cubecart, oscommerce, php-sdk, KPPaymentSDK, kiple-tcp-pkg). Read: 9 in full on their latest branch. `kiple-tcp-pkg` (Go TCP library) and `kiplestar` (Go web framework) and `saml` are internal tooling with no payment API; skimmed and excluded.
- Packagist `greenpacket/kiplepay` points to `github.com/m2c/kiplepay-php-sdk`, which now returns 404 (deleted). Not used. Packagist `mprince/kiplepay` is third-party; not used.
- Live checks on 2026-10-21: all entry and enquiry URLs in `kiple-api.md` answered 200; UAT entry GET shows the validation error list; enquiry GET returns `Invalid Parameters`; iOS deeplink and `:94` portal hosts did not respond.
- Endpoints in official sources: 8 (5 current, 3 legacy iOS). All are in the Endpoint Catalogue.

## Hosted Payment, Hash, Return, Callback

- WooCommerce v4.5.6 (latest, 18/05/2023 changelog): `https://github.com/m2c/woocommerce/blob/woocommerce_plugin_v4.5.6/wc-kiple.php`
- WooCommerce readme: `https://github.com/m2c/woocommerce/blob/woocommerce_plugin_v4.5.6/readme.txt`
- WooCommerce v4.5.x (diff check): `https://github.com/m2c/woocommerce/tree/woocommerce_plugin_v4.5.x`
- PHP SDK v1.0: `https://github.com/m2c/php-sdk/blob/Kiple_PHP_SDK_v1.0/Kiple_PHP_SDK_v1.0/src/Kiple/PaymentGateway.php` plus `request.php`, `return.php`, `enquiry.php`

Use for: form fields, `version=2.0`, `dynamic_callback_url`, `nbcb` callback detection, SHA-1 request and response hash, return codes, field validation rules, UAT vs live URLs.

## Recurring

- OpenCart 3.1.5 controller: `https://github.com/m2c/opencart/blob/kiplepay_opencart_plugin_version_v3.1.5/kiplepay.ocmod/upload/catalog/controller/extension/payment/kiplepay.php`
- OpenCart model and form template: same branch, `catalog/model/extension/payment/kiplepay.php`, `catalog/view/theme/default/template/extension/payment/kiplepay.twig`
- OpenCart readme (recurring rules): `https://github.com/m2c/opencart/blob/kiplepay_opencart_plugin_version_v3.1.5/Readme.txt`

Use for: `ord_recurring_*` fields, `payment_code=CC`, frequency and charging-date rules, recurring callback, MYR-only note, `SameSite=None` cookie fix.

## Split Orders And Master Merchant

- Magento 2.3.5-p1 controller: `https://github.com/m2c/magento/blob/Magento_v2.3.5-p1/KiplePay_Plugin_Magento_v2.3.5-p1-common/KiplePay/Seamless/Controller/Index/Index.php`
- Magento 1.9 (legacy fields `ord_fcur`, `ord_famt`, test merchant): `https://github.com/m2c/magento/tree/Magento_v1.9`

Use for: `is_split_orders`, pipe-separated IDs and amounts, `ord_mmercID`, `all_split_totalamt`, `promo_bin`, return hash with master merchant.

## Enquiry (Requery)

- PrestaShop 1.7.1 (inside `webcash.zip`): `https://github.com/m2c/prestashop/tree/kiplepay_Prestashop_v1.7.1`, file `webcash/controllers/front/paymentreturn.php`
- PHP SDK `enquiry()`: see above

Use for: enquiry URLs (`/enquiry.php`, `/enquiries`), query parameters, `S`/`F`/`A` responses, requery-before-fulfil pattern.

## Other Platform Plugins

- WHMCS 7.2.0: `https://github.com/m2c/whmcsv/tree/WHMCS_plugin_v7.2.0` (`webcash.php`, `wccallback.php`, `Readme.txt` with test merchant)
- CubeCart 6.0: `https://github.com/m2c/cubecart/tree/kiplepay_Cubecart_v6.0` (`gateway.class.php`)
- osCommerce 2.3.4.1: `https://github.com/m2c/oscommerce/tree/oscommerce_v-2.3.4.1` (`epayment/response.php`, `install.txt`)

Use for: cross-checking the hash formula (same in all), legacy `staging.webcash.com.my` host, duplicate `wcID` check, echoed `ord_shipcountry`.

## Legacy iOS SDK

- KPPaymentSDK: `https://github.com/m2c/KPPaymentSDK` (`KPPaymentSDK/KPPayment.swift`, `KPPayment+Extension.swift`, `Sample3/`)

Use for: deeplink URL, `CheckSum` formulas, `Status` values, status API path. Historical only.

## Live Gateway Pages

- UAT entry: `https://uat.kiplepay.com/wcgatewayinit.php` (validation messages, RM1.0 minimum)
- Enquiry: `https://uat.kiplepay.com/enquiry.php`, `https://kiplepay.com/enquiries`

## Merchant, Settlement, Legal

- Merchant registration: `https://kiplepay.com/kyc/register`
- Legacy gateway service page: `https://kiplepay.com/pages/service/gateway-service`
- Merchant Terms: `https://kiplepay.com/img/document/20190601-Merchant-Terms.pdf`
- Settlement Process Schedule v2.0: `https://kiplepay.com/img/document/20190528-Merchant-Settlement-Process-Schedule.pdf`
- Dispute Process Schedule: `https://kiplepay.com/img/document/20190509-Merchant-Dispute-Process-Schedule.pdf`
- SME pricing: `https://kiple.com/sme/`

Use for: registration fields, settlement formula, refund and chargeback liability, pricing link.

## Not Covered (No Official Source)

Refund/void/capture API, payout API, settlement report API, tokenisation API, webhook retries, callback acknowledgement format, IP allowlist, rate limits, test cards, FPX test banks, payment method codes, preselecting a method, payment links API.
