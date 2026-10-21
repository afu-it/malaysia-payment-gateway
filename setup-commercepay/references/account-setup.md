# CommercePay Account Setup

Use this reference when explaining CommercePay registration, credentials, staging vs production, back office, enablement of recurring/payouts, plugins, settlement and pricing. Checked 2026-10-21.

## Source URLs

- `https://www.commercepay.asia`
- `https://www.commercepay.asia/register`
- `https://www.commercepay.asia/get-started`
- `https://www.commercepay.asia/faq`
- `https://www.commercepay.asia/developer`
- `https://docs.commercepay.asia/get-started`
- `https://docs.commercepay.asia/api-credentials`
- `https://docs.commercepay.asia/service-environments`
- `https://docs.commercepay.asia/recurring-payments`
- `https://docs.commercepay.asia/payout-integration`

## Company

CommercePay is operated by Commerce DotAsia Payments Sdn Bhd (1303036-W), a wholly owned subsidiary of Commerce.Asia, licensed by Bank Negara Malaysia to provide merchant acquiring services for online payments (site footer and FAQ).

## Register As A Merchant

1. Fill the New Merchant Application at `https://www.commercepay.asia/register` (3 steps: person in charge name and contact number, business email for setup and notifications, customer support email and number, then business details).
2. A dedicated account manager contacts you within 1 to 2 days to sign the application form.
3. Approval takes 3 to 5 business days (register page). The site says go-live can be as soon as 3 business days.
4. Credentials are then issued by the Business Development Team. Contact: `payments@commerce.asia` or your account manager.

The exact document list for KYC (SSM, bank statement, ID, and so on) is not published; the account manager collects it during the signing step.

## Credentials

Four credentials per environment (staging and production are separate):

| Credential | Purpose | Suggested env var |
| --- | --- | --- |
| Merchant Id | `Abp-TenantId` header, `tenantId` body fields | `COMMERCEPAY_MERCHANT_ID` |
| Username | Authenticate `userNameOrEmailAddress` | `COMMERCEPAY_USERNAME` |
| Password | Authenticate `password` | `COMMERCEPAY_PASSWORD` |
| Secret Key | Sign requests, verify callbacks | `COMMERCEPAY_SECRET_KEY` |

Also configure:

- `COMMERCEPAY_BASE_URL`: `https://staging-payments.commerce.asia/` or `https://payments.commerce.asia/`
- `COMMERCEPAY_CALLBACK_URL`: your public HTTPS callback endpoint (the exact string is part of the callback signature)
- `COMMERCEPAY_RETURN_URL`: browser return page
- Per-environment channel ID map (IDs differ between staging and production)

Store Password and Secret Key in server-side env vars or a vault. Never ship them to the browser or mobile app. Credentials are delivered by CommercePay, not generated in a self-serve dashboard; there is no documented key-rotation flow, so ask the account manager.

## Back Office

- Production merchant back office: `https://backoffice-payments.commerce.asia/` ("Merchant Login" on the site).
- Staging back office host (from the Signature Validator link): `https://staging-backoffice-payments.commerce.asia/`.
- Signature Validator Tool: `https://staging-backoffice-payments.commerce.asia/app/paymentgateway/signaturevalidator`.
- The back office shows transactions, downloadable reports, the payout wallet balance and history, payment links, payment forms, and Recurring Payment Forms.

## Staging vs Production

| Item | Staging | Production |
| --- | --- | --- |
| API base | `https://staging-payments.commerce.asia/` | `https://payments.commerce.asia/` |
| Credentials | Staging set | Production set |
| Payout channelId | `15` | `9` |
| Channel IDs | See codes file (staging list) | See codes file (production list) |
| Test banks | FPX `TEST0021`-`TEST0023`, DuitNow simulators | Real banks |
| Payout test accounts | Published (codes file) | Real accounts |

Staging access is requested from CommercePay; there is no public self-signup sandbox.

## Feature Enablement

- **Payment channels**: whatever `GetTenantChannelsByCountry` returns for your Merchant Id is what is enabled. Inactive (`status: 0`) means maintenance or onboarding pending.
- **Recurring Payments**: must be approved by Business Development, who also give you the recurring-enabled channel(s). Credit Card and TNG eWallet only. Direct Integration only.
- **Payouts**: need a payout wallet created and topped up; arrange with `payments@commerce.asia`.
- **Platform charges** (`platformCharge`): authorized partners only.
- **Instalment Payment Plan (IPP) and BInfinite loyalty**: marketed on the site (`/credit-card-ipp`, `/binfinite-loyalty`); API fields for loyalty exist (`loyaltyPaymentInfoDto`, loyalty fields on Query). Ask Business Development for enablement.

## Payment Methods (site and channel list)

FPX B2C and B2B, DuitNow Online Banking B2C and B2B, DuitNow QR, Visa/Mastercard debit and credit cards, card recurring, Apple Pay, Google Pay, TNG eWallet, GrabPay, ShopeePay, Boost, Alipay, Atome, KK Mart cash over counter (staging list). Confirm per account with Get Channel List.

## Plugins

Listed on `https://www.commercepay.asia/developer`:

- WooCommerce (and a Sequential Order Numbers variant)
- OpenCart 3.0.3.8 and 2.1.0.1
- NopCommerce (nopcommerce.com marketplace)
- WHMCS
- SiteGiant (setup guide on SiteGiant support site)

Plugins need the same Merchant Id and Secret Key. Recurring API integration is not supported through plugins.

## Settlement

The site says earnings settle within 1 to 2 business days (features section); the homepage hero says 1 to 3 business days. Settlement goes to your registered business bank account. Per-transaction `settlementStatus` is visible in Query Payment (`0` Unsettled to `4` CompleteProcessing). A settlement report API is not documented.

## Pricing

Do not quote numbers from this skill. CommercePay states no setup or annual maintenance fee and per-transaction pricing (fixed or percentage), quoted on request. See `https://www.commercepay.asia/faq` and contact `payments@commerce.asia`. IPP pricing: `https://www.commercepay.asia/credit-card-ipp`.

## Support

- Business Development / onboarding: `payments@commerce.asia`
- Customer support: `capsupport@commerce.asia`
- Contact page: `https://www.commercepay.asia/contact-us`
