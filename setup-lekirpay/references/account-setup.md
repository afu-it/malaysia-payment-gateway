# LekirPay Account Setup

Use this reference when explaining LekirPay registration, credentials, sandbox vs live, payment methods, refunds, settlement, and pricing. Checked 2026-10-21.

## Source URLs

- `https://lekirpay.com/`
- `https://lekirpay.com/#pricing`
- `https://lekirpay.com/terms-and-condition/`
- `https://app.lekirpay.com/validate`
- `https://app.lekirpay.com/doc/flow`
- `https://app.lekirpay.com/doc/introtoken`
- `https://wordpress.org/plugins/lekirpay-givewp/`

## Who LekirPay Is

LekirPay is run by Lekir Tech (M) Sdn Bhd (1381659-X), Cyberjaya. The website describes it as a commoditised payment platform for Malaysian businesses with payment forms, custom fields, payment links, QR codes, WhatsApp and email notifications, recurring billing, payouts, a dashboard, and a PHP/REST API.

## Registration

1. Sign up at `https://app.lekirpay.com/validate` (live) with an email address; captcha required; optional referral ID. `https://app.lekirpay.com/register` redirects there.
2. Sandbox signup link per the plugin readme: `https://sandbox.lekirpay.com/`. On 2026-10-21 that host returned HTTP 500; ask LekirPay support for a working sandbox.
3. Complete merchant verification. The payment API pages state: "Your profile needs to be verified before using this APIs". The plugin readme also says "Make sure account is verified".
4. KYC: the terms define a Merchant Onboarding Form and KYC information requested by Lekir Tech. The exact document list is not published (checked 2026-10-21); expect business registration and bank account details. Ask LekirPay for the current list.

## Credentials

After registration you get three values (`/doc/flow`):

| Dashboard name | API field | Use |
|---|---|---|
| ID | `client_id` | Token request |
| Lekirkey | `client_secret` | Token request |
| Lekir-Signature key | (HMAC key) | `checksum` on `/api/payments/payment/secure` |

Where to find them:

- `/doc/flow`: top right menu, click the arrow down icon, click Account, open the API tab.
- Plugin readme: sign in, top right menu, click the store icon, click Profile, open the API tab.
- `/doc/introtoken` (hidden page): Profile > API.

The UI wording differs between pages; look for the API tab under the account/profile menu.

Group ID: create one with the Payment Group API or in the dashboard (plugin setting says "found in your Lekirpay Account Settings"). Optional.

## Environment Variables

Suggested names (the docs do not define any):

```text
LEKIRPAY_BASE_URL=https://app.lekirpay.com          # or the sandbox host
LEKIRPAY_CLIENT_ID=<CLIENT_ID>
LEKIRPAY_CLIENT_SECRET=<LEKIRKEY>
LEKIRPAY_SIGNATURE_KEY=<LEKIR_SIGNATURE_KEY>
LEKIRPAY_GROUP_ID=<OPTIONAL_GROUP_ID>
LEKIRPAY_CALLBACK_URL=https://yourapp.example/payments/lekirpay/callback
LEKIRPAY_REDIRECT_URL=https://yourapp.example/payments/lekirpay/return
```

Keep separate values for sandbox and live. Never ship them to the browser.

## Payment Methods And Plans

From the website pricing section (do not copy numbers into code or docs; link `https://lekirpay.com/#pricing` because rates change):

- Start Up plan: FPX, MYR only, payment form, custom field, payment link, QR code.
- DuitNow Online Banking (DOBW): marked "coming soon".
- Enterprises plan: adds credit card acceptance; has a sign-up fee and annual fee.
- All plans state "MYR Only".

Card payments therefore need the Enterprises plan. Confirm the plan on the merchant account before offering cards.

The `payment_method` field in the callback tells you what the payer used. A list of method codes is not documented.

## Settlement And Payout

- Website: "Same-Day Settlement (T+0)" for FPX; card settlement listed as "Bi-Weekly".
- Terms: settlement goes to the Merchant Bank Account per the Settlement Cycle Schedule in the onboarding form, less deductions. Changing the bank account needs 30 days' written notice.
- No settlement report API is documented.

## Refunds And Chargebacks

- No refund API. Terms section 7: refunds are debited from the merchant account; an authorised person requests them through LekirPay's system or by letter on company letterhead with signature.
- Chargeback/dispute investigation is described in the terms; plan for manual handling.

## Plugins

- GiveWP: `lekirpay-givewp` on WordPress.org by `lekirtech`, version 3.1.2, last updated 2021-01-15, tested to WordPress 5.5. Settings: Client ID, LekirKey, Lekir-signature Key, Group ID, Server End Point (enabled = live `app.lekirpay.com`, disabled = sandbox `matkilau.lekirpay.com`).
- No official WooCommerce or other ecommerce plugin found on WordPress.org (checked 2026-10-21).

## Support

- Contact form: `https://lekir.tech/ask-us-now/` (linked from the website).
- Ticket desk (plugin readme): `https://lekirpay.freshdesk.com/support/login`.
- The FAQ link in the plugin readme (`https://www.lekirpay.com/faq-getting-started/`) returned an error on 2026-10-21.

## Pre-Launch Checklist

1. Live merchant profile verified.
2. Correct plan for the methods you offer (cards need Enterprises).
3. Live Client ID, Lekirkey and Lekir-Signature key stored as secrets.
4. Callback URL is public HTTPS and returns 200.
5. Status requery wired into callback and redirect handlers.
6. One real low-value live payment tested end to end, including a status requery.
