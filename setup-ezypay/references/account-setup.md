# Ezypay Account Setup

Use this when explaining onboarding, credentials, sandbox access, go-live, fees model, settlement, and environment variables.

## Source URLs

- `https://www.ezypay.com/my`
- `https://www.ezypay.com/pricing-plan`
- `https://www.ezypay.com/sandbox-request`
- `https://www.ezypay.com/become-a-partner`
- `https://www.ezypay.com/contact`
- `https://developer.ezypay.com/docs/integrating-with-ezypay`
- `https://developer.ezypay.com/docs/partner-integration-process`
- `https://developer.ezypay.com/docs/integrated-merchant-integration-process`
- `https://developer.ezypay.com/docs/merchant-onboarding`
- `https://developer.ezypay.com/docs/ezypays-fees`
- `https://developer.ezypay.com/docs/display-customer-paid-fees`
- `https://developer.ezypay.com/docs/authentication-1`
- `https://developer.ezypay.com/docs/sandbox-intro`
- `https://developer.ezypay.com/docs/go-live-checklist`
- `https://developer.ezypay.com/docs/partners-go-live-checklist`
- `https://developer.ezypay.com/docs/integrated-merchant-go-live-checklist`
- `https://developer.ezypay.com/docs/settlement`
- `https://developer.ezypay.com/docs/chargeback`

## Positioning

Ezypay is an Australian-founded (1996) subscription billing platform for recurring and one-off payments, focused on gyms, health clubs and membership businesses, sold mainly through software partners (see the partner directory) or direct API integration. Its Malaysia page markets "subscription & direct debit", but the developer docs list Malaysia as cards only (Visa, Mastercard, Apple Pay and Google Pay in beta), no bank direct debit. Kuala Lumpur office: Level 14, Vista Tower, The Intermark, No. 348 Jalan Tun Razak, 50400 Kuala Lumpur (from the Malaysia failed payment guide footer). Support: `help@ezypay.com`.

## Getting access

1. Sandbox: request at `https://www.ezypay.com/sandbox-request`. Sandbox credentials are shared when integration starts. Sandbox accounts inactive for 8 weeks are disabled (ask Ezypay to extend).
2. Choose your path:
   - Software partner (`/docs/partner-integration-process`): build once, offer Ezypay to many merchants. Partner agreement, system demo, operations plan, marketing plan, partner setup sign-off.
   - Integrated merchant (`/docs/integrated-merchant-integration-process`): process payments in your own platform. Integrated merchant agreement, system demo, operations plan, integration setup sign-off.
3. Merchant onboarding (`/docs/merchant-onboarding`): the merchant submits an application form (business information, fee schedule, settlement details, terms) with supporting documents; Ezypay runs risk and compliance checks; then configures the account and issues API credentials. Typical timeline: a couple of business days after a complete application. The exact document list is given by Ezypay; it is not published in the developer docs (checked 2026-10-21).
4. Each branch needs its own merchant account. Sharing a merchant account across branches is not allowed.

## Credentials

| Credential | Owner | Where it comes from |
| --- | --- | --- |
| `client_id`, `client_secret` | partner or integrator | issued by Ezypay after integration sign-off; one pair reused for all merchants |
| username, password | each merchant | Welcome Email after onboarding |
| merchant ID (UUID) | each merchant | issued with merchant credentials; sent in the `merchant` header |
| webhook client key | you | you generate it and register it on each webhook |

There is no self-serve dashboard path for API keys in the developer docs. A merchant portal exists at `https://portal.ezypay.com/login`. Ezypay recommends a secure form in your own portal so merchants enter their username, password and merchant ID themselves.

Sandbox and production credentials are different. Production credentials are issued only after the pre go-live checklist.

## Go-live checklist (post go-live steps)

1. Switch to production `client_id` and `client_secret`.
2. Switch every host: `identity.ezypay.com`, `vault.ezypay.com`, `api-global.ezypay.com`, `hosted-global.ezypay.com`.
3. Enter merchant credentials for every new merchant or branch.
4. Create the webhook for every new merchant or branch (automate it).

Mandatory integration items Ezypay checks (from `/docs/integrating-with-ezypay`):

1. Customers accept Ezypay's Terms and Conditions.
2. All Ezypay fees charged to the customer are displayed during payment method creation.
3. Payment methods relevant to each region are integrated and enabled.
4. Webhooks are consumed to reflect invoice and subscription status.
5. Invoice status (with failure reasons) and Ezypay fee details are shown per invoice.
6. Customers are notified of successful and failed payments.
7. Merchants can manually retry failed invoices and an automatic retry exists.
8. Merchants can write off or mark invoices externally paid.
9. A refund process for disputed payments.
10. A chargeback process.

## Fees model

Pricing is not published: `https://www.ezypay.com/pricing-plan` ends in a contact form. Link to it; do not hardcode rates.

Fee types (configured per merchant, each merchant-paid or customer-paid):

- `load_fee`: new customer, charged on first successful debit.
- `transaction_fee`: every debit attempt, success or failure.
- `failed_payment_fee`: each failed attempt (not charged when declined for suspected fraud, since 04/04/2025).

Customer-paid fees are added to the invoice as items, so the customer pays more than your item amount. Fees stack per attempt. Fee capping is case by case. Retrieve the customer-paid schedule with `GET /v2/billing/merchant/pricing/fees` and display it before payment method creation; show the fee breakdown from `invoice_created`, `invoice_past_due` and `invoice_paid` afterwards. Ezypay refunds its fees only on full refunds. Writing off or recording external payment reallocates fees to the merchant.

## Settlement

- Funds go to the merchant's bank account daily (AU and NZ only), weekly or monthly, on a day the merchant chooses.
- Holding period for Malaysia: 3 business days (T+3). Public holidays push it out.
- Settlement = successful collections minus fees, refunds, failed payment deductions. Reports are emailed to the merchant: Settlement Summary (PDF), Settlement Detailed Report (CSV), Settlement Tax Invoice (PDF). External payments do not appear.
- Use `transaction_settled` webhooks and the settlement APIs to reconcile (see `ezypay-billing.md`).
- Chargebacks are deducted from the next settlement and returned if the merchant wins. Merchants must send evidence quickly (signed authority, membership agreement, access logs, refund copies, prior non-disputed payments).

## Payment method activation

Payment methods are not enabled by default on merchant accounts. Ask Ezypay to enable Visa, Mastercard and, if wanted, Apple Pay and Google Pay (beta in Malaysia). Apple Pay also needs domain registration and a management URL; Google Pay in apps needs Google approval. Merchant configuration also controls: who pays each fee, automatic retry schedule and maximum attempts, failed payment emails and Pay Now, subscription stop rules, checkout failed-attempt threshold (default 3), and settlement schedule.

## Environment variables

Use names that match the app. Suggested:

```text
EZYPAY_ENV=sandbox
EZYPAY_IDENTITY_URL=https://identity-sandbox.ezypay.com
EZYPAY_API_URL=https://api-sandbox.ezypay.com/v2
EZYPAY_VAULT_URL=https://vault-sandbox.ezypay.com
EZYPAY_HOSTED_URL=https://hosted-global-sandbox.ezypay.com
EZYPAY_CLIENT_ID=<CLIENT_ID>
EZYPAY_CLIENT_SECRET=<CLIENT_SECRET>
EZYPAY_MERCHANT_ID=<MERCHANT_ID>
EZYPAY_MERCHANT_USERNAME=<MERCHANT_USERNAME>
EZYPAY_MERCHANT_PASSWORD=<MERCHANT_PASSWORD>
EZYPAY_WEBHOOK_CLIENT_KEY=<RANDOM_SECRET_YOU_GENERATE>
EZYPAY_WEBHOOK_URL=https://example.com/api/webhooks/ezypay
EZYPAY_CHECKOUT_SUCCESS_URL=https://example.com/pay/ezypay/success
EZYPAY_CHECKOUT_CANCEL_URL=https://example.com/pay/ezypay/cancel
```

Production:

```text
EZYPAY_ENV=production
EZYPAY_IDENTITY_URL=https://identity.ezypay.com
EZYPAY_API_URL=https://api-global.ezypay.com/v2
EZYPAY_VAULT_URL=https://vault.ezypay.com
EZYPAY_HOSTED_URL=https://hosted-global.ezypay.com
```

For partners with many merchants, store merchant ID, username and encrypted password per merchant in the database instead of env vars, and keep one webhook (and client key) per merchant.
