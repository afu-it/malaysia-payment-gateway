# Ezypay Source Map

All pages are official. Guides: `https://developer.ezypay.com/docs/<slug>`. API reference: `https://developer.ezypay.com/reference/<slug>`. Changelog: `https://developer.ezypay.com/changelog/<slug>`. Index used: `https://developer.ezypay.com/llms.txt` (97 URLs) plus the full API reference sidebar from the reference HTML (110 slugs). Checked 2026-10-21.

## Start here

- `docs/supported-payment-methods`: Use for: Malaysia method list, 3DS and CVV, currency, settlement holding period, failed payment guide link.
- `docs/integrating-with-ezypay`: Use for: partner vs integrated merchant, subscription vs on-demand, the 10 mandatory integration items.
- `reference/reference-getting-started`, `reference/authentication`, `docs/authentication-1`: Use for: token request, refresh, scopes, rate limits, `merchant` header.
- `reference/sandbox`, `docs/sandbox-intro`: Use for: sandbox and production hosts, sandbox expiry.
- `reference/privacy-and-security`, `reference/terms-and-conditions`, `reference/about-us`: Use for: HTTPS rule, terms, background.

## Onboarding and go-live

- `docs/partner-integration-process`, `docs/integrated-merchant-integration-process`: Use for: integration stages and sign-offs.
- `docs/merchant-onboarding`: Use for: application form, checks, credential issue.
- `docs/go-live-checklist`, `docs/partners-go-live-checklist`, `docs/integrated-merchant-go-live-checklist`: Use for: pre and post go-live steps, host switch.
- `docs/ezypays-fees`, `docs/display-customer-paid-fees`: Use for: fee types, who pays, how to display fees.
- External: `https://www.ezypay.com/my`, `https://www.ezypay.com/pricing-plan` (no rates published), `https://www.ezypay.com/sandbox-request`, `https://www.ezypay.com/become-a-partner`, `https://www.ezypay.com/contact`.

## Customers and payment methods

- `docs/customer-creation`, `docs/customer-transfer`: Use for: customer fields, moving customers between merchants.
- `reference/customer-1`, `create-a-customer`, `list-customers`, `retrieve-a-customer`, `update-a-customer`: Use for: customer field tables.
- `docs/payment-method-collection`, `docs/payment-capture-page`, `docs/payment-capture-page-styling`, `docs/difference-between-hpp-and-pcp`, `docs/hosted-payment-page`: Use for: PCP iframe, parameters, postMessage, styling, legacy HPP.
- `docs/mobile-app-integration`, `docs/for-ios`, `docs/for-android`: Use for: SFSafariViewController, WKWebView, Custom Tabs, WebView bridges.
- `docs/apple-pay-payment-integration`, `docs/google-pay-payment-integration`: Use for: wallet setup, domain verification, payload display.
- `docs/payment-methods-payload`, `docs/payment-method-management`, `docs/cards-network-tokenization`, `docs/vault-environment`, `docs/public-keys`: Use for: payload fields, replace or delete, network tokens, retention, PGP key.
- `reference/payment-methods`, `payment-methods-2`, `create-a-payment-method`, `list-payment-methods`, `retrieve-a-payment-method`, `retrieve-primary-payment-method`, `update-primary-payment-method`, `replace-a-payment-method`, `delete-a-payment-method`, `create-a-bank-payment-method-token`, `create-a-qr-payment-method-token`: Use for: payment method and vault calls.
- `reference/hosted-pages`, `get-a-hosted-page-token-1`, `payment-capture-page-styling`, `create-a-styling`, `get-a-styling`: Use for: hosted page token and styling API.

## Billing

- `docs/on-demand`: Use for: on-demand invoice flow, `externalInvoiceId`, `processingModel`.
- `docs/subscription`, `docs/subscription-management`, `docs/subscription-payment-scenarios`, `docs/prorate-calculation`: Use for: plans, subscriptions, billing start and end, prorate.
- `docs/checkout`, `docs/checkout-full-process`: Use for: hosted checkout, redirects, session disable.
- `reference/invoices`, `invoices-2`, `create-an-invoice`, `list-invoices`, `retrieve-an-invoice`, `update-an-invoice`, `retry-a-payment-on-an-invoice`, `write-off-an-invoice`, `record-an-external-payment-for-an-invoice`, `cancel-a-qr-payment-request`: Use for: invoice fields and status requery.
- `reference/create-a-checkout-session`, `disable-a-checkout-session`: Use for: checkout fields.
- `reference/plans`, `create-a-plan`, `list-plans`, `retrieve-a-plan`, `update-a-plan`: Use for: plan fields.
- `reference/subscriptions`, `create-a-subscription`, `preview-a-subscription`, `list-subscriptions`, `retrieve-a-subscription`, `update-subscription`, `activate-a-subscription`, `cancel-a-subscription`, `undo-a-subscription-cancellation`, `update-subscriptions-payment-method`: Use for: subscription fields.
- `reference/future-invoices`, `list-future-invoices`, `update-a-future-invoice`, `delete-a-future-invoice`, `record-an-external-payment-for-a-future-invoice`: Use for: skipping or editing upcoming cycles.

## Failed payments, refunds, chargebacks

- `docs/failed-payment-responses`, `docs/payment-recovery`, `docs/retry-schedule`, `docs/resolving-failed-invoice`, `docs/subscription-failed-payment-handling`, `docs/sample-failed-payment-email`: Use for: failure reasons, retries, write off, stop rules.
- `docs/immediate-online-payment-recovery-pay-now`, `docs/pay-now-full-process`: Use for: Pay Now link.
- External: `http://www.ezypay.com/failed-payment-response-guide/my/mastercard-visa` (redirects to an ezypay.qwilr.com page): Use for: Malaysia processor code mapping.
- `docs/refunds`, `reference/refund`, `refund-an-invoice`: Use for: refund rules and errors.
- `docs/chargeback`, `docs/chargeback-common-questions`: Use for: chargeback flow and evidence.
- `reference/transactions`, `list-transactions`, `retrieve-a-transaction`, `list-credit-notes`, `retrieve-a-credit-note`: Use for: transaction and credit note fields.

## Webhooks

- `docs/webhook`, `docs/webhook-events`, `docs/webhook-security`, `docs/webhook-behaviour`, `docs/webhook-management`: Use for: events, payloads, HMAC-SHA1 signature, retries, management.
- `reference/webhooks-3`, `create-webhook`, `update-webhook`, `list-webhooks`, `retrieve-webhook-details`, `delete-webhook`, `list-webhook-notification-logs`, `events-1`, `resend-event`: Use for: webhook API fields.

## Settlement and reports

- `docs/settlement`, `docs/settlement-reports-details`, `docs/reports-1`: Use for: holding period, schedule, report contents.
- `reference/settlements`, `list-settlements`, `generate-a-settlement-document`, `file-service`, `get-file`, `reports`, `generate-merchant-transactions-report`, `generate-outstanding-collections-report`: Use for: settlement and report APIs.
- `reference/merchants`, `retrieve-a-merchant-1`, `retrieve-customer-paid-fee-schedule-for-merchant`, `retrieve-collected-next-settlement-amount`: Use for: merchant, fees, next settlement.

## Testing

- `docs/test-payment-data`, `reference/test-tutorial`: Use for: test cards and amounts.
- `docs/sandbox-apple-pay-integration`, `docs/sandbox-google-pay-integration`: Use for: wallet sandbox setup.

## Partner multi-split (summarised)

- `docs/multi-split-payouts`: Use for: wallet-to-wallet transfers between merchants.
- `reference/merchant-connections`, `list-merchant-connections`, `retrieve-a-merchant-connection`, `partner-invoices`, `create-a-partner-invoice`, `list-partner-invoices`, `attempt-payment-on-a-partner-invoice`, `partner-subscriptions`, `create-a-partner-subscription`, `list-partner-subscriptions`, `cancel-a-partner-subscription`, `record-a-partner-subscription-add-on-usage`, `list-partner-subscription-add-on-usage`: Use for: partner billing fields.

## Not Malaysia (left out or summarised)

- PayTo, Australia: `docs/about-payto`, `docs/payto-faqs`, `docs/payto-mandate-sandbox`, `reference/mandate-types`, `get-a-mandate-type`, `list-mandate-types`. Endpoints listed in the catalogue only.
- GCash, Philippines: `docs/about-gcash`, `docs/sandbox-gcash-integration`. Left out.
- PromptPay, Thailand: `docs/prompt-pay`. Only the QR token and cancel calls are catalogued.
- Korean Hosted Payment Page: `docs/korean-hosted-payment-page`. Left out.
- Tap to Pay, AU and NZ: `docs/terminal-integration`, `docs/sandbox-tap-to-pay-integration`, `reference/tap-to-pay-1`, `register-terminal-device`, `terminal-create-an-invoice`, `list-terminal-invoices`, `retrieve-a-terminal-invoice`. Summarised.
- AU and NZ bank direct debit test data and bank token API. Summarised.

## Changelog

- `docs/document-changelog` and `changelog/` pages: `week-ending-2401`, `changelog-week-ending-06122024`, `changelog-week-ending-20122024`, `changelog-week-ending-3012025`, `changelog-week-ending-07022025`, `changelog-week-ending-21022025`, `change-log-week-ending-07032025`, `changelog-week-ending-21032025`, `changelog-week-ending-04042025`, `changelog-week-ending-31072025`. Use for: methods not enabled by default, NZ 60-day refund limit, no failed payment fee on suspected fraud.

## Coverage

| Area | Enumerated | Read | Treatment |
| --- | --- | --- | --- |
| Guides (`/docs`) | 70 (69 guides plus `page/overview`, which is placeholder text) | 70 | Full; long mobile code blocks and big JSON samples skimmed |
| API reference (`/reference`) | 110 slugs: 79 operations plus section and info pages | 110 | Full; OpenAPI request and response schemas extracted by script |
| Changelog | 10 | 10 | Full |
| Website pages | 4 (`/my`, pricing, sandbox request, Malaysia failed payment guide) | 4 | Facts only, no prices |
| Total | 194 | 194 | |

Global-only areas (PayTo, GCash, PromptPay, Korean HPP, Tap to Pay, AU and NZ bank debit) were read but left out or summarised, because the docs say they are not offered in Malaysia. No official SDK was found. Not documented in official docs (checked 2026-10-21): general API rate limits beyond identity, webhook source IPs, an idempotency header, production NPP host.
