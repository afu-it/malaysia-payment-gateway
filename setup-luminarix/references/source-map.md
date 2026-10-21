# Luminarix Source Map

Use this map to pick the right source before answering or coding. All sources checked on 2026-10-21.

## Coverage

- Luminarix publishes no public developer docs. Checked: `https://www.lmnrx.com/`, `/docs`, `/llms.txt`, `/sitemap.xml` (all HTTP 403, Cloudflare bot check to automated fetch); `docs.`, `developer.`, `developers.`, `api.`, `sandbox.`, `portal.`, `dashboard.lmnrx.com` (no DNS record); `merchant.lmnrx.com` (resolves, Cloudflare challenge, login portal not readable). npm search `lmnrx`: no packages. Packagist `luminarix/*` packages belong to an unrelated Laravel vendor, not this gateway. GitHub search was rate-limited; no official repo is linked from any official source.
- Index used: WordPress.org plugin API for slug `luminarix-payment-gateway-for-woocommerce` (author profile `lmnrx`, "Luminarix Sdn. Bhd."), the only official technical source.
- Enumerated plugin files: 11. Read: 11 (`luminarix-woocommerce-gateway.php` 1523 lines in full, `readme.txt`, `uninstall.php`, `assets/js/blocks.js`, `assets/js/admin.js`, `assets/css/admin.css`, `LICENSE`, and 4 `index.php` stubs). All readme sections read (description, installation, FAQ, changelog 1.1.0 to 1.6.0).
- Endpoints found: 3 API endpoints plus 2 outbound flows (callback, return). All are in the catalogue in `luminarix-api.md`.
- Left out: WooCommerce-only internals (admin UI, Action Scheduler wiring, Blocks registration, order-meta keys) except where they show API behaviour.

## Plugin (Reference Implementation)

- Plugin page: `https://wordpress.org/plugins/luminarix-payment-gateway-for-woocommerce/`
- ZIP v1.6.0: `https://downloads.wordpress.org/plugin/luminarix-payment-gateway-for-woocommerce.1.6.0.zip`
- Metadata and readme: `https://api.wordpress.org/plugins/info/1.2/?action=plugin_information&request[slug]=luminarix-payment-gateway-for-woocommerce`

Use for: everything in `luminarix-api.md`.

File map inside `luminarix-woocommerce-gateway.php`:

- `Luminarix_WC_API::request()`: headers, JSON encoding, timeout, 500/503 retry on `INTERNAL_ERROR`/`PROVIDER_UNAVAILABLE`, 429 `Retry-After`, Cloudflare 403 detection, response envelope.
- `Luminarix_WC_API::checksum()`: request HMAC-SHA256 Base64 over `|`-joined parts.
- `Luminarix_WC_API::verify_callback()`: `X-Timestamp|rawBody` HMAC, tolerance window, constant-time compare.
- `Luminarix_WC_API::format_amount()`: 2-decimal ringgit string.
- `process_payment()` and `payer_from_order()`: deposit endpoint, body fields, deposit string-to-sign, payer object.
- `generate_api_order_id()`: per-attempt `orderId` format and 255-char limit.
- `extract_payment_instructions()`: instruction fields when no `checkoutUrl`.
- `handle_return()` and `finish_return()`: return query parameters and server-side confirmation.
- `handle_callback()` and `process_callback_event()`: callback headers, reference checks, dedupe, ack body, success re-verification, retry delays.
- `check_order_status()`: transaction-status endpoint, fields, string-to-sign.
- `process_refund()` and `apply_refund_status()`: refund endpoint, fields, string-to-sign, refund transaction tracking.
- `apply_status()`: status values, aliases, no-downgrade rule.
- `admin_test_connection()`: connection-test codes `TRANSACTION_NOT_FOUND`, `OK`, `VALIDATION_FAILED`.
- `init_form_fields()` and `method_row_fields()`: credentials, defaults (currency, timeout, tolerance, polling).

## Readme (readme.txt)

Use for: features list (sandbox/live separation, HMAC-SHA256 Base64, signed callback with timestamp and replay protection, partial refunds, per-method credentials), external-service data disclosure, installation steps, FAQ (credentials from onboarding, callback URL, environment switching), changelog facts (1.4.1 signature in `X-Signature` header and `orderId` in deposit signature; 1.4.4 JSON `SUCCESS` ack; 1.5.0 status verification before completion; 1.5.2 fresh `orderId` per attempt; 1.6.0 `countryCallingCode` required).

## Company Site

- Home: `https://www.lmnrx.com/`
- Pricing: `https://www.lmnrx.com/#pricing`
- Privacy policy: `https://www.lmnrx.com/agreements/Privacy%20Policy.pdf`

Use for: registration entry point, pricing link, settlement claim, product scope. The site blocks automated fetch; facts about plans and settlement come from a browser read recorded in the MPG listing (2026-10-21).

## Not Documented (checked 2026-10-21)

Base URLs, Payment System Code and channel code list, callback retry policy and source IPs, `X-Timestamp` format, full callback schema, error code list, rate limit numbers, sandbox test data, refund rules, payment expiry, cancel/capture/void, recurring billing, payment links API, tokenisation, payout and settlement report APIs.
