# LeanX Account Setup

Use this reference when explaining LeanX registration, credentials, sandbox vs production, plugin setup, payout pool funding, settlement, and pricing. Checked 2026-10-21.

## Source URLs

- `https://leanx.io`
- `https://leanx.io/pricing`
- `https://onboard.leanx.io`
- `https://docs.leanx.io/api-docs/credential`
- `https://docs.leanx.io/api-docs/signature-validation`
- `https://docs.leanx.io/api-docs/support/frequently-asked-question`
- `https://docs.leanx.io/api-docs/suite-plugin/woocommerce`
- `https://docs.leanx.io/api-docs/suite-plugin/open-cart`

## Product Positioning

LeanX (Lean.x, by Leanis Solutions Sdn Bhd) is a Malaysian payment platform covering online collection (FPX B2C and B2B, cards, e-wallets, DuitNow online banking, DuitNow QR, BNPL, PayPal, Stripe card routing per the payment type list), Collections and Bills, a hosted payment page, instant DuitNow payouts from a prefund pool, and WooCommerce / OpenCart plugins. Which payment types are actually enabled depends on your account and plan.

## Registration

1. Register at `https://onboard.leanx.io`.
2. After the first successful login, the onboarding dashboard gives you sandbox credentials.
3. Complete onboarding for a production merchant account. The required business documents are not listed in the public developer docs (checked 2026-10-21); the onboarding portal and LeanX support state them. Check the pricing page for any extra approval needed for card acceptance.
4. Log in to the Merchant Portal: sandbox `https://portal.leanx.dev`, production `https://portal.leanx.io`.

## Where Credentials Live

Merchant Portal > API page:

1. Add a new API key and enable it (an "INVALID_TOKEN" / 5688 error usually means the key was not created or enabled in that portal).
2. Read the Merchant ID, UUID, Hash Key, and Auth Token for that key.
3. Optional: API > API Key > Edit API Key > X-Signature to require HMAC-signed requests.

A walkthrough video is linked from the Credential page: `https://youtu.be/Mu6TwEE1zbw`.

| Credential | Secret? | Use |
| --- | --- | --- |
| Merchant ID | No | Identity, part of Auth Token |
| UUID | Low | API key id, part of Auth Token and X-Signature message |
| Auth Token | Yes | `auth-token` header on every request |
| Hash Key | Yes | X-Signature HMAC key and callback JWT verification |
| Collection UUID | No | Required on every bill; create in portal or via API |
| `virtual_pool_reference` | No | Payout pool id from portal Prefund / Pool Fund page |

## Sandbox vs Production

| Item | Sandbox | Production |
| --- | --- | --- |
| API | `https://api.leanx.dev` | `https://api.leanx.io` |
| Portal | `https://portal.leanx.dev` | `https://portal.leanx.io` |
| Transactions | Mock, contained in sandbox | Live |
| IP whitelist | Not applied | Available (configure with LeanX) |

- A public generic sandbox credential is published on the Credential page (see `leanx-api.md`); use your own sandbox key for realistic responses.
- Credentials, Collection UUIDs, payment service IDs and pool references differ by environment. Never mix them.
- Payout test accounts: `1111111111` (FAILED), `1212121212` (PENDING).

## Required Environment Variables

Use names that match the target app. Suggested:

```text
LEANX_ENV=sandbox
LEANX_API_URL=https://api.leanx.dev
LEANX_MERCHANT_ID=<MERCHANT_ID>
LEANX_UUID=<API_KEY_UUID>
LEANX_AUTH_TOKEN=<AUTH_TOKEN>
LEANX_HASH_KEY=<HASH_KEY>
LEANX_COLLECTION_UUID=<COLLECTION_UUID>
LEANX_SIGNATURE_ENABLED=false
LEANX_CALLBACK_URL=https://example.com/api/webhooks/leanx
LEANX_REDIRECT_URL=https://example.com/payments/leanx/return
LEANX_PAYOUT_POOL_REFERENCE=<VIRTUAL_POOL_REFERENCE>
LEANX_PAYOUT_CALLBACK_URL=https://example.com/api/webhooks/leanx-payout
```

For production set `LEANX_ENV=production`, `LEANX_API_URL=https://api.leanx.io`, and use production credentials, collection, pool reference and URLs.

## Payment Method Activation

- Enabled channels are returned by `list-payment-services` with `payment_status: "active"`. Query each `payment_type` you plan to show.
- FPX B2C and B2B have different rates and separate service lists.
- Card via the Credit Card library uses `payment_service_id` `1100` in production.
- Refresh the service list daily.

## Plugins

### WooCommerce

- Requirements: PHP 7.0+, WooCommerce 3.0 to 6.7.
- Install WooCommerce first, then upload the LeanX plugin (Plugins > Add New), activate, open Settings.
- Tick "Enable LeanX", tick "Enable Sandbox Mode" for sandbox or untick for production.
- Fill Auth Token (labelled API key), Collection UUID, Hash Key, Bill Invoice prefix.
- Download link: `https://drive.google.com/file/d/1shoSLSnKwhW7BxwGjXzgnh0w9r_psCUt/view`.
- Plugin behaviour: callback route `/wp-json/leanx/v1/callback`; pending orders are requeried after 5 minutes and cancelled after 12 hours when an admin opens the Orders page.

### OpenCart 3.0

- Requirements: PHP 7.0+, OpenCart 3.0.x only.
- Extensions > Installer, upload the `.ocmod.zip`, then Extensions > Extensions > Payments, install LeanX, set Auth Token, Collection UUID, Hash Key, set Status Enabled.
- Download link: `https://drive.google.com/file/d/12dBLvZqHiQhqATgCkupI3EHTwuCy1ofO/view?usp=share_link`.

Both plugins decode callbacks by posting the JWT and Hash Key to a LeanX decode endpoint and use legacy hosts (`stag-api.leanpay.my`). Test callback delivery in the target store.

## Settlement And Payout Notes

- By default collection income settles to the merchant's bank account. Settling directly into the payout pool can be arranged before going live.
- Payout pool: one `virtual_pool_reference` per merchant; keep at least RM2.00 above each payout; the RM2 is refunded on account deactivation.
- Payouts are real time, 24/7 including weekends and public holidays; recipients see "Leanis Solutions Sdn Bhd" as the sender.
- Settlement timing (for example T+1) is a commercial term; read it from the pricing page or your agreement.

## Pricing

See `https://leanx.io/pricing` for current plans, per-method rates, setup fees and settlement terms. Do not hardcode pricing in application logic; it changes.

## Support

Contact LeanX via the portal or `https://leanx.io` for production IP whitelisting, refunds (no public refund API), payout pool settlement routing, and onboarding documents.
