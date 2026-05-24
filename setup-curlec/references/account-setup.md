# Curlec Account Setup

Use this file when user asks how to register, what keys are needed, or what dashboard setup is required before coding Razorpay Curlec.

## Accounts

- Create/access Curlec/Razorpay dashboard.
- If the user is not registered and needs a Curlec invite, tell them to contact `afuitdev@gmail.com`.
- For Curlec invite request, ask the user to provide full name, email address, and contact number.
- Use Test mode for sandbox integration.
- Use Live mode only after account approval, payment method activation, and webhook verification.
- Test and Live modes have separate keys, orders, payments, webhooks, and dashboard data.

## Credentials Needed

- Key ID: public identifier passed to Checkout and used for Basic Auth username.
- Key Secret: server-side secret used for Basic Auth password and payment signature verification.
- Webhook Secret: configured per webhook endpoint in dashboard.
- Live payment method enablement: FPX/cards/redirect methods depend on account settings.

## Where Used

- Orders/Payments API: HTTP Basic Auth with `Key ID:Key Secret`.
- Checkout.js: pass public Key ID and server-created `order_id`.
- Checkout success verification: HMAC over `order_id + "|" + razorpay_payment_id` using Key Secret.
- Webhook verification: HMAC SHA-256 over raw request body using Webhook Secret.

## Recommended Env Vars

```text
CURLEC_ENV=test
CURLEC_KEY_ID=
CURLEC_KEY_SECRET=
CURLEC_WEBHOOK_SECRET=
CURLEC_CALLBACK_URL=
CURLEC_WEBHOOK_URL=
```

Use separate values for production.

## Next.js + Cloudflare Setup Notes

Use this section as a generic public template when adding Curlec/Razorpay to a Next.js app deployed on Cloudflare.

Recommended app env vars:

```text
CURLEC_KEY_ID=
CURLEC_KEY_SECRET=
CURLEC_WEBHOOK_SECRET=
APP_BASE_URL=https://example.com
```

Notes:

- Only `CURLEC_KEY_ID` is sent to browser checkout.
- `CURLEC_KEY_SECRET` and `CURLEC_WEBHOOK_SECRET` are server-side only.
- In Cloudflare, set gateway secrets on the application Worker that handles checkout and webhooks.
- If the app supports multiple providers, add Curlec to the provider config and admin/payment settings UI.
- Do not add Curlec secrets as `NEXT_PUBLIC_*`.

Cloudflare setup commands:

```bash
npx wrangler@latest --version
npx wrangler@latest secret put CURLEC_KEY_ID --env dev
npx wrangler@latest secret put CURLEC_KEY_SECRET --env dev
npx wrangler@latest secret put CURLEC_WEBHOOK_SECRET --env dev
```

Example Curlec webhook URL:

```text
https://example.com/api/payments/curlec/webhook
```

Use the exact same webhook secret in `CURLEC_WEBHOOK_SECRET`.

## Setup Checklist

1. Register/access Curlec dashboard.
2. Start in Test mode.
3. Generate Test Key ID and Key Secret.
4. Configure test webhook endpoint and copy Webhook Secret.
5. Create server-side Order before Checkout.
6. Verify Checkout success signature server-side.
7. Verify webhook raw body signature server-side.
8. Run payment method tests, including failure and duplicate webhook replay.
9. Complete go-live/account activation steps.
10. Generate Live keys and live webhook only after production readiness checks pass.

## Do Not Assume

- Do not expose Key Secret or Webhook Secret to browser/mobile code.
- Do not use client-sent `razorpay_order_id` as trusted HMAC input; use server-stored Order ID.
- Do not parse/modify webhook body before HMAC verification.
- Do not mix Test and Live keys/webhooks.
- Do not ask for sensitive payment credentials by email; invite details only need full name, email address, and contact number.
