# Revenue Monster Source Map

Use this map to pick the right source before answering or coding. Index used: `https://doc.revenuemonster.my/sitemap.xml` (checked 2026-10-21). The site's `/llms.txt` returns the HTML home page, not an index.

## Overview And Environments

- Overview: `https://doc.revenuemonster.my/docs/introduction/overview`
- Product terms: `https://doc.revenuemonster.my/docs/product-terms`
- SDK list: `https://doc.revenuemonster.my/docs/quickstart/sdk`

Use for: sandbox and production OAuth/Open API hosts, merchant sign-up, sandbox activation, order ID vs transaction ID vs reference ID.

## Authentication

- Client credentials: `https://doc.revenuemonster.my/docs/quickstart/accesstoken/client-credentials`
- Refresh token: `https://doc.revenuemonster.my/docs/quickstart/accesstoken/refresh-token`
- Authorization code (plugin developers): `https://doc.revenuemonster.my/docs/quickstart/accesstoken/authorization-code`

Use for: `/v1/token`, Basic auth header, `grantType`, token expiry, refresh flow, scopes.

## Signatures

- Signature algorithm: `https://doc.revenuemonster.my/docs/quickstart/signature-algorithm`
- Verify signature: `https://doc.revenuemonster.my/docs/quickstart/verify-signature`
- Go SDK: `https://github.com/RevenueMonster/rm-go-sdk/blob/master/signature.go`, `client.go`, `constant.go`
- JS SDK: `https://github.com/RevenueMonster/rm-js-sdk/blob/main/src/signature.ts`
- PHP SDK: `https://github.com/RevenueMonster/rm-php-sdk/blob/main/src/Modules/Module.php`

Use for: string-to-sign order, JSON sorting and escaping, RSA PKCS#1 v1.5 SHA-256, `X-Signature` / `X-Nonce-Str` / `X-Timestamp`, nonce and timestamp windows, `INVALID_REQUEST_SIGNATURE` debug output, callback verification with server public key.

## Online Payment

- Online payment (hosted, direct checkout, redirect, notify, query checkout): `https://doc.revenuemonster.my/docs/v2/payment/online-payment`
- Card-on-file tokens: `https://doc.revenuemonster.my/docs/v2/payment/online-payment-backup`
- Legacy notify URL payload (doc source repo): `https://github.com/RevenueMonster/doc-v2/blob/master/docs/payment/webpayment/notify-url.md`
- Apple Pay: `https://doc.revenuemonster.my/docs/digital-wallet/apple-pay`
- Google Pay: `https://doc.revenuemonster.my/docs/digital-wallet/google-pay`

Use for: `POST /v3/payment/online` fields, `item.url`, Direct Payment Checkout modes, FPX bank list, card native, QR modes, redirect params, notify payload, checkout status.

## Status, Refund, Reconciliation

- Query transaction: `https://doc.revenuemonster.my/docs/v2/payment/query-transaction`
- Refund and reverse: `https://doc.revenuemonster.my/docs/v2/payment/cancel-transaction`
- Reconciliation: `https://doc.revenuemonster.my/docs/v2/payment/reconciliation`

Use for: query by order ID / transaction ID, Transaction Object, transaction statuses, refund body, reverse on timeout, settlement amounts.

## Other Payment Flows

- Tokenization and recurring: `https://doc.revenuemonster.my/docs/v2/payment/tokenization-payment`
- Quick Pay: `https://doc.revenuemonster.my/docs/v2/payment/quick-pay`
- Deeplink: `https://doc.revenuemonster.my/docs/v2/payment/deeplink-integration`
- Terminal: `https://doc.revenuemonster.my/docs/v2/payment/terminal-integration`

Use for: recurring customers, saved cards, merchant-scan wallet payments, app deeplinks, terminals.

## Appendix

- Payment methods: `https://doc.revenuemonster.my/docs/payment-method`
- FPX bank codes: `https://doc.revenuemonster.my/docs/bank-code`
- Error codes: `https://doc.revenuemonster.my/docs/error-codes`
- Testing wallets: `https://doc.revenuemonster.my/docs/downloads/testing-wallets`
- Stores API: `https://doc.revenuemonster.my/docs/settings/store-detail/store-details`

Use for: `<METHOD>_MY` names, B2C/B2B bank codes, error meanings, sandbox wallet support, Store ID lookup.

## Plugins

- Low-code checkout: `https://doc.revenuemonster.my/docs/ecom-plugin/lowCodeCheckout`
- WooCommerce docs: `https://doc.revenuemonster.my/docs/ecom-plugin/wooCommerce`
- WooCommerce plugin source: `https://github.com/RevenueMonster/woocommerce-revenuemonster-payment-gateway`
- Plugin platform: `https://doc.revenuemonster.my/docs/v2/plugin/introduction`, `https://doc.revenuemonster.my/docs/v2/plugin/integration`

Use for: no-code checkout, reference implementation of redirect plus notify handling (settles by querying transaction by order ID).

## Commercial

- Pricing: `https://revenuemonster.my/pricing`
- Merchant Portal: `https://merchant.revenuemonster.my/`, sandbox `https://sb-merchant.revenuemonster.my/`

Use for: current fees (link only), registration, credentials.

## Invoice

- Create: `https://doc.revenuemonster.my/docs/invoice/create-invoice`
- List: `https://doc.revenuemonster.my/docs/invoice/get-invoices`
- Get: `https://doc.revenuemonster.my/docs/invoice/get-invoice-by-id`
- Edit: `https://doc.revenuemonster.my/docs/invoice/edit-invoice`
- Cancel: `https://doc.revenuemonster.my/docs/invoice/cancel-invoice`

Use for: `/api/v3/invoice*` paths, invoice fields, statuses, 422 `VALIDATION_ERROR` shape (`references/revenue-monster-invoice.md`).

## In-Store And POS

- POS payment, cancellation, card refund, card settlement: `https://doc.revenuemonster.my/docs/pos/payment`, `/docs/pos/cancellation`, `/docs/pos/card-refund`, `/docs/pos/card-settlement`
- Gourmet card verify: `https://doc.revenuemonster.my/docs/campaign/gourmet-card`
- Deeplink demo: `https://github.com/RevenueMonster/applink-demo`

Use for: `references/revenue-monster-instore.md` (Quick Pay variants, terminal events, receipt types, deeplink transaction types).

## Other Products (catalogue only)

All pages under these sitemap sections were read for method, path and purpose: `/docs/campaign/*` (loyalty, member, voucher, chop stamp), `/docs/alacarte-open/*`, `/docs/ekyc/*`, `/docs/visa-vop/*`, `/docs/sms/send-sms`, `/docs/push-notification/push-to-merchant`, `/docs/merchant-onboarding/*`, `/docs/settings/*`, `/docs/merchant-wallet/*`, `/docs/downloads/*`, `/docs/ecom-plugin/*`, `/docs/quickstart/mobile-sdk`, `/docs/introduction/usecase`.

Use for: the purpose-only rows of the Endpoint Catalogue in `references/revenue-monster-api.md`.

## Coverage

Checked 2026-10-21.

- Index: `https://doc.revenuemonster.my/sitemap.xml` lists 141 URLs: 134 documentation pages under `/docs/`, 6 blog/footer pages, and the home page. No `llms.txt` index (returns the HTML home page), no OpenAPI file, no Postman collection published.
- Documentation pages enumerated: 134. Pages fetched and read: 134 (all returned HTTP 200 on `doc.revenuemonster.my`).
- Read in full with field tables carried into the skill (payment scope, 40 pages): introduction overview and use case (2), product terms (1), quickstart access token (3), signature and verify (2), SDK and mobile SDK (2), all `/docs/v2/payment/*` (9), `/docs/v2/plugin/*` (2), digital wallet Apple Pay and Google Pay (2), appendix payment method, bank code, error codes (3), downloads testing wallets (1), e-commerce plugins (4), invoice (5), POS (4).
- Read for method, path and purpose only (94 pages): campaign loyalty/member/voucher/chop stamp/gourmet card (33; gourmet card verify is also used in the in-store file), a la carte (18), eKYC (5), Visa VOP (5), SMS (1), push notification (1), merchant onboarding (10), settings (14), merchant wallet (4), downloads logo and application (3).
- Total: 40 + 94 = 134 of 134 enumerated pages read.
- Left out of field-level detail, and why: loyalty, voucher, a la carte, eKYC, Visa VOP, SMS, push notification, partner onboarding, account/store settings and merchant wallet are not payment acceptance; an integrator building checkout, settlement, refunds or recurring billing does not need their field tables. Their endpoints are still in the catalogue so an agent knows they exist and where to read more.
- Blog and footer pages (6): not developer docs; skipped.
- Doc source repo `https://github.com/RevenueMonster/doc-v2` (official): 152 files in `docs/`. 26 older files under `docs/payment/` are not served by the live site (their URLs return the site fallback page, verified by identical HTML for all 26). Read for endpoints only; the 5 endpoints unique to them are listed as legacy in the catalogue. The live site also has pages missing from the repo (invoice, digital wallet, two voucher pages, update account agent rate), so the live site is the source of truth.
- SDK and plugin source read for signatures: `rm-go-sdk` (`signature.go`, `client.go`, `constant.go`), `rm-js-sdk` (`src/signature.ts`), `rm-php-sdk` (`src/Modules/Module.php`), `woocommerce-revenuemonster-payment-gateway`.

## Corrections Made In The 2026-10-21 Top-Up

- `SKILL.md` Core Facts: the escape rule had lost its targets (it read "escaped as `<`, `>`, `&`"); fixed to the backslash-u forms `003c`, `003e`, `0026` per the signature page.
- Sandbox WeChat Pay: the skill said WeChat Pay works only in production. The testing wallets page table says sandbox No, but its note says a real WeChat Pay wallet works in both environments and recommends one sandbox round. Both are now stated (`revenue-monster-api.md`, `account-setup.md`).
- Token expiry: added that the refresh-token page example shows `72591999` while the client-credentials example shows `2591999`, both described as 30 days.
- Verify Signature: added that the page's worked example puts `requestUrl` last; the SDKs use alphabetical order.
- Authorization code: the code-for-token exchange body is not documented; the catalogue now says so instead of implying a `grantType` value.

## Not Documented (checked 2026-10-21)

Notify request header names and signature on payment notify, payment notify retry policy (only voucher callbacks document 5 retries at 1-minute intervals), IP allowlists, numeric rate limits, partial refund request format, how to pay with a stored card-on-file token, invoice webhooks, settlement payout schedule, `DAILY` recurring target meaning.
