# Qashier Account Setup

Use this reference when explaining Qashier sign-up, credentials, staging vs production, payment methods, plug-ins, Pay-by-Link, settlement, and pricing. Checked 2026-10-21.

## Source URLs

- `https://qashier.com/my/`
- `https://qashier.com/my/qashierpay-online`
- `https://qashier.com/my/pay-by-link/`
- `https://qashier.com/my/api-integration`
- `https://qashier.com/my/ecommerce-integration`
- `https://qashier.com/my/mdr-rates`
- `https://docs.qashier.com/docs/intro`
- `https://docs.qashier.com/docs/guides/checkout-session`

## Product Positioning

Qashier is POS-first (QashierPOS, terminals, Soundbox) with QashierPay Online as its online gateway. QashierPay Online accepts DuitNow QR, credit and debit cards, e-wallets (GrabPay, Touch 'n Go, Boost and more), cross-border payments, and BNPL in one hosted checkout, and syncs with QashierPOS reporting in QashierHQ. In Malaysia, Qashier partners with Bank Simpanan Nasional (BSN), a BNM-registered merchant acquirer.

Integration options listed on the QashierPay Online page:

1. Plug-in: Shopify, Wix, WooCommerce (no coding).
2. Custom: hosted checkout, API, webhooks, with Qashier's team supporting the integration.
3. Pay-by-Link: dynamic payment links from QashierHQ, shared over WhatsApp, Instagram, email or in-store. Single-use; status shown in real time.

Note: the Shopify/WooCommerce page at `/my/ecommerce-integration` is about POS inventory sync, not payments. No official plug-in source or plug-in setup docs were found (WordPress.org search for "qashier" returned no plugins, checked 2026-10-21).

## Registration

- Sign up through the "Get started" form on the QashierPay Online page or the API Integration page ("Get in touch"). There is no self-serve developer sign-up documented.
- Required business documents: Not documented in official docs (checked 2026-10-21). Expect KYC by Qashier and the acquirer.
- The docs intro says: for questions, reach out to your account manager.

## Credentials

Online Checkout API:

| Item | Where | Env var |
|---|---|---|
| API key (`sk_...`, test keys shown as `sk_test_...`) | "Qashier Dashboard" per the API docs; exact menu path not documented | `QASHIER_API_KEY` |
| Webhook secret | Not documented where it is shown | `QASHIER_WEBHOOK_SECRET` |
| API base | Staging or production URL | `QASHIER_API_BASE` |

In-store partner API: partner ID (`QASHIER_PARTNER_ID`), HMAC key (`QASHIER_HMAC_KEY`), plus client ID and store ID, all issued by Qashier, and your server IPs must be whitelisted.

The merchant back office is QashierHQ (`https://hq.qashier.com`); test payments appear under Transactions.

## Staging vs Production

- Staging base: `https://api.staging.qashier.com/v3/payment/qpay`. Production: `https://api.qashier.com/v3/payment/qpay`.
- How staging accounts and keys are issued is not documented; ask the Qashier representative.
- Keep keys, webhook URL, and webhook secret separate per environment.

## Payment Method Activation

Methods are not chosen in the API. Checkout shows what the account has enabled. If a method is missing, it is an account setting; ask the Qashier representative to enable it. Create a test session and open its `url` to see the live set.

## Settlement

Marketing pages say DuitNow and card transactions settle the next business day, and card and GrabPay payouts arrive the next day; Pay-by-Link funds arrive next day (T+1). Settlement for other methods and payout reports: Not documented in official docs (checked 2026-10-21). No settlement API exists.

## Pricing

Link only, do not copy numbers (they change and are indicative): `https://qashier.com/my/mdr-rates` and `https://qashier.com/my/qashierpay-online`. Final rates depend on business profile and volume.

## Production Readiness Checklist

1. Production API key and webhook secret stored in server secrets.
2. Webhook URL registered in production as Qashier instructs.
3. Required methods (DuitNow QR, cards, e-wallets) confirmed enabled by opening a live session URL.
4. Signature verification tested against a real staging event.
5. Success and return pages do not fulfil; webhook plus retrieve does.
