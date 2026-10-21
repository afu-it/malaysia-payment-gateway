# OxPay Account Setup

Use this reference when explaining OxPay merchant registration, merchant portal access, API keys and Sign Keys, go-live testing, payment method activation, settlement reports, pricing and support.

## Source URLs

- `https://oxpay.com.my/`
- `https://oxpay.com.my/payment/`
- `https://oxpay.com.my/support/`
- `https://oxpay.com.my/contact/`
- `https://oxpay.com.my/about/`
- `https://docs.oxpayfinancial.com/getting-started`
- `https://docs.oxpayfinancial.com/credentials`
- Portal guides linked from Getting Started (Google Drive PDFs published by OxPay): "How to get Signature Key and API Key", "Merchant Portal User Guide" (rev 1.0, 29 Sep 2025), "[MFA] How to access Merchant Portal" (rev 2.0.0, 2025-12-17), "How to create new account in OxPay" (2025-10-23), "OxPay Lite User Guide" (rev 1.0, 25 Sep 2025). URLs in `source-map.md`.

## Entity And Positioning

- Malaysian entity: Oxpay (M) Sdn Bhd (942856M), D1-3A-05, Solaris Dutamas, No 1 Jalan Dutamas 1, 50480 Kuala Lumpur. Tel +603 6205 3015 (contact page). The about page says OxPay was formerly known as MC Payment, established 2005.
- Group: OxPay Financial (SGX: TVV). Docs and portal are run by OxPay SG Pte Ltd and are shared across Singapore, Malaysia and Thailand.
- The Malaysia site markets card acceptance, alternative payments (e-wallets, bank transfers, BNPL), payouts, online and offline (ffastpay O2O, ffastpay Limo) solutions, and says WordPress plugins and Magento extensions are available on request ("Get in touch for details"). No plugin download is published.

## Becoming A Merchant

Two paths are documented:

1. **Sales-led onboarding** (Getting Started, section 2): merchant signs up (Malaysia site "Apply for Merchant Account" Google Form on `https://oxpay.com.my/support/`), onboarding starts, OxPay emails credentials (MID, TID, API document), merchant integrates in parallel, runs production tests with OxPay test cases, submits results, goes live on OxPay technical approval.
2. **Self sign-up in the merchant portal** (Merchant Portal User Guide): go to `https://login.oxpayfinancial.com/login`, click Sign Up, register with Google SSO or name/phone/email/password (email OTP), submit business information for compliance verification, select payment methods, accept Terms & Conditions for immediate activation. Funds are disbursed only after compliance checks pass; if verification fails, collected funds are returned to customers.

Prerequisites listed in Getting Started: OxPay merchant account, MID, TID and API document, merchant portal access, a Basecamp channel for technical questions.

Required compliance documents are not listed publicly. Ask OxPay during onboarding.

For portal access after sales-led onboarding, give OxPay the email you want registered; then sign up at the portal with that same email (it must match the production details) and verify the OTP.

## Merchant Portal

- Portal: `https://merchant.oxpayfinancial.com`. Login: `https://login.oxpayfinancial.com/login`.
- Login by email/password always requires email MFA; Google sign-in skips MFA.
- Sections: Products (OxPay Terminal, eCommPay, OxPay API, OxPay Lite, Click2Pay, Scan2Pay, Payout), Dashboard (Statistics, Transactions with CSV export, Report), Settings (Payment Method, Configurations > API Keys, Configurations > Lite Account, Documentations > API Specifications, Support).
- Reports: Settlement Reports (payout schedules and transfer amounts), Daily Reports, Monthly Reports.

## API Key And Sign Key

From `/credentials` and the "How to get Signature Key and API Key" PDF:

1. Log in to the merchant portal.
2. Configurations > API Keys.
3. Add New Key.
4. Select the Merchant Name.
5. Select Read and Create (the PDF says Read and Write).
6. Save. A text file downloads to your computer with the credentials.

The API Key identifies your business (sent as `Authorization: Bearer <API_KEY>`). The Sign Key signs the payload (HMAC-SHA256, `Signature` header) and verifies webhooks. Keys can be listed and deleted in the same screen. Store the downloaded file in a secret manager and delete the local copy.

Suggested env vars:

```text
OXPAY_BASE_URL=https://api.oxpayfinancial.com
OXPAY_API_KEY=<API_KEY>
OXPAY_SIGN_KEY=<SIGN_KEY>
OXPAY_MERCHANT_ID=<MID>
OXPAY_TERMINAL_ID=<TID_14_CHARS>
OXPAY_NOTIFICATION_URL=https://<your-domain>/webhooks/oxpay
```

## Sandbox Versus Live

- The OpenAPI file lists a sandbox server `https://api.uat.oxpayfinancial.com`.
- Getting Started says there is no UAT environment and testing happens in production with OxPay-supplied test cases.
- No public test cards, test wallets or sandbox signup are published. Ask OxPay which host and credentials to use for testing, and keep live and test keys separate.

## Payment Methods

- Enable methods in the portal under Settings > Payment Method, and confirm with OxPay that each is provisioned on your terminal.
- The hosted page lists what is enabled; `GET /payment?session_id=` returns `payment_brand_list`.
- The public API documents card, Alipay, WeChat Pay, GrabPay, ShopeePay, Atome, Dash, PayNow and UPI. Malaysian rails (FPX, DuitNow QR, Touch 'n Go, Boost, Grab PayLater) are not in the public API docs; confirm availability, brand strings and MYR support in writing before building for them.
- The hosted page requires the terminal's Ecommerce feature.

## Pricing And Settlement

- Pricing is not published. Contact OxPay sales via `https://oxpay.com.my/contact/`. Do not quote numbers.
- Settlement timing is not published; the portal's Settlement Reports show payout schedules and amounts.
- Payout API responses include a `transfer_fee` per payout.

## Support

- Email: `support@oxpayfinancial.com`.
- Hotline and WhatsApp: (65) 6299 0030, Mon to Fri 9am to 6pm SGT, excluding public holidays (portal guide).
- Help desk: `https://oxpay.freshdesk.com/support/home`.
- Include MID or registered email, issue description, screenshots and timestamp when raising a ticket.
