# FavePay Source Map

Use this map to pick the right source before answering or coding. All checked 2026-10-21.

## Coverage

- Index used: `https://developers.myfave.com/` (Fave Partner Documentation). It lists two guides: FavePay Omni (FPO) API and Deals Marketplace API. No `llms.txt`, sitemap, OpenAPI or Postman collection is published on that host (all return 404).
- Enumerated: 2 guide pages plus the index (3 pages). The FPO guide is one long page with 22 sections (overview, auth, UAT, flows, QR formats, request QR, omni_ref redirect, static QR, merchant scan, webhooks, accepting webhooks, get transaction, acknowledge, cancel, list, refund, IDs, payment types, payload verification, errors, support, test parameter).
- Read in full: the index and the FPO guide (all 22 sections). The Deals Marketplace guide was enumerated and skimmed for scope only.
- Official plugin: FavePay for WooCommerce v1.0.17, all 9 PHP files and the readme read.
- Left out: Deals Marketplace API (voucher reseller listings and reservations, not payment acceptance).
- No official Fave SDK on GitHub, npm or Packagist was found (the GitHub `fave` org is unrelated).

## Product And Market Overview

- FaveBiz home: `https://www.favebiz.com/`
- FaveBiz payments page: `https://www.favebiz.com/services/payments/`
- Fave consumer site index: `https://myfave.com/llms.txt`

Use for: company and product positioning, merchant signup links, markets.

## Partner Docs Index

- `https://developers.myfave.com/`

Use for: list of official partner guides, note that staging credentials are provisioned by Fave during onboarding.

## Authentication And Signing

- `https://developers.myfave.com/fpo-guide.html#auth`
- `https://developers.myfave.com/fpo-guide.html#payload-verification`
- Plugin `includes/abstracts/abstract-favepay-wc-client.php` (`generate_api_signature`, `validate_ipn_response`)

Use for: HMAC-SHA256 `sign`, excluded fields, field order, bracket flattening, `+` for spaces, PHP `http_build_query` encoding behaviour, worked webhook example.

## Environments And UAT

- `https://developers.myfave.com/fpo-guide.html#overview`
- `https://developers.myfave.com/fpo-guide.html#uat`
- Plugin `abstract-favepay-wc-client.php` (`PRODUCTION_URL`, `SANDBOX_URL`), `includes/functions.php` (country and currency map)

Use for: base URLs, country path, sandbox credentials list, test mode, UAT cases and sign-off, FaveBiz sandbox Staff Mode.

## Payment Flows And Formats

- `https://developers.myfave.com/fpo-guide.html#flows`
- `https://developers.myfave.com/fpo-guide.html#qr-formats`

Use for: dynamic online, dynamic offline and static QR sequences, choosing `format`, auto-acknowledge behaviour.

## Create Payment

- `https://developers.myfave.com/fpo-guide.html#qr-api` (includes Use of test parameter)
- `https://developers.myfave.com/fpo-guide.html#qr-redirect-omni-ref`
- `https://developers.myfave.com/fpo-guide.html#merchant-scan`
- `https://developers.myfave.com/fpo-guide.html#static-qr-api`
- Plugin `includes/regular-checkout.php` (`process_payment`, `get_qr_code_params`)

Use for: request and response fields, amount in cents, `omni_reference` rules, `test=true`, encrypted `omni_ref` redirect, merchant scan, static QR status.

## Webhooks

- `https://developers.myfave.com/fpo-guide.html#webhook`
- `https://developers.myfave.com/fpo-guide.html#webhook-accept`
- Plugin `regular-checkout.php` (`handle_ipn_callback`, `thankyou_page`)

Use for: callback body fields, statuses, 2XX acceptance, retries, idempotency, WooCommerce redirect format.

## Lookup, Acknowledge, Cancel, List

- `https://developers.myfave.com/fpo-guide.html#txn-api`
- `https://developers.myfave.com/fpo-guide.html#ack-api`
- `https://developers.myfave.com/fpo-guide.html#cancel-api`
- `https://developers.myfave.com/fpo-guide.html#history-api`
- Plugin `includes/class-favepay-wc-api.php`, `regular-checkout.php` (`check_transaction_status`)

Use for: requery fallback, acknowledging static QR or missed webhooks, cancelling unpaid QRs, listing unacknowledged payments.

## Refunds

- `https://developers.myfave.com/fpo-guide.html#refund-api`
- Plugin `regular-checkout.php` (`process_refund`)

Use for: full and partial refunds, same-day window, `stale_transaction`, already-refunded behaviour.

## Reference Data And Errors

- `https://developers.myfave.com/fpo-guide.html#ids`
- `https://developers.myfave.com/fpo-guide.html#payment-types`
- `https://developers.myfave.com/fpo-guide.html#errors`
- `https://developers.myfave.com/fpo-guide.html#support`

Use for: status codes, ID prefixes, payment type values, error table, support path.

## Account Setup And Plugin

- Merchant signup: `https://app.favebiz.com/sign-up?ref=favebiz`
- Merchant help centre: `https://support.favebiz.com/support/home`
- WooCommerce plugin page: `https://wordpress.org/plugins/favepay-for-woocommerce/`
- Plugin source: `https://downloads.wordpress.org/plugin/favepay-for-woocommerce.latest-stable.zip`

Use for: onboarding steps, integration contact emails, plugin settings, markets, refund behaviour in WooCommerce, `omni_reference` generation pattern.

## Pricing

Not published on official Fave sites (checked 2026-10-21). Ask Fave sales.
