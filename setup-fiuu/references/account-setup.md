# Fiuu Account Setup

Use this file when user asks how to register, what keys are needed, or what dashboard setup is required before coding Fiuu.

## Accounts

- Create/access a Fiuu merchant account and portal.
- Use sandbox credentials for integration testing before live.
- Keep sandbox/UAT/dev/live credentials and endpoint URLs separate.
- Contact Fiuu support when a feature must be enabled, such as recurring/tokenization, pre-authorization, Direct Server API, Mobile XDK app authorization, or channel availability.

## Credentials Needed

- Merchant ID: identifies the merchant/domain.
- Verify Key: used to generate request hashes such as `vcode`.
- Secret Key: used to verify response hashes such as `skey`, XDK checksum, and response verification values.
- API password/API credentials: needed by some SDK/API/mobile surfaces.
- Environment setting: sandbox/UAT/dev/production.

## Where Used

- Hosted Payment Page: Merchant ID, Verify Key, Secret Key, Return URL, Notify URL, Callback URL, Cancel URL.
- Seamless Integration: Merchant ID, Verify Key, Secret Key, approved domain, subscribed channels, channel status checks.
- Inpage Checkout: Merchant ID/keys plus Inpage-specific repo/wiki setup.
- Mobile XDK: username, password, Merchant ID, app name, verification key, optional Secret Key verification on backend, registered app/platform when required.
- Recurring/token payments: tokenization/recurring enablement from Fiuu and exact API credentials from the recurring API docs.
- Direct Server: explicit Fiuu approval and PCI-DSS handling before collecting card data on merchant systems.

## Recommended Env Vars

```text
FIUU_ENV=sandbox
FIUU_MERCHANT_ID=
FIUU_VERIFY_KEY=
FIUU_SECRET_KEY=
FIUU_API_PASSWORD=
FIUU_RETURN_URL=
FIUU_NOTIFY_URL=
FIUU_CALLBACK_URL=
FIUU_CANCEL_URL=
```

Use separate values for production.

## Dashboard / Portal Setup

1. Log in to Fiuu merchant portal.
2. Get Merchant ID.
3. Get Verify Key and Secret Key from payment/security settings.
4. Configure Return URL, Notify/Notification URL, Callback URL, and Cancel URL.
5. Register domains for endpoints when they differ from main business website.
6. For Seamless Integration, register/approve the checkout domain.
7. For sandbox demo bank testing, whitelist the environment IP if required by sandbox portal.
8. Enable required channels and features for the target payment method.
9. Create a sandbox payment, receive Notify URL, verify `skey`, and run a status requery.
10. Move live only after live credentials, live URLs, channel enablement, domain approval, and idempotent settlement tests are complete.

## Framework Setup Notes

For web apps:

- Add server-only env vars to `.env.example`.
- Add provider config and checkout creation route.
- Add backend Notify/Notification URL and Callback URL handlers.
- Store pending payment before redirect/form submit.
- Verify `skey` before settlement.
- Add tests for invalid `skey`, duplicate notification, wrong amount/currency/order, pending status, and successful settlement.

For Flutter/mobile:

- Before installing packages, check current Flutter/Dart versions, verify minimum platform requirements, and update package manager/tooling if needed.
- Use latest stable package version in `pubspec.yaml`.
- After install, verify with `flutter --version` and `flutter pub outdated`.
- Keep app-side credentials minimal. Send payment result to backend for final verification and settlement.

## Do Not Assume

- Do not mark paid from Return URL, mobile callback, or frontend redirect alone.
- Do not expose Secret Key, Verify Key, API password, or checksum formulas with live values in frontend/mobile bundles.
- Do not assume `22` pending means success.
- Do not assume Seamless, recurring, tokenization, pre-auth, Direct Server, Apple Pay, Google Pay, or specific channels are enabled for every merchant.
- Do not reuse sandbox merchant IDs, keys, URLs, or whitelists in production.
