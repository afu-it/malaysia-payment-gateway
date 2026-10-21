# Herepay Account Setup

Use this reference when explaining Herepay registration, credentials, UAT setup, callback and redirect URLs, payment methods, pricing, and settlement. Checked 2026-10-21.

## Source URLs

- `https://www.herepay.org/`
- `https://www.herepay.org/soalan-lazim/` (FAQ, Malay)
- `https://www.herepay.org/pusat-bantuan/` (help centre, Malay)
- `https://www.herepay.org/harga/` (pricing)
- `https://herepay.readme.io/reference/getting-started-1`
- `https://herepay.readme.io/reference/how-to-get-your-api-key`
- `https://herepay.readme.io/reference/callback-and-redirect-urls`
- `https://herepay.readme.io/reference/introduction`
- `https://herepay.readme.io/reference/settlement-fee`
- `https://github.com/Herepay/herepay-for-woocommerce` (README credential and webhook steps)

## Product Positioning

Herepay is a Malaysian collection platform and FPX payment gateway aimed at schools, PIBG, mosques, NGOs, communities, and SSM-registered businesses. It offers payment forms, payment links, a RESTful API, and WordPress/WooCommerce plugins. Its FAQ states it is SSM-registered and a Maybank payment gateway agent.

## Registration

1. Register at `https://app.herepay.org/register` (login: `https://app.herepay.org/login`).
2. Verify your email (Getting Started page).
3. Complete account verification. The FAQ says verification takes under 1 hour.

Requirements stated on the official site:

- An **organisation bank account** is required. Individual (personal) accounts cannot register (FAQ).
- Herepay Biz (business route) requires SSM business registration. PIBG, mosques, NGOs, associations, and cooperatives use the organisation route, which needs different documents (help centre). The exact document list is not published; ask Herepay support.

## Credentials

You need three values per environment:

| Value | Where | Env var (SDK/plugin naming) |
| --- | --- | --- |
| Secret Key | Provided during onboarding; dashboard API settings | `HEREPAY_SECRET_KEY` |
| API Key (sent as `XApiKey`) | Dashboard > Settings > Personal > Regenerate API Token | `HEREPAY_API_KEY` |
| Private Key (checksum HMAC key) | Dashboard API settings (plugin README) | `HEREPAY_PRIVATE_KEY` |

Plus `HEREPAY_SANDBOX=true|false` to pick `https://uat.herepay.org` or `https://app.herepay.org` (SDK config naming).

Notes:

- The docs say Secret Key and XApiKey are "provided during the process of onboarding". The "Obtain API Key" page shows only the API token under Settings > Personal. Where to find the Private Key in the dashboard is not on readme.io; the WooCommerce README says "API Settings or Developer Tools". Confirm with support if not visible.
- Regenerating the API token invalidates the old one; update deployments together.
- UAT keys only work on UAT and production keys only on production.

## UAT vs Production

- UAT: `https://uat.herepay.org`, use UAT keys and the test banks in `herepay-api.md`.
- Production: `https://app.herepay.org`, production keys, real banks.
- Data and keys are not shared between them. Register callback and redirect URLs in each environment separately.

## Callback and Redirect URLs

The Managing URLs page says you must provide a callback URL (server-side endpoint for payment updates) and a redirect URL (customer-facing page after payment) during UAT or production setup, in the dashboard. The WooCommerce README says to set the webhook URL under the dashboard's "Webhook Settings or API Configuration". The direct API samples can also pass `redirect_url` in the initiate payload, and the payment link sample can pass `redirect_url` and `callback_url` per link.

The callback URL must be public HTTPS. For local testing, the official samples suggest ngrok.

## Payment Methods

Via API (Payment Channels): `Online Banking` (FPX B2C) and `Corporate Banking` (FPX B2B). The site also lists FPX credit card (Visa/Mastercard, B2C). Activation steps for cards or e-wallets are not documented; ask support.

## Pricing

Do not copy fee numbers; they change and differ between the docs and the site. Link to `https://www.herepay.org/harga/`. The Payment Channels API also returns a `charges` object per payment method for your account.

The Settlement page describes an "absorbed fees" model: fees are deducted from the merchant's settlement and the customer pays the displayed amount.

## Settlement

- Settlement page: typically 1 to 3 business days, depending on your agreement.
- FAQ: 1 to 3 working days to the organisation account.
- Payment gateway page: settlement processed daily, next working day transfer.

These sources differ; quote the range and tell the user to confirm their own agreement.

## Support

- WhatsApp: `+60 11-5140 4297` (Getting Started page, sample repo README).
- Contact page: `https://www.herepay.org/hubungi/`.
