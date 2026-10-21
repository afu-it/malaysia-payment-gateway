# Kiple Account Setup

Use this reference when explaining Kiple merchant registration, credentials, UAT vs live, plugin configuration, payment method activation, settlement and pricing. Checked 2026-10-21.

## Source URLs

- Kiple home and payments page: `https://kiple.com/`, `https://kiple.com/payments/` (Cloudflare blocks plain fetches; open in a browser)
- SME packages and pricing: `https://kiple.com/sme/`
- Merchant registration form: `https://kiplepay.com/kyc/register`
- Legacy gateway service page: `https://kiplepay.com/pages/service/gateway-service`
- Merchant Terms: `https://kiplepay.com/img/document/20190601-Merchant-Terms.pdf`
- Merchant Settlement Process Schedule v2.0: `https://kiplepay.com/img/document/20190528-Merchant-Settlement-Process-Schedule.pdf`
- Merchant Dispute Process Schedule: `https://kiplepay.com/img/document/20190509-Merchant-Dispute-Process-Schedule.pdf`
- Official plugins: `https://github.com/m2c` (organisation "Kiplepay Sdn Bhd")

## Who Kiple Is

Kiplepay Sdn Bhd (formerly Webonline Dot Com Sdn Bhd, brand formerly "Webcash") is a subsidiary of Green Packet Berhad and runs the Kiple e-wallet and payment gateway (legacy service page footer). The gateway offers cards, FPX, e-wallets and DuitNow QR on one hosted page, plus recurring payments and tokenisation per the payments page.

## Registration Checklist

1. Submit the online merchant form at `https://kiplepay.com/kyc/register` (or the current registration link on `https://kiple.com/`). It asks for business type (sole proprietorship, partnership, Sdn Bhd, Bhd, government entity, society), business name, registration number, incorporation date, address, contact number, business website URL, and owner / primary contact details.
2. Kiple reviews the application and sets the settlement cycle in the Merchant Onboarding Form (Settlement Process Schedule).
3. After approval Kiple provides the **Merchant ID** and **Secret Key** ("Configure the merchant settings which is provided from Kiple", WooCommerce readme; "Merchant ID which you should have received in separate email", Magento 1.9 readme). Kiple's technical API document is not published; ask Kiple for it.
4. Ask Kiple for separate **UAT** credentials before coding. UAT host: `uat.kiplepay.com`.
5. For recurring payments, subscribe separately: "contact support@kiplepay.com" (OpenCart readme). You get access to a recurring portal for cancellations.
6. For split orders (marketplace), confirm with Kiple that the master and sub-merchant IDs are provisioned.

Required KYC documents list: Not documented publicly (checked 2026-10-21); the form collects the business details above.

## Credentials And Env Vars

| Credential | Where it comes from | Plugin labels | Suggested env var |
|---|---|---|---|
| Merchant ID | Kiple approval email | "Merchant ID" (WooCommerce), "Merchant Id" (OpenCart), "Merchant Code" (WHMCS, PrestaShop) | `KIPLE_MERCHANT_ID` |
| Secret Key (hash key) | Kiple approval email | "Verify Key" (WooCommerce), "Hash Key" (OpenCart), "Merchant Key" (WHMCS, PrestaShop) | `KIPLE_SECRET_KEY` |
| Environment | your config | "Test Mode" checkbox / Sandbox vs Live | `KIPLE_ENV=uat|production` |
| Return URL | your app | plugin builds it | `KIPLE_RETURN_URL` |
| Callback URL | your app | `dynamic_callback_url` | `KIPLE_CALLBACK_URL` |

Keep the Secret Key server-side. It is the only thing protecting the request and response hashes. Where the key lives in the kiplePay Portal (if it is shown there at all): Not documented.

## Plugin Configuration Notes

- WooCommerce: settings Merchant ID, Verify Key, Test Mode. Callback URL is `https://<your-domain>/?wc-api=WC_webcash_Gateway`. Requires SSL in live mode. Despite the field help text, the Verify Key must be the real Secret Key.
- OpenCart 3: Extensions > Payments > Kiplepay; set Merchant Id, Hash Key, Sandbox or Live, order statuses, and "recurring payment status" if subscribed.
- PrestaShop 1.7: the plugin hard-codes the UAT URLs; edit `webcash.php` line 41 and `controllers/front/paymentreturn.php` line 86 for production (plugin readme).
- WHMCS 7.2: replace the `****` domain placeholders in `webcash.php` and `wccallback.php` before use.
- All plugins: store currency must be MYR.

## Payment Methods

The hosted page shows whichever methods Kiple activated for the merchant: credit/debit cards (local, foreign, UnionPay), FPX B2C and B2B, DuitNow QR, and e-wallets. Selecting a method from your site through an API field is Not documented (the only method field seen is `payment_code=CC` for recurring).

## Settlement

From the Merchant Settlement Process Schedule v2.0:

- Kiple settles per the Settlement Cycle in your Merchant Onboarding Form.
- Settlement Amount = Transaction amount minus MDR minus tax on MDR.
- Download settlement reports and check transaction status in the kiplePay Portal; raise disputes per the Merchant Dispute Process Schedule.

Merchant Terms clause 7.6: receipt of funds is not cleared funds; the merchant is liable for refunds and chargebacks. Refunds have no API; handle them through Kiple.

## Pricing

Do not copy numbers. Link to the official SME packages page: `https://kiple.com/sme/`. Prices and packages change.

## Support Contacts (from official plugins)

- `customercare@kiplepay.com` (WooCommerce header)
- `support@kiplepay.com` (OpenCart readme, recurring)
- Contact page: `https://kiple.com/contactus` (linked from the WooCommerce settings)
