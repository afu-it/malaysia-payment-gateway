# eGHL Account Setup

Use this reference when explaining eGHL/ADAPTIS registration, credentials, test vs live, plugins, pricing, and operational notices. Checked 2026-10-21.

## Source URLs

- `https://my.nttdatapay.com/en`
- `https://my.nttdatapay.com/en/payment-services/adaptis-e-commerce`
- `https://my.nttdatapay.com/en/contact-us`
- `https://my.nttdatapay.com/en/company/our-story`
- `https://my.nttdatapay.com/en/downtime-announcements`
- `https://bitbucket.org/eghl/adaptis-woocommerce/src/main/` (installation guide PDF and plugin)
- `https://bitbucket.org/eghl/adaptis-magento/src/main/` (installation guide PDF and plugin)
- `https://bitbucket.org/eghl/adaptis-opencart/src/main/`
- `https://bitbucket.org/eghl/merchantapi/src/master/README.md`

## Brand And Entity

Official sources state:

- The plugins are titled "ADAPTIS Payment Gateway (formerly eGHL)" (WooCommerce 8.2.4, Magento 2.1.7, OpenCart). The WooCommerce guide revision history records "Updated Guide for ADAPTIS Rebranding" on 03-12-2025.
- The ADAPTIS guides are copyright "NTT DATA Payment Services Group of Companies". The OpenCart plugin links ADAPTIS to `https://my.nttdatapay.com/`.
- The NTT DATA Payment Services Malaysia contact page lists NTT DATA ePayments Sdn. Bhd. (201301010585 / 1040427-M) among its Malaysian entities, along with NTT DATA Payment Services Sdn Bhd, NTT DATA eCommerce Solutions Sdn. Bhd., e-pay (M) Sdn Bhd, and NTT DATA Paysys Solutions (M) Sdn. Bhd.
- NTT DATA Payment Services is a wholly owned subsidiary of NTT DATA Japan (Our Story page).

The official pages do not say which entity signs a given eGHL merchant agreement. Check the merchant's contract. The payment hosts are still `e-ghl.com` in the current official plugins.

## Product Positioning

ADAPTIS e-Commerce (the eGHL online gateway) is described as an all-in-one internet payment gateway for local and cross-border payments, with:

- Direct Link checkout (one-off payments by card, e-wallet, or online banking).
- Email Payment Link.
- Recurring Payment.
- Easy Payment Plan (0% instalment).
- Payment acceptance: cards, online banking, e-wallets, instalments, BNPL, others.

## Registration

1. Submit an enquiry at `https://my.nttdatapay.com/en/contact-us` (form asks for name, email, phone, company name, company website, country, enquiry reason).
2. Sales and onboarding are handled by an account manager. The WooCommerce guide says the Merchant Service ID and Password are "obtained from your account manager".
3. Required KYC documents are not listed on official public pages; ask the account manager.

Helpdesk numbers (contact page): NTT DATA Payment Services Sdn Bhd customer care `+603-6286 5222`. The SDK READMEs name `support.eghl@ghl.com` for developer repo access; confirm it is still monitored after the rebrand.

## Credentials

| Item | Where it comes from | Suggested env var |
| --- | --- | --- |
| Merchant Service ID (`ServiceID`, 3 chars) | Issued by eGHL/ADAPTIS account manager | `EGHL_SERVICE_ID` |
| Merchant Password (hash secret) | Issued by eGHL/ADAPTIS account manager | `EGHL_MERCHANT_PASSWORD` |
| Payment URL | Test or live URL from `eghl-api.md` | `EGHL_PAYMENT_URL` |
| Return URL | Your app | `EGHL_RETURN_URL` |
| Callback URL | Your app | `EGHL_CALLBACK_URL` |

A self-serve merchant portal location for keys is not documented in official public sources. Credentials are delivered by the account manager.

## Test vs Live

- Test (UAT) URL: `https://pay.e-ghl.com/IPGSG/payment.aspx`. Plugins call this "Enable ADAPTIS Testing" or "Test Mode".
- Live URL: `https://securepay.e-ghl.com/IPG/Payment.aspx`.
- Test and live credentials differ. SDK demos use `SIT` / `sit12345` on the test URL only.
- Get UAT test cards and bank test data from the account manager (not published).

## Payment Method Activation

- `PymtMethod=ANY` shows every method enabled on your Service ID.
- Magento plugin options: All Payment Methods (`ANY`), Credit Card (`CC`), Direct Debit (`DD`), e-Wallet (`WA`).
- Which methods (FPX banks, cards, e-wallets, BNPL, instalment) are enabled is set on the merchant account by eGHL/ADAPTIS.
- Supported currencies listed by the WooCommerce plugin include MYR, SGD, USD, THB, PHP, IDR, and others; actual availability depends on your contract.

## Official Plugins

| Platform | Repo | Notes |
| --- | --- | --- |
| WooCommerce | `bitbucket.org/eghl/adaptis-woocommerce` | v8.2.4, PHP 8.1+; callback `/wp-json/eghl/v1/server`, return `/wp-json/eghl/v1/redirect` |
| Magento 2.x | `bitbucket.org/eghl/adaptis-magento` | v2.1.7; config under Stores > Configuration > Sales > Payment Methods |
| OpenCart 3.x | `bitbucket.org/eghl/adaptis-opencart` | `ghl.ocmod.zip` |
| WHMCS 6.x/7.x | `bitbucket.org/eghl/adaptis-whmcs` | zip dated 2025-07-09 |
| PrestaShop 1.6 | `bitbucket.org/eghl/adaptis-prestashop-1.6.x` | older eGHL-branded module |

Plugin settings are Merchant Service ID, Merchant Password, test mode, page timeout, and invoice prefix. If one eGHL account serves several stores, use a unique invoice prefix per store (WooCommerce guidance).

Other official plugin settings worth mirroring in a custom build:

- Magento: Payment URL (test MY, test PH, production), Pay Method (`ANY`/`CC`/`DD`/`WA`), Currency Type (`Base` or `Display` currency sent as the payment amount), page timeout (default 780), order statuses for success/pending/failed/cancelled. Its `config.xml` ships the public demo pair `SIT` / `sit12345` as defaults; replace them before go-live.
- WooCommerce: "Enable OCP" (card remembered with shopper consent, `TokenType=OCP`), "Sequential Order Number Plugin" (looks the order up via `Param6`), "Checkout Page Redirect" for failed payments.
- PrestaShop 1.6: Mode = Test, Production, or Localhost (Localhost is internal; never use it).

Official PHP packages on Packagist (vendor `e-ghl`): `e-ghl/merchant-api`, `e-ghl/omnipay`, `e-ghl/param-validator`, `e-ghl/exception`. The merchantapi README says the hashing package is in a private repo (access via `support.eghl@ghl.com`); the public `eghl/eghl_hash` repo holds the same class.

## Pricing And Settlement

- Pricing: not published for the e-commerce gateway. Request a quote via `https://my.nttdatapay.com/en/contact-us`. Do not quote rates.
- The site links a card interchange fee PDF (in-store card acquiring): `https://d1aixqkujf9yos.cloudfront.net/NTT_Data_Cardpay_Interchange_Fee_095e800e2c.pdf`.
- Payout schedule and settlement reports: not documented in official public sources.

## Operational Status

- Scheduled and ad hoc downtime (FPX, bank card channels): `https://my.nttdatapay.com/en/downtime-announcements`. Check before launches and when investigating failures.
