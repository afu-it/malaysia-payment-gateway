# Adyen Account Setup

Checked 2026-10-21.

## Source URLs

- `https://docs.adyen.com/set-up-adyen-payments`
- `https://docs.adyen.com/set-up-adyen-payments/application-requirements`
- `https://docs.adyen.com/online-payments/go-live-checklist`
- `https://docs.adyen.com/online-payments/integration-checklist`
- `https://docs.adyen.com/development-resources/api-credentials` and `.../api-credentials/roles`
- `https://docs.adyen.com/development-resources/client-side-authentication`
- `https://docs.adyen.com/development-resources/live-endpoints`
- `https://docs.adyen.com/payment-methods/add-payment-methods`
- `https://docs.adyen.com/account/getting-paid`, `https://docs.adyen.com/account/supported-currencies`
- `https://docs.adyen.com/reporting/automatically-get-reports`, `https://docs.adyen.com/reporting/settlement-detail-report`
- `https://docs.adyen.com/development-resources/pci-dss-compliance-guide`
- Pricing (link only): `https://www.adyen.com/pricing`

## Product Positioning

Adyen is a payment provider and acquirer (its own words on the setup page). Malaysia is listed as an acquiring country with MYR payouts on the supported currencies page. It suits larger online merchants; live accounts are approved after Sales contact and checks. Malaysian methods in the docs: cards, FPX (through Razer Merchant Services), DuitNow QR, Touch 'n Go, GrabPay, Boost, Atome, Alipay, Alipay+, WeChat Pay, Apple Pay, Google Pay.

## Account Setup Checklist

Source: `https://docs.adyen.com/set-up-adyen-payments`

1. Create a free test account at `https://www.adyen.com/signup` (test Customer Area `https://ca-test.adyen.com/`). A test account does not guarantee a live account.
2. Check the prohibited and restricted list `https://www.adyen.com/legal/list-restricted-prohibited`, then contact Sales `https://www.adyen.com/contact/sales`.
3. Build and test on the test account.
4. Apply for the live account (documents below). If approved, sign the contract and, for cards, the PCI data security attestation (Drop-in, Components, Hosted Checkout and Pay by Link: SAQ A, if you meet the SAQ A eligibility rules; API-only with raw card data: SAQ D. Source: `https://docs.adyen.com/development-resources/pci-dss-compliance-guide`).
5. Get the live Customer Area `https://ca-live.adyen.com/` (login sent to the user who created the test account). Nothing is copied from test.
6. Configure live: users and roles, payment methods, payout bank account, API credential, client key, webhooks, HMAC key, reports, risk rules. Then follow the go-live checklist.

## Documents For The Live Application

Source: `https://docs.adyen.com/set-up-adyen-payments/application-requirements`

- Company registration document from a public, government or judicial authority, issued or signed within the last 12 months, showing legal entity name and registration number (trading name is not accepted). Malaysia has no row in the registration number format table (checked 2026-10-21); ask Adyen which number format to give.
- Proof of tax information or VAT information.
- Payout account proof (optional at application): bank statement, deposit form, online banking screenshot, bank letter or cheque, under 12 months old, showing your legal name, account number, bank identity. No photos of bank cards.
- Proof of identity for the contract signer and shareholders with 25% or more (passport data page, ID card front and back, or driving licence), plus proof of residency.
- Ownership and control chart if asked (UBOs, signatories). Malaysia is in the list where ID numbers can be given for UBOs.

Website requirements (checked by Adyen):

- Terms and conditions: legal entity name, product description, how payments happen, delivery, refund and cancellation policy, third parties, governing law.
- Subscriptions and trials: explicit consent; reminder with an online cancellation link at least 7 days before charging after a trial or a change; receipts showing trial length and amounts; easy cancellation.
- Privacy policy (data stored, shared, cookies), delivery information, refund policy, contact details (legal name, email, phone).
- At checkout: legal entity name, trading name, company registration number, company location, and a checkbox "I agree to the Terms and Conditions and the Refund and Cancellation policy" with links, required before paying.

## Where Credentials Live

| Item | Customer Area path | Notes |
| --- | --- | --- |
| Merchant account code | Account switcher (top) | Sent as `merchantAccount`. Company account holds merchant accounts. |
| API key | **Developers** > **API credentials** > `ws_...` > **Server settings** > **Authentication** > **API key** > Generate | Shown once. Old key valid 24 h after regenerate. Separate key in live. |
| Basic auth password | Same page, **Basic authentication** | Replaces the old password at once. |
| Roles | Same credential > **Roles** | Checkout webservice, Merchant PAL webservice, Merchant Recurring (tokens), Checkout encrypted cardholder data. |
| Allowed IP range | Same credential | Optional. |
| Client key | Same credential > **Client settings** > **Authentication** > **Client key** | Plus **Allowed origins** (HTTPS in live). |
| Live URL prefix | Live Customer Area > **Developers** > **API URLs** > **Prefix** | Builds the live Checkout host. |
| Webhook + HMAC key | **Developers** > **Webhooks** > **Create new webhook** > Standard webhook > Security > **HMAC key** | One key per endpoint; new key in live. |
| Additional data | **Developers** > **Additional data** | Extra fields in API responses; copy test settings to live. |
| Payment methods | **Payments** > **Payment methods** > **Request payment methods** | Role **Change payment methods**. |
| Capture delay | **Settings** > **Account settings** > **Capture delay** | Immediate default. |
| Tokenization toggle | **Payments** > **Checkout settings** > **Tokenization** | Optional automatic tokens. |
| Themes (Hosted Checkout, links) | **Pay by Link** > **Themes** | |
| Report user | **Developers** > **API credentials** > Create > User type **Report user**, role **Merchant Report Download role** | For report downloads. |

## Required Environment Variables

Names are suggestions; keep them server-side except the client key and environment.

```text
ADYEN_ENVIRONMENT=test              # or live
ADYEN_API_KEY=<API_KEY>             # server only
ADYEN_MERCHANT_ACCOUNT=<MERCHANT_ACCOUNT>
ADYEN_CLIENT_KEY=<CLIENT_KEY>       # test_... or live_..., safe for the browser
ADYEN_HMAC_KEY=<HMAC_KEY>           # hex string, server only
ADYEN_LIVE_URL_PREFIX=<PREFIX>      # live only, from Developers > API URLs
ADYEN_WEBHOOK_USERNAME=<USERNAME>   # if Basic auth on the webhook
ADYEN_WEBHOOK_PASSWORD=<PASSWORD>
ADYEN_RETURN_URL=https://shop.example.my/checkout/return
ADYEN_REPORT_API_KEY=<REPORT_USER_API_KEY>   # optional, report downloads
```

Official Node library: `new Config({ apiKey, environment: EnvironmentEnum.TEST })`; for live set `environment: EnvironmentEnum.LIVE` and `liveEndpointUrlPrefix: <PREFIX>`. Package `@adyen/api-library`; frontend `@adyen/adyen-web`. Other official server libraries (Java, PHP, C#, Go, Python, Ruby) and example apps under `https://github.com/adyen-examples` are listed at `https://docs.adyen.com/development-resources/libraries`.

## Payment Method Activation

Source: `https://docs.adyen.com/payment-methods/add-payment-methods`

1. Switch to the merchant account; **Payments** > **Payment methods** > **Request payment methods**.
2. Filter by location (Malaysia) and type; select; **Add**; apply to this merchant account; fill in any prompts.
3. Repeat in the live Customer Area (requests are reviewed).
4. Through Support instead: DuitNow (test and live), Touch 'n Go (test and live), Atome live, WeChat Pay mobile browser enrolment, GrabPay test wallet credentials, Atome sandbox shopper. See `adyen-payment-methods-my.md`.
5. Activate or deactivate later under **More actions**.

## Payouts And Settlement

Source: `https://docs.adyen.com/account/getting-paid`

- Funds are paid to your payout bank account per merchant account, on a net basis (sales minus refunds, chargebacks, fees).
- Payout models: Sales day payout (all of a sales day in one batch) or Pass-through payout (batches as schemes pay Adyen). Payout frequency is configurable per merchant account.
- After a batch closes, payout initiation takes one to two business days, plus up to three business days for the bank. No payouts on bank holidays.
- MYR payouts only to a local Malaysian bank account; no SWIFT for MYR. Source: `https://docs.adyen.com/account/supported-currencies`
- Per-method settlement delays for Malaysian methods: Not documented in official docs (checked 2026-10-21); adyen.com payment method pages may state them. Confirm with Adyen.
- Sales day payout delay (source `https://docs.adyen.com/account/sales-day-payout`): 2 business days when at least 80% of transactions are in AUD, CAD, EUR, GBP, NZD, RON, SEK or USD acquired locally; 3 business days for other Sales day payout accounts (this covers MYR); 3, 7 or 12 business days when onboarded through a partner. Slow-settling schemes (Amex 7 days) can lengthen it.
- Check payouts: Customer Area > **Balances** > **Sales to payouts** > **Payouts**.
- Optional reserve for refunds and chargebacks when in-process funds are short.

## Reports And Reconciliation

Source: `https://docs.adyen.com/reporting/automatically-get-reports`

1. Configure automatic report generation (role **Merchant Report** or **Generate key financial reports**).
2. Receive `REPORT_AVAILABLE` webhook: `pspReference` = file name, `reason` = download URL.
3. Create a Report user credential with the **Merchant Report Download role** (API key or Basic auth).
4. Download with HTTP GET on the `reason` URL, header `x-API-key: <REPORT_USER_API_KEY>`, `Accept-Encoding: gzip`.
5. Reconcile with the Settlement details report (per payout batch, transaction level; `https://docs.adyen.com/reporting/settlement-reconciliation/transaction-level/settlement-details-report`). Journal types to map: `Settled` (credit), `Refunded` (debit), `Fee`, `Chargeback`, `SecondChargeback`, `ChargebackReversed`, `RefundedReversed`, `MerchantPayout`, `BankInstructionReturned`, `InvoiceDeduction`, `DepositCorrection`, `Balancetransfer`, and for Sales day payout `CaptureFailed`, `RefundFailed`, `RefundNotCleared`, `SettledReversed`. Settlement has no webhook.
6. Debugging: Customer Area API logs keep JSON requests for 30 days; webhook event logs show deliveries. Sources: `https://docs.adyen.com/development-resources/logs-resources/api-logs`, `https://docs.adyen.com/development-resources/logs-resources/webhook-event-logs`

## Pricing

See `https://www.adyen.com/pricing`. Do not hardcode fees; they change and are often negotiated.

## Go-Live Summary

Source: `https://docs.adyen.com/online-payments/go-live-checklist`

1. Live users, roles, two-factor authentication, notification center alerts.
2. Payment methods requested in live; `shopperStatement` set.
3. Payout bank account and reports configured.
4. Risk profile reviewed, 3DS rules (Dynamic 3D Secure), PCI attestation done.
5. Live API key, client key, prefix endpoint, live webhook with live HMAC key, additional data settings mirrored, end-to-end tests on real devices.
