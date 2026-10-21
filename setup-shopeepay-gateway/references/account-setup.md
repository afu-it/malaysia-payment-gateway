# ShopeePay Account Setup (Malaysia)

Use this when explaining how to become a direct ShopeePay merchant, which credentials exist, how sandbox works, plugin setup, and what Malaysia supports. Checked 2026-10-21.

## Source URLs

- `https://product.shopeepay.com/integration/get-started`
- `https://product.shopeepay.com/products/functionalities`
- `https://product.shopeepay.com/products/e-commerce-plug-in/overview`
- `https://product.shopeepay.com/products/woocommerce-plug-in/setup-guideline`
- `https://product.shopeepay.com/products/shopify-plug-in/add-shopeepay-at-checkout`
- `https://product.shopeepay.com/products/brand-guidelines`
- `https://shopeepay.com.my/en/merchant/be-a-partner`
- `https://shopeepay.com.my/en/service-status`
- `https://wordpress.org/plugins/checkout-with-shopeepay/`

## Who Operates It

ShopeePay Malaysia Sdn Bhd (Company No. 201601007371 (1178299-D)), per the footer of `shopeepay.com.my`. Developer docs live at `product.shopeepay.com` and cover ID, MY, PH, SG, TH, VN.

## Onboarding Steps

From Get Started:

1. Contact the ShopeePay integration team; sign the NDA and the commercial agreement.
2. Choose the integration: In-Person (CPM, MPM) or Online (Checkout with ShopeePay, Payment Gateway, Link & Pay, Subscription, Auth & Capture).
3. Follow the ShopeePay/SPayLater branding guideline in the payment flow.
4. Receive onboarding credentials for integration testing.
5. Build against the API rules and signature spec; test; go live.

Entry points for Malaysian merchants:

- `https://shopeepay.com.my/en/merchant/be-a-partner`: "Register as a ShopeePay and SPayLater Partner". Steps shown: prepare supporting documents per the provided guide, submit the application with documents for your business type, review by the ShopeePay team within 7 working days, account activation email on approval. The document list itself is not published on the page (checked 2026-10-21).
- E-commerce plugin overview: merchant interest form `https://forms.gle/PuYieuQqPfj8bej58`; ShopeePay says it replies within 7 business days.

## Credentials

| Credential | Where it comes from | Use |
| --- | --- | --- |
| Client ID | Issued by ShopeePay after the agreement | `X-Airpay-ClientId` header |
| Secret Key (plugin calls it Client Secret) | Issued by ShopeePay | HMAC-SHA256 signing and callback verification |
| `merchant_ext_id` (Merchant ID) | Issued/agreed at onboarding | Every request body |
| `store_ext_id` (Store ID) | Issued/agreed at onboarding | Every request body |
| Environment confirmation | ShopeePay tells you whether credentials are Sandbox/UAT or Production | Pick host |
| Shopify access key (32 digits) | Provided to ShopeePay merchants | Activating the Shopify ShopeePay Payment App |

There is no self-serve dashboard for API keys documented in the developer docs. Never hard-code the secret key in a frontend; all API calls must come from the merchant server.

Suggested env vars:

```text
SHOPEEPAY_ENV=uat|production
SHOPEEPAY_CLIENT_ID=<CLIENT_ID>
SHOPEEPAY_SECRET_KEY=<SECRET_KEY>
SHOPEEPAY_MERCHANT_EXT_ID=<MERCHANT_EXT_ID>
SHOPEEPAY_STORE_EXT_ID=<STORE_EXT_ID>
SHOPEEPAY_API_BASE=https://api.uat.wallet.airpay.com.my     # production: https://api.wallet.airpay.com.my
SHOPEEPAY_GW_BASE=https://api.gw.uat.airpay.com.my          # hosted gateway only; production: https://api.gw.airpay.com.my
```

## Callback URL

The Notify Transaction Status (and account link) callback URL is provided to ShopeePay during onboarding; it is not a request field. The WooCommerce guide says to contact the ShopeePay Integration Team to register the plugin's callback URL (`/?wc-api=shopeepay_callback`), otherwise no payment notification arrives. Register separate URLs for UAT and production.

## Sandbox (UAT)

- Hosts: `api.uat.wallet.airpay.com.my` (wallet APIs) and `api.gw.uat.airpay.com.my` (hosted gateway).
- Sandbox credentials are assigned by ShopeePay; no public test wallet, test card, or test phone number is published (checked 2026-10-21).
- UAT checkout and linking pages appear under `uat`/`test` ShopeePay domains in the doc samples; do not hard-code them, always use the returned URL.

## Malaysia Availability

From Product Functionalities (checked 2026-10-21):

| Method | Malaysia |
| --- | --- |
| ShopeePay Wallet | Live |
| SPayLater | Live |
| Linked bank | Coming soon |
| Credit / debit card | Coming soon |

Hosted ShopeePay Payment Gateway `allowed_payment_method` also lists `online_banking` (MY only) and `duitnow_qr` (MY national QR) and `card`; availability per merchant depends on the agreement.

Limits relevant to Malaysia: Checkout and MPM order/QR validity default 1200 s, max 3600 s; CPM default 120 s; hosted gateway max 86400 s; Direct Payment max 1800 s; auth hold max 14 days; refund window 365 days; refunds blocked 12am to 5am local time by default.

## Plugins

- WooCommerce: "Checkout with ShopeePay" by shopeepay on the WordPress plugin directory (`https://wordpress.org/plugins/checkout-with-shopeepay/`, v1.0.0, supports classic and Blocks checkout). Settings: Enable, Title, Description, Environment (Production or Sandbox), Country/Market (Malaysia (MYR)), Client ID, Client Secret, Merchant ID, Store ID, Debug Logging; Test Connection button; displays the callback URL to register. It uses Checkout with ShopeePay (`/v3/merchant-host/order/create`), verifies callbacks with `X-Airpay-Req-H`, requeries on return, supports full/partial refunds, and invalidates the order if the WooCommerce order is cancelled. Store currency must match the market.
- Shopify: ShopeePay Payment App for Malaysia `https://apps.shopify.com/shopeepay-malaysia`; needs the 32-digit merchant access key; disable Test Mode for live; toggle off channels not in your agreement (for example SPayLater). Store currency and billing address must match the ShopeePay region. Shopify also has an SPayLater On-Site Messaging App.
- If another gateway already offers ShopeePay/SPayLater in Shopify, ShopeePay suggests disabling those options there and using the official app.

## Branding

Use the ShopeePay/SPayLater logos and acceptance marks per `https://product.shopeepay.com/products/brand-guidelines` (minimum sizes, clear space; standalone "Pay" or "SPayLater" logotype is not allowed). Online branding PDF: `https://product.shopeepay.com/static/files/ShopeePay_SPayLater_Online_Branding_Guide.pdf`.

## Settlement, Reconciliation, Pricing

- Reconciliation is T+1; the docs allow accepting a customer's in-app proof of success and settling discrepancies at reconciliation.
- `field1`-`field3` in `additional_info` appear in the ShopeePay Clearing Order Report.
- Hosted gateway status `Settled` means the amount has been escrowed to the merchant.
- Payout schedule, MDR, setup and monthly fees are not published in the developer docs. The partner page mentions a "Standard MDR Rate" without numbers. Ask ShopeePay; link only to `https://shopeepay.com.my/en/merchant/be-a-partner`.
- Service status page: `https://shopeepay.com.my/en/service-status`. Scheduled maintenance returns errcode 1908.
