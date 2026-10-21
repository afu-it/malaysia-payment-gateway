# LeanX Source Map

Use this map to pick the right official source before answering or coding. All pages checked 2026-10-21.

## Coverage

- Index used: `https://docs.leanx.io/llms.txt` (43 entries) and `https://docs.leanx.io/api-docs/sitemap-pages.xml` (43 URLs; the llms.txt "Introduction" is the sitemap root `/api-docs`).
- Full text read: `https://docs.leanx.io/llms-full.txt` (all 43 pages, 3,987 lines).
- Pages enumerated: 43. Pages read: 43.
- Also read: official WooCommerce plugin source and OpenCart 3.0 plugin source (Google Drive downloads linked from the docs plugin pages).
- The Credential page "Postman Collection" section has no content. No OpenAPI spec, SDK repo, or PDF guide is published. A GitHub repo `AdamSallehLeanis/leanx-opencart` exists under a personal account; not used as it is not an official org.
- Nothing was left out as global-only; the docs are Malaysia-only (MYR).

## Overview And Credentials

- Introduction: `https://docs.leanx.io/api-docs/introduction`
- Credential: `https://docs.leanx.io/api-docs/credential`
- Token Validation: `https://docs.leanx.io/api-docs/token-validation`

Use for: base URLs, sandbox vs production, public sandbox credential, credential types, `merchant/validate`, HTTPS requirement.

## Request Signing And Errors

- Signature Validation: `https://docs.leanx.io/api-docs/signature-validation`
- Error Codes: `https://docs.leanx.io/api-docs/error-codes`

Use for: X-Signature message order, HMAC-SHA256 with Hash Key, `x-signature` / `x-timestamp` / `x-nonce`, 5-minute validity, portal toggle, full error code table.

## Quick Starts

- Accepting via Payment Page: `https://docs.leanx.io/api-docs/quick-start/accepting-via-payment-page`
- Accepting Using Own Page: `https://docs.leanx.io/api-docs/quick-start/accepting-using-own-page`
- Paying Out: `https://docs.leanx.io/api-docs/quick-start/paying-out`

Use for: end-to-end flows, `create-bill-page` and `create-bill-silent` fields, callback JWT sample, status requery, payout steps.

## Collection And Bill APIs

- Payment Service: `https://docs.leanx.io/api-docs/cloud-payment/payment-service`
- Collection: `https://docs.leanx.io/api-docs/cloud-payment/collection`
- Create Collection: `https://docs.leanx.io/api-docs/cloud-payment/collection/create-collection`
- Manage (collections): `https://docs.leanx.io/api-docs/cloud-payment/collection/manage`
- Get Collection List: `https://docs.leanx.io/api-docs/cloud-payment/collection/manage/get-collection-list`
- Get a Collection: `https://docs.leanx.io/api-docs/cloud-payment/collection/manage/get-a-collection`
- Update Collection: `https://docs.leanx.io/api-docs/cloud-payment/collection/manage/update-collection`
- Activate Collection: `https://docs.leanx.io/api-docs/cloud-payment/collection/manage/activate-collection`
- Deactivate Collection: `https://docs.leanx.io/api-docs/cloud-payment/collection/manage/deactivate-collection`
- Delete Collection: `https://docs.leanx.io/api-docs/cloud-payment/collection/manage/delete-collection`
- Bill: `https://docs.leanx.io/api-docs/cloud-payment/bill`
- Create Bill: `https://docs.leanx.io/api-docs/cloud-payment/bill/create-bill`
- Manage (bills): `https://docs.leanx.io/api-docs/cloud-payment/bill/manage`
- Get Bill List: `https://docs.leanx.io/api-docs/cloud-payment/bill/manage/get-bill-list`
- Get a Bill: `https://docs.leanx.io/api-docs/cloud-payment/bill/manage/get-a-bill`
- Get Bill Transaction Status: `https://docs.leanx.io/api-docs/cloud-payment/bill/manage/get-bill-transaction-status`
- Update Bill: `https://docs.leanx.io/api-docs/cloud-payment/bill/manage/update-bill`
- Delete Bill: `https://docs.leanx.io/api-docs/cloud-payment/bill/manage/delete-bill`

Use for: payment types and `payment_model_reference_id`, FPX B2C/B2B lists, collection CRUD, bill creation (page and silent), DuitNow QR `metadata` (`png` / `emv`) and 3-second polling, bill management fields.

## Callbacks And Status

- Bill Callback: `https://docs.leanx.io/api-docs/cloud-payment/bill/callback`
- Transaction Status: `https://docs.leanx.io/api-docs/cloud-payment/transaction-status`
- Transaction List: `https://docs.leanx.io/api-docs/cloud-payment/transaction-list`
- FAQ: `https://docs.leanx.io/api-docs/support/frequently-asked-question`

Use for: JWT callback format and payload, Hash Key decoding, success-only callbacks, 3 retries until HTTP 200, `manual-checking-transaction`, transaction report and 100-row cap, IP whitelist, token behaviour.

## Account And Payouts

- Pool Balance: `https://docs.leanx.io/api-docs/account/pool-balance`
- Payout Service: `https://docs.leanx.io/api-docs/payout/payout-service`
- Payout Transaction List: `https://docs.leanx.io/api-docs/payout/payout-transaction-list`
- Create Payout: `https://docs.leanx.io/api-docs/payout/create-payout`
- Payout Status: `https://docs.leanx.io/api-docs/payout/payout-status`
- Payout Callback: `https://docs.leanx.io/api-docs/payout/payout-status/callback`
- Status Checking: `https://docs.leanx.io/api-docs/payout/payout-status/status-checking`
- Bank Account Verification: `https://docs.leanx.io/api-docs/payout/bank-account-verification`
- Bank Verification Channel (Deprecated): `https://docs.leanx.io/api-docs/payout/bank-account-verification/bank-verification-channel-deprecated`
- Bank Account Validation: `https://docs.leanx.io/api-docs/payout/bank-account-verification/bank-account-validation`

Use for: prefund pool, RM2 minimum, payout creation fields, payout statuses, payout callback payload, account name validation, payout test accounts.

## Plugins And Libraries

- WooCommerce: `https://docs.leanx.io/api-docs/suite-plugin/woocommerce`
- OpenCart: `https://docs.leanx.io/api-docs/suite-plugin/open-cart`
- Credit Card library: `https://docs.leanx.io/api-docs/library/credit-card`
- Card SDK script: `https://sdk.leanx.io/index.umd.min.js`
- WooCommerce plugin download: `https://drive.google.com/file/d/1shoSLSnKwhW7BxwGjXzgnh0w9r_psCUt/view`
- OpenCart plugin download: `https://drive.google.com/file/d/12dBLvZqHiQhqATgCkupI3EHTwuCy1ofO/view?usp=share_link`

Use for: plugin settings, legacy plugin endpoints (`public-merchant/...`, `jwt/decode`), status strings seen in practice, card form embedding with `transaction_invoice_no`.

## Commercial And Onboarding

- Website: `https://leanx.io`
- Pricing: `https://leanx.io/pricing`
- Onboarding: `https://onboard.leanx.io`
- Sandbox portal: `https://portal.leanx.dev`
- Production portal: `https://portal.leanx.io`

Use for: registration, plan comparison, fees, settlement terms. Link to pricing; do not copy numbers.
