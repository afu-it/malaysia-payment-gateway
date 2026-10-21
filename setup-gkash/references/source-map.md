# Gkash Source Map

Use this map to pick the right source before answering or coding. All pages checked 2026-10-21.

Note: `https://doc.gkash.my/` is a Blazor WebAssembly site. A plain HTTP fetch returns only the navigation shell; render the page with JavaScript to read it. There is no `llms.txt`, sitemap, or OpenAPI file (all return 404).

## Docs Index

- Home and navigation: `https://doc.gkash.my/`

Use for: list of current pages (online payments, offline payments, query, refund, plugins, developer tools, legacy docs).

## Getting Started and Sandbox

- Getting started: `https://doc.gkash.my/getting-started`
- Sandbox merchant portal: `https://demo.pay.asia`

Use for: sandbox sign-up form, sandbox portal, where to start.

## Online Checkout

- Web payments (host-to-host, current): `https://doc.gkash.my/v2/web-payments`
- Web payments (legacy browser form): `https://doc.gkash.my/web-payments`

Use for: `POST /api/payment/form`, request fields, request signature, `redirect.url` / `redirect.html` response, callback fields and signature, `OK` acknowledgement, return URL fields, legacy `/api/paymentform.aspx` form post.

## Pre-auth and Recurring

- Pre-auth and capture (current): `https://doc.gkash.my/v2/preauth-and-capture`
- Pre-auth and capture (legacy): `https://doc.gkash.my/preauth-and-capture`
- Recurring payments (current): `https://doc.gkash.my/v2/recurring-payments`
- Recurring payments (legacy): `https://doc.gkash.my/recurring-payments`

Use for: `v_preauth`, capture via `/api/payment/submit`, `recurringtype`, `subscriptioncount`, `epkey`, `SubscriptionRemID`, `SubscriptionCount`.

## Status and Refunds

- Query transaction status: `https://doc.gkash.my/query-transaction-status`
- Refund: `https://doc.gkash.my/refund`

Use for: `POST /api/payment/query` (version `1.3.0`), `POST /api/payment/refund` (version `1.3.1`), their signatures, response fields, refund status codes.

## Payment Methods

- Google Pay: `https://doc.gkash.my/google-pay`

Use for: Google Pay enablement, `CRYPTOGRAM_3DS` only, eligibility.

## Offline Payments (out of scope for online checkout)

- E-wallet payments: `https://doc.gkash.my/ewallet-payments`
- Terminal payments: `https://doc.gkash.my/terminal-payments`
- Soundbox payments: `https://doc.gkash.my/soundbox-payments`

Use for: customer and merchant presented QR via `/api/payment/submit`, `paymentid` codes, static DuitNow QR, SoftPOS `RemotePay` / `RemoteCancel`, Soundbox cancel and e-Invoice QR.

## Plugins and SDKs

- WooCommerce docs: `https://doc.gkash.my/woocommerce`
- WooCommerce plugin: `https://wordpress.org/plugins/woo-gkash/` (download `https://downloads.wordpress.org/plugin/woo-gkash.zip`, version 1.3.0, author Gkash Sdn. Bhd.)
- OpenCart docs: `https://doc.gkash.my/opencart`
- GitHub org: `https://github.com/gkashmy`
- iOS SDK: `https://github.com/gkashmy/gkash-ios-sdk` (`GkashPayment/PaymentRequest.swift`)
- Android SDK: `https://github.com/gkashmy/gkash-android-sdk`
- React Native SDK: `https://github.com/gkashmy/gkash-react-native-sdk`

Use for: production host `https://api.gkash.my` (plugin `CheckStagingEnv`, iOS `HOST_URL`), a reference implementation of request and callback signatures, idempotent callback handling, mobile integration.

## Merchant Onboarding, Pricing, Settlement

- FAQ: `https://gkash.com/faq_1/`
- Onboarding: `https://onboard.gkash.my/`
- Contact: `https://gkash.com/contact-us/`

Use for: registration documents, approval time, how API credentials are delivered, staging account pointer, settlement timing rules, MDR and fees (link only), support email.

## Offline Reference Sources (added 2026-10-21)

- E-wallet payments: `https://doc.gkash.my/ewallet-payments`
- Terminal payments: `https://doc.gkash.my/terminal-payments`
- Soundbox payments: `https://doc.gkash.my/soundbox-payments`
- SoftPOS local web API: `https://github.com/gkashmy/gkash-softpos-local-web-api`
- SoftPOS Android TCP SDK: `https://github.com/gkashmy/gkash-client-tcp-softpos-android`
- SoftPOS Windows TCP sample: `https://github.com/gkashmy/gkash-client-tcp-softpos-windows`
- Socket library: `https://github.com/gkashmy/client-socket-connection`

Use for: everything in `references/gkash-offline.md` (`paymentid` and SoftPOS `PaymentType` codes, `RemotePay`, `RemoteCancel`, `SoundboxCancel`, `PublishInvoice`, `GetDuitNowStaticQR`, family B signature vectors, local SoftPOS endpoints).

## Coverage

Checked 2026-10-21.

- Index used: the docs site has no `llms.txt`, `llms-full.txt`, `sitemap.xml`, `robots.txt`, or OpenAPI (all empty or 404). Pages were enumerated from the navigation in the site shell and the route strings inside the Blazor bundle `GkashDocs.Client.wasm` (`@page` routes plus every `https://api-staging.pay.asia/...` URL).
- Docs pages enumerated: 15 (`/getting-started`, `/v2/web-payments`, `/v2/preauth-and-capture`, `/v2/recurring-payments`, `/web-payments`, `/preauth-and-capture`, `/recurring-payments`, `/query-transaction-status`, `/refund`, `/google-pay`, `/ewallet-payments`, `/terminal-payments`, `/soundbox-payments`, `/woocommerce`, `/opencart`). The "Developer Tools" nav entry is only a link to `https://github.com/gkashmy`.
- Docs pages read in full: 15 of 15.
- API URLs found in the bundle: 12 distinct Gkash paths, all in the Endpoint Catalogue of `gkash-api.md` (plus your own callback and return URLs).
- Official GitHub repos enumerated (`github.com/gkashmy`): 7. Read: iOS SDK source and README; Android SDK and React Native SDK READMEs; SoftPOS local web API, Android TCP, and Windows TCP READMEs; `client-socket-connection` has no README.
- Official plugin read: WooCommerce `woo-gkash` 1.3.0 (`wc-gkash.php`, `readme.txt`). OpenCart plugin source is on the OpenCart marketplace and was not downloaded; its doc page was read.
- Left out on purpose: in-depth SoftPOS TCP SDK internals (local device integration, not an online gateway API). Their entry points are listed in `gkash-offline.md`.

## Corrections and Doc Inconsistencies Noted (2026-10-21)

- No existing content in this skill was found wrong during the top-up.
- `RemotePay` signature: hash the raw string, then uppercase the hex. The doc vector proves the input is not uppercased.
- `PublishInvoice` signature: doc text says uppercase the input, but the doc vector only matches the non-uppercased input with uppercase hex output. Recorded in `gkash-offline.md`.
- `RemoteCancel` / `SoundboxCancel` parameter table example signature (`D0E18286...`) differs from the worked vector (`ED17C82C...`); table value treated as a placeholder.
- Soundbox request header says JSON, cURL example uses form encoding.
- `RemotePay` field table says `callbackurl`, cURL example sends `CallbackURL`; the `PreAuth` description is a copy error.
- Sync response signatures on `/api/payment/submit` examples (`7f152d80...`) do not match the documented verify formula for the example data.
