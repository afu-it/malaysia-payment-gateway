# GrabPay & PayLater Source Map

Use this map to pick the right official source before answering or coding. All pages checked 2026-10-21.

## Product And Market Overview

- Malaysia merchant page: `https://www.grab.com/my/merchant/checkout-solutions/`
- Malaysia sign-up: `https://www.grab.com/my/merchant/pay/apply/`
- PayLater for merchants (MY): `https://www.grab.com/my/merchant/finance/pay-later/`
- Merchant Resource Centre: `https://merchant.grab.com/en-my/brand-centre/paylater-by-grab`

Use for: who can apply, plugins list, settlement claim (within 5 working days), tokenisation mention, positioning.

## One Time Charge API (current)

- OTC reference (Redoc): `https://developer.grab.com/docs/payment-otc/api/v2`

Use for: getting started, credentials, headers, both HMAC algorithms, init, Web URL, token, complete, both status endpoints, refunds, refund status, webhook schema, status and reason tables, redirect errors, OAuth errors, deep links, FAQ (lifetimes, 15-minute cancel, refund balance), rate limit, settlement reports, amount table.

## One Time Charge Guide (previous version, still online)

- Index: `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/`
- Prerequisites, scopes, security: `.../prerequisites`
- API and HMAC signatures, environments: `.../api-hmac`
- POP signature: `.../generate-hmac`
- Service discovery: `.../service-discovery-endpoint`
- Init (nested `metaInfo`, `items`, `shippingDetails`, itemCategory enum): `.../init-payments`
- Web URL and PKCE: `.../generate-web-url`
- Redirect and errors: `.../grabpay-redirect`
- Get OAuth token: `.../get-oauth`
- Complete: `.../complete-payment`
- OTC status (`oAuthCode` rules, 5 second retry): `.../otc-endpoint`
- Partner charge status: `.../partner-charge-endpoint`
- Check payment status comparison: `.../check-payment-status`
- Processing status: `.../processing-status`
- Webhook (`application/json; charset=utf-8` rule, payload, rewardsMeta): `.../handle-payment-noti`
- Refunds and refund status: `.../generate-refunds`, `.../check-refund-status`
- Reasons: `.../charge-status-reasons`, `.../refund-status-reasons`, `.../appendix-reason-params`, `.../appendix-troubleshooting`
- Amounts, deep links, user flows, changelog: `.../appendix-amt-details`, `.../appendix-deep-links`, `.../appendix-user-flows`, `.../otc-changelog`
- PDF v2.1: `https://developer.grab.com/assets/docs/grab-pay/GrabPay_One_time_Payments_Integration_Guide.pdf`

Use for: field detail the Redoc page collapses, and cross-checking signatures.

## Official SDK

- `https://github.com/grab/grabpay-merchant-sdk` (files read: `README.md`, `Node/README.md`, `Node/src/utils/Utils.ts`, `Node/src/utils/Requester.ts`, `Node/src/MerchantIntegrationOnline.ts`, `Node/src/configs/*.ts`, `Node/src/types.ts`, `Python/src/RestClient.py`, `Python/src/MerchantIntegrationOnline.py`, `Python/src/Config.py`)

Use for: exact HMAC on GET (empty digest, path with query), POP JSON, scope string `openid payment.one_time_charge`, code_challenge encoding, refund `partnerTxID` naming, hosts per country.

## Tokenised Payments v4 (recurring)

- Index: `https://developer.grab.com/docs/tokenisation-v4`
- Pages: `started-prereqs`, `started-api`, `enable-recurring`, `service-discovery`, `bind-init`, `auth-web-url`, `auth-get-oauth`, `auth-hmac-oauth`, `bind-status`, `check-payment-method`, `get-promo`, `validate-promo`, `charge-create`, `charge-topup`, `charge-status`, `charge-unknown-status`, `txn-status`, `refund-generate`, `refund-status`, `unbind`, `webhook-noti`, `reason-codes`, `http-response-codes`, `troubleshoot`, `ux-flows`, `ux-flows-old`, `test-setup`, `changelog` (all under `https://developer.grab.com/docs/tokenisation-v4/`)

Use for: bind, saved-wallet charges, top-up redirect, promos, unbind, v4 webhooks (2 retries).

## PayLater Integrations Launchpad (plugins and branding)

- v2 index: `https://developer.grab.com/docs/paylater-integrations-launchpad-v2`
- Plugins: `.../plugins-woocommerce`, `.../plugins-magento`, `.../plugins-shopify`, `.../plugins-web`, `.../plugins-price-divider`
- Guidelines: `.../guidelines-checkout-option-display`, `.../guidelines-web-store`, `.../guidelines-promoting-responsible-spending`, `.../guidelines-paylater-by-grab-text`, `.../guidelines-paylater-acceptance-key-messages`, plus logo, lockup, social, KOL, promotions pages
- Brand assets: `.../brand-assets-*` pages
- v1 launchpad (`https://developer.grab.com/docs/paylater-integrations-launchpad`) and GrabPay launchpad (`https://developer.grab.com/docs/grabpay-integrations-launchpad`): older equivalents

Use for: plugin configuration, Price Divider widget, checkout naming and copy, responsible-spending wording.

## GrabID

- `https://developer.grab.com/docs/grab-id`, `https://developer.grab.com/docs/grab-id-oauth-cookbook`, `https://developer.grab.com/assets/docs/grab-id/GrabID_OAuth_API_Reference_v2.pdf`

Use for: OAuth background, scope assignment process, `/token-info`. Note the cookbook shows a form-encoded token call; the payment docs and SDK use JSON. Follow the payment docs.

## Coverage

- Index used: `https://developer.grab.com/docs/sitemap.xml` (320 URLs). `developer.grab.com/llms.txt` and the root sitemap return the SPA shell, not an index.
- Payment-related pages enumerated: 147 URLs (146 unique). Read in full: OTC v2 Redoc (1), OTC previous guide (28), Tokenisation v4 (29), PayLater launchpad v2 (30), launchpad v1 (15), GrabPay launchpad (11), OTC PDF v2.1, plus the SDK files above. Total read: 115 pages + 1 PDF + SDK source.
- Enumerated, skimmed for scope only, not distilled (out of scope for online checkout): POS API v3 (27 pages, in-store QR), `paysi-partner` (POS QR), `paysi-partner-qr-interop-v1` (QR interoperability), `paysi-agent-partner-v2` (legacy Tokenised v2, superseded by v4), `grab-id` and the cookbook (login, partially used), Partner Apps (30 pages, Grab SuperApp mini-app platform, not a web checkout).
- Non-payment Grab APIs in the sitemap (food, mart, express, maps, gifts, Kios, message center): out of scope.

## Known Doc Conflicts (tell the integrator)

1. Authorize `scope`: table says `payment.one_time_charge`; sample URL and SDK use `openid payment.one_time_charge`.
2. HMAC for GET: SDK uses empty digest and signs path with query; tokenisation v4 JS hashes an empty string and signs path only.
3. Refund `originTxID`: required in previous guide, optional in v2 schema.
4. Test environment: current OTC docs say test on Production with test credentials; endpoints still list staging hosts.
5. Webhook HMAC path is not specified.
6. Tokenisation v4 refund path: table `/v4/direct/refund`, sample curl `/v4/refund`.
