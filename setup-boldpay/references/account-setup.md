# BOLD.Pay Account Setup

Use this reference when explaining BOLD.Pay registration, credentials, portal settings, plugins, sandbox access, pricing and settlement.

## Source URLs

- `https://macrokiosk.com/boldpay-payment-gateway`
- `https://macrokiosk.com/boldpay-payment-channels`
- `https://macrokiosk.com/boldpay-direct-carrier-billing`
- `https://macrokiosk.com/enquiry/woocommerce`
- `https://www.boldpass.com/register`
- `https://www.boldpass.com/login`
- `https://wordpress.org/plugins/bold-pay/` (`readme.txt`, plugin settings in `includes/boldpay.php`)

## Product Positioning

BOLD.Pay is the payment product of MACROKIOSK (BOLD. platform). The product page says merchants connect "via an easy web interface, API and/or plugin" and accept Credit/Debit Cards, Online Banking, E-Wallets, Buy Now Pay Later (BNPL) and Direct Carrier Billing (DCB). It names plugins for WooCommerce, OpenCart and EasyStore. The plugin readme lists auto-debit payment, card instalments, real-time payment status and reports.

The older WordPress.org listing text says the service is only available to businesses that reside in Malaysia.

## Registration

1. Register a BOLD.Pass account at `https://www.boldpass.com/register`.
2. Wait for approval. The plugin readme refers to an "approved BOLD.Pay Username and API Key".
3. For questions, use the enquiry form `https://macrokiosk.com/enquiry/woocommerce`; a business consultant follows up.

Documents needed for KYC: Not documented in official docs (checked 2026-10-21). Expect MACROKIOSK to request them during approval.

## Credentials

| Credential | Where | Env var (suggested) |
|---|---|---|
| Username | The BOLD.Pass sign-up username | `BOLDPAY_USERNAME` |
| API Key | BOLD.Pass > Admin Management > Business Setting > API Key section | `BOLDPAY_API_KEY` |
| Environment | `sandbox` or `live` | `BOLDPAY_ENV` |
| Callback URL | Your browser return URL | `BOLDPAY_CALLBACK_URL` |
| Notification URL | Your server notification URL | `BOLDPAY_NOTIFICATION_URL` |
| Server IP sent as `CustIP` | Fixed outbound IP of your server | `BOLDPAY_SERVER_IP` |
| Currency code | `1` MYR, `3` THB | `BOLDPAY_CURRENCY_CODE` |

There is no separate webhook secret. Plugin 1.6.0 to 1.7.1 contain a commented-out "Secret Key" setting, so a secret may be added later; it is not used today.

## Portal Configuration (from the plugin readme)

1. Log in at `https://www.boldpass.com/login`.
2. Open Admin Management > Business Setting.
3. Scroll to the API Key section.
4. Enter your domain or IP in Domain/IP Restrictions and click Add Domain.
5. Click Save Changes.

Requests from a domain or IP not on this list fail with error 415.

## WooCommerce Plugin Setup

1. Install WooCommerce, then install "BOLD.Pay for WooCommerce" from Plugins (search `boldpay`).
2. WooCommerce > Settings > Payments > BOLD.Pay.
3. Enter User Name and API Key.
4. Callback URL default: `https://<your-domain>/checkout/`.
5. Notification URL default: `https://<your-domain>/?wc-api=boldpay_check_notification`.
6. Select Currency (MYR or THB) and Environment (Sandbox or Live, default Live).
7. Tick Enable, optionally Logging (logs to WooCommerce logs, source `plugin-bold-pay`), then Save Changes.

Plugin requirements from `readme.txt` 1.7.1: WordPress 5.2.1+, WooCommerce 2.6.0+, tested to WordPress 7.1.0 and WooCommerce 11.1.0. Block Checkout is supported from 1.7.0. The plugin makes billing phone required at checkout.

Diagnostic URL: `https://<your-domain>/?wc-api=testfunction` prints plugin name, version and "Online".

## Other Plugins

- OpenCart: "BOLD.Pay for OpenCart" is listed on the OpenCart marketplace (extension 39904); the page was blocked to automated fetch on 2026-10-21, so its source was not read.
- EasyStore: named on the product page; setup steps Not documented publicly.

## Sandbox

The plugin has a Sandbox environment pointing to `uat.pay.etracker.cc`. How to get UAT credentials and test data: Not documented in official docs (checked 2026-10-21). Ask MACROKIOSK.

## Payment Methods

Channels returned in `PymtMethod`: `CC` card, `DD` online banking (FPX), `WA` e-wallet, `BNPL`, `OTC`. Which channels are active depends on the merchant's approval; there is no documented channel-selection parameter.

## Pricing And Settlement

- Pricing: no public rates. See `https://macrokiosk.com/boldpay-payment-gateway` and ask MACROKIOSK sales. Do not quote numbers.
- Settlement schedule and payout: Not documented in official docs (checked 2026-10-21).
- Transaction minimum and maximum: account limits enforced with errors 417 and 418.
