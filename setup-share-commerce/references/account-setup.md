# Share Commerce Account Setup

Use this reference when explaining Share Commerce onboarding, credentials, staging vs production, IP whitelisting, callback registration, plugin setup, payment methods, pricing and settlement caveats. Checked 2026-10-21.

## Source URLs

- `https://share-commerce.com/`
- `https://share-commerce.com/solutions-and-services/payment-gateway-solutions/`
- `https://share-commerce.com/contact-us/`
- `https://devdocs.share-commerce.com/PreRequisite`
- `https://devdocs.share-commerce.com/Plugins/WooCommerce`, `/Plugins/OpenCart`, `/Plugins/Magento`, `/Plugins/Prestashop`, `/Plugins/EasyStore`
- `https://portal.share-commerce.com/auth` (merchant portal login, linked from the site menu)

## Company

Share Commerce Sdn. Bhd., Bukit Jalil, Kuala Lumpur. Offers an online payment gateway plus white-label gateway, in-store terminals, terminal management and NFC e-receipts. The payment gateway page lists web and app payments, payment links, one-click checkout (tokenisation), recurring billing and multi-currency.

## Onboarding (sales-led, no self sign-up documented)

1. Contact Share Commerce through `https://share-commerce.com/contact-us/` (form, WhatsApp) or `support@share-commerce.com`.
2. Before onboarding, prepare (PreRequisite page):
   - **Server IP**: Share Commerce whitelists your server IP range for Enquiry calls (Recurring Enquiry also requires it).
   - **Callback URL**: Share Commerce stores it and POSTs transaction results to it. It is not a per-request field.
3. After onboarding, Share Commerce emails, separately for UAT and production:
   - Merchant ID
   - Secret Key
   - Testing credentials
4. Business documents required for KYC: not documented in official docs (checked 2026-10-21). Ask Share Commerce.

There is no documented self-service dashboard page for keys or webhook settings. The merchant portal (`https://portal.share-commerce.com/auth`) exists; what it exposes is not documented publicly.

## Environments and Config

| Item | Staging (UAT) | Production |
| --- | --- | --- |
| Gateway base | `https://stagingpayment.share-commerce.com` | `https://payment.share-commerce.com` |
| Merchant ID | UAT value | Production value |
| Secret Key | UAT value | Production value |
| Callback URL | registered with Share Commerce for UAT | registered for production |
| Server IP whitelist | UAT servers | Production servers |

Suggested environment variables (names are this skill's convention, not Share Commerce's):

```text
SHARECOMMERCE_ENV=staging            # or production
SHARECOMMERCE_MERCHANT_ID=<MERCHANT_ID>
SHARECOMMERCE_SECRET_KEY=<SECRET_KEY>
SHARECOMMERCE_REDIRECT_URL=https://example.com/payment/return
SHARECOMMERCE_CALLBACK_URL=https://example.com/payment/callback   # must match what Share Commerce registered
```

Never commit the Secret Key. Never mix UAT keys with the production base URL.

## Payment Methods

Codes documented in Appendix 2: `CARD`, `FPX`, `ALIPAY`, `BOOST`, `GRAB`, `MAYBANKQR`, `MCASH`, `PRESTO`, `RAZER`, `SHOPEE`, `TNG`, `WECHAT_CN`, `WECHAT_MY`, `BNPL_MYIOU`. Which methods are active on an account is set by Share Commerce during onboarding; how to request activation is not documented. Send `PaymentMethod` empty to let the customer choose on the Share Commerce page.

## Plugins (official)

All plugins ask for Merchant ID and Secret Key ("Different between UAT and Production") and an environment switch.

| Platform | Source | Notes |
| --- | --- | --- |
| WooCommerce 8.x | `https://github.com/share-commerce/Plugin.WooCommerce-8.x` | Upload via Plugins > Add New > Upload Plugin. Hash Type `sha256`. Redirects with GET; callback reads JSON body and replies `OK`. |
| WooCommerce 5.9.0 | `https://github.com/share-commerce/Plugin.WooCommerce-5.9.0` | Older line |
| OpenCart 3.x | `https://github.com/share-commerce/Plugin.Opencart3` | Extensions > Payment > Share Commerce Payment. Hash `sha256`. |
| OpenCart 4.x | `https://github.com/share-commerce/Plugin.Opencart4` (link on docs page reads `Plugin.Opencart4.-`) | Upload `sharecommerce_payment.ocmod.zip` in Extension > Installer |
| Magento 2 | `https://github.com/share-commerce/Plugin.Magento.2.3.6` | Copy files, run `php bin/magento setup:upgrade && php bin/magento cache:flush`, then Stores > Configuration > Sales > Payment Methods |
| PrestaShop 1.7 | `https://github.com/share-commerce/Plugin.Prestashop.1.7.-` | Modules > Module Catalog, search "sharecommerce" |
| EasyStore | built in | Settings > Payments > add Share Commerce, enter Merchant ID and Secret Key |

The PrestaShop and Magento plugins send the legacy request without `PaymentMethod`/`Versioning` and hard-code `MYR` (PrestaShop). WooCommerce 8.x and OpenCart send `Versioning=7`.

## Pricing, Settlement and Payouts

- Pricing: not published. Share Commerce quotes per merchant; see `https://share-commerce.com/contact-us/`. Do not quote numbers.
- Settlement schedule and payout timing: not documented in official docs (checked 2026-10-21).
- Refunds: no refund API documented; same-day void only via `/Payment/Void`. Ask Share Commerce how later refunds are processed.
- Reports: the marketing site mentions comprehensive reporting; no reporting API is documented.

## Go-Live Checklist

1. Production Merchant ID and Secret Key received and stored as secrets.
2. Production callback URL registered with Share Commerce and reachable over HTTPS.
3. Production server egress IPs whitelisted with Share Commerce.
4. Signer tested in UAT against a real transaction (some doc sample hashes do not reproduce).
5. Callback, redirect and Enquiry paths all verify SCSign and settle idempotently.
6. `Versioning` value confirmed with Share Commerce.
