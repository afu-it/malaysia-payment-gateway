# 2C2P Source Map

Use this map to pick the right official page before answering or coding. All pages are on `https://developer.2c2p.com` (append `.md` to any docs URL for the raw Markdown). Checked 2026-10-21.

Which reference file covers which group:

| Group | Reference file |
| --- | --- |
| Getting started, Redirect API, notifications, inquiry | `2c2p-api.md` |
| Direct API, web front end, OpenAPI | `2c2p-direct-api.md` |
| Maintenance, tokens, recurring, keys | `2c2p-maintenance.md` |
| Codes and reference data | `2c2p-codes.md` |
| QuickPay, Payout | `2c2p-quickpay-payout.md` |
| Batch, reconciliation, SoftPOS, plugins, SNAP | `2c2p-batch-recon.md` |
| Mobile SDK | `2c2p-mobile-sdk.md` |
| Merchant onboarding, keys, pricing | `account-setup.md` |

## Coverage

- Index used: `https://developer.2c2p.com/llms.txt` (225 page URLs).
- Pages enumerated: 339 unique URLs (225 from the index plus 114 found by following links inside fetched pages, including versioned and hidden pages).
- Pages retrieved and read: 317 (all 225 index pages plus 92 linked pages). Pages central to Malaysian online acceptance (environments, JWT, Payment Token, notifications, inquiry, Direct API, maintenance, codes, test data, QuickPay, payout, batch, SDK) were read in full. Global-only pages were read for scope and summarised briefly.
- Not retrievable (HTTP 404 on 2026-10-21, 22 URLs): `docs/2c2p`, `docs/apptoapp`, `docs/batch-services-reconcile-report-full-payment`, `docs/credit-cards`, `docs/how-it-works-1-2`, `docs/how-to-integrate-1`, `docs/payment-maintenance-ipp-guide`, `docs/prepare-payment-request`, `docs/reference-codes-payment-action-result-code`, `docs/reference-codes-payment-action-status-code`, `docs/reference-payment-channel-matrix`, `recipes/using-android-webview`, `reference/android`, `reference/com`, `reference/payment`, `reference/paymentinquiry`, `reference/paymentoption`, `reference/paymentoptiondetails`, `reference/paymenttoken`, `reference/request-objects`, `reference/response-objects`, `reference/transactionstatus`. The result and status code lists exist under `docs/response-code-payment-maintenance-*` and `docs/payment-action-references`, which were read.
- Left out or summarised only, and why:
  - SNAP API (Indonesia, Bank Indonesia standard): not usable for MYR; endpoint list kept so agents do not confuse it with a Malaysian flow.
  - Non-Malaysian methods and test data (Thailand, Singapore, Philippines, Hong Kong, Myanmar, Vietnam, Indonesia wallets, banks, counters, ATM, PayNow, PromptPay, Click2Pay): out of scope for Malaysian merchants; Malaysian equivalents are documented.
  - Airline passenger data, loyalty point redemption, MCP / DCC FX details, sub-merchant (master merchant) splits: niche; field names are listed but not expanded.
  - SoftPOS SDK method-level details (in-person Android tap on phone): summarised; the online payment skill does not need them.
  - Promotional and marketing pages outside the developer portal: not developer docs.
- Pricing: no Malaysian rate card is published; see `account-setup.md`.

## Getting Started, Environments, Sandbox, JWT

- `https://developer.2c2p.com/docs/general`
- `https://developer.2c2p.com/docs/json-web-tokens-jwt`
- `https://developer.2c2p.com/docs/reference-environment-guide`
- `https://developer.2c2p.com/docs/sandbox`
- `https://developer.2c2p.com/docs/sandbox-setup`
- `https://developer.2c2p.com/recipes/prepare-request-payload-with-jwt`
- `https://developer.2c2p.com/recipes/process-response-payload-with-jwt`

Use for: environments, base URLs, demo merchants and Secret Keys (sandbox only), JWT HS256 signing and decoding, go-live checklist.

## Redirect API, Payment Token, Notifications, Payment Inquiry

- `https://developer.2c2p.com/docs/api-payment-inquiry`
- `https://developer.2c2p.com/docs/api-payment-inquiry-request-parameter`
- `https://developer.2c2p.com/docs/api-payment-inquiry-response-parameter`
- `https://developer.2c2p.com/docs/api-payment-response-back-end-parameter`
- `https://developer.2c2p.com/docs/api-payment-response-backend`
- `https://developer.2c2p.com/docs/api-payment-response-front-end-parameter`
- `https://developer.2c2p.com/docs/api-payment-response-frontend`
- `https://developer.2c2p.com/docs/api-payment-token`
- `https://developer.2c2p.com/docs/api-payment-token-request-parameter`
- `https://developer.2c2p.com/docs/api-payment-token-response-parameter`
- `https://developer.2c2p.com/docs/redirect-api-how-it-works`
- `https://developer.2c2p.com/docs/redirect-api-integrate-with-payment`
- `https://developer.2c2p.com/docs/using-iframe`

Use for: hosted checkout flow, Payment Token request and response fields, backend and frontend notification formats, Payment Inquiry, iframe mode.

## Direct API And Web Front End

- `https://developer.2c2p.com/docs/api-card-installment-plan-info`
- `https://developer.2c2p.com/docs/api-card-installment-plan-info-request-parameter`
- `https://developer.2c2p.com/docs/api-card-installment-plan-info-response-parameter`
- `https://developer.2c2p.com/docs/api-card-tokens-information-response-parameter`
- `https://developer.2c2p.com/docs/api-do-payment`
- `https://developer.2c2p.com/docs/api-do-payment-request-parameter`
- `https://developer.2c2p.com/docs/api-do-payment-response-parameter`
- `https://developer.2c2p.com/docs/api-payment-option`
- `https://developer.2c2p.com/docs/api-payment-option-details`
- `https://developer.2c2p.com/docs/api-payment-option-details-request-parameter`
- `https://developer.2c2p.com/docs/api-payment-option-details-response-parameter`
- `https://developer.2c2p.com/docs/api-payment-option-request-parameter`
- `https://developer.2c2p.com/docs/api-payment-option-response-parameter`
- `https://developer.2c2p.com/docs/api-transaction-status-inquiry`
- `https://developer.2c2p.com/docs/api-transaction-status-inquiry-request-parameter`
- `https://developer.2c2p.com/docs/api-transaction-status-inquiry-response-parameter`
- `https://developer.2c2p.com/docs/direct-api-apple-pay`
- `https://developer.2c2p.com/docs/direct-api-flow-offline-payment`
- `https://developer.2c2p.com/docs/direct-api-flow-scan-qr`
- `https://developer.2c2p.com/docs/direct-api-flow-server-to-server`
- `https://developer.2c2p.com/docs/direct-api-flow-third-party-redirection`
- `https://developer.2c2p.com/docs/direct-api-how-it-works`
- `https://developer.2c2p.com/docs/direct-api-method-3ds-card-payment`
- `https://developer.2c2p.com/docs/direct-api-method-click2pay`
- `https://developer.2c2p.com/docs/direct-api-method-digital-payment-wallet`
- `https://developer.2c2p.com/docs/direct-api-method-internet-mobile-banking`
- `https://developer.2c2p.com/docs/direct-api-method-non-3ds-card-payment`
- `https://developer.2c2p.com/docs/direct-api-method-pay-at-counter`
- `https://developer.2c2p.com/docs/direct-api-method-qr-payment`
- `https://developer.2c2p.com/docs/direct-api-method-self-service-machines`
- `https://developer.2c2p.com/docs/direct-api-method-web-payment`
- `https://developer.2c2p.com/docs/direct-api-payment-methods`
- `https://developer.2c2p.com/docs/encryption-of-card-data-information`
- `https://developer.2c2p.com/docs/google-pay`
- `https://developer.2c2p.com/docs/loyalty-point-info-api`
- `https://developer.2c2p.com/docs/loyalty-point-info-request-parameters`
- `https://developer.2c2p.com/docs/loyalty-point-info-response-parameters`
- `https://developer.2c2p.com/docs/reference-secure-pay-js-lib`
- `https://developer.2c2p.com/docs/using-secure-pay-javascript-library`
- `https://developer.2c2p.com/docs/using-securefields`
- `https://developer.2c2p.com/docs/web-sdk-drop-in-ui`

Use for: own checkout UI, Payment Option and Option Details, Do Payment, Transaction Status, Secure Pay JS card encryption, Drop-In UI, Apple Pay, Google Pay, Click2Pay, per-method guides.

## OpenAPI Reference (PGW Payment API 4.3)

- `https://developer.2c2p.com/reference/post_payment-4-3-applepay-merchantvalidation`
- `https://developer.2c2p.com/reference/post_payment-4-3-canceltransaction`
- `https://developer.2c2p.com/reference/post_payment-4-3-cardinstallmentplaninfo`
- `https://developer.2c2p.com/reference/post_payment-4-3-cardtokeninfo`
- `https://developer.2c2p.com/reference/post_payment-4-3-exchangerate`
- `https://developer.2c2p.com/reference/post_payment-4-3-exchangerate-apmmccexchangerate`
- `https://developer.2c2p.com/reference/post_payment-4-3-exchangerate-secure`
- `https://developer.2c2p.com/reference/post_payment-4-3-initialization`
- `https://developer.2c2p.com/reference/post_payment-4-3-loyaltypointinfo`
- `https://developer.2c2p.com/reference/post_payment-4-3-payment`
- `https://developer.2c2p.com/reference/post_payment-4-3-paymentinquiry`
- `https://developer.2c2p.com/reference/post_payment-4-3-paymentinstruction`
- `https://developer.2c2p.com/reference/post_payment-4-3-paymentnotification`
- `https://developer.2c2p.com/reference/post_payment-4-3-paymentoption`
- `https://developer.2c2p.com/reference/post_payment-4-3-paymentoptiondetails`
- `https://developer.2c2p.com/reference/post_payment-4-3-paymentsimulate`
- `https://developer.2c2p.com/reference/post_payment-4-3-paymenttoken`
- `https://developer.2c2p.com/reference/post_payment-4-3-transactionstatus`
- `https://developer.2c2p.com/reference/post_payment-4-3-userpreference`

Use for: exact 4.3 paths and schemas for every PGW operation, including operations with no narrative page.

## Payment Maintenance, Tokens, Recurring, Keys

- `https://developer.2c2p.com/docs/api-parameters-2`
- `https://developer.2c2p.com/docs/api-payment-action-fx-rate-inquiry`
- `https://developer.2c2p.com/docs/api-payment-action-payment-process`
- `https://developer.2c2p.com/docs/api-payment-action-recurring-maintenance`
- `https://developer.2c2p.com/docs/api-payment-action-store-card-maintenance`
- `https://developer.2c2p.com/docs/certificate-generation-guide`
- `https://developer.2c2p.com/docs/customer-token-maintenance`
- `https://developer.2c2p.com/docs/customer-token-maintenance-api`
- `https://developer.2c2p.com/docs/payment-action-references`
- `https://developer.2c2p.com/docs/payment-maintenance-card-token-guide`
- `https://developer.2c2p.com/docs/payment-maintenance-fx-rate-inquiry-guide`
- `https://developer.2c2p.com/docs/payment-maintenance-how-it-works`
- `https://developer.2c2p.com/docs/payment-maintenance-inquiry-guide`
- `https://developer.2c2p.com/docs/payment-maintenance-recurring-payment-guide`
- `https://developer.2c2p.com/docs/payment-maintenance-refund-guide`
- `https://developer.2c2p.com/docs/payment-maintenance-refund-status-guide`
- `https://developer.2c2p.com/docs/payment-maintenance-settle-guide`
- `https://developer.2c2p.com/docs/payment-maintenance-void-guide`
- `https://developer.2c2p.com/docs/recurring-payment-plan-maintenance`
- `https://developer.2c2p.com/docs/reference-card-on-file-cit-mit`
- `https://developer.2c2p.com/docs/reference-convert-existing-transactions-to-card-on-file-model`
- `https://developer.2c2p.com/docs/reference-jwt-with-key`
- `https://developer.2c2p.com/recipes/prepare-request-payload-with-jwt-jws-with-keys`
- `https://developer.2c2p.com/recipes/process-response-payload-with-jwt-jws-with-keys`
- `https://developer.2c2p.com/docs/installment-payment-plan-options-inquiry`

Use for: refund, void, settle, maintenance inquiry, refund status, refund notification, JWE + JWS keys and certificates, recurring plan maintenance, card and customer token maintenance, FX rate inquiry, card-on-file CIT / MIT, legacy Payment Action and IPP option inquiry.

## Codes And Reference Data

- `https://developer.2c2p.com/docs/reference-agent-code-update-for-fpx-payment`
- `https://developer.2c2p.com/docs/reference-codes-account-funding-account-identifier-type`
- `https://developer.2c2p.com/docs/reference-codes-account-funding-identification-type`
- `https://developer.2c2p.com/docs/reference-codes-account-funding-purpose-code`
- `https://developer.2c2p.com/docs/reference-codes-agent-code`
- `https://developer.2c2p.com/docs/reference-codes-currency`
- `https://developer.2c2p.com/docs/reference-codes-payment-scheme`
- `https://developer.2c2p.com/docs/reference-payment-channels`
- `https://developer.2c2p.com/docs/reference-payment-channels-cut-off-time`
- `https://developer.2c2p.com/docs/reference-test-information-hk`
- `https://developer.2c2p.com/docs/reference-test-information-my`
- `https://developer.2c2p.com/docs/reference-test-information-ph`
- `https://developer.2c2p.com/docs/reference-test-information-sg`
- `https://developer.2c2p.com/docs/reference-test-information-th`
- `https://developer.2c2p.com/docs/reference-testing-information`
- `https://developer.2c2p.com/docs/response-code-card-iso-8583`
- `https://developer.2c2p.com/docs/response-code-payment`
- `https://developer.2c2p.com/docs/response-code-payment-flow`
- `https://developer.2c2p.com/docs/response-code-payment-maintenance-result-code`
- `https://developer.2c2p.com/docs/response-code-payment-maintenance-status-code`

Use for: payment, flow, maintenance result and status codes, payment channels, FPX agent codes and migration, agent and scheme codes, currency codes, cut-off times, test cards.

## QuickPay Payment Links

- `https://developer.2c2p.com/docs/api-quickpay-delete`
- `https://developer.2c2p.com/docs/api-quickpay-delete-request-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-delete-response-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-generate-and-send-link`
- `https://developer.2c2p.com/docs/api-quickpay-generate-and-send-link-request-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-generate-and-send-link-response-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-generate-link`
- `https://developer.2c2p.com/docs/api-quickpay-generate-link-request-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-generate-link-response-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-query`
- `https://developer.2c2p.com/docs/api-quickpay-query-request-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-query-response-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-send-link`
- `https://developer.2c2p.com/docs/api-quickpay-send-link-request-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-send-link-response-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-update`
- `https://developer.2c2p.com/docs/api-quickpay-update-request-parameter`
- `https://developer.2c2p.com/docs/api-quickpay-update-response-parameter`
- `https://developer.2c2p.com/docs/quickpay-how-it-works`
- `https://developer.2c2p.com/docs/response-code-quickpay`

Use for: QuickPay generate, query, update, delete, generate and send, send, HMAC-SHA1 hash orders, QuickPay response codes.

## Payout

- `https://developer.2c2p.com/docs/beneficiary-inquiry`
- `https://developer.2c2p.com/docs/beneficiary-inquiry-request-parameter`
- `https://developer.2c2p.com/docs/beneficiary-inquiry-response-parameter`
- `https://developer.2c2p.com/docs/beneficiary-registration`
- `https://developer.2c2p.com/docs/beneficiary-registration-request-parameter`
- `https://developer.2c2p.com/docs/beneficiary-registration-response-parameters`
- `https://developer.2c2p.com/docs/create-payout`
- `https://developer.2c2p.com/docs/payout-bank-code`
- `https://developer.2c2p.com/docs/payout-how-it-works`
- `https://developer.2c2p.com/docs/payout-inquiry`
- `https://developer.2c2p.com/docs/payout-inquiry-request-parameter`
- `https://developer.2c2p.com/docs/payout-inquiry-response-parameter`
- `https://developer.2c2p.com/docs/payout-preferred-provider`
- `https://developer.2c2p.com/docs/payout-request-parameter`
- `https://developer.2c2p.com/docs/payout-response-code`
- `https://developer.2c2p.com/docs/payout-response-notification`
- `https://developer.2c2p.com/docs/payout-source-of-fund`
- `https://developer.2c2p.com/docs/preferred-provider-request-parameter`
- `https://developer.2c2p.com/docs/preferred-provider-response-parameter`
- `https://developer.2c2p.com/docs/sof-list-request-parameter`
- `https://developer.2c2p.com/docs/sof-list-response-parameter`

Use for: payout create and inquiry, beneficiary registration and inquiry, providers, source of fund, payout notification, payout bank codes, payout codes.

## Batch Services And Reconciliation

- `https://developer.2c2p.com/docs/batch-services-authorization`
- `https://developer.2c2p.com/docs/batch-services-authorization-offline`
- `https://developer.2c2p.com/docs/batch-services-card-tokenization`
- `https://developer.2c2p.com/docs/batch-services-promo-code`
- `https://developer.2c2p.com/docs/batch-services-quickpay`
- `https://developer.2c2p.com/docs/batch-services-reconcile-report-info`
- `https://developer.2c2p.com/docs/batch-services-reconcile-report-ipp`
- `https://developer.2c2p.com/docs/batch-services-reconcile-report-non-ipp`
- `https://developer.2c2p.com/docs/batch-services-refund`

Use for: SFTP connection, batch file formats and schedules, reconciliation report columns and versions.

## Mobile SDK

- `https://developer.2c2p.com/docs/api-sdk-transaction-status-inquiry`
- `https://developer.2c2p.com/docs/sdk-api-cancel-transaction`
- `https://developer.2c2p.com/docs/sdk-api-customer-tokens-information`
- `https://developer.2c2p.com/docs/sdk-api-do-payment`
- `https://developer.2c2p.com/docs/sdk-api-exchange-rate`
- `https://developer.2c2p.com/docs/sdk-api-loyalty-point-information`
- `https://developer.2c2p.com/docs/sdk-api-payment-notification`
- `https://developer.2c2p.com/docs/sdk-api-payment-option`
- `https://developer.2c2p.com/docs/sdk-api-payment-option-details`
- `https://developer.2c2p.com/docs/sdk-api-payment-ui`
- `https://developer.2c2p.com/docs/sdk-api-pgw-initialization`
- `https://developer.2c2p.com/docs/sdk-api-user-preference`
- `https://developer.2c2p.com/docs/sdk-card-loyalty-point-payment`
- `https://developer.2c2p.com/docs/sdk-classes-apis-interface`
- `https://developer.2c2p.com/docs/sdk-classes-payment-enums`
- `https://developer.2c2p.com/docs/sdk-classes-payment-info`
- `https://developer.2c2p.com/docs/sdk-classes-payment-request`
- `https://developer.2c2p.com/docs/sdk-customer-tokenization`
- `https://developer.2c2p.com/docs/sdk-customer-tokenization-without-authorization`
- `https://developer.2c2p.com/docs/sdk-deep-link-payment`
- `https://developer.2c2p.com/docs/sdk-download-sdk`
- `https://developer.2c2p.com/docs/sdk-download-sdk-flutter`
- `https://developer.2c2p.com/docs/sdk-download-sdk-helper`
- `https://developer.2c2p.com/docs/sdk-download-sdk-react-native`
- `https://developer.2c2p.com/docs/sdk-download-securepay`
- `https://developer.2c2p.com/docs/sdk-download-ui-sdk`
- `https://developer.2c2p.com/docs/sdk-e-wallet-tokenization-without-authorization`
- `https://developer.2c2p.com/docs/sdk-handle-pgw-payment-authentication`
- `https://developer.2c2p.com/docs/sdk-helper-apis-interface`
- `https://developer.2c2p.com/docs/sdk-helper-payment-enums`
- `https://developer.2c2p.com/docs/sdk-helper-payment-info-classes`
- `https://developer.2c2p.com/docs/sdk-helper-payment-request`
- `https://developer.2c2p.com/docs/sdk-how-it-work`
- `https://developer.2c2p.com/docs/sdk-import-sdk`
- `https://developer.2c2p.com/docs/sdk-initial-sdk`
- `https://developer.2c2p.com/docs/sdk-installment-payment-plan`
- `https://developer.2c2p.com/docs/sdk-installment-payment-plan-local`
- `https://developer.2c2p.com/docs/sdk-method-alipay`
- `https://developer.2c2p.com/docs/sdk-method-apple-pay`
- `https://developer.2c2p.com/docs/sdk-method-boost-wallet`
- `https://developer.2c2p.com/docs/sdk-method-buy-now-pay-later`
- `https://developer.2c2p.com/docs/sdk-method-card-payments`
- `https://developer.2c2p.com/docs/sdk-method-cb-pay`
- `https://developer.2c2p.com/docs/sdk-method-credit-debit-card`
- `https://developer.2c2p.com/docs/sdk-method-credit-debit-card-local`
- `https://developer.2c2p.com/docs/sdk-method-gcash`
- `https://developer.2c2p.com/docs/sdk-method-google-pay`
- `https://developer.2c2p.com/docs/sdk-method-grab-pay`
- `https://developer.2c2p.com/docs/sdk-method-internet-banking`
- `https://developer.2c2p.com/docs/sdk-method-kbzpay`
- `https://developer.2c2p.com/docs/sdk-method-line-pay`
- `https://developer.2c2p.com/docs/sdk-method-linkaja`
- `https://developer.2c2p.com/docs/sdk-method-m-pitesan`
- `https://developer.2c2p.com/docs/sdk-method-master-pass`
- `https://developer.2c2p.com/docs/sdk-method-momo-wallet`
- `https://developer.2c2p.com/docs/sdk-method-ok-dollar-wallet`
- `https://developer.2c2p.com/docs/sdk-method-ovo`
- `https://developer.2c2p.com/docs/sdk-method-pay-at-counter`
- `https://developer.2c2p.com/docs/sdk-method-paypal-wallet`
- `https://developer.2c2p.com/docs/sdk-method-qr-payment`
- `https://developer.2c2p.com/docs/sdk-method-self-service-machines`
- `https://developer.2c2p.com/docs/sdk-method-spa-wallet`
- `https://developer.2c2p.com/docs/sdk-method-third-party-payment`
- `https://developer.2c2p.com/docs/sdk-method-touch-n-go`
- `https://developer.2c2p.com/docs/sdk-method-triplea`
- `https://developer.2c2p.com/docs/sdk-method-true-money-wallet`
- `https://developer.2c2p.com/docs/sdk-method-wave-pay`
- `https://developer.2c2p.com/docs/sdk-method-web-payment`
- `https://developer.2c2p.com/docs/sdk-method-web-payment-card-wpc`
- `https://developer.2c2p.com/docs/sdk-method-zalopay`
- `https://developer.2c2p.com/docs/sdk-online-direct-debit`
- `https://developer.2c2p.com/docs/sdk-payment-authorization`
- `https://developer.2c2p.com/docs/sdk-payment-features`
- `https://developer.2c2p.com/docs/sdk-payment-methods`
- `https://developer.2c2p.com/docs/sdk-payment-with-customer-token`
- `https://developer.2c2p.com/docs/sdk-recurring-payment-plan`
- `https://developer.2c2p.com/docs/sdk-references-apple-pay-prerequisite`
- `https://developer.2c2p.com/docs/sdk-references-google-pay-prerequisite`
- `https://developer.2c2p.com/docs/sdk-references-handle-deeplink-payment-flow-by-pgw-sdk-helper`
- `https://developer.2c2p.com/docs/sdk-references-zalopay-prerequisite`
- `https://developer.2c2p.com/docs/sdk-securepay-apis-interface`
- `https://developer.2c2p.com/docs/sdk-securepay-initialize`
- `https://developer.2c2p.com/docs/sdk-securepay-payment-enums`
- `https://developer.2c2p.com/docs/sdk-securepay-payment-request`
- `https://developer.2c2p.com/docs/sdk-user-address-for-payment`

Use for: Android, iOS, Flutter, React Native PGW SDK setup, methods, response handling, Apple Pay and Google Pay in app.

## SoftPOS

- `https://developer.2c2p.com/docs/app-to-app-apis`
- `https://developer.2c2p.com/docs/activation-apptoapp`
- `https://developer.2c2p.com/docs/inquiry-apptoapp`
- `https://developer.2c2p.com/docs/overview-apptoapp`
- `https://developer.2c2p.com/docs/payment-apptoapp`
- `https://developer.2c2p.com/docs/request-parameters-apptoapp`
- `https://developer.2c2p.com/docs/response-parameters-apptoapp`
- `https://developer.2c2p.com/docs/sdk-download-softpos`
- `https://developer.2c2p.com/docs/sdk-softpos-api-payment`
- `https://developer.2c2p.com/docs/sdk-softpos-api-query`
- `https://developer.2c2p.com/docs/sdk-softpos-api-void`
- `https://developer.2c2p.com/docs/sdk-softpos-apis`
- `https://developer.2c2p.com/docs/sdk-softpos-apis-interface`
- `https://developer.2c2p.com/docs/sdk-softpos-how-to-integrate`
- `https://developer.2c2p.com/docs/sdk-softpos-initialize`
- `https://developer.2c2p.com/docs/sdk-softpos-payment-enums`
- `https://developer.2c2p.com/docs/sdk-softpos-payment-info-classes`
- `https://developer.2c2p.com/docs/softpos-device-to-app-overview`
- `https://developer.2c2p.com/docs/softpos-dta-activation-api`
- `https://developer.2c2p.com/docs/softpos-dta-cancel-api`
- `https://developer.2c2p.com/docs/softpos-dta-enquiry-api`
- `https://developer.2c2p.com/docs/softpos-dta-request-parameters`
- `https://developer.2c2p.com/docs/softpos-dta-response-parameters`
- `https://developer.2c2p.com/docs/softpos-dta-transaction-api`
- `https://developer.2c2p.com/docs/softpos-dta-void-api`
- `https://developer.2c2p.com/docs/softpos-notification`
- `https://developer.2c2p.com/docs/softpos-overview`
- `https://developer.2c2p.com/docs/void-apptoapp`

Use for: SoftPOS integration models, Pay Server LAN API, App-to-App poslib, SoftPOS SDK, SoftPOS notification.

## Plugins

- `https://developer.2c2p.com/docs/client-api`
- `https://developer.2c2p.com/docs/magento2`
- `https://developer.2c2p.com/docs/opencart-3`
- `https://developer.2c2p.com/docs/shopify`
- `https://developer.2c2p.com/docs/woocommerce`

Use for: Shopify, WooCommerce, Magento 2, OpenCart 3 installation and settings.

## SNAP API (Indonesia only)

- `https://developer.2c2p.com/docs/snap-account-registration-binding`
- `https://developer.2c2p.com/docs/snap-account-registration-inquiry`
- `https://developer.2c2p.com/docs/snap-direct-debit-payment-cancel`
- `https://developer.2c2p.com/docs/snap-direct-debit-payment-direct-integration`
- `https://developer.2c2p.com/docs/snap-direct-debit-payment-redirect-integration`
- `https://developer.2c2p.com/docs/snap-direct-debit-payment-refund`
- `https://developer.2c2p.com/docs/snap-direct-debit-payment-status`
- `https://developer.2c2p.com/docs/snap-oauth`
- `https://developer.2c2p.com/docs/snap-overview`
- `https://developer.2c2p.com/docs/snap-qr-codes-generate`
- `https://developer.2c2p.com/docs/snap-qr-codes-inquiry`
- `https://developer.2c2p.com/docs/snap-qr-codes-refund`
- `https://developer.2c2p.com/docs/snap-virtual-account-create`
- `https://developer.2c2p.com/docs/snap-virtual-account-inquiry`

Use for: recognising SNAP as Indonesia-only; endpoint list and header signature for completeness.

## Changelog

- `https://developer.2c2p.com/changelog/43-21-april-2025`
- `https://developer.2c2p.com/changelog/43-2nd-april-2025`
- `https://developer.2c2p.com/changelog/43-3rd-april-2025`
- `https://developer.2c2p.com/changelog/430-september-2025`
- `https://developer.2c2p.com/changelog/add-acquirerresponsecode`
- `https://developer.2c2p.com/changelog/add-new-payout-api-12`
- `https://developer.2c2p.com/changelog/new-release-feature-in-v430-1`
- `https://developer.2c2p.com/changelog/payment-v43`
- `https://developer.2c2p.com/changelog/quick-pay-v24`
- `https://developer.2c2p.com/changelog/recon-file-update-v21-v19`

Use for: version changes (v4.3 main version, payout v1.2, recon file versions, field type changes).

