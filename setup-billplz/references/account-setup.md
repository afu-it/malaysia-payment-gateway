# Billplz Account Setup

Use this file when user asks how to register, what keys are needed, or what dashboard setup is required before coding Billplz.

## Accounts

- Create/access Billplz account for production.
- Use sandbox at `https://www.billplz-sandbox.com` for testing.
- Keep sandbox and production API Secret Key, X Signature Key, Collection ID, bills, callbacks, and Payment Orders separate.
- Check service health before investigating provider outages:
  - `https://status.billplz.com/`
  - `https://check.billplz.com/`
  - `https://fpxstatus.billplz.com/`

## Credentials Needed

- API Secret Key: used for HTTP Basic Auth as username with blank password.
- Collection ID: required to create Bills. Get it from Collection page in Billplz account.
- X Signature Key: used to verify X Signature callback/redirect and V5 checksums.
- Optional OAuth partner credentials: only for Billplz partners using OAuth 2.0.
- Payment Order Collection ID: required for Payment Order payout API.

## Where Used

- Bill creation: API Secret Key and Collection ID.
- Callback/redirect verification: X Signature Key with HMAC-SHA256.
- Payment Order V5 requests: X Signature Key with HMAC-SHA512 checksum plus `epoch`.
- Payment Order callbacks: X Signature Key with HMAC-SHA512 checksum.

## Recommended Env Vars

```text
BILLPLZ_ENV=sandbox
BILLPLZ_API_SECRET_KEY=
BILLPLZ_COLLECTION_ID=
BILLPLZ_X_SIGNATURE_KEY=
BILLPLZ_CALLBACK_URL=
BILLPLZ_REDIRECT_URL=
```

For Payment Order:

```text
BILLPLZ_PAYMENT_ORDER_COLLECTION_ID=
```

Use separate values for production.

## Next.js + Cloudflare Setup Notes

Use this section as a generic public template when adding Billplz to a Next.js app deployed on Cloudflare.

Add server-side env vars to `.env.example`, Cloudflare Worker secrets, payment library, checkout route branch, webhook route, provider config, settlement tests, and docs.

Recommended app env vars:

```text
BILLPLZ_API_SECRET_KEY=
BILLPLZ_COLLECTION_ID=
BILLPLZ_X_SIGNATURE_KEY=
BILLPLZ_CALLBACK_URL=https://example.com/api/payments/billplz/webhook
BILLPLZ_REDIRECT_URL=https://example.com/billing?provider=billplz
APP_BASE_URL=https://example.com
```

Cloudflare setup commands:

```bash
npx wrangler@latest --version
npx wrangler@latest secret put BILLPLZ_API_SECRET_KEY --env dev
npx wrangler@latest secret put BILLPLZ_COLLECTION_ID --env dev
npx wrangler@latest secret put BILLPLZ_X_SIGNATURE_KEY --env dev
```

Implementation checklist:

1. Add Billplz to the app payment provider config.
2. Add Billplz option to the admin/payment settings UI if the app has one.
3. Add `lib/payments/billplz.js` with amount conversion, payload builder, API calls, and X Signature verification.
4. Add server-side checkout/session creation route.
5. Add webhook route, for example `app/api/payments/billplz/webhook/route.ts`.
6. Set `payments.provider='billplz'`, store Bill ID in `gateway_ref`, and keep settlement idempotent.
7. Add tests for X Signature mismatch, duplicate callback, wrong amount, wrong Collection ID, and paid success.

Do not add Billplz secrets as `NEXT_PUBLIC_*`.

## Setup Checklist

1. Register/access Billplz account.
2. Create/select Collection.
3. Copy Collection ID from Collection page.
4. Copy API Secret Key from account settings.
5. Copy X Signature Key from settings page.
6. Configure public HTTPS `callback_url`.
7. Configure optional `redirect_url` for better user experience.
8. Create sandbox Bill and complete test payment.
9. Verify X Signature, amount, paid/state, Bill ID, Collection ID, and local reference before settlement.
10. For Payment Order, create Payment Order Collection, verify limit, use unique `reference_id`, and verify V5 checksum.
11. Move live only after live credentials, live Collection, and live payment gateway settings are ready.

## Do Not Assume

- Do not expose API Secret Key or X Signature Key in frontend/mobile code.
- Do not mark paid from redirect alone.
- Do not rely on callback/redirect execution order.
- Do not use Open Collection to create bills via API.
- Do not create Payment Order payouts without unique references and checksum validation.
