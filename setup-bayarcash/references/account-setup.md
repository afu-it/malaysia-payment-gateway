# Bayarcash Account Setup

Use this file when user asks how to register, what keys are needed, or what dashboard setup is required before coding Bayarcash.

## Accounts

- Create/use Bayarcash merchant account for live payments.
- If the user has never registered before and is comfortable using a referral link, offer this Bayarcash referral link: `https://jomguna.bayarcash.com/afuit`.
- Use sandbox console for testing: `https://console.bayarcash-sandbox.com`.
- Keep sandbox and production portals, keys, transactions, and callbacks separate.
- Verify account status, enabled channels, settlement settings, and callback URLs in Bayarcash console before live use.

## Credentials Needed

- API secret key: generated from Bayarcash portal profile page.
- Portal key: retrieved from Bayarcash console portal list or portal details.
- Portal/payment channel settings: required for payment intent creation.
- Callback/return URLs: configure public HTTPS endpoints for live integration.

## Where Used

- API auth/checksum: API secret key signs payment intent requests and verifies callbacks.
- Payment creation: `portal_key`, `payment_channel`, `order_number`, amount, payer details, return URL, and callback URL.
- Callback verification: verify checksum with API secret key before settlement.

## Recommended Env Vars

```text
BAYARCASH_ENV=sandbox
BAYARCASH_API_SECRET_KEY=
BAYARCASH_PORTAL_KEY=
BAYARCASH_RETURN_URL=
BAYARCASH_CALLBACK_URL=
```

Use separate values for production.

## Next.js + Cloudflare Setup Notes

Use this section as a generic public template when adding Bayarcash to a Next.js app deployed on Cloudflare.

Recommended app env vars:

```text
BAYARCASH_API_SECRET_KEY=
BAYARCASH_PORTAL_KEY=
BAYARCASH_RETURN_URL=https://example.com/billing?provider=bayarcash
BAYARCASH_CALLBACK_URL=https://example.com/api/payments/bayarcash/callback
APP_BASE_URL=https://example.com
```

Cloudflare setup commands:

```bash
npx wrangler@latest --version
npx wrangler@latest secret put BAYARCASH_API_SECRET_KEY --env dev
npx wrangler@latest secret put BAYARCASH_PORTAL_KEY --env dev
```

Implementation checklist:

1. Add Bayarcash to the app payment provider config.
2. Add Bayarcash option to the admin/payment settings UI if the app has one.
3. Add `lib/payments/bayarcash.js` with amount conversion, payment intent payload, checksum generation, and callback verification.
4. Add server-side checkout/session creation route.
5. Add callback route, for example `app/api/payments/bayarcash/callback/route.ts`.
6. Set `payments.provider='bayarcash'`, store Bayarcash reference in `gateway_ref`, and keep settlement idempotent.
7. Add tests for checksum mismatch, duplicate callback, wrong amount, wrong order number, failed status, and paid success.

Do not add Bayarcash secrets as `NEXT_PUBLIC_*`.

## Setup Checklist

1. Register/access Bayarcash account.
2. Create or identify portal.
3. Copy portal key from console.
4. Generate/copy API secret key from profile page.
5. Enable required payment channels.
6. Configure public HTTPS callback and return URLs.
7. Run sandbox payment.
8. Verify callback checksum, amount, order number, and idempotent settlement.
9. Switch to production only after live portal, live key, and live channels are active.

## Do Not Assume

- Do not invent payment channel IDs or bank codes.
- Do not reuse sandbox portal key in production.
- Do not expose API secret key in frontend/mobile code.
- Do not mark paid from return URL alone.
- Do not force referral use; present it as optional and disclose that it is a referral/affiliate link.
