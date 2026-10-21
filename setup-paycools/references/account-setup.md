# PayCools Account Setup

Use this reference when explaining PayCools merchant onboarding, credentials, the RSA key exchange, environment config, channel activation, pricing, and settlement caveats.

## Source URLs

- `https://www.paycools.com.my/`
- `https://www.paycools.com.my/about/`
- `https://www.paycools.com.my/contact/`
- `https://www.paycools.com.my/products/qrcode/`
- `https://docs.paycools.com.my/en/9367977m0` (merchant RSA keys)
- `https://docs.paycools.com.my/en/9367979m0` (PayCools public key)
- `https://docs.paycools.com.my/en/9368011m0`, `https://docs.paycools.com.my/en/9368008m0` (channels)

## Who PayCools Is

- Operated by Micro Bluetech Sdn Bhd (company no. 202001042390 / 1398711-M), Bangsar South, Kuala Lumpur.
- Describes itself as a SaaS business-solutions company with operations in Malaysia, Indonesia, and the Philippines. These docs cover the Malaysia API only (`docs.paycools.com.my`).
- Products on the Malaysia site: payment gateway (banks, e-wallets, cards, DuitNow QR), Payment Links, QR Code payments, disbursements, and a merchant dashboard for monitoring incoming payments.

## Becoming A Merchant

The website has no self-serve signup. The documented path is:

1. Contact sales: `salesmy@paycools.com` (Monday to Friday, 9:00 AM to 6:00 PM).
2. Application and discussion with a Business Development Executive.
3. System integration (PayCools advertises free setup and integration help).
4. Live account.

Required KYB documents: Not documented in official docs (checked 2026-10-21). Expect the usual Malaysian SSM and bank documents; ask PayCools for the exact list.

## Credentials

| Item | Who creates it | Where it lives |
| --- | --- | --- |
| `appId` | PayCools issues it | Merchant config (`PAYCOOLS_APP_ID`) |
| Merchant RSA private key (4096-bit, PKCS#8, Base64) | Merchant generates | Server secret store only (`PAYCOOLS_PRIVATE_KEY`) |
| Merchant RSA public key (X.509, Base64) | Merchant generates | Sent to PayCools; PayCools verifies your requests with it |
| PayCools platform RSA public key | PayCools | Request from the PayCools administrator; store as `PAYCOOLS_PLATFORM_PUBLIC_KEY` and use to verify notifications |
| API base URL | PayCools | Not published; store as `PAYCOOLS_API_BASE_URL` |

There is no API key or HMAC secret. Authentication is the RSA signature plus `appId`, and likely an IP allowlist (error `21016 Merchant ip forbidden`).

### Generate the key pair

The docs give a Java `KeyPairGenerator` example (4096-bit RSA, Base64 of the encoded keys). An OpenSSL equivalent that produces the same formats:

```sh
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out paycools_private.pem   # PKCS#8 PEM
openssl pkey -in paycools_private.pem -pubout -out paycools_public.pem                   # X.509 PEM
# Base64 body only (what the Java sample prints):
grep -v -- '-----' paycools_public.pem | tr -d '\n'
```

Send only the public key to PayCools. Never reuse the example key pair printed in the docs.

## Environment Variables

```text
PAYCOOLS_API_BASE_URL=<FROM_PAYCOOLS>
PAYCOOLS_APP_ID=<APP_ID>
PAYCOOLS_PRIVATE_KEY=<BASE64_PKCS8_PRIVATE_KEY>
PAYCOOLS_PLATFORM_PUBLIC_KEY=<BASE64_X509_PAYCOOLS_PUBLIC_KEY>
PAYCOOLS_NOTIFY_URL=https://<your-domain>/api/paycools/notify
PAYCOOLS_REDIRECT_URL=https://<your-domain>/checkout/return
PAYCOOLS_REFUND_CALLBACK_URL=https://<your-domain>/api/paycools/refund-notify
```

Keep UAT and live in separate config sets (different `appId`, keys, base URL, and callback URLs).

## Sandbox / UAT

Not documented in official docs (checked 2026-10-21). No public sandbox signup, host, test cards, or test bank logins are published. Ask PayCools for UAT access, the UAT base URL, a UAT `appId`, and the UAT platform public key.

## Payment Method Activation

- Channels are chosen per order with `channelCode`. If a channel is not enabled for your account, the API returns `21005 Merchant channel is unavailable`.
- WebPay channels: FPX banks, DuitNow RPP online banking banks, cards (`CARD_URL`), Touch 'n Go, Boost, GrabPay, MCash.
- QR: `DUITNOW_DYNAMIC_QR` only via API; static DuitNow QR cannot be created via the Open API for now.
- Confirm with PayCools which channels are active on your account before showing them at checkout.

## Pricing

PayCools advertises free setup and "low transaction fee". Rates are quote-based; there is no public pricing page. Check `https://www.paycools.com.my/` and ask sales; do not hard-code rates.

## Settlement And Payouts

- Settlement schedule to the merchant bank: Not documented in official docs (checked 2026-10-21).
- The merchant holds a PayCools balance (`/open-api/account/balance`) that funds payouts (`21014 Merchant balance insufficient`).
- Refunds do not return the transaction fee.

## Support

- Email: `salesmy@paycools.com`.
- Address: Unit 32-01, Level 32, Vertical Corporate Office Tower B, Bangsar South, Kuala Lumpur.
- No public status page found.
