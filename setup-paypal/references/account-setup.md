# PayPal Account Setup (Malaysia)

Use this when explaining registration, credentials, sandbox vs live, env vars, dashboard settings, Malaysian account limits, pricing and payouts.

## Source URLs

- `https://www.paypal.com/my/business`
- `https://www.paypal.com/my/business/open-business-account`
- `https://www.paypal.com/my/business/paypal-business-fees`
- `https://www.paypal.com/webapps/mpp/country-worldwide`
- `https://developer.paypal.com/api/rest`
- `https://developer.paypal.com/api/rest/production`
- `https://developer.paypal.com/api/rest/reference/policies-and-guidelines`
- `https://developer.paypal.com/api/codes/currency`
- `https://developer.paypal.com/sandbox-testing/accounts`
- `https://developer.paypal.com/v5/save-payment-methods/overview`
- `https://developer.paypal.com/expanded/eligibility`
- `https://developer.paypal.com/payouts/supported-features`

## Product Positioning For Malaysia

PayPal is a global wallet and checkout. For a Malaysian seller it is mainly a way to accept payments from buyers who pay with PayPal (balance or linked cards) or with a card through PayPal's hosted guest flow where eligible, often from overseas. The developer docs list no FPX, DuitNow QR or Malaysian e-wallets, so pair PayPal with a local gateway for domestic buyers.

## Register

1. Open a PayPal Business account for Malaysia: `https://www.paypal.com/my/business/open-business-account`. The page lists what is needed: full legal name, email address, a password, business registration number, business description, and business bank account details. No set-up fee is stated there. Sales contact shown on the page: +60-3-7724-0440.
2. Verify the account. Live REST credentials need "a verified Premier or verified Business PayPal account" (`/api/rest/production`).
3. Log in to the Developer Dashboard (`https://developer.paypal.com/dashboard/`) with the same account.

## Credentials

- Developer Dashboard > Apps & Credentials > Sandbox/Live toggle > REST API apps > Default Application or Create App. Copy Client ID and Secret per environment.
- Sandbox and live credentials, Webhook IDs and buyer accounts are separate. The sandbox app ID is not accepted by production.
- No IP list for REST calls or webhook senders is published in the pages read (the only published subnets are for SFTP report transfer at `/secure-file-transfer`). Webhook authenticity comes from signature verification.

Suggested env vars (names are a convention, not PayPal's):

```text
PAYPAL_ENV=sandbox                 # or live
PAYPAL_API_BASE=https://api-m.sandbox.paypal.com   # live: https://api-m.paypal.com
PAYPAL_CLIENT_ID=<CLIENT_ID>       # public, also used in the browser SDK
PAYPAL_CLIENT_SECRET=<CLIENT_SECRET>   # server only
PAYPAL_WEBHOOK_ID=<WEBHOOK_ID>     # per environment, from the app's Webhooks section
PAYPAL_CURRENCY=MYR
```

## Dashboard Settings To Check

- App > Webhooks: add the listener URL (HTTPS, port 443) and events; record the Webhook ID.
- App > Features: payment capabilities (for example Save payment methods / vaulting; PayPal and Venmo toggle). Card vaulting additionally needs Expanded Checkout approval, which is not listed for Malaysia.
- PayPal account > Account Settings > Payment preferences > Block payments: set how payments in currencies you do not hold are handled. Otherwise those payments stay pending until accepted manually (capture `status_details.reason` `RECEIVING_PREFERENCE_MANDATES_MANUAL_ACTION`). Also controls duplicate invoice ID blocking.
- Business profile: merchant descriptor shown on card statements (combined with `soft_descriptor`).
- Partial capture for PayPal authorizations must be enabled on the account if you use partial captures.

## Malaysian Account Notes (officially documented)

- MYR is supported as a payment and balance currency for in-country (Malaysian) PayPal accounts only; for accounts outside Malaysia PayPal converts MYR to the primary currency with a spread or fee (`/api/codes/currency`, `/v5/checkout/fx/currency-codes`).
- Malaysian users can send MYR only to other Malaysian users, cannot send payments that need exchange to or from MYR, and cannot send MYR balances to non-Malaysian users (`/payouts/supported-features`).
- API calls must not be used to provide personal payments in Malaysia (`/api/rest/reference/policies-and-guidelines`).
- Expanded Checkout (advanced card fields, card vaulting, 3D Secure controls) lists 37 countries; Malaysia is not one of them (`/expanded/eligibility`).
- Pay Later, PayPal Credit and Venmo are not offered for Malaysia per the eligibility tables.
- Regulatory context (not from developer docs; from the MPG gateway record): PayPal Pte Ltd is the contracting entity in the Malaysian user agreement `https://www.paypal.com/my/legalhub/paypal/useragreement-full`.

## Sandbox

1. Developer Dashboard > Testing Tools > Sandbox Accounts. Default business and personal accounts are created for you.
2. Create a Business sandbox account with Country = Malaysia to match your live account, and Personal buyer accounts for the markets you target.
3. Optional: enable Negative Testing on the sandbox business account.
4. Use sandbox REST app credentials and `https://www.sandbox.paypal.com` logins.

## Go Live Checklist (PayPal's, adapted)

1. Business account verified; app registered on the Live tab.
2. Live Client ID and Secret in server config; `api-m.paypal.com` host; live client ID in the JS SDK.
3. Live webhook URL subscribed and live Webhook ID stored.
4. Seller preferences (payment receiving preferences, descriptor) match what you tested.
5. Test a small real transaction, confirm funds arrive in the PayPal balance, then refund it.
6. Follow the PayPal Application Policies and Developer Agreement (`https://www.paypal.com/webapps/mpp/ua/xdeveloper-full`).

## Pricing And Payouts

- Pricing: `https://www.paypal.com/my/business/paypal-business-fees` (Malaysia merchant fees page; check the "Last Updated" date). Do not hard-code rates.
- Funds arrive in the PayPal balance and are withdrawn to a bank; withdrawal fees and timings are on the fee page and in the account, not in the developer docs.
- Payouts fees: `/payouts/fees` and the merchant fee page.
