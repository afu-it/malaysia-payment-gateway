# GoMobi (Mobi) Account Setup

Use this for onboarding, credentials, sandbox vs live, env vars, IP whitelisting, settlement, plugins and pricing. Checked 2026-10-21.

## Source URLs

- `https://gomobi.io/documents/getting-started`
- `https://gomobi.io/documents/authentication`
- `https://gomobi.io/documents/sandbox-environments`
- `https://gomobi.io/documents/support/whitelist-ip`
- `https://gomobi.io/documents/support/faq`
- `https://gomobi.io/documents/support/contact`
- `https://gomobi.io/documents/checkout/malaysia/authorization`
- `https://gomobi.io/documents/payin/malaysia/plugins/shopify` (and woocommerce, prestashop, opencart, magento)
- `https://www.mobi.xyz/product`, `https://www.mobi.xyz/contact`
- `https://gomobi.io/portal/login`

## Company and Product

- Brand: Mobi (`mobi.xyz`), legal entity Mobi Asia Sdn Bhd, formerly Mobiversa Sdn Bhd. `gomobi.io` now redirects to `mobi.xyz` except `/documents`, `/documents_v2` and `/portal`.
- Malaysia methods in the docs: FPX (B2C, B2B), cards (Visa, Mastercard, UnionPay), Boost, GrabPay, ShopeePay, Touch 'n Go, DuitNow QR, 7-Eleven cash, payouts.
- Docs also cover Indonesia and India (out of scope for this skill).
- Malaysia office (contact page): 17-106, Equatorial Plaza, Jalan Sultan Ismail, Kuala Lumpur 50250.

## Registration

Self-serve sign-up is Not documented in official docs (checked 2026-10-21). Onboarding is sales-led:

1. Submit the "Talk to the Team" form at `https://www.mobi.xyz/contact` (Sales enquiry) or use `https://gomobi.io/documents/support/contact`.
2. Complete merchant onboarding with Mobi (KYB document list Not documented in official docs, checked 2026-10-21).
3. Receive staging credentials for integration and sandbox testing.
4. Pass testing; Mobi onboards the merchant to production and issues live credentials (the checkout API secret is issued only after production onboarding).
5. Log in to the merchant portal at `https://gomobi.io/portal/login` for transactions, balance and payout reports.

## Credentials and Env Vars

Names below are suggestions; values come from Mobi. Use separate values per environment.

| Env var | Credential | Used by |
|---|---|---|
| `MOBI_ENV` | `staging` or `production` | host selection |
| `MOBI_API_KEY` | mobiApiKey / motoApiKey | all direct APIs, AES password for wallets/DuitNow/cash |
| `MOBI_LOGIN_ID` | Merchant Login ID | FPX status, cards, plugins |
| `MOBI_SECRET_KEY` | mobiSecretKey | 3DES `mobiApiKey` header, IPN decryption |
| `MOBI_SALT` | AES salt | AES-CBC PBKDF2 |
| `MOBI_FPX_PARAM1` / `MOBI_FPX_PARAM2` | FPX AES password/salt (emailed by Mobi) | FPX checkSum |
| `MOBI_FPX_MID` / `MOBI_FPX_TID` | FPX merchant/terminal ids | FPX deposit |
| `MOBI_SUB_MID` | sub-merchant id | when sub-accounts are used |
| `MOBI_CHECKOUT_API_KEY` / `MOBI_CHECKOUT_API_SECRET` | hosted checkout | Bearer token and HMAC |
| `MOBI_IPN_URL` | your IPN endpoint | registered with Mobi |

Rules: server-side only, never in frontend bundles, never commit. Do not reuse the sample keys shown in the docs.

## Sandbox vs Live

- Staging hosts are mostly `m-uat.gomobi.io`, plus `checkout.gomobi.io` (checkout) and `sandbox-cash.gomobi.io` (cash). Production hosts: see `gomobi-api.md` Environments.
- Sandbox errors are identical to production.
- Sandbox only test data:
  - FPX: choose Retail, bank "SBI Bank A", user id `1234`, password `1234`.
  - Card: `5452 0100 0009 5323`, CVV `123`, expiry `12/28`.
- Sandbox test data for e-wallets, DuitNow QR and cash is Not documented in official docs (checked 2026-10-21).

## IPN URL and IP Whitelisting

- FPX IPN URL is static, set during onboarding or in the portal. E-wallets take a `callback`/`merchantCallback` per request. POST-format IPNs for FPX and wallets need activation on request.
- Inbound: get Mobi's IPN IP list from `csmobi@gomobi.io` and allow it on your firewall; Mobi does not publish it.
- Outbound: payout merchants submit their server IPs to Mobi. Non-whitelisted requests are silently dropped. DuitNow QR production is also IP-restricted.

## Activation

Each method (FPX, cards incl. international cards, each e-wallet, DuitNow QR, EZYCASH, payout, settlement payout) is enabled per merchant and sub-merchant. Errors such as "Access Denied: Payment not enabled", "Duitnow QR not enabled", "EZYCash not enabled", "Foreign Card Restricted" and "Settlement via payout is not enabled" mean Mobi must enable it.

## Settlement and Operations

- Settlement timing follows the merchant agreement; no settlement on weekends or public holidays. Mobi marketing states same-day settlement (`mobi.xyz/product`); confirm in your agreement.
- Payment channels run 24/7 including weekends and public holidays; bank maintenance may affect only that bank.
- Payout systems run 24/7.
- Daily reconciliation report is emailed on business days before 13:00 MYT.
- Balance: portal menu (top left) > "Balance", real time.
- Finance team: business days 11:00 to 20:00 MYT, not on weekends or public holidays.
- Callbacks cannot be resent manually. For pending transactions wait 5 to 10 minutes after expiry before contacting support.
- Refunds: no API; FPX refunds are manual through Mobi.

## Pricing

Not published. Pricing is by agreement: contact sales via `https://www.mobi.xyz/contact`. Do not quote fees.

## Plugins

| Platform | Install | Settings |
|---|---|---|
| Shopify | "Add App" from the Mobi page on the Shopify App Store, authorize, then enter credentials on the Mobi dashboard | Merchant Login ID, API key, Sandbox/Live mode |
| WooCommerce | Plugins > Add New > Upload Plugin; put `gomobi-thankyou.php` in `wp-content/themes/yourtheme/` and create a page `gomobi-thankyou` using it | Merchant Login ID, Mobi API Key, Callback URL (that page), mode SandBox/Live |
| PrestaShop | Modules > Module Manager > Upload a Module (`ezyway.zip`), then Configure | Merchant Login ID, key, Sandbox/Live mode |
| OpenCart | upload `mobi.ocmod.zip` (v2.0 or v3.0) in Extension Installer, refresh Extensions > Modifications, install under Extensions > Extensions > Payments | credentials from Mobi |
| Magento 2.4.2 | copy the `Mobi.zip` app folder into the Magento root, then run the docs' commands: `magento:enable Mobi_mobi`, `setup:di:compile`, `setup:static-content:deploy`, `cache:clean` (standard Magento syntax is `bin/magento module:enable Mobi_mobi`); configure in Stores > Configuration > Sales > Payment Methods | server field `test` for sandbox, `live` with live credentials, debug log `No` in live |

Plugin download links are provided on the plugin pages or by Mobi; verify the source before installing.

## Support

- `csmobi@gomobi.io`: customer support, IPN IP list, IPN failures.
- `getsupport@gomobi.io`: hosted checkout API secret.
- Support form: `https://gomobi.io/documents/support/contact`.
