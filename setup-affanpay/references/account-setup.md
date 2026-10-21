# AffanPay Account Setup

Use this reference for registration, credentials, sandbox vs live, payment channel approval, settlement, and env vars. Checked 2026-10-21. Verify current terms in the dashboard or with AffanPay before claiming certainty.

## Source URLs

- `https://affanpay.my/` (homepage, pricing section, payment methods)
- `https://affanpay.my/contact.html` (FAQ, support contacts, business hours)
- `https://app.affanpay.my/register` (live registration)
- `https://sandbox.affanpay.my/register` (sandbox registration)
- `https://app.affanpay.my/api` (Token section: where the API token and secret key live)

## Product Positioning

- Malaysian payment gateway operated by Affanpay Sdn Bhd (Ampang, Selangor).
- Channels marketed: FPX online banking (30+ banks), DuitNow Online Banking, DuitNow QR, e-wallets (TNG, ShopeePay, Boost, GrabPay and more), Visa and Mastercard cards.
- No-code tools: payment links, QR codes, dashboard. Developer tools: REST API and a sandbox environment.
- Sister product Bayar.io (`https://bayar.io`) turns a phone into a DuitNow QR terminal; it is separate from the API.

## Account Setup Checklist

1. Register at `https://app.affanpay.my/register`: enter a valid email, accept the terms and PDPA notice, then open the one-time verification link (expires in 24 hours; check Spam/Promotions).
2. Upload supporting documents listed on the registration page:
   - MyKad or passport: a photo of yourself with the ID clearly visible, not mirrored.
   - Company SSM certificate.
   - Bank statement header showing bank name, account holder name, and account number.
3. The contact FAQ says a business needs SSM registration and a business bank account; individuals need IC and a personal bank account. Verification is marketed as under 24 hours (FAQ says 24 to 48 hours to accept payments).
4. After approval, sign in at `https://app.affanpay.my/sign-in`.
5. Go to Settings > Developer:
   - Under API tokens, name a token and click Generate token. Copy it immediately; it is shown once.
   - Copy the secret key shown on the same page (used to verify callback HMAC).
6. Optional: create bill categories in the Category section and note the Category ID for the `category` field.
7. Request approval for DuitNow QR, e-wallet, and card channels if needed (marked "subject to approval" on the pricing section). FPX/DuitNow Online Banking is not marked as needing approval.

## Sandbox Notes

- Sandbox host: `https://sandbox.affanpay.my` with its own registration (`/register`), sign-in (`/sign-in`), and docs (`/api`).
- Sandbox docs are identical to live docs except every URL uses the sandbox host.
- Sandbox credentials (token, secret key) are separate from live. Never mix them.
- Sandbox test cards, test banks, or simulated outcomes are not documented in official docs (checked 2026-10-21). Ask AffanPay support how to simulate Success and Failed in sandbox.

## Pricing And Settlement

- Pricing: see the Transparent Pricing section on `https://affanpay.my/`. Do not hardcode rates; they change and custom enterprise rates are available.
- No monthly or setup fees are stated on the homepage and FAQ.
- Settlement: T+1 (next business day) for all payment methods, per the contact FAQ.
- `fee_charge_payer: true` on a bill passes the transaction fee to the payer, which makes callback `paid_amount` higher than `bill.amount`.

## Support

- Email `contact@affanpay.my`, WhatsApp link on the contact page, office hours Monday to Friday 9:00 AM to 5:00 PM (closed weekends and public holidays). Payments process 24/7.
- Use support for anything the API docs do not cover: refunds, sandbox test data, rate limits, callback IPs.

## Required Environment Variables

Use names that match the target app's conventions. Suggested:

```text
AFFANPAY_ENV=sandbox
AFFANPAY_BASE_URL=https://sandbox.affanpay.my/api
AFFANPAY_API_TOKEN=<SANDBOX_API_TOKEN>
AFFANPAY_SECRET_KEY=<SANDBOX_SECRET_KEY>
AFFANPAY_CATEGORY_ID=<OPTIONAL_CATEGORY_ID>
AFFANPAY_REDIRECT_URL=https://example.com/pay/affanpay/return
AFFANPAY_CALLBACK_URL=https://example.com/api/webhooks/affanpay
```

For production, change to:

```text
AFFANPAY_ENV=production
AFFANPAY_BASE_URL=https://app.affanpay.my/api
```

and use the live API token, live secret key, live redirect URL, and live callback URL.

Keep `AFFANPAY_API_TOKEN` (Bearer auth for outgoing calls) and `AFFANPAY_SECRET_KEY` (HMAC key for incoming callbacks) as two distinct secrets.
