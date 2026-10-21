# BOLD.Pay Source Map

Use this map to pick the right source before answering or coding. All sources checked on 2026-10-21.

## Coverage

- No public developer docs exist. `https://www.boldpass.com/` is an Angular single-page portal behind login (OAuth via `auth.etracker.cc`); `/llms.txt`, `/docs`, `/api-docs` return the same login shell. Its main JS bundle was scanned and contains no API reference.
- `https://macrokiosk.com/sitemap.xml` lists marketing pages only; 4 BOLD.Pay pages read (product, payment channels, direct carrier billing, WooCommerce enquiry).
- Official plugin source read in full: WordPress.org `bold-pay` 1.7.1 (author MACROKIOSK): `boldpay.php` (163 lines), `includes/boldpaycore.php` (313), `includes/boldpay.php` (785), `includes/boldpayblocks.php` (57), `assets/js/boldpay-blocks.js` (25), `readme.txt` (91). 6 of 6 files read.
- Older tags compared: 1.0.0 and 1.5.9 (`src/boldpay.php`) for token and field changes. 19 tags exist (1.0.0 to 1.7.1).
- Searched, not found: GitHub org or repos, npm, Packagist, Postman, OpenAPI, PDF guide. OpenCart extension 39904 page blocked (Cloudflare) to automated fetch.
- Endpoints found: 2 BOLD.Pay endpoints (PaymentRequest, QueryRequest) plus 2 merchant-side receivers (callback, notification). All in the catalogue in `boldpay-api.md`.
- Left out: none of the plugin's integration logic. WooCommerce admin UI code is summarised only where it shows a setting.

## Product And Channels

- Product page: `https://macrokiosk.com/boldpay-payment-gateway`
- Payment channels: `https://macrokiosk.com/boldpay-payment-channels` (content loads client-side; only the heading is static)
- Direct carrier billing: `https://macrokiosk.com/boldpay-direct-carrier-billing` (same)

Use for: channel list (cards, online banking, e-wallets, BNPL, DCB), web interface / API / plugin options, plugin platforms.

## Registration And Support

- Register: `https://www.boldpass.com/register`
- Login: `https://www.boldpass.com/login`
- Enquiry: `https://macrokiosk.com/enquiry/woocommerce`
- Plugin `readme.txt` installation steps: `https://plugins.svn.wordpress.org/bold-pay/trunk/readme.txt`

Use for: approval flow, where the API key lives, Domain/IP Restrictions.

## Environments, Endpoints, Token

- `includes/boldpaycore.php`: `STAGING_BOLDPAY_URL`, `LIVE_BOLDPAY_URL`, `STAGING_BOLDPAY_QUERYURL`, `LIVE_BOLDPAY_QUERYURL`, `prepareAccessToken()`, `callQueryAPI()`, status constants, `getStatusCodeDescription()`, `getPaymentMethodDescription()`
- `https://plugins.svn.wordpress.org/bold-pay/trunk/includes/boldpaycore.php`

Use for: base URLs, MD5 AccessToken string order, query call and redirect parsing, status and error enums, channel codes.

## Payment Request Fields

- `includes/boldpay.php` `process_payment()`: `https://plugins.svn.wordpress.org/bold-pay/trunk/includes/boldpay.php`
- `includes/boldpay.php` `init_form_fields()`: settings, currency codes, default callback and notification URLs
- `boldpay.php`: phone validation, notification route `wc-api=boldpay_check_notification`

Use for: every PaymentRequest field, PayID format, CustIP source, amount format, currency codes.

## Callback, Notification, Requery, Settlement

- `includes/boldpay.php` `check_boldpay_response()` and `handleFailReturn()`

Use for: response fields, callback vs notification detection, `OK` reply, token recompute, QueryRequest field names, tamper checks, order status mapping, error-variant handling.

## Version History

- `https://plugins.svn.wordpress.org/bold-pay/tags/1.0.0/src/boldpay.php`
- `https://plugins.svn.wordpress.org/bold-pay/tags/1.5.9/src/boldpay.php`
- Changelog in `readme.txt`

Use for: token used `CallbackURL` twice before 1.6.0; separate `NotificationURL` added in 1.6.0; Block Checkout from 1.7.0.
