# Adyen Source Map

Use this map to pick the right official source before answering or coding. All URLs checked 2026-10-21. Pages are on `https://docs.adyen.com/` unless a full URL is given. Each docs page also exists as Markdown by adding `.md` (for example `https://docs.adyen.com/online-payments/refund.md`).

## Coverage

- Index used: `https://docs.adyen.com/llms.txt` (2,578 links) and `https://docs.adyen.com/llms-full.txt` (2,577 pages, about 68 MB), cross-checked with `https://docs.adyen.com/sitemap.xml`.
- Enumerated: 2,577 pages. By section: payment-methods 1,044, point-of-sale 328, platforms 190, marketplaces 157, online-payments 127, issuing 120, development-resources 105, plugins 74, business-accounts 74, payouts 63, risk-management 54, classic-platforms 53, reporting 36, capital 36, account 33, unified-commerce 32, others 51.
- In scope for a Malaysian online merchant: 280 pages (online payments, development resources, Malaysian and global wallet payment methods for web, Pay by Link, account, payouts, reporting, disputes).
- Read in full or in part: 119 pages (116 of the 280 plus Hosted Checkout, payment result codes, add payment methods). Hosted Checkout was read from the live HTML because it is missing from `llms-full.txt`.
- The other 164 in-scope pages were scanned by heading and by keyword (Malaysia, MYR, MOLPay, FPX, DuitNow, Touch 'n Go, GrabPay); facts found were added (for example the `molpay_cash` 7-Eleven variant). They cover areas a basic MY integration does not need (donations, partial payments, account updater, network tokens, surcharges, payment facilitators, enhanced scheme data, web v6 upgrade, Data Protection API, mTLS, PGP, reserve and balances detail).
- Official machine sources read: `https://github.com/Adyen/adyen-openapi` `json/CheckoutService-v72.json` (all 31 operations, field tables generated into `adyen-fields.md`), `json/CheckoutService-v71.json`, `json/Webhooks-v1.json`, `json/TokenizationNotification-v1.json`, `json/RecurringService-v68.json`, `json/DisputeService-v30.json`, `json/ManagementNotificationService-v3.json`. Official HMAC code: `https://github.com/Adyen/adyen-node-api-library/blob/main/src/utils/hmacValidator.ts`.
- Left out, and why:
  - Point of sale (328): in-person terminals, not online.
  - Platforms, marketplaces, classic platforms, issuing, capital, business accounts, payouts (about 690): Balance Platform products that need separate contracts; not standard merchant Checkout.
  - Plugins (74): third-party platform setup; the API facts are the same.
  - Mobile SDK pages (iOS, Android, React Native, Flutter) for each method: web only here; server calls are identical.
  - Payment methods not offered in Malaysia (about 950 pages: EU, US, LATAM, India, Korea, Japan local methods).
  - Classic API integrations (`online-payments/classic-integrations`), India and EU (PSD2) regulation guides, Korean cards, batch processing, Amazon Pay, release notes beyond Web environment and SRI notes.

## Get Started And Account

- `set-up-adyen-payments`
- `set-up-adyen-payments/application-requirements`
- `online-payments/integration-checklist`
- `online-payments/go-live-checklist`
- `account/payments-lifecycle`
- `account/getting-paid`, `account/sales-day-payout`, `account/supported-currencies`
- `payment-methods/add-payment-methods`
- `development-resources/pci-dss-compliance-guide`

Use for: test and live accounts, documents, website requirements, Customer Area paths, go-live steps, payouts, PCI level, lifecycle statuses.

## Environments, Auth And API Behaviour

- `development-resources/live-endpoints`
- `development-resources/api-authentication`
- `development-resources/api-credentials`, `development-resources/api-credentials/roles`
- `development-resources/client-side-authentication`
- `development-resources/api-idempotency`
- `development-resources/response-handling`
- `development-resources/versioning`
- `development-resources/currency-codes`
- `development-resources/security/integration-security/allowlisting`
- `development-resources/libraries`, `development-resources/mcp-server`, `https://github.com/Adyen/adyen-mcp`
- `development-resources/logs-resources/api-logs`

Use for: base URLs, live prefix, API key and client key, roles, idempotency and retries, HTTP errors, minor units, IP ranges, SDKs, MCP server, debugging.

## Checkout Flows

- `online-payments/build-your-integration`
- `online-payments/build-your-integration/sessions-flow`
- `online-payments/build-your-integration/advanced-flow`
- `online-payments/build-your-integration/hosted-checkout`
- `online-payments/payment-result-codes`
- `online-payments/upgrade-your-integration/upgrade-to-checkout-api-v72`
- `online-payments/web-best-practices`
- `online-payments/3d-secure/native-3ds2`
- `online-payments/3d-secure-for-regulation-compliance`
- `online-payments/release-notes` (Web environments, `live-apse`, SRI)
- `https://docs.adyen.com/api-explorer/Checkout/latest/overview`

Use for: `/sessions`, `/paymentMethods`, `/payments`, `/payments/details`, Drop-in setup, actions, result codes, v72 validation, 3DS2 fields, Malaysian 3DS note.

## Webhooks

- `development-resources/webhooks`
- `development-resources/webhooks/configure-and-manage`
- `development-resources/webhooks/handle-webhook-events`
- `development-resources/webhooks/secure-webhooks`
- `development-resources/webhooks/secure-webhooks/verify-hmac-signatures`
- `development-resources/webhooks/webhook-types`
- `development-resources/webhooks/webhook-types/additional-settings`
- `development-resources/webhooks/troubleshoot`
- `development-resources/logs-resources/webhook-event-logs`

Use for: endpoint setup, HMAC algorithm and order, accept-then-process, retries, event codes, duplicates, `out.adyen.com`.

## Modifications

- `online-payments/capture`, `online-payments/capture/failure-reasons`
- `online-payments/refund`
- `online-payments/cancel`
- `online-payments/reversal`
- `online-payments/adjust-authorisation`

Use for: capture modes and delays, partial captures, refunds and reasons, cancel by PSP or own reference, reversals, authorisation validity, failure handling.

## Malaysian Payment Methods

- `payment-methods` (overview table with countries and settlement currencies)
- `payment-methods/fpx-online-banking-malaysia` plus `/api-only`, `/web-component`, `/web-drop-in`, `/web`
- `payment-methods/duitnow` plus `/web-component`, `/web-drop-in`, `/web`
- `payment-methods/touchngo` plus `/api-only`, `/web-component`, `/web-drop-in`, `/web`
- `payment-methods/grabpay` plus `/api-only`, `/web-component`, `/web-drop-in`, `/web`
- `payment-methods/boost` plus `/api-only`, `/web-component`, `/web-drop-in`, `/web`
- `payment-methods/atome` plus `/api-only`, `/web-component`, `/web-drop-in`, `/web`
- `payment-methods/alipay` plus `/api-only`, `/web-component`, `/web-drop-in`, `/web`
- `payment-methods/alipay-plus` plus `/api-only`, `/web-component`, `/web-drop-in`, `/web`
- `payment-methods/wechat-pay`, `payment-methods/wechat-pay/wechat-pay-desktop-qr-payments` plus `/api-only`, `/web-component`, `/web-drop-in`; in-app and mini program pages (read, skipped as non-web)
- `payment-methods/cards/web-drop-in`, `payment-methods/cards/credit-card-installments`, `payment-methods/apple-pay`, `payment-methods/google-pay`
- `development-resources/paymentmethodvariant`

Use for: method types, FPX issuer list and PayNet rule, required fields (Atome, GrabPay PayLater, Alipay+), offer timeouts, refund limits, enablement path, simulator testing, 7-Eleven variant.

## Tokenization And Payment Links

- `online-payments/tokenization`
- `online-payments/tokenization/create-tokens`
- `online-payments/tokenization/make-token-payments`
- `online-payments/tokenization/managing-tokens`
- `unified-commerce/pay-by-link`
- `unified-commerce/pay-by-link/create-payment-links/api`
- `unified-commerce/pay-by-link/supported-payment-methods`

Use for: `storePaymentMethodMode`, `recurringProcessingModel`, token webhooks, ContAuth payments, token list and delete, payment link create, status, expiry, supported methods.

## Testing And Errors

- `development-resources/test-cards-and-credentials/test-card-numbers`
- `development-resources/test-cards-and-credentials/alternative-payment-method-credentials`
- `development-resources/testing/3d-secure-2-authentication`
- `development-resources/testing/result-codes`
- `development-resources/testing/tokenization`
- `development-resources/error-codes`
- `development-resources/refusal-reasons`
- `development-resources/raw-acquirer-responses`

Use for: sandbox cards, 3DS passwords, result triggers, refusal codes, API error codes.

## Reporting And Disputes

- `reporting/automatically-get-reports`
- `reporting/settlement-reconciliation/transaction-level/settlement-details-report`
- `risk-management/disputes-api`

Use for: `REPORT_AVAILABLE` download flow, report user credential, journal types, dispute defence API.

## Pricing

- `https://www.adyen.com/pricing`

Use for: current fees only by link; do not copy numbers.
