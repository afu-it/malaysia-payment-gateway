# 2C2P Account Setup

Use this reference when explaining how a Malaysian merchant gets a 2C2P account, credentials, sandbox access, keys, payment method activation, pricing and settlement.

## Source URLs

- `https://2c2p.com/countries/malaysia/`
- `https://developer.2c2p.com/docs/sandbox`
- `https://developer.2c2p.com/docs/sandbox-setup`
- `https://developer.2c2p.com/docs/reference-environment-guide`
- `https://developer.2c2p.com/docs/reference-jwt-with-key`
- `https://developer.2c2p.com/docs/certificate-generation-guide`
- `https://developer.2c2p.com/docs/reference-payment-channels`
- `https://developer.2c2p.com/docs/woocommerce`
- `https://developer.2c2p.com/docs/batch-services-authorization`

## Company Context

- 2C2P is a regional payment platform, now branded "2C2P by Antom". Its Malaysian entity, 2C2P System (Malaysia) Sdn Bhd, is on Bank Negara Malaysia's list of non-bank merchant acquirers (checked 2026-10-21).
- It targets mid to large merchants. Pricing, setup fees and settlement terms are quoted by sales.

## Getting An Account

1. Contact 2C2P sales through the Malaysia page (`https://2c2p.com/countries/malaysia/`) for a quote and onboarding.
2. Complete onboarding with 2C2P. The KYC document list for Malaysian merchants: Not documented in official docs (checked 2026-10-21); 2C2P provides it during onboarding.
3. Receive the production Merchant ID and Secret Key from 2C2P. The plugin guides say both can be found in the my2C2P merchant portal.
4. Ask 2C2P to enable each payment channel you need (cards, MyDebit, FPX, DuitNow QR, Touch 'n Go, GrabPay, Boost, ShopeePay, BNPL, IPP banks, Apple Pay, Google Pay). Enabled channels are per merchant profile; call Payment Option or Payment Option Details with a token to see what is live.
5. Set default frontend and backend return URLs in the merchant portal, or send `frontendReturnUrl` and `backendReturnUrl` on every Payment Token.

## Sandbox

- No registration needed for basic tests: use the published demo merchant for Malaysia, `JT07` (MYR / 458), with its published Secret Key from `https://developer.2c2p.com/docs/sandbox`. Sandbox only; these values are public and must never be used in production.
- Sandbox API host `https://sandbox-pgw.2c2p.com`; production `https://pgw.2c2p.com`.
- A dedicated sandbox account (needed for your own return URLs in the portal, maintenance keys or SFTP) comes from 2C2P during onboarding.
- Test cards, 1-2-3 test login and FPX test banks (`MYSBIA`, `MYSBIB`, `MYSBIC`) are in `2c2p-codes.md`.

## Keys And Where They Live

| Key | Used for | Where |
| --- | --- | --- |
| Merchant ID | All APIs | Issued by 2C2P; my2C2P portal |
| Secret Key ("SHA Key") | JWT HS256 for PGW APIs and payout, QuickPay HMAC-SHA1 | Issued by 2C2P; my2C2P portal |
| 2C2P public key / certificate | JWE encryption and JWS verification for Payment Maintenance and token maintenance | Merchant Portal > Account > Options > 2C2P Public Keys |
| Merchant RSA key pair | JWS signing (private) and JWE decryption (private); upload the X.509 public key | Generate yourself (certificate generation guide); upload under Merchant Portal > Account > Options > Merchant Public Keys and set as default |
| SFTP account | Batch files and reconciliation reports | Request with the SFTP setup form to 2C2P support |
| GPG public key | Encrypt batch authorization and token files | Download link on the batch services pages |

Suggested environment variable names (the docs do not prescribe any):

```bash
TWOC2P_ENV=sandbox                       # or production
TWOC2P_MERCHANT_ID=<MERCHANT_ID>
TWOC2P_SECRET_KEY=<SECRET_KEY>
TWOC2P_PGW_BASE=https://sandbox-pgw.2c2p.com
TWOC2P_MAINTENANCE_URL=https://demo2.2c2p.com/PaymentAction/2.0/action
TWOC2P_PUBLIC_CERT_PEM=<2C2P_PUBLIC_CERT_PEM>
TWOC2P_MERCHANT_PRIVATE_KEY_PEM=<MERCHANT_PRIVATE_KEY_PEM>
TWOC2P_FRONTEND_RETURN_URL=https://shop.example.my/2c2p/return
TWOC2P_BACKEND_RETURN_URL=https://shop.example.my/2c2p/notify
```

Keep separate values per environment. Never commit the Secret Key or private key; never send them to the browser or a mobile app.

## Go-Live Checklist

1. Production Merchant ID and Secret Key received and stored in a secret manager.
2. Base URL switched to `https://pgw.2c2p.com` (and maintenance to `https://t.2c2p.com/PaymentAction/2.0/action`).
3. Production `backendReturnUrl` reachable over HTTPS from the internet.
4. Production merchant public key uploaded and set as default; production 2C2P public key downloaded.
5. Required channels confirmed enabled on the production profile.
6. One small live transaction completed end to end, then refunded or voided.

## Pricing, Settlement And Payout

- Pricing: see `https://2c2p.com/countries/malaysia/` and request a quote from 2C2P. No Malaysian rate card is published; do not quote numbers.
- Settlement timing to the merchant bank account: Not documented in official docs (checked 2026-10-21); agreed in the merchant contract.
- Daily reconciliation reports arrive by SFTP (see `2c2p-batch-recon.md`).
- Payout (disbursement to third parties) is a separate feature that 2C2P must enable (see `2c2p-quickpay-payout.md`).

## Plugins

Shopify, WooCommerce, Magento 2 and OpenCart 3 plugins are documented. They need only Merchant ID, Secret Key and mode. See `2c2p-batch-recon.md`.
