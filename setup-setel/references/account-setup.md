# Setel Account Setup

Use this reference for Setel merchant onboarding, credentials, dashboard, sandbox vs production, env vars and commercial questions. Checked 2026-10-21.

## Source URLs

- `https://www.setel.com/business/payments`
- `https://developer.setel.com/docs/integrating-your-platform-with-setel-api`
- `https://developer.setel.com/docs/system-environment`
- `https://developer.setel.com/reference/authentication-headers-for-setel-partners`
- `https://developer.setel.com/docs/accessing-settlement-reports`
- `https://developer.setel.com/docs/enabling-manual-settlement-for-virtualcardless-payments`

## Product Positioning

- Operator: Setel Ventures Sdn Bhd, Kuala Lumpur (business page footer).
- Business page offers "Pay online" (add Setel as a payment option in an online store so customers pay with their Setel account) and "Scan & Pay" (in-store).
- Payment options named on the business page: Setel Wallet, Family Wallet, bank cards (CardTerus). The developer docs list Setel Wallet and Setel CardTerus for online checkout.
- It reaches existing Setel app users. It is not a general FPX, DuitNow or multi-wallet gateway.

## Registration

There is no self-serve sign-up in the public docs.

1. Email `commercial@setel.com` (business page: "Drop us an email at commercial@setel.com to discuss further"). The page also links `business@setel.com`.
2. Agree commercial terms (Merchant Discount Rate is agreed "during the onboarding process" per the reconciliation page).
3. Provide the designated settlement bank account during onboarding.
4. Receive `x-api-key`, `x-api-secret` and `merchantId` ("after onboarding as a merchant with Setel").
5. Receive Setel Dashboard credentials (`https://dashboard.setel.my/login`).
6. If needed, ask for `x-interface-id` / `x-interface-type` values, SSO `client_id` / `client_secret` and the Setel Connect authorise URL, `enterpriseId` for manual settlement, and SFTP setup.

Documents required for onboarding: not documented in official docs (checked 2026-10-21). Ask the Setel commercial team.

## Environments And Credentials

| Item | Sandbox | Production |
| --- | --- | --- |
| API base | `http://api-pre-prod.setel.com` | `https://api.setel.com` |
| Keys | Issued by Setel | Issued by Setel |
| Dashboard | Not documented | `https://dashboard.setel.my/login` |

- Where keys live in the dashboard is not documented; keys are handed over during onboarding.
- Keep sandbox and production keys, webhook URLs and stored sessions separate.
- Sandbox test accounts are not published ("To be created" on the code scan page). Ask Setel for test Setel app users/wallet balance.

Suggested env vars:

```text
SETEL_BASE_URL=http://api-pre-prod.setel.com   # production: https://api.setel.com
SETEL_API_KEY=<SETEL_API_KEY>
SETEL_API_SECRET=<SETEL_API_SECRET>
SETEL_MERCHANT_ID=<SETEL_MERCHANT_ID>
SETEL_SUCCESS_URL=https://<your-domain>/pay/setel/return
SETEL_CANCEL_URL=https://<your-domain>/pay/setel/cancel
SETEL_WEBHOOK_URL=https://<your-domain>/webhooks/setel
# optional
SETEL_INTERFACE_ID=<value from Setel>
SETEL_INTERFACE_TYPE=<value from Setel>
SETEL_ENTERPRISE_ID=<value from Setel>
```

Never commit these; Setel says keys "must be kept secure and not uploaded to any version control system".

## Payment Method Activation

- Online: Setel Wallet and Setel CardTerus. No activation steps are documented; confirm enabled methods with Setel.
- Code scan: Setel Wallet only.

## Settlement And Payout

- Automatic daily settlement to the registered bank account; report at 01:00 GMT+8 for the previous day. See `setel-settlement.md`.
- Manual settlement can be enabled in Dashboard > Settings > Payments.
- Refunds use available balance only.

## Pricing

Setel does not publish merchant rates. Point users to `https://www.setel.com/business/payments` and `commercial@setel.com`. Do not quote numbers.

## Launch Checklist

1. Production keys stored in the secret manager, not in code.
2. `SETEL_BASE_URL` switched to `https://api.setel.com`.
3. Production success, cancel and webhook URLs on HTTPS.
4. Webhook signature verified with the production `x-api-secret`.
5. Requery job for sessions without a webhook.
6. Settlement report import (Dashboard or SFTP) mapped to `referenceId`.
7. Setel confirmed the live checkout link field names and any IP allowlisting.
