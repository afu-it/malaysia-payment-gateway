# MOBY Source Map

Use this map to pick the right official source before answering or coding. All checked 2026-10-21.

The docs site `https://docs.mobypay.my/` is a single-page app. Pages load at `https://docs.mobypay.my/docs/<slug>` (moby.my links use the form `https://docs.mobypay.my/?path=%2Fdocs%2F<slug>`). There is no llms.txt or sitemap.xml (both return the app shell). Page content was read from the site's own published bundle.

## Coverage

- Docs site sidebar: 61 pages enumerated, 61 read (Getting Started 3, Payments 9, Platform Plugins 9, POS & Kiosk 5, Financing 4, Loyalty 4, Merchant Management 4, Merchant User Guide 21, API Reference explorer 1, Support 1). Three legacy alias routes (`/docs/woocommerce`, `/docs/shopify`, `/docs/easystore`) duplicate the plugin pages.
- API Reference explorer: 19 v1 operations and 9 Coming Soon v2 operations, all read.
- OpenAPI 3.0 spec `https://dev.pay.mobycheckout.com/docs?api-docs.json`: 28 operations, all read. Production copy `https://pay.mobycheckout.com/docs?api-docs.json` is byte-identical.
- Official plugin source read: WooCommerce Moby Checkout v2.0, Magento 2 `Moby_Checkout`, OpenCart 4.0 (signature, callback, refund behaviour).
- GitHub `MobyPayTech/moby-plugins` (self-marked deprecated): `direct-integration`, `recurring-payments`, `sandbox-testing` READMEs read.
- Scoped out: Financing, Loyalty, Merchant Management APIs (all marked Coming Soon / not in production) are listed in the catalogue only; merchant user guide pages are video walkthroughs and are summarised, not transcribed. Kiosk platform code samples (Kotlin, Swift, C#, Python) are summarised by protocol in `moby-pos.md`.

## Getting Started

- Introduction: `https://docs.mobypay.my/docs/introduction`
- Sandbox Testing: `https://docs.mobypay.my/docs/sandbox-testing`
- Authentication: `https://docs.mobypay.my/docs/authentication`

Use for: base URLs, getting credentials, token call, token expiry, scopes, test cards, FPX test bank, e-wallet live-only warning, rate limits.

## Online Payments

- Hosted Checkout: `https://docs.mobypay.my/docs/hosted-checkout`
- Direct API Cards: `https://docs.mobypay.my/docs/direct-cards`
- Direct API FPX: `https://docs.mobypay.my/docs/direct-fpx`
- Direct API E-Wallets: `https://docs.mobypay.my/docs/direct-ewallets`
- Direct API DuitNow QR: `https://docs.mobypay.my/docs/direct-duitnow`
- Direct API BNPL: `https://docs.mobypay.my/docs/direct-bnpl`
- API Reference explorer: `https://docs.mobypay.my/docs/api-reference`

Use for: hosted session fields, `payLink`, method list, `payMethod` codes, direct pay fields, `payUrl`, DuitNow verify, status, detail, capture, void, refund field tables.

## Callbacks And Signatures

- Webhooks & Notifications: `https://docs.mobypay.my/docs/notifications`
- Signature Verification: `https://docs.mobypay.my/docs/signature-verification`
- Deprecated detail (return redirect fields, retry note): `https://github.com/MobyPayTech/moby-plugins/tree/main/direct-integration`
- Plugin source: `https://docs.mobypay.my/plugins/downloads/woocommerce/moby-checkout/v2.0/moby-checkout.zip`, `https://docs.mobypay.my/plugins/downloads/magento/moby-checkout/Moby_Checkout.zip`, `https://docs.mobypay.my/plugins/downloads/opencart/v4.0/mobypay.ocmod.zip`

Use for: callback payload, the 8 signed fields and order, HMAC-SHA256 with the Secret Key, `{"status":"success"}` acknowledgement, idempotency, extra statuses `cancelled`/`processing`, `?`-prefixed query keys, refund callback handling.

## Machine-Readable Spec

- Swagger UI: `https://dev.pay.mobycheckout.com/api/documentation`
- OpenAPI JSON: `https://dev.pay.mobycheckout.com/docs?api-docs.json`

Use for: the full live endpoint list including OpenAPI-only operations (omnipanel refund, subscription API, customer mobile move, BNPL merchant status), optional `transactionId`/`callback_url` on void and refund, `x-token` scheme, field types.

## Recurring

- Recurring Payments (API): `https://docs.mobypay.my/docs/recurring-payments`
- Recurring Billing (dashboard): `https://docs.mobypay.my/docs/guide-recurring-billing`
- Deprecated detail: `https://github.com/MobyPayTech/moby-plugins/tree/main/recurring-payments`

Use for: tokenisation redirect, token list/delete, token charge with or without 3DS, charge status, async charge refund, error shapes, dashboard plans/links/retries/portal.

## POS And Kiosk

- Overview: `https://docs.mobypay.my/docs/instore-overview`
- TCP Protocol & Security: `https://docs.mobypay.my/docs/instore-tcp`
- Message Format: `https://docs.mobypay.my/docs/instore-messages`
- Installment Plans (IPP): `https://docs.mobypay.my/docs/instore-ipp`
- Platform Examples: `https://docs.mobypay.my/docs/instore-platforms`
- Reference kiosk: `https://github.com/MobyPayTech/moby-plugins/tree/main/pos-kiosk-integration`

Use for: TCP framing, payload HMAC, nonce/timestamp rules, message types, IPP REST endpoints, Visa plan cancel.

## Plugins

- WooCommerce: `https://docs.mobypay.my/docs/woocommerce-checkout`, `https://docs.mobypay.my/docs/woocommerce-price-divider`
- Shopify: `https://docs.mobypay.my/docs/shopify-checkout`, `https://docs.mobypay.my/docs/shopify-islamic`, `https://docs.mobypay.my/docs/shopify-price-divider`
- Magento: `https://docs.mobypay.my/docs/magento`
- OpenCart: `https://docs.mobypay.my/docs/opencart`
- EasyStore: `https://docs.mobypay.my/docs/easystore-checkout`, `https://docs.mobypay.my/docs/easystore-price-divider`

Use for: no-code setup, which credential goes in which plugin field, plugin requirements and troubleshooting.

## Coming Soon (not live)

- Financing: `https://docs.mobypay.my/docs/financing-overview` (+ `-applications`, `-documents`, `-repayments`)
- Loyalty: `https://docs.mobypay.my/docs/loyalty-overview` (+ `-points`, `-transactions`, `-campaigns`)
- Merchant Management: `https://docs.mobypay.my/docs/merchant-overview` (+ `-profile`, `-outlets`, `-settings`)

Use for: telling users these APIs are not in production yet. Do not build against them.

## Merchant And Commercial

- Merchant User Guide: `https://docs.mobypay.my/docs/user-guide` (Payments, Refund & Disputes, Payment Links, Invoices, Users, Terminal, Brand Settings, Stores, and others under `/docs/guide-*`)
- Support: `https://docs.mobypay.my/docs/support`
- Website: `https://moby.my/`, `https://moby.my/get-started/`, `https://moby.my/contact-sales`, `https://moby.my/bnpl/`
- Pricing: `https://moby.my/pricing/`
- Dashboard login: `https://omni.mobypay.my`

Use for: onboarding route, dashboard features, settlement reports, support contacts, pricing link (do not copy numbers).
