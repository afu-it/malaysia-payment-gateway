# GrabPay & PayLater Account Setup (Malaysia)

Use when explaining how a Malaysian merchant gets GrabPay and PayLater by Grab on a web store, which credentials exist, how testing works, plugins, branding, and settlement. Checked 2026-10-21.

## Source URLs

- `https://www.grab.com/my/merchant/checkout-solutions/` (Malaysia merchant page; `/my/merchant/pay/` redirects here)
- `https://www.grab.com/my/merchant/pay/apply/` (sign-up form linked as "Sign Up as a GrabPay and PayLater Merchant")
- `https://developer.grab.com/docs/paylater-integrations-launchpad-v2` (also links a Malaysia Google Form for merchant interest)
- `https://developer.grab.com/docs/payment-otc/api/v2` (Getting started, Credentials, Settlement)
- `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/prerequisites`
- `https://developer.grab.com/docs/paylater-integrations-launchpad-v2/plugins-woocommerce`
- `https://merchant.grab.com/en-my/brand-centre/paylater-by-grab` (Merchant Resource Centre)
- `https://help.grab.com/merchant/en-my/360001721012-GrabPay-Merchants` (Help Centre)

## Who should apply

Grab Malaysia: "Merchants who have an online web store and want to add GrabPay and/or PayLater as checkout options." One sign-up covers both GrabPay and PayLater.

## Onboarding steps

1. Submit details via "Sign Up as a GrabPay and PayLater Merchant" (`https://www.grab.com/my/merchant/pay/apply/`). A Grab representative contacts you for business information and documents. The exact document list is not published; Grab collects it after first contact.
2. After verification, a Grab Integration Manager guides the integration.
3. Integration Manager collects your test Redirect URL and Webhook URL, then issues test API credentials and a test Grab customer account (current OTC docs say these are Production test credentials).
4. Build and test every step (auth, init, authorise, complete, notifications, refunds).
5. Give the Integration Manager your live Redirect URL and Webhook URL; receive live credentials.
6. For recurring/tokenised payments ask your Account Manager to enable Tokenisation and the `payment.recurring_charge` scope.

Grab says account setup is manual; there is no self-serve sandbox. Scopes are assigned by Grab after review ("We want to make sure the scopes requested by a partner is absolutely needed by them and they have the required legal approval"). Once assigned, they show in the console at `https://developer.grab.com`.

## Credentials and env vars

| Grab credential | Suggested env var | Notes |
|---|---|---|
| `partner_id` | `GRAB_PARTNER_ID` | Not secret, but keep server-side |
| `partner_secret` | `GRAB_PARTNER_SECRET` | Secret; partner HMAC key |
| `client_id` | `GRAB_CLIENT_ID` | OAuth client |
| `client_secret` | `GRAB_CLIENT_SECRET` | Secret; token exchange and POP key |
| `merchant_id` | `GRAB_MERCHANT_ID` | One per currency (MYR) |
| redirect URL | `GRAB_REDIRECT_URI` | Must exactly match the registered URL |
| base URL | `GRAB_API_BASE` | `https://partner-api.grab.com` or `https://partner-api.stg-myteksi.com` |

Plugins (WooCommerce) ask for the same five values: Partner ID, Partner Secret, Client ID, Client Secret, Merchant ID, "in the Activation email that you received from Grab". The official SDK README says to retrieve them "from Developer Home".

Grab security guidance: store client IDs and secrets in a KMS, not flat config files, not in source control, not in Redis; store user tokens in an encrypted database with least privilege; never send secrets or tokens to the browser or app; if leaked, contact your GrabPay account manager to rotate. In production, do not test with cURL or Postman.

## Payment methods

- GrabPay wallet is always shown.
- PayLater by Grab options appear in Grab checkout if enabled for the merchant and eligible for the user: pay next month (`POSTPAID`) and instalments (`INSTALMENT_4`, `INSTALMENT_8`, `INSTALMENT_12`). Grab Malaysia: "pay today with GrabPay, pay next month or over 4, 8 or 12 installments with PayLater by Grab."
- Merchants can hide PayLater per transaction with `hidePaymentMethods`.
- Price Dividers require GrabPay and PayLater to be activated first.

## Pricing

Not published as numbers on the official Malaysian merchant page. Grab provides pricing after sign-up. See `https://www.grab.com/my/merchant/checkout-solutions/`. Do not quote fees.

## Settlement

- Marketing page (MY): "Settlement is made to you within 5 working days with no risk involved."
- Reports (OTC docs): SFTP (partner-hosted: give Grab your IP for whitelisting and public SSH key; or Grab-hosted: Grab gives its IP and public SSH key) or Grab Merchant Portal `https://merchant.grab.com/` (access from your Integration Manager). Delivery T+1 12:00:00 daily, cut-off 23:59:59 daily, CSV (pipe delimited) or XLS.
- Merchant wallet is cashed out daily, so its balance is zero at the start of the business day; refunds can fail with insufficient merchant balance.

## Plugins and no-code

Grab Malaysia lists pre-built integrations for Shopify, WooCommerce, Magento, EasyStore (Wix coming soon), payments partners, or Direct API.

- WooCommerce: WooCommerce > Settings > Payments > GrabPay Gateway, enable, choose Production Mode in API Environment, enter Partner ID, Partner Secret, Client ID, Client Secret, Merchant ID. Price Divider settings in the same window (min amount, max amount up to 100,000, product page placement; fixed at 4 instalments).
- Magento: Admin > Stores > Settings > Configuration > Sales > Payment Methods > "Grab - Pay today or later at 0% interest" > Price Divider Tool.
- Shopify Price Divider: install via `https://api.grabpay.com/mb-shopify/install` (enter `myshop.myshopify.com`); installation can take up to 48 hours.
- Custom web Price Divider: `<script src="https://cdn-gp01.grabpay.com/paylater/v2/grab-widget-paylater.js" async></script>` and `<div class="grab-price-divider-widget" data-widget-type="single_line_short_v2" data-currency-code="MYR" data-money-format="RM {{amount}}" data-product-price="120" data-min-price="10" data-max-price="300" data-widget-size="medium" data-widget-alignment="left"></div>`. Required: `data-widget-type` (`single_line`, `single_line_short`, `single_line_short_v2`, `multi_line`), `data-currency-code`, `data-money-format` (keep `{{amount}}`), `data-product-price`. Optional: `data-min-price`, `data-max-price`, `data-widget-size` (`very_small` to `very_large`), `data-widget-alignment` (`left`, `right`, `center`, `right_left`, `left_right`, `center_left`, `center_right`, `left_center`, `right_center`).

## Branding rules (Integrations Launchpad v2)

- Checkout method name: "Grab", with the Grab logo.
- Suggested text: short "Pay today or pay later. Earn rewards, 0% interest."; long "Earn GrabRewards points and enjoy the flexibility to pay now, next month or over 4 interest-free instalments."
- Responsible spending: say "Available amount", "Monthly payment", "Spending limit"; do not say "Credit limit" or "Loan Repayment", or messages like "Need extra cash?".
- Assets (checkout button, payment badge, lockups, banners) are on the launchpad brand-assets pages.
- Note: launchpad copy says "interest-free" and "4 instalments"; Malaysia now offers 4, 8 or 12, and per-instalment fees may apply for some plans (`processingFeePercentage` in the tokenisation payment-methods response). Check current Grab Malaysia terms before publishing consumer copy.

## Support

- Integration issues: your GrabPay Integration Manager.
- Help Centre: `https://help.grab.com/merchant`.
- Security reports: `https://hackerone.com/grab`.
