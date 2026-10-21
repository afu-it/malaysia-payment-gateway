# SPayLater Account Setup

Use this reference when explaining how a Malaysian business gets SPayLater, which credentials exist, sandbox vs live, env var names, plugins, settlement, and pricing caveats.

## Source URLs

- `https://product.shopeepay.com/integration/get-started`
- `https://product.shopeepay.com/products/functionalities`
- `https://product.shopeepay.com/products/e-commerce-plug-in/overview`
- `https://product.shopeepay.com/products/shopify-plug-in/add-shopeepay-at-checkout`
- `https://product.shopeepay.com/products/woocommerce-plug-in/setup-guideline`
- `https://wordpress.org/plugins/checkout-with-shopeepay/`
- `https://shopeepay.com.my/en/merchant`
- `https://shopeepay.com.my/en/spaylater`
- `https://seller.shopee.com.my/edu/article/16542` (Shopee marketplace seller SPayLater fees)

## Two Different Things Called "SPayLater For Merchants"

1. Shopee marketplace sellers: SPayLater is a buyer payment option inside Shopee. There is no API or integration; fees are set by Shopee Seller Centre (see the seller article above). This skill does not apply.
2. Merchants on their own website, app or shop: accept SPayLater through ShopeePay (ShopeePay Malaysia Sdn Bhd as platform partner and payment facilitator). In Malaysia SPayLater is a BNPL program provided by Monee Capital Malaysia Sdn. Bhd. This skill covers this case.

## Who Provides What (Malaysia)

- Product Functionalities page lists Malaysia (MYR) as live for ShopeePay Wallet and SPayLater; Linked Bank and Credit/Debit are "Coming soon". The Payment Gateway page separately lists `online_banking` and `duitnow_qr` for MY and `card` for all markets. Confirm what your agreement enables.
- The ShopeePay Malaysia merchant page lists SPayLater tenures 1M, 3M, 6M, 12M, 18M and 24M (18M and 24M for eligible users), says ShopeePay handles all credit risk with full settlement to the merchant, and that SPayLater is Shariah certified by Amanie Advisors.

## Onboarding Steps

1. Register interest: ShopeePay Malaysia partner page `https://shopeepay.com.my/en/merchant` ("Join Us Now"), or the plugin merchant interest form linked from `https://product.shopeepay.com/products/e-commerce-plug-in/overview`. ShopeePay says it replies within 7 business days (plugins page) or reviews documents within 7 working days (MY merchant page).
2. Prepare supporting documents by business type. The exact document list is in ShopeePay's application guide and is not published on the developer docs (checked 2026-10-21).
3. Sign the NDA and commercial agreement with the ShopeePay integration team. Ask for SPayLater to be included explicitly.
4. Receive integration-test credentials: Client ID and Secret Key, plus your `merchant_ext_id` and `store_ext_id`.
5. Give ShopeePay your callback URL(s) during onboarding (v3 notify URL, Gateway callback URL, account-link notify URL if used). There is no self-serve dashboard field for this in the public docs.
6. Build and test on staging hosts, then get production credentials and switch hosts.
7. For the promotional widget, ask the account manager for the SPL Access Key and, for apps, the Android `.aar` or iOS XCFrameworks.

## Credentials And Env Vars

| Credential | From | Suggested env var | Notes |
| --- | --- | --- | --- |
| Client ID | ShopeePay integration team | `SHOPEEPAY_CLIENT_ID` | Sent in `X-Airpay-ClientId`. |
| Secret Key | ShopeePay integration team | `SHOPEEPAY_SECRET_KEY` | HMAC key. Server only. Separate per environment. |
| Merchant external ID | Agreed at onboarding | `SHOPEEPAY_MERCHANT_EXT_ID` | `merchant_ext_id` in bodies. |
| Store external ID | Agreed at onboarding | `SHOPEEPAY_STORE_EXT_ID` | `store_ext_id` in bodies. |
| Environment | Your config | `SHOPEEPAY_ENV` (`staging` or `production`) | Selects host pair. |
| Wallet API base | Derived | `SHOPEEPAY_API_BASE` | `https://api.uat.wallet.airpay.com.my` or `https://api.wallet.airpay.com.my`. |
| Gateway API base | Derived | `SHOPEEPAY_GW_BASE` | `https://api.gw.uat.airpay.com.my` or `https://api.gw.airpay.com.my`. |
| SPL widget Access Key | Account manager | `SPL_WIDGET_ACCESS_KEY` | Goes to the browser by design. Same for test and live. |
| Shopify Access Key | ShopeePay | (entered in app) | 32-digit merchant access key for the Shopify ShopeePay Payment App. |

Never commit keys. Never put the Secret Key in a frontend or mobile app.

## Sandbox

- Staging hosts are listed in `spaylater-api.md`. Credentials come only from ShopeePay after the agreement.
- No public test accounts, test phone numbers, or SPayLater approval simulators are documented (checked 2026-10-21). Ask the integration team for staging test users with SPayLater credit.
- The widget has its own non-production loader URL and `SANDBOX`/`UAT` SDK environments.

## Plugins

- Shopify: ShopeePay Payment App per country (Malaysia `https://apps.shopify.com/shopeepay-malaysia`) plus the separate SPayLater On-Site Messaging app. Disable SPayLater in the app if your agreement does not include it. If another gateway already offers ShopeePay/SPayLater on your store, ShopeePay suggests disabling it there and using the official app.
- WooCommerce: "Checkout with ShopeePay" by shopeepay (`https://wordpress.org/plugins/checkout-with-shopeepay/`, v1.0.0 checked). Settings: Environment, Country/Market, Client ID, Client Secret, Merchant ID, Store ID, Debug Logging. The settings page shows the Callback URL (`/?wc-api=shopeepay_callback`); ShopeePay must register it or no notifications arrive. The plugin uses `/v3/merchant-host/order/create` without `preferred_payment_method_type`, so the customer chooses SPayLater in the ShopeePay flow.

## Settlement And Payout

- Settlement report per chosen frequency, listing each payment and refund with fees; can be emailed to set recipients.
- Frequencies: Daily (T+1), Weekly (every Monday, for the week since previous Monday), Monthly (first day of month, for previous month).
- Settled in the region's default currency (MYR).
- Funds may take up to two days after settlement to reach the bank account.
- `field1` to `field3` in `additional_info` appear in the ShopeePay clearing order report for reconciliation.

## Pricing

Merchant rates for SPayLater through ShopeePay are agreed in the commercial agreement and are not published on the developer docs. The ShopeePay MY merchant page mentions a "Standard MDR Rate" without numbers. For Shopee marketplace seller fees see `https://seller.shopee.com.my/edu/article/16542`. Do not quote numbers from memory; they change.

## Regulatory Note

SPayLater copy touches consumer credit. Follow the widget compliance rules in `spaylater-widget.md` and confirm market wording with ShopeePay and your own legal team before launch.
