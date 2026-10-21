# Tulus Source Map

Use this map to pick the right official source before answering or coding. All checked 2026-10-21. `developers.tulus.my` and `developers.snapnpay.my` serve the same portal.

## Product and Onboarding

- Gateway page: `https://tulus.my/payment-gateway/`
- Joining form page: `https://tulus.my/sertai-kami/`
- Contact and enquiry form: `https://tulus.my/hubungi-kami/`
- Partners list: `https://tulus.my/kerjasama/`
- Site map: `https://tulus.my/sitemap.xml`

Use for: who Tulus is, PayNet payment exchange status, channels (FPX, cards), onboarding contact, settlement-by-agreement wording, partner institutions.

## Developer Portal Overview

- Home and Quick Start: `https://developers.tulus.my/`
- Interactive guide: `https://developers.tulus.my/how-it-works/`
- Official agent skill: `https://developers.tulus.my/SKILL.md`

Use for: two-host architecture, payment flow, auth methods, signing steps, code recipes, gotchas. Note: their "POST has no path" statement conflicts with the SDK and Laravel handoff; see `tulus-api.md`.

## Checkout and Signing

- Laravel handoff (rendered `/laravel`): `https://developers.tulus.my/laravel-handoff.md`
- SDK playground: `https://developers.tulus.my/sdk/`
- PHP SDK: `https://developers.tulus.my/download/sdk.zip` (`Tulus.php` `SignURL`, `SignMessage`; `SnapNPay.php`; `index.php`; `payment.php`; `example/receipt.php`; `example/update/update.php`; `example/report/*`)

Use for: checkout fields, `cancelUrl`, UTM and preference fields, environments (`pay.tulus.my`, `paydev.tulus.my`), `exp`, signature message with path, encoding rules, official test vectors, 403 behaviour, storage and idempotent settlement pattern, reconciliation job, `12.34` test amounts.

## Callbacks

- Callback reference (rendered `/sdk/callback`): `https://developers.tulus.my/sdk/callback.md`
- MPGS card callback example (rendered `/mpgs`): `https://developers.tulus.my/mpgs_callbackresponse.md`

Use for: direct vs indirect, field tables, `fpx` JSON fields, status mapping, full `fpx_debit_auth_code` table, verification steps, retry cycle, timestamp formats, card callback fields.

## Reports and QR

- SDK source `Tulus.php` (endpoint list, report params, `fpxrequest`, transactions, QR generators, version/echo)
- QR generator page: `https://developers.tulus.my/sdk/qrcode/`

Use for: Report API params and response shape, status requery, QR fields, signed vs unsigned QR.

## Zakat and Schemas

- Zakat page: `https://developers.tulus.my/zakat/`
- Schemas index: `https://developers.tulus.my/schema/`
- `https://developers.tulus.my/schema/PPZMAIWP.schema.json`, `PPZMAIWP.example.json`, `ZKLZS.schema.json`, `ZKLZS.example.json`, `report.schema.json`, `JHRFITRAH.json`, `ZKLZS.json`

Use for: zakat agency codes, fitrah rate files, required zakat form fields, report schema.

## Partner and Customisation

- ADB webhook: `https://developers.tulus.my/adb/`, schema `https://developers.tulus.my/schema/adb.json`
- Grab webhook (draft): `https://developers.tulus.my/grab/`, schema `https://developers.tulus.my/schema/grab.json`
- Custom receipts: `https://developers.tulus.my/receipt/`
- Receipt email customisation: `https://developers.tulus.my/customemail/`

Use for: partner inbound transaction pushes and receipt/email customisation only.

## Plugin

- WooCommerce plugin: `https://developers.tulus.my/download/woocommerce-gateway-snapnpay.zip` (v0.10.1, WC tested 3.8)

Use for: historical field usage only (`agency`, `refNo`, `amount`, `email`, `returnUrl`, path-included signing). Its callback signature check is disabled and it does not send `exp`; do not copy its verification.

## Coverage

- Enumerated: no sitemap or llms.txt on the portal (both 404), so pages were enumerated from the portal navigation and every in-page link. 13 portal HTML pages (home, how-it-works, sdk, sdk/qrcode, sdk/callback, mpgs, laravel, schema, zakat, adb, grab, receipt, customemail), 4 markdown sources those pages load (SKILL.md, callback.md, mpgs_callbackresponse.md, laravel-handoff.md), 9 schema/JSON files, the SDK zip (15 files), the WooCommerce zip (4 code files), and 4 tulus.my product pages. Read: all 30 pages/documents plus the two archives.
- Also probed: `developer.tulus.my` and `dev.tulus.my` (SnapNPay login pages, gated, internal), `reports.tulus.my` (merchant login), GitHub, npm, Packagist and WordPress.org (no official Tulus/SnapNPay packages found).
- Left out: the SDK's `woocommerce.php` (a WooCommerce REST client for listing customers, not payment acceptance), ColdFusion example, and the customemail `endpoint.php` sample output. Fitrah JSON files beyond `JHRFITRAH` and `ZKLZS` were only status-checked; several linked codes return 404 (see `tulus-codes.md`).
- Not documented anywhere checked: refunds, cancel/void, recurring, tokenisation, payouts, settlement report API, rate limits, IP allowlist, public staging credentials, onboarding documents, pricing.
