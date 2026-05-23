# Bayarcash API Reference

## Sources

- API docs: `https://api.webimpian.support/bayarcash`
- GitHub docs repo snapshot: `references/bayarcash-api-documentation.md`, sourced from `https://github.com/webimpian/bayarcash-api-documentation`
- Bayarcash docs: `https://docs.bayarcash.com/`
- PHP SDK: `https://github.com/webimpian/bayarcash-php-sdk`
- WordPress plugin example supplied by user: `https://wordpress.org/plugins/e-invoice-for-myinvois-lhdn/`
- Boost.space invite supplied by user: `https://integrator.boost.space/app/invite/adcf82114e0d7042cbe2471d4f200cc8`
- Smithery MCP page supplied by user: `https://smithery.ai/servers/webimpianteam/bayarcash-mcp-server#api`

## Live Docs Lookup

Bayarcash API docs expose Markdown pages. Prefer `.md` URLs over scraping HTML.

Index:

```text
https://api.webimpian.support/bayarcash/~gitbook/site-index
https://api.webimpian.support/bayarcash/sitemap.md
https://api.webimpian.support/bayarcash/llms-full.txt
```

Question endpoint pattern:

```text
GET https://api.webimpian.support/bayarcash/<page>.md?ask=<specific question>
```

Use live lookup when details are missing or likely changed: payment channel IDs, bank list/availability, plugin versions, fees, settlement rules, account settings, MCP API shape, Boost.space connector behavior.

## Environments

Sandbox console:

```text
https://console.bayarcash-sandbox.com
API v2: console.bayarcash-sandbox.com/api/v2
API v3: api.console.bayarcash-sandbox.com/v3
```

Production console:

```text
https://console.bayar.cash
API v2: console.bayar.cash/api/v2
API v3: api.console.bayar.cash/v3
```

Authentication in examples uses:

```text
Authorization: Bearer <Personal_Access_Token>
Content-Type: application/json
```

## Endpoint Map

Pages from site index:

- `GET /banks`: FPX bank list. v2 `console.bayar.cash/api/v2/banks`, v3 `api.console.bayar.cash/v3/banks`.
- Payment channel: `/bayarcash/payment/payment-channel.md`.
- Payment intent: `POST /payment-intents`. v2 `console.bayar.cash/api/v2/payment-intents`, v3 `api.console.bayar.cash/v3/payment-intents`.
- Payment intent ID: `/bayarcash/payment/payment-intent-id.md`.
- Callback: `/bayarcash/transaction/callback.md`.
- Transaction ID: `/bayarcash/transaction/transaction-id.md`.
- All transactions: `/bayarcash/transaction/all-transactions.md`.
- Checksum validation: `/bayarcash/checksum/checksum-validation.md`.
- Payment intent checksum: `/bayarcash/checksum/payment-intent.md`.
- Transaction callback checksum: `/bayarcash/checksum/transaction-callback.md`.
- Portal list/create: `/bayarcash/portal/list.md`, `/bayarcash/portal/create.md`.
- Direct debit/e-Mandate: enrollment, maintenance, termination, activate, deactivate, callback, mandate ID, mandate transaction ID under `/bayarcash/direct-debit/`.
- Enterprise partner: merchant registration/status, payout bank list, merchant payout ID, all merchant payouts under `/bayarcash/enterprise-partner/`.

## Payment Intent

Create payment intent:

```text
v2 POST console.bayar.cash/api/v2/payment-intents
v3 POST api.console.bayar.cash/v3/payment-intents
```

Required request fields:

- `payment_channel` integer
- `portal_key` string
- `order_number` string
- `amount` integer
- `payer_name` string
- `payer_email` string

Optional request fields:

- `payer_telephone_number` integer; docs say Malaysia number only.
- `payer_bank_code` string
- `payer_bank_name` string
- `metadata` string; docs say currently only WooCommerce order items.
- `return_url` string; server-to-browser redirect callback, use `GET`.
- `callback_url` string; server-to-server callback, use `POST`, v3 only.
- `platform_id` string
- `checksum` string

Response includes checkout `url`; v3 response also includes `type` and `id`, for example `pi_MGWpzp`.

## Banks

FPX bank list:

```text
v2 GET console.bayar.cash/api/v2/banks
v3 GET api.console.bayar.cash/v3/banks
```

Use returned `bank_code` and `bank_name` for `payer_bank_code` and `payer_bank_name`. Response also contains `bank_display_name`, `bank_code_hashed`, and `bank_availability`.

## Callbacks

Callback routes:

```text
v2 POST {return_url}
v3 GET {return_url}
v3 POST {callback_url}
```

Server callback must return HTTP `200`. Bayarcash retries up to 5 attempts every 300 seconds from last attempt.

Transaction status map:

```text
0 New
1 Pending
2 Failed
3 Success
4 Cancelled
```

Transaction callback fields include `record_type`, `transaction_id`, `exchange_reference_number`, `exchange_transaction_id`, `order_number`, `currency`, `amount`, `payer_name`, `payer_email`, `payer_bank_name`, `status`, `status_description`, `datetime`, and `checksum`.

Fulfillment rules:

- Verify checksum before accepting success.
- Process only terminal paid state once: status `3`.
- Persist raw callback and verification result for audit.
- Make callback handler idempotent by `transaction_id` and/or `order_number`.

## Checksums

Docs show payment intent checksum as HMAC SHA256 using API secret key:

1. Build payload data.
2. Sort payload keys with `ksort`.
3. Concatenate values with `|`.
4. Generate `hash_hmac('sha256', $payloadString, $secretKey)`.

Note: The published sample variable uses `$stringPayload` after defining `$payloadString`. Treat this as a likely docs typo and verify against SDK/live docs before copying code.

SDK README says checksum values and validation are optional but recommended for security.

## PHP SDK

Install:

```bash
composer require webimpian/bayarcash-php-sdk
```

Basic usage:

```php
$bayarcash = new Webimpian\BayarcashSdk\Bayarcash(TOKEN_HERE);
$bayarcash->useSandbox();
$bayarcash->setApiVersion('v3');
```

Laravel usage expects:

```text
BAYARCASH_API_TOKEN=
BAYARCASH_API_SECRET_KEY=
```

Important SDK methods from README:

- `getPortals()`
- `getChannels($portalKey)`
- `fpxBanksList()`
- `createPaymentIntenChecksumValue($secretKey, $data)` as spelled in README.
- `createPaymentIntent($data)`
- `verifyPreTransactionCallbackData($data, $secretKey)`
- `verifyTransactionCallbackData($data, $secretKey)`
- `verifyReturnUrlCallbackData($data, $secretKey)`
- `getPaymentIntent($paymentIntentId)` v3 only.
- `getTransaction($transactionId)`
- v3 transaction filters: `getAllTransactions`, `getTransactionByOrderNumber`, `getTransactionsByPayerEmail`, `getTransactionsByStatus`, `getTransactionsByPaymentChannel`, `getTransactionByReferenceNumber`.
- Direct debit helpers: enrollment, maintenance, termination intent creation plus checksum helpers.

Payment channels exposed as SDK constants include FPX, manual transfer, FPX direct debit, DuitNow, QR, credit card, Alipay, WeChat Pay, Touch 'n Go, Boost, GrabPay, ShopeePay, and others. Verify exact numeric IDs from docs before raw API calls.

## WordPress, Boost.space, MCP

Bayarcash API docs link official WordPress plugins through `https://profiles.wordpress.org/webimpian/#content-plugins` and integration listings through `https://plugin.bayarcash.com/`.

User supplied `e-invoice-for-myinvois-lhdn` plugin URL. Treat it as a separate MyInvois/e-invoice plugin unless code or docs show direct Bayarcash integration.

User supplied Boost.space invite and Smithery MCP server URL. These may require browser/session or live MCP setup. Do not assume tool names, schemas, or auth. Inspect live page/server docs before writing automation against them.
