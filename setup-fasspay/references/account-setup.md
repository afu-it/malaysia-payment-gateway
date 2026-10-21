# Fasspay Account Setup

Use this reference when explaining Fasspay registration, credentials, UAT vs production, payment methods, settlement, pricing, and the WooCommerce plugin setup. Checked 2026-10-21.

## Source URLs

- `https://fasspay.com`
- `https://fasspay.com/productsandservices`
- `https://fasspay.com/support/contact-us`
- `https://fasspay.com/support/pricing`
- `https://fasspay.com/support/merchant-guide`
- `https://fasspay.com/regulated-partners`
- `https://wordpress.org/plugins/fasspay-for-woocommerce/`
- `https://developers.softspace.com.my/` and `https://developers.softspace.com.my/support/contact-us`

## Who Fasspay Is

- Operator: Fass Payment Solutions Sdn. Bhd. (Company Registration No. 201201032773 (1017261-P)), part of the Soft Space group.
- Developer portal owner: Soft Space Sdn Bhd (Company Registration No. 201201002345 (975870-D)).
- Products: payments (mPOS, Fasstap tap on phone, terminals, online "Payment Gateway"), e-wallet, money lending.
- The Products and Services page lists "Payment Gateway: Accept payments on your online store with our secure payment gateway" and says to contact Fasspay to go online.

## Registration

There is no public self-signup for the online gateway. Steps:

1. Contact Fasspay: `https://fasspay.com/support/contact-us`, hotline +603-7494 1222, office Unit 15-15, 2A, Q Sentral, Jalan Stesen Sentral 2, KL Sentral, 50470 Kuala Lumpur. Hours Monday to Friday 9:00am to 6:00pm, Saturday 9:00am to 1:00pm.
2. Choose a business tier on the site (Corporation, Enterprise, SME, Start-Up) or complete the tier questionnaire.
3. Submit documents as Fasspay requests. The required document list for online merchants: Not documented publicly (checked 2026-10-21). The pricing page says card MDR is set by the bank after full documentation and processing may take about 3 to 4 weeks (stated for mPOS).
4. Receive the production Merchant ID and production Secret Key from Fasspay. The plugin FAQ: "These are provided by Fasspay when you register your merchant account."

For in-person SDKs, contact Soft Space (`hello@softspace.com.my`) for the PGP key exchange, SDK package, Maven credentials, developer ID, and tokens. See `fasspay-in-person.md`.

## Credentials And Environment Variables

| Item | Where it comes from | Suggested env var |
|---|---|---|
| Merchant ID (`mid`) | Fasspay onboarding (production); public UAT MID in plugin | `FASSPAY_MID` |
| Secret Key (shared secret for HMAC) | Fasspay onboarding (production); UAT secret in plugin source | `FASSPAY_SECRET_KEY` |
| API base URL | `https://wallet-uat.fasspay.com` or `https://ecom.fasspay.com` | `FASSPAY_BASE_URL` |
| Sandbox flag | `true` for UAT, `false` for production | `FASSPAY_SANDBOX` |
| Callback URL | Your public HTTPS endpoint sent as `backendURL` | `FASSPAY_CALLBACK_URL` |
| Return URL | Your page sent as `responsePageURL` | `FASSPAY_RETURN_URL` |

A merchant dashboard for API keys: Not documented publicly. The developer portal has a `/login`, but what it exposes is not documented.

## UAT vs Production

- UAT uses shared public credentials embedded in the official plugin (`includes/class-fasspay-gateway.php`, constants `SANDBOX_MERCHANT_ID` and `SANDBOX_SECRET`). Use them only for testing.

  Shared public UAT credentials (sandbox only, published in the official plugin source, checked 2026-10-21):

  | Setting | UAT value |
  | --- | --- |
  | `FASSPAY_MID` | `800010000000179` |
  | `FASSPAY_SECRET_KEY` | `0EvpYKg0I9iS80ef3WhMHqtLUBigyZxVhXtCKYqYCVs=` |

  These are the same for every developer. Never use them in production, and expect Fasspay to rotate them without notice; if signing fails in UAT, re-check the latest plugin release.
- Production uses your own MID and secret, and `sandboxMode: false`.
- Hosted page branding (logo up to 200 KB, theme colour) applies in production only.
- Test cards, FPX test banks, and wallet simulators: Not documented. Ask Fasspay.

## Payment Methods

Per the plugin and Fasspay pages: Visa, Mastercard, JCB, MyDebit, FPX online banking, DuitNow QR, e-wallets (online); UnionPay and NFC tap-and-pay for in-person. Which methods are active on your MID is set by Fasspay. The plugin checkout description mentions cards, FPX and DuitNow QR. Method selection happens on the Fasspay hosted page.

## WooCommerce Plugin Setup

1. Install "Fasspay for WooCommerce" (requires WordPress 6.6+, PHP 7.4+, WooCommerce 5.0+; HPOS compatible).
2. WooCommerce > Settings > Payments > Fasspay for WooCommerce > Manage.
3. Toggle Sandbox Mode (UAT). For live, untick it and enter Production Merchant ID and Production Secret Key.
4. Optional: payment logos, brand logo, theme colour.
5. WooCommerce > Fasspay > Test Connection.
6. The callback URL is `https://<your-site>/wc-api/fasspay_webhook`; it must be reachable from the internet.
7. Store currency must be MYR or the gateway is hidden.

## Settlement And Payout

- Online gateway settlement timing: Not documented publicly (checked 2026-10-21).
- For mPOS, the pricing page states revenues minus MDR go to a Hong Leong Bank account the next business day; non-HLB accounts receive IBG transfer about two days later. Confirm whether this applies to the online gateway.

## Pricing

Do not quote numbers; they change and depend on the merchant profile. Link to:

- `https://fasspay.com/productsandservices`
- `https://fasspay.com/support/pricing` (mPOS device and MDR notes)
- Contact Fasspay for a quote.

## Merchant Guides (in-person devices)

`https://fasspay.com/support/merchant-guide` links PDFs for Fasstap (v2.0, July 2022), FLite (v1.0), K9 (v2.0, July 2022), and Lighthouse. These cover device use, not the online API.
