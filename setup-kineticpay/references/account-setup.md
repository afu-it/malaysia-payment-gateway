# KineticPay Account Setup

Use this reference when explaining KineticPay registration, dashboard setup, credentials, sandbox mode, payment methods, settlement, pricing, and plugin setup.

## Source URLs

- `https://kineticpay.my/`
- `https://docs.kineticpay.my/`
- `https://docs.kineticpay.my/quickstart`
- `https://docs.kineticpay.my/authentication`
- `https://docs.kineticpay.my/sandbox`
- `https://docs.kineticpay.my/companies`
- `https://manage.kineticpay.my/register`
- `https://wordpress.org/plugins/kineticpay-for-woocommerce/`

## Product Positioning

KineticPay is a Malaysian payment gateway and collection-form platform. The homepage presents FPX for all Malaysian banks, a Credit Card + FPX plan, next working day settlement, collection forms/links, buyer ban lists (anti-scammer), and WooCommerce and WHMCS plugins with tutorials. E-wallet, international cards and BNPL are marked "Coming Soon" (checked 2026-10-21). The footer says "Powered by PayNet, FPX Secured".

## Account Setup Checklist

1. Register at `https://manage.kineticpay.my/register`.
2. Add your company in the dashboard. Companies are reviewed and approved by the KineticPay team (`/companies` docs). The company record holds name, SSM number, SSM document, address, phone, office phone, so expect to upload an SSM registration document. The full document list for approval is not documented; ask KineticPay.
3. Create a Collection. Fill in Collection name and your website URL; the merchant key is then generated automatically and shown on the Collection page (WooCommerce plugin FAQ).
4. If you use API v2 (banks, companies), generate the API key at Dashboard > API Key (`https://manage.kineticpay.my/api-key`).
5. Use the Collection's sandbox toggle for testing, then turn it off before live.

## Credentials And Env Vars

| Env var (suggested) | Value | Used for |
|---|---|---|
| `KINETICPAY_MERCHANT_KEY` | Collection merchant key | `/payment/create`, `/payment/status` |
| `KINETICPAY_API_KEY` | Dashboard API key | `Authorization: Bearer` on `/api/v2/*` |
| `KINETICPAY_CALLBACK_BASE_URL` | Your public HTTPS base | building `callback_success`, `callback_error`, `callback_status` |

Keep both keys server-side. Reset the API key from the dashboard if it may be compromised (Authentication docs). How to rotate a merchant key is not documented.

Each Collection has its own merchant key. If you run several sites, use one Collection per site so keys and reports stay separate.

## Sandbox

From `https://docs.kineticpay.my/sandbox`:

1. Log in to the dashboard.
2. Open the Collection page.
3. Toggle sandbox mode on.
4. Transactions then run against a simulated sandbox page; no real money moves.
5. Endpoints and behaviour are the same as live.
6. Turn sandbox off when ready for real payments.

Published sandbox test banks, test accounts or test credentials: not documented in official docs (checked 2026-10-21). Because sandbox is a toggle on the same key, add a deploy check or admin reminder that the Collection is in the expected mode.

## Payment Methods

- FPX (B2C banks, plus `MBB0228` Maybank2E corporate in the plugin list). This is the only method exposed in the published bill fields (`bank` code).
- Credit card: a Credit Card + FPX plan is advertised; how card payments are selected in the API is not documented. Ask KineticPay before building card checkout.
- E-wallet, international card, BNPL: "Coming Soon" on the homepage.
- Recurring billing, refunds via API: not documented.

## Settlement

The homepage advertises next working day settlement. Settlement report API is listed in the docs nav ("Settlements") but has no published page. Reconcile from the dashboard until that API is documented.

## Pricing

See the pricing section on `https://kineticpay.my/` (`#pricing`). Do not hard-code rates; they change and the card plan percentage is not published.

## Plugins

Official WordPress.org plugins (author `faizalbahasan`, homepage `https://kineticpay.my/`):

- `kineticpay-for-woocommerce` (3.0, also adds custom checkout fields and a buyer ban list; readme asks you to enable stock management with minimum stock 2)
- `kineticpay-for-givewp`
- `kineticpay-for-gravityforms`
- `kineticpay-for-forminator`

WooCommerce setup (readme): install, activate, put the merchant key in WooCommerce > Settings > Payments > kineticPay.

The homepage also mentions a WHMCS plugin; its source is not published on an official public repo (the GitHub org `https://github.com/kineticpay-my` has no public repositories, checked 2026-10-21). Get it from KineticPay.

## Support

- Email: `support@kineticpay.my` (docs footer).
- Facebook: `https://www.facebook.com/kineticpay` (plugin readme).
