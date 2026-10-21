# Finexus Source Map

Official sources used for this skill, grouped by topic. All docs pages are under `https://sandbox.finexusgroup.com/myxaas/docs/` (paths below are relative to it). Checked 2026-10-21.

## General and onboarding (5 pages)

Use for: Onboarding steps, certificate import for gated modules, change log version, Google Pay through UPP (`PayMaster-MPI/googlepay`), gated OpenAPI spec and e-Invoice pages.

`change-log`, `einvoice/intro`, `get-started`, `openapi-spec`, `PayMaster-MPI/googlepay`

## PayMaster overview and notes (6 pages)

Use for: Product intro, rate limit (important notes), brand guidelines, secure payment link, DuitNow request page, Google Pay plugin download.

`paymaster/brand-guidelines`, `paymaster/duitnow-request`, `paymaster/important-notes`, `paymaster/intro`, `paymaster/plugin/download-plugin`, `paymaster/secure-payment-link`

## PayMaster API (raw HTTPS, no MPI) (12 pages)

Use for: Main source for field tables: UPP payment / query / reversal, DuitNow Pay bank listing / payment / query / refund, hash rules and test vectors (appendix1), bank codes (appendix2).

`paymaster/api/appendix/appendix1`, `paymaster/api/appendix/appendix2`, `paymaster/api/duitnow-pay/overview`, `paymaster/api/duitnow-pay/set-parameters/bank-listing`, `paymaster/api/duitnow-pay/set-parameters/payment`, `paymaster/api/duitnow-pay/set-parameters/query`, `paymaster/api/duitnow-pay/set-parameters/refund`, `paymaster/api/intro`, `paymaster/api/upp/overview`, `paymaster/api/upp/set-parameters/payment`, `paymaster/api/upp/set-parameters/query`, `paymaster/api/upp/set-parameters/reversal`

## PayMaster appendices (6 pages)

Use for: Response hash verification (MPI section), sample hash code, query status codes, transaction status codes, processing codes.

`paymaster/appendix/appendix1`, `paymaster/appendix/appendix2/generate`, `paymaster/appendix/appendix2/verify`, `paymaster/appendix/appendix3/query-status`, `paymaster/appendix/appendix3/transaction-status`, `paymaster/appendix/appendix4`

## UPP via MPI (9 pages)

Use for: UPP flow, MPI download links, properties files, embed steps for request / response / query, TxnStatDetCde 5015 check.

`paymaster/upp/configure-properties`, `paymaster/upp/download-plugin`, `paymaster/upp/embed-plugin/payment-query`, `paymaster/upp/embed-plugin/payment-request`, `paymaster/upp/embed-plugin/payment-response`, `paymaster/upp/embed-plugin/pre-requisites`, `paymaster/upp/overview`, `paymaster/upp/set-parameters/payment-parameters`, `paymaster/upp/set-parameters/query-parameters`

## UPP pre-auth (16 pages)

Use for: Pre-auth, completion, void and reversal properties, requests and responses.

`paymaster/upp/preauth-payment/configure-properties/completion-properties`, `paymaster/upp/preauth-payment/configure-properties/payment-properties`, `paymaster/upp/preauth-payment/request/completion-request`, `paymaster/upp/preauth-payment/request/payment-request`, `paymaster/upp/preauth-payment/response/completion-response`, `paymaster/upp/preauth-payment/response/payment-response`, `paymaster/upp/preauth-payment/reversal-payment/configure-properties/completion-properties`, `paymaster/upp/preauth-payment/reversal-payment/configure-properties/payment-properties`, `paymaster/upp/preauth-payment/reversal-payment/request/completion-request`, `paymaster/upp/preauth-payment/reversal-payment/request/payment-request`, `paymaster/upp/preauth-payment/reversal-payment/response/completion-response`, `paymaster/upp/preauth-payment/reversal-payment/response/payment-response`, `paymaster/upp/preauth-payment/reversal-payment/set-parameters/completion-parameters`, `paymaster/upp/preauth-payment/reversal-payment/set-parameters/payment-parameters`, `paymaster/upp/preauth-payment/set-parameters/completion-parameters`, `paymaster/upp/preauth-payment/set-parameters/payment-parameters`

## UPP recurring (4 pages)

Use for: Token based recurring charge (C04) fields and flow.

`paymaster/upp/recurring-payment/configure-properties`, `paymaster/upp/recurring-payment/payment-request`, `paymaster/upp/recurring-payment/payment-response`, `paymaster/upp/recurring-payment/set-parameters`

## UPP reversal (4 pages)

Use for: Reversal via MPI: properties, request, response, field rules.

`paymaster/upp/reversal-payment/configure-properties`, `paymaster/upp/reversal-payment/payment-request`, `paymaster/upp/reversal-payment/payment-response`, `paymaster/upp/reversal-payment/set-parameters`

## DuitNow Pay (OBW) via MPI (18 pages)

Use for: Online banking flow, bank listing, payment, query, reversal via MPI.

`paymaster/duitnow-pay`, `paymaster/duitnow-pay/obw/configure-properties/bank-listing`, `paymaster/duitnow-pay/obw/configure-properties/payment`, `paymaster/duitnow-pay/obw/configure-properties/query`, `paymaster/duitnow-pay/obw/embed-plugin/payment-query`, `paymaster/duitnow-pay/obw/embed-plugin/pre-requisites`, `paymaster/duitnow-pay/obw/request/bank-listing`, `paymaster/duitnow-pay/obw/request/payment`, `paymaster/duitnow-pay/obw/response/bank-listing`, `paymaster/duitnow-pay/obw/response/payment`, `paymaster/duitnow-pay/obw/reversal-payment/configure-properties`, `paymaster/duitnow-pay/obw/reversal-payment/payment-request`, `paymaster/duitnow-pay/obw/reversal-payment/payment-response`, `paymaster/duitnow-pay/obw/reversal-payment/set-parameters`, `paymaster/duitnow-pay/obw/set-parameters/bank-listing`, `paymaster/duitnow-pay/obw/set-parameters/payment`, `paymaster/duitnow-pay/obw/set-parameters/query`, `paymaster/duitnow-pay/overview`

## Dynamic QR (15 pages)

Use for: QR payload generation (0180), callback (0250), query, reversal, QR design rules and countdowns.

`paymaster/dynamic-qr/configure-properties/qrpayment-properties`, `paymaster/dynamic-qr/configure-properties/query-properties`, `paymaster/dynamic-qr/design-qr-code`, `paymaster/dynamic-qr/embed-plugin/payment-query`, `paymaster/dynamic-qr/embed-plugin/payment-request`, `paymaster/dynamic-qr/embed-plugin/payment-response`, `paymaster/dynamic-qr/embed-plugin/pre-requisites`, `paymaster/dynamic-qr/overview`, `paymaster/dynamic-qr/reversal-payment/configure-properties`, `paymaster/dynamic-qr/reversal-payment/payment-parameters`, `paymaster/dynamic-qr/reversal-payment/payment-request`, `paymaster/dynamic-qr/reversal-payment/payment-response`, `paymaster/dynamic-qr/set-parameters/callback-parameters`, `paymaster/dynamic-qr/set-parameters/payment-parameters`, `paymaster/dynamic-qr/set-parameters/query-parameters`

## Consumer presented QR (13 pages)

Use for: Merchant scans wallet QR (0240), query, reversal / refund.

`paymaster/cp-qr/configure-properties/qrpayment-properties`, `paymaster/cp-qr/configure-properties/query-properties`, `paymaster/cp-qr/embed-plugin/payment-query`, `paymaster/cp-qr/embed-plugin/payment-request`, `paymaster/cp-qr/embed-plugin/payment-response`, `paymaster/cp-qr/embed-plugin/pre-requisites`, `paymaster/cp-qr/overview`, `paymaster/cp-qr/reversal-payment/configure-properties`, `paymaster/cp-qr/reversal-payment/payment-parameters`, `paymaster/cp-qr/reversal-payment/payment-request`, `paymaster/cp-qr/reversal-payment/payment-response`, `paymaster/cp-qr/set-parameters/payment-parameters`, `paymaster/cp-qr/set-parameters/query-parameters`

## Static QR (3 pages)

Use for: Push notification JSON, RSA key exchange, merchant reply.

`paymaster/static-qr/key-exchange`, `paymaster/static-qr/overview`, `paymaster/static-qr/set-parameters`

## WooCommerce plugin (6 pages)

Use for: Install, configure, debug mode, recurring with Subscriptions extension. Plugin zip source reviewed for hashing and return handling.

`paymaster/woocommerce/configure-properties`, `paymaster/woocommerce/enable-debug-mode`, `paymaster/woocommerce/install-plugin`, `paymaster/woocommerce/overview`, `paymaster/woocommerce/recurring-payment/configure-properties`, `paymaster/woocommerce/recurring-payment/subscribe-extension`

## OpenCart plugin (4 pages)

Use for: Install, configure, transaction history.

`paymaster/opencart/install-oc-plugin`, `paymaster/opencart/oc-configure-properties`, `paymaster/opencart/overview`, `paymaster/opencart/transaction-history`

## Kayaaku POS App-to-App and intro (14 pages)

Use for: Kayaaku POS intro, Android Intent / service integration, sales / void / refund / completion / enquiry parameters.

`kayaakupos/app-to-app/embed-code`, `kayaakupos/app-to-app/overview`, `kayaakupos/app-to-app/payment/embed-code`, `kayaakupos/app-to-app/payment/set-parameters/completion-parameters`, `kayaakupos/app-to-app/payment/set-parameters/sales-parameters`, `kayaakupos/app-to-app/pre-requisites/device-connection`, `kayaakupos/app-to-app/pre-requisites/download-kpos`, `kayaakupos/app-to-app/reversal-payment/embed-code`, `kayaakupos/app-to-app/reversal-payment/set-parameters/refund-parameters`, `kayaakupos/app-to-app/reversal-payment/set-parameters/void-parameters`, `kayaakupos/app-to-app/set-parameters`, `kayaakupos/app-to-app/status-enquiry/embed-code`, `kayaakupos/app-to-app/status-enquiry/set-parameters`, `kayaakupos/intro`

## Kayaaku POS Bluetooth (11 pages)

Use for: Bluetooth ECR integration parameters.

`kayaakupos/bluetooth/embed-code`, `kayaakupos/bluetooth/overview`, `kayaakupos/bluetooth/pre-requisites/device-connection`, `kayaakupos/bluetooth/pre-requisites/download-kpos`, `kayaakupos/bluetooth/pre-requisites/enable-integration`, `kayaakupos/bluetooth/pre-requisites/key-exchange`, `kayaakupos/bluetooth/reversal-payment/embed-code`, `kayaakupos/bluetooth/reversal-payment/set-parameters/refund-parameters`, `kayaakupos/bluetooth/reversal-payment/set-parameters/void-parameters`, `kayaakupos/bluetooth/set-parameters`, `kayaakupos/bluetooth/status-enquiry`

## Kayaaku POS USB (13 pages)

Use for: USB serial integration (mPOS only).

`kayaakupos/usb/embed-code`, `kayaakupos/usb/overview`, `kayaakupos/usb/payment/embed-code`, `kayaakupos/usb/payment/set-parameters/completion-parameters`, `kayaakupos/usb/payment/set-parameters/sales-parameters`, `kayaakupos/usb/pre-requisites/device-connection`, `kayaakupos/usb/pre-requisites/download-kpos`, `kayaakupos/usb/pre-requisites/key-exchange`, `kayaakupos/usb/reversal-payment/embed-code`, `kayaakupos/usb/reversal-payment/set-parameters/refund-parameters`, `kayaakupos/usb/reversal-payment/set-parameters/void-parameters`, `kayaakupos/usb/set-parameters`, `kayaakupos/usb/status-enquiry`

## Kayaaku POS HTTP (13 pages)

Use for: HTTPS integration: enable setting, key exchange, JWE embed code, submit / cancel / refund / enquiry / notification / abort / print fields.

`kayaakupos/http/embed-code`, `kayaakupos/http/overview`, `kayaakupos/http/payment-parameters`, `kayaakupos/http/pre-requisites/enable-integration`, `kayaakupos/http/pre-requisites/key-exchange`, `kayaakupos/http/reversal-payment/embed-code`, `kayaakupos/http/reversal-payment/set-parameters/cancel-parameters`, `kayaakupos/http/reversal-payment/set-parameters/refund-parameters`, `kayaakupos/http/set-parameters/abort-parameters`, `kayaakupos/http/set-parameters/notification-parameters`, `kayaakupos/http/set-parameters/payment-parameters`, `kayaakupos/http/set-parameters/print-parameters`, `kayaakupos/http/status-enquiry`

## Kayaaku POS pre-auth HTTPS (14 pages)

Use for: thirdparty pre-auth, completion, void, refund, enquiry; RSA signedMessage.

`kayaakupos/preauth/card-payment/embed-code`, `kayaakupos/preauth/card-payment/reversal-payment/embed-code`, `kayaakupos/preauth/card-payment/reversal-payment/refund-parameters`, `kayaakupos/preauth/card-payment/reversal-payment/set-parameters/refund-parameters`, `kayaakupos/preauth/card-payment/reversal-payment/set-parameters/void-parameters`, `kayaakupos/preauth/card-payment/set-parameters/completion-parameters`, `kayaakupos/preauth/card-payment/set-parameters/payment-parameters`, `kayaakupos/preauth/overview`, `kayaakupos/preauth/pre-requisites/enable-integration`, `kayaakupos/preauth/pre-requisites/key-exchange`, `kayaakupos/preauth/qr-payment/embed-code`, `kayaakupos/preauth/qr-payment/set-parameters/payment-parameters`, `kayaakupos/preauth/qr-payment/set-parameters/refund-parameters`, `kayaakupos/preauth/status-enquiry`

## Kayaaku POS UCAT (40 pages)

Use for: Unattended terminals: HTTP, pre-auth HTTP, pre-auth EV HTTP, pre-auth serial cable.

`kayaakupos/ucat/http/embed-code`, `kayaakupos/ucat/http/overview`, `kayaakupos/ucat/http/pre-requisites/enable-integration`, `kayaakupos/ucat/http/pre-requisites/key-exchange`, `kayaakupos/ucat/http/reversal-payment/embed-code`, `kayaakupos/ucat/http/reversal-payment/set-parameters/cancel-parameters`, `kayaakupos/ucat/http/reversal-payment/set-parameters/refund-parameters`, `kayaakupos/ucat/http/set-parameters/abort-parameters`, `kayaakupos/ucat/http/set-parameters/notification-parameters`, `kayaakupos/ucat/http/set-parameters/payment-parameters`, `kayaakupos/ucat/http/status-enquiry`, `kayaakupos/ucat/preauth-ev/http/card-payment/embed-code`, `kayaakupos/ucat/preauth-ev/http/card-payment/set-parameters/cancel-parameters`, `kayaakupos/ucat/preauth-ev/http/card-payment/set-parameters/notification-parameters`, `kayaakupos/ucat/preauth-ev/http/card-payment/set-parameters/payment-parameters`, `kayaakupos/ucat/preauth-ev/http/overview`, `kayaakupos/ucat/preauth-ev/http/pre-requisites/enable-integration`, `kayaakupos/ucat/preauth-ev/http/pre-requisites/key-exchange`, `kayaakupos/ucat/preauth-ev/http/qr-payment/embed-code`, `kayaakupos/ucat/preauth-ev/http/qr-payment/set-parameters/notification-parameters`, `kayaakupos/ucat/preauth-ev/http/qr-payment/set-parameters/payment-parameters`, `kayaakupos/ucat/preauth/http/card-payment/embed-code`, `kayaakupos/ucat/preauth/http/card-payment/set-parameters/cancel-parameters`, `kayaakupos/ucat/preauth/http/card-payment/set-parameters/notification-parameters`, `kayaakupos/ucat/preauth/http/card-payment/set-parameters/payment-parameters`, `kayaakupos/ucat/preauth/http/overview`, `kayaakupos/ucat/preauth/http/pre-requisites/enable-integration`, `kayaakupos/ucat/preauth/http/pre-requisites/key-exchange`, `kayaakupos/ucat/preauth/http/qr-payment/embed-code`, `kayaakupos/ucat/preauth/http/qr-payment/set-parameters/notification-parameters`, `kayaakupos/ucat/preauth/http/qr-payment/set-parameters/payment-parameters`, `kayaakupos/ucat/preauth/serial-cable/card-payment/embed-code`, `kayaakupos/ucat/preauth/serial-cable/card-payment/set-parameters/notification-parameters`, `kayaakupos/ucat/preauth/serial-cable/card-payment/set-parameters/payment-parameters`, `kayaakupos/ucat/preauth/serial-cable/overview`, `kayaakupos/ucat/preauth/serial-cable/pre-requisites/enable-integration`, `kayaakupos/ucat/preauth/serial-cable/pre-requisites/key-exchange`, `kayaakupos/ucat/preauth/serial-cable/qr-payment/embed-code`, `kayaakupos/ucat/preauth/serial-cable/qr-payment/set-parameters/notification-parameters`, `kayaakupos/ucat/preauth/serial-cable/qr-payment/set-parameters/payment-parameters`

## Kayaaku POS appendices (15 pages)

Use for: Signature generate / verify samples, response codes, result codes, EC key generation, serial message format, e-invoice classification and state codes, FIDU app and QR.

`kayaakupos/appendix/appendix1/generate/enquiry`, `kayaakupos/appendix/appendix1/generate/http`, `kayaakupos/appendix/appendix1/generate/preauth`, `kayaakupos/appendix/appendix1/verify/enquiry`, `kayaakupos/appendix/appendix1/verify/http`, `kayaakupos/appendix/appendix1/verify/preauth`, `kayaakupos/appendix/appendix2`, `kayaakupos/appendix/appendix3`, `kayaakupos/appendix/appendix4`, `kayaakupos/appendix/appendix5`, `kayaakupos/appendix/appendix6`, `kayaakupos/appendix/appendix7`, `kayaakupos/appendix/appendix8/app`, `kayaakupos/appendix/appendix8/qr-generation/embed-code`, `kayaakupos/appendix/appendix8/qr-generation/set-parameters`

## Other official sources

Use for: product claims, onboarding form fields, MPI internals, plugin behaviour, SOAP query operations.

- Merchant payments page: https://www.finexusgroup.com/businesses/payments
- Application form: https://sandbox.finexusgroup.com/myxaas/form
- Docs sitemap (index used): https://sandbox.finexusgroup.com/myxaas/sitemap.xml
- PHP MPI 3.0.3 zip (PaymasterEntryPoint.json, UPP / Card / QR / FPX / BankListing schema JSON, .ini files): linked from `paymaster/upp/download-plugin`
- JavaScript MPI test kit 1.2.0 zip: linked from `paymaster/upp/download-plugin`
- WooCommerce plugin 4.0.5 zip (MessageHashing.php, return handler): Google Drive link on `paymaster/plugin/download-plugin`
- MPI SOAP WSDL: `https://dev.finexusgroup.com:10501/upp/services/Gateway?wsdl` (operations UPPPaymentQuery, PANProfileMaintenanceQuery, PANListQuery)
- MyDebit, Visa Instalment from RM 300, daily settlement claims: merchant payments page above.

## Coverage

- Enumerated: 241 docs pages from `/myxaas/sitemap.xml` (246 URLs in the sitemap; 241 are under `/docs/`, the other 5 are the portal home, application form, search and two grant (geran) marketing pages).
- Read: 241 of 241 pages fetched (all HTTP 200) and text extracted. 2 of them return only an access-denied shell: `openapi-spec` and `einvoice/intro` need a `.p12` client certificate issued after onboarding, so their content is not in this skill.
- Online payments (PayMaster, 117 pages) are fully covered in `finexus-api.md`.
- In-person Kayaaku POS (120 pages) is covered in `finexus-kayaaku-pos.md`. It is kept separate because it needs a Finexus terminal; most web integrations do not need it. The 40 UCAT pages repeat the HTTP and pre-auth field sets for unattended devices and are summarised, not copied field by field.
- Left out on purpose: the e-Invoice module (gated) and the browser certificate import screenshots for Chrome / Edge / Safari (no integration content).
- Known doc inconsistencies are flagged inline in the reference files (hash field order wording, 0290 hash example, 0780 version example, pre-auth completion URL, QR timeout 3 vs 5 minutes, Kayaaku PHP signing sample).
