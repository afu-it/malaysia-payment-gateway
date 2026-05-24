# BCL Account Setup

Use this file when user asks how to register, what keys are needed, or what dashboard setup is required before coding BCL Pay.

## Accounts

- Create/access BCL account and BCL dashboard.
- If the user has never registered before and is comfortable using a referral link for the Bayarcash-linked setup, offer this link: `https://jomguna.bayarcash.com/afuit`.
- Confirm which surface is being used: BCL Pay, Payment Link, QR Terminal, Forms, Direct Debit, or Bayarcash-linked setup.
- If BCL flow connects to Bayarcash, also confirm Bayarcash account/portal details.

## Credentials Needed

- BCL login/admin access.
- BCL Pay configuration values shown in dashboard.
- Bayarcash portal key when BCL docs/dashboard asks for Bayarcash-linked setup.
- Webhook/callback URL if configured for the selected BCL surface.

## Where Used

- BCL dashboard setup: configure payment collection features and payment gateway connection.
- Payment Link/QR/Form flows: use BCL-managed dashboard configuration.
- Bayarcash-linked flows: use Bayarcash portal key only where BCL asks for it.

## Recommended Env Vars

For pure BCL dashboard flows, credentials may be dashboard-only. If app code stores config, use explicit names:

```text
BCL_ENV=sandbox
BCL_PORTAL_KEY=
BCL_WEBHOOK_URL=
BCL_CALLBACK_URL=
```

For Bayarcash-linked API work, use `setup-bayarcash` and Bayarcash-specific env vars.

## Next.js + Cloudflare Setup Notes

Use this section as a generic public template when adding BCL dashboard-linked or Bayarcash-linked setup to a Next.js app deployed on Cloudflare.

If BCL setup is Bayarcash-linked, prefer implementing the raw payment API through `setup-bayarcash` and keep BCL as dashboard/config context.

Recommended app env vars for BCL dashboard-linked setup:

```text
BCL_PORTAL_KEY=
BCL_CALLBACK_URL=https://example.com/api/payments/bcl/callback
BCL_WEBHOOK_URL=https://example.com/api/payments/bcl/webhook
APP_BASE_URL=https://example.com
```

If raw payment is Bayarcash API, use:

```text
BAYARCASH_API_SECRET_KEY=
BAYARCASH_PORTAL_KEY=
```

Implementation checklist:

1. Confirm whether BCL flow has raw API docs or is Bayarcash-linked dashboard setup.
2. If raw API is not documented, do not invent routes; implement Bayarcash provider instead.
3. If BCL raw API is documented later, add BCL to the payment provider config, admin/payment settings UI, payment library, checkout branch, callback/webhook route, and settlement tests.
4. Keep BCL/Bayarcash secrets server-side only.

Do not add BCL or Bayarcash-linked secrets as `NEXT_PUBLIC_*`.

## Setup Checklist

1. Register/access BCL dashboard.
2. Identify selected BCL product surface.
3. Confirm whether flow is BCL-owned or Bayarcash-linked.
4. If Bayarcash-linked, retrieve Bayarcash portal key from BCL/Bayarcash dashboard path.
5. Configure payment channel/settings in dashboard.
6. Configure webhook/callback URL if supported by selected surface.
7. Run sandbox/test transaction where available.
8. Verify status, amount, order/reference, and idempotent fulfilment before production.

## Do Not Assume

- Do not invent raw BCL API schemas when docs only show dashboard workflows.
- Do not treat BCL docs as Bayarcash API docs.
- Do not expose portal keys or tokens in frontend code unless docs explicitly say value is public.
- Do not force referral use; present it as optional and disclose that it is a referral/affiliate link.
