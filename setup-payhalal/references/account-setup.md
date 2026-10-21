# Payhalal Account Setup

Use this reference when explaining Payhalal registration, merchant requirements, where credentials live, UAT vs live, dashboard URL configuration, plugins and pricing. Checked 2026-10-21.

## Source URLs

- `https://payhalal.my/home_v3/register/`
- `https://payhalal.my/home_v3/pricing/`
- `https://payhalal.my/home_v3/developers/`
- `https://payhalal.my/home_v3/faq/`
- `https://merchant.payhalal.my/login.php`
- `https://souqa-fintech.gitbook.io/api-docs/payhalal-introduction`
- `https://souqa-fintech.gitbook.io/api-docs/payments/payment-response-callback/callback-url-to-your-server`
- `https://github.com/SouqaFintech/payhalal-woocommerce-plugin` (README and wiki)

## Who Runs Payhalal

Payhalal is operated by Souqa Fintech Sdn Bhd (Bangsar South, Kuala Lumpur), described on the site as a Merchant Acquirer / Non Bank Financial Institution, with Payhalal Inc listed as the payment gateway entity. Support: `tech_support@payhalal.my` (plugin README), support ticket at `https://payhalal.freshdesk.com/support/tickets/new`.

## Merchant Requirements

From the GitBook introduction:

1. Your products and services must be 100% Syariah compliant.
2. Your company must be located in Malaysia.

## Registration

1. Register at `https://payhalal.my/home_v3/register/` (the older GitBook link `https://payhalal.my/merchant-register` points to the previous form).
2. The form asks for: login email and password, payment channels (Online Payments, Payment Terminal, Soundbox) and online package, company details (full name, phone, identification number, company name, company registration number, address, state), bank details for settlement (bank, account number, holder name), and website URL plus shopping cart (WooCommerce, PrestaShop, Magento, Others). A website is optional.
3. Verify your email address and activate the account.
4. Log in to the Merchant Dashboard at `https://merchant.payhalal.my/login.php` and complete account verification.
5. Documents required after submission: copy of owner/director ID, copy of SSM documents, latest bank statement. The bank account must match the form.

Until verification is complete the account is in UAT mode and only has sandbox (testing) API keys.

## Where Credentials Live

- Merchant Dashboard > General > Developer Tools: select the app and copy its keys (App ID / App Key and App Secret). Developer Tools are only available to Online merchants (plugin wiki).
- Live and testing keys are different pairs (docs examples use `live-...` and `testing-...` prefixes). The OpenCart extension stores `app_key`, `app_secret`, `app_key_testing`, `app_secret_testing`.
- `merchant_id` for the reconciliation APIs is the merchant login email.

## Dashboard URL Configuration

In Developer Tools > Edit App, set for each app (UAT and live):

| Setting | Purpose |
| --- | --- |
| Return URL | Browser return after payment (POST with result fields). |
| Success URL | Browser return on success. |
| Cancel URL | Where the customer goes on cancel. |
| Callback URL | Server-to-server result. Optional; if empty Payhalal sends nothing to your server. |

The official WooCommerce plugin points Return, Success and Callback to `https://your-site/?wc-api=payhalal_callback` and Cancel to `/cart`.

## Environment Variables (suggested names)

```text
PAYHALAL_ENV=uat            # or live
PAYHALAL_APP_ID=<APP_ID>
PAYHALAL_APP_SECRET=<APP_SECRET>
PAYHALAL_MERCHANT_ID=<MERCHANT_LOGIN_EMAIL>
PAYHALAL_PAY_URL=https://api-testing.payhalal.my/pay        # live: https://api.payhalal.my/pay
PAYHALAL_RECONCILE_BASE=https://api-merchant-uat.payhalal.my # live: https://api-merchant.payhalal.my
```

Keep separate values for UAT and live. Never put the App Secret in client code.

## Payment Methods

The pricing page lists online acceptance of FPX (B2C and B2B), credit, debit and foreign cards, e-wallets (GrabPay, Touch 'n Go, DuitNow, ShopeePay) and BNPL (GrabPay Later, Atome, SPayLater). The customer chooses the channel on the hosted payment page; the API has no field to preselect a channel (not documented). The callback `channel` field reports what was used.

## Pricing

See `https://payhalal.my/home_v3/pricing/`. Payhalal sells packages with setup and annual fees and per-method rates; do not quote numbers from memory because they change.

## Plugins

- WooCommerce: `https://github.com/SouqaFintech/payhalal-woocommerce-plugin` (v1.0.5, Checkout Block and HPOS support, test mode toggle, test and live App ID / Secret Key fields, recurring via `payment-cycle` attribute when the MID is enabled).
- OpenCart 3.x: `https://github.com/SouqaFintech/opencart3.x-payhalal-extension`
- OpenCart 1.x: `https://github.com/SouqaFintech/opencart1.x-payhalal-extension`
- PrestaShop: `https://github.com/SouqaFintech/payhalal-prestashop-plugin`
- The `payhalal/woocommerce-plugin` (2018) and `payhalal/php-api` (README only) repositories are old; prefer the SouqaFintech ones.

## Settlement And Refunds

Settlement goes to the bank account submitted at registration. Settlement timing and a refund API are not documented in official docs (checked 2026-10-21). The FAQ tells customers to contact the store for refunds; ask Payhalal support how merchant-initiated refunds are processed.

## Go-Live Checklist

1. Account verified (SSM, ID, bank statement accepted) and live keys visible in Developer Tools.
2. Live app has Return, Success, Cancel and Callback URLs set.
3. Code switched to live pay URL, live App ID and live secret together.
4. One small live transaction (at least RM5.00) settled through the callback and confirmed with the reconcile API.
