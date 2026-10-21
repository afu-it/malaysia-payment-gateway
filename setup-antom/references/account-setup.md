# Antom Account Setup

Onboarding, keys, sandbox, go-live, and settlement for a merchant using Antom. Checked against official docs on 2026-10-21.

## Source URLs

- Get started (Dashboard): `https://docs.antom.com/ac/merchant_service/antom_dashboard.md`
- Merchant onboarding: `https://docs.antom.com/ac/merchant_service/merchant_onboard.md`
- Enable payment methods: `https://docs.antom.com/ac/merchant_service/enable_payment.md`
- API key configuration: `https://docs.antom.com/ac/merchant_service/key_config.md`
- Notification URL: `https://docs.antom.com/ac/merchant_service/notification.md`
- Integration guide (checklist): `https://docs.antom.com/integration_guide_en.md`
- Sandbox: `https://docs.antom.com/ac/ref/sandbox.md`
- Test wallet: `https://docs.antom.com/ac/ref/testwallet.md`
- Settlement: `https://docs.antom.com/ac/merchant_service/settlements.md`
- Antom Dashboard: `https://dashboard.antom.com` (docs also link `https://dashboard.alipay.com/global-payments/home`)
- Contact us: `https://www.antom.com/contact-us/`

## Product Positioning

- Antom is Ant International's merchant payment service. One integration covers cards, digital wallets (including Alipay+ wallets), online banking, and more.
- For Malaysia the docs list FPX, Touch'n Go eWallet, Boost, GrabPay MY, ShopeePay MY, and cards. See `antom-codes.md` for limits and acquirer entities.
- Plugins: Shopify and WooCommerce (per method support differs; see `antom-codes.md`).

## Account Setup Checklist

1. Request access through the Antom Contact us form. The docs say you are notified within 1 day and then receive an invitation email.
2. Register an Antom Dashboard account from the invitation, then click **Start** on the Home page to register a merchant account (business registration number required; review takes up to 1 business day).
3. After activation, enable payment methods in Live mode at **Payments > Payment methods** (single or batch enablement). Contact Antom Business Development for payment method setup and settlement account binding.
4. Generate keys at **Developer > Key configuration > Generate key** (or **Developer > Quick start > Integration resources and tools > Integration resources**). Copy the private key at once: it is shown only once and Antom does not store it.
5. Get the gateway URL, client ID, and Antom public key from **Developer > Quick start > Integration resources and tools > Integration resources**.
6. Set the notification URL at **Developer > Notification URL** (Configuration tab). Test it from the Test tab.
7. Link a bank account at **Finance > Settlement overview > Bank account > + Add a new bank account**.

Up to 20 merchant accounts can be created under one Dashboard login.

## Credentials

| Item | Where | Store as |
| --- | --- | --- |
| Client ID | Integration resources | `<CLIENT_ID>` (sandbox starts with `SANDBOX_`) |
| Merchant private key | Generated once in Key configuration | `<MERCHANT_PRIVATE_KEY>` (server secret) |
| Merchant public key | Uploaded or generated in Dashboard | Not secret |
| Antom public key | Integration resources | `<ANTOM_PUBLIC_KEY>` |
| Gateway domain | Integration resources | `<ANTOM_DOMAIN>` |

- Test mode and live mode need different keys and client IDs.
- Replace keys with **Replace keys** (one click, or upload your own public key generated with Antom's offline tool for Windows or macOS). Update your private key everywhere after replacing.
- If the private key is lost, replace the keys.

## Sandbox

1. In the Dashboard, switch to Test mode and copy the sandbox client ID, domain, and public key from Integration resources.
2. Call the sandbox path `/ams/sandbox/api/...` (see `antom-api.md`).
3. Install the test wallet: Android download link on the test wallet page; iOS via TestFlight (`IAP_Wallet+`). Log in with your Antom Dashboard account. It logs out after about 30 minutes idle.
4. Test cards: see `https://docs.antom.com/ac/ref/card.md`. Any CVC, postal code, expiry, and first two digits of the payment password are accepted. Amount values `11`, `1100`, or `1111` simulate a risk-control failure. `is3DSAuthentication` `true` triggers 3DS.
5. Online banking and other methods: on a login page, enter any value; click a QR code to pay.
6. Dashboard tools: API call simulation, Notification URL test, Error scenario simulation, Test case, Request log, Notification log.

- Test wallet methods include Boost and Touch'n Go eWallet.
- FPX is not in the test wallet list. The "How to test other payment methods" page covers online banking via login page or redirect.
- The sandbox accelerates subscription cycles (for example, a 31-day period bills every 3 minutes during acceleration).

## Go Live

1. Make sure the merchant account is activated and switch the Dashboard to Live mode.
2. Generate live keys and copy the live gateway URL and client ID.
3. Replace sandbox client ID, keys, and path (`/ams/sandbox/api` to `/ams/api`).
4. Set the live notification URL.
5. Run a live pilot payment, then tell Antom you are live so they can monitor.

## Settlement

- Settlement follows T+N rules set in your contract. View period, linked bank accounts, and history in **Finance** (Settlement overview, Balance) or **Balances > Settlement** (Sales, Settlements, Settlement plan, Calendar).
- Settlement statuses: Pending, To-be-settled, Ready to settle, Settled, Settlement failure or return.
- `settlementStrategy.settlementCurrency` is required when you signed several settlement currencies.
- Sandbox SFTP reports are usually issued around 5 PM on T+1.
- The settlement period for Malaysia is not documented in official docs (checked 2026-10-21). Confirm with Antom.

## Pricing

- No public pricing page was found (checked 2026-10-21; `https://www.antom.com/pricing` returns 404).
- The docs say fees and fixed processing fees per method are shown in Antom Dashboard > **Payments > Payment methods > Price**.
- For a quote, use `https://www.antom.com/contact-us/`. Do not copy numbers from third parties.

## Support And Tooling

- Antom Integration Skill: `https://docs.antom.com/ac/ref/skill.md`.
- Antom MCP server (allowlisted merchants, OAuth): `https://docs.antom.com/ac/ref/mcp.md`. Request access at `pub_antom_integration_service@antom.com`.
