# Airwallex Account Setup (Malaysia)

Use this when explaining registration, credentials, sandbox vs live, payment method activation, settlement, and pricing. Checked 2026-10-21.

## Source URLs

- Malaysia site: `https://www.airwallex.com/my`
- Malaysia pricing: `https://www.airwallex.com/my/pricing`
- Get started with payments: `https://www.airwallex.com/docs/payments/get-started/get-started-with-payments`
- Activate payment methods: `https://www.airwallex.com/docs/payments/get-started/payment-method-activation`
- Payment method onboarding requirements: `https://www.airwallex.com/docs/payments/payment-methods/payment-method-onboarding-requirement`
- Manage API keys: `https://www.airwallex.com/docs/developer-tools/api/manage-api-keys`
- API key best practices: `https://www.airwallex.com/docs/developer-tools/api/api-key-best-practices`
- API key scopes: `https://www.airwallex.com/docs/developer-tools/api/api-key-scopes`
- IP whitelisting: `https://www.airwallex.com/docs/developer-tools/api/ip-whitelisting`
- Sandbox: `https://www.airwallex.com/docs/developer-tools/sandbox-environment`
- Settlements: `https://www.airwallex.com/docs/payments/about-airwallex-payments/settle-into-your-airwallex-balance`
- Reserve plan: `https://www.airwallex.com/docs/payments/about-airwallex-payments/reserve-plan`
- Fees and pricing models: `https://www.airwallex.com/docs/payments/about-airwallex-payments/fees-and-pricing-models`
- Plugins: `https://www.airwallex.com/docs/payments/integration-options/web-checkout/plugins`
- WooCommerce plugin: `https://www.airwallex.com/docs/payments/integration-options/web-checkout/plugins/built-by-airwallex/woocommerce/install-the-woocommerce-plugin`
- AI tools (MCP, CLI): `https://www.airwallex.com/docs/developer-tools/ai/developer-connector`, `https://www.airwallex.com/docs/developer-tools/ai/agentos`, `https://www.airwallex.com/docs/developer-tools/cli`

## Product positioning

Airwallex is a global financial platform. Malaysian businesses contract with Airwallex (Malaysia) Sdn. Bhd. Online payments settle into a multi-currency Airwallex Wallet, from which funds can be paid out, converted, or spent. For acceptance it offers cards, Apple Pay, Google Pay, FPX, DuitNow, Touch 'n Go, Boost, GrabPay, ShopeePay, Atome and more (see `airwallex-malaysia-methods.md`).

## Registration and activation (production)

1. Create an Airwallex business account at `https://www.airwallex.com/my` and complete business verification (KYB).
2. In the web app go to **Payments > Get started**: business details, payment solution, payment methods. Auto-approval is possible; otherwise review usually takes 1 to 2 business days and may raise an RFI answered in the web app.
3. Prepare the website first: business name, registration number, address, email, phone; content matching your industry; Terms and Conditions, shipping and refund, privacy policies; a checkout with clear prices; no redirects to external sites; governing law matching your registered country (Visa and Mastercard).
4. Track each method in **Payments > Payment methods > My payment methods**: Not activated, Under review, Updates required, Information required, Ineligible. Request more under **Other available payment methods**. Activation times differ (FPX 15-20 business days, DuitNow/TNG/Boost 7-14, GrabPay/ShopeePay 5-10, cards instant).
5. Configure settlement currencies (principal, for example MYR, and a fallback) in **Wallet > Balances**.
6. Review your reserve plan and settlement timings under **Payments > Settings**.
7. PCI: HPP, Drop-in and Elements need PCI-DSS SAQ A for cards. Native API card handling needs PCI-DSS ROC and Airwallex approval. Local methods have no PCI requirement.
8. Apple Pay / Google Pay: enable in **Payments > Settings**.

Documents: the docs describe business details, KYB, and RFIs but do not publish a fixed document checklist for Malaysia. Not documented in official docs (checked 2026-10-21); follow the web app prompts.

## Credentials

| Item | Where | Notes |
| --- | --- | --- |
| Client ID | **Settings > Developer > API keys** | Same for a key across regenerations |
| API key (scoped recommended) | **Settings > Developer > API keys > New scoped API key** | Shown once. Admin keys are discouraged for new integrations |
| Account ID (for `x-login-as`, Checkout Sessions `account_id`) | **Settings > Account details** | Not the organization ID |
| Webhook secret | **Developer > Webhooks** (per notification URL) | Or returned as `secret` by `POST /api/v1/webhooks/create` |
| IP whitelist for a key | **Settings > Developer > API keys > (key) > IP whitelist** | IPv4, `/24` to `/32` |

The Developer app needs the Owner, Admin, or Developer role. Scoped keys for payments use account-level resources such as "Payment Acceptance", "Payment Disputes", "Payment Links", "Funds Splits", "POS Terminals" with Read or Write. A scoped key on several accounts needs `x-login-as` at login.

Suggested environment variables (names are this skill's convention, not Airwallex's):

```bash
AIRWALLEX_ENV=sandbox                    # or production
AIRWALLEX_BASE_URL=https://api.sandbox.airwallex.com
AIRWALLEX_CLIENT_ID=<CLIENT_ID>
AIRWALLEX_API_KEY=<API_KEY>
AIRWALLEX_ACCOUNT_ID=<ACCOUNT_ID>        # only if x-login-as or Checkout Sessions need it
AIRWALLEX_WEBHOOK_SECRET=<WEBHOOK_SECRET>
AIRWALLEX_RETURN_URL=https://shop.example/payments/airwallex/return
AIRWALLEX_WEBHOOK_URL=https://shop.example/webhooks/airwallex
```

## Sandbox vs production

| | Sandbox | Production |
| --- | --- | --- |
| Signup | `https://www.sandbox.airwallex.com/global/signup` | `https://www.airwallex.com/my` |
| Web app | `https://www.sandbox.airwallex.com/app/` | `https://www.airwallex.com/app/` |
| API | `https://api.sandbox.airwallex.com` | `https://api.airwallex.com` |
| Keys, webhook secrets | Separate | Separate |
| Webhook source IPs | 6 sandbox IPs | 25 production IPs |
| Payment methods | Cards and a few wallets by default; request MY methods | As activated after review |

Sandbox onboarding is auto-approved. Extra sandbox accounts must use the entity name "New Business Sandbox" or "Sandbox Business".

## Settlement and payouts

- Funds settle to the Airwallex Wallet per method schedule (for example FPX T+3, DuitNow T+2, TNG T+1, cards 3 business days after capture for MY entities) and your reserve plan, whichever is later.
- Only MY-registered businesses settle Boost, GrabPay, ShopeePay in MYR.
- Small settlements under 100 USD equivalent may be held until the threshold for some methods.
- Payout from the Wallet to a Malaysian bank account uses Airwallex Payouts (out of scope for this skill; see the endpoint catalogue).

## Pricing

Do not quote numbers from memory. Link the official pages: `https://www.airwallex.com/my/pricing` and the fee model explanation at `https://www.airwallex.com/docs/payments/about-airwallex-payments/fees-and-pricing-models` (gateway fee, payment method fee, 3DS/fraud fees, FX fees; interchange++ or blended pricing per contract).

## Plugins and no-code

- Built by Airwallex: Shopify, WooCommerce, Magento, BigCommerce, PrestaShop, Salesforce Commerce Cloud, SHOPLINE. Partner orchestrators are also listed.
- WooCommerce: WordPress 4.5+, WooCommerce 3.0+, PHP 7.3+. Connect by Airwallex login (OAuth) or by API keys; with API keys you must also create a webhook and paste its secret into the plugin's Webhook Secret field. Activate methods in Airwallex before enabling them in the plugin.
- Payment Links: web app or API, no code.

## AI tooling published by Airwallex

- Developer MCP (docs and SDK guidance): `https://mcp.sandbox.airwallex.com/developer`.
- AgentOS MCP (production account actions, OAuth): `https://mcp.airwallex.com/mcp`.
- Airwallex CLI for production reads and writes (see the CLI page).
