# PayNet Account Setup

Checked 2026-10-21. Pricing: PayNet does not publish merchant fees in its developer docs. See `https://www.paynet.my` for product pages; fees are set by your acquirer (bank or licensed acquirer). Not documented in official docs (checked 2026-10-21) for merchant-level pricing.

## First decide your path

| You are | Path |
|---|---|
| An ordinary merchant (online shop, SaaS, clinic) | Do not onboard with PayNet. Sign up with an FPX acquiring bank, a DuitNow Pay acquirer, or a payment gateway that is a PayNet participant. PayNet: "To be a part of the DuitNow Pay platform, you will need to be affiliated with our merchant acquirers' network." Use the acquirer's own API. |
| A merchant or Third Party Acquirer (TPA) integrating FPX directly | FPX merchant onboarding through an FPX acquiring bank, then the FPX key and certificate steps below. |
| A bank, e-wallet or licensed acquirer (RPP participant) | Developer Portal project for DuitNow Pay, DuitNow QR, Online Banking/Wallets or other v3 APIs. |

## Developer Portal account (participants and FPX testers)

1. Open the Developer Portal `https://developer.paynet.my` and choose "Sign Up to Get Started" (PayNet SSO at `https://account.paynet.my`). Full SSO guide: `https://portal.paynet.my/home/user-guide/37175090424345/37606722574617`.
2. Fill in the registration form and click Register Now. Follow the email link, then set a new password on first login.
3. If your company is new to PayNet, email `ask@paynet.my` so PayNet can verify you and your company. "Your company must be verified by PayNet in order to proceed to go live (Product setup)."
4. If you cannot create a project or your product is not listed, ask your company administrator for the right role, or contact `ask@paynet.my`.

## Create and manage a project

1. Log in to the Developer Portal or PayNet One Stop Portal, then Dashboard, then Developer Portal, then My Projects, then Create Project.
2. Select product (for example DuitNow, MyDebit), API features, and role. For DuitNow Pay the role is Acquirer. Review and accept the Participant Agreement.
3. Invite team members under Manage Your Team Members. Only Owners and Administrators remove members; only the Owner deletes a project (not recoverable).

## Certification Centre (simulator stage)

1. Open View Test Suite. Fill Configuration Management (DuitNow "Pre-requisite Fields") and click Complete Configuration.
2. The simulator (DAVE) activates. Use the Server Details shown for endpoints and connectivity whitelisting.
3. Run test cases in order, not in parallel. Types: Pre-requisite, Mandatory, Optional, PayNet-Assisted; mock-fired cases are triggered by PayNet, self-triggered cases are sent by you.
4. Pass all Certification Centre cases. The portal issues a Completion ID and letter.

## System Verification (end-to-end UAT)

1. Click Request Access in the System Verification section.
2. Provide the BIC (Bank Identification Code) for the acquirer role.
3. Provide server configuration:

| Item | Purpose |
|---|---|
| CSR | PayNet signs and authenticates your certificate |
| Webhook URL | Where PayNet sends notifications |
| IP address range | What PayNet whitelists |

4. Wait for PayNet validation (typically about 3 working days).
5. Register a few test merchants: Template A for new merchants (new RPP profile), Template B for existing RPP merchants (adds a Product ID to the existing profile). Upload one file at a time.
6. Get PayNet's public key from "One-Time Configuration For Your Application" in the test suite.
7. Complete all System Verification test cases.

## Go live (production)

1. In Go live, click "Submit a production request" and give production CSR, webhook URL and IP range.
2. PayNet validates and emails you. "View Production Configuration Details" shows the production PayNet public key and lets you add more IPs.
3. Register production merchants (Templates A and B). Each merchant gets a Product ID used as `merchant.productId`. Edit merchant details later in the RPP Back Office portal.
4. Renew certificates through PayNet One Stop Portal Certificate Management (Renew appears 90 days before expiry). DuitNow v3 can use one centralised certificate across projects; production must use an authorised CA-signed certificate per Bank Negara Malaysia requirements.

## FPX merchant keys and certificate

1. Generate a 2048-bit RSA key and CSR on your server (placeholders, never commit the key):

```bash
openssl genrsa -out <EXCHANGE_ID>.key 2048
openssl req -out <EXCHANGE_ID>.csr -key <EXCHANGE_ID>.key -new -sha256
```

Leave the challenge password and optional company name blank.

2. UAT: self-signed certificates are accepted; PayNet encourages submitting through the Developer Portal Project section. PayNet's Security Administrator approves, uploads it to FPX Webview, and returns the certificate.
3. Production: PayNet approves and authorises; "Merchant will receive the new certificate from MSC Trustgate."
4. Download the FPX public certificate (`fpxuat.cer` for UAT, production certificate) from `/docs/fpx/resources`. For renewal the docs give one naming rule: "UAT: change from fpxuat.cer to fpxuat_current_cer" (the production rule is cut off on the page). The PHP sample checks two certificates (return codes 07 and 08 cover one or both expired), so keep the current and the new certificate side by side during the switch.
5. Exchange ID (`fpx_sellerExId`) and Seller ID (`fpx_sellerId`) are issued at FPX merchant registration. How a merchant registers for FPX with an acquiring bank: Not documented in official docs (checked 2026-10-21).

## Environment variables (placeholders)

```bash
# FPX
FPX_SELLER_EX_ID=EX<placeholder>
FPX_SELLER_ID=SE<placeholder>
FPX_PRIVATE_KEY_PATH=/secure/path/<EXCHANGE_ID>.key
FPX_PUBLIC_CERT_PATH=/secure/path/fpxuat.cer
FPX_BASE_URL=https://uat.mepsfpx.com.my/FPXMain

# DuitNow Pay (acquirer)
DUITNOW_PAY_BASE_URL=<provided by PayNet at onboarding>
DUITNOW_PAY_BIC=<your BIC>
DUITNOW_PAY_PROJECT_ID=<project id>
DUITNOW_PAY_PRIVATE_KEY_PATH=/secure/path/duitnow_pay_priv.key
DUITNOW_PAY_PAYNET_PUBLIC_KEY_PATH=/secure/path/paynet_pub.pem

# RPP v3
RPP_BIC=<your BIC>
RPP_JWT_KEY=<JWT credential key>
RPP_CERT_SERIAL=<certificate serial for kid>
RPP_PRIVATE_KEY_PATH=/secure/path/rpp_priv.key
```

Keep keys in a secret store, never in the repo.

## Contacts

- `ask@paynet.my`: onboarding, company verification, Developer Portal access.
- `commercial@paynet.my`: named in the v3 OpenAPI specs for access to pre-release APIs.
- Financial Reports API client ID and secret: request by email to the One Stop Portal support address (see `/docs/operations/financial-reports/tech-refresh/introduction`).
