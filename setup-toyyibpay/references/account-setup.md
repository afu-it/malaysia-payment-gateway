# toyyibPay Account Setup

Use this file when user asks how to register, what keys are needed, or what dashboard setup is required before coding toyyibPay.

## Accounts

- Register production account at `https://toyyibpay.com/access/registration`.
- If the user has never registered before and is comfortable using a referral link, offer this ToyyibPay referral link: `https://toyyibpay.com/e/586756305483202823`.
- Register sandbox account at `https://dev.toyyibpay.com` for API testing.
- Keep sandbox and production User Secret Key, Category Code, bills, and callbacks separate.
- Complete account/profile/bank documentation as required before expecting live settlement.

## Credentials Needed

- User Secret Key: required for `createCategory`, `createBill`, callback hash verification, and other user APIs.
- Category Code: required for `createBill`.
- Enterprise User Secret Key: only for Enterprise Partner APIs such as create account/user status/settlement summary.
- DuitNow QR activation: account-level approval/activation needed before enabling DNQR on bills.

## Where Used

- Category creation: `userSecretKey`.
- Bill creation: `userSecretKey`, `categoryCode`, amount, return URL, callback URL, payer info, external reference.
- Callback verification: `MD5(userSecretKey + status + order_id + refno + "ok")`.
- Status re-check: `getBillTransactions` by `billCode` and optional status.

## Recommended Env Vars

```text
TOYYIBPAY_ENV=sandbox
TOYYIBPAY_USER_SECRET_KEY=
TOYYIBPAY_CATEGORY_CODE=
TOYYIBPAY_RETURN_URL=
TOYYIBPAY_CALLBACK_URL=
TOYYIBPAY_ENABLE_DUITNOW_QR=false
```

Use separate values for production.

## Next.js + Cloudflare Setup Notes

Use this section as a generic public template when adding toyyibPay to a Next.js app deployed on Cloudflare.

Add server-side env vars to `.env.example`, Cloudflare Worker secrets, payment library, checkout route branch, callback route, provider config, settlement tests, and docs.

Recommended app env vars:

```text
TOYYIBPAY_USER_SECRET_KEY=
TOYYIBPAY_CATEGORY_CODE=
TOYYIBPAY_CALLBACK_URL=https://example.com/api/payments/toyyibpay/callback
TOYYIBPAY_RETURN_URL=https://example.com/billing?provider=toyyibpay
TOYYIBPAY_ENABLE_DUITNOW_QR=false
APP_BASE_URL=https://example.com
```

Cloudflare setup commands:

```bash
npx wrangler@latest --version
npx wrangler@latest secret put TOYYIBPAY_USER_SECRET_KEY --env dev
npx wrangler@latest secret put TOYYIBPAY_CATEGORY_CODE --env dev
```

Implementation checklist:

1. Add toyyibPay to the app payment provider config.
2. Add toyyibPay option to the admin/payment settings UI if the app has one.
3. Add `lib/payments/toyyibpay.js` with amount conversion, create Bill, transaction lookup, and MD5 callback hash verification.
4. Add server-side checkout/session creation route.
5. Add callback route, for example `app/api/payments/toyyibpay/callback/route.ts`.
6. Set `payments.provider='toyyibpay'`, store Bill Code in `gateway_ref`, and keep settlement idempotent.
7. Add tests for hash mismatch, duplicate callback, wrong amount, wrong Bill Code, pending/fail status, and paid success.

Do not add toyyibPay secrets as `NEXT_PUBLIC_*`.

## Setup Checklist

1. Register/access toyyibPay account.
2. For sandbox, register/use `dev.toyyibpay.com`.
3. Get User Secret Key from toyyibPay dashboard/API setup flow.
4. Create Category and store Category Code.
5. Confirm payment channels in `My Payment Channel`.
6. If using DuitNow QR, confirm account activation with `checkDuitNowQRStatus`.
7. Configure public HTTPS callback URL; localhost will not receive callbacks.
8. Create sandbox Bill and complete test payment.
9. Verify callback hash, status, amount, Bill Code, and local order reference.
10. Move live only after live Category Code, live User Secret Key, and live channels are ready.

## Do Not Assume

- Do not expose User Secret Key in frontend/mobile code.
- Do not mark paid from Return URL alone.
- Do not enable dynamic amount bills for fixed-price products unless settlement validates allowed amount.
- Do not enable DuitNow QR unless account is activated.
- Do not force referral use; present it as optional and disclose that it is a referral/affiliate link.
