# Riipay Source Map

Use this map to pick the right official source before answering or coding. All sources checked 2026-10-21.

## Coverage

- Index used: `https://developers.riipay.my/llms.txt` (4 pages) cross-checked with `https://developers.riipay.my/sitemap-pages.xml` (4 URLs). Enumerated 4, read 4 (via the `.md` versions).
- Guide source repo: `https://github.com/riipay/riipay-gitbook` (official `riipay` GitHub account, last commit 2021-10-25). Read all 6 markdown files (`README.md`, `SUMMARY.md`, `overview.md`, `getting-started.md`, `endpoints/payment-request.md`, `endpoints/transaction-requery.md`). Content matches the live guide.
- User guide: `https://docs.riipay.my/llms.txt` (2 pages) and `sitemap-pages.xml` (2 URLs). Enumerated 2, read 2.
- Official plugin: WordPress.org `riipay-for-woocommerce` v1.0.27. Read `index.php`, `src/riipay.php`, `src/riipay-surcharge.php`, `src/riipay-script.js`, `readme.txt`, `README.md`, `uninstall.php`; `src/riipay-theme.php` scanned (product price display only).
- Supporting: merchant FAQ JSON (`https://api.riipay.my/website/faq/merchant`, 6 Q&As), website config bundle (sign-up URL, contacts), business page strings.
- Endpoints documented: 2 Riipay endpoints plus 2 merchant-side receivers (return, callback); 1 extra plugin-only endpoint (`/v1/payment/preview`). All are in the catalogue in `riipay-api.md`.
- Signature samples in the guide recomputed with MD5 and confirmed.

## Developer Guide (primary)

- Index: `https://developers.riipay.my/llms.txt`
- Introduction: `https://developers.riipay.my/master.md`
- Getting Started: `https://developers.riipay.my/getting-started.md`
- Payment Request: `https://developers.riipay.my/endpoints/payment-request.md`
- Transaction Requery: `https://developers.riipay.my/endpoints/transaction-requery.md`

Use for: payment flow, staging and production URLs, portal settings (Return URL, Callback URL, Callback Method, bank account), Merchant Code and Secret Key, payment request fields, request signature, return vs callback, response fields, status codes, error codes, response signature, requery fields, query signature, rate limit.

## Guide Source Repo

- `https://github.com/riipay/riipay-gitbook`

Use for: confirming the guide text and history; same facts as the developer guide.

## Customer Payment Experience

- `https://docs.riipay.my/merchant/payment-experience`
- `https://docs.riipay.my/master.md`

Use for: shopper steps (phone number, OTP, new-user details, card entry), confirmation emails after 5 to 10 minutes, failed payment redirect to store.

## Official WooCommerce Plugin

- Listing: `https://wordpress.org/plugins/riipay-for-woocommerce/`
- Download: `https://downloads.wordpress.org/plugin/riipay-for-woocommerce.1.0.27.zip`
- Plugin info API: `https://api.wordpress.org/plugins/info/1.0/riipay-for-woocommerce.json`

Use for: real-world field values sent on every request, response signature using stored order values, `F` error codes that must not fail the order, `410` redirect, `A` on-hold and `S` complete mapping, idempotency guard, callback HTTP 200 JSON reply, MYR-only, RM1000 maximum note, preview endpoint, portal profile URLs, suggested return and callback URLs, changelog.

## Merchant And Account

- Business page: `https://riipay.my/business/`
- FAQ page: `https://riipay.my/faq` (data: `https://api.riipay.my/website/faq/merchant`)
- Sign-up: `https://secure.riipay.my/merchant-sign-up`
- Portals: `https://merchant.riipay.my`, `https://merchant.uat.riipay.my`

Use for: registration, coexisting with other gateways, fee model (contact sales), refunds policy, integration time, contacts.

## Checked But Not Found

- `https://riipay.my/llms.txt`, `/sitemap.xml`: SPA fallback HTML, no index.
- `developer.riipay.my`: no DNS response. `riipay.gitbook.io` redirects to `docs.riipay.my`.
- Old guide paths on `docs.riipay.my/getting-started` and `/endpoints/*`: 404 (moved to `developers.riipay.my`).
- npm, Packagist: no Riipay packages. GitHub: only `riipay/riipay-gitbook` under the official account; the README's `riipay/riipay-wordpress-plugin` link is not public. Third-party mirrors ignored.
- Not documented anywhere checked: refund API, callback retry policy, callback IPs, sandbox test cards, payment expiry, requery by merchant reference, settlement timing, required onboarding documents.
