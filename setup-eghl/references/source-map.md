# eGHL Source Map

Use this map to pick the right official source before answering or coding. All checked 2026-10-21. The full eGHL Merchant API specification PDF is shared with merchants only; the ADAPTIS guide PDFs say they are not for public circulation, so cite them only for setup steps, not as an API spec.

## Merchant API SDK (Query, Refund, Reversal, Capture)

- README: `https://bitbucket.org/eghl/merchantapi/src/master/README.md`
- Base class: `https://bitbucket.org/eghl/merchantapi/src/master/src/core/APIBase.php`
- Query status helpers: `https://bitbucket.org/eghl/merchantapi/src/master/src/Query.php`
- Refund / Reversal / Capture classes: `https://bitbucket.org/eghl/merchantapi/src/master/src/`
- Packagist: `https://packagist.org/packages/e-ghl/merchant-api`

Use for: test and live payment URLs, server-side POST pattern, URL-encoded responses, response hash check, TxnStatus codes (0, 1, 2, 9, 10, 15, 16, 31, -1, -2), refund/reversal/capture rules.

## Hashing

- Hash class: `https://bitbucket.org/eghl/eghl_hash/src/master/eGHL_HASH.php`
- WHMCS callback (explicit HashValue2 string): `https://bitbucket.org/eghl/adaptis-whmcs/src/main/` (`modules/gateways/callback/eghl.php` inside the zip)
- Magento response handler (`calculateHash2`): `https://bitbucket.org/eghl/adaptis-magento/src/main/` (`Controller/Index/ResponseHandler.gte_m230.php` inside the zip)

Use for: SHA-256 algorithm, field order for SALE, HashValue2, API request and response hashes, case-insensitive comparison.

## Payment Request Fields

- Param validator: `https://bitbucket.org/eghl/paramvalidator/src/master/src/PaymentRequest.php`
- Validator rules: `https://bitbucket.org/eghl/paramvalidator/src/master/src/core/validatorBase.php`
- Settlement request fields: `https://bitbucket.org/eghl/paramvalidator/src/master/src/SettlementRequest.php`

Use for: required fields, types, max lengths, 2-decimal amount rule, `;` for `&` in URLs, TransactionType values (QUERY, CAPTURE, RSALE, REFUND, SETTLE).

## Hosted Checkout, Return, And Callback

- Omnipay README: `https://bitbucket.org/eghl/eghl-omnipay/src/master/README.md`
- Omnipay gateway and messages: `https://bitbucket.org/eghl/eghl-omnipay/src/master/src/`
- WooCommerce plugin: `https://bitbucket.org/eghl/adaptis-woocommerce/src/main/` (`eghl-payment-gateway-class.php` inside `adaptis-woocommerce-8.2.4.zip`)
- WHMCS plugin: `https://bitbucket.org/eghl/adaptis-whmcs/src/main/`
- Magento plugin: `https://bitbucket.org/eghl/adaptis-magento/src/main/`
- OpenCart plugin: `https://bitbucket.org/eghl/adaptis-opencart/src/main/`

Use for: auto-submit form, return vs callback roles, response fields, `OK` acknowledgement, buyer-cancelled handling, pending handling, PymtMethod options, idempotent order updates.

## Account, Brand, And Operations

- NTT DATA Payment Services Malaysia: `https://my.nttdatapay.com/en`
- ADAPTIS e-Commerce: `https://my.nttdatapay.com/en/payment-services/adaptis-e-commerce`
- Contact and entities: `https://my.nttdatapay.com/en/contact-us`
- Our Story: `https://my.nttdatapay.com/en/company/our-story`
- Downtime announcements: `https://my.nttdatapay.com/en/downtime-announcements`
- WooCommerce guide PDF: `https://bitbucket.org/eghl/adaptis-woocommerce/src/main/ADAPTIS_WooCommerce_Merchant_Installation_Guide_V3_20251210.pdf`
- Magento guide PDF: `https://bitbucket.org/eghl/adaptis-magento/src/main/ADAPTIS_Magento_2x_Installation_Guide_v2_20251210.pdf`

Use for: ADAPTIS rebrand, NTT DATA entities, enquiry/onboarding path, plugin setup, product list, downtime checks.

## API Request Validators, SOPBL, And Plugin Behaviour (added 2026-10-21 top-up)

- `https://bitbucket.org/eghl/paramvalidator/src/master/src/QueryRequest.php`, `RefundRequest.php`, `ReversalRequest.php`, `CaptureRequest.php`, `SettlementRequest.php`, `core/validatorFactory.php`, `README.md`
- `https://bitbucket.org/eghl/eghl-exception/src/master/` (`src/Exception.php`, `README.md`)
- `https://bitbucket.org/eghl/merchantapi/src/master/src/core/APIFactory.php`, `src/Capture.php`, `src/Refund.php`, `src/Reversal.php`
- Magento (`eghl-magento-2.1.7.zip`): `Helper/Data.php` (SOPBL forex GET, JSON `Forex`), `Controller/Index/ResponseHandler.gte_m230.php` (`validateAmount` 0.1 forex tolerance, `GET_LOCK`, `ERROR` on fraud), `Block/Terminal.php`, `Model/Config/Source/PayMethods.php`, `PaymentURL.php`, `CurrencyType.php`, `etc/config.xml`
- PrestaShop 1.6 (`https://bitbucket.org/eghl/adaptis-prestashop-1.6.x/src/main/`, `eghl-eghl-prestashop1.6.x-20250523.zip`): `modules/eghl/eghl.php`, `eGHLResponseHandler.php`, `eghl.tpl`, guide `eGHL Prestashop 16x.pdf`
- OpenCart (`ghl.ocmod.zip`): `catalog/controller/payment/ghl.php`, `catalog/view/template/payment/ghl.twig`; guide `ADAPTIS_Opencart_3x_Integration_Guide_v2_20251210.pdf`
- WooCommerce 8.2.4: `eghl-payment-gateway-class.php` (`init_form_fields`, `is_valid_for_use` currency list, callback/redirect handlers)
- Packagist vendor `e-ghl`: `https://packagist.org/packages/e-ghl/merchant-api`, `/e-ghl/omnipay`, `/e-ghl/param-validator`, `/e-ghl/exception`

Use for: endpoint catalogue, API request field tables, settlement fields, SOP/SOP2/SOPBL/SOPCARD hash orders, SOPBL forex call, response field table, multi-currency handling, acknowledgement bodies, validator error messages, currency list, plugin anti-patterns.

## Coverage

- Index used: Bitbucket workspace listing `https://api.bitbucket.org/2.0/repositories/eghl` (10 public repos: `eghl_hash`, `eghl-omnipay`, `paramvalidator`, `merchantapi`, `eghl-exception`, `adaptis-woocommerce`, `adaptis-whmcs`, `adaptis-magento`, `adaptis-opencart`, `adaptis-prestashop-1.6.x`). Each local copy was checked against the latest commit hash on 2026-10-21 and is current. Packagist search lists 4 official `e-ghl/*` packages (all mirrors of the repos above).
- No docs site exists: `my.nttdatapay.com/llms.txt`, `/en/developer`, `/en/developers` return 404; its `robots.txt` sitemap line is `undefined/sitemap.xml`. `developer.e-ghl.com`, `docs.e-ghl.com`, `e-ghl.com` do not resolve. No OpenAPI or Postman collection is published.
- Enumerated: 10 repos, 62 non-asset files at repo level plus the plugin zips (194 PHP files inside plugins, excluding vendored copies), 5 PDFs (WooCommerce, Magento, OpenCart, PrestaShop guides; card interchange fee PDF).
- Read in full: all SDK source and READMEs (`merchantapi`, `eghl_hash`, `paramvalidator`, `eghl-omnipay`, `eghl-exception`), every gateway/request/response/callback file of the 5 plugins (WooCommerce, Magento, OpenCart, WHMCS, PrestaShop), and the 4 plugin guide PDFs. Plugin admin UI, language, image, stock, logger, and test files were scanned (grep for hashing, URLs, `TransactionType`, tokens), not read line by line.
- Operations documented: 12 (see Endpoint Catalogue in `eghl-api.md`); full field tables given for SALE/AUTH, payment response, QUERY, REFUND, RSALE, CAPTURE, SETTLE, SOPBL. SOP/SOP2/SOPCARD have hash orders only.
- Left out: Philippines test host (not Malaysia) beyond a mention; mobile SDKs (`cordova-plugin-eghl`, `react-native-eghl` on npm are published by individuals, not eGHL); GitHub accounts named eGHL* have no public repos.

## Corrections Made In The Top-Up (2026-10-21)

- `LanguageCode` examples `EN`, `MY`, `CN` were not in any official source. Replaced with what plugins send (`en`, or the Magento store locale's language part).
- SKILL.md said reuse of `OrderNumber` across attempts "is allowed" without a source. Reworded to what plugins do (resend per order) plus the WooCommerce rule that `OrderNumber` must not collide across stores.
- `Bill*`/`Ship*` rows merged types confusingly; split into one row per field with exact types (`BillCity`/`ShipCity` are ANS, not A).
- `Token` row: added that WooCommerce sends `TokenType=OCP` without `Token`, contrary to the validator rule.

## Not Covered By Public Official Sources

- Callback retry policy, callback source IPs, rate limits.
- Full error code list, sandbox test cards and bank credentials.
- Recurring, token charge, payment link, and settlement (`SETTLE`) API details.
- KYC document list, pricing, payout schedule.

Ask the eGHL/ADAPTIS account manager for the merchant API specification for these.
