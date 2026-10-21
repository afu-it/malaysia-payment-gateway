# SPayLater Source Map

Use this map to pick the right official source before answering or coding. SPayLater online acceptance is documented inside the ShopeePay developer site `https://product.shopeepay.com`. All pages checked 2026-10-21.

## Coverage

- Index used: no `llms.txt`, `sitemap.xml` or OpenAPI file exists (404/403). Pages were enumerated by crawling every internal link starting at `/integration/get-started`, including the sidebar navigation.
- Enumerated: 34 docs pages on `product.shopeepay.com`. Read: 34 of 34 (including expanded "Child parameters" and request/response samples embedded in each page).
- Also read: branding PDF linked from the docs, the official WooCommerce plugin `checkout-with-shopeepay` v1.0.0 source (API client, webhook handler, country config, gateway), and 4 pages on `shopeepay.com.my` (merchant, be-a-partner, spaylater, service-status).
- Left out on purpose: per-country details for ID, PH, SG, TH, VN (bank_transfer, QRIS, PromptPay, VietQR, QRPh, Maribank, TH/VN co-funding and retry rules) except where they explain Malaysian behaviour. Reason: this skill targets Malaysian merchants. The full ShopeePay gateway is covered by the separate `setup-shopeepay-gateway` skill.

## Getting Started, Auth, Signature

- Get Started: `https://product.shopeepay.com/integration/get-started`
- Gateway getting started (same rules, gateway hosts): `https://product.shopeepay.com/integration/api/shopeepay-gateway`
- Official plugin signing and webhook code: `https://wordpress.org/plugins/checkout-with-shopeepay/` (download `https://downloads.wordpress.org/plugin/checkout-with-shopeepay.1.0.0.zip`, files `includes/class-shopeepay-api.php`, `includes/class-shopeepay-webhook.php`, `includes/class-shopeepay-countries.php`)

Use for: NDA/onboarding, Client ID and Secret Key, access node hosts (wallet and gateway), protocol rules, empty-value defaults, `X-Airpay-ClientId` and `X-Airpay-Req-H`, HMAC-SHA256 Base64 algorithm, test vector, callback header name, backward-compatible change policy.

## Payment Creation

- ShopeePay Payment Gateway API: `https://product.shopeepay.com/integration/api/shopeepay-gateway`
- Checkout with ShopeePay API: `https://product.shopeepay.com/integration/api/checkout-with-shopeepay`
- Merchant Presented Mode API: `https://product.shopeepay.com/integration/api/merchant-presented-mode`
- Customer Presented Mode API: `https://product.shopeepay.com/integration/api/customer-presented-mode`

Use for: `/v1/checkout` with `allowed_payment_method: ["spay_later"]`, `/v3/merchant-host/order/create` with `preferred_payment_method_type: "spay_later"`, MPM QR, CPM "Processing" behaviour for SPayLater, field tables, errcodes, HTTP error codes, validity limits for Malaysia.

## Status, Callbacks, Cancel

- Check Transaction Status: `https://product.shopeepay.com/integration/api/check-transaction-status`
- Notify Transaction Status: `https://product.shopeepay.com/integration/api/notify-transaction-status`
- Invalidate Order: `https://product.shopeepay.com/integration/api/invalidate-order`
- Invalidate QR: `https://product.shopeepay.com/integration/api/invalidate-qr`
- Gateway status, cancel and callbacks: `https://product.shopeepay.com/integration/api/shopeepay-gateway`

Use for: requery fields, polling schedule (5 s steps to 100 s, then 5 min to 24 h), Malaysia single-send callback (no retry), `errcode 0` acknowledgement, `payment_status` vs `transaction_status`, Gateway event types, cancel/invalidate.

## Refunds

- Refund a Payment (v3): `https://product.shopeepay.com/integration/api/refund-a-payment`
- Gateway refunds: `https://product.shopeepay.com/integration/api/shopeepay-gateway`

Use for: full/partial refund rules, 365-day window (180 days on Functionalities page), 12 am to 5 am block, idempotency, refund errcodes, Gateway refund status values.

## SPayLater Tokenized Flows

- Account Linking: `https://product.shopeepay.com/integration/api/account-linking`
- Link & Pay: `https://product.shopeepay.com/integration/api/link-and-pay`
- Subscription: `https://product.shopeepay.com/integration/api/subscription`
- Auth & Capture: `https://product.shopeepay.com/integration/api/auth-and-capture`
- Product overviews: `https://product.shopeepay.com/products/online-payments/link-and-pay`, `.../subscription`, `.../auth-and-capture`, `.../checkout-with-shopeepay`, `.../payment-gateway`

Use for: access tokens, `spaylater_info` balance, Get Payment Methods `spay_later.spaylater_options` (tenure, monthly payment, compliance notices), Direct Payment with `payment_option_reference`, risk `additional_info`, auth hold/capture/reverse, SPL credit limit errcode `1102`.

## Enums

- Common Objects: `https://product.shopeepay.com/integration/common-objects`
- Revision History: `https://product.shopeepay.com/integration/revision-history` (v1.0, 11 Jul 2024)

Use for: transaction types, transaction status, payment status, payment channels 4 to 12 (SPayLater tenures), point of initiation, payment method types, promotion types.

## Market, Settlement, Onboarding

- Product Functionalities: `https://product.shopeepay.com/products/functionalities`
- Home: `https://product.shopeepay.com/`
- In-person product pages: `https://product.shopeepay.com/products/in-person-payments/customer-presented-mode`, `.../merchant-presented-mode`
- ShopeePay Malaysia merchant: `https://shopeepay.com.my/en/merchant` and `https://shopeepay.com.my/en/merchant/be-a-partner`
- SPayLater consumer page: `https://shopeepay.com.my/en/spaylater`
- ShopeePay Malaysia service status (renders in browser only): `https://shopeepay.com.my/en/service-status`
- Shopee seller fee article (marketplace only): `https://seller.shopee.com.my/edu/article/16542`

Use for: Malaysia method availability, Monee Capital as SPayLater provider, settlement frequencies and payout timing, registration steps, tenures offered, credit risk and full settlement statement, buyer-side fee context (link only, do not quote), operational status, pricing pointers.

## Plugins And Widget

- E-commerce plugin overview: `https://product.shopeepay.com/products/e-commerce-plug-in/overview`
- Shopify add ShopeePay at checkout: `https://product.shopeepay.com/products/shopify-plug-in/add-shopeepay-at-checkout`
- Shopify SPayLater On-Site Messaging app: `https://product.shopeepay.com/products/shopify-plug-in/spaylater-on-site-messaging-app`
- WooCommerce setup: `https://product.shopeepay.com/products/woocommerce-plug-in/setup-guideline`
- SPL widget about/web/android/ios: `https://product.shopeepay.com/products/functionalities/spl-promotional-widget/about`, `.../web`, `.../android`, `.../ios`
- Brand guidelines: `https://product.shopeepay.com/products/brand-guidelines` and PDF `https://product.shopeepay.com/static/files/ShopeePay_SPayLater_Online_Branding_Guide.pdf`

Use for: no-code setup, Access Keys, callback URL registration, promotional messaging install, CSP, compliance wording, logo sizes.

## Gaps Only ShopeePay Can Answer

- What to sign for bodyless GET requests on the Gateway, and the exact response signature header name.
- Gateway callback acknowledgement format and retry policy; account-link callback acknowledgement.
- Callback source IPs, rate limits, SPayLater minimum/maximum amounts.
- Meaning of `spaylater_info.status_info` values.
- Return URL query parameters and their `signature` (seen only inside a sample URL).
- Whether SPayLater can be charged via Subscription and Auth & Capture in Malaysia.
- Refund window (365 vs 180 days) for SPayLater payments.
- Sandbox test users with SPayLater credit.
