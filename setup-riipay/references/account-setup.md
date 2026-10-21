# Riipay Account Setup

Use this reference when explaining Riipay merchant registration, portal setup, credentials, staging vs production, plugins, refunds, pricing, and settlement. All sources checked 2026-10-21.

## Source URLs

- Merchant page: `https://riipay.my/business/`
- Merchant FAQ: `https://riipay.my/faq` (content served from `https://api.riipay.my/website/faq/merchant`)
- Merchant sign-up form: `https://secure.riipay.my/merchant-sign-up`
- Developer guide, Getting Started: `https://developers.riipay.my/getting-started.md`
- WooCommerce plugin: `https://wordpress.org/plugins/riipay-for-woocommerce/`

## Product Positioning

- Riipay is an additional payment method; a merchant can keep its existing payment gateway (merchant FAQ).
- Shoppers split a purchase into interest-free instalments with a debit or credit card.
- The merchant page lists use cases: Online Store (e-commerce platform integration), In Store (QR solution), Non-Store Retailer (payment link). Only the online redirect API is documented for developers.

## Registration

1. Submit details on the sign-up form `https://secure.riipay.my/merchant-sign-up` (linked from the website as the merchant register URL). FAQ: "Submit your details here and our team will reach out to you shortly."
2. Required business documents are Not documented in official docs (checked 2026-10-21); Riipay's team states them during onboarding.
3. After successful registration Riipay provides merchant dashboard login credentials (Getting Started).
4. Staging access: use `https://merchant.uat.riipay.my`. How staging credentials are issued is not stated; ask Riipay during onboarding.

Contacts: `support@riipay.my` (website), `contact@riipay.my` (plugin readme and FAQ).

## Merchant Portal Setup

Portals:

- Staging: `https://merchant.uat.riipay.my`
- Production: `https://merchant.riipay.my`
- Profile edit page (plugin): `https://merchant.riipay.my/profile/update` and `https://merchant.uat.riipay.my/profile/update`

In the profile settings set:

- Return URL: page the customer returns to after completing or cancelling payment.
- Callback URL: endpoint that receives server-to-server payment updates.
- Callback Method: GET, POST x-www-form-urlencoded, or POST JSON. Default POST JSON.
- Bank Account Details: for settlement.

Credentials on the profile page:

- Merchant Code
- Secret Key

## Environment Variables

Suggested names (the docs do not prescribe any):

```text
RIIPAY_BASE_URL=https://secure.uat.riipay.my     # production: https://secure.riipay.my
RIIPAY_MERCHANT_CODE=<MERCHANT_CODE>
RIIPAY_SECRET_KEY=<SECRET_KEY>
RIIPAY_RETURN_URL=https://example.com/payments/riipay/return
RIIPAY_CALLBACK_URL=https://example.com/payments/riipay/callback
```

Keep staging and production values separate. Never expose `RIIPAY_SECRET_KEY` to the browser.

## Integration Options

- FAQ: plugins exist for leading e-commerce platforms; merchants not on a platform receive API documentation and integration help. A platform integration "could take just a day".
- Public plugin source: only WooCommerce (`riipay-for-woocommerce`, author Riipay, v1.0.27, last updated 2025-01-06). Settings: Merchant Code, Secret Key, Minimum Order Amount (default 0), Maximum Order Amount (default 1000), Environment (production or sandbox), Visibility (everyone or admins only), product price text options. Suggested Return URL `https://<site>/wc-api/riipay` and Callback URL `https://<site>/wc-api/riipay?callback=1`.
- Other platform plugins (for example EasyStore or Shopify, mentioned in the MPG listing) have no public source or docs; ask Riipay.

## Payment Methods

Riipay is one payment method (BNPL instalments). The customer pays with a debit or credit card on a bank-hosted page. FPX, DuitNow, and e-wallets are not offered through Riipay per the docs.

## Refunds

Merchant FAQ: "Customers refund is subjected to merchant's policy. Once refund is approved from merchant's side, customers will be credited with agreed refund amount." No refund API is documented; handle refunds through Riipay's merchant support or portal.

## Pricing

Do not copy numbers. Merchant FAQ: fees vary by industry and average monthly turnover, charged as a percentage of each successful Riipay transaction; contact Riipay for the fee structure. See `https://riipay.my/business/` and `https://riipay.my/faq`.

## Settlement

- The portal holds bank account details "for settlement purpose".
- Settlement timing for online merchants is Not documented in official docs (checked 2026-10-21). The website's separate Riipay Credit (agent stock purchase) page says "Weekly settlement of full payment"; do not assume that applies to online checkout without confirming.

## Production Readiness Checklist

1. Production Merchant Code and Secret Key stored as server secrets.
2. Production portal Return URL, Callback URL, and Callback Method match the deployed handlers.
3. Handler parses the configured callback method.
4. Order limits (minimum RM1.00, current maximum) agreed with Riipay.
5. A test transaction completed on staging with `S`, `F`, `410` cancel, and `A` paths.
