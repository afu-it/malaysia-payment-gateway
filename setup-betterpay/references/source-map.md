# Betterpay Source Map

Use this map to pick the right source before answering or coding. All docs links are anchors on one page: `https://www.betterpay.me/docs/` (title "QlicknPay APIs V1"). Checked 2026-10-21.

## Product, Brand, Pricing

- Website: `https://www.betterpay.me/`
- Pricing: `https://www.betterpay.me/v2/pricing.html`
- Docs Introduction: `https://www.betterpay.me/docs/#introduction`

Use for: QlicknPay to Betterpay rename ("All API endpoints using the TLD Betterpay.me or qnp.com is valid"), HTTPS-only rule, "all requests must be hashed", plan names, settlement timing. Never copy fee numbers.

## Auth, Rate Limit, Hashing

- Rate Limit: `https://www.betterpay.me/docs/#rate_limit`
- PRO hashing guides: inside each PRO section, e.g. `#standard_payment_gateway1`
- LITE Hashing Guide (HMAC-SHA256): `https://www.betterpay.me/docs/#bplite_status_code` (contains the Hashing Guide)
- Invoice HMAC guide: `https://www.betterpay.me/docs/#invoiceHashing`
- Official Postman collections (pre-request hash scripts, sandbox paths): `https://github.com/betterpayme/POSTMAN-APIs`
  - `BP PRO APIs`, `BP LITE APIs`, `BP PRO APIs - DuitNow QR (OCBC)`, `BP PRO APIs - Card Pre-Authentication`
  - `Recurring (BP PRO)`, `Recurring (BP LITE)`, `Tokenization (BP PRO)`, `Tokenization (BP LITE)`
- Official WooCommerce plugin source: `https://wordpress.org/plugins/betterpay/` (`src/Betterpay.php`, v1.2.4)

Use for: MD5 pipe orders, HMAC sort/concat/strip-spaces rule, nested-object hashing, LITE callback verification, PRO/LITE detection (`R` in merchant ID), production hosts (`www.betterpay.me`, `lite.betterpay.me`), rate limit and 429 body.

## PRO Payment Acceptance

- Standard Payment (hosted checkout, callback, FPX response codes, error guide): `https://www.betterpay.me/docs/#standard_payment_gateway1`
- Create Bill: `#create_bill1`. Remove Created Bill: `#create_bill_remove1`
- Direct Payment (bank codes, DuitNow QR, BNPL, IPP): `#direct_payment1`
- Get Bank List: `#bank_list1`
- Get Payment Status (requery): `#requery1`

Use for: request fields, payment_method and bank codes, `respond=1` JSON, DuitNow QR fields, callback fields and MD5 verification, `OK` reply and retry rule, requery fields and codes.

## LITE Payment Acceptance

- Standard Payment: `#bplite_standard_payment_gateway1`
- Direct Payment: `#bplite_direct_payment_gateway1`
- Callback: `#callback_response1`
- Get Payment Channels: `#bplite_channels1`
- Get Payment Status: `#bplite_requery1`
- Status codes and Hashing Guide: `#bplite_status_code`
- Get Settlement (LITE): `#settlementbplite1`
- WooCommerce (LITE): `#plugins_woocommerce_bplite`

Use for: LITE fields (buyer fields required, `hostedCard`), channel list, LITE status codes, HMAC.

## Refunds

- FPX Refund API: `#refund1`
- DuitNow QR (OCBC) Refund API: `#dnqrrefund1`
- Get Refund Status: `#re_refund1`

Use for: own-FPX-seller-ID rule, 10 per request and 5-day limits, refund_list formats, refund callback hash, refund codes.

## Reports And Settlement

- Get Transactions: `#transactions`. Get Transaction by invoice: `#get_transaction1`
- Get Settlement: `#settlement1`. Get Callbacks: `#callbacks1`. Get Balance: `#balance1`. Get Disburse Records: `#disburse1`

Use for: reconciliation fields, callback delivery log, disburse T+3 and sandbox limits.

## Payment Links And Invoices

- PRO Payment Link: `#payment_link1`, `#payment_link4`, `#payment_link5`, `#payment_link6`
- LITE Payment Link: `#bplite_payment_link1`, `#bplite_payment_link4`, `#bplite_payment_link5`, `#bplite_payment_link6`
- Invoice API: `#invoice1`, `#invoice2`, `#invoice3`, `#invoice4`, `#invoiceHashing`

Use for: link fields and responses, invoice items/tax/reminders format.

## Recurring

- PRO Direct Debit (e-Mandate): `#recurring1`, `#recurring2`, `#recurring2Update`, `#recurring3` to `#recurring8`, callback `#recurring9`
- OCBC collection: `#batchrecurring1`, `#batchrecurring2`, `#batchrecurring4`, `#batchrecurring6`, `#batchrecurring10`, `#batchrecurring8`, `#batchrecurring9`, callback `#batchrecurring11`
- LITE card recurring: `#bplite_recurring1`, `#bplite_recurring2`, `#bplite_recurring2-v2`, `#bplite_recurring3` to `#bplite_recurring7`

Use for: package types, subscriber fields, two-step approval callbacks, collection date rules, batch limits, collection status list.

## Tokenization And Pre-authorization

- PRO tokenization: `#mpgs_tokenization1`, `#mpgs_tokenization2`, `#mpgs_tokenization6`, `#mpgs_tokenization7`, `#mpgs_tokenization10`, `#mpgs_tokenization13`, `#mpgs_tokenization16`, `#mpgs_tokenization19`, `#mpgs_tokenization22`
- MPGS error codes: `#mpgs_tokenization25` (links `./error_code_mpgs/error.json`)
- Token pre-auth (Authorize, Capture, Void): `#pre_auth_tokenization`, `#capture_card_tokenization`, `#void_auth_tokenization`
- Batch charge: `#batch_mpgs_tokenization1` to `#batch_mpgs_tokenization5`
- Card Pre-authorization: `#mpgs_preauth1`, `#mpgs_preauth2`, `#mpgs_preauth_callback`, `#mpgs_preauth_pay`, `#mpgs_preauth_void`, `#mpgs_preauth_transactions`
- LITE tokenization and batch: `#tokenization1` to `#tokenization22`, `#batch_tokenization1` to `#batch_tokenization5`

Use for: verification URL flow, token charge rules (5-minute gap, 1.50 minimum), sandbox simulation fields, pre-auth expiry and USD 2 / 48h void rule, batch formats.

## Plugins And Channel Setup

- Plugins: `#plugins_opencart`, `#plugins_prestashop`, `#plugins_woocommerce`, `#plugins_ecwid`, `#plugins_drupal`, `#plugins_easystore`, `#plugins_magento`
- Channel setup: `#paypal`, `#bnpl`, `#2c2p`, `#third_parties` (Lazada/Shopee sync, not payments)
- WordPress plugin page: `https://wordpress.org/plugins/betterpay/`

Use for: install steps, dashboard settings for BNPL/2C2P/PayPal, EasyStore sandbox flag.

## Coverage

- Index used:
  - The docs have no `llms.txt`, `sitemap.xml` or `robots.txt` (all return the 404 page, checked 2026-10-21).
  - The docs are a single HTML page. Its section headings were the index: 115 anchored sections (h1 + h2), 80 PRO/shared and 35 LITE.
  - Read: 115 of 115 sections.
- Official GitHub repo `betterpayme/POSTMAN-APIs`: 8 of 8 collections and the README read (all request bodies and pre-request scripts).
- Official WooCommerce plugin: 5 of 5 source files read (`index.php`, `class-block.php`, `checkout.js`, `src/Betterpay.php`, `readme.txt`). Sandbox and production hosts were confirmed from it.
- Pricing page read for plan names and settlement timing only.
- Left out:
  - Indonesia/Thailand/Philippines specifics beyond field enums (`indo_bank`, `agent_channel_code`, IDR/THB/PHP currencies). They are listed but not expanded, because this skill targets Malaysian merchants.
  - Lazada/Shopee product sync (not payment acceptance).
- Gaps (behind login): production endpoint URLs for every API except the plugin endpoints; sandbox paths for operations missing from Postman (marked "Not published").
