# HitPay Account Setup

Use this reference when explaining HitPay registration, dashboard setup, sandbox setup, Malaysia payment methods, pricing/payout caveats, plugin setup, and operational readiness.

## Source URLs

- `https://hitpayapp.com/my/`
- `https://docs.hitpayapp.com/introduction`
- `https://docs.hitpayapp.com/getting-started/account-setup`
- `https://docs.hitpayapp.com/getting-started/business-verification`
- `https://docs.hitpayapp.com/payments/payments`
- `https://docs.hitpayapp.com/getting-started/availability-by-market`
- `https://docs.hitpayapp.com/apis/guide/sandbox`
- `https://docs.hitpayapp.com/plugins/overview`
- `https://status.hitpayapp.com/`

## Product Positioning

HitPay is an omnichannel payment platform for online payments, in-person payments, payment links, invoicing, recurring billing, POS, static QR, cross-border payments, plugins, APIs, and payouts.

The Malaysia page presents these relevant capabilities:

- Online checkout.
- POS and card terminals.
- Payment links and invoicing.
- Plugin integrations for platforms such as Shopify, WooCommerce, and Wix.
- Recurring billing.
- Developer API.
- Local and cross-border payment methods.

## Account Setup Checklist

For production:

1. Create or log in to a production HitPay account.
2. Complete business profile and business verification.
3. Go to Payments > Payment Methods.
4. Connect and activate required payment methods.
5. Go to Settings > API Keys and generate production API keys.
6. Go to Developers > Webhook Endpoints and register the live webhook URL.
7. Store the production webhook salt securely.
8. Configure any plugin, Payment Link, Drop-In UI default link, or checkout settings needed by the integration.
9. Check status page before launch or incident-sensitive operations.

For sandbox:

1. Sign up for a sandbox account at the sandbox dashboard.
2. Use dummy business details.
3. Use the sandbox dashboard at `https://dashboard.sandbox.hit-pay.com/login`.
4. Enable test payment methods under Settings > Payment Methods.
5. Generate sandbox API keys under Settings > API Keys.
6. Register sandbox webhook endpoints separately from production.
7. Use sandbox request logs under Developers > Request Logs for debugging.

HitPay docs state sandbox and production are separate accounts. No verification or approval is required for sandbox, and dummy business details can be used.

## Malaysia Payment Method Notes

HitPay docs list Malaysia-local methods including:

- Touch 'n Go.
- DuitNow.
- Cards.
- FPX.
- WeChat Pay.
- ShopBack Pay.
- Boost.
- Maybank QR.
- AliPay.
- GrabPay.
- GrabPay PayLater.
- Atome.
- ShopeePay.
- SPayLater.

Availability, activation time, settlement timing, recurring support, and pricing can vary by method and account. Always verify current dashboard state before promising a method is live.

## Sandbox Notes

Sandbox supports test-only transactions and webhook events to test endpoints. HitPay documentation lists sandbox support for cards, PayNow, ShopeePay, GrabPay, GrabPay PayLater, Atome, DOKU QRIS, and mock flows for some saved payment method and recurring billing scenarios.

Sandbox QR testing can be simulated by scanning generated QR codes with a mobile phone camera rather than a payment app.

Sandbox webhook IP:

```text
54.179.156.147
```

Production webhook IPs:

```text
3.1.13.32
52.77.254.34
```

## Plugins And Integrations

HitPay docs describe 16+ ecommerce plugins and integrations across Southeast Asia and beyond. Listed examples include:

- Shopify.
- WooCommerce.
- Wix.
- Magento.
- PrestaShop.
- OpenCart.
- Ecwid.
- Odoo.
- EasyStore.
- Shoplazza.
- Xero.
- QuickBooks.
- Zoho Books.
- Zapier.
- Make.

Plugin integrations can include automatic payment status synchronization, refund handling, and multi-currency checkout. Still verify the plugin-specific guide and test webhook delivery in the target store because store security rules can block callbacks.

## Pricing And Payout Caveats

The Malaysia page advertises zero monthly fees and per-transaction pricing. It also displays example method pricing and payout messaging, but these commercial details are time-sensitive.

Rules for agents:

- Do not hardcode pricing into application logic.
- Link or point to current HitPay pricing for commercial decisions.
- Verify settlement and payout schedule in dashboard under Sales and Reports > Bank Payouts.
- Verify current operational state on `https://status.hitpayapp.com/` before production launch or incident response.

## Status Page

HitPay publishes a status page with current service status and historical uptime for components such as:

- HitPay Services / Payment APIs and Web Dashboard.
- POS Terminal.
- Checkout Page.

Use the status page for operational awareness, not as a payment source of truth. Payment state must come from signed webhooks and authenticated status API calls.

## Required Environment Variables

Use names that match the target app's conventions. Common variables:

```text
HITPAY_ENV=sandbox
HITPAY_API_URL=https://api.sandbox.hit-pay.com
HITPAY_API_KEY=your_sandbox_api_key
HITPAY_WEBHOOK_SALT=your_sandbox_webhook_salt
HITPAY_WEBHOOK_SECRET_VERSION=optional_rotation_marker
HITPAY_REDIRECT_URL=https://example.com/payments/hitpay/return
HITPAY_WEBHOOK_URL=https://example.com/api/webhooks/hitpay
```

For production, change to:

```text
HITPAY_ENV=production
HITPAY_API_URL=https://api.hit-pay.com
```

and use production API key, production webhook salt, production redirect URL, and production webhook URL.
