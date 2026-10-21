# Payright Account Setup

Use this reference when explaining Payright (Malaysia) registration, portal setup, credentials, sandbox vs live, payment methods, settlement, and operational readiness. Checked 2026-10-21.

## Source URLs

- `https://www.payright.my/en` (product site)
- `https://www.payright.my/en/pricing` (plans and fees, link only)
- `https://www.payright.my/en/developer` (redirects to the API docs)
- `https://www.payright.my/en/integrations` (plugin list)
- `https://www.payright.my/en/contact` (support channels)
- `https://www.payright.my/en/system/status` (redirects to `https://payright.instatus.com/`)
- `https://onboard.payright.my/register` (merchant onboarding)
- `https://portal.payright.my/` (merchant portal, linked from the API docs)
- `https://docs.payright.my/introduction`, `/authentication`, `/signature-validation`, `/api/bill/callback`, `/sandbox`

## Product Positioning

Payright Sdn Bhd is a Kuala Lumpur payment gateway focused on low flat-fee FPX online banking. The site lists:

- Developer API, ready plugins, and public GitHub repositories.
- Plans (Lite, Growth, Enterprise coming soon) with FPX, e-wallet, credit card and BNPL listed as plan payment methods.
- Next-business-day FPX B2C settlement with a custom settlement schedule; Enterprise teases real-time FPX settlement.
- Payment Forms and Bills in the portal for no-code collection.
- Integrations page names platforms such as Magento, OpenCart and WHMCS.

The public Bill API documents FPX bank selection only (`provider_id`). How card, e-wallet or BNPL are offered through the API is not documented in official docs (checked 2026-10-21); ask Payright before promising those methods in a custom integration.

Not to be confused with Payright Australia (payright.com.au, BNPL, GitHub org `payright-engineering`).

## Pricing

Do not copy fee numbers into code or docs; they change. Link to `https://www.payright.my/en/pricing` and confirm the merchant's plan in the portal.

## Account Setup Checklist

Production:

1. Register at `https://onboard.payright.my/register` (the site's "Get Started" flow). Required business documents are not listed publicly (checked 2026-10-21); the onboarding flow or Payright support will request them.
2. Log in to the merchant portal (`https://portal.payright.my/`).
3. Go to `Accounts > API Keys > View API Keys (magnify icon)` and copy the **Auth Token**.
4. In the same view, copy the **Hash Key** (`Account > API Keys > View API Key > Hash Key`). It verifies callback JWTs and signs `x-signature`.
5. Decide on request signing: `Account > API Keys > Edit > JWT Verification`. If enabled, every API call must carry `x-signature`.
6. Create a Collection in the portal or with `POST /merchant/collections`; store its `collection_uuid`.
7. Confirm the payment services you need are active (errors 10018 / 10041 mean a service or provider is not active).
8. For payouts, confirm your virtual payout pool reference (`virtual_pool_reference`) and that the payout pool is funded (`GET /merchant/balance`).

Sandbox:

1. Use the sandbox API base `https://api.payright-sandbox.my/api/v1`.
2. Use sandbox credentials (Auth Token, Hash Key) from the sandbox account; never mix them with live keys. How to obtain a sandbox account is not stated in the current docs (checked 2026-10-21); the 2022 WooCommerce plugin readme pointed to `https://sandbox.payright.my`, which belongs to the legacy API. Ask Payright support for current sandbox access.
3. The sandbox payment page is a simulator. Failed sandbox bills do not count toward dashboard or settlement totals.
4. The official Postman collection (link on `https://docs.payright.my/postman-collection`) ships with a sandbox environment: `API_BASE_URL=https://api.payright-sandbox.my/`, `API_VERSION=api/v1/`, `AUTH_TOKEN`.

## Environment Variables

Suggested names (not mandated by Payright):

```text
PAYRIGHT_BASE_URL=https://api.payright-sandbox.my/api/v1   # or https://api.payright.my/api/v1
PAYRIGHT_AUTH_TOKEN=<AUTH_TOKEN>
PAYRIGHT_HASH_KEY=<HASH_KEY>
PAYRIGHT_COLLECTION_UUID=<COLLECTION_UUID>
PAYRIGHT_JWT_VERIFICATION=false        # true if enabled in the portal
PAYRIGHT_CALLBACK_URL=https://merchant.example/payment/payright/callback
PAYRIGHT_REDIRECT_URL=https://merchant.example/payment/return
PAYRIGHT_VIRTUAL_POOL_REFERENCE=<POOL_REF>   # payouts only
```

Keep separate values per environment. Never expose the Auth Token or Hash Key to the browser.

## Callback And Redirect Setup

- `callback_url` and `redirect_url` are passed per bill; there is no documented portal webhook registration.
- The callback URL must be public HTTPS and accept a JSON POST with a `data` JWT.
- Callbacks are documented only for successful transactions, so schedule a Get Bill requery for pending bills.

## Settlement And Payouts

- FPX B2C settlement is listed as next business day on Lite and Growth, with a "custom settlement schedule" on the pricing page; confirm options in the portal.
- Payout API moves money from your payout pool to recipient accounts; the same amount, account number and recipient reference within 3 hours is rejected as a duplicate (10149).
- No settlement report API is documented (checked 2026-10-21); reconcile from the portal.

## Refunds

No refund API is documented (checked 2026-10-21). Payright publishes a Refund Policy PDF at `https://www.payright.my/en/refund-policy`. Handle refunds through the portal or Payright support and record them in your own ledger.

## Support And Operations

- Support: existing merchants open a support ticket from the portal; the contact page also offers email and Messenger, with 24/7 support claimed.
- Security issues (from the official GitHub org): `dev@payright.my`.
- Status page: `https://payright.instatus.com/` (via `https://www.payright.my/en/system/status`). Use it for incidents, not as payment truth.
