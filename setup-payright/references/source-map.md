# Payright Source Map

Use this map to pick the right official source before answering or coding. All checked 2026-10-21.

## Coverage

- Index used: `https://docs.payright.my/llms.txt` (23 pages) cross-checked against `https://docs.payright.my/sitemap-pages.xml` (23 URLs).
- Pages enumerated: 23. Pages read: 23 (full text via `https://docs.payright.my/llms-full.txt`, and each page's `.md` version compared line by line; no extra content in the per-page files).
- Also read: the official Postman collection zip linked from the docs (7 requests with sample responses, plus sandbox environment file), and the three official `PayrightMY` GitHub repos (legacy, see below).
- No OpenAPI/Swagger spec is published. Card, e-wallet, BNPL, refund, recurring and settlement-report APIs are not in the public docs; nothing was left out for being global-only.

## Overview And Setup

- Introduction: `https://docs.payright.my/introduction`
- Endpoints (base URLs): `https://docs.payright.my/endpoints`
- Sandbox: `https://docs.payright.my/sandbox`
- Product site: `https://www.payright.my/en`
- Pricing: `https://www.payright.my/en/pricing`
- Integrations: `https://www.payright.my/en/integrations`
- Onboarding: `https://onboard.payright.my/register`
- Merchant portal: `https://portal.payright.my/`

Use for: base URLs, sandbox behaviour, product positioning, plan and payment method claims, registration.

## Authentication And Signing

- Authentication: `https://docs.payright.my/authentication`
- Signature Validation: `https://docs.payright.my/signature-validation`

Use for: `auth-token` header, Auth Token location, JWT Verification toggle, `x-signature` HS256 payload (`auth-token`, `http_method`, `url_path`, `iat`, `exp`).

## Errors

- Error Codes: `https://docs.payright.my/error-codes`

Use for: HTTP codes, body `code` values (401, 403, 404, 425, 500, 6666, 7777, 9100, 10018, 10041, 10149, 10201), payout duplicate rule.

## Collection And Bill (checkout)

- Payment Provider: `https://docs.payright.my/api/payment-provider`
- Collection: `https://docs.payright.my/api/collection`
- Create Collection: `https://docs.payright.my/api/collection/create-collection`
- Get Collection: `https://docs.payright.my/api/collection/get-collection`
- Update Collection: `https://docs.payright.my/api/collection/update-collection`
- Get All Collections: `https://docs.payright.my/api/collection/get-all-collections`
- Bill: `https://docs.payright.my/api/bill`
- Create Bill: `https://docs.payright.my/api/bill/create-bill`
- Get Bill: `https://docs.payright.my/api/bill/get-bill`

Use for: FPX bank list and `provider_id`, collection lifecycle, bill request fields, `payment_url`, status requery.

## Callbacks

- Bill Callback: `https://docs.payright.my/api/bill/callback`
- Payout Callback: `https://docs.payright.my/api/payout/callback`

Use for: `{"data": JWT, "response_code"}` body, Hash Key location, HS256 decode, decoded field lists.

## Payouts And Balance

- Payout overview: `https://docs.payright.my/api/payout`
- Payout Provider: `https://docs.payright.my/api/payout-provider`
- Create Payout: `https://docs.payright.my/api/payout/create-payout`
- Get Payout: `https://docs.payright.my/api/payout/get-payout`
- Get Balance: `https://docs.payright.my/api/get-balance`

Use for: disbursement flow, payout request fields, payout status values, pool balances.

## Postman Collection

- Page: `https://docs.payright.my/postman-collection`
- File: Google Drive zip linked from that page, containing `Payright API (Sandbox).postman_collection.json` and `PayrightSandbox.postman_environment.json`.

Use for: confirming the `auth-token` header name, sandbox base URL, `invoice_ref` in Create Bill, and the full sandbox provider list (IDs 16 to 86).

## Operations And Support

- Contact: `https://www.payright.my/en/contact`
- Status: `https://www.payright.my/en/system/status` (redirects to `https://payright.instatus.com/`)
- Refund Policy: `https://www.payright.my/en/refund-policy`
- Terms: `https://www.payright.my/en/terms-and-conditions`

Use for: support routes, incidents, policy documents. Not payment truth.

## Legacy Sources (recognise, do not copy)

- `https://github.com/PayrightMY/payment-php` (PHP client, 2022)
- `https://github.com/PayrightMY/payright-for-woocommerce` (WooCommerce plugin 1.0.3, 2022; WordPress.org listing closed 2023-07-24)
- `https://github.com/PayrightMY/Payright-For-GravityForms` (2022)
- Linked from `https://www.payright.my/en/developer/github`.

Use for: identifying old integrations that call `https://payright.my/api/v1` with `Authorization: Bearer`, sen amounts and an HMAC `signature` field. These do not match the current API.

## Not Payright Malaysia

- `https://github.com/payright-engineering` and `payright.com.au` are Payright Australia (BNPL). Ignore for Malaysian FPX work.

## Known Gaps (not documented, checked 2026-10-21)

Refunds, bill cancel/expiry, full status enum, callback retries and failure callbacks, redirect query parameters, rate limits, IP allowlists, test bank credentials for the simulator, card/e-wallet/BNPL via API, recurring billing API, settlement report API, sandbox account signup URL.
