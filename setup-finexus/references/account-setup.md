# Finexus Account Setup

Use this reference for Finexus (MyXaaS / PayMaster / Kayaaku POS) registration, credentials, sandbox and live setup, method activation, and settlement questions. Checked 2026-10-21.

## Source URLs

- Get started: https://sandbox.finexusgroup.com/myxaas/docs/get-started
- Application form: https://sandbox.finexusgroup.com/myxaas/form
- Merchant payments product page (pricing claims, licences): https://www.finexusgroup.com/businesses/payments
- PayMaster intro: https://sandbox.finexusgroup.com/myxaas/docs/paymaster/intro
- UPP pre-requisites (MPI setup): https://sandbox.finexusgroup.com/myxaas/docs/paymaster/upp/embed-plugin/pre-requisites
- Static QR key exchange: https://sandbox.finexusgroup.com/myxaas/docs/paymaster/static-qr/key-exchange
- Kayaaku POS HTTP key exchange: https://sandbox.finexusgroup.com/myxaas/docs/kayaakupos/http/pre-requisites/key-exchange

## Who Finexus is

FINEXUS Cards Sdn Bhd (Finexus Group). Its product page says it is a registered merchant acquirer and licensed e-money issuer under Bank Negara Malaysia, with PCI DSS Level 1 and EMV compliance. Brands: PayMaster (online gateway), Kayaaku POS (terminals), MyXaaS (developer platform).

## Registration

1. Fill in the MyXaaS Application Enquiry Form (link above). Sections:
   - Business profile: merchant type, business nationality, business ID type and number, business name, trading name, business type, premise type, nature of business.
   - Business contact: name, ID type and number, nationality, phone, email. The email becomes the login for the Finexus sandbox merchant business portal.
   - Business address.
   - eCommerce info: industry, website.
   - Subscription service: PayMaster MPI and / or Merchant Payment (Online and Offline); other requests.
   - Company e-signature plus a witness e-signature; accept Finexus terms and confirm a separate NDA.
2. Finexus reviews the application and sends a confirmation email after approval.
3. Sandbox access follows approval. Some doc modules (OpenAPI spec, e-Invoice) need a `.p12` client certificate imported into the browser; its password is the Business ID Number from the form.

Supporting documents list (SSM, bank statement, director IC, etc.): Not documented in official docs (checked 2026-10-21). Finexus asks during review.

## Credentials and where they come from

| Item | Source | Use |
| --- | --- | --- |
| Merchant ID (MID, 15 digits) | Finexus at onboarding | `f001_MID` / `MerchantID` |
| Secret Key | "Obtain the Secret Key from the PayMaster Gateway" (Finexus) | HMAC-SHA256 hash for every PayMaster message |
| Service ID | Fixed `FNX` | `f260_ServID` |
| Terminal ID (TID) | Finexus, for QR and POS | `f354_TID`, `tid` |
| PayMaster host URLs | Finexus. Docs mask hosts as `xxx.finexusgroup.com`. | Payment, query, reversal URLs |
| Static QR RSA keys | Merchant generates RSA 2048, sends CSR; Finexus returns signed cert and its public key. Repeat for production. | Verify push notification, sign reply |
| Kayaaku POS keys | Merchant sends EC public key and Device Serial Number; Finexus returns its public key and `sourceSystem` | JWE and ECDSA signature |
| Pre-auth HTTPS keys | Merchant RSA 2048 key pair | `signedMessage` |

No dashboard path for viewing the Secret Key is documented. Ask Finexus how keys are delivered and rotated.

## Suggested environment variables

These names are a convention for your app, not Finexus names:

```
FINEXUS_ENV=sandbox                    # sandbox | production
FINEXUS_MERCHANT_ID=<MERCHANT_ID>
FINEXUS_SECRET_KEY=<SECRET_KEY>
FINEXUS_SERVICE_ID=FNX
FINEXUS_PAYMASTER_BASE_URL=<https://host-from-finexus>
FINEXUS_TERMINAL_ID=<TERMINAL_ID>      # QR / POS only
FINEXUS_RETURN_URL_SUCCESS=https://<your-domain>/payments/finexus/return/success
FINEXUS_RETURN_URL_FAIL=https://<your-domain>/payments/finexus/return/fail
FINEXUS_RETURN_URL_CANCEL=https://<your-domain>/payments/finexus/return/cancel
FINEXUS_QR_CALLBACK_URL=https://<your-domain>/payments/finexus/qr-callback
```

If you use the MPI, its properties file holds `MerchantID`, `SecretKey` and URLs. Do not commit it with the real key; generate it at deploy time or inject the key.

## Sandbox vs live

- Sandbox: docs examples use `https://sandbox.finexusgroup.com/upp/faces/...`; the MPI ships pointing at `https://dev.finexusgroup.com:10501/upp/...`.
- Live: host, MID, Secret Key and keys are issued by Finexus for production. Not documented in official docs (checked 2026-10-21): the production hostname and the go-live checklist.
- Static QR and Kayaaku POS key exchanges must be repeated for production.
- Production limits repeated requests per IP (1 minute block, extended by 1 minute per further attempt).

## Payment methods and activation

- Online (PayMaster UPP): Visa, Mastercard, UnionPay (`CUP`), and others by card type codes; MyDebit is named on the product page; FPX / DuitNow online banking (DuitNow Pay, `RPP`); DuitNow QR and Alipay+ QR; Google Pay through UPP when enabled by Finexus; card tokenisation and recurring; pre-auth.
- In person (Kayaaku POS): cards contactless or chip and PIN, DuitNow QR, Alipay, Alipay+, Visa Instalment Plan (from RM 300 per product page).
- Activation is done by Finexus per merchant. A self-serve toggle is Not documented in official docs (checked 2026-10-21).
- Plugins: WooCommerce (Subscriptions extension needed for recurring) and OpenCart.

## Pricing

See https://www.finexusgroup.com/businesses/payments. Do not copy rates into code or docs; confirm the merchant's contract.

## Settlement

- The product page mentions automated daily settlement, with instant settlement described as upcoming. The merchant portal shows settlements and reports.
- Reversal and void only before settlement cut-off (same day); refund after settlement or beyond one day.
- Settlement report API or file format: Not documented in official docs (checked 2026-10-21).

## Brand rules (dynamic QR)

Show "Powered by Finexus" on the QR screen, a countdown (5 minutes DuitNow QR, 3 minutes Alipay+), and follow the size rules on https://sandbox.finexusgroup.com/myxaas/docs/paymaster/dynamic-qr/design-qr-code and https://sandbox.finexusgroup.com/myxaas/docs/paymaster/brand-guidelines.
