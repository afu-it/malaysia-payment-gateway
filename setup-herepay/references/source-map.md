# Herepay Source Map

Use this map to pick the right source before answering or coding. All pages checked 2026-10-21. Append `.md` to any `herepay.readme.io/reference/...` URL to get the markdown version.

## Docs Index

- Docs index for agents: `https://herepay.readme.io/llms.txt`
- Introduction: `https://herepay.readme.io/reference/integrating-with-herepay-api-payment`

Use for: page list, product summary, supported methods (FPX, cards, e-wallets per intro text).

## Onboarding And Environments

- Getting Started: `https://herepay.readme.io/reference/getting-started-1`
- Environment: `https://herepay.readme.io/reference/introduction`
- Obtain API Key: `https://herepay.readme.io/reference/how-to-get-your-api-key`
- Managing URLs: `https://herepay.readme.io/reference/callback-and-redirect-urls`

Use for: registration steps, UAT and production base URLs, key separation, where the API token is generated, callback and redirect URL setup, support contact.

## Auth And Headers

- Headers: `https://herepay.readme.io/reference/headers-1`
- Common headers: `https://herepay.readme.io/reference/headers`
- Authentication Header: `https://herepay.readme.io/reference/common-headers-copy`

Use for: `SecretKey` and `XApiKey` headers, `Content-Type`, backend-only key handling.

## Payment Flow And Creation

- Transaction Flow: `https://herepay.readme.io/reference/p`
- V1 overview: `https://herepay.readme.io/reference/v1`
- Get Payment Channels: `https://herepay.readme.io/reference/get_api-v1-herepay-payment-channels`
- Initiate Invoice: `https://herepay.readme.io/reference/post_api-v1-herepay-initiate`
- Mapping Table: `https://herepay.readme.io/reference/banks`

Use for: step-by-step FPX flow, channel response shape, `bank_prefix` and `payment_method` values, initiate request fields and limits, "Checksum verification failed" behaviour, bank code reference.

## Checksum

- What is Checksum: `https://herepay.readme.io/reference/what-is-checksum`
- Generating Checksum: `https://herepay.readme.io/reference/generating-checksum`
- Parent page: `https://herepay.readme.io/reference/checksum`

Use for: sort-by-key, JSON-encode nested values, comma join, HMAC-SHA256 hex with the Private Key, code in PHP, Java, Python, JavaScript, C#.

## Callback, Redirect, Status

- Callback: `https://herepay.readme.io/reference/post_callback`
- Status Codes: `https://herepay.readme.io/reference/error-codes`
- HTTP Status Codes: `https://herepay.readme.io/reference/http-status-codes`
- Success: `https://herepay.readme.io/reference/success`
- Client Errors: `https://herepay.readme.io/reference/client-error`
- Server Errors: `https://herepay.readme.io/reference/server-error`

Use for: callback field list, `status` / `status_code` values (and the `0/1/2` vs `00/29/30` mismatch), HTTP error meanings.

## Requery

- Get Transactions: `https://herepay.readme.io/reference/get_api-v1-herepay-transactions`
- Get Transactions By Reference Code: `https://herepay.readme.io/reference/get_api-v1-herepay-transactions-reference-code`

Use for: status fallback by `reference_code`, transaction list fields, 404 shape, rate limit headers in examples.

## Settlement And Pricing

- Settlement: `https://herepay.readme.io/reference/settlement-fee`
- Pricing: `https://www.herepay.org/harga/`
- FAQ: `https://www.herepay.org/soalan-lazim/`
- Payment gateway page: `https://www.herepay.org/gerbang-bayaran/`
- Help centre: `https://www.herepay.org/pusat-bantuan/`

Use for: absorbed-fee model, settlement timing, registration requirements (organisation bank account, SSM), pricing link only.

## Official Code

- Resources page: `https://herepay.readme.io/reference/resources`
- Integration samples: `https://github.com/Herepay/herepay-integration-samples`
  - `php/README.md`: direct-api vs payment-link comparison, status code summary.
  - `php/direct-api/submit-payment.php`: initiate with form-urlencoded body, optional `redirect_url`.
  - `php/direct-api/callback.php`: callback checksum verification with `hash_equals`.
  - `php/direct-api/transaction-status.php`: requery by `reference_code`.
  - `php/payment-link/create-payment-link.php`: payment link fields and `data.pay_url`.
  - `nodejs/index.js`: Node checksum and initiate.
- PHP SDK: `https://github.com/Herepay/herepay-php-sdk` (env var names; note the `openssl_pkey_get_private` quirk in `generateChecksum`).
- WooCommerce plugin: `https://github.com/Herepay/herepay-for-woocommerce` (credential names, dashboard webhook step, callback and redirect handling, refunds via dashboard).

Use for: details not on readme.io: payment link API, `redirect_url`, content type used in practice, callback verification code, env var naming.

## Codes And Enums

- Mapping Table: `https://herepay.readme.io/reference/banks`
- Get Payment Channels example: `https://herepay.readme.io/reference/get_api-v1-herepay-payment-channels`
- Status Codes, Callback, HTTP status pages (listed above)

Use for: `references/herepay-codes.md` (B2C and B2B bank codes, UAT test banks, status and callback enums, HTTP codes, defensive status mapping).

## Coverage

Checked 2026-10-21 (completeness top-up).

- Index used: `https://herepay.readme.io/llms.txt` (re-fetched 2026-10-21, unchanged). It lists 26 linked pages; all 26 were downloaded as markdown and read. Two index entries ("API Reference", "Callback") are unlinked group headings, not pages.
- OpenAPI: the per-endpoint OpenAPI blocks on the 4 V1 pages, the Callback page, and the combined `v1` page were parsed. They define exactly 4 V1 operations plus the merchant-side `/callback`. No other paths exist in the published spec.
- `llms-full.txt`: read; it is the concatenated content of the same pages, nothing extra. The saved `sitemap.xml` download was empty, so `llms.txt` is the only enumeration index.
- Official code read in full: `Herepay/herepay-integration-samples` (PHP direct-api 7 files, PHP payment-link 3 files, Node.js `index.js`, READMEs), `Herepay/herepay-php-sdk` (`src/` 5 files, tests, README), `Herepay/herepay-for-woocommerce` (`init.php`, main plugin file, 3 `includes/` classes, JS assets, README, readme.txt, CHANGELOG, `INTEGRATION_COMPLETE.md`).
- Marketing/help pages read for account setup: `herepay.org` home, `/harga/`, `/gerbang-bayaran/`, `/soalan-lazim/`, `/pusat-bantuan/`.
- Pages enumerated: 26 docs pages + 3 repos + 5 site pages. Pages read: all.
- Left out: nothing developer-facing. Plugin-internal WordPress code (admin UI, blocks registration, i18n) is not repeated beyond what affects the API contract.

## Corrections And Additions In Top-up (2026-10-21)

- Added Endpoint catalogue and full response field tables for Get Payment Channels, Get Transactions, and Get Transaction By Reference Code to `herepay-api.md`.
- Added `herepay-codes.md` with the full Mapping Table bank codes (previously only test banks were listed).
- New finding: the WooCommerce plugin requeries `/api/v1/herepay/transactions/{payment_code}` (`init.php` `checkTransactionStatus`), while readme.io, SDK, and samples use `{reference_code}`. Documented as unconfirmed; no existing content was wrong.
- New finding: transaction list examples return `payment_method` `FPX`, not the `Online Banking` string used in initiate.
- New finding: the WooCommerce callback handler signs a fixed 12-field whitelist instead of all posted fields.
