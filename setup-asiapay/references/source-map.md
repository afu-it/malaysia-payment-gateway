# AsiaPay Source Map

Use this map to pick the right official source before answering or coding. Portal pages share the prefix `https://developer.asiapay.com/integration-guide`.

## Coverage

- Index used: the developer portal has no `llms.txt`, `llms-full.txt` or working `sitemap.xml` (all 404, checked 2026-10-21). It is a Theneo site; the full navigation tree (`documentMenu`) is embedded in each page's `__NEXT_DATA__`. That tree lists 235 pages under one project, "Paydollar PayGate Integration Guide" (v1).
- Pages enumerated: 235. Pages fetched and extracted: 235. All web, datafeed, secure hash, Merchant API, exceptional handling, appendix, FAQ, security, MPS and tokenization pages were read in full. Mobile SDK pages (Android, iOS, eWallet SDK steps) were read for API surface and settlement rules; SDK UI customization and native code samples are summarised, not copied.
- Supplementary official sources read: PayDollar PayGate Integration Guide PDF v3.67 (production URLs, datafeed IP ranges), Merchant API OnlineRefund PDF v1.2 (production onlineApi URL), Membership and MemberPay PDF v2.2, eWallet Direct PDF, and the official WooCommerce plugin source (hash code, multi-hash datafeed check).
- Left out on purpose: SiamPay/PesoPay specifics, Hong Kong-only methods (FPS, Octopus, PayMe, BoC Pay, PPS), China/Japan/Korea wallets, airline and OTA risk data details beyond field names, DCC/SCP/MCP screens. They do not apply to a typical Malaysian MYR merchant; field names are still listed in `asiapay-api.md` where relevant.

## Product And Market

- `https://www.asiapay.com.my/`
- `https://www.paydollar.com/en/contact-us.html`
- `/introduction`
- `/payment-methods-by-regions-countries`

Use for: AsiaPay Malaysia entity, contacts, sales, connection method overview, Malaysian payment methods.

## Hosted Payment Page (Client Post Through Browser)

- `/web-integration/client-post-through-browser`
- `/web-integration/client-post-through-browser/sample-code`
- `/web-integration/client-post-through-browser/integration-parameters`
- `/web-integration/client-post-through-browser/integration-parameters/required-parameters`
- `/web-integration/client-post-through-browser/integration-parameters/optional-parameters/*` (installment, airline-data, multi-currency-pricing-mcp, promotion, connect-to-payment-page, shipping-address, customer-details, ota-service-fulfillment, merchant-custom-data, schedule-payment, protect-pay)
- `/web-integration/client-post-through-browser/3ds-2-0/*` (basic-parameters, recurring-installment, billing-address, shipping-delivery, gift-card-prepaid-card, pre-order-purchase, account-info, account-authentication-info, pay-token, redirect-url)
- `https://www.paydollar.com/pdf/op/enpdintguide.pdf` (production `payForm.jsp` URL)

Use for: payForm.jsp fields, payMethod codes, currency and language codes, redirect URLs, 3DS 2.0 data.

## Direct Client Side And Server Side Direct

- `/web-integration/direct-client-side-connection` and its `sample-code`, `integration-parameters/required-parameters`, `integration-parameters/optional-parameters/*`, `3ds-2-0/*`
- `/web-integration/server-side-direct-connection` and its `sample-code`, `integration-parameters/required-parameter`, `integration-parameters/optional-parameters/*` (installment, airline-data, billing-information, connect-to-our-payment-interface, multi-currency-pricing-mcp, third-party-ewallet, card-data-encryption, cof-and-mit), `integration-parameters/3d-transaction`, `3ds-2-0/basic-parameters`, `3ds-2-0/return-parameter`
- `/mobile-integration/ewallet-direct-connection/server-side-direct-connection/*` and `/sdk-integration-step/*`
- `https://github.com/asiapay-lib/integration_guide` (eWallet Direct PDF)

Use for: payComp.jsp fields, pMethod values, synchronous response string, COF/MIT recurring flags, wallet `errMsg` payloads.

## Secure Hash

- `/transaction-security-by-secure-hash/introduction`
- `/transaction-security-by-secure-hash/client-library`
- `/transaction-security-by-secure-hash/manual-secure-hashing`
- `/transaction-security-by-secure-hash/enable-secure-hash`
- `/security/web-integration`
- `https://github.com/asiapay-lib/asiapay-Woocommerce` (`paydollar-payment-gateway-woocommerce-1.0.3/index.php`)

Use for: request and datafeed string orders, SHA-1/SHA-256, enabling, secret location and rotation, comma-separated datafeed hashes.

## Datafeed

- `/data-feed-handling`
- `/data-feed-handling/data-feed-setup`
- `/data-feed-handling/output-parameters/*` (data-feed-output, card-information, mps-vas, schedulepay-vas, memberpay-vas, third-party-ewallet, promopay-vas, installment, rewards-program, 3ds-2-0)
- `/data-feed-handling/sample-data-feed-page`
- `/data-feed-handling/data-feed-validation`
- `/faq/data-feed`
- `https://www.paydollar.com/pdf/op/enpdintguide.pdf` (FAQ 13, datafeed IP ranges)

Use for: datafeed fields, `OK` acknowledgement, ports, retry, validation, source IPs.

## Merchant API (Query, Capture, Void, Refund, Cancel, Reports)

- `/functions-of-merchant-api/introduction`
- `/functions-of-merchant-api/capture-authorized-payment/integration-parameters`
- `/functions-of-merchant-api/void-accepted-payment/integration-parameters`
- `/functions-of-merchant-api/request-refund-for-accepted-payment/integration-parameters`
- `/functions-of-merchant-api/query-payment-status/integration-parameters` and `/output-parameters/*`, `/sample`
- `/functions-of-merchant-api/query-request-log/integration-parameters`
- `/functions-of-merchant-api/cancel-payment/integration-parameters`
- `/functions-of-merchant-api/void-online-refund-or-partial-refund` and `/integration-parameters`, `/cases-for-void-refund`
- `/functions-of-merchant-api/settlement-report-request/definition-of-parameters-in-the-integration-page`
- `/functions-of-merchant-api/sample-source-code`
- `https://github.com/asiapay-lib/integration_guide` (Merchant API OnlineRefund PDF v1.2, production onlineApi URL)

Use for: status requery, captures, voids, refunds, partial refunds, cancel pending, settlement reconciliation.

## Tokens, MemberPay And Mobile

- `/mobile-integration/tokenization-flow/*` (get-started-with-paydollar-api, initiate-payment-token, paysdk-implementation, query-api, refund-api, status-code-and-error-code)
- `/mobile-integration/paysdk-android-mobile-sdk/*` and `/mobile-integration/paysdk-ios-mobile-sdk/*`
- `/security/mobile-sdk-integration`
- `https://github.com/asiapay-lib/integration_guide` (Membership and MemberPay PDF v2.2)
- `https://github.com/asiapay-lib/paysdk-android-lib`, `https://github.com/asiapay-lib/paysdk-ios-lib`

Use for: PaySDK REST API (JWE auth, payment token, query, refund), native SDK setup, MemberPay one-time and static tokens.

## Status, Codes, Testing And Failures

- `/appendix-a/payment-status-classification`
- `/appendix-a/paydollar-siampay-pesopay-payment-response-code`
- `/appendix-a/list-of-response-code/bank-s-response-code`
- `/appendix-a/list-of-response-code/response-code-from-paydollar-siampay-pesopay`
- `/appendix-a/list-of-response-code/other-response-code`
- `/appendix-a/list-of-country-code`
- `/appendix-a/3ds-testing-cards`
- `/appendix-a/payment-flow/*`, `/appendix-a/3d-transaction`, `/appendix-a/non-3d-transaction`
- `/exceptional-transaction-handling/*` (unsuccessful-redirection, unsuccessful-data-feed, incomplete-3d-authentication, incomplete-alternative-payment-transactions-by-customer)
- `/faq/common-problems`, `/faq/system-setup`, `/faq/3-d-secure-authentication`, `/faq/data-security`, `/faq/support`

Use for: orderStatus values, PRC/SRC meanings, sandbox cards, pending and retry behaviour, environment mismatch errors, support contacts.

## Multi-Currency

- `/multi-currency-processing-service/introduction`, `/simple-currency-conversion`, `/multi-currency-pricing`, `/dynamic-currency-conversion-dcc` and samples

Use for: SCP/MCP/DCC rules (rate expiry, rounding, tolerance). Not needed for MYR-only merchants.

## Security Configuration

- `/security/backend-configuration`

Use for: iframe domain enablement, API and server-side payment IP whitelisting.

## Plugins

- `https://github.com/asiapay-lib` (WooCommerce, Magento, OpenCart, PrestaShop, Shopify, Drupal and others)

Use for: no-code platform setup and reference implementations of hashing and datafeed handling.
