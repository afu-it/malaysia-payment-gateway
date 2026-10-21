# Luminarix Account Setup

Use this reference when explaining Luminarix registration, credentials, sandbox vs live, per-method setup, the WooCommerce plugin, pricing and settlement.

## Source URLs

- Company site: `https://www.lmnrx.com/` (behind a Cloudflare bot check; read it in a browser)
- Pricing section: `https://www.lmnrx.com/#pricing`
- Privacy policy: `https://www.lmnrx.com/agreements/Privacy%20Policy.pdf`
- WooCommerce plugin: `https://wordpress.org/plugins/luminarix-payment-gateway-for-woocommerce/`

## Company

Luminarix Sdn. Bhd., Petaling Jaya, Malaysia. Methods named by Luminarix: FPX, cards, e-wallets (Touch 'n Go, GrabPay, Boost, ShopeePay, MAE) and DuitNow QR (plugin readme).

## Registration

1. Apply at `https://www.lmnrx.com/`. The plugin readme says credentials "are provided by Luminarix when you apply" and that Luminarix "provisions sandbox and production credentials during merchant onboarding".
2. Documents required, approval time and eligibility rules are not documented in official sources (checked 2026-10-21). Ask Luminarix during onboarding.
3. The Terms of Service and Merchant Agreement are available from the Luminarix website and onboarding (readme).

## What To Collect From Onboarding

For each environment (sandbox, live):

- API base URL.

For each payment method you want to offer, per environment:

- Payment System Code (for example `MYR_TNG`).
- Optional Channel Code.
- Merchant ID.
- Sub-Merchant ID (numeric).
- Client Secret.

Also ask for: the full Payment System Code list, sandbox test data, callback retry policy and source IPs, rate limits, and whether your server IP must be allowlisted (the API sits behind Cloudflare and may block unknown servers).

## Environment Variables

Suggested names (one set per environment; add a method suffix when offering several methods):

```text
LUMINARIX_ENV=sandbox            # or live
LUMINARIX_BASE_URL=<BASE_URL>
LUMINARIX_MERCHANT_ID=<MERCHANT_ID>
LUMINARIX_SUB_MERCHANT_ID=<SUB_MERCHANT_ID>
LUMINARIX_CLIENT_SECRET=<CLIENT_SECRET>
LUMINARIX_PAYMENT_SYSTEM_CODE=<PAYMENT_SYSTEM_CODE>
LUMINARIX_CHANNEL_CODE=          # optional
LUMINARIX_CALLBACK_URL=https://your.app/api/luminarix/callback
LUMINARIX_RETURN_URL=https://your.app/pay/luminarix/return
```

Example for two methods: `LUMINARIX_TNG_MERCHANT_ID`, `LUMINARIX_TNG_CLIENT_SECRET`, `LUMINARIX_FPX_MERCHANT_ID`, `LUMINARIX_FPX_CLIENT_SECRET`, and so on. Keep secrets server-side.

## Sandbox vs Live

- Separate base URLs and separate credentials (readme: "Separate sandbox and production URLs and credentials").
- Record the environment on every order; callbacks for an order must be verified with that environment's secret even after you switch (readme FAQ).
- Place and complete a sandbox order before enabling live (readme installation step 8).
- Sandbox test cards, banks or wallets: not documented in official sources (checked 2026-10-21).

## WooCommerce Plugin Setup

1. Install "Luminarix Payment Gateway for WooCommerce" (WordPress 6.4+, PHP 7.4+, WooCommerce 7.8+).
2. WooCommerce > Settings > Payments > Luminarix Payment.
3. Choose Test / Sandbox, enter the sandbox and live API base URLs.
4. Add a payment method card per method: checkout name, description, Payment System Code, optional Channel Code, sandbox and live Merchant ID, Sub-Merchant ID, Client Secret. Drag to order them.
5. Shared settings: API order prefix (default `WC-`, keep stable), allowed currency (default `MYR`), send payer details (default on), default phone country code (default `60`), debug log, scheduled polling (default on, 5 minutes), max polling age (72 h), orders per poll (10), stuck-order alert hours (6), alert email, API timeout (30 s), webhook timestamp tolerance (600 s).
6. Save and click Test API Connection for each method.
7. Callback URL shown in settings: `https://<site>/wc-api/luminarix_callback/`. Return URL: `https://<site>/wc-api/luminarix_return/`. The plugin sends both automatically.
8. Logs: WooCommerce > Status > Logs, source `luminarix-wc`. Sync health page: WooCommerce > Luminarix Sync.
9. Supports HPOS and Cart/Checkout Blocks.

The marketing site names Shopify, Magento, OpenCart and PrestaShop plugins; none was found in a public official source (checked 2026-10-21).

## Pricing

See the official pricing section: `https://www.lmnrx.com/#pricing`. Rates are quoted per merchant; do not hard-code fees.

## Settlement

Settlement timing is stated on the Luminarix site; check `https://www.lmnrx.com/` and your merchant agreement. Settlement or payout APIs and reports are not documented in official sources (checked 2026-10-21).

## Licensing Check

The Luminarix name was not found on Bank Negara Malaysia merchant-acquirer or e-money lists (MPG research note, 2026-10-21). Merchants should ask Luminarix which licensed acquirer or partner processes their funds.
