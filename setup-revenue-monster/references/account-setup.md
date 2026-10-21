# Revenue Monster Account Setup

Use this reference when explaining Revenue Monster registration, Merchant Portal setup, credentials, sandbox activation, payment method activation, pricing, plugins, and env vars. Checked 2026-10-21.

## Source URLs

- `https://doc.revenuemonster.my/docs/introduction/overview`
- `https://doc.revenuemonster.my/docs/quickstart/accesstoken/client-credentials`
- `https://doc.revenuemonster.my/docs/quickstart/signature-algorithm`
- `https://doc.revenuemonster.my/docs/quickstart/verify-signature`
- `https://doc.revenuemonster.my/docs/settings/store-detail/store-details`
- `https://doc.revenuemonster.my/docs/payment-method`
- `https://doc.revenuemonster.my/docs/downloads/testing-wallets`
- `https://doc.revenuemonster.my/docs/ecom-plugin/lowCodeCheckout`
- `https://doc.revenuemonster.my/docs/error-codes`
- `https://revenuemonster.my/pricing`

## Product Positioning

Revenue Monster (Revenue Monster Sdn Bhd) offers unified payments (online checkout, QR, terminals), loyalty, vouchers, eKYC, and plugins through RESTful Open APIs and SDKs (Go, PHP, NodeJS, Kotlin/Java, Python; C# deprecated). Not every SDK implements every API.

## Registration

1. Sign up as a merchant on the Merchant Portal: production `https://merchant.revenuemonster.my/`, sandbox `https://sb-merchant.revenuemonster.my/`.
2. Submit the documents the portal asks for and wait for RM admin approval. The exact document list is shown in the portal, not in public docs. Business help: bd@revenuemonster.my.
3. Sandbox is self sign-up, but you must email support@revenuemonster.my to activate the sandbox account.

## Credentials Checklist

Do this separately for sandbox and production.

1. Merchant Portal > Developer > Applications: create an application.
2. Open it and copy the Client ID; click Show to reveal the Client Secret. The application can be toggled ON/OFF (inactive returns `InActiveClient`).
3. Generate a 2048-bit RSA key pair (portal: Developer > Application > Generate RSA Key, or your own tool). Keep the private key on your server only.
4. Upload your public key to the application. Missing key returns `CLIENT_NOT_SET_PUBLIC_KEY`.
5. Copy the RM Server public key from the same application page; you need it to verify RM signatures.
6. Get your Store ID from the portal or `GET /v3/stores`.
7. Optional: the Signature Debugger tool in the portal signs and verifies test strings with your keys.

## Payment Method Activation

Methods must be activated on the merchant account. Errors such as `BOOST_MALAYSIA_NOT_ACTIVE`, `GRABPAY_MALAYSIA_NOT_ACTIVE`, `ALIPAY_CHINA_NOT_ACTIVE`, or `DOES_NOT_HAVE_ACTIVE_WALLET` mean contact RM (support@revenuemonster.my) to activate. Use `method: []` in checkout to show all enabled methods. WeChat Pay mini program needs RM to bind your Mini Program App ID first.

## Sandbox Notes

- Hosts: `sb-oauth.revenuemonster.my`, `sb-open.revenuemonster.my`, checkout on `sb-pg.revenuemonster.my`.
- Only some wallets have sandbox test wallets (Alipay, ShopeePay, Maybank, RazerPay). The table marks WeChat Pay, GrabPay, MCash as production only; card via GOBIZ is "Credit Card" in production and No in sandbox. The page note contradicts the table for WeChat Pay (it says a real WeChat Pay wallet works in both sandbox and production and recommends one sandbox round with it).
- Sandbox error responses include a `debug` object, useful for signature problems.

## Plugins

Official options: Low-Code Checkout Button (portal: Developer > Applications > your app > Low Code Checkout; set allowed host, redirect URL, optional notify URL), WooCommerce (`github.com/RevenueMonster/woocommerce-revenuemonster-payment-gateway`), OpenCart (`github.com/RevenueMonster/opencart-payment-gateway`), EasyStore. Use a plugin when a custom API integration is not needed.

Low-code button (`https://doc.revenuemonster.my/docs/ecom-plugin/lowCodeCheckout`): load `https://rm-component.oss-ap-southeast-3.aliyuncs.com/checkout-button/index.css` and `index.js` (`type="module"`; sandbox host `rm-component-sandbox.oss-ap-southeast-3.aliyuncs.com`), then render `<rm-pay-button>` with attributes `clientId` (required), `storeId` (required), `currency` (required, `MYR`), `amount` (required, sen), `title` (required), `orderId`, `detail`, `additionalData`, `customerId` (needed for tokenization), `customerName`, `customerEmail`, plus styling `variant` (`default`, `destructive`, `outline`, `secondary`, `ghost`, `link`) and `class`. Amount and order data sit in the browser, so still verify every paid order server-side with Query Transaction.

Plugin settings: WooCommerce asks for Store ID, Client ID, Client Secret, Client Private Key, Server Public Key; OpenCart asks for Sandbox Mode, RM Client ID, Client Secret, Store ID, Client Private Key, minimum Total (`0.10`); EasyStore asks for Merchant ID (portal: Setting > Merchant) and Store ID (portal: Store Management).

## Apple Pay And Google Pay Onboarding

- Apple Pay: ask RM to enable Apple Pay; get a CSR from RM's integration team and use it to obtain the Apple Payment Processing Certificate; create an Apple Merchant ID and Merchant Identity Certificate; verify your domains with Apple (`apple-developer-merchantid-domain-association`). Source: `https://doc.revenuemonster.my/docs/digital-wallet/apple-pay`.
- Google Pay: sign up with Google for a Google Pay Merchant ID, complete Google's web integration checklist and production access request, accept Google's Acceptable Use Policy and API Terms; confirm `gateway` and `gatewayMerchantId` values with RM. Help: support@revenuemonster.my. Source: `https://doc.revenuemonster.my/docs/digital-wallet/google-pay`.

## Partner Accounts

Merchant Onboarding APIs (`/v3/partner/...`) are for partner accounts only; contact RM to activate a partner account. After onboarding, use the onboarded merchant's own Developer Application for all other Open API calls. Source: `https://doc.revenuemonster.my/docs/merchant-onboarding/introduction`.

## Pricing And Settlement

Pricing: `https://revenuemonster.my/pricing`. Do not hardcode rates. The reconciliation API (`POST /v3/payment/reconciliation`) returns per-transaction `mdr`, `serviceFee`, and `settlementAmount`; use it to reconcile payouts. Settlement schedule is not documented in the public developer docs (checked 2026-10-21); confirm with RM or the portal.

## Required Environment Variables

Use names that match the target app. Common set:

```text
RM_ENV=sandbox
RM_OAUTH_URL=https://sb-oauth.revenuemonster.my
RM_API_URL=https://sb-open.revenuemonster.my
RM_CLIENT_ID=<CLIENT_ID>
RM_CLIENT_SECRET=<CLIENT_SECRET>
RM_PRIVATE_KEY=<PEM of your RSA private key>
RM_SERVER_PUBLIC_KEY=<PEM of RM server public key>
RM_STORE_ID=<STORE_ID>
RM_REDIRECT_URL=https://example.com/payments/rm/return
RM_NOTIFY_URL=https://example.com/api/webhooks/rm
```

For production:

```text
RM_ENV=production
RM_OAUTH_URL=https://oauth.revenuemonster.my
RM_API_URL=https://open.revenuemonster.my
```

and use the production Client ID/Secret, a production key pair, production server public key, production Store ID, and production URLs. Store PEM keys with real newlines (or Base64-encode them and decode at startup). Cache the access token and refresh token in server storage, not in env vars.
