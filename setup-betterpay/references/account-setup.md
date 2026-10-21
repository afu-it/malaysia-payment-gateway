# Betterpay Account Setup

Checked: 2026-10-21.

## Registration

- Register at `https://www.betterpay.me/`. The official WP plugin readme and its "Register" button point there.
- The docs sidebar "SIGN UP NOW" link goes to `https://qlicknpay.com/v1/pricing` (the legacy brand).
- The plans on the pricing page are Starter, Lite and Pro. Pick PRO or LITE before you build. The two products use different merchant IDs, dashboards, endpoints and hash methods.
- Required business documents and KYC steps: Not documented in official docs (checked 2026-10-21). Follow the onboarding flow or ask support.
- Pricing: `https://www.betterpay.me/v2/pricing.html`. Do not copy rates into code or docs; they change.

## Where Credentials Live

| Product | Dashboard page | What you get |
|---|---|---|
| PRO | API Management: `https://www.betterpay.me/merchant/v-api-management` (legacy `https://www.qlicknpay.com/merchant/v-api-management`) | Numeric Merchant ID, API Key |
| LITE | API Management: `https://lite.betterpay.me/merchant/api/management` | Merchant ID starting with `R`, API Key |

The docs Introduction calls this the "Manage API" tab and says to keep the credentials secure. The WP plugin readme says the Merchant ID and API key come from the Betterpay Dashboard at `https://www.betterpay.me/merchant/`.

Also set these in API Management:

- Your website / API URL and callback URL. Without them you get "Indirect error ... URL is not specified in the API Management setting".
- **Whitelist Domain**: the host of every `callback_url_be` you send must match it or the dashboard callback URL.
- Default return URLs (`callback_url_fe_succ` / `callback_url_fe_fail` defaults).
- Optional channels:
  - BNPL: tick "Buy Now Pay Later Payment (BNPL)", enter the BNPL Merchant ID and BNPL API Key, then click Update BNPL.
  - 2C2P eWallet: tick "2C2P eWallet", paste the 2C2P Merchant ID and Secret Key, choose the e-wallets, then click Update 2C2P eWallet.
  - PayPal: create a PayPal REST app. Use the sandbox Client ID/Secret for the demo account and the live ones for production.

LITE recurring callback URL: Dashboard > Recurring > Settings > Callback URL tab.

## Sandbox vs Live

- Sandbox (demo) merchant portal and APIs: `https://www.demo.betterpay.me`. Older docs use `https://www.demo.qlicknpay.com`.
- Live PRO: `https://www.betterpay.me`. Live LITE dashboard: `https://lite.betterpay.me`.
- Sandbox and live are separate accounts with separate merchant IDs and API keys. Never send a sandbox key to live or the reverse.
- Sandbox-only behaviour:
  - Refund approval happens on the Refund List (Bank Simulation) page under Transaction Report.
  - Batch token callbacks are triggered by hand (MPGS Tokenization > Batch Token Approval).
  - Get Disburse Records is disabled.
  - `sandbox_charge_status` / `simulation_status` simulate card results.
- How a sandbox account is issued: Not documented in official docs (checked 2026-10-21). Ask support.

## Environment Variables (suggested names)

```
BETTERPAY_PRODUCT=pro            # pro | lite  (decides MD5 vs HMAC and endpoint set)
BETTERPAY_ENV=sandbox            # sandbox | production
BETTERPAY_BASE_URL=https://www.demo.betterpay.me
BETTERPAY_MERCHANT_ID=<MERCHANT_ID>
BETTERPAY_API_KEY=<API_KEY>
BETTERPAY_CALLBACK_URL=https://<your-domain>/payments/betterpay/callback
BETTERPAY_RETURN_SUCCESS_URL=https://<your-domain>/checkout/success
BETTERPAY_RETURN_FAIL_URL=https://<your-domain>/checkout/failed
```

- Keep `BETTERPAY_API_KEY` server-side only. It signs every request and verifies every callback.
- The callback host must match the API Management Whitelist Domain for that environment.

## Payment Method Activation

- FPX B2C/B2B, cards, e-wallets, DuitNow QR, BNPL and IPP depend on the plan and on activation on your account. Check the dashboard and use Get Bank List (PRO) / Get Payment Channels (LITE) to see what is live (`status` Online/Offline, `0`/`1`).
- **Card tokenization**: not enabled by default. Email `support@qlicknpay.com` (docs) or `support@betterpay.me` (plugin readme).
- **FPX Refund API**: only for merchants with their OWN FPX seller ID. Production refunds also need a maker account, account number and organization ID from your bank portal.
- **Recurring**: PRO direct debit (FPX e-Mandate), OCBC collection batches, LITE card recurring. Ask support to enable what you need.

## Plugins (official)

| Platform | Source | Notes |
|---|---|---|
| WooCommerce | `https://wordpress.org/plugins/betterpay/` | Settings: Merchant ID, API key. Set website URL in the dashboard |
| OpenCart | OpenCart marketplace (Betterpay extension) | |
| PrestaShop | `https://www.qlicknpay.com/merchant/plugin/prestashop/{1.4\|1.6\|1.7}/Qlicknpay.zip` | |
| Magento 2.3+ | `https://www.qlicknpay.com/merchant/plugin/magento/Betterpay.zip` | Currency MYR or IDR |
| Drupal 8 + Commerce | `https://www.drupal.org/project/paydirectfpx` | Do not edit the endpoint URL |
| Ecwid | Choose "PayDirect FPX" | |
| EasyStore | Add payment method "Betterpay" | Sandbox: add `?testing=true` to the checkout URL |

Lazada and Shopee product sync guides also exist (docs, Third Party Platform Guides). They are not payment acceptance.

Do at least one test transaction before selling (docs, every plugin section).

## Settlement And Payout Notes

- Settlement timing per plan is shown on the pricing page. On 2026-10-21 it showed:
  - FPX online banking: T+1.
  - Cards on Lite: T+2.
  - Pro: "Instant" bank-transfer settlement to Maybank, OCBC or Bank Islam accounts.
- Confirm the timing in your agreement. Do not hardcode it.
- How the Starter/Lite/Pro plans map to the PRO and LITE API products: Not documented in official docs (checked 2026-10-21). Use the merchant ID format (`R` prefix = LITE) and ask support.
- Reconcile with Get Settlement (`method`, `date`, `invoice_no`, `amount`, `fee`, `due`), Get Balance (`fee`, `total_balance`) and Get Disburse Records (`date_paid`, `reference_no`, `invoice_no`, `amount_paid`, `payment_to`). Disburse data is available T+3 days after the transaction date.
- Support: `support@betterpay.me` (plugin readme), `support@qlicknpay.com` (docs).
