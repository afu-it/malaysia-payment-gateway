# Paydibs Account Setup

Use this reference when explaining Paydibs registration, credentials, staging vs production, payment method activation, settlement, and pricing. Checked 2026-10-21.

## Source URLs

- `https://paydibs.com/`
- `https://paydibs.com/online-payment/`
- `https://paydibs.com/payment-gateway-stack/`
- `https://paydibs.com/contact-us/`
- `https://paydibs.com/paydibs-service-agreement/`
- `https://v3api-docs.paydibs.com/docs/introduction`

## Company

Paydibs Sdn. Bhd. (201001018323 (902029-A)), Kuala Lumpur. The service agreement states it is regulated by Bank Negara Malaysia as a Non-Bank Merchant Acquiring Services provider and maintains PCI-DSS Level 1 certification.

Online products: Payment Gateway Stack (connect an existing site or app) and Paydibs Nexus (storefront plus checkout). In-store: Paydibs NEO, Payment Terminal, Payment App.

## Registration

There is no public self-signup or merchant dashboard documented. The path is a sales enquiry:

1. Submit the enquiry form at `https://paydibs.com/contact-us/` (name, email, phone, business type, product "Payment Gateway Stack"), or contact sales via the phone/WhatsApp number on that page.
2. Paydibs onboards the merchant under the Paydibs Service Agreement. The agreement says Paydibs may request further information or documentation to validate identity.
3. Required documents list: Not documented in official public pages (checked 2026-10-21). Ask Paydibs sales.

## Credentials

From Introduction 1.1: "A test account is needed for merchant integration. Upon account generated, Merchant ID, Merchant Password and payment URL will be provided." From 1.5: "Production URL will be provided via email upon live payment account created."

| Value | Where it comes from | Suggested env var |
| --- | --- | --- |
| Merchant ID (6 chars) | Issued by Paydibs | `PAYDIBS_MERCHANT_ID` |
| Merchant Password (hash secret) | Issued by Paydibs | `PAYDIBS_MERCHANT_PASSWORD` |
| Payment URL host | Issued by Paydibs (production by email) | `PAYDIBS_BASE_URL` |
| Environment flag for the SDK | `stg` or `prod` | `PAYDIBS_ENV` |
| Return URL | Merchant | `PAYDIBS_RETURN_URL` |
| Callback URL | Merchant | `PAYDIBS_CALLBACK_URL` |

Keep the Merchant Password in server-side secrets only. Staging and production have separate IDs, passwords and URLs.

For reference, the official Hosted Session SDK uses `https://stgapi.paydibs.com` (stg) and `https://v3api.paydibs.com` (prod). Still use the URL Paydibs issues.

## Staging (test account)

- Requested from Paydibs; not self-serve.
- Public test cards, test banks or a simulator guide: Not documented in official docs (checked 2026-10-21). Ask Paydibs for test data with the test account.
- Test TLS 1.2 connectivity to production before going live (QUERY and REFUND pages).

## Payment Methods

- Method codes: `OB` online banking (FPX), `CC` credit/debit card, `WA` e-wallet. The introduction also names BNPL.
- Some features depend on the channel the merchant is subscribed to: E-KYC (FPX), Cash on Delivery, recurring, saved cards, TNG AutoDebit, e-mandate/direct debit, in-store DuitNow QR. Confirm which are enabled on the account.
- Activation timelines: Not documented (checked 2026-10-21).

## Settlement And Payouts

- The service agreement defines Settlement and Payout Period, and allows deductions (for example refunds, chargebacks) from settlement proceeds.
- The payout schedule (T+N) is not published on public pages (checked 2026-10-21). It is set in the merchant's agreement schedule.
- Chargebacks and reversals can claw back settled funds; the agreement lists merchant evidence duties (proof of shipment, proof of delivery).

## Pricing

Not published as a price list on paydibs.com (checked 2026-10-21). Pricing is quoted through sales: `https://paydibs.com/contact-us/`. Do not quote numbers.

## Go-Live Checklist

1. Live Merchant ID, Password and production URL received.
2. Production secrets set; staging values removed from production config.
3. Return and callback URLs are HTTPS, publicly reachable, and return HTTP 200 from the callback.
4. TLS 1.2 connectivity to production confirmed.
5. One low-value live payment, QUERY and (next day) refund tested end to end.
