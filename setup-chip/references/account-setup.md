# CHIP Collect Account Setup

Use this file when user asks how to register, what keys are needed, or what dashboard setup is required before coding CHIP Collect.

## Accounts

- Create/access CHIP Collect merchant account.
- Use CHIP Collect portal Developers section for API keys, Brand ID, and webhooks.
- Keep test and live API keys, brands, webhooks, and transactions separate.

## Credentials Needed

- Secret API key: used as Bearer token in `Authorization: Bearer <secret key>`.
- Brand ID: required for purchases, payment methods, and many checkout flows.
- Webhook public key: stored on created webhook object as `Webhook.public_key`.
- Success callback public key: retrieved from `GET /public_key/`.

## Where Used

- API calls: `Authorization: Bearer <secret key>`.
- Checkout/purchase creation: Brand ID plus purchase payload.
- Payment methods: usually Brand ID and currency.
- Callback verification: verify `X-Signature` using correct public key and raw request body.

## Recommended Env Vars

```text
CHIP_ENV=test
CHIP_API_KEY=
CHIP_BRAND_ID=
CHIP_WEBHOOK_PUBLIC_KEY=
CHIP_SUCCESS_CALLBACK_PUBLIC_KEY=
CHIP_SUCCESS_CALLBACK_URL=
CHIP_FAILURE_CALLBACK_URL=
```

Use separate values for production.

## Next.js + Cloudflare Setup Notes

Use this section as a generic public template when adding CHIP Collect to a Next.js app deployed on Cloudflare.

Recommended app env vars:

```text
CHIP_SECRET_KEY=
CHIP_BRAND_ID=
CHIP_CALLBACK_PUBLIC_KEY=
APP_BASE_URL=https://example.com
```

Notes:

- `CHIP_SECRET_KEY`, `CHIP_BRAND_ID`, and `CHIP_CALLBACK_PUBLIC_KEY` are server-side only.
- `APP_BASE_URL` or `NEXT_PUBLIC_APP_BASE_URL` is used to build redirect/webhook URLs.
- In Cloudflare, set gateway secrets on the application Worker that handles checkout and webhooks.
- If the app supports multiple providers, add CHIP to the provider config and admin/payment settings UI.
- Do not add CHIP secrets as `NEXT_PUBLIC_*`.

Cloudflare setup commands:

```bash
npx wrangler@latest --version
npx wrangler@latest secret put CHIP_SECRET_KEY --env dev
npx wrangler@latest secret put CHIP_BRAND_ID --env dev
npx wrangler@latest secret put CHIP_CALLBACK_PUBLIC_KEY --env dev
```

For production, use the project's production secret procedure and app Worker config. Do not paste live secrets into chat or commit them.

## Setup Checklist

1. Register/access CHIP Collect portal.
2. Open Developers section.
3. Generate/copy test API key.
4. Copy Brand ID.
5. Create webhook or configure callback URLs.
6. Store webhook public key or retrieve success callback public key.
7. Create test purchase and open returned `checkout_url`.
8. Verify signature with raw body before settling payment.
9. Switch to live only after live API key, live Brand ID, and live webhook are configured.

## Do Not Assume

- Do not omit trailing slashes on documented endpoints.
- Do not verify webhook with success-callback public key, or success callback with webhook public key.
- Do not expose secret API key in frontend/mobile code.
- Do not mark paid without signature and amount/currency/order checks.
