# senangPay Account Setup

Use this reference when explaining senangPay registration, dashboard setup, sandbox setup, credentials, package capabilities, pricing caveats, or support requirements.

## Source URLs

- `https://senangpay.com/pricing/`
- `https://senangpay.com/social-selling/`
- `https://senangpay.com/online-store/`
- `https://senangpay.com/enterprise-solutions/`
- `https://guide.senangpay.com/getting-started`
- `https://guide.senangpay.com/account-and-subscriptions`
- `https://guide.senangpay.com/testing-sandbox`
- `https://guide.senangpay.com/merchant-id-and-secret-key`
- `https://guide.senangpay.com/manual-integration-api`
- `https://guide.senangpay.com/direct-api`
- `https://guide.senangpay.com/payments-settlements-and-charges`
- `https://guide.senangpay.com/frauds-and-disputes`
- `https://guide.senangpay.com/rules-and-regulations`
- `https://guide.senangpay.com/senangpay-to-doku-platform-migration`

## Registration and Dashboard

senangPay merchant registration starts from the senangPay website and dashboard. The public guide groups onboarding under Getting Started:

- Merchant account registration.
- Requirements or restrictions.
- Approval timing.
- Required document submission.
- Dashboard login.
- Website policy templates.

Do not guess merchant eligibility, approval time, or required documents for a specific business. Use the current guide/dashboard/support when those details matter.

## Sandbox

senangPay provides sandbox testing for webstore or app development.

Useful URLs:

- `https://guide.senangpay.com/testing-sandbox`
- `http://sandbox.senangpay.my`
- Hosted sandbox checkout: `https://sandbox.senangpay.my/payment/{merchant_id}`
- Direct API sandbox: `https://api.sandbox.senangpay.my/payment/v3`
- FPX bank list sandbox: `https://api.sandbox.senangpay.my/fpx_bank_list`
- Refund API sandbox base: `https://api.sandbox.senangpay.my`

Sandbox and production credentials must be separated. Do not reuse production Merchant ID, Secret Key, callback URLs, or Return URLs in sandbox unless the dashboard explicitly says to.

## Credentials

For manual/hosted and plugin integrations:

1. Log in to senangPay dashboard.
2. Go to `Settings > Profile`.
3. Find `Shopping Cart Integration Link`.
4. Copy `Merchant ID`.
5. Copy `Secret Key`.
6. Select Hash Type.
7. Configure Return URL.
8. Configure Callback URL.
9. Save changes.

Secret Key can be regenerated. Regenerating it can break existing deployed integrations until environment variables and webhook verification code are updated.

## Hash Type

senangPay dashboard supports Hash Type selection.

- SHA256 is recommended.
- MD5 is not recommended, is older, and is described as vulnerable/deprecated.
- Always choose SHA256 unless a system provider explicitly confirms otherwise.

Agents should state which Hash Type their code expects and document it in environment setup.

## Required Environment Variables

Recommended app environment variables:

```text
SENANGPAY_ENV=sandbox
SENANGPAY_MERCHANT_ID=...
SENANGPAY_SECRET_KEY=...
SENANGPAY_HASH_TYPE=sha256
SENANGPAY_RETURN_URL=https://example.com/payments/senangpay/return
SENANGPAY_CALLBACK_URL=https://example.com/api/payments/senangpay/callback
```

For Direct API:

```text
SENANGPAY_DIRECT_API_ENABLED=false
SENANGPAY_API_BASE_URL=https://api.sandbox.senangpay.my
```

Do not expose `SENANGPAY_SECRET_KEY` to browser or mobile clients.

## Return URL and Callback URL Setup

Return URL is where senangPay redirects the buyer after payment processing. Use it for customer experience.

Callback URL is a backend notification URL for shopping cart integrations. It is recommended for data integrity and should be used for settlement.

Callback requirements:

- Must handle HTTP POST.
- Must not require a user session.
- Must avoid redirects.
- Must return exactly `OK` without HTML tags after successful durable processing.
- Non-secure HTTP ports may require whitelisting from senangPay support.

## Packages and Capabilities

Public pages describe package tiers and feature groups such as Starter, Advance, and Enterprise. Pricing, fees, and package names can change, especially during promotions.

Source-observed examples from public pricing pages include:

- Starter and Advance annual packages for small businesses.
- Enterprise custom package with custom rates and advanced features.
- FPX, cards, e-wallets, BNPL, payment forms, dashboard, plugins, e-invoice/quotation, API integration, recurring, payout, tokenisation, split settlement, faster settlement, and foreign cards.

Do not hardcode current pricing in app docs unless the user explicitly asks and you verify the live pricing page. Prefer wording such as "verify current pricing and package capabilities in the senangPay pricing page/dashboard."

## Direct API Access

Direct API is available only for Enterprise package users and restricted to authorised merchants. Merchants must contact senangPay and whitelist their server outgoing IP address.

Do not build Direct API as the default integration unless the user confirms:

- Enterprise package or approval.
- Direct API authorisation.
- Server outgoing IP whitelisting.
- Required payment methods.

## Product Selection Guidance

Choose by merchant need:

- Social seller without website: senangPay payment forms.
- Standard custom web app: hosted/manual payment checkout with callback and query verification.
- WooCommerce/OpenCart/Magento/etc.: use official plugin/platform guide first.
- Seamless/advanced checkout: Direct API only if Enterprise and authorised.
- Subscription/installment: recurring payment references and package capability checks.
- Marketplace or multi-party settlement: split settlement and Enterprise support checks.
- Refund automation: Refund API with local idempotency and approval workflow awareness.
- Vendor/customer payouts: Payout API and Enterprise support checks.

## Support and Operational Caveats

Use senangPay support or dashboard for:

- Package upgrades/downgrades.
- Payment method/channel enablement.
- Direct API approval and IP whitelisting.
- Non-secure callback port whitelisting.
- Current pricing and transaction rates.
- Settlement timing and payout schedules.
- DOKU platform migration behavior.
- Fraud/dispute handling and prohibited items.
- Required documents or business restrictions.

## Production Readiness Checklist

- Merchant ID and Secret Key configured server-side.
- Hash Type is SHA256 unless legacy integration requires MD5.
- Return URL and Callback URL set in dashboard.
- Public HTTPS callback URL deployed.
- Callback handler returns exact `OK` only after durable event recording.
- Sandbox and production settings separated.
- Local pending payment created before checkout redirect/session.
- Duplicate callbacks are harmless.
- Failed-then-success callback transitions are handled.
- Query status path exists for reconciliation.
- Secret Key regeneration process is documented.
- Package-specific features are verified before launch.
