# ShopeePay Source Map

Use this map to pick the right official source. All pages checked 2026-10-21.

## Coverage

- Index used: the site navigation of `https://product.shopeepay.com` (no `llms.txt`, `robots.txt`, or public `sitemap.xml`: `/llms.txt` and `/robots.txt` return 404, `/sitemap.xml` returns 403). Pages were fetched server-rendered and nested parameter tables were extracted from the embedded page data, including collapsed "Child parameters".
- Enumerated: 33 pages (16 integration pages, 17 product pages) plus the official WooCommerce plugin source and 2 `shopeepay.com.my` merchant pages.
- Read: all 16 integration pages in full, 13 product pages in full, the 4 SPayLater widget pages skimmed (out of scope, see below), the plugin source (`class-shopeepay-api.php`, `class-shopeepay-webhook.php`, `class-shopeepay-countries.php`, `class-wc-gateway-shopeepay.php`, `readme.txt`), and both `shopeepay.com.my` pages.
- Left out on purpose: SPayLater Promotional Messaging widget install details (Android `.aar` SDK, iOS ShopeePayKit, web `loader.js`) and the Shopify SPayLater On-Site Messaging app. They are marketing widgets, not payment APIs; they are covered by the separate SPayLater skill. Non-Malaysia market specifics (IDR/VND/THB/PHP/SGD limits, ID bank transfer codes, PH Maribank, TH/VN co-funding) are noted only where they change Malaysian behaviour.
- The site's "Code Examples" tab shows the same request/response samples as the Documentation tab.

## Getting Started, Auth, Signature

- Get Started: `https://product.shopeepay.com/integration/get-started`

Use for: onboarding steps, Client ID and Secret Key, access node hosts per region, protocol rules, empty-value rules, HMAC-SHA256 Base64 signature, test vector, `X-Airpay-ClientId` / `X-Airpay-Req-H` headers, response signature validation, non-breaking change policy.

## Online Checkout

- Checkout with ShopeePay API: `https://product.shopeepay.com/integration/api/checkout-with-shopeepay`
- Invalidate Order: `https://product.shopeepay.com/integration/api/invalidate-order`
- Product page: `https://product.shopeepay.com/products/online-payments/checkout-with-shopeepay`

Use for: `/v3/merchant-host/order/create` fields, `platform_type`, validity limits, `redirect_url_http`, customer journey with or without the app, invalidating abandoned orders.

## Hosted ShopeePay Payment Gateway

- API: `https://product.shopeepay.com/integration/api/shopeepay-gateway`
- Product page: `https://product.shopeepay.com/products/online-payments/payment-gateway`

Use for: `api.gw.*` hosts, `/v1/checkout`, `allowed_payment_method` (`online_banking`, `duitnow_qr`, `card`), items/fee/discount math, card tokenization flag, checkout status enum, cancel, `/v1/refund`, refund status, webhook event types.

## In-Person QR

- MPM API: `https://product.shopeepay.com/integration/api/merchant-presented-mode`
- CPM API: `https://product.shopeepay.com/integration/api/customer-presented-mode`
- Invalidate QR: `https://product.shopeepay.com/integration/api/invalidate-qr`
- Product pages: `https://product.shopeepay.com/products/in-person-payments/merchant-presented-mode`, `https://product.shopeepay.com/products/in-person-payments/customer-presented-mode`

Use for: dynamic QR creation, convenience fees (MY), `qr_content`/`qr_url`, static QR sourcing, CPM `payment_code` scan, CPM error codes and processing behaviour.

## Tokenized Payments

- Account Linking: `https://product.shopeepay.com/integration/api/account-linking`
- Link & Pay: `https://product.shopeepay.com/integration/api/link-and-pay`
- Subscription: `https://product.shopeepay.com/integration/api/subscription`
- Auth & Capture: `https://product.shopeepay.com/integration/api/auth-and-capture`
- Product pages: `https://product.shopeepay.com/products/online-payments/link-and-pay`, `.../subscription`, `.../auth-and-capture`

Use for: linking, access token, unlink, link status callback, user info, coin redemption, Get Payment Methods, Direct Payment, recurring charges, auth/capture/reverse, risk `additional_info`.

## Callbacks, Status, Refunds

- Notify Transaction Status: `https://product.shopeepay.com/integration/api/notify-transaction-status`
- Check Transaction Status: `https://product.shopeepay.com/integration/api/check-transaction-status`
- Refund a Payment: `https://product.shopeepay.com/integration/api/refund-a-payment`

Use for: callback fields per market, success values, acknowledgement and retry rules, requery request, refund rules and error codes.

## Enums And History

- Common Objects: `https://product.shopeepay.com/integration/common-objects`
- Revision History: `https://product.shopeepay.com/integration/revision-history` (single entry: 11 Jul 2024, v1.0 initial document)

Use for: transaction types, transaction status, payment status, payment channel, point of initiation, payment method type, promotion types.

## Account, Plugins, Branding, Availability

- Product Functionalities (market matrix): `https://product.shopeepay.com/products/functionalities`
- E-commerce Plugin Overview: `https://product.shopeepay.com/products/e-commerce-plug-in/overview`
- WooCommerce setup guideline: `https://product.shopeepay.com/products/woocommerce-plug-in/setup-guideline`
- WooCommerce plugin (official source): `https://wordpress.org/plugins/checkout-with-shopeepay/`
- Shopify app guide: `https://product.shopeepay.com/products/shopify-plug-in/add-shopeepay-at-checkout`
- Shopify SPayLater messaging: `https://product.shopeepay.com/products/shopify-plug-in/spaylater-on-site-messaging-app`
- Brand guidelines: `https://product.shopeepay.com/products/brand-guidelines`
- SPayLater widget pages: `https://product.shopeepay.com/products/functionalities/spl-promotional-widget/about` (plus `/android`, `/ios`, `/web`)
- Malaysia partner registration: `https://shopeepay.com.my/en/merchant/be-a-partner`
- Malaysia service status: `https://shopeepay.com.my/en/service-status`

Use for: Malaysia method availability, onboarding route, plugin settings and callback registration, callback header name (plugin source), branding rules, operational status.

## Known Doc Gaps And Inconsistencies

- Callback signature header name is not stated on the notify page; plugin source uses `X-Airpay-Req-H`.
- Response signature header name is not stated.
- Direct Payment and Subscription request tables omit `payment_reference_id` (and Subscription omits `currency`) although responses reference it.
- Capture response table says `transaction_type` 1000; sample shows 1001.
- Refund response field is described as `transaction` but the sample uses `transaction_list`.
- Hosted gateway refund status text says `succeeded`; samples return `successful`. Checkout status enum is capitalised in text, lowercase in samples.
- Hosted gateway: header names listed as `client_id`/`signature` but example uses `X-Airpay-*`; signing for bodiless calls, Get Refund Status method, error body shape, and webhook acknowledgement are not documented.
- No public sandbox test data, rate limits, callback IP list, settlement report API, or pricing.
