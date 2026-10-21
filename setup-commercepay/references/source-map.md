# CommercePay Source Map

Use this map to pick the right official source before answering or coding. All pages checked 2026-10-21.

## Coverage

- Index used: `https://docs.commercepay.asia/llms.txt`, `https://docs.commercepay.asia/sitemap.xml` (32 URLs), `https://docs.commercepay.asia/llms-full.txt`, `https://docs.commercepay.asia/openapi.json` (13 operations, 50+ schemas).
- Pages enumerated: 32. Pages read: 32 (each also fetched as `.md` for exact field names), plus the full OpenAPI spec.
- Extra official sources read: WooCommerce plugin v1.4 source (`cap-gateway.php`), marketing site pages (home, FAQ, developer, get-started, register).
- Superseded: legacy Stoplight docs at `https://commerceasiapayments.stoplight.io/docs/commerce-asia-payments-gateway-cap-api/` now show "This documentation has moved"; not used for facts.
- Left out: nothing in the public developer docs. Global-only areas: none (CommercePay is Malaysia-focused).

## Product And Onboarding

- Overview: `https://docs.commercepay.asia/overview`
- Setup Guide: `https://docs.commercepay.asia/get-started`
- API Credentials: `https://docs.commercepay.asia/api-credentials`
- Service Environments: `https://docs.commercepay.asia/service-environments`
- Site home: `https://www.commercepay.asia`
- Register: `https://www.commercepay.asia/register`
- FAQ: `https://www.commercepay.asia/faq`

Use for: credentials, staging/production base URLs, onboarding contact, BNM licence, settlement timing, fee model (no numbers).

## Auth, Headers, Signature

- Request Headers: `https://docs.commercepay.asia/request-headers`
- Get Access Token: `https://docs.commercepay.asia/api-reference/get-access-token`
- Generate Signature: `https://docs.commercepay.asia/generate-signature`

Use for: Bearer token, `Abp-TenantId`, `cap-signature` POST and GET algorithms, encoding rules, test vectors, Signature Validator Tool.

## Checkout Flows

- Direct Integration guide: `https://docs.commercepay.asia/direct-integration`
- Hosted Session Checkout guide: `https://docs.commercepay.asia/hosted-session-checkout`
- Direct Integration API: `https://docs.commercepay.asia/api-reference/direct-integration-api-request-payment`
- Request Payment API (full schema): `https://docs.commercepay.asia/api-reference/paymentgateway/request-payment`
- Hosted Session Checkout API: `https://docs.commercepay.asia/api-reference/initial-session`
- Process Payment API: `https://docs.commercepay.asia/api-reference/paymentgateway/process-payment`

Use for: flow sequence, RequestPayment and InitialSession fields, redirection types, pre-auth and capture.

## Channels

- Channel List: `https://docs.commercepay.asia/channel-list`
- Get Channel List API: `https://docs.commercepay.asia/api-reference/get-tenant-channels-by-country`
- Get Provider Channels API: `https://docs.commercepay.asia/api-reference/get-provider-channels`

Use for: `channelId` per environment, `isProviderHostChannel`, FPX/DuitNow bank `providerChannelId` codes, channel refund types.

## Callbacks And Status

- Callback Notifications: `https://docs.commercepay.asia/callback-notifications`
- Query Payment API: `https://docs.commercepay.asia/api-reference/query-payment`

Use for: callback payload, callback signature verification, payment status codes, live vs stored query behaviour, session queries, settlement status.

## Refunds

- Refund Integration: `https://docs.commercepay.asia/refund-integration`
- Refund Payment API: `https://docs.commercepay.asia/api-reference/refund-payment`
- Query Refund Payment API: `https://docs.commercepay.asia/api-reference/query-refund-payment`

Use for: full vs partial refund, bank support table, cross-border limitation, refund status and errors.

## Saved Payments And Recurring

- Recurring Payments: `https://docs.commercepay.asia/recurring-payments`
- Add Saved Payment API: `https://docs.commercepay.asia/api-reference/add-saved-payment`
- Check Saved Payment API: `https://docs.commercepay.asia/api-reference/check-saved-payment`
- Revoke Saved Payment API: `https://docs.commercepay.asia/api-reference/revoke-saved-payment`

Use for: `recurringPaymentInfo`, supported channels, approval requirement, MYR 1.00 verification, one saved method per customer per channel, revoke.

## Payouts And Wallets

- Payout Integration: `https://docs.commercepay.asia/payout-integration`
- Payout Provider Channel Code: `https://docs.commercepay.asia/payout-provider-channel`
- Payout Test Accounts: `https://docs.commercepay.asia/payout-test-account`
- Request Payout API: `https://docs.commercepay.asia/api-reference/payout-api-requestpayout`
- Query Wallets API (payout): `https://docs.commercepay.asia/api-reference/payout-api-querywallets`
- Query Wallets API (payment gateway): `https://docs.commercepay.asia/api-reference/paymentgateway/query-wallets`
- Query Wallet Histories API: `https://docs.commercepay.asia/api-reference/payout-api-querywallethistories`

Use for: payout channel IDs, bank and DuitNow Proxy codes, staging test recipients, wallet balance and history.

## Plugins And Reference Code

- Developer page with plugin downloads: `https://www.commercepay.asia/developer`
- WooCommerce plugin: `https://capstoragedata.blob.core.windows.net/landingpage/plugins/woocommerce/cap-gateway.zip`
- WooCommerce (Sequential Order Numbers): `https://capstoragedata.blob.core.windows.net/landingpage/plugins/woocommerce-sequential-order-numbers/cap-gateway.zip`
- OpenCart 3: `https://capstoragedata.blob.core.windows.net/landingpage/plugins/opencart-v3.0.8/commercepay.ocmod.zip`
- OpenCart 2: `https://capstoragedata.blob.core.windows.net/landingpage/plugins/opencart-v2.1.0.1/commercepay.zip`
- WHMCS: `https://capstoragedata.blob.core.windows.net/landingpage/plugins/whmcs/whmcs.zip`
- NopCommerce: `https://www.nopcommerce.com/en/commerceasia-payments`

Use for: plugin setup, and as secondary evidence for callback verification over the raw body and return URL query parameters (`ReferenceCode`, `TransactionNumber`). Plugin v1.4 predates current docs on timestamp units and Bearer auth.

## Back Office

- Production: `https://backoffice-payments.commerce.asia/`
- Staging Signature Validator: `https://staging-backoffice-payments.commerce.asia/app/paymentgateway/signaturevalidator`

Use for: transaction reports, wallet view, payment links/forms, signature debugging. Do not use as payment truth for fulfilment.
