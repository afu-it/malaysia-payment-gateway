# SecurePay Account Setup

Use this reference for SecurePay registration, documents, credentials, sandbox vs live, interface settings, settlement, refunds, plugins and support. Checked 2026-10-21.

## Source URLs

- `https://www.securepay.my/docs/`
- `https://docs.securepay.my/master`
- `https://docs.securepay.my/api/introduction`
- `https://docs.securepay.my/guides/create-account`
- `https://docs.securepay.my/guides/settlement`
- `https://v2.docs.securepay.my/` (SecurePay Console docs)
- `https://securepay-4.gitbook.io/console-docs/getting-started/quickstart/account-registration`
- `https://securepay-4.gitbook.io/console-docs/getting-started/quickstart/supporting-documents`
- `https://securepay-4.gitbook.io/console-docs/basics/interfaces`
- `https://securepay-4.gitbook.io/console-docs/basics/settlements`
- `https://www.securepay.my/settlement-schedule/`
- `https://www.securepay.my/refund/`
- `https://wordpress.org/plugins/securepay/`

## Company and Product

- SecurePay Sdn Bhd, Malaysia. Site: `https://www.securepay.my`. Describes itself as a "Secure marketplace & payment platform for Malaysian businesses. FPX, cards, e-wallets, escrow and real-time reporting."
- Products listed on the site: Payment Platform, Payment Collection, SecurePay Amanah (escrow).
- API v1 covers FPX online banking (B2C and B2B1) and FPX credit card. API v2 adds DuitNow, FPX direct debit eMandate, closed wallet, memberships, split settlement.
- No-code: Payment Forms (Forms > Collections > New) with redirect and callback URLs; premium short domains (bayar.my, jual.my and similar, per the WooCommerce plugin readme).

## Registration

v1 dashboard (SecurePay Apps):

- Sign up: `https://securepay.my/apps/users/sign_up`
- Sign in: `https://securepay.my/apps/users/sign_in`
- Video tutorial (Bahasa Malaysia): `https://youtu.be/ny0LSeiAtEc` (Create Account guide)

v2 (SecurePay Console) registration steps from the Account Registration page:

1. Go to `https://console.securepay.my/users/sign_up` (observed redirecting to the SecurePay Apps sign up on 2026-10-21).
2. Provide full name or company name, email, contact number, other required info.
3. Choose a username and a strong password (minimum 8 characters with uppercase, lowercase, number and special character).
4. Verify your email via the confirmation link.
5. Complete profile: billing information, business registration details; accept Terms and Privacy Policy.
6. Submit for review. SecurePay notifies you on approval; then generate API keys for testing and production.

## Supporting Documents (Starter package)

- Soft copy of the company's SSM certification.
- Soft copy of the director's identification card.
- Soft copy of the latest bank statement.
- Maximum 2MB per document. Other packages or payment providers may require more checks.

## Credentials

### v1 (SecurePay Apps > Interfaces)

Each interface (API) has an API UID, Auth Token and Checksum Token. Sandbox interfaces: `https://sandbox.securepay.my/apps/interfaces`. Production interfaces are in the production dashboard. Multiple credential sets are allowed; each has its own name (returned as `api_name` / `interface_name`).

Also on the API settings page: B2C or B2B1 model and other defaults.

Suggested env vars:

```text
SECUREPAY_ENV=sandbox
SECUREPAY_BASE_URL=https://sandbox.securepay.my/api
SECUREPAY_UID=<API_UID>
SECUREPAY_AUTH_TOKEN=<AUTH_TOKEN>
SECUREPAY_CHECKSUM_TOKEN=<CHECKSUM_TOKEN>
SECUREPAY_PARTNER_UID=<PARTNER_UID>   # partners only
SECUREPAY_CALLBACK_URL=https://example.com/payments/securepay/callback
SECUREPAY_REDIRECT_URL=https://example.com/payments/securepay/return
```

### v2 (SecurePay Console > Interfaces)

Client ID and Client Secret per interface. Partners get separate partner Token and Secret. Optional RSA/PQC keys per interface (Certificate page).

```text
SECUREPAY_V2_BASE_URL=https://sandbox.securepay.dev/api
SECUREPAY_CLIENT_ID=<CLIENT_ID>
SECUREPAY_CLIENT_SECRET=<CLIENT_SECRET>
```

## Sandbox vs Production

| | v1 | v2 |
| --- | --- | --- |
| Sandbox API | `https://sandbox.securepay.my/api` | `https://sandbox.securepay.dev/api` |
| Production API | `https://securepay.my/api` | `https://console.securepay.my/api` |

Sandbox and production use different credentials. Test in sandbox first, then switch credentials and base URL together. Production requires a verified account and approved documents.

## Interface Configuration (v2 Console)

- Status must be Active to process payments.
- Charge Mode: Merchant (fee deducted from amount), Customer (customer pays fee), Credit (fee deducted from prepaid Credit Account). FPX and DuitNow can override with Inherit.
- URL settings: Redirect, Cancel, Callback, Timeout. Per-request URLs can also be sent in the API call.
- Gateways: FPX (bank type B2C or B2B), DuitNow (participating bank, typically Retail).
- Virtual account bank and optional split settlement rule.
- B2B1 (corporate FPX) must be enabled for the merchant ("Merchant need to enable this module").

## Payment Method Activation

Enabled methods for a v2 interface are returned by `GET /v1/interface/gateways`. For v1, FPX B2C is the default; B2B1 needs enabling. Card, e-wallet and DuitNow availability on a given account is not documented per account; confirm with SecurePay.

## Pricing

Use SecurePay's own pages and sales team: `https://www.securepay.my` and `https://www.securepay.my/contact/`. Do not quote per-transaction rates from memory or third-party sites. The WooCommerce plugin readme says pricing is per transaction with no startup or monthly fees; confirm current terms with SecurePay.

## Settlement

- v1 guide: "At this moment the settlement period is T+1 (next business day)." Settlement statements can be downloaded from the dashboard settlement section (change log 25 July 2021).
- Schedule (`https://www.securepay.my/settlement-schedule/` and v2 Settlements page): payments before 11:59pm MYT cut-off are settled the next business day; Friday, Saturday and Sunday payments are initiated Monday; public holidays push it to the next working day.
- Initiation time differs between sources: the v2 docs say "by 11:00 AM", the website schedule says "Before 11:00pm". Treat timing as indicative.
- Split settlement and virtual accounts (Main, Credit, Default, Commission; Auto or Manual transfer) are configured in the Console. Negative settlement amounts are held.
- Credit Account must have sufficient balance when fees are charged to credit.
- Custom settlement arrangements: contact SecurePay.

## Refunds

No refund API. Per the Refund & Cancellation Policy: refunds only for technical errors (double debit, excess debit); apply to `support@securepay.my` with the transaction number and receipt; processed within 7 working days after verification, net of gateway charges; 3 to 7 days for funds to reach the bank account.

## Plugins (official, by SecurePay on WordPress.org)

- SecurePay For WooCommerce `https://wordpress.org/plugins/securepay/` (1.0.18, last updated 2023). Settings: Token, Checksum Token, UID, optional Partner UID, sandbox mode, test mode, bank list option. Note: it does not verify the callback checksum.
- SecurePay For GravityForms, WPForms, Fluent Forms, GiveWP, Paid Memberships Pro, WPJobster, Restrict Content Pro (all by the `securepay` WordPress.org profile).
- Verified v1 partners listed in docs: Shoppegram, Ordersini, Kiah Store. v2 partners: Didik Digital, e-Maik.

## Support

- Email: `support@securepay.my` (docs hub, v2 Supports page). Plugin readme also lists `hello@securepay.my` and `registry@securepay.my` (account creation).
- WhatsApp: `wa.me/60174007979`, Monday to Friday 9am to 6pm MYT. Weekends and holidays: email only, 24 to 48 hour response.
- When reporting: account ID or email, steps to reproduce, screenshots or errors, expected vs actual.
