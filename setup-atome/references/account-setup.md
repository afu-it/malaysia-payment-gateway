# Atome Account Setup

Use this reference when explaining how a Malaysian merchant gets Atome credentials, test vs production setup, plugin options, pricing, and settlement. Checked 2026-10-21.

## Source URLs

- `https://www.atome.my/en-my/business`
- `https://partner.apaylater.com/`
- `https://partner.apaylater.com/docs/Help/FAQ`
- `https://partner.apaylater.com/docs/Help/CustomerPaymentProcess`
- `https://partner.apaylater.com/docs/WooCommerce/PluginIntegration`
- `https://doc.apaylater.com/v2/swagger.yaml`

## Product Positioning

Atome is a Buy Now Pay Later (BNPL) provider. The Malaysia site describes it as "3 easy payments. 0% interest." for shoppers. The merchant gets paid by Atome; the shopper repays Atome in instalments. Atome also lists merchants in the Atome app for extra traffic.

## Registration

There is no self-serve sign-up or public sandbox sign-up in the official docs (checked 2026-10-21).

1. Contact Atome sales through "Get in touch" on `https://www.atome.my/en-my/business`, or email `merchant@atome.my` (retailer contact listed on the Malaysia site footer).
2. Atome assigns an Account Manager.
3. The Account Manager issues the API username and password (Basic auth) for test and for production.
4. Ask the Account Manager for the callback signing secret and its exact `X-Signature` encoding (the spec says verification details are given on request).
5. Ask for test shopper accounts for the test environment.

Required business documents and KYC checklist: not documented in official docs (checked 2026-10-21). Expect the Account Manager to request them during onboarding.

## Credentials And Env Vars

| Env var | Value |
| --- | --- |
| `ATOME_BASE_URL` | `https://api.apaylater.net/v2` (test) or `https://api.apaylater.com/v2` (production) |
| `ATOME_API_USERNAME` | Basic auth username ("API Key" in plugin settings) |
| `ATOME_API_PASSWORD` | Basic auth password ("API Password" in plugin settings) |
| `ATOME_CALLBACK_SECRET` | Shared secret for `X-Signature` HMAC-SHA256, from the Account Manager |
| `ATOME_COUNTRY` | `MY` |
| `ATOME_CURRENCY` | `MYR` |

Keep test and production values in separate environment configs. Never mix a test username with the production base URL.

A merchant portal exists (the spec says `merchantReferenceId` appears there and in the settlement report as "E-commerce Platform Order ID"), but its URL and menu paths are not documented in the public docs (checked 2026-10-21).

## First-Time Checks

1. Call `POST {base}/auth` with `{"countryCode":"MY","callbackUrl":"<your callback URL>"}` once per environment. A 200 confirms credentials and that Atome can reach your callback.
2. If your server or WAF filters by IP, allow Atome Malaysia outbound IPs: `47.254.230.20`, `47.254.238.153` (production) and `47.254.247.54` (test).
3. Run a small-amount test payment (at least RM10.00) through to `PAID`, then a partial refund, a full refund, and a cancel of an unpaid payment.

## Platform Plugins

Official plugin guides exist for Shopify, WooCommerce, Magento, PrestaShop, OpenCart, EasyStore, Wix, and Salesforce on `https://partner.apaylater.com/`. The WooCommerce plugin settings show what Atome expects from any integration:

- Country/Region (drives currency).
- Environment: Test or Production.
- API Key and API Password.
- Optional min/max order amount for showing Atome.
- Cancel timeout, default 720 minutes (12 hours).
- Optional price divider ("or 3 instalments of ...") and SKU permission (lists products in the Atome app).

## Malaysia Notes

- Currency `MYR`, amounts in sen, API minimum RM10.00.
- Developer portal FAQ lists Malaysia as "Pay in App" only (no "Continue in Browser"), so Malaysian shoppers complete payment in the Atome app.
- The customer-process page says plugins display Atome only when the total is over MYR50. That is a plugin display rule; confirm your contracted min/max with the Account Manager.
- Atome asks merchants to give one week's notice before large campaigns.

## Pricing And Settlement

- Merchant pricing (MDR) is not published on official Atome pages (checked 2026-10-21). It is negotiated with Atome sales. Link: `https://www.atome.my/en-my/business`.
- Settlement schedule and payout timing: not documented in official docs (checked 2026-10-21). The spec mentions a settlement report that shows `merchantReferenceId`; get the schedule from your contract.
- Once a payment is `PAID`, missed shopper instalments do not change the merchant's payment status.
