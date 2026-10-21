# Axaipay Account Setup

Use this reference when explaining Axaipay registration, credentials, staging vs production, plugins, AutoDebit products, refunds, dashboard security, and pricing.

## Source URLs

- `https://www.axaipay.com/online-payments`
- `https://www.axaipay.com/autodebit`
- `https://www.axaipay.com/3rd-party-integration`
- `https://www.axaipay.com/contact-us`
- `https://www.axaipay.com/prohibited-activities`
- `https://support.axaipay.com/support/solutions` (knowledge base root)
- `https://dashboard.axaipay.my/login` (merchant portal)

## Company And Product

- Operator: Axai Digital Sdn Bhd (Axaipay Malaysia), regulated as a Merchant Acquirer by Bank Negara Malaysia under the Financial Services Act 2013 (site footer). Axaipay Australia is a separate entity (Outlier Private Pty Ltd).
- eComPay is the online gateway: cards, debit cards, FPX online banking, e-wallets, BNPL. Official plugin handbook lists cards, FPX (individual/company), e-wallets (Touch 'n Go, Boost, GrabPay, MCash and more), DuitNow QR, instalment (CIMB cards), PayLater and UnionPay. MYR only.
- AutoDebit: FPX DirectDebit eMandate recurring collection; the site says 22 participating banks.
- No-code: AxaiCart payment pages and links, QRPay for in-store QR on a smartphone, AutoDebit subscription links.

## Registration Checklist

1. Contact Axaipay sales via `https://www.axaipay.com/contact-us` ("Talk to an Expert"). There is no self-serve API signup in the official docs.
2. Check your business is not on the prohibited list (`https://www.axaipay.com/prohibited-activities`; KB article "Prohibited Business Activities"). Accounts in those segments are suspended.
3. Required documents: not published in the official docs (checked 2026-10-21); Axaipay's onboarding team provides the list.
4. Request a staging (demo) account from `support@axaipay.com` before integrating (both API PDFs say this).
5. Give Axaipay your default Redirect URL and Backend URL at registration. Per-request `redirectUrl`/`backendUrl` can override them for eComPay.
6. After onboarding you receive production Merchant ID, Signing Key and API Key.

## Credentials

| Credential | Used for | Where |
|---|---|---|
| Merchant ID (`mchtId` / `merchantId`) | Every request | Issued by Axaipay; shown in the merchant dashboard (PrestaShop and VirtueMart plugin docs) |
| Signing Key | HMAC-SHA512 secret for all signatures | Issued by Axaipay; "Found in Merchant Dashboard" per plugin docs |
| API Key | `x-api-key` header for QR, inquiry, void, AutoDebit inquiry | Issued at onboarding (WooCommerce plugin docs) |

Staging credentials come from the staging merchant dashboard and differ from production. The exact dashboard menu path to view keys is not documented.

Suggested env var names:

```text
AXAIPAY_ENV=staging            # or production
AXAIPAY_MERCHANT_ID=<MERCHANT_ID>
AXAIPAY_SIGNING_KEY=<SIGNING_KEY>
AXAIPAY_API_KEY=<API_KEY>
AXAIPAY_REDIRECT_URL=https://example.com/pay/axaipay/return
AXAIPAY_BACKEND_URL=https://example.com/pay/axaipay/callback
```

## Merchant Portal

- Login: `https://dashboard.axaipay.my/login`.
- Logo on payment page: profile menu ("Hi, XXXXX") > Profile > Merchant Logo (PNG, JPG, JPEG).
- 2FA: profile icon > Login Protection > Enable two-factor authentication, scan with an authenticator app, enter the 6-digit pin. "Remember me" trusts a device for 30 days. Staff can get their own logins and 2FA devices. High-risk actions (refunds, settlement bank changes) require an active 2FA session. Lockout recovery via `support@axaipay.com`.
- AutoDebit products: create the recurring subscription link in the portal; its URL `https://autodebit.axaipay.my/<productCode>` gives the `productCode` for the API.

## Plugins (official, downloadable from the KB)

| Platform | Version | Settings |
|---|---|---|
| WooCommerce | 2.0.0 (16 Oct 2025), tested WP 6.8.3 / WC 10.2.2 | Live Merchant ID, Signing Key, API Key; Test Mode with test Merchant ID, Signing Key, API Key. WooCommerce > Settings > Payments |
| PrestaShop | alpha 1.0, PrestaShop 7 and 8 | Merchant ID, Signing Key, Sandbox Merchant ID, Sandbox Signing Key, Environment Type |
| VirtueMart | 1.0, Joomla 3.x, VirtueMart 3/4 (Joomla 4 not supported) | Merchant ID, Signing Key, sandbox pair, Use Sandbox, order status mapping, currency MYR |
| OpenCart | 1.0 (2022) | Download via OpenCart Marketplace or Google Drive link; uses older request fields |

The WooCommerce plugin is the best code reference: it uses the v1.18 fields, verifies callback signatures with `hash_equals`, and answers the backend callback with HTTP 200 `OK`.

## Refunds And Disputes

- Refunds are manual: fill the Refund Request Form PDF and email `support@axaipay.com`.
- Axaipay processes within 2 working days; the customer may see funds after up to 14 working days.
- Transaction fees are not refunded and a refund fee applies per the merchant agreement.
- Card refunds return to the original card; other refunds may go by DuitNow/IBG.
- Axaipay's internal sources of funds: reverse if unsettled, deduct from upcoming settlement, or merchant repays.
- Customers see "Axai" or "Axai Digital Sdn Bhd" on statements, sometimes with the merchant name.

## AutoDebit Customer Experience

- Customer opens the subscription link (must start `https://autodebit.axaipay.my/`), creates an account with email OTP, picks B2C or B2B and a bank, pays RM1 via FPX.
- First billing is reduced by the RM1 already paid; debits fall on a fixed date, moved after weekends or holidays.
- Current, savings and credit card accounts of participating banks work; debit cards do not.
- Customers maintain or cancel at `https://users.axaipay.my/login` > Maintenance/Termination. Cancelled mandates cannot be charged.

## Settlement And Payout

Settlement timing, payout schedule and rolling reserve are not published in the official docs (checked 2026-10-21). They come from the merchant agreement.

## Pricing

Do not quote rates. Axaipay states no monthly or yearly maintenance fees and per-sale pricing on `https://www.axaipay.com/online-payments`; actual rates come from sales via `https://www.axaipay.com/contact-us`.

## Support

- Technical and merchant support: `support@axaipay.com`.
- Knowledge base: `https://support.axaipay.com/support/solutions`.
- Phishing reports: forward to `support@axaipay.com`. Genuine payment pages are on `https://secured.axaipay.my`.
