# Touch 'n Go eWallet Account Setup

Use this reference for TNGD merchant registration, partner onboarding for API access, credentials, sandbox vs production, settlement, and pricing.

## Source URLs

- Merchant registration: `https://www.tngdigital.com.my/business/merchant`
- Merchant help centre: `https://merchantsupport.tngdigital.com.my/hc/en-us`
- MDR article: `https://merchantsupport.tngdigital.com.my/hc/en-us/articles/15599821031321-How-much-is-the-MDR`
- Payment Link article: `https://merchantsupport.tngdigital.com.my/hc/en-us/articles/360061912153-Payment-Link`
- Daily settlement report: `https://merchantsupport.tngdigital.com.my/hc/en-us/articles/360057944133-How-do-I-view-daily-settlement-report-in-Merchant-Dashboard`
- Mini Program project planning: `https://miniprogram.tngdigital.com.my/docs/miniprogram_tngd/mpdev/project_planning`
- Apply for an account: `https://miniprogram.tngdigital.com.my/docs/miniprogram_tngd/mpdev/quick-start_create-account`
- Release a mini program: `https://miniprogram.tngdigital.com.my/docs/miniprogram_tngd/mpdev/quick-start_release-mini-program`
- Settlement process: `https://miniprogram.tngdigital.com.my/docs/miniprogram_tngd/mpdev/partner_settlement_process`
- WooCommerce plugin onboarding link: `https://tngdigitalbizhelp.zendesk.com/hc/en-my/requests/new?ticket_form_id=59240529886361`

## Two Kinds Of Merchant

1. QR and Payment Link merchant. Self-serve: TNGD says registration takes about 30 minutes, needs no documents, and is approved instantly. You get the merchant dashboard, static QR, Payment Links (one-time and static), Soundbox devices, and a Business Account. No API keys.
2. API partner (mini program or web acquiring). Not self-serve. TNGD runs an onboarding project and issues credentials by email. Use this path for any server integration in this skill.

## API Partner Onboarding (Mini Program)

Milestones from `project_planning`:

1. Sandbox set up request (partner): create the mini program workspace, give the Mini Program ID (16 digits, shown in the console at `https://miniprogram.tngdigital.com.my/console/miniprogrammanage/index`).
2. Sandbox credential set up (TNGD): server access, Postman script for the Open APIs, development documents, TNGD and partner public keys.
3. Detailed solution (partner): UI/UX and integration flow, sequence diagrams.
4. Solution review and sign off (TNGD).
5. Listing preparation: mini program logo, name, ID, list of internal test users.
6. Sandbox submission: fully tested app in sandbox, plus test accounts for TNGD.
7. Production credential request (partner): your public key (used by TNGD to verify your response to the payment notification and future APIs) and the IP list for SFTP settlement access.
8. TNGD sandbox UAT: sign-in (`my.getAuthCode`, applyToken, user info), payment (pay, refund, notify response), UI/UX, performance report.
9. Code freeze.
10. Partner production credentials (TNGD, by email): Partner ID / Merchant ID, TNGD public key, SFTP user name and password, production API endpoints.
11. Support set up, joint pilot (whitelist) test, greyscale launch.

Developer accounts: the workspace admin invites developers by email. Only a Mini Program admin can create, submit for review, and publish a mini program (`quick-start_create-account`, `quick-start_release-mini-program`).

## Web Acquiring Onboarding

The WooCommerce plugin points new merchants to TNGD's onboarding request form (link above). Credentials it expects per environment: Client ID, Client Secret, Merchant ID, Product Code, Merchant RSA private key, MCC, and the API base URL. Sandbox base default `https://api-sd.tngdigital.com.my`; production base comes from TNGD.

## Keys

- Generate an RSA key pair (RSA256 signing). Keep the private key in a secret store. Send only the public key to TNGD.
- Store the TNGD public key per environment and per `keyVersion`.
- Sandbox and production keys, Client IDs, and hosts are different. Never mix them.

## Environment Variables

Use names that match the app. Suggested:

```text
TNG_ENV=sandbox
TNG_API_HOST=<host from TNGD>                 # Open API, used as https://<host>/acl/api/...
TNG_WEB_API_BASE=https://api-sd.tngdigital.com.my   # web acquiring sandbox (plugin default)
TNG_CLIENT_ID=<CLIENT_ID>
TNG_CLIENT_SECRET=<CLIENT_SECRET>             # web acquiring only
TNG_PARTNER_ID=<PARTNER_ID>                   # also called Merchant ID / MID
TNG_PRODUCT_CODE=51051000101000100001         # cashier payment (mini program); web value comes from TNGD
TNG_KEY_VERSION=<n>
TNG_PRIVATE_KEY=<PEM, secret>
TNG_PUBLIC_KEY=<TNGD PEM public key>
TNG_NOTIFY_URL=https://example.com/api/webhooks/tng
TNG_RETURN_URL=https://example.com/payments/tng/return
```

## Settlement

- T+1 to the bank account captured at onboarding (API partners). CSV on SFTP at 9 AM T+1; your IP must be whitelisted. Field list is in `tng-ewallet-api.md`.
- QR merchants: daily transaction reports are prepared at end of day and downloadable in the Merchant Dashboard. Settlement method can be changed in the dashboard.

## Pricing

Do not copy rates into code or docs. Link to TNGD's MDR article above and confirm the rate in the merchant agreement. The public MDR article covers static QR for SME merchants only; API partner pricing is set by contract (the `productCode` selects the contract).

## Sandbox Test Data

No public test accounts, test wallets, or test balances (checked 2026-10-21). TNGD issues them at sandbox setup, and sandbox mini programs run in the TNGD Partner Test App.
