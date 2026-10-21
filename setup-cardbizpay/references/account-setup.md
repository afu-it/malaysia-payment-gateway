# CardBizPay Account Setup

Use this reference when explaining CardBizPay registration, credentials, staging access, payment methods, settlement, and pricing. Checked 2026-10-21.

## Source URLs

- `https://cardbiz.com.my/cardbiz-payment-services/cardbizpay/`
- `https://cardbizpay.my/`
- `https://cardbizpay.my/guide/how-do-i-register/`
- `https://cardbizpay.my/guide/required-documents-to-submit/`
- `https://cardbizpay.my/guide/1-1-pre-requisite/`
- `https://cardbizpay.my/guide/1-1-pre-requisite-of-merchant-integration/`
- `https://cardbizpay.my/guide/payment-refunds/`
- `https://cardbizpay.my/guide/settlement/`
- `https://cardbizpay.my/guide/how-to-update-the-merchants-bank-account-details/`
- `https://cardbizpay.my/guide/can-i-change-the-registered-email-address/`
- `https://cardbizpay.my/guide/dashboard-overview/`, `/guide/transactions/`, `/guide/reports/`
- `https://cardbizpay.my/contact/`
- `https://cardbizpay.my/pricing/`
- `https://cardbiz.com.my/cardbiz-payment-services/cardbizpay-rates/`

## Product Positioning

CardBizPay is the online payment gateway of the Cardbiz group (Cardbiz Payment Services Sdn Bhd). It offers a hosted payment page (Merchant API), card tokenization and pre-auth (Open Payment API), Payment Links (one-time and multi-use, bulk import), eMOTO (mail/telephone order card payments), and a Merchant Portal with dashboard, reports, refunds and bulk refund files. It states PCI DSS Level 1 compliance and adherence to BNM, PayNet and MyDebit guidelines.

## Registration

1. Contact CardBizPay (no self-serve signup is documented): `sales@cardbiz.com.my`, `+603 7890 3000`, or the form at `https://cardbizpay.my/contact/`. The cardbiz.com.my product page has an "Apply Now" button.
2. Submit documents (guide "Required Documents to submit"):
   - Sole proprietor / partnership: full SSM registration copy, all directors' ICs, latest 3 months' company bank statements.
   - Sdn Bhd registered before 31/01/2017: M&A, Form 24 and Form 49, directors' ICs, latest 3 months' bank statements.
   - Sdn Bhd registered after 31/01/2017: Section 14, Section 17, Section 58 and 236, directors' ICs, latest 3 months' bank statements.
3. CardBizPay's administrator creates the merchant profile and enables the APIs and payment methods for it.

Many registration FAQ pages on cardbizpay.my (eligibility, foreigners, multiple accounts, approval check) are unpublished placeholders; treat those topics as "ask CardBizPay".

## Credentials And Configuration

Issued or configured by the CardBizPay system administrator, not self-generated:

| Item | Where it comes from | Use |
|---|---|---|
| Merchant No (16 chars) | Issued after profile creation | `MerchantNo` in every request |
| Access Key | Issued with Merchant No | Named in the prerequisites; used in "all web payment methods" |
| Secret Key | Unique per merchant, shared only with CardBizPay | HMAC-SHA512 signing key |
| Payment Return URL (Callback URL) | Merchant gives it to the administrator | Merchant API response post target |
| Notify URL | Merchant gives it to the administrator | Open Payment API result post after interruptions |
| API access flags | Profile settings | Enquiry/void/refund may be disabled (codes `10`, `11`, `12`) |
| Subscribed payment methods | Profile settings | Drives `PymtMethodSelect` validity (codes `14`, `15`) |

The docs do not say whether the Access Key and the Secret Key are the same value; ask CardBizPay. The Payment Return URL can be changed by the administrator, or by the merchant in the Merchant Portal (`https://cardbizpay.com/Merchant`) when a portal login is given.

Suggested environment variables (names are this skill's convention, not CardBizPay's):

```text
CARDBIZPAY_BASE_URL=https://uat.cardbizpay.com/paymentgateway   # production: https://www.cardbizpay.com/paymentgateway
CARDBIZPAY_MERCHANT_NO=<MERCHANT_NO>
CARDBIZPAY_SECRET_KEY=<SECRET_KEY>
CARDBIZPAY_ACCESS_KEY=<ACCESS_KEY>   # only if CardBizPay says it differs from the secret key
```

## Staging vs Production

- Staging base: `https://uat.cardbizpay.com/paymentgateway`. Production base: `https://www.cardbizpay.com/paymentgateway`.
- Ask CardBizPay for separate staging credentials and a staging Return URL. Public test cards, FPX test banks and a sandbox signup are not published.

## Payment Methods

From the Payment & Refunds FAQ: credit and debit cards (Visa, Mastercard, MyDebit); e-wallets (Touch 'n Go, Boost, GrabPay, ShopeePay, Maybank QRPay, DuitNow, WeChat Pay, Alipay+); online banking and FPX (B2C and B2B). Availability depends on what is subscribed on the merchant profile. The API exposes method groups only (`Card`, `FPX`, `FPX B2C`, `FPX B2B`, `e-Wallet`, `DuitNow`); individual bank or wallet codes are not documented.

Processing times (FAQ): cards usually instant (up to 24 hours in some cases), e-wallets typically instant, FPX usually minutes (up to 1 to 2 business days).

## Settlement

From the Settlement FAQ:

- Cards, e-wallets and FPX: typically T+3 business days (T = transaction day), business days only.
- Early settlement may be available for a fee; ask the account manager.
- Delays: holidays, high-risk checks, wrong bank details, AML/KYC checks.
- Bank account change: email `cardbizpay@cardbiz.com.my` or call `03-7890 3000`, then send the new bank statement (plus company registration documents and an authorization letter if required); 3 to 5 business days.
- Bulk settlement is supported for high volumes.

## Refunds And Disputes

- Merchants refund from the Merchant Portal or the Refund API; refunds go back to the original payment method.
- Duplicate charges: refunds processed within 5 to 7 business days per the FAQ.
- Chargebacks: provide proof of fulfilment, usually within 30 to 45 days.

## Pricing

Do not quote numbers from memory; rates change and the two official pages differ. Link to:

- `https://cardbizpay.my/pricing/`
- `https://cardbiz.com.my/cardbiz-payment-services/cardbizpay-rates/`

## Support

- Email: `cardbizpay@cardbiz.com.my` (account changes such as registered email), `sales@cardbiz.com.my` (sales).
- Phone: `+603 7890 3000`.
- No public status page is documented.
