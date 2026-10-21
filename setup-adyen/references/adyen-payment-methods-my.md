# Adyen Malaysian Payment Methods (web)

Use this file to offer Malaysian local methods through Adyen Checkout: FPX, DuitNow QR, Touch 'n Go, GrabPay, Boost, Atome, Alipay, Alipay+, WeChat Pay, plus notes on cards and wallets. Web only (Drop-in, Components, API-only). Checked 2026-10-21 against the pages named under each section.

7-Eleven Malaysia: the only official mention is the variant `molpay_cash` ("7-Eleven via MOLPay") in `https://docs.adyen.com/development-resources/paymentmethodvariant`; there is no integration page, so treat it as Not documented (checked 2026-10-21) and ask Adyen. Other MOLPay variants listed there: `molpay_ebanking_fpx_MY`, `molpay_ebanking_MY`, `molpay_fpx` (Online banking Malaysia), `molpay_boost`, `molpay_alipay`, `molpay_points` (RazerGold).

Pay by Link supports (Customer Area and API) Alipay, Apple Pay (Adyen certificate only), Atome, cards, Google Pay (add `eu.adyen.link` in Google console), GrabPay and MOLPay (FPX). Touch 'n Go, Boost and DuitNow are not in that list. Source: `https://docs.adyen.com/unified-commerce/pay-by-link/supported-payment-methods`

Settlement currencies per the payment methods overview `https://docs.adyen.com/payment-methods`: FPX, Boost, Touch 'n Go MYR or USD; GrabPay MYR, SGD, USD; Atome MYR, SGD, USD; DuitNow not stated.

Not found in official docs (checked 2026-10-21): per-method settlement delays (except WeChat Pay), FPX minimum and maximum amounts.

## 0. Cross-method summary


| Method | `paymentMethod.type` | Country / currency | Flow (`action.type`) | Recurring | Refund full / partial / multiple partial | Separate capture | Chargebacks | Shopper input | Enable how |
|---|---|---|---|---|---|---|---|---|---|
| FPX Online Banking Malaysia | `molpay_ebanking_fpx_MY` (+ required `issuer`) | MY / MYR | redirect | No | Yes / Yes / Yes | No | No | Bank (issuer) picker | Add in Customer Area |
| DuitNow (QR only) | `duitnow` | MY / MYR | Additional action (QR code), no redirect | No | Yes / Yes / Yes | No | No | None | Contact Support |
| Touch 'n Go eWallet | `touchngo` | MY / MYR | redirect | No | Yes / Yes / Yes | No | No | None | Contact Support |
| GrabPay | `grabpay_MY` (also `grabpay_SG`, `grabpay_PH`) | MY, SG, PH / MYR, SGD, PHP | redirect | No | Yes / Yes / Yes | No | No | None (PayLater needs extra request fields) | Add in Customer Area |
| Boost Wallet | `molpay_boost` | MY / MYR | redirect | No | Yes / **No** / **No** | No | No | None | Add in Customer Area |
| Atome (BNPL) | `atome` | MY, SG / MYR, SGD | redirect | No | Yes / Yes / Yes | No (auto-captured) | No | Name, email, phone, billing address | Add in test CA; live via Support |
| Alipay | `alipay` | International incl. MY / incl. MYR | redirect (table says flow "None") | No | Yes / Yes / Yes | No | No | None | Add in Customer Area |
| Alipay+ | `alipay_plus` (resolves to `alipay_plus_<wallet>`) | International incl. MY / incl. MYR | redirect | No (tokenization rejected) | Yes / Yes / Yes | No | No | None | Add in Customer Area; wallet onboarding SLA 1 month |
| WeChat Pay web (desktop QR / mobile H5) | `wechatpayWeb` | International / AUD, CAD, CNY, EUR, GBP, HKD, JPY, NZD, SGD, USD (no MYR listed) | redirect (QR on desktop) | No | Yes / Yes / (n/a in table) | n/a | No | None | Add in Customer Area; mobile browser flow needs Support enrolment |

Sources (support tables): https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia, https://docs.adyen.com/payment-methods/duitnow, https://docs.adyen.com/payment-methods/touchngo, https://docs.adyen.com/payment-methods/grabpay, https://docs.adyen.com/payment-methods/boost, https://docs.adyen.com/payment-methods/atome, https://docs.adyen.com/payment-methods/alipay, https://docs.adyen.com/payment-methods/alipay-plus, https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-desktop-qr-payments

Common rules across the method pages:
- Endpoints in the examples: `POST https://checkout-test.adyen.com/v72/paymentMethods`, `POST .../v72/payments`, `POST .../v72/payments/details`, `POST .../v72/sessions` (some pages still show v68, v69 or v71 in examples). Headers: `x-api-key: <API_KEY>`, `content-type: application/json`, `idempotency-key: <UUID>`. Source: https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia/api-only
- Sessions flow Drop-in/Components for these methods require Checkout API v68 or later. Source: https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia/web-drop-in
- None of these methods support recurring/tokenization, separate captures, partial captures or chargebacks (per the support tables above).
- Redirect handling: redirect the shopper to `action.url` (method GET). On return, call `POST /payments/details` with `details.redirectResult` (URL-decoded). Use `resultCode` for the shopper, but rely on the AUTHORISATION webhook for the final result. Source: https://docs.adyen.com/payment-methods/touchngo/api-only
- Testing: every method except WeChat Pay has the "Adyen Payment Simulator", on by default once the method is added in the test Customer Area. Redirect methods: shopper lands on an Adyen-hosted simulator page, you pick the outcome, simulator returns shopper, then (Advanced flow) call `/payments/details`. Check result in Customer Area > Payments > Payment list. The simulator does not reproduce the real wallet/bank journey. Source: https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia/api-only
- No method page here states min/max amounts except Atome, and none states settlement delay except WeChat Pay ("funds in 2-3 days"). For all others: Not documented in the method pages.

## 1. FPX Online Banking Malaysia

Source: https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia
- FPX from PayNet, offered through partner Razer Merchant Services (RMS), formerly MOLPay.
- Shopper is redirected to their bank's website. Result is asynchronous: rely on webhooks to know whether payment succeeded.
- MY only, MYR only. Refunds, partial refunds, multiple partial refunds: yes. Recurring, separate/partial captures, chargebacks: no.
- Min/max amount, settlement delay, offer expiry: Not documented in the method pages.

Source: https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia/api-only
- Requirements: existing API-only integration with redirect handling (`action.type`: `redirect`); add FPX in Customer Area first.
- Payment form: show a bank list; pass the selected **Issuer** value to the server.
- Compliance rule (must): "All issuers in this list must be included on your checkout to be compliant with PayNet local regulations." Do not hide banks.
- Full issuer list (copy verbatim, 18 banks):

| Bank name | Issuer ID |
|---|---|
| Affin Bank | `fpx_abb` |
| Agro Bank | `fpx_agrobank` |
| Alliance Bank | `fpx_abmb` |
| Am Bank | `fpx_amb` |
| Bank Islam | `fpx_bimb` |
| Bank Muamalat | `fpx_bmmb` |
| Bank Rakyat | `fpx_bkrm` |
| Bank Simpanan Nasional | `fpx_bsn` |
| CIMB Bank | `fpx_cimbclicks` |
| Hong Leong Bank | `fpx_hlb` |
| HSBC Bank | `fpx_hsbc` |
| Kuwait Finance House | `fpx_kfh` |
| Maybank | `fpx_mb2u` |
| OCBC Bank | `fpx_ocbc` |
| Public Bank | `fpx_pbb` |
| RHB Bank | `fpx_rhb` |
| Standard Chartered Bank | `fpx_scb` |
| UOB Bank | `fpx_uob` |

(Same list is repeated on https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia/web-component)

- The issuer list is also returned in `/paymentMethods` under the entry with `type`: `molpay_ebanking_fpx_MY`, as `issuers: [{ "id": "fpx_abb", "name": "Affin Bank" }, ...]`. Prefer the live list from `/paymentMethods` at runtime; the static table is a fallback.
- Logos for RMS and the issuing banks can be downloaded (see the "Downloading logos" section of the API-only integration guide).
- `/paymentMethods` request: `countryCode`: `MY`, `amount.currency`: `MYR` (example also sends `shopperLocale`: `my-MY`, `amount.value`: 1000).
- `/payments` request: no extra parameters beyond the standard ones. Body:
  ```json
  {
    "amount": { "currency": "MYR", "value": 1000 },
    "paymentMethod": { "type": "molpay_ebanking_fpx_MY", "issuer": "fpx_abb" },
    "reference": "<ORDER_REF>",
    "returnUrl": "https://<your-site>/checkout?shopperOrder=...",
    "merchantAccount": "<MERCHANT_ACCOUNT>"
  }
  ```
- Response: `resultCode`: `RedirectShopper`, `action`: `{ "paymentMethodType": "molpay_ebanking_fpx_MY", "method": "GET", "url": "https://checkoutshopper-test.adyen.com/checkoutshopper/checkoutPaymentRedirect?redirectData=...", "type": "redirect" }`.
- OpenAPI v72 check: schema `MolPayDetails` requires both `type` and `issuer`; `type` enum is `molpay_ebanking_fpx_MY` or `molpay_ebanking_TH`. (Source: CheckoutService-v72.json, github.com/Adyen/adyen-openapi)
- Testing: (a) Adyen Payment Simulator, type `molpay_ebanking_fpx_MY`; (b) "Test directly with Online banking Malaysia": make a test payment with an issuer ID available for the component in that country. No specific test bank credentials are published. Check Customer Area (ca-test) > Payments > Payment list.
- Go live: submit a request for Online banking Malaysia in the live Customer Area (ca-live.adyen.com).

Source: https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia/web-component
- Import `MolPayEBankingMY` from `@adyen/adyen-web`. Initialize: `new MolPayEBankingMY(checkout, config).mount('#molPayEBankingMY-container')`.
- Optional config: `issuer` (preselect a bank ID), `highlightedIssuers` (IDs shown at top of the dropdown, added in Web v5.1.0), `showImage` (`false` removes bank logos), `placeholder` (dropdown text; custom translations override it).
- Sessions and Advanced flow: no extra `/sessions` or `/payments` parameters.

Source: https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia/web-drop-in
- Drop-in: no FPX-specific configuration; optional tree-shaking import of `MolPayEBankingMY` (`import { AdyenCheckout, Dropin, MolPayEBankingMY } from '@adyen/adyen-web'`).

Source: https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia/web
- Index page only; links to Drop-in and Components (Sessions and Advanced flow guides).

## 2. DuitNow (QR)

Source: https://docs.adyen.com/payment-methods/duitnow
- DuitNow is PayNet's Malaysian real-time payments network. Adyen only offers DuitNow for QR code payments.
- Payment type "Real-time payments", flow "Additional action", MY / MYR. Refunds, partial, multiple partial: yes. Recurring, captures, chargebacks: no.
- Pricing and settlement are only on adyen.com (not in docs). Min/max, QR expiry: Not documented in the method pages.
- No API-only page for DuitNow in this group. Not documented in the method pages: the raw `/payments` response shape for DuitNow (expect an additional action of QR type, per the "Additional action" flow; confirm in the API-only guide).

Source: https://docs.adyen.com/payment-methods/duitnow/web-component
- Setup: contact Adyen Support to add DuitNow in the (test) Customer Area. Live: contact Support again to add it in the live Customer Area.
- No additional request fields for `/sessions` or `/payments`.
- v6: `import { AdyenCheckout, DuitNow } from '@adyen/adyen-web'`; `new DuitNow(checkout).mount('#duitnow-container')`. v5: `checkout.create('duitnow')`.
- Testing (simulator), type `duitnow`: your integration shows a QR code; scan it as the shopper, choose the outcome on the Adyen-hosted simulator page; then **wait for the webhook** with the final outcome. "There is no shopper redirect and no `/payments/details` request." Drop-in refreshes automatically with the selected outcome.
- Direct testing: "with real payment details and small amounts".

Source: https://docs.adyen.com/payment-methods/duitnow/web-drop-in
- Drop-in: no DuitNow-specific configuration; v6 import `DuitNow`.

Source: https://docs.adyen.com/payment-methods/duitnow/web
- Index page only.

## 3. Touch 'n Go eWallet

Source: https://docs.adyen.com/payment-methods/touchngo
- Desktop: shopper redirected to a browser page showing a QR code, scans with Touch 'n Go app, confirms in app.
- Mobile: redirected to the Touch 'n Go app; returns to your shop after confirming. Without the app installed: redirected to a TNG mobile web page to enter phone number and PIN.
- Wallets, Redirect, MY / MYR. Refunds, partial, multiple partial: yes. Recurring, captures, chargebacks: no.

Source: https://docs.adyen.com/payment-methods/touchngo/api-only
- Setup: contact Support to add it in your Customer Area (page has a blank method name in that sentence; it means Touch 'n Go).
- `/paymentMethods`: `countryCode` `MY`, `amount.currency` `MYR`, `amount.value`. No shopper input needed.
- `/payments`: `paymentMethod.type`: `touchngo` plus standard `merchantAccount`, `reference`, `amount`, `returnUrl`. (Example uses `/v68/payments`.)
- Response: `resultCode` `RedirectShopper`, `action.type` `redirect`, `action.method` `GET`, `action.url`.
- Redirect guidance: in apps, use SFSafariViewController / Chrome Custom Tabs, not WebView; for mobile integrations redirect to the device default browser (best for app-to-app switching and MFA).
- After return: `POST /v72/payments/details` with `{"details":{"redirectResult":"<URL-decoded value>"}}`; response gives `resultCode` and `pspReference`.
- resultCode values:
  - `Authorised`: success.
  - `Error`: response has `refusalReason`.
  - `Pending` or `Received`: final result comes in AUTHORISATION webhook. **If not authorised within 30 minutes, the offer closes**; optionally receive an `OFFER_CLOSED` webhook.
  - `Refused`: ask shopper to use another method.
- Webhooks if shopper never returns: `AUTHORISATION` with `success` `false` (cancel order) or `true` (fulfil).
- Testing: simulator type `touchngo`. Direct testing: no credentials published; check Payment list. Live: contact Support.

Source: https://docs.adyen.com/payment-methods/touchngo/web-component
- v6: `new Redirect(checkout, { type: 'touchngo' }).mount('#touchngo-container')`. v5: `checkout.create('touchngo')`. No extra fields.

Source: https://docs.adyen.com/payment-methods/touchngo/web-drop-in
- Drop-in: no Touch 'n Go specific configuration.

Source: https://docs.adyen.com/payment-methods/touchngo/web
- Index page only.

Note: Touch 'n Go eWallet also appears inside Alipay+ as `alipay_plus_touchngo` (cross-border). Source: https://docs.adyen.com/payment-methods/alipay-plus/api-only

## 4. GrabPay

Source: https://docs.adyen.com/payment-methods/grabpay
- Shopper redirected to GrabPay login, authenticates with phone number + OTP, picks payment option, confirms.
- Options: **Wallet** (pay full amount now) and **PayLater** (pay next month, or 4 equal instalments over 3 months).
- Wallet in MY, SG, PH. PayLater only in MY and SG.
- Refunds, partial, multiple partial: yes. Recurring, captures, chargebacks: no.
- **For refunds, `reference` is required: max 32 characters, only alphanumeric, dashes and underscores.**
- POS (in-person QR) exists; skipped (not web).

Source: https://docs.adyen.com/payment-methods/grabpay/api-only
- `paymentMethod.type` by country: `grabpay_MY` (Malaysia, MYR), `grabpay_PH` (Philippines, PHP), `grabpay_SG` (Singapore, SGD). `/paymentMethods`: `countryCode` `MY` + `amount.currency` `MYR` for Malaysia.
- Wallet `/payments` fields: `paymentMethod.type` `grabpay_MY`; `returnUrl`; `reference` (**max 32 chars, alphanumeric, dashes, underscores only**); `amount`.
- PayLater `/payments` additionally needs: `deliveryAddress` (city, country, houseNumberOrName, postalCode, street) and `lineItems` (quantity, taxPercentage, description, id, amountIncludingTax, productUrl, imageUrl in the example). **`lineItems[].amountIncludingTax` cannot be 0.** Same `paymentMethod.type` as Wallet (no separate PayLater type in the request).
- Response: `RedirectShopper` with `action.type` `redirect`, `method` `GET`.
- `/payments/details` response may include `additionalData.paymentMethodVariant`:
  - Wallet: `grabpay_wallet_SG`, `grabpay_sg`, `grabpay_wallet_MY`, `grabpay_my`, `grabpay_wallet_PH`, `grabpay_ph`
  - PayLater: `grabpay_postpaid_SG`, `grabpay_instalment4_SG`, `grabpay_postpaid_MY`, `grabpay_instalment4_MY`
- resultCode values: `Authorised` (funds "in a number of days"), `Cancelled` (shopper cancelled, e.g. in GrabPay app), `Refused`, `Pending`/`Received` (wait for AUTHORISATION webhook).
- Webhooks: `AUTHORISATION` success `false` / `true`; **`OFFER_CLOSED` success `true` = shopper did not complete; cancel order (timed out).** Timeout duration: Not documented in the method pages.
- Testing: simulator type shown as `grabpay_SG`. Direct testing: contact Support for **test wallet credentials**; in the simulator log in to Grab with the registered phone number and the OTP received on that phone.
- Go live: submit a request for GrabPay in the live Customer Area.
- The Java/C#/Go SDK samples model GrabPay as `StoredPaymentMethodDetails` (OpenAPI v72 also lists `grabpay_MY` under `StoredPaymentMethodDetails`). That is a schema naming quirk, not recurring support.

Source: https://docs.adyen.com/payment-methods/grabpay/web-component
- Pick component type by country: `grabpay_MY` / `grabpay_PH` / `grabpay_SG`. v6: `new Redirect(checkout, { type: 'grabpay_MY' }).mount(...)`; v5: `checkout.create('grabpay_MY')`.
- `deliveryAddress` and `lineItems` required for PayLater on both `/sessions` and `/payments`. Adyen "highly recommend" sending them for Wallet too, so PayLater can be enabled later without rework.

Source: https://docs.adyen.com/payment-methods/grabpay/web-drop-in
- Same PayLater field requirements; no Drop-in specific configuration.

Source: https://docs.adyen.com/payment-methods/grabpay/web
- Index page only.

## 5. Boost Wallet

Source: https://docs.adyen.com/payment-methods/boost
- Malaysian e-wallet (Axiata), offered via Razer Merchant Services (RMS, formerly MOLPay).
- Wallets, redirect, MY / MYR. **Refunds: yes. Partial refunds: NO. Multiple partial refunds: NO.** Recurring, captures, chargebacks: no. Only full refunds are supported.

Source: https://docs.adyen.com/payment-methods/boost/api-only
- Setup: add Boost Wallet in Customer Area. No form fields needed.
- `/paymentMethods`: `countryCode` `MY`, `amount.currency` `MYR`. Response entry: `{ "name": "Boost", "type": "molpay_boost" }`.
- `/payments`: no extra parameters; send `paymentMethod.type`: `molpay_boost`. (Gotcha: the page's own curl example omits `paymentMethod` entirely; that is a docs error. The response example shows `paymentMethodType` `molpay_boost`, and the simulator section confirms the type.)
- Response: `RedirectShopper`, `action.type` `redirect`, `method` `GET`.
- Testing: simulator type `molpay_boost`. Direct testing: "live penny tests" only (no sandbox credentials).

Source: https://docs.adyen.com/payment-methods/boost/web-component
- Import `Redirect`; `new Redirect(checkout, { type: 'molpay_boost' }).mount(...)` (page example has a typo, unquoted `molpay_boost` and `#None-container`). No extra config, no extra `/sessions` or `/payments` fields.

Source: https://docs.adyen.com/payment-methods/boost/web-drop-in
- Optional tree-shaking import of `Redirect`. Nothing else.

Source: https://docs.adyen.com/payment-methods/boost/web
- Index page only (links to the Redirect component).

## 6. Atome (BNPL)

Source: https://docs.adyen.com/payment-methods/atome
- Pay in three instalments. Atome runs risk checks on the shopper. **Captured automatically after authorisation.** First instalment is charged during authorisation.
- Flow: redirect to Atome page; shopper either "Pay using app" (desktop: scan QR in Atome app; mobile: app opens automatically) or "Continue in browser" (phone number + OTP); sees instalment breakdown; confirms; redirected back.
- **Must:** Atome is not available for portable electronics (phones, laptops, speakers and similar). Implement checkout logic that disables Atome for these items.
- MY, SG / MYR, SGD. Refunds, partial, multiple partial: yes. Recurring, separate captures, chargebacks: no.
- Transaction limits:

| Country | Min | Max (debit card) | Max (credit card) |
|---|---|---|---|
| Malaysia | MYR 10 | MYR 1,500 | MYR 5,000 |
| Singapore | SGD 1.50 | SGD 1,000 | SGD 3,000 |

Source: https://docs.adyen.com/payment-methods/atome/api-only
- Setup: add Atome in Customer Area (test). Live: contact Support to add in live Customer Area.
- Collect for risk checks: full name, email, phone, billing address and delivery address.
- `/paymentMethods`: `MY`+`MYR` or `SG`+`SGD`. Response entry `{ "name": "Atome", "type": "atome" }`.
- `/payments` (and `/sessions`) required fields: `paymentMethod.type` `atome` (payments only), `shopperName` (`firstName`, `lastName`), `shopperEmail`, `telephoneNumber`, `billingAddress`, `lineItems`, `countryCode` (`MY` or `SG`; must match currency: MY with MYR, SG with SGD). Optional: `deliveryAddress` (optional only if same as billing).
- Example lineItems carry `description`, `quantity`, `amountIncludingTax`, `amountExcludingTax`, `taxAmount`, `id`. Example billing/delivery address fields: `city`, `country`, `houseNumberOrName`, `postalCode`, `stateOrProvince`, `street`.
- Response: `RedirectShopper`, `action.type` `redirect`.
- Partial refund: `POST /payments/{paymentPspReference}/refunds` with `amount`.
- Testing: simulator type `atome`. Direct testing needs an Atome sandbox test shopper account (request from your Adyen contact); use a separate test account per country. QR flow test apps: Malaysia Android https://www.pgyer.com/vBsC, iOS https://testflight.apple.com/join/SPxKJdnB; Singapore Android https://www.pgyer.com/m4OL, iOS same TestFlight link.

Source: https://docs.adyen.com/payment-methods/atome/web-component
- Import and initialize `Atome`: `new Atome(checkout, config).mount('#atome-container')`.
- Prefill via `data`: `personalDetails.firstName`, `personalDetails.lastName`, `personalDetails.shopperEmail`, `personalDetails.telephoneNumber`, `billingAddress.street`, `billingAddress.postalCode`, `billingAddress.city`, `billingAddress.country` (ISO alpha-2).
- `visibility` defaults: `companyDetails` hidden, `personalDetails` editable, `billingAddress` editable, `deliveryAddress` hidden, `bankAccount` hidden. For Atome, `deliveryAddress` and `companyDetails` are forced hidden (so send `deliveryAddress` server-side if it differs).
- Sessions example uses `/v71/sessions`.

Source: https://docs.adyen.com/payment-methods/atome/web-drop-in
- Same server fields on `/sessions` or `/payments`; optional `Atome` import for tree shaking.

Source: https://docs.adyen.com/payment-methods/atome/web
- Index page only.

## 7. Alipay

Source: https://docs.adyen.com/payment-methods/alipay
- Only for merchants selling outside China; Adyen does not process Alipay within China.
- **Offer stays open for 12 hours**: shopper can complete up to 12 hours after initiation (so a pending order can still turn Authorised hours later).
- Wallets, International, multiple currencies. Refunds, partial, multiple partial: yes. Recurring, captures, chargebacks: no.
- POS supported; skipped.

Source: https://docs.adyen.com/payment-methods/alipay/api-only
- `/paymentMethods` countries include **MY**; currencies include **MYR**. Full lists: countries AT, AU, BE, BG, CA, CH, CY, CZ, DE, DK, EE, ES, FI, FR, GB, GR, HK, HR, HU, ID, IS, IT, JP, KR, LT, LU, LV, MT, MY, NL, NO, NZ, PH, PL, PR, PT, RO, SE, SG, SI, SK, TH, TW, US, VN; currencies AUD, CAD, CHF, DKK, EUR, GBP, HKD, JPY, MYR, NOK, NZD, SEK, SGD, THB, USD.
- Response entry `{ "name": "Alipay", "type": "alipay" }`. `/payments`: no extra parameters; `paymentMethod.type` `alipay`; redirect handling.
- Testing: simulator type `alipay`. Direct: install Alipay app with a card and make a live penny test; view in live Customer Area.
- Live: submit request for Alipay in live Customer Area.

Source: https://docs.adyen.com/payment-methods/alipay/web-component
- Use `Redirect` class (`import { AdyenCheckout, Redirect }`); no config. Page snippet has typo `newRedirect`.

Source: https://docs.adyen.com/payment-methods/alipay/web-drop-in
- No config; the import snippet uses `Alipay` while text says `Redirect` (docs inconsistency).

Source: https://docs.adyen.com/payment-methods/alipay/web
- Index page only.

## 8. Alipay+

Source: https://docs.adyen.com/payment-methods/alipay-plus
- Cross-border "wallet of wallets". 11 wallets: Alipay, AlipayHK, DANA, Kakao Pay, K Plus, NAVER Pay, Touch 'n Go eWallet, Toss, TrueMoney, GCash, Rabbit LINE Pay.
- Onboarding request goes to all wallets; each reviews by its own risk rules. **Expected SLA for all wallets to be onboarded: within 1 month.** TrueMoney needs a separate onboarding form completed by Adyen Support.
- Refunds, partial, multiple partial: yes. Recurring, captures, chargebacks: no.

Source: https://docs.adyen.com/payment-methods/alipay-plus/api-only
- `/paymentMethods` countries include MY; currencies include MYR. Countries: AT, AU, BE, CA, CH, CN, CY, CZ, DE, DK, EE, ES, FI, FR, GB, GR, HK, HU, IE, IS, IT, JP, LI, LT, LU, LV, MT, MY, NL, NO, NZ, PL, PT, RO, SE, SG, SI, SK, US. Currencies: AED, AUD, CAD, CHF, CNY, CZK, DKK, EUR, GBP, HKD, JPY, MYR, NOK, NZD, SEK, SGD, USD. Response entry `type` `alipay_plus`.
- `/payments` extra fields: `channel` (required: `Web`, `iOS`, or `Android`) and `browserInfo` (required when `channel` is `Web`; userAgent, acceptHeader, language, colorDepth, screenHeight, screenWidth, timeZoneOffset, javaEnabled, javaScriptEnabled, collected from the shopper's browser at checkout).
- Adyen uses `channel` + `browserInfo.userAgent` to pick wallet entry points: desktop browser (Web + desktop UA), mobile browser (Web + mobile UA), in-app webview (`iOS`/`Android`, `browserInfo` not required). Wrong channel (e.g. Web for an in-app webview) gives an incompatible redirect experience. No native SDK.
- Response: `RedirectShopper`, `action.url` on `global.alipay.com`; shopper selects their wallet on the Alipay+ page.
- Return: `/payments/details` with `details.redirectResult` (+ `paymentData` from the original response in Advanced flow; handled automatically in Drop-in/Sessions).
- **`paymentMethod.type` changes**: request uses `alipay_plus`; `/payments/details` response and the AUTHORISATION webhook `paymentMethod` / `additionalData.paymentMethodVariant` show the actual wallet:

| Wallet | Market | Value |
|---|---|---|
| Alipay | China | `alipay_plus_alipay_cn` |
| AlipayHK | Hong Kong | `alipay_plus_alipay_hk` |
| DANA | Indonesia | `alipay_plus_dana` |
| GCash | Philippines | `alipay_plus_gcash` |
| Kakao Pay | South Korea | `alipay_plus_kakaopay` |
| KPlus | Thailand | `alipay_plus_kplus` |
| Naver Pay | South Korea | `alipay_plus_naverpay` |
| Rabbit LINE Pay | Thailand | `alipay_plus_rabbitlinepay` |
| Touch 'n Go eWallet | Malaysia | `alipay_plus_touchngo` |
| Toss | South Korea | `alipay_plus_tosspay` |
| TrueMoney | Thailand | `alipay_plus_truemoney` |

- Must/do not rules: always send `paymentMethod.type` `alipay_plus` (do not try to preselect a wallet); reconcile per wallet using webhook `paymentMethod`; frontend must call `/payments/details` with `redirectResult` after return; **`storePaymentMethod: true` fails validation** (no tokenization); always send `browserInfo` when `channel` is `Web`.
- Testing: simulator type `alipay_plus`; direct = live penny test with a wallet app.

Source: https://docs.adyen.com/payment-methods/alipay-plus/web-component
- Docs say import/initialize `redirect` (lowercase, likely meaning the `Redirect` class) with type `alipay_plus`. Sessions: no extra fields. Advanced flow: send `channel` and `browserInfo` as above.

Source: https://docs.adyen.com/payment-methods/alipay-plus/web-drop-in
- Same as component; Advanced flow `/payments` needs `channel` + `browserInfo`.

Source: https://docs.adyen.com/payment-methods/alipay-plus/web
- Index page only.

## 9. WeChat Pay

Source: https://docs.adyen.com/payment-methods/wechat-pay
- Mobile websites: only payments initiated from inside the WeChat app are supported (otherwise use in-app flow).
- Most payments confirm immediately; a small number need a webhook to confirm.
- Refunds, partial, multiple partial: yes. Recurring, captures, chargebacks: no.

Source: https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-desktop-qr-payments
- Desktop browser: show a QR code (works once WeChat Pay is added in Customer Area). Mobile browser (H5): redirect to WeChat app and back; **requires enrolment: contact Support with your web shop URL**.
- Currencies listed: AUD, CAD, CNY, EUR, GBP, HKD, JPY, NZD, SGD, USD. **MYR is not listed.** For a MYR-pricing Malaysian merchant, confirm currency support with Adyen before offering.

Source: https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-desktop-qr-payments/api-only
- `/paymentMethods`: country outside China, supported currency, and `channel` (`Web`, `iOS`, `Android`). Response type `wechatpayWeb`.
- `/payments`: `paymentMethod.type` `wechatpayWeb`; response `RedirectShopper` with `action.type` `redirect`; then `/payments/details` with `redirectResult`.
- resultCode: `Authorised` ("funds in 2-3 days"), `Error` (`refusalReason`), `Pending`/`Received` (wait for AUTHORISATION webhook; "funds in 2 days" if successful), `Refused`.
- **No test platform.** Test with a personal WeChat account by cancelling at verification (recommended) or small live payments. Live: submit request in live Customer Area.
- OpenAPI v72 also has `wechatpayQR` in `PaymentDetails`; not described in the method pages.

Source: https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-desktop-qr-payments/web-component
- v6: `import { AdyenCheckout, WeChat }`; `new WeChat(checkout).mount('#wechatpayWeb-container')`. v5: `checkout.create('wechatpayWeb')`. No extra fields.

Source: https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-desktop-qr-payments/web-drop-in
- No Drop-in specific configuration.

Skipped (native app / WeChat ecosystem, not web):
- In-app (`wechatpaySDK`, `action.type` `sdk` with `sdkData` appid/noncestr/packageValue/partnerid/prepayid/sign/timestamp; `/payments/details` with `details.resultCode` from SDK + `paymentData`). Needs WeChat Open Platform developer account, registered app, and AppID + AppSecret sent to Adyen Support so funds settle to your Adyen account. Source: https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-in-app-payments/api-only, https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-in-app-payments/enable-wechat-pay, https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-in-app-payments
- Mini Program (`wechatpayMiniProgram` + `paymentMethod.openid`; shopper always pays in CNY; allow-list `https://[YOUR_MERCHANT_PREFIX]-checkout-live.adyenpayments.com`; make all Adyen calls from your server; `/payments/details` with `paymentData` + `details.paymentStatus` = `requestPayment:ok` / `requestPayment:fail cancel` / `requestPayment:fail ...`). Source: https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-mini-program, https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-mini-program/api-only

## 10. Customer Area setup summary

- Add via Customer Area (self-service, test then live request): FPX, GrabPay, Boost, Atome (test), Alipay, Alipay+, WeChat Pay web (desktop QR). Link: https://docs.adyen.com/payment-methods/add-payment-methods
- Contact Adyen Support (test and live): DuitNow, Touch 'n Go, Atome (live), WeChat Pay mobile-browser H5 enrolment (send shop URL), GrabPay test wallet credentials, Atome sandbox shopper account (via your Adyen contact).
- Check test results: Customer Area (ca-test.adyen.com) > Payments > Payment list.
- Sources: pages listed in sections 1 to 9.

## 11. Top gotchas

1. FPX: you must show ALL 18 issuers (PayNet regulation) and `issuer` is required with `molpay_ebanking_fpx_MY`. https://docs.adyen.com/payment-methods/fpx-online-banking-malaysia/api-only
2. DuitNow is QR-only, no redirect and no `/payments/details`; the final status arrives only by webhook. https://docs.adyen.com/payment-methods/duitnow/web-component
3. Boost supports full refunds only (no partial). https://docs.adyen.com/payment-methods/boost
4. GrabPay refund `reference` (and payment `reference`) max 32 chars, `[A-Za-z0-9_-]` only; PayLater needs `deliveryAddress` + `lineItems` with non-zero `amountIncludingTax`. https://docs.adyen.com/payment-methods/grabpay
5. Atome: MYR 10 min, MYR 1,500 (debit) / MYR 5,000 (credit) max; required shopper fields; block portable electronics; auto-captured. https://docs.adyen.com/payment-methods/atome
6. Touch 'n Go offer closes after 30 minutes if not authorised (`OFFER_CLOSED`); Alipay offer stays open 12 hours. https://docs.adyen.com/payment-methods/touchngo/api-only, https://docs.adyen.com/payment-methods/alipay
7. Alipay+ request type `alipay_plus`, but responses/webhooks report `alipay_plus_<wallet>`; no tokenization; `channel` + `browserInfo` required on Web. https://docs.adyen.com/payment-methods/alipay-plus/api-only
8. WeChat Pay: no test platform and MYR not in the listed currencies. https://docs.adyen.com/payment-methods/wechat-pay/wechat-pay-desktop-qr-payments

## 12. Cards, Apple Pay, Google Pay for Malaysia

- Cards (`scheme`) are enabled by default per merchant account setup; brands are in `/paymentMethods` `brands`. Card data must be encrypted client side (Drop-in or Components); raw card data needs the **API PCI Payments role** and PCI SAQ D. Source: `https://docs.adyen.com/development-resources/api-credentials/roles`
- 3D Secure in Malaysia: "issuing banks may require authentication on their BINs, otherwise, authorisation rates will be low." Liability shift for MY applies to 3DS2 completed (frictionless or challenge) after October 2019. Source: `https://docs.adyen.com/online-payments/3d-secure-for-regulation-compliance`
- Card installments page lists Brazil, Mexico and Japan only; Malaysian card installments are not documented (checked 2026-10-21). Source: `https://docs.adyen.com/payment-methods/cards/credit-card-installments`
- Apple Pay: web integrations can use Adyen's Apple Pay certificate; mobile apps need your own. Source: `https://docs.adyen.com/payment-methods/apple-pay`
- Google Pay: register with Google, choose **Gateway** tokenization, `gateway` = `adyen`, `gatewayMerchantId` = your merchant or company account name. Source: `https://docs.adyen.com/payment-methods/google-pay`
- Test cards and 3DS test data: `adyen-errors-testing.md`.
