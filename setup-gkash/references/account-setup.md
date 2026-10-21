# Gkash Account Setup

Use this reference when explaining Gkash registration, sandbox setup, credentials, payment methods, pricing, and settlement.

## Source URLs

- `https://doc.gkash.my/getting-started`
- `https://gkash.com/faq_1/`
- `https://onboard.gkash.my/`
- `https://demo.pay.asia`
- `https://doc.gkash.my/google-pay`
- `https://doc.gkash.my/woocommerce`
- `https://doc.gkash.my/opencart`

## Product Positioning

Gkash offers online payments (cards, internet banking, e-wallets, BNPL, installments) on a hosted payment page, plus offline payments (e-wallet QR, DuitNow QR, SoftPOS terminal, Soundbox). Developers integrate online payments with the host-to-host checkout API, the legacy hosted form, a plugin, or a mobile SDK.

## Sandbox Setup

1. Open `https://doc.gkash.my/getting-started`.
2. Fill in the sign-up form: company name, email address, contact number.
3. Open the verification link Gkash emails you. The email also contains the information needed to start integrating.
4. Log in to the sandbox merchant portal at `https://demo.pay.asia` to view transactions and reports.
5. Point the integration at `https://api-staging.pay.asia`.

The Gkash FAQ also says: "Please create your staging account at https://doc.gkash.my/."

## Production Registration

1. Register at `https://onboard.gkash.my/`.
2. Basic documents listed in the FAQ: all directors' IC, SSM or any business license, bank statement (first page), outlet or signboard photo.
3. Gkash says approval takes 3 working days after full documentation.
4. API credentials: "You will receive the API Key information from your registered email." For help: `merchantsupport@gkash.com`.
5. The FAQ says integration documentation can also be downloaded from the merchant portal under the documentation tab.

## Credentials

| Item | What it is | Where it comes from |
| --- | --- | --- |
| `CID` | Gkash "User ID" / Merchant ID, for example `M161-U-xxx` | Issued on onboarding, sent by email |
| Signature key | Secret used in every SHA512 signature | Issued on onboarding, sent by email |

Staging and production credentials are different. The plugins expose a "Sandbox Mode" / "Staging Environment" switch for this.

Suggested environment variables:

```text
GKASH_CID=<CID>
GKASH_SIGNATURE_KEY=<SIGNATURE_KEY>
GKASH_BASE_URL=https://api-staging.pay.asia   # production: https://api.gkash.my
GKASH_CALLBACK_URL=https://example.com/payments/gkash/callback
GKASH_RETURN_URL=https://example.com/payments/gkash/return
```

Keep `GKASH_SIGNATURE_KEY` server-side only.

## Payment Methods

- Hosted page tabs: credit/debit card (`ECOMM`), internet banking (`EBANKING`), e-wallet (`EWALLET`). Limit or preselect them with `paymentmethod` and `preselection`.
- Installments: `CIMB`, `PBB`, `HLB`, `RHB`, 6 to 36 months, when enabled.
- Google Pay: selected merchants only. Enable in the merchant portal under Payment Settings > Customize Payment Page > Google Pay tab. Only `CRYPTOGRAM_3DS` is supported; `PAN_ONLY` is not.
- Which methods are live on an account depends on Gkash onboarding. Confirm with Gkash or the merchant portal; the docs do not publish a per-account activation process.

## Plugins

- WooCommerce: `https://wordpress.org/plugins/woo-gkash/`. Settings: Title, Description, User ID, Signature Key, Staging Environment.
- OpenCart: Gkash extension on the OpenCart marketplace. Settings: Merchant ID, Signature Key, Sandbox Mode, Total, Geo Zone, Status.

## Pricing

Do not copy rates into code or docs; they depend on the package. See the official FAQ (Fee & Charges section): `https://gkash.com/faq_1/`, or contact Gkash sales.

## Settlement

From `https://gkash.com/faq_1/`:

- Settlement period depends on the subscribed package or special request.
- If settlement falls on a weekend or public holiday, it is processed on the next working day.
- Daily settlement reports are in the egkash merchant portal; reconcile orders against the settlement report and bank statement.
- MDR is deducted per transaction and shown in the settlement report.

## Go-Live Checklist

1. Production CID and signature key stored as secrets.
2. `GKASH_BASE_URL` switched to production, and endpoint paths confirmed with Gkash.
3. Callback URL is public HTTPS, accepts form POST, and returns `OK`.
4. Return URL accepts form POST.
5. One real low-value payment tested end to end, then refunded.
6. Reconciliation job uses the query API and the portal settlement report.
