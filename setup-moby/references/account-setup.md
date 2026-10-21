# MOBY Account Setup

Use this reference when explaining MOBY (MobyPay) merchant onboarding, credentials, sandbox vs live, dashboard features, payment methods, plugins, pricing and settlement caveats. Checked 2026-10-21.

## Source URLs

- `https://moby.my/`
- `https://moby.my/get-started/`
- `https://moby.my/contact-sales`
- `https://moby.my/pricing/`
- `https://moby.my/bnpl/`
- `https://docs.mobypay.my/docs/introduction`
- `https://docs.mobypay.my/docs/sandbox-testing`
- `https://docs.mobypay.my/docs/authentication`
- `https://docs.mobypay.my/docs/user-guide` and the `/docs/guide-*` pages
- `https://docs.mobypay.my/docs/support`
- `https://github.com/MobyPayTech/moby-plugins/tree/main/direct-integration` (Getting Your Credentials)

## Company And Product

- Operator: Moby Money Sdn. Bhd. (registration 201901032761 (1342091-H)), shown on moby.my.
- Product family "Moby Omni": online Checkout, Payment Link, POS Terminal, Pay Later (BNPL), bank instalment plans, Cashback loyalty, business financing.
- MOBY states all its BNPL products are shariah-compliant; there is also a "Moby Islamic" BNPL brand with its own Shopify app.
- Online methods named in the docs: cards (Visa/Mastercard via MPGS), FPX online banking, e-wallets (Touch 'n Go eWallet, GrabPay, Boost, ShopeePay), DuitNow QR, BNPL. Which methods your account has is returned by `POST /api/merchant/payment/method/list`.

## Onboarding

1. Start from `https://moby.my/get-started/` (book a demo) or `https://moby.my/contact-sales`.
2. MOBY onboards the merchant. Required KYC documents are not published (checked 2026-10-21); ask MOBY sales.
3. On completion MOBY issues a Client ID and Secret Key. Developer docs: "Contact merchantsupport@moby.my to get your Client ID and Secret Key."
4. Sandbox credentials are a separate pair, also requested from `merchantsupport@moby.my`.
5. Your merchant category matters: insurance merchants must send the customer NRIC on hosted checkout.
6. Extra API scopes (loyalty, financing) are requested from Merchant Support.

There is no self-serve API key page documented in the merchant dashboard guides.

## Credential Names

| MOBY term | Also called | Used for |
|---|---|---|
| Client ID (e.g. `MOBY00000123`) | Merchant ID | `clientId` / `client_id` in every request |
| Secret Key | API Key | `/api/auth/token` and callback HMAC key |
| Bearer token | access token | All API calls, about 30 minutes |

Keep the Secret Key server-side only. It both mints tokens and signs callbacks.

## Environment Variables

Use names that fit the app. Suggested:

```text
MOBY_ENV=sandbox
MOBY_BASE_URL=https://dev.pay.mobycheckout.com
MOBY_CLIENT_ID=<MOBY_CLIENT_ID>
MOBY_SECRET_KEY=<MOBY_SECRET_KEY>
MOBY_CALLBACK_URL=https://example.com/api/webhooks/moby
MOBY_RETURN_URL=https://example.com/payments/moby/return
```

Production:

```text
MOBY_ENV=production
MOBY_BASE_URL=https://pay.mobycheckout.com
```

with the production Client ID and Secret Key. The callback HMAC key is the same Secret Key, so a separate `MOBY_WEBHOOK_SECRET` is only an alias; keep one source of truth.

## Sandbox

- Base URL `https://dev.pay.mobycheckout.com`.
- Test cards, expiry and CVV result codes and FPX test bank: see `moby-api.md` Sandbox Test Data.
- FPX sandbox: only ACF Bank B2B.
- No e-wallet sandbox. E-wallet tests are real charges.
- Plugins: "Sandbox"/"Test"/"test" mode in WooCommerce, Shopify, Magento selects the sandbox host. WooCommerce shows "Merchant does not has a test account" when sandbox credentials are missing.
- Callbacks need a publicly reachable URL; local dev servers will not receive them (use a tunnel).

## Merchant Dashboard

The Login button on moby.my points to `https://omni.mobypay.my`. The Merchant User Guide (videos) covers:

- Payments: transaction list, analytics, settlements list and details.
- Refund & Disputes: make refund requests, refund list, dispute status, analytics.
- Payment Links: create links, link details, attempts.
- Recurring Billing: plans, links, bills, settings (see `moby-recurring.md`).
- Contacts, Invoices, Campaigns, Vouchers, Discount Settings, Consumers, Broadcast.
- Marketplace shops: TikTok Shop, Lazada, Shopee.
- Loans, Insurance.
- Settings: Users, Terminal devices, Brand Settings (checkout look), Stores.

## Settlement And Payout

- Settlement records are viewable in Dashboard > Payments > Settlements.
- MOBY's BNPL page (`https://moby.my/bnpl/`) says the merchant receives the full payment amount upfront while the customer pays in instalments.
- Settlement timing is not published in the developer docs (checked 2026-10-21). The settlements API (`GET /merchants/{id}/settlements`) is "Not yet in production".

## Pricing

Pricing is published at `https://moby.my/pricing/`. Plans and rates change; do not copy numbers into code or docs. Merchants above the plan tiers are directed to sales.

## Support

- Merchant Support: `merchantsupport@moby.my`, 011 1111 7177 (credentials, scopes, sandbox, merchant category).
- Customer Care: `customercare@moby.my`, 011 1111 5155.
- No public status page is documented.
