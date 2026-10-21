# Tulus Account Setup

Use this reference when explaining Tulus onboarding, credentials, staging vs production, payment methods, pricing and settlement. Checked 2026-10-21.

## Source URLs

- `https://tulus.my/payment-gateway/`
- `https://tulus.my/sertai-kami/`
- `https://tulus.my/hubungi-kami/`
- `https://tulus.my/kerjasama/`
- `https://developers.tulus.my/` (Quick Start)
- `https://developers.tulus.my/laravel-handoff.md`
- `https://developers.tulus.my/sdk/`

## Who Tulus Is

- Tulus Digital Sdn Bhd (201101002165 (930302-A)), Shah Alam, Selangor. Footer: "An Acture company".
- Tulus Payment Gateway is described as owned and managed by Tulus Digital, "a Payment Exchange under PayNet".
- Same gateway also branded SnapNPay (`snapnpay.my`, `developers.snapnpay.my`).
- Serves zakat institutions, waqf administrators, charities, businesses and digital platforms. Many state zakat bodies, waqf bodies and AEON Bank collect through it (see `/kerjasama/`).
- Syariah adviser Lariba Consulting confirmed the gateway operations on 19 August 2026 (certificate PDF linked from `/payment-gateway/`).

## Payment Methods

- Gateway page: FPX, debit card and credit card, "channels enabled for your institution".
- Developer docs also mention DuitNow (checkout and DuitNow QR generator), card via MPGS, Touch 'n Go and SPayLater (field naming note).
- Channels, receipts and settlement are agreed per institution during onboarding. Confirm the enabled list for the agency before promising a method.

## Onboarding

1. Contact Tulus: "Talk to Tulus" enquiry form at `https://tulus.my/hubungi-kami/`, or the joining form (Google Forms) linked from `https://tulus.my/sertai-kami/`. Phone +603 5567 0103, `tulus@tulus.my`; developer support `support@tulus.my`.
2. Agree channels, integration, receipts and settlement arrangements.
3. Receive the agency code (also called merchant ID; alphanumeric with dashes, for example `snapnpay`).
4. Get merchant dashboard access at `https://reports.tulus.my/`.
5. Get the API key (UUID) from the dashboard API keys page (Laravel handoff), or ask Tulus.
6. Ask Tulus for staging access and to register your callback/return URL if your agency uses a registered callback (the callback reference says Tulus POSTs to "the callback URL you registered for your agency"; checkout also sends `returnUrl`).
7. Optional: ask Tulus to enable JSON response injection (`fpx` in the indirect return), a custom receipt email endpoint, or custom receipts.

The required documents and KYC list for onboarding are not published (checked 2026-10-21).

## Credentials

| Credential | Where | Use |
|---|---|---|
| Agency code | Onboarding | Every request |
| API key (UUID) | Dashboard API keys page | Sign checkout, signed reports and QR, verify callbacks |
| AuthToken (JWT) | `reports.tulus.my`, top-right menu, "Copy AuthToken to Clipboard" | Bearer auth for reports, multi-agency |
| Partner secrets (ADB, Grab) | Issued by Tulus | Partner webhooks only |

Do not use the deprecated username/password `token` method.

## Environment Variables

```dotenv
TULUS_AGENCY=<AGENCY_CODE>
TULUS_API_KEY=<API_KEY_UUID>
TULUS_AUTHTOKEN=<AUTHTOKEN_JWT>          # only if using Bearer reports
TULUS_CHECKOUT_URL=<confirm with Tulus>  # docs: https://tulus.my/v2/checkout
TULUS_REPORT_URL=https://reports.tulus.my
TULUS_RETURN_URL=https://<your-app>/tulus/return
```

Names follow the official Laravel handoff. Keep them in secrets storage, never in the repo or database.

## Staging vs Production

- Production checkout (SDK playground): `https://pay.tulus.my/`. Staging: `https://paydev.tulus.my/` with FPX test banks, no real money.
- The docs do not say whether staging uses separate agency codes or API keys (checked 2026-10-21); ask Tulus and keep separate config per environment anyway.
- Live smoke tests: use amounts ending in `12.34`.

## Pricing and Settlement

- Pricing is not published; there is no public pricing page (checked 2026-10-21). Fees are agreed with Tulus.
- A platform fee may be added on top of the order amount (`addCharge` in callbacks).
- "Settlement timing, deductions and accounts follow your agreement" (gateway page). Settlement and reconciliation records are in the merchant portal.

## Refunds

No refund API or refund process is documented in the developer docs (checked 2026-10-21). Ask Tulus how reversals are handled for your agency.
