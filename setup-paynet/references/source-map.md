# PayNet Source Map

All sources are official PayNet sites, checked 2026-10-21. Paths below are relative to `https://docs.paynet.my` unless a full URL is given.

## Index used

- New docs site `https://docs.paynet.my`: `sitemap.xml` (350 URLs) plus `llms.txt` and `llms-full.txt`. Current source of truth.
- Old docs site `https://docs.developer.paynet.my/docs` (the URL given in the task): `sitemap.xml` (378 URLs). The site says it is no longer being updated; used only to cross-check and for content missing on the new site.
- OpenAPI specs: 32 downloaded from the new site `api-reference` pages, 33 listed on the old site (`list.txt`).
- Company site `https://www.paynet.my` for product names and pricing link only.

## Key pages

- `/docs/fpx/b2c-overview`
  Use for: FPX B2C flow and message order.
- `/docs/fpx/authentication`
  Use for: FPX checksum: field order, RSA-SHA1, hex.
- `/docs/fpx/browser-redirection`
  Use for: FPX AR, AC, AE fields and status mapping.
- `/docs/fpx/key-management`
  Use for: FPX key and CSR generation, certificate request and renewal.
- `/docs/fpx/resources`
  Use for: FPX simulators, sample code, FPX certificates.
- `/docs/fpx/response-codes`
  Use for: FPX response codes.
- `/docs/fpx/bulk-refund`
  Use for: FPX bulk refund flow and B/C codes.
- `/docs/fpx/uat-test-cases`
  Use for: FPX UAT test cases, test banks, minimum amounts.
- `/docs/duitnow-pay/integration/api-authentication`
  Use for: DuitNow Pay JWT claims and signing.
- `/docs/duitnow-pay/integration/construct-header`
  Use for: DuitNow Pay required headers.
- `/docs/duitnow-pay/browser-redirection`
  Use for: Redirect parameters and bank URL paths.
- `/docs/duitnow-pay/response-codes`
  Use for: Superset of RPP reason, reject and D codes.
- `/docs/duitnow-pay/mapping-tables`
  Use for: DuitNow Pay enums and code tables.
- `/docs/duitnow-pay/registration/configure-system`
  Use for: BIC, CSR, webhook URL, IP range; merchant Templates A and B.
- `/docs/duitnow-pay/registration/go-live`
  Use for: Production request and public key.
- `/docs/online-banking-wallet/message-signature`
  Use for: RPP v3 JWS signing and verification, sample digest.
- `/docs/online-banking-wallet/construct-header`
  Use for: RPP v3 headers.
- `/docs/online-banking-wallet/repeat-handling`
  Use for: Duplicate and resend rules.
- `/docs/online-banking-wallet/store-and-forward`
  Use for: SAF rules.
- `/docs/duitnow-qr/merchant-presented-qr/qr-generation-specification/merchant-presented-mode`
  Use for: QR root tags and rules.
- `/docs/duitnow-qr/merchant-presented-qr/examples`
  Use for: Four official QR strings.
- `/docs/operations/financial-reports/financial-reports-api/responses`
  Use for: Reports API errors E0001 to E9999.
- `/docs/API/http-status-codes`
  Use for: Common HTTP codes and 429 body.
- `/docs/getting-started/create-project`
  Use for: Project creation and roles.

## All new-site pages by section

### Home (1 page)

Use for: Landing page and product list.

- `/`

### api-reference (26 pages)

Use for: OpenAPI reference page for the spec of the same name; field tables in the reference files were generated from these specs.

- `/api-reference/consumer-presented-qr/issuer`
- `/api-reference/reports/reports`
- `/api-reference/cross-border/acquirer`
- `/api-reference/reports/reportsv2`
- `/api-reference/online-banking-wallet/acquirer`
- `/api-reference/transfer/acquirer`
- `/api-reference/duitnow-qr/merchant-presented-qr/acquirer`
- `/api-reference/jompay-bvs`
- `/api-reference/merchant-profile`
- `/api-reference/fpx/bank-webview`
- `/api-reference/jompay-brs`
- `/api-reference/consumer-presented-qr/acquirer`
- `/api-reference/cross-border/issuer`
- `/api-reference/transfer/issuer`
- `/api-reference/duitnow-reversal/issuer`
- `/api-reference/jompay-bns`
- `/api-reference/tsp-acq`
- `/api-reference/tsp-issuer`
- `/api-reference/network-administration/acquirer`
- `/api-reference/online-banking-wallet/issuer`
- `/api-reference/duitnow-reversal/acquirer`
- `/api-reference/duitnow-qr/merchant-presented-qr/issuer`
- `/api-reference/national-addressing-database/issuer`
- `/api-reference/network-administration/issuer`
- `/api-reference/paynet-trust-intelligence-services-ptis/pre-transaction-screening-api`
- `/api-reference/paynet-trust-intelligence-services-ptis/id-trust-api`

### docs/duitnow-pay (36 pages)

Use for: DuitNow Pay acquirer API: auth, every endpoint, webhooks, enquiry rules, redirection, AutoDebit batch, codes, registration. Main input for paynet-duitnow-pay.md.

- `/docs/duitnow-pay/integration/paynet-hosted-page/payment-intent`
- `/docs/duitnow-pay/integration/paynet-hosted-page/save-payment-method`
- `/docs/duitnow-pay/integration/duitnow-autodebit`
- `/docs/duitnow-pay/overview`
- `/docs/duitnow-pay/response-codes`
- `/docs/duitnow-pay/integration/self-hosted-page/initiate-checkout`
- `/docs/duitnow-pay/integration/self-hosted-page/initiate-consent`
- `/docs/duitnow-pay/mapping-tables`
- `/docs/duitnow-pay/registration/go-live`
- `/docs/duitnow-pay/registration/configure-system`
- `/docs/duitnow-pay/integration/payment-status`
- `/docs/duitnow-pay/integration/checkout-details`
- `/docs/duitnow-pay/integration/payment-details`
- `/docs/duitnow-pay/integration/refund-status`
- `/docs/duitnow-pay/integration/terminate-consent`
- `/docs/duitnow-pay/integration/api-authentication`
- `/docs/duitnow-pay/integration/generate-key-pair`
- `/docs/duitnow-pay/integration/construct-header`
- `/docs/duitnow-pay/integration/payment-method-status`
- `/docs/duitnow-pay/registration/register-account`
- `/docs/duitnow-pay/user-journey`
- `/docs/duitnow-pay/integration/introduction`
- `/docs/duitnow-pay/integration/self-hosted-page/get-bank-list`
- `/docs/duitnow-pay/registration/create-project`
- `/docs/duitnow-pay/batchfilelist`
- `/docs/duitnow-pay/merchant-registration`
- `/docs/duitnow-pay/csr-renewal`
- `/docs/duitnow-pay/integration/initiate-refund`
- `/docs/duitnow-pay/registration/test-with-simulator`
- `/docs/duitnow-pay/browser-redirection`
- `/docs/duitnow-pay/registration/introduction`
- `/docs/duitnow-pay/registration/test-the-integration`
- `/docs/duitnow-pay/integration/duitnow-autodebit-batch`
- `/docs/duitnow-pay/branding-guidelines`
- `/docs/duitnow-pay/transaction-codes`
- `/docs/duitnow-pay/reportlist`

### docs/operations (17 pages)

Use for: Financial reports (API, OAuth, HMAC, file IDs), merchant profile, tech refresh. Input for paynet-api.md and paynet-codes.md.

- `/docs/operations/financial-reports/financial-reports-api/overview`
- `/docs/operations/financial-reports/overview`
- `/docs/operations/merchant-profile/overview`
- `/docs/operations/financial-reports/tech-refresh/introduction`
- `/docs/operations/financial-reports/financial-reports-api/authentication`
- `/docs/operations/fi-server`
- `/docs/operations/financial-reports/financial-reports-api/mydebit-specifications`
- `/docs/operations/merchant-profile/authentication`
- `/docs/operations/financial-reports/financial-reports-api/rpp-specifications`
- `/docs/operations/financial-reports/financial-reports-api/api-flow`
- `/docs/operations/financial-reports/financial-reports-api/responses`
- `/docs/operations/financial-reports/financial-reports-api/fpx-specifications`
- `/docs/operations/financial-reports/new-reports/introduction`
- `/docs/operations/reports/fpx`
- `/docs/operations/financial-reports/new-reports/itemised-report`
- `/docs/operations/financial-reports/new-reports/summary-report`
- `/docs/operations/reports/rpp`

### docs/network-administration (16 pages)

Use for: Sign on, sign off, echo, event notifications. Summarised in paynet-rpp-v3.md.

- `/docs/network-administration/overview`
- `/docs/network-administration/echo-request`
- `/docs/network-administration/mapping-tables`
- `/docs/network-administration/repeat-handling`
- `/docs/network-administration/construct-header`
- `/docs/network-administration/generate-key-pair`
- `/docs/network-administration/message-signature`
- `/docs/network-administration/store-and-forward`
- `/docs/network-administration/validations`
- `/docs/network-administration/response-codes`
- `/docs/network-administration/communication-protocol`
- `/docs/network-administration/user-guide-for-sample-app`
- `/docs/network-administration/connectivity-requirement`
- `/docs/network-administration/sign-on-and-sign-off-request`
- `/docs/network-administration/sdk`
- `/docs/network-administration/transaction-codes`

### docs/online-banking-wallet (17 pages)

Use for: RPP v3 shared rules: JWS signature, headers, BizMsgId, repeat handling, SAF, connectivity, validations, response codes. Main input for paynet-rpp-v3.md.

- `/docs/online-banking-wallet/overview`
- `/docs/online-banking-wallet/sdk`
- `/docs/online-banking-wallet/mapping-tables`
- `/docs/online-banking-wallet/repeat-handling`
- `/docs/online-banking-wallet/construct-header`
- `/docs/online-banking-wallet/generate-key-pair`
- `/docs/online-banking-wallet/message-signature`
- `/docs/online-banking-wallet/integration`
- `/docs/online-banking-wallet/batch-files-processing`
- `/docs/online-banking-wallet/store-and-forward`
- `/docs/online-banking-wallet/validations`
- `/docs/online-banking-wallet/branding-guidelines`
- `/docs/online-banking-wallet/response-codes`
- `/docs/online-banking-wallet/communication-protocol`
- `/docs/online-banking-wallet/user-guide-for-sample-app`
- `/docs/online-banking-wallet/connectivity-requirement`
- `/docs/online-banking-wallet/transaction-codes`

### docs/duitnow-reversal (17 pages)

Use for: Reversal 011/012 rules. Summarised in paynet-rpp-v3.md.

- `/docs/duitnow-reversal/overview`
- `/docs/duitnow-reversal/mapping-tables`
- `/docs/duitnow-reversal/repeat-handling`
- `/docs/duitnow-reversal/construct-header`
- `/docs/duitnow-reversal/generate-key-pair`
- `/docs/duitnow-reversal/message-signature`
- `/docs/duitnow-reversal/batch-files-processing`
- `/docs/duitnow-reversal/store-and-forward`
- `/docs/duitnow-reversal/validations`
- `/docs/duitnow-reversal/integration`
- `/docs/duitnow-reversal/branding-guidelines`
- `/docs/duitnow-reversal/response-codes`
- `/docs/duitnow-reversal/communication-protocol`
- `/docs/duitnow-reversal/user-guide-for-sample-app`
- `/docs/duitnow-reversal/connectivity-requirement`
- `/docs/duitnow-reversal/sdk`
- `/docs/duitnow-reversal/transaction-codes`

### docs/consent (21 pages)

Use for: DuitNow Consent registration flows and rules. Summarised in paynet-rpp-v3.md and paynet-duitnow-pay.md.

- `/docs/consent/overview`
- `/docs/consent/consent-registration-decoupled-request-to-pay`
- `/docs/consent/mapping-tables`
- `/docs/consent/repeat-handling`
- `/docs/consent/construct-header`
- `/docs/consent/generate-key-pair`
- `/docs/consent/message-signature`
- `/docs/consent/consent-registration-after-duitnow-request-autodebit-with-request-to-pay`
- `/docs/consent/consent-maintenance`
- `/docs/consent/consent-registration-via-issuer-flow`
- `/docs/consent/consent-registration-duitnow-pay-save-payment-flow`
- `/docs/consent/batch-files-processing`
- `/docs/consent/store-and-forward`
- `/docs/consent/validations`
- `/docs/consent/branding-guidelines`
- `/docs/consent/response-codes`
- `/docs/consent/communication-protocol`
- `/docs/consent/user-guide-for-sample-app`
- `/docs/consent/connectivity-requirement`
- `/docs/consent/sdk`
- `/docs/consent/transaction-codes`

### docs/national-addressing-database (20 pages)

Use for: NAD proxy registration (participant-only). Left out, see Coverage.

- `/docs/national-addressing-database/overview`
- `/docs/national-addressing-database/proxy-resolution-enquiry`
- `/docs/national-addressing-database/mapping-tables`
- `/docs/national-addressing-database/deregistration`
- `/docs/national-addressing-database/registration`
- `/docs/national-addressing-database/repeat-handling`
- `/docs/national-addressing-database/construct-header`
- `/docs/national-addressing-database/generate-key-pair`
- `/docs/national-addressing-database/message-signature`
- `/docs/national-addressing-database/modification`
- `/docs/national-addressing-database/suspend-or-activation`
- `/docs/national-addressing-database/store-and-forward`
- `/docs/national-addressing-database/validations`
- `/docs/national-addressing-database/transfer-bank`
- `/docs/national-addressing-database/response-codes`
- `/docs/national-addressing-database/communication-protocol`
- `/docs/national-addressing-database/user-guide-for-sample-app`
- `/docs/national-addressing-database/connectivity-requirement`
- `/docs/national-addressing-database/sdk`
- `/docs/national-addressing-database/transaction-codes`

### docs/duitnow-qr (24 pages)

Use for: DuitNow QR string spec (tags, CRC, examples, acquirer IDs), MPM/CPM flows, cross-border. Main input for paynet-duitnow-qr.md.

- `/docs/duitnow-qr/merchant-present-qr/overview`
- `/docs/duitnow-qr/consumer-presented-qr/overview`
- `/docs/duitnow-qr/response-codes`
- `/docs/duitnow-qr/sdk`
- `/docs/duitnow-qr/merchant-presented-qr/examples`
- `/docs/duitnow-qr/merchant-presented-qr/appendix-a`
- `/docs/duitnow-qr/mapping-tables`
- `/docs/duitnow-qr/repeat-handling`
- `/docs/duitnow-qr/merchant-presented-qr/merchant-presented-mode-domestic-qr`
- `/docs/duitnow-qr/merchant-presented-qr/cross-border-outbound-qr`
- `/docs/duitnow-qr/merchant-presented-qr/cross-border-inbound-qr`
- `/docs/duitnow-qr/construct-header`
- `/docs/duitnow-qr/generate-key-pair`
- `/docs/duitnow-qr/message-signature`
- `/docs/duitnow-qr/consumer-presented-qr/consumer-presented-mode-domestic-qr`
- `/docs/duitnow-qr/merchant-presented-qr/qr-generation-specification/merchant-presented-mode`
- `/docs/duitnow-qr/store-and-forward`
- `/docs/duitnow-qr/validations`
- `/docs/duitnow-qr/branding-guidelines`
- `/docs/duitnow-qr/communication-protocol`
- `/docs/duitnow-qr/merchant-presented-qr/qr-data-object`
- `/docs/duitnow-qr/user-guide-for-sample-app`
- `/docs/duitnow-qr/connectivity-requirement`
- `/docs/duitnow-qr/transaction-codes`

### docs/autodebit (17 pages)

Use for: DuitNow AutoDebit flows (871, 210, 230, 080). Summarised in paynet-rpp-v3.md.

- `/docs/autodebit/overview`
- `/docs/autodebit/mapping-tables`
- `/docs/autodebit/repeat-handling`
- `/docs/autodebit/construct-header`
- `/docs/autodebit/generate-key-pair`
- `/docs/autodebit/message-signature`
- `/docs/autodebit/integration`
- `/docs/autodebit/batch-files-processing`
- `/docs/autodebit/store-and-forward`
- `/docs/autodebit/validations`
- `/docs/autodebit/branding-guidelines`
- `/docs/autodebit/response-codes`
- `/docs/autodebit/communication-protocol`
- `/docs/autodebit/user-guide-for-sample-app`
- `/docs/autodebit/connectivity-requirement`
- `/docs/autodebit/sdk`
- `/docs/autodebit/transaction-codes`

### docs/request (17 pages)

Use for: DuitNow Request (851/853) flows for acquirers and issuers. Summarised in paynet-rpp-v3.md.

- `/docs/request/overview`
- `/docs/request/mapping-tables`
- `/docs/request/repeat-handling`
- `/docs/request/construct-header`
- `/docs/request/generate-key-pair`
- `/docs/request/message-signature`
- `/docs/request/batch-files-processing`
- `/docs/request/store-and-forward`
- `/docs/request/validations`
- `/docs/request/integration`
- `/docs/request/branding-guidelines`
- `/docs/request/response-codes`
- `/docs/request/communication-protocol`
- `/docs/request/user-guide-for-sample-app`
- `/docs/request/connectivity-requirement`
- `/docs/request/sdk`
- `/docs/request/transaction-codes`

### docs/transfer (20 pages)

Use for: DuitNow Transfer (010/110, proxy) issuer and acquirer flows. Summarised in paynet-rpp-v3.md; issuer side left out.

- `/docs/transfer/overview`
- `/docs/transfer/mapping-tables`
- `/docs/transfer/repeat-handling`
- `/docs/transfer/integration/credit-transfer`
- `/docs/transfer/integration/pay-by-proxy`
- `/docs/transfer/construct-header`
- `/docs/transfer/generate-key-pair`
- `/docs/transfer/message-signature`
- `/docs/transfer/integration/transaction-enquiry`
- `/docs/transfer/store-and-forward`
- `/docs/transfer/validations`
- `/docs/transfer/branding-guidelines`
- `/docs/transfer/integration/account-enquiry`
- `/docs/transfer/response-codes`
- `/docs/transfer/communication-protocol`
- `/docs/transfer/user-guide-for-sample-app`
- `/docs/transfer/integration/credit-transfer-reversal`
- `/docs/transfer/connectivity-requirement`
- `/docs/transfer/sdk`
- `/docs/transfer/transaction-codes`

### docs/san (19 pages)

Use for: SAN self-certification for switch migration (participant-only). Left out, see Coverage.

- `/docs/san/self-certification/request-support`
- `/docs/san/self-certification/manage-members`
- `/docs/san/self-certification/create-project`
- `/docs/san/self-certification/system-verification`
- `/docs/san/self-certification/test-execution/cash-withdrawal-acquirer`
- `/docs/san/self-certification/test-execution/cash-withdrawal-issuer`
- `/docs/san/self-certification/test-execution/ibcd-acquirer`
- `/docs/san/self-certification/test-execution/ibcd-issuer`
- `/docs/san/self-certification/test-execution/ibft-summary`
- `/docs/san/self-certification/test-execution/ibft-1-issuer`
- `/docs/san/self-certification/test-execution/ibft-1-acquirer`
- `/docs/san/self-certification/test-execution/ibft-2-issuer`
- `/docs/san/self-certification/test-execution/ibft-2-acq-ben`
- `/docs/san/self-certification/test-execution/ibft-3-acq-iss`
- `/docs/san/self-certification/test-execution/ibft-3-beneficiary`
- `/docs/san/self-certification/test-execution/ibft-4-acquirer`
- `/docs/san/self-certification/test-execution/ibft-4-issuer`
- `/docs/san/self-certification/test-execution/ibft-4-beneficiary`
- `/docs/san/self-certification/report-testing`

### docs/getting-started (16 pages)

Use for: Developer Portal account, projects, Certification Centre, System Verification, centralised certificates. Input for account-setup.md.

- `/docs/getting-started/centralised-configuration/mydebit/setup-in-project`
- `/docs/getting-started/centralised-configuration/mydebit/test-data`
- `/docs/getting-started/centralised-configuration/mydebit/simulator-configuration`
- `/docs/getting-started/centralised-configuration/duitnow-v3/centralised-certificate`
- `/docs/getting-started/create-account`
- `/docs/getting-started/centralised-configuration/duitnow-v3/apply-centralised-certificate`
- `/docs/getting-started/centralised-configuration/duitnow-v3/manage-and-renew-certificate`
- `/docs/getting-started/centralised-configuration/duitnow-pay/manage-and-renew-certificate`
- `/docs/getting-started/centralised-configuration/duitnow-pay/apply-centralised-certificate-in-project`
- `/docs/getting-started/certification-centre/project-guide`
- `/docs/getting-started/centralised-configuration/duitnow-pay/centralised-certificate`
- `/docs/getting-started/system-verification/project-guide`
- `/docs/getting-started/manage-project`
- `/docs/getting-started`
- `/docs/getting-started/create-project`
- `/docs/getting-started/certification-centre/troubleshooting`

### docs/certificate-management (10 pages)

Use for: Certificate lifecycle (PayNet-signed vs CA-signed, renewal, modulus). Input for account-setup.md and paynet-rpp-v3.md.

- `/docs/certificate-management/renewal-guide/paynet-signed/verify-certificate`
- `/docs/certificate-management/renewal-guide/additional-information/modulus-extraction`
- `/docs/certificate-management/getting-started`
- `/docs/certificate-management/renewal-guide/paynet-signed/introduction`
- `/docs/certificate-management/renewal-guide/paynet-signed/generate-key`
- `/docs/certificate-management/renewal-guide/paynet-signed/key-rotation`
- `/docs/certificate-management/renewal-guide/additional-information/generate-key`
- `/docs/certificate-management/renewal-guide/authorised-ca-signed/introduction`
- `/docs/certificate-management/renewal-guide/authorised-ca-signed/verify-and-complete-renewal`
- `/docs/certificate-management/renewal-guide/authorised-ca-signed/guide`

### docs/mydebit (12 pages)

Use for: MyDebit card scheme (card acquirers and issuers). Left out, see Coverage.

- `/docs/mydebit/overview/mydebit-secure-card-not-present`
- `/docs/mydebit`
- `/docs/mydebit/overview/mydebit-cnp-for-issuer`
- `/docs/mydebit/overview/mydebit-cnp-for-acquirer`
- `/docs/mydebit/overview/mydebit-tokenisation`
- `/docs/mydebit/resources/tokenisation`
- `/docs/mydebit/authentication`
- `/docs/mydebit/response-codes`
- `/docs/mydebit/glossary`
- `/docs/mydebit/branding-guidelines`
- `/docs/mydebit/overview/introduction`
- `/docs/mydebit/mydebit-card-not-present-and-tokenisation-testing-enhancements`

### docs/fpx (14 pages)

Use for: FPX direct integration (B2C, B2B1, B2B2), checksum, bank list, bulk refund, exception handling, codes, UAT tests, keys. Main input for paynet-fpx.md and paynet-codes.md.

- `/docs/fpx`
- `/docs/fpx/browser-redirection`
- `/docs/fpx/key-management`
- `/docs/fpx/b2b-2-model`
- `/docs/fpx/response-codes`
- `/docs/fpx/b2c-overview`
- `/docs/fpx/exception-handling`
- `/docs/fpx/mapping-table`
- `/docs/fpx/bulk-refund`
- `/docs/fpx/b2b-1-model`
- `/docs/fpx/authentication`
- `/docs/fpx/branding-guidelines`
- `/docs/fpx/uat-test-cases`
- `/docs/fpx/resources`

### docs/san-self-certification (2 pages)

Use for: SAN landing page. Left out, see Coverage.

- `/docs/san-self-certification`
- `/docs/san-self-certification/new-switch-migration`

### docs/jompay (9 pages)

Use for: JomPAY biller registration, notification and validation (bank-side). Endpoints listed in paynet-api.md; details left out.

- `/docs/jompay`
- `/docs/jompay/overview/biller-notification-system`
- `/docs/jompay/overview/biller-validation-module`
- `/docs/jompay/overview/biller-registration-system`
- `/docs/jompay/mapping-table`
- `/docs/jompay/authentication`
- `/docs/jompay/response-codes`
- `/docs/jompay/resources`
- `/docs/jompay/branding-guidelines`

### docs/API (6 pages)

Use for: Common API conventions: HTTP codes, rate limit, versioning, environments. Input for paynet-api.md.

- `/docs/API/webhooks`
- `/docs/API/API-specification`
- `/docs/API/versioning`
- `/docs/API/http-status-codes`
- `/docs/API/rate-limit`
- `/docs/API/ip-and-domains`

### docs/certification (2 pages)

Use for: Certification Centre and DAVE simulator concepts. Input for account-setup.md.

- `/docs/certification/dave-introduction`
- `/docs/certification/introduction`

### docs/duitnow (1 page)

Use for: DuitNow getting-started journey (PayNet account, eligibility). Input for account-setup.md.

- `/docs/duitnow/getting-started-with-duitnow`

### docs/financial-reports (1 page)

Use for: Financial reports landing page. Details in paynet-api.md.

- `/docs/financial-reports`

### docs/reversal (1 page)

Use for: Reversal landing page. Summarised in paynet-rpp-v3.md.

- `/docs/reversal/connectivity-requirement`

### docs/nfp (1 page)

Use for: National Fraud Portal onboarding (participant-only). Left out, see Coverage.

- `/docs/nfp/onboarding-guide`

### docs/paynet-trust-intelligence-services (4 pages)

Use for: PTIS fraud screening (participant-only). Endpoints listed in paynet-api.md.

- `/docs/paynet-trust-intelligence-services/resources/risk-signals-catalog`
- `/docs/paynet-trust-intelligence-services/resources/rule-catalog`
- `/docs/paynet-trust-intelligence-services/resources/transaction-codes`
- `/docs/paynet-trust-intelligence-services/introduction`

### docs/errorcode (1 page)

Use for: DuitNow error code page. Cross-check for paynet-codes.md.

- `/docs/errorcode/duitnow/error1`

### docs/qr-consumer-presented-mode (1 page)

Use for: QR CPM transaction and event code table (TCP/IP vs HTTP). Input for paynet-codes.md.

- `/docs/qr-consumer-presented-mode/transaction-codes`

### docs/qr-merchant-presented-mode (1 page)

Use for: QR MPM transaction and event code table (TCP/IP vs HTTP). Input for paynet-codes.md.

- `/docs/qr-merchant-presented-mode/transaction-codes`

## Old-site pages (docs.developer.paynet.my)

378 URLs enumerated from its sitemap, 377 extracted to text (one page returned no content). Sections and page counts: duitnow-pay 39, v3 22, duitNow-QR 22, duitNow-consent 20, san 19, national-addressing-database 19, duitnow-transfer 19, operations 18, duitNow-request 16, duitNow-online-banking-wallets 16, duitNow-AutoDebit 16, network-administration 15, duitNow 14, fpx 13, duitnow-reversal 13 (plus 3 under duitNow-reversal), mydebit 10, getting-started 10, certificate-management 9, jompay 8, certification 7, integration 6, API 6, duitnowbill 5, reports 4, mydebit-cnp 4, dbill-API-standards 2, and single pages (glossary, changelog, mappingtable, batchfile, authentication, brandingguidelines, merchant-profile, tsp-issuer, tsp-acq, nfp, jompay-bns/brs/bvs, fpx-webview-bank, mydebitoverview, san-self-certification).

Use for: cross-checking new-site content (FPX field lists and codes are the same on both), the FPX PHP sample code and signer tool in `/docs/fpx/resources` (read, not executed), XML-signature variants of RPP services, and the DuitNow Bill (duitnowbill, dbill-API-standards) pages that the new site no longer lists. When the two sites differ, the new site wins.

## OpenAPI specs

New site (32): Bank_Webview, Biller_Notification, Biller_Registration, Biller_Validation, Consumer_Presented_Mode_Domestic_QR_Acquirer/Issuer, Cross_Border_Inbound_QR_Acquirer, Cross_Border_Outbound_QR_Issuer, DuitNow_Auto_Debit_Acquirer/Issuer, DuitNow_Consent_Acquirer/Issuer, DuitNow_Online_Banking_Acquirer/Issuer, DuitNow_Request_Acquirer/Issuer, DuitNow_Reversal_Acquirer/Issuer, DuitNow_Transfer_Acquirer/Issuer, Financial_Reports_Download (NextSwitch), Financial_Reports_Download_v2 (PayNet Pricing Initiative), ID_Trust_API, Merchant_Presented_Mode_Domestic_QR_Acquirer/Issuer, Merchant_Profile_API, National_Addressing_Database_NAD_Issuer, Network_Administration_Acquirer/Issuer, Pre_Transaction_Screening_PTS_API, Tokenisation_for_Acquirer_Merchant, Tokenisation_for_Issuer.

Old site (33): the v3 DuitNow set (22, including NAD Acquirer, which failed to parse), fpx-bank, jompay biller-notification/registration/validation, mssot merchants, nfp-main, reports, reportsv2, token, tsp-issuer, tsp-merchant.

Use for: every endpoint in the `paynet-api.md` catalogue and the generated field tables in `paynet-rpp-v3.md` and `paynet-duitnow-qr.md`.

## Coverage

- Enumerated: 728 docs URLs (350 new, 378 old) and 65 OpenAPI spec files. Text extracted for 727 pages; all 32 new-site specs and 32 of 33 old-site specs parsed.
- Read in full and used for field-level content: all fpx, duitnow-pay, duitnow-qr, online-banking-wallet, getting-started, certification, API and operations/financial-reports pages, plus the acquirer specs for Online Banking, Request, Reversal, AutoDebit, Consent, MPM QR and Network Administration.
- Read and summarised (flows and endpoint names only): request, consent, autodebit, duitnow-reversal, transfer, network-administration, certificate-management, jompay, paynet-trust-intelligence-services, and the issuer, CPM, cross-border, tokenisation, NAD, PTS and ID Trust specs.
- Left out on purpose, with reason:
  - SAN self-certification and switch migration: only for existing switch participants moving to the new switch.
  - National Addressing Database (NAD) and proxy registration: issuer and bank-side, not merchant acceptance.
  - Issuer-side APIs (debiting agent webhooks, TSP issuer tokenisation, cross-border outbound): the skill targets accepting payments.
  - MyDebit and MyDebit card-not-present: a card scheme for card acquirers; separate onboarding.
  - JomPAY biller-side bank APIs, PTIS, PTS, ID Trust and National Fraud Portal: participant-only services; endpoints are listed in the catalogue without field tables.
  - DuitNow Bill old-site pages: not listed on the current site.
- Not downloaded: binary attachments (Figma links, logo packs, xls test files other than the FPX UAT sheet, sample private keys). Sample keys were not copied.
