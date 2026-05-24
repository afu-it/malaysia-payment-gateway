# Xendit Account Setup

Use this file when user asks how to register, what keys are needed, or what dashboard setup is required before coding Xendit.

## Accounts

- Create/access Xendit dashboard.
- No referral link is configured for Xendit.
- Use test mode for development.
- Use live mode only after target payment channels, webhook URL, and account settings are ready.
- Keep test and live keys, webhooks, payment objects, and reporting separate.

## Credentials Needed

- Secret API key: server-side API authentication.
- Public API key: only for supported public-key flows such as tokenization/components when docs require it.
- Webhook/callback token: dashboard token used to verify `x-callback-token`.
- Optional xenPlatform sub-account IDs: used for platform/marketplace routing.

## Where Used

- API calls: HTTP Basic Auth username is secret API key, password is empty.
- Manual Basic Auth format: Base64 of `{secret_api_key}:`, including trailing colon.
- Webhook receiver: compare `x-callback-token` to dashboard webhook token.
- xenPlatform: use master account key plus routing header/body fields where docs require.

## Recommended Env Vars

```text
XENDIT_ENV=test
XENDIT_SECRET_API_KEY=
XENDIT_PUBLIC_API_KEY=
XENDIT_WEBHOOK_TOKEN=
XENDIT_WEBHOOK_URL=
XENDIT_RETURN_URL=
```

For platforms:

```text
XENDIT_FOR_USER_ID=
```

Use separate values for production.

## Next.js + Cloudflare Setup Notes

Use this section as a generic public template when adding Xendit to a Next.js app deployed on Cloudflare.

Recommended app env vars:

```text
XENDIT_SECRET_KEY=
XENDIT_WEBHOOK_TOKEN=
APP_BASE_URL=https://example.com
```

Notes:

- `XENDIT_SECRET_KEY` and `XENDIT_WEBHOOK_TOKEN` are server-side only.
- In Cloudflare, set gateway secrets on the application Worker that handles checkout and webhooks.
- If the app supports multiple providers, add Xendit to the provider config and admin/payment settings UI.
- Do not add Xendit secrets as `NEXT_PUBLIC_*`.

Cloudflare setup commands:

```bash
npx wrangler@latest --version
npx wrangler@latest secret put XENDIT_SECRET_KEY --env dev
npx wrangler@latest secret put XENDIT_WEBHOOK_TOKEN --env dev
```

Example Xendit webhook URL:

```text
https://example.com/api/payments/xendit/webhook
```

## Setup Checklist

1. Register/access Xendit dashboard.
2. Enable target country, currency, and payment channels.
3. Generate least-privilege test secret key.
4. Configure webhook endpoint and copy callback token.
5. Create test payment request/session from server.
6. Verify webhook `x-callback-token`.
7. Store Xendit object IDs and merchant `reference_id`.
8. Test duplicate/out-of-order webhook handling.
9. Generate live keys only after live channel activation and production webhook setup.

## Do Not Assume

- Do not expose secret API key in frontend/mobile code.
- Do not omit trailing colon in manual Basic Auth.
- Do not fulfil from client redirect alone.
- Do not route xenPlatform payments without confirming master vs sub-account ownership.
