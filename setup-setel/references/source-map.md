# Setel Source Map

Official sources only: `developer.setel.com` (Setel developer portal) and `www.setel.com`. Every portal page is available as Markdown with an embedded OpenAPI 3.1 definition by appending `.md` to its URL. Checked 2026-10-21.

## Coverage

- Index used: `https://developer.setel.com/llms.txt` (51 links) plus the portal sidebar links on `/docs/pay-with-setel-online` and `/reference/create-a-checkout-session`, which list every guide and reference page.
- Enumerated: 100 unique pages (31 `/docs/*` guides, 69 `/reference/*` pages including 43 endpoint pages with OpenAPI definitions and 26 category/overview pages).
- Read: 100 of 100 (all fetched as `.md`; OpenAPI definitions parsed for paths, parameters, request schemas and examples).
- Not available: `/llms-full.txt` (404), `/sitemap.xml` (404). No official SDK, npm package, GitHub repo or ecommerce plugin from Setel was found.
- Also read: `https://www.setel.com/business/payments`.
- Scope: checkout, payment intents, webhooks, refunds, settlement are covered in full detail. Accounts, SSO, loyalty, retail, parking and EV are covered at endpoint and field level in `setel-other-apis.md` because they are partner programmes rather than standard online payment acceptance.

## Product And Onboarding

- `https://www.setel.com/business/payments`
- `https://developer.setel.com/docs/integrating-your-platform-with-setel-api`

Use for: product positioning, payment options, commercial contact, credentials received at onboarding.

## Environments And Auth

- `https://developer.setel.com/docs/system-environment`
- `https://developer.setel.com/reference/headers`
- `https://developer.setel.com/reference/common-headers`
- `https://developer.setel.com/reference/authentication-headers-for-setel-partners`
- `https://developer.setel.com/reference/authentication-headers-for-setel-users`

Use for: base URLs, `x-api-key` / `x-api-secret`, `access-token`, `Content-Type`.

## Pay With Setel Online

- `https://developer.setel.com/docs/pay-with-setel-online`
- `https://developer.setel.com/reference/sessions`
- `https://developer.setel.com/reference/create-a-checkout-session`
- `https://developer.setel.com/reference/create-a-checkout-session-that-grants-loyalty-points-to-user`
- `https://developer.setel.com/reference/retrieve-a-checkout-session`
- `https://developer.setel.com/reference/retrieve-a-checkout-session-by-referenceid`
- `https://developer.setel.com/reference/finish-a-checkout-session`
- `https://developer.setel.com/reference/cancel-a-checkout-session`

Use for: checkout flow, PC vs mobile redirect, request/response fields, requery.

## Payment Intents, Capture, Refund, Cancel

- `https://developer.setel.com/docs/more-payment-scenarios`
- `https://developer.setel.com/docs/how-payment-intents-work`
- `https://developer.setel.com/docs/placing-a-hold-and-capturing-payments`
- `https://developer.setel.com/docs/after-the-payment`
- `https://developer.setel.com/docs/refunding-and-canceling-payments`
- `https://developer.setel.com/reference/payment-intents`
- `https://developer.setel.com/reference/retrieve-a-payment-intent`
- `https://developer.setel.com/reference/capture-a-payment-intent`
- `https://developer.setel.com/reference/capture-a-payment-intent-and-grant-loyalty-points-based-on-product`
- `https://developer.setel.com/reference/cancel-a-payment-intent`
- `https://developer.setel.com/reference/refund-a-payment-intent`

Use for: statuses, 30-minute hold expiry, capture limits, refund balance rule, cancel rules.

## Webhooks And Signature

- `https://developer.setel.com/docs/receive-payment-notifications-with-webhooks`

Use for: webhook payload, `signature` header, HMAC-SHA256 algorithm, test vector, duplicate/lost webhook guidance.

## Errors

- `https://developer.setel.com/reference/http-status-codes`
- `https://developer.setel.com/reference/success`
- `https://developer.setel.com/reference/client-errors`
- `https://developer.setel.com/reference/server-errors`
- `https://developer.setel.com/reference/error-codes`
- `https://developer.setel.com/reference/checkout`
- `https://developer.setel.com/reference/single-sign-on`

Use for: HTTP codes, error body, checkout error code table, SSO error values.

## Code Scan And User Payment Methods

- `https://developer.setel.com/docs/pay-with-setel-code-scan`
- `https://developer.setel.com/reference/wallets`
- `https://developer.setel.com/reference/create-a-wallet-transaction`
- `https://developer.setel.com/reference/retrieve-a-wallet-transaction`
- `https://developer.setel.com/reference/retrieve-a-wallet-transaction-by-referenceid`
- `https://developer.setel.com/reference/payment-methods`
- `https://developer.setel.com/reference/retrieve-all-payment-methods-of-a-user`
- `https://developer.setel.com/reference/retrieve-payment-code`
- `https://developer.setel.com/reference/retrieve-balance-of-each-payment-method-of-a-user`

Use for: in-person QR/barcode payments, user payment method ids for Finish session.

## Settlement

- `https://developer.setel.com/docs/reconciling-payments-with-settlements`
- `https://developer.setel.com/docs/accessing-settlement-reports`
- `https://developer.setel.com/docs/accessing-settlement-reports-sftp-connection`
- `https://developer.setel.com/docs/enabling-manual-settlement-for-virtualcardless-payments`
- `https://developer.setel.com/reference/settlements`
- `https://developer.setel.com/reference/create-a-settlement`

Use for: report timing, MPF file format, SFTP setup, manual settlement API.

## SSO, Accounts, Loyalty, Retail

- `https://developer.setel.com/docs/introduction-to-single-sign-on`
- `https://developer.setel.com/docs/sso-with-client-secret`
- `https://developer.setel.com/docs/sso-with-pkce`
- `https://developer.setel.com/docs/available-scopes`
- `https://developer.setel.com/docs/error-handling`
- `https://developer.setel.com/reference/oauth-tokens`, `/reference/issue-oauth-token`
- `https://developer.setel.com/reference/accounts`, `/reference/accounts-1`, `/reference/create-an-account-with-unverified-phone-number`, `/reference/create-passcode-for-account`, `/reference/reset-the-passcode`, `/reference/create-account-session-by-passcode`, `/reference/create-otp-for-account-session`, `/reference/verify-account-session-by-otp`, `/reference/create-account-session-by-otp`, `/reference/delete-account-session`
- `https://developer.setel.com/reference/loyalty-card-actions`, `/reference/activate-mesra-card`, `/reference/link-physical-mesra-card`, `/reference/retrieve-user-loyalty-card-details-with-access-token`, `/reference/retrieve-users-loyalty-card-details-with-mesra-card-number`
- `https://developer.setel.com/reference/fuel-order`, `/reference/retrieve-fuel-order-options`, `/reference/external-orders`, `/reference/create-an-external-order`, `/reference/retrieve-an-external-order`

Use for: partner apps acting on behalf of Setel users, Mesra points.

## Automated Parking And EV Charging

- `https://developer.setel.com/docs/how-automated-parking-works`, `/docs/automated-parking-webhooks`, `/docs/fee-query`, `/docs/session-status-check`, `/docs/void-parking-session`, `/docs/parking-location-deployment`
- `https://developer.setel.com/reference/parking-sessions`, `/reference/parking-events`, `/reference/create-a-parking-session`, `/reference/retrieve-a-parking-session`, `/reference/end-a-parking-session`, `/reference/close-a-parking-session`, `/reference/charge-a-parking-session`, `/reference/void-a-parking-session`, `/reference/update-a-parking-status`
- `https://developer.setel.com/docs/how-ev-charging-works`, `/docs/webhooks-needed-from-integrators`, `/docs/create-a-charging-session`, `/docs/end-a-charging-session`, `/docs/get-charging-location`, `/docs/retrieve-charging-session-information`
- `https://developer.setel.com/reference/charging-sessions`, `/reference/charging-events`, `/reference/update-charging-session-status-event`, `/reference/update-charging-session-payment-invoice`

Use for: parking operators and EV charging vendors only.

## Dashboard

- `https://dashboard.setel.my/login`

Use for: settlement reports, manual settlement toggle. Do not treat dashboard screens as payment truth for a single order; use the API.
