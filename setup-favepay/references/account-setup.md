# FavePay Account Setup

Use this reference when explaining FavePay merchant registration, the credentials Fave issues, sandbox UAT, environment variables, payment methods, plugins, pricing and settlement caveats.

## Source URLs

- `https://www.favebiz.com/`
- `https://app.favebiz.com/sign-up?ref=favebiz`
- `https://support.favebiz.com/support/home`
- `https://developers.myfave.com/`
- `https://developers.myfave.com/fpo-guide.html#uat`
- `https://developers.myfave.com/fpo-guide.html#support`
- `https://wordpress.org/plugins/favepay-for-woocommerce/`

## Product Positioning

- Fave (Fave Asia Technologies Sdn. Bhd.) runs FaveBiz, a merchant platform to accept payments, track sales and run loyalty (Cashback, eCards) and deals marketing.
- FavePay Omni (FPO) is the partner API to "accept payments online or in-store with Fave": dynamic QR, hosted `web_url` checkout, merchant scan, static QR (planned), lookup, cancel, refund and webhooks.
- Markets in the FPO guide: Malaysia (`my`) and Singapore (`sg`). The WooCommerce plugin readme says it is "Applicable for merchants selling in Malaysia and Singapore".
- The WooCommerce plugin says merchants can "accept major debit and credit cards as well as local e-wallets while enabling loyalty programs through Cashback", plus eCards.
- FavePay Later (BNPL) was removed from the plugin in v1.0.16 (2024-12-17). Transaction objects still carry `fpl_*` fields.

## Account Setup Checklist

1. Sign up as a Fave merchant through FaveBiz (`https://app.favebiz.com/sign-up?ref=favebiz`, or the contact form linked from the plugin settings `https://www.favebiz.com/#contactForm`). This gives access to the FaveBiz merchant dashboard.
2. Ask Fave for API integration. New merchants are guided through onboarding by the Fave team. Existing FavePay merchants contact their business development manager, or email `myfp.integration@myfave.com` (Malaysia) or `sgfp.integration@myfave.com` (Singapore).
3. At the technical kick-off, Fave provisions sandbox access and you give Fave your sandbox webhook URL.
4. Receive from Fave for sandbox: base URL `https://omni.dino.fave.ninja`, sandbox `app_id` and secret key, sandbox `outlet_id` with FavePay enabled, your `omni_reference` prefix, and optionally the FaveBiz sandbox URL `https://omni-staging.favebiz.fave.ninja` with an outlet partner code for Staff Mode.
5. Complete the UAT checklist in sandbox (see below) and share the summary with your Fave contact.
6. After sign-off, Fave mints new production credentials (`app_id`, secret, outlet, prefix) for `https://omni.myfave.com`. Register the production webhook URL. Never reuse sandbox secrets.

Staging credentials and environment access are provisioned by Fave during partner onboarding; there is no self-serve API key page documented. The plugin settings describe the API key and outlet ID as obtainable from the "FavePay dashboard"; the readme says to get them through Fave onboarding or your BD manager.

Some setups use an outlet-specific secret instead of the integration secret; use the key Fave confirms.

## Sandbox UAT

Payments are simulated: create a QR with `test=true` (formats `favepay`, `txt`, `base64`, also `web_url`, `paynow`), open the returned test page URL and click success or failure. No Fave app needed. There are no published test cards.

| Case | Required | What to prove |
| --- | --- | --- |
| A1 | Yes | Valid signed `POST /qr_codes` returns 200 and `code`. |
| A2 | Yes | Wrong `sign` returns 401. |
| A3 | Yes | Fake `app_id` returns 404 `resource_not_found`. |
| A4 | Yes | Outlet not linked to vendor returns 401 (or documented outlet error). |
| B1 | Yes | `test=true` create returns a test page URL. |
| B2 | Yes | Simulated success reaches the paid path. |
| B3 | No | Simulated failure does not mark paid. |
| B4 | No | Duplicate `omni_reference` behaviour noted. |
| B5 | No | Payment visible in FaveBiz sandbox Staff Mode > Transactions. |
| C1 | Yes | Success webhook received, 2XX returned, sign verified, order paid once. |
| C2 | No | Failure or unpaid gives no false paid. |
| D1 | Yes | Get transaction status matches webhook. |
| D2 | Yes | Cancel unpaid QR works. |
| D3 | Yes | Full refund works. |
| D4 | No | Partial refund works. |
| E1 | No | Duplicate webhook keeps order paid once. |
| E2 | No | Runbook can quote `omni_reference` and time for support. |
| E3 | No | Separate production keys and webhook, no staging secrets in production. |

Minimum before production keys: A1 to A4, B1 to B2, C1, D1 to D3.

## Malaysia Payment Method Notes

- Customers pay in the Fave app or on Fave's hosted page. Webhook `payment_type` examples: `Credit Card`, `Debit Card`, `DuitNow`; variants such as `Visa`, `MasterCard`, `Touch N'Go eWallet`.
- `RPP_` transaction IDs are DuitNow. Partial refunds are not supported for DuitNow.
- Wallet-specific QR formats (`grab`, `alipay`, `wcp`, `shopeepay`, `atome`) depend on country and what Fave enables for the account. `paynow` is Singapore.
- FPX is not mentioned in the FPO guide (checked 2026-10-21).
- Customers can pay partly with Fave credits, merchant cashback, eCard credits, promo codes or AABig points; `charged_amount_cents` is the card or wallet part and `merchant_takeback_cents` is what the merchant receives.

## Plugins And Integrations

- FavePay for WooCommerce (official, by Fave Asia), `https://wordpress.org/plugins/favepay-for-woocommerce/`. Settings: API Key, Outlet ID, Sandbox checkbox, Cashback Rate (promo widget), Instalments promo text, promotional messaging toggles, Debug Log. Country is derived from the store currency (MYR, SGD, IDR). Refunds are full only. The plugin sends `WC()->api_request_url('WC_Favepay')` as `callback_url` on every QR request (handled by the `woocommerce_api_wc_favepay` action) and the order-received page as `redirect_url`.
- The FPO guide says Magento, WooCommerce, Jocom and EasyStore integrations use different redirect formats, and platform-specific checkout integrations are separate from the core Omni endpoints; ask Fave if you need them.

## Pricing And Settlement Caveats

- Fave does not publish merchant pricing or settlement terms on its official sites (checked 2026-10-21). Ask Fave sales or your BD manager. Do not quote rates.
- Per-transaction fee fields come back on every transaction: `fave_fees_percentage`, `fave_fees_cents`, `sst_on_total_fees_cents`, `merchant_takeback_cents`. Use these for reconciliation, not as published pricing.
- Settlement or payout timing and any settlement report API: not documented (checked 2026-10-21).
- Refund window is same calendar day by default; some merchants have a different policy. Disputes after that go through Fave.

## Support

Contact your Fave technical contact for credentials, sandbox access or integration issues. Merchant help centre: `https://support.favebiz.com/support/home`.

## Required Environment Variables

Suggested names (not mandated by Fave):

```
FAVEPAY_ENV=sandbox                  # or production
FAVEPAY_BASE_URL=https://omni.dino.fave.ninja   # production: https://omni.myfave.com
FAVEPAY_COUNTRY=my                   # or sg
FAVEPAY_APP_ID=<APP_ID>
FAVEPAY_SECRET_KEY=<FAVE_SECRET_KEY> # signs requests, verifies webhooks, decrypts omni_ref
FAVEPAY_OUTLET_ID=<OUTLET_ID>
FAVEPAY_REFERENCE_PREFIX=<PREFIX>    # assigned by Fave, used in omni_reference
FAVEPAY_REDIRECT_URL=https://shop.example/payment/return
FAVEPAY_CALLBACK_URL=https://shop.example/webhooks/favepay
```

Keep sandbox and production values in separate environments.
