# Malaysia Payment Gateway Skills

![Made in Malaysia](https://img.shields.io/badge/Made%20in-Malaysia-red)
![Agent Skills](https://img.shields.io/badge/Agent-Skills-blue)
![Payment Gateway](https://img.shields.io/badge/Payment-Gateway-green)

Agent skills for implementing Malaysian payment gateway integrations with server-side checkout, verified webhooks, and idempotent settlement.

Made from Malaysia for developers building FPX, DuitNow QR, card, e-wallet, and local payment flows.

## Choose a gateway first

Not sure which gateway to use? [Malaysia Payment Gateway (MPG)](https://malaysiapg.com.my/) compares the fees of 81 Malaysian payment gateways (FPX, card, e-wallet, DuitNow QR, BNPL), taken from each provider's official pricing page. It has a free [JSON API](https://malaysiapg.com.my/api/gateways.json) and an MCP server (`https://mcp.malaysiapg.com.my/mcp`) your agent can query. Each skill below links to that gateway's fee page.

## Skills

| Skill | Use for | Fees |
| --- | --- | --- |
| `malaysia-payment-gateway` | Gateway-agnostic planning, review, and security checklist | [compare all](https://malaysiapg.com.my/gateways/) |
| `setup-chip` | CHIP Collect checkout, purchases, callbacks, refunds, payment methods | [fees](https://malaysiapg.com.my/gateway/chip/) |
| `setup-curlec` | Curlec/Razorpay Orders API, Checkout.js, callbacks, webhooks | [fees](https://malaysiapg.com.my/gateway/curlec/) |
| `setup-xendit` | Xendit Payment Sessions, Payment Requests, webhooks, xenPlatform | [fees](https://malaysiapg.com.my/gateway/xendit/) |
| `setup-bayarcash` | Bayarcash API v2/v3, payment intents, callbacks, checksums | [fees](https://malaysiapg.com.my/gateway/bayarcash/) |
| `setup-bcl` | BCL Pay, Payment Link, QR Terminal, Forms, Bayarcash-linked setup | [fees](https://malaysiapg.com.my/gateway/bcl/) |
| `setup-toyyibpay` | toyyibPay categories, bills, callbacks, DuitNow QR, transaction checks | [fees](https://malaysiapg.com.my/gateway/toyyibpay/) |
| `setup-billplz` | Billplz collections, bills, X Signature callbacks, Payment Order payouts | [fees](https://malaysiapg.com.my/gateway/billplz/) |
| `setup-fiuu` | Fiuu hosted payment, Seamless Integration, Inpage Checkout, XDK, callbacks | [fees](https://malaysiapg.com.my/gateway/fiuu/) |
| `setup-senangpay` | senangPay hosted/manual checkout, Direct API, callbacks, query status, refunds, plugins, recurring, payout, tokenisation, split settlement | [fees](https://malaysiapg.com.my/gateway/senangpay/) |
| `setup-hitpay` | HitPay Payment Request API, hosted checkout, Drop-In UI, webhooks, refunds, plugins, recurring billing, payouts | [fees](https://malaysiapg.com.my/gateway/hitpay/) |
| `setup-stripe-malaysia` | Stripe Malaysia Checkout, Payment Element, webhooks, Stripe CLI, MYR, cards, FPX | [fees](https://malaysiapg.com.my/gateway/stripe-malaysia/) |
| `setup-2c2p` | PGW v4.3/v4.5 Payment Token API with JWT HS256, hosted redirect checkout, Direct API Do Payment, Secure Pay JS, Drop-In UI | [fees](https://malaysiapg.com.my/gateway/2c2p/) |
| `setup-adyen` | Checkout API v72 (/sessions, /paymentMethods, /payments, /payments/details), Hosted Checkout, Web Drop-in and Components, HMAC-SHA256 signed webhooks, captures | [fees](https://malaysiapg.com.my/gateway/adyen/) |
| `setup-affanpay` | Bearer API tokens, Create Bill, hosted bill payment page redirect, callback_url webhooks, X-AFFANPAY-HMAC HMAC-SHA256 signatures with X-AFFANPAY-TIMESTAMP | [fees](https://malaysiapg.com.my/gateway/affanpay/) |
| `setup-ahapay` | External API V2 (and legacy V1), X-ApiKey auth, Create Default Order hosted checkout, POS and QR POS orders, online_order_payment_link redirect | [fees](https://malaysiapg.com.my/gateway/ahapay/) |
| `setup-airwallex` | Payment Intents API, Hosted Payment Page, Checkout Sessions, Drop-in and Embedded Elements (Airwallex.js), Native API | [fees](https://malaysiapg.com.my/gateway/airwallex/) |
| `setup-antom` | One-time Payments pay and createPaymentSession APIs, hosted and embedded Checkout Page, Payment Element, API-only redirects, RSA256 request signing and response or notification signature checks | [fees](https://malaysiapg.com.my/gateway/antom/) |
| `setup-asiapay` | PayDollar hosted payForm, Direct Connection, SHA-1/SHA-256 secureHash, datafeed verification with OK acknowledgement, Merchant API | [fees](https://malaysiapg.com.my/gateway/asiapay/) |
| `setup-atome` | HTTP Basic auth, create payment, redirectUrl and appPaymentUrl checkout, server-to-server callbacks, X-Signature HMAC-SHA256 callback verification | [fees](https://malaysiapg.com.my/gateway/atome/) |
| `setup-axaipay` | eComPay hosted payment page (HTML form POST), bypass payment page with FPX acquirer codes, DuitNow QR and Alipay+ QR code generation, HMAC-SHA512 Base64 request and response signatures, redirect URL and backend URL callbacks | [fees](https://malaysiapg.com.my/gateway/axaipay/) |
| `setup-bayarlah` | Upay Web API, terminal activation and check-in, MD5 request signing, DuitNow QR and WeChat Pay, RSA callbacks, refunds | [fees](https://malaysiapg.com.my/gateway/bayarlah/) |
| `setup-betterpay` | PRO and LITE accounts, Standard and Direct Payment, MD5 and HMAC-SHA256 hashing, callbacks, recurring, tokenization | [fees](https://malaysiapg.com.my/gateway/betterpay/) |
| `setup-boldpay` | hosted PaymentRequest redirect, MD5 AccessToken, CallbackURL browser return, NotificationURL server POST, QueryRequest status requery | [fees](https://malaysiapg.com.my/gateway/boldpay/) |
| `setup-cardbizpay` | Merchant API hosted payment page (Payment/Pymt), Payment Return URL callbacks, HMAC-SHA512 base64 signatures, payment enquiry, void | [fees](https://malaysiapg.com.my/gateway/cardbizpay/) |
| `setup-commercepay` | TokenAuth bearer tokens, Abp-TenantId header, cap-signature HMAC-SHA256 request signing, Direct Integration RequestPayment, Hosted Session Checkout InitialSession | [fees](https://malaysiapg.com.my/gateway/commercepay/) |
| `setup-eghl` | the Payment.aspx hosted payment page, SALE/AUTH requests, SHA-256 HashValue and HashValue2 checks, MerchantReturnURL and MerchantCallBackURL handling, the "OK" callback acknowledgement | [fees](https://malaysiapg.com.my/gateway/eghl/) |
| `setup-ezypay` | OAuth 2.0 password-grant tokens, merchant header, customers, Payment Capture Page iframe tokenisation, on-demand invoices | [fees](https://malaysiapg.com.my/gateway/ezypay/) |
| `setup-fasspay` | Online hosted payment page, HMAC-SHA256 X-Signature, callback verification, status check, void/capture, Fasstap in-person SDKs | [fees](https://malaysiapg.com.my/gateway/fasspay/) |
| `setup-favepay` | FavePay Omni (FPO) API, hosted checkout, dynamic and static QR, merchant scan, signed webhooks, refunds | [fees](https://malaysiapg.com.my/gateway/favepay/) |
| `setup-finexus` | PayMaster UPP hosted page, PayMaster API and MPI, HMAC-SHA256 secure hash, payment query, reversal | [fees](https://malaysiapg.com.my/gateway/finexus/) |
| `setup-gkash` | host-to-host checkout (POST /api/payment/form), legacy hosted payment form, SHA512 signatures, server-side callback with "OK" acknowledgement, return URL handling | [fees](https://malaysiapg.com.my/gateway/gkash/) |
| `setup-gomobi` | Mobi hosted checkout (HMAC-SHA256), AES-CBC payload encryption, FPX B2C/B2B, cards with saved tokens, e-wallets, payouts | [fees](https://malaysiapg.com.my/gateway/gomobi/) |
| `setup-grab-paylater` | the One Time Charge (OTC) Partner API, GrabID OAuth 2.0 with PKCE, partner HMAC-SHA256 signatures, X-GID-AUX-POP proof-of-possession signatures, charge init and complete | [fees](https://malaysiapg.com.my/gateway/grab-paylater/) |
| `setup-herepay` | the V1 direct API, Payment Channels, Initiate Invoice, callback, checksum (HMAC-SHA256 over comma-joined values sorted by key) | [fees](https://malaysiapg.com.my/gateway/herepay/) |
| `setup-ipay88` | OPSG hosted payment, HMAC-SHA512 signatures, BackendURL callbacks with RECEIVEOK, requery webservice (SOAP/POST), signature tester | [fees](https://malaysiapg.com.my/gateway/ipay88/) |
| `setup-kineticpay` | FPX bill creation via /payment/create, auto-submit HTML checkout, callback_success/callback_error/callback_status URLs, server-side status requery via /payment/status, merchant key from the Collection page | [fees](https://malaysiapg.com.my/gateway/kineticpay/) |
| `setup-kiple` | Hosted payment form, SHA-1 merchant_hashvalue and ord_key verification, return and callback URLs, enquiry requery, recurring | [fees](https://malaysiapg.com.my/gateway/kiple/) |
| `setup-leanx` | auth-token header, optional X-Signature HMAC-SHA256 request signing, Collections and Bills, hosted payment page (create-bill-page), own-page silent bills (create-bill-silent) with payment_service_id | [fees](https://malaysiapg.com.my/gateway/leanx/) |
| `setup-lekirpay` | OAuth2 client_credentials access tokens, Send Payment and Secure Send Payment APIs, Lekir-Signature HMAC-SHA256 checksum, payment_url hosted checkout, callback_url notifications | [fees](https://malaysiapg.com.my/gateway/lekirpay/) |
| `setup-luminarix` | v1 payments API (deposit, transaction-status, refund), hosted checkout, X-Signature HMAC-SHA256, signed callbacks | [fees](https://malaysiapg.com.my/gateway/luminarix/) |
| `setup-m1pay` | Keycloak OAuth2 client-credentials tokens, the M1Pay Payment Wall transaction API, individual payment channels, FPX bank list API, Get Transaction status requery | [fees](https://malaysiapg.com.my/gateway/m1pay/) |
| `setup-maybank2u-pay` | JavaScript and PHP SDK source, getEncryptionString send-string building, AES-128-ECB double encryption, envType playground/UAT/production action URLs, m2upay.initPayment pop-up form post (q and i fields) | [fees](https://malaysiapg.com.my/gateway/maybank2u-pay/) |
| `setup-moby` | Bearer token auth, hosted checkout payLink, Direct API, FPX, HMAC-SHA256 callbacks, recurring, refunds | [fees](https://malaysiapg.com.my/gateway/moby/) |
| `setup-omise` | Charges API, Sources API, Tokens API on vault.omise.co, Omise.js, 3-D Secure | [fees](https://malaysiapg.com.my/gateway/omise/) |
| `setup-oxpay` | Payments API v2, Hosted Payment Page, Create Payment, void, refund, HMAC-SHA256 signed webhooks | [fees](https://malaysiapg.com.my/gateway/oxpay/) |
| `setup-paycools` | WebPay hosted checkout, DuitNow dynamic QR, payment notifications, RSA SHA256withRSA signatures over the param string, appId and RSA key exchange | [fees](https://malaysiapg.com.my/gateway/paycools/) |
| `setup-paydee` | MPI spec v5, per-session RSA key exchange, hosted (mercReq) and non-hosted (mpReq) payments, void, refund, recurring | [fees](https://malaysiapg.com.my/gateway/paydee/) |
| `setup-paydibs` | PAY request hosted checkout form post to /PPG/trigger, SHA512 Sign hashing, MerchantRURL browser redirect, MerchantCallbackURL server-to-server callback, QUERY status inquiry | [fees](https://malaysiapg.com.my/gateway/paydibs/) |
| `setup-payhalal` | hosted payment page V1 and V2 payment link API, QR Code API, SHA256 request hash, callback hash verification, return/success/cancel/callback URLs | [fees](https://malaysiapg.com.my/gateway/payhalal/) |
| `setup-paynet` | FPX direct and TPA messages, DuitNow Pay (payment intent, consent, AutoDebit, refunds, webhooks), RPP DuitNow, DuitNow QR strings | [fees](https://malaysiapg.com.my/gateway/paynet/) |
| `setup-paypal` | REST API OAuth 2.0, Orders v2 create/capture/authorize, Payments v2 refunds/voids/reauthorization, JavaScript SDK v6 and v5 buttons, webhooks with CRC32 plus SHA256withRSA signature verification or verify-webhook-signature postback | [fees](https://malaysiapg.com.my/gateway/paypal/) |
| `setup-payright` | the Collection and Bill APIs, FPX bank (payment provider) selection, hosted payment_url redirect, JWT (HS256) encoded callbacks decoded with the Hash Key, Get Bill status requery | [fees](https://malaysiapg.com.my/gateway/payright/) |
| `setup-qashier` | QashierPay Online Checkout Session API, hosted checkout redirect, HTTP Basic Auth API keys, Idempotency-Key, X-Qashier-Signature HMAC-SHA256 webhooks | [fees](https://malaysiapg.com.my/gateway/qashier/) |
| `setup-revenue-monster` | OAuth 2.0 client credentials, RSA SHA256 request signatures (X-Signature, X-Nonce-Str, X-Timestamp), hosted Web Payment checkout (v3/payment/online), Direct Payment Checkout, notify URL callbacks | [fees](https://malaysiapg.com.my/gateway/revenue-monster/) |
| `setup-riipay` | GET redirect payment requests, return and server callbacks, MD5 request and response signatures, transaction query | [fees](https://malaysiapg.com.my/gateway/riipay/) |
| `setup-ringgitpay` | Hosted payment page form POST, SHA-256 checksums, server callbacks, enquiry requery, Direct Debit e-Mandate | [fees](https://malaysiapg.com.my/gateway/ringgitpay/) |
| `setup-securepay` | API v1 FPX payments, Payment Session, API v2 payment intents, HMAC-SHA256 checksums, callback and redirect URLs | [fees](https://malaysiapg.com.my/gateway/securepay/) |
| `setup-setel` | Pay with Setel online checkout sessions, payment intents, direct capture or hold-and-capture, webhook signature verification (HMAC-SHA256 keyed with SHA-256 of x-api-secret), session requery | [fees](https://malaysiapg.com.my/gateway/setel/) |
| `setup-share-commerce` | hosted Payment Request redirect, Seemless (iframe) payment, Enquiry requery, same-day Payment Void, Pre-Auth/Capture | [fees](https://malaysiapg.com.my/gateway/share-commerce/) |
| `setup-shopeepay-gateway` | Checkout with ShopeePay (jump app / redirect), Merchant Presented Mode dynamic QR, Customer Presented Mode scan, hosted ShopeePay Payment Gateway checkout sessions, Account Linking | [fees](https://malaysiapg.com.my/gateway/shopeepay-gateway/) |
| `setup-spaylater` | Payment Gateway v1 hosted checkout with allowed_payment_method spay_later, Checkout with ShopeePay v3 with preferred_payment_method_type spay_later, X-Airpay-ClientId and X-Airpay-Req-H HMAC-SHA256 Base64 signatures, Notify Transaction Status callbacks, Check Transaction Status polling | [fees](https://malaysiapg.com.my/gateway/spaylater/) |
| `setup-tng-ewallet` | Mini-program /v1/payments/pay, my.tradePay, RSA256 signatures, notifyPayment callbacks, inquiryPayment, refunds, web checkout | [fees](https://malaysiapg.com.my/gateway/tng-ewallet/) |
| `setup-tulus` | /v2/checkout signed order POST, hosted pay URLs, truncated HMAC-SHA256 signatures, signed callbacks, Report API, DuitNow QR | [fees](https://malaysiapg.com.my/gateway/tulus/) |

### Not covered (no public developer docs)

These gateways keep their integration docs behind a merchant sign-up or a sales contact, so there is no skill for them yet. Ask the provider for its integration guide, then use `malaysia-payment-gateway` for the review checklist: Alliance Bank, AmBank, Ampersand Pay, Boost, CardUp, CIMB, Hong Leong Bank, HSBC Omni Collect, J&C Pacific, ManagePay (MPay), OCBC, Public Bank, RevPay, RHB, SMAP Pay, SwiF, Syntrino, UOB.

## Install

Before installing:

1. Check current installed version.
2. Verify your agent supports Skills.
3. Update npm if needed.

Install from this repository:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway
```

The CLI can show available skills and let you select them interactively.

List available skills:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --list
```

Install all skills:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --all
```

Install one skill:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --skill setup-curlec
```

Install globally:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway -g
```

Verify CLI version:

```bash
npx skills@latest --version
```

## Recommended Setup

1. Install the overview skill first:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --skill malaysia-payment-gateway
```

2. Install the provider you need:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --skill setup-curlec
```

3. Ask your agent to inspect your project before coding.

4. Ask your agent to implement checkout, webhook verification, settlement, tests, and docs together.

## How To Ask Your Agent

Use the skill name in your prompt:

```text
Use $malaysia-payment-gateway to review my app payment flow and tell me what is missing before production.
```

```text
Use $setup-curlec to implement Curlec checkout, callback verification, webhook settlement, and tests in this Next.js project.
```

```text
Use $setup-chip to add CHIP Collect checkout and webhook handling. Do not unlock access from redirect alone.
```

```text
Use $setup-xendit to implement hosted checkout with Xendit Payment Sessions and idempotent webhook settlement.
```

```text
Use $setup-bayarcash to integrate Bayarcash payments with checksum/callback verification.
```

```text
Use $setup-bcl to set up BCL Pay and explain which parts are BCL-owned versus Bayarcash-linked.
```

```text
Use $setup-toyyibpay to integrate toyyibPay bills, callback hash verification, and idempotent settlement.
```

```text
Use $setup-billplz to integrate Billplz bills, X Signature verification, and idempotent settlement.
```

```text
Use $setup-fiuu to integrate Fiuu hosted checkout, Notify URL verification, Callback URL handling, and idempotent settlement.
```

```text
Use $setup-senangpay to integrate senangPay hosted checkout, Callback URL verification, query status reconciliation, and idempotent settlement.
```

```text
Use $setup-hitpay to integrate HitPay Payment Request checkout, webhook signature verification, status fallback, and idempotent settlement.
```

```text
Use $setup-stripe-malaysia to integrate Stripe Malaysia checkout, webhook verification, and idempotent settlement.
```

```text
Use $setup-2c2p to integrate 2C2P checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-adyen to integrate Adyen checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-affanpay to integrate AffanPay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-ahapay to integrate AhaPay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-airwallex to integrate Airwallex checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-antom to integrate Antom checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-asiapay to integrate AsiaPay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-atome to integrate Atome checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-axaipay to integrate Axaipay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-bayarlah to integrate bayarlah checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-betterpay to integrate Betterpay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-boldpay to integrate BOLD.Pay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-cardbizpay to integrate CardBizPay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-commercepay to integrate CommercePay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-eghl to integrate eGHL checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-ezypay to integrate Ezypay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-fasspay to integrate Fasspay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-favepay to integrate FavePay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-finexus to integrate Finexus checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-gkash to integrate Gkash checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-gomobi to integrate GoMobi checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-grab-paylater to integrate GrabPay & PayLater checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-herepay to integrate Herepay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-ipay88 to integrate iPay88 checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-kineticpay to integrate KineticPay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-kiple to integrate Kiple checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-leanx to integrate LeanX checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-lekirpay to integrate LekirPay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-luminarix to integrate Luminarix checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-m1pay to integrate M1Pay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-maybank2u-pay to integrate Maybank2u Pay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-moby to integrate MOBY checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-omise to integrate Omise checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-oxpay to integrate OxPay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-paycools to integrate PayCools checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-paydee to integrate Paydee checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-paydibs to integrate Paydibs checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-payhalal to integrate Payhalal checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-paynet to integrate PayNet checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-paypal to integrate PayPal checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-payright to integrate Payright checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-qashier to integrate Qashier checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-revenue-monster to integrate Revenue Monster checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-riipay to integrate Riipay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-ringgitpay to integrate RinggitPay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-securepay to integrate SecurePay checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-setel to integrate Setel checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-share-commerce to integrate Share Commerce checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-shopeepay-gateway to integrate ShopeePay Gateway checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-spaylater to integrate SPayLater checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-tng-ewallet to integrate Touch 'n Go eWallet checkout, verify its callback, and settle orders idempotently.
```

```text
Use $setup-tulus to integrate Tulus checkout, verify its callback, and settle orders idempotently.
```

## Agent Rules

Ask the agent to:

- Read existing code before adding new files.
- Read provider `references/account-setup.md` before explaining registration, credentials, dashboard setup, or required API keys.
- Use provider `references/account-setup.md` for framework setup notes, required env vars, and deployment secrets where documented.
- When referral or invite guidance exists, present it as optional and transparent; never imply it is required unless provider registration actually requires an invite.
- Keep gateway secrets server-side.
- Create a local pending payment before provider checkout.
- Verify webhook/callback signatures.
- Re-fetch payment state from the provider when possible.
- Match provider, gateway reference, amount, currency, and local payment status before marking paid.
- Make settlement idempotent.
- Add tests for invalid signatures, duplicate events, wrong amount/currency, and successful settlement.

## Production Checklist

- Required environment variables are documented.
- Database migrations are included.
- Webhook URLs are configured in sandbox and live dashboards.
- Sandbox payment succeeds.
- Duplicate webhook replay is harmless.
- Fake success redirect cannot unlock paid access.
- Live and sandbox keys are separated.

## Contribute A New Gateway

Contributions are welcome. Use one folder per skill.

### Step 1: Pick a skill name

Use action-style names:

```text
setup-toyyibpay
setup-senangpay
setup-stripe-malaysia
```

Rules:

- Lowercase only.
- Use hyphens.
- Keep it short.
- Folder name must match `name:` in `SKILL.md`.

### Step 2: Create the folder

```bash
mkdir -p setup-yourgateway/references setup-yourgateway/agents
```

### Step 3: Add `SKILL.md`

Minimum shape:

```markdown
---
name: setup-yourgateway
description: Set up, build, debug, review, and explain YourGateway payment integrations. Use when working with checkout, payment links, callbacks, webhooks, signatures, refunds, sandbox/live setup, and payment settlement for YourGateway.
---

# Setup YourGateway

Use this skill for YourGateway integration work.

## Source

Read `references/yourgateway.md` before giving factual API guidance or writing integration code.

## Core Workflow

1. Identify environment: sandbox or production.
2. Create local pending payment first.
3. Create checkout/session server-side.
4. Verify callback/webhook signature.
5. Re-fetch payment state from provider when possible.
6. Settle idempotently after amount, currency, provider, and reference match.
```

### Step 4: Add `agents/openai.yaml`

```yaml
interface:
  display_name: "Setup YourGateway"
  short_description: "Set up YourGateway payments"
  default_prompt: "Use $setup-yourgateway to integrate YourGateway payments."
```

### Step 5: Add references

Create:

```text
setup-yourgateway/references/yourgateway.md
```

Include:

- Official docs URLs.
- Environment base URLs.
- Auth method.
- Checkout/session endpoint.
- Webhook/callback signature rules.
- Amount unit rules.
- Sandbox test notes.
- Common failure fixes.

Do not include real API keys, tokens, merchant secrets, or private customer data.

### Step 6: Validate

Run validator:

```bash
python3 /home/user/.codex/skills/.system/skill-creator/scripts/quick_validate.py setup-yourgateway
```

If you do not have that validator, install the latest Skills CLI and at minimum verify:

```bash
npx skills@latest --version
```

### Step 7: Test install locally

From a separate test directory:

```bash
npx skills@latest add /path/to/malaysia-payment-gateway --skill setup-yourgateway
```

### Step 8: Open pull request

```bash
git checkout -b add-setup-yourgateway
git add setup-yourgateway
git commit -m "Add setup-yourgateway skill"
git push origin add-setup-yourgateway
```

Then open a pull request with:

- Gateway name.
- What products/features are covered.
- Official docs source links.
- Validation result.
- Any known gaps.
