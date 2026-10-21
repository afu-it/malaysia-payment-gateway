# M1Pay Account Setup

Use this reference when explaining M1Pay registration, credentials, merchant portal settings, UAT (sandbox) setup, payment channels, pricing and payout caveats.

## Source URLs

- `https://www.m1pay.com.my/`
- `https://www.m1pay.com.my/assets/documents/Form-A1M1Pay.pdf` (merchant application form, dated May 2023)
- `https://www.m1pay.com.my/#/docs/introduction/get-started`
- `https://www.m1pay.com.my/#/docs/introduction/overview`
- `https://www.m1pay.com.my/merchant-portal/` (production portal)
- `https://m1payall.com/merchant-portal/` (UAT portal)
- `https://m1payall.com/payment-test/` (official "Try Sandbox" demo page)

All checked 2026-10-21.

## Product Positioning

M1Pay is the payment gateway of MobilityOne Sdn Bhd (Kuala Lumpur). The site offers FPX, cards and e-wallets through one integration, plus a merchant portal. Channels shown on the homepage: Online Banking (FPX), Card Payment, DuitNow, Touch 'n Go eWallet, eM-ONEi, Maybank QRPay, GrabPay, ShopeePay. The API docs only give channel codes for some of these (see `m1pay-api.md`).

## Registration

1. Download the application form from the homepage "Register" button: `https://www.m1pay.com.my/assets/documents/Form-A1M1Pay.pdf`.
2. Tick "M1Pay Payment Processing Services Application (eCommerce Merchant)" and the services you need (FPX, Credit Card, MyDebit, eWallets, DuitNow QR).
3. Fill in business details, website/app URL and IP address, delivery time of goods, and the settlement bank account.
4. Attach due-diligence documents listed in the form:
   - Latest company registration search.
   - Latest certificate of incorporation/registration.
   - NRIC/passport of all directors, shareholders and authorised signatories.
   - Latest month bank statement showing beneficiary name, address and account number.
   - Exterior photo (front with signboard) and interior photo (with products) of the business premises.
   - Office-use checklist mentions SSM forms (sole proprietor Form 9, 13 and SSM; Sdn Bhd Form 49, 24 and SSM).
5. Submit to MobilityOne business development: `bd@mobilityone.com.my`, phone `+603-9213 0669` (from the site footer). Individuals put their name as business name and NRIC as registration number.
6. MobilityOne may run CTOS/credit checks and site reviews before approval.

## Credentials

The Get Started page says M1Pay emails these after account creation:

| Item | Use |
| --- | --- |
| Merchant Code | Identifies the merchant; sent as `merchantId` and used as OAuth2 `client_id` |
| Private Key | Server-side RSA key (`{merchant_code}.key`) used to create `signedData` |
| Secret Key | OAuth2 `client_secret` |

The OAuth2 page says Client ID and Client Secret "can be found in the M1Pay Merchant Portal". To verify callbacks you also need M1Pay's public certificate (`.crt`, used in the official verification samples). Where to download it is not documented; ask M1Pay if it is not in the portal.

The private key can be regenerated in the merchant dashboard at any time. Replace the server key file in the same change window, or new payments will fail signature checks.

## Merchant Portal Settings

The merchant must give M1Pay (or set in the portal):

- IP Address: outgoing static IP of the payment server, for whitelisting.
- Web Redirect URL: where the customer lands after payment.
- Host to Host Callback URL: server endpoint for POSTed status, max 255 characters, one per merchant account. Can be set in the portal or emailed to M1Pay support.

Set these separately in the UAT portal and the production portal.

## UAT (Sandbox) vs Production

| | UAT | Production |
| --- | --- | --- |
| Hosts | `gateway.m1payall.com`, Keycloak realm `master` | `gateway.m1pay.com.my`, Keycloak realm `m1pay-users` |
| Portal | `https://m1payall.com/merchant-portal/` | `https://www.m1pay.com.my/merchant-portal/` |
| Channels | `ONLINE_BANKING`, `CARD_PAYMENT` only | Channels subscribed in the portal |
| FPX test bank | `SBI Bank A` | Real banks |
| Test credentials | From M1Pay support | Issued after approval |

Keys, certificates, client credentials and URLs are per environment. Do not reuse them across environments.

The demo page `https://m1payall.com/payment-test/` lets you see the hosted flow. It calls an internal M1Pay endpoint, not the merchant API, so do not copy its requests.

## Environment Variables

Suggested names (the docs do not prescribe any):

```text
M1PAY_ENV=uat|production
M1PAY_MERCHANT_ID=<MERCHANT_CODE>
M1PAY_CLIENT_ID=<CLIENT_ID>
M1PAY_CLIENT_SECRET=<CLIENT_SECRET>
M1PAY_PRIVATE_KEY_PATH=/secure/path/<MERCHANT_CODE>.key
M1PAY_PUBLIC_CERT_PATH=/secure/path/m1pay-public.crt
M1PAY_REDIRECT_URL=https://example.com/payments/m1pay/return
M1PAY_CALLBACK_URL=https://example.com/payments/m1pay/callback
```

On serverless platforms, store the PEM contents in a secret instead of a file path.

## Pricing, Settlement and Refunds

- Pricing: see the MDR schedule in the official application form (`Form-A1M1Pay.pdf`, Schedule 3 Appendix C). Do not quote numbers from memory; they change and some fields are blank. Ask `bd@mobilityone.com.my` for a current quote.
- Payout timing: listed in the same Appendix C. Confirm current terms in your merchant agreement.
- Settlement report: the form says a settlement report is made available after 12.00pm Malaysia time each business day (via the portal/online logon). No API for it is documented.
- Chargebacks and refunds: the form says MobilityOne processes refunds required by law and deducts them from the merchant balance. No refund API is documented; handle refunds through M1Pay.
