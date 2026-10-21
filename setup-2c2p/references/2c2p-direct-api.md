# 2C2P Direct API and Web Front-End Reference

Use this reference when the merchant builds its own checkout UI instead of redirecting to the 2C2P hosted page: Direct API (Payment Option, Payment Option Details, Do Payment, Transaction Status), card encryption with the Secure Pay JS library, iFrame and Drop-In UI, Apple Pay, Google Pay and Click2Pay. The hosted redirect flow, Payment Token, Payment Inquiry, backend notification and JWT signing live in `2c2p-api.md`.

Checked against the official docs on 2026-10-21. Verify the channels enabled on the merchant account before promising any method is live.

## Source URLs

- `https://developer.2c2p.com/docs/direct-api-how-it-works`
- `https://developer.2c2p.com/docs/direct-api-payment-methods`
- `https://developer.2c2p.com/docs/api-do-payment`
- `https://developer.2c2p.com/docs/api-payment-option`
- `https://developer.2c2p.com/docs/api-payment-option-details`
- `https://developer.2c2p.com/docs/api-transaction-status-inquiry`
- `https://developer.2c2p.com/docs/encryption-of-card-data-information`
- `https://developer.2c2p.com/docs/reference-payment-channels`
- `https://developer.2c2p.com/reference/post_payment-4-3-payment` (OpenAPI, PGW Payment API 4.3)

## Version And URL Notes (read first)

- The narrative docs pages show `/payment/4.5/<operation>` URLs with camelCase operation names, for example `https://sandbox-pgw.2c2p.com/payment/4.5/paymentOption`.
- The OpenAPI reference (title "PGW Payment API 4.3", sandbox server `https://sandbox-pgw.2c2p.com`) lists the same operations at `/payment/4.3/<operation>` in lowercase, for example `/payment/4.3/paymentoption`.
- Some docs samples use yet other spellings: Click2Pay calls `/payment/4.3/PaymentOptionDetails` and `/payment/4.3/payment`; the Apple Pay sample posts to `https://sandbox-pgw-ui.2c2p.com/payment/4.3/doPayment`.
- Rule: use one version consistently and confirm with 2C2P which version your merchant profile is set up for. Do not mix 4.3 and 4.5 in the same flow. Path case handling is not documented in official docs (checked 2026-10-21); copy the exact casing shown for your chosen version.
- Production host is `https://pgw.2c2p.com`; sandbox host is `https://sandbox-pgw.2c2p.com`. Hosted UI hosts are `https://pgw-ui.2c2p.com` (production) and `https://sandbox-pgw-ui.2c2p.com` (sandbox).

Source: https://developer.2c2p.com/docs/api-do-payment, https://developer.2c2p.com/docs/api-payment-option, https://developer.2c2p.com/reference/post_payment-4-3-payment, https://developer.2c2p.com/docs/direct-api-method-click2pay, https://developer.2c2p.com/docs/direct-api-apple-pay

## Two Request Styles

2C2P Direct API operations use two different request styles. Get this right before coding.

| Style | Body | Used by |
| --- | --- | --- |
| JWT payload | `{"payload": "<JWT HS256 signed with merchant secret key>"}`; response is also `{"payload": "<JWT>"}` that you must verify and decode. | Payment Token, Payment Inquiry, Payment Instruction, Payment Simulate, Exchange Rate secure (`/exchangerate/secure`). These are merchant server calls. |
| Plain JSON keyed by `paymentToken` | Plain JSON such as `{"paymentToken": "...", "clientID": "...", "locale": "en"}`; plain JSON response. | Payment Option, Payment Option Details, Do Payment, Transaction Status, Cancel Transaction, Card Token Info, Card Installment Plan Info, Loyalty Point Info, Exchange Rate, Initialization, User Preference, Payment Notification, Apple Pay merchant validation. |

Notes:

- The docs page header for Do Payment, Payment Option, Payment Option Details, Transaction Status, Card Installment Plan Info and Loyalty Point Info says "Requests and Responses are encrypted using JWT", but every sample request on those pages, the OpenAPI schemas, and the Click2Pay browser sample send plain JSON keyed by `paymentToken`. Follow the samples and the OpenAPI schema (plain JSON) and confirm with 2C2P if a call is rejected.
- Because these calls only carry the short-lived `paymentToken` (valid 30 minutes per the Payment Token response spec), 2C2P samples call some of them from the browser (Click2Pay, Apple Pay). Never put the merchant secret key in the browser. Payment Token creation and Payment Inquiry always stay server-side.
- `clientID` is a UUID the merchant generates (docs: "Formula : Get UUID"); the SDK stores it in app preference. Optional for web.
- Content type: send `Content-Type: application/json`. The OpenAPI also lists `application/json-patch+json`, `text/json` and `application/*+json` as accepted request types.

Source: https://developer.2c2p.com/docs/api-do-payment, https://developer.2c2p.com/docs/api-payment-token-response-parameter, https://developer.2c2p.com/reference/post_payment-4-3-payment, https://developer.2c2p.com/reference/post_payment-4-3-paymenttoken, https://developer.2c2p.com/docs/direct-api-method-click2pay

## Direct API Flows

All flows start with a server-side Payment Token request (see `2c2p-api.md`). Steps marked optional can be skipped when you already know the channel codes.

Server-to-Server (non-3DS card, no redirect):

1. Server requests Payment Token.
2. Optional: Payment Option, then Payment Option Details (with `categoryCode` and `groupCode`) to build the UI.
3. Customer submits; merchant calls Do Payment with `paymentToken` and payment data.
4. 2C2P returns the Do Payment response; backend notification is sent if enabled.
5. If backend notification is not enabled, call Payment Inquiry. Display result.

Third Party Redirection (3DS card, e-wallet, FPX / internet banking):

1. Token, optional options, Do Payment as above.
2. Do Payment returns a URL in `data`; merchant redirects the browser (new tab or same window, per `respCode`).
3. Acquirer returns status to 2C2P; 2C2P sends backend notification, then redirects the browser to `frontendReturnUrl`.
4. If backend notification is not enabled, call Payment Inquiry.

Scan QR (DuitNow QR, other QR):

1. Token, optional options, Do Payment.
2. Display the QR from `data` with a pending status.
3. Front end loops on Transaction Status until an end result is returned.
4. 2C2P sends backend notification when the third party confirms.

Offline / Over the Counter (payment slip):

1. Token, optional options, Do Payment.
2. Display the payment slip with pending status; front end loops on Transaction Status.
3. Customer pays at counter or kiosk; 2C2P sends backend notification.

Settlement truth: fulfil only from the verified backend notification or a server-side Payment Inquiry, never from Do Payment `2000`, Transaction Status, iFrame postMessage, or Drop-In callback alone.

Source: https://developer.2c2p.com/docs/direct-api-how-it-works, https://developer.2c2p.com/docs/direct-api-flow-server-to-server, https://developer.2c2p.com/docs/direct-api-flow-third-party-redirection, https://developer.2c2p.com/docs/direct-api-flow-scan-qr, https://developer.2c2p.com/docs/direct-api-flow-offline-payment

## Payment Flow Response Codes (Do Payment, Transaction Status, front end)

| respCode | Platform | Meaning | Action |
| --- | --- | --- | --- |
| `1000` | Web / Mobile | Load redirect URL in iFrame / WebView. | Close the iFrame when it loads the result URL and read the body message. |
| `1001` | Web / Mobile | Full redirection to a web page. | Full redirect to the third party page in `data`. |
| `1002` | Web / Mobile | Redirect to scheme URL (deep link) or web URL, then query status. | Mobile: open the app, call Transaction Status on resume. Web: open in external tab, loop or long-poll Transaction Status. |
| `1003` | Web / Mobile | Payment slip. | Show payslip from `data` / `extras`, set status pending. |
| `1004` | Mobile | Redirect to external app with app callback. | Open app scheme; `fallbackData` is a web payment fallback if the app is not installed. |
| `1005` | Web / Mobile | QR generated, waiting for scan. | Show QR, loop on Transaction Status. `expiryTimer` (ms) and `expiryDescription` apply. |
| `2000` | Web / Mobile | Transaction completed. | With backend notification: check your own DB status. Without: call Payment Inquiry. Then show result. |
| Other | Web / Mobile | Failed or rejected. | Run Payment Inquiry for full status and show the result. See Payment Response Codes in `2c2p-api.md`. |

Source: https://developer.2c2p.com/docs/response-code-payment-flow

## Endpoint Catalogue (Direct API and UI helpers)

Sandbox host `https://sandbox-pgw.2c2p.com`, production host `https://pgw.2c2p.com`. All `POST`.

| Operation | Docs page path (4.5) | OpenAPI path (4.3) | Style | Purpose |
| --- | --- | --- | --- | --- |
| Payment Option | `/payment/4.5/paymentOption` | `/payment/4.3/paymentoption` | Plain JSON | List enabled channel categories and groups for this token. |
| Payment Option Details | `/payment/4.5/paymentOptionDetails` | `/payment/4.3/paymentoptiondetails` | Plain JSON | Channels, required inputs, validation regex, IPP plans for one category/group. |
| Do Payment | `/payment/4.5/payment` | `/payment/4.3/payment` | Plain JSON | Submit the payment. |
| Transaction Status | `/payment/4.5/transactionStatus` | `/payment/4.3/transactionstatus` | Plain JSON | UI status polling. Not for settlement. |
| Card Installment Plan Info | `/payment/4.5/cardinstallmentplaninfo` | `/payment/4.3/cardinstallmentplaninfo` | Plain JSON | Visa IPP plans for an encrypted card. Docs: "used only for VISA IPP". |
| Loyalty Point Info | `/payment/4.5/loyaltypointinfo` | `/payment/4.3/loyaltypointinfo` | Plain JSON | Card loyalty balance and rewards. |
| Card Token Info | Not documented on a docs page | `/payment/4.3/cardtokeninfo` | Plain JSON | Stored card tokens for the token's customer. |
| Cancel Transaction | Not documented on a docs page | `/payment/4.3/canceltransaction` | Plain JSON | Cancel the in-progress payment for a token. |
| Initialization | Not documented on a docs page | `/payment/4.3/initialization` | Plain JSON | Supported locales list. OpenAPI shows no request body. |
| User Preference | Not documented on a docs page | `/payment/4.3/userpreference` | Plain JSON | Saved customer preference and recent channels. |
| Payment Notification | Not documented on a docs page | `/payment/4.3/paymentnotification` | Plain JSON | Ask 2C2P to share the result to a messaging platform. |
| Exchange Rate | Not documented on a docs page | `/payment/4.3/exchangerate` | Plain JSON | FX rates for a card BIN (MCP/DCC). |
| Exchange Rate secure | Not documented on a docs page | `/payment/4.3/exchangerate/secure` | JWT payload | Server-side JWT variant ("JWT Request Endpoint"). |
| APM MCC Exchange Rate | Not documented on a docs page | `/payment/4.3/exchangerate/apmmccexchangerate` | Plain JSON | FX rates for APM multi-currency conversion. |
| Payment Instruction | Not documented on a docs page | `/payment/4.3/paymentinstruction` | JWT payload | Payload fields not documented in official docs (checked 2026-10-21). |
| Payment Simulate | Not documented on a docs page | `/payment/4.3/paymentsimulate` | JWT payload | Payload fields not documented in official docs (checked 2026-10-21). |
| Apple Pay merchant validation | Not documented on a docs page | `/payment/4.3/applepay/merchantvalidation` | Plain JSON | 2C2P-side Apple Pay merchant session. |

Payment Token and Payment Inquiry (`/payment/4.x/paymentToken`, `/payment/4.x/paymentInquiry`, JWT payload) are covered in `2c2p-api.md`.

Source: https://developer.2c2p.com/docs/api-payment-option, https://developer.2c2p.com/docs/api-payment-option-details, https://developer.2c2p.com/docs/api-do-payment, https://developer.2c2p.com/docs/api-transaction-status-inquiry, https://developer.2c2p.com/docs/api-card-installment-plan-info, https://developer.2c2p.com/docs/loyalty-point-info-api, https://developer.2c2p.com/reference/post_payment-4-3-paymentoption, https://developer.2c2p.com/reference/post_payment-4-3-paymentoptiondetails, https://developer.2c2p.com/reference/post_payment-4-3-payment, https://developer.2c2p.com/reference/post_payment-4-3-transactionstatus, https://developer.2c2p.com/reference/post_payment-4-3-cardtokeninfo, https://developer.2c2p.com/reference/post_payment-4-3-canceltransaction, https://developer.2c2p.com/reference/post_payment-4-3-initialization, https://developer.2c2p.com/reference/post_payment-4-3-userpreference, https://developer.2c2p.com/reference/post_payment-4-3-paymentnotification, https://developer.2c2p.com/reference/post_payment-4-3-exchangerate, https://developer.2c2p.com/reference/post_payment-4-3-exchangerate-secure, https://developer.2c2p.com/reference/post_payment-4-3-exchangerate-apmmccexchangerate, https://developer.2c2p.com/reference/post_payment-4-3-paymentinstruction, https://developer.2c2p.com/reference/post_payment-4-3-paymentsimulate, https://developer.2c2p.com/reference/post_payment-4-3-applepay-merchantvalidation, https://developer.2c2p.com/reference/post_payment-4-3-cardinstallmentplaninfo, https://developer.2c2p.com/reference/post_payment-4-3-loyaltypointinfo

## Common Base Request (plain JSON calls)

Type legend used by 2C2P: A alphabet, AN alphanumeric, C characters, B boolean, D decimal, N numeric, NT ntext. M mandatory, C conditional, O optional.

| Field | Type | M/C/O | Notes |
| --- | --- | --- | --- |
| `paymentToken` | C 255 | M | From Payment Token API. |
| `clientID` | C 255 (uuid) | O | Merchant-generated UUID. |
| `locale` | C 10 | O | ISO 639. Empty uses the token locale; if `clientID` is set the user preference locale is used. Locale list comes from Initialization. |
| `browserDetails` | object | O | `deviceType`, `name`, `os`, `version` (AN 255 each). |

Source: https://developer.2c2p.com/docs/api-payment-option-request-parameter, https://developer.2c2p.com/reference/post_payment-4-3-paymentoption

## Payment Option

Request: base request above.

Response fields:

| Field | Type | M/C/O | Notes |
| --- | --- | --- | --- |
| `paymentToken` | C 255 | M | |
| `merchantDetails` | object | M | `id` AN 25 M, `name` A 50 M, `address` C 255 O, `email` C 50 O, `logoUrl` C 255 O, `bannerUrl` C 255 O. OpenAPI also lists `nationalMid`. |
| `channelCategories[]` | array | M | `sequenceNo` N 5, `name` C 50, `code` AN 6 (category code, for example `GCARD`, `DPAY`, `QR`, `WEBPAY`), `iconUrl`, `default` B, `groups[]` with `sequenceNo`, `name`, `code` AN 1-6 (group code, for example `CC`, `IPP`, `EWALLET`), `iconUrl`, `default`. OpenAPI group objects also list `channelCode`, `agentChannelCode`, `optionName`, `paymentCode`, `qrType`, `expirationTimer`, `expirationDescription`, `expiration`. |
| `transactionDetails` | object | M | `amount` D (12,5), `currencyCode` A 3, `invoiceNo` C 20, `description` C 255, `recurring` (C: `amount`, `interval` days, `count`, `chargeNextDate` N 8, `chargeOnDate` N 4). OpenAPI also lists `dateTime`, `paymentItems[]`, `showFxRate`. |
| `configuration` | object | M | `payment`: `immediatePayment`, `tokenize`, `tokenizeOnly`, `cardTokenOnly`, `fx.mcp` / `fx.dcc` (`active`, `terms`). `notification`: `facebook`, `whatsApp`, `line`. |
| `userDetails` | object | O | OpenAPI only: `address.billing` / `address.shipping` (`address1-3`, `postalCode`, `city`, `state`, `countryCode`). |
| `respCode` | N 4 | M | `0000` success. |
| `respDesc` | C 255 | M | |

Source: https://developer.2c2p.com/docs/api-payment-option, https://developer.2c2p.com/docs/api-payment-option-response-parameter, https://developer.2c2p.com/reference/post_payment-4-3-paymentoption

## Payment Option Details

Request: base request plus:

| Field | Type | M/C/O | Notes |
| --- | --- | --- | --- |
| `categoryCode` | AN 6 | M | From Payment Option. Docs say default is `CC`; samples use `GCARD`. |
| `groupCode` | AN 10 | M | From Payment Option. |

Click2Pay additionally needs the HTTP header `X-Merchant-Domain: <your website domain>` (must match the domain given at onboarding).

Response fields:

| Field | Type | M/C/O | Notes |
| --- | --- | --- | --- |
| `totalChannel` | N 20 | M | |
| `name`, `categoryCode`, `groupCode`, `iconUrl` | | M | |
| `validation.cardNo` | object | C | `prefixes` (array N 8), `regex`. |
| `validation.cardTypes[]` | array | C | `sequenceNo`, `name`, `prefixes`, `regex`, `iconURL`. |
| `channels[].sequenceNo`, `name`, `currencyCodes`, `iconUrl`, `logoUrl` | | M/C | OpenAPI also lists `bankShortName`. |
| `channels[].isDown` | B | M | Channel online/offline. Hide channels where `true`. |
| `channels[].immediatePayment` | B | M | |
| `channels[].payment.code` | object | M | `channelCode` AN 10, `agentCode` C 10, `agentChannelCode` C 10. Copy these into Do Payment. |
| `channels[].payment.input` | object | M | Per field (`name`, `email`, `mobileNo`, `cardNo`, `expiryDate`, `securityCode`, `pin`, `token`, `accountNo`): `M` mandatory, `O` optional, `I` ignore/hide. Tells you which `payment.data` fields Do Payment needs. |
| `channels[].payment.validation` | object | M | Regex per input field, plus `additional.cardNo` (`luhn` B, `prefixes`) and `additional.amount` (`min`, `max` D (12,2)). |
| `channels[].payment.info` | object | C | `terms`, `promoUrl`, `plans[]` (IPP: `sequenceNo`, `period`, `interestType`, `interestRate`, `monthlyAmount`, `monthlyInterest`, `currencyCode`, `label`). OpenAPI also lists `customData[]`, `provider`, `learnMoreUrl`, `termsConsent`, `token` (Click2Pay capture context), `srcUrl`. |
| OpenAPI-only channel flags | | | `checkEligibleOption`, `ippProviderCode`, `registrationStatus`, `partnerMerchantRefID`, `displayProcessingAmount`. |
| `configuration` | object | M | As Payment Option, plus `customerTokenOnly` and `fx.apmMcc`. |
| `respCode`, `respDesc` | | M | |

Source: https://developer.2c2p.com/docs/api-payment-option-details, https://developer.2c2p.com/docs/api-payment-option-details-request-parameter, https://developer.2c2p.com/docs/api-payment-option-details-response-parameter, https://developer.2c2p.com/reference/post_payment-4-3-paymentoptiondetails

## Do Payment

`POST https://sandbox-pgw.2c2p.com/payment/4.5/payment` (production `https://pgw.2c2p.com/payment/4.5/payment`; OpenAPI `/payment/4.3/payment`).

Request fields:

| Field | Type | M/C/O | Notes |
| --- | --- | --- | --- |
| `paymentToken` | C 255 | M | |
| `clientID` | C 255 | O | UUID. |
| `locale` | C 10 | O | |
| `responseReturnUrl` | C 255 | O | Only for third-party vendors building a payment UI for a merchant. Merchants use `frontendReturnUrl` in Payment Token. Payment token is appended to the URL. |
| `clientIP` | string | O | In OpenAPI and all method samples; not in the docs parameter table. |
| `browserDetails` | object | O | OpenAPI. |
| `payment.code.channelCode` | AN 10 | M | From the channel list, for example `CC`, `123`, `TNG`. |
| `payment.code.agentCode` | C 10 | C | Needed for `123` channels (FPX bank, counter). |
| `payment.code.agentChannelCode` | C 10 | C | Needed for `123` channels, for example `WEBPAY`, `IBANKING`, `OVERTHECOUNTER`. |
| `payment.data.name` | C 50 | C | Customer name. |
| `payment.data.email` | C 150 | C | Customer email. |
| `payment.data.mobileNo` | C 255 | O | |
| `payment.data.accountNo` | C 255 | O | |
| `payment.data.token` | C MAX | O | Apple Pay, Google Pay, CS token, Click2Pay. Card scheme token examples: `base64({"channel":"GOOGLEPAY","authMethod":"{CRYPTOGRAM_3DS / PAN_ONLY}","cryptogram":"{cryptogram}"})` (no cryptogram when `PAN_ONLY`), `base64({"channel":"APPLEPAY","authMethod":"3DSecure","cryptogram":"{cryptogram}"})`, `base64({"channel":"EXTERNAL","cryptogram":"{cryptogram}"})`. |
| `payment.data.securePayToken` | C 255 | C | Encrypted card from Secure Pay JS (`encryptedCardInfo`) or SecurePay mobile SDK. Alternative to raw card fields. |
| `payment.data.cardNo` | N 20 | O | Raw card number. Only if PCI scope allows; prefer `securePayToken`. |
| `payment.data.expiryMonth` | N 2 | O | |
| `payment.data.expiryYear` | N 4 | O | |
| `payment.data.securityCode` | N 4 | O | |
| `payment.data.cardBank`, `cardCountry` | C 255 | O | Issuer bank and country. |
| `payment.data.accountTokenization` | B | O | Let customer save card. |
| `payment.data.interestType` | A 1 | O | IPP interest type. |
| `payment.data.installmentPeriod` | N 2 | O | IPP months. |
| `payment.data.payLaterPeriod` | N 2 | O | Pay later months. |
| `payment.data.customerToken` | C 255 | O | Stored card or wallet token. |
| `payment.data.qrType` | C 255 | O | `ALL`, `RAW`, `BASE64`, `URL`. |
| `payment.data.fxRateID` | C 255 | O | From Exchange Rate. |
| `payment.data.paymentExpiry` | C 19 | O | `yyyy-MM-dd HH:mm:ss`. Defaults to token or profile expiry. |
| `payment.data.loyaltyPoints[]` | array | O | `providerID`, `accountNo`, `accountAuthData`, `externalMerchantId`, `redeemAmount` D 12,5, `rewards[]` (`id`, `quantity`). |
| `payment.data.customerNote` | C 255 | O | |
| `payment.data.userAgent` | C 255 | O | |
| OpenAPI-only data fields | string | O | `pin`, `billingAddress1`, `billingAddress2`, `billingAddress3`, `billingCity`, `billingState`, `billingPostalCode`, `billingCountryCode`. |

Response fields:

| Field | Type | M/C/O | Notes |
| --- | --- | --- | --- |
| `channelCode` | AN 6 | M | |
| `invoiceNo` | AN 50 | C | Only when `respCode` is `2000`. |
| `type` | A 6 | C | QR data type. |
| `expiryTimer` | N 10 | C | Milliseconds, flow `1005` only. |
| `expiryDescription` | C 255 | C | Flow `1005` only. |
| `data` | C 5000 | C | Redirect URL, deep link, slip URL, or QR. |
| `extras` | object | O | `qrData`, `barcodeData`, `referenceNo` (slip), `paymentExpiry`, `agentPaymentCode`. The sample shows the key as `extra`; accept both. |
| `fallbackData` | C 255 | C | Flow `1004` only, web fallback if app missing. |
| `respCode` | C 4 | M | Payment flow code (table above). |
| `respDesc` | C 255 | M | |
| OpenAPI-only | | | `payload`, `rawData`. |

Minimal card example (sandbox, values are placeholders):

```json
{
  "paymentToken": "<PAYMENT_TOKEN>",
  "clientID": "<UUID>",
  "locale": "en",
  "clientIP": "<CUSTOMER_IP>",
  "payment": {
    "code": { "channelCode": "CC" },
    "data": { "name": "<NAME>", "email": "<EMAIL>", "securePayToken": "<encryptedCardInfo>" }
  }
}
```

Source: https://developer.2c2p.com/docs/api-do-payment, https://developer.2c2p.com/docs/api-do-payment-request-parameter, https://developer.2c2p.com/docs/api-do-payment-response-parameter, https://developer.2c2p.com/reference/post_payment-4-3-payment

## Malaysian Channel Shapes For Do Payment

Codes come from the Payment Channels reference. The Payment Token `paymentChannel` filter takes the Group Code (or `123` channel group codes); Do Payment `payment.code` takes the Channel Code, and for `123` channels also the Agent Code and Agent Channel Code. Always prefer the codes returned by Payment Option Details for the live account.

| Method | `channelCode` | `agentCode` | `agentChannelCode` | `payment.data` | Expected flow |
| --- | --- | --- | --- | --- | --- |
| Cards (Visa, Mastercard, Amex, JCB, UnionPay, MyDebit) | `CC` (category `GCARD`, group `CC`) | | | `securePayToken` (or raw card fields), `name`, `email` | `2000` (non-3DS) or `1000`/`1001` (3DS ACS redirect) |
| Apple Pay / Google Pay | `APPLEPAY` / `GOOGLEPAY` (category `DPAY`, group `CWALLET`); scheme tokens may also route as `CSTOKEN` | | | `token` (base64 wallet token), `name`, `email` | `2000` or redirect |
| FPX online banking | `123` (category and group `WEBPAY`) | Bank code, for example `MYM2U` (Maybank2U), `MYCIMBCLICKS`, `MYPBB`, `MYRHB`, `MYHLB`, `MYAMB`, `MYBISM`, `MYBKRM`, `MYBSN`, `MYABB`, `MYABMB`, `MYAGB`, `MYBMMB`, `MYBOC`, `MYCITI`, `MYDBB`, `MYHSBC`, `MYKFH`, `MYOCBC`, `MYPBBE`, `MYSCB`, `MYUOB`, `MYUOBE`, `MYM2E`, `M2UPAY` | `WEBPAY` | `name`, `email`, `mobileNo` as required by `input` | `1001` redirect to bank |
| FPX sandbox test bank | `123` | `MYSBIA`, `MYSBIB`, `MYSBIC` (UAT / sandbox only) | `WEBPAY` | as above | `1001` |
| DuitNow QR | `DUITNOW` (category `QR`, group `QRC`) | | | `name`, `email`; optional `qrType` | `1005` QR, poll Transaction Status |
| Touch 'n Go eWallet | `TNG` (category `DPAY`, group `EWALLET`, tokenisation yes) | | | `name`, `email` | `1001` / `1002` / `1004` |
| Boost | `BOOST` (group `EWALLET`) | | | `name`, `email` | redirect |
| GrabPay | `GRAB` (group `EWALLET`, tokenisation SG and MY) | | | `name`, `email` | `1001` (docs sample) |
| ShopeePay | `SHPPAY` (group `EWALLET`, tokenisation yes) | | | `name`, `email` | redirect |
| ShopBack Pay, WeChat Pay, Alipay | `SHOPBACK`, `WECHAT`, `ALIPAY` (group `EWALLET`) | | | `name`, `email` | redirect |
| Customer-presented QR | `SHPQR` (ShopeePay), `GPQR` (GrabPay), `GIPQR` (group `QRC`) | | | | `1005` |
| Atome | `ATOME` (category `GBNPL`, group `BNPL`) | | | `name`, `email` | redirect |
| Grab PayLater | `GRABBNPL` (group `BNPL`) | | | `name`, `email` | redirect |
| SPayLater | `SPAYLATER` (group `BNPL`) | | | `name`, `email`; `payLaterPeriod` if used | redirect |
| Card instalment (IPP) | `IPP` (category `GCARD`, group `IPP`); MY banks listed: HSBC, Maybank, Public Bank, CIMB, Hong Leong, RHB, OCBC, Alliance, Amex, AmBank, Affin, Standard Chartered, UOB | | | `securePayToken`, `installmentPeriod`, `interestType` | `1000` / `1001` |
| Over the counter (EPAY) | `123` | `MYEPAY` | `OVERTHECOUNTER` | `name`, `email`, `mobileNo` | `1003` slip or `1001` |

FPX agent code migration: legacy agent codes `FPX_M2U`, `FPX_CIMB`, `FPX_PBB` and so on are replaced by the `MY...` codes above (for example `FPX_M2U` to `MYM2U`, `FPX_BIMB` to `MYBISM`). 2C2P strongly recommends updating; new features are not available on old codes. Affects Do Payment (any V4 version), mobile SDK internet banking and legacy APIs.

Sample FPX request (from the Web Payment guide pattern, values placeholders):

```json
{
  "paymentToken": "<PAYMENT_TOKEN>",
  "clientIP": "<CUSTOMER_IP>",
  "locale": "en",
  "payment": {
    "code": { "channelCode": "123", "agentCode": "MYM2U", "agentChannelCode": "WEBPAY" },
    "data": { "name": "<NAME>", "email": "<EMAIL>", "mobileNo": "<MOBILE>" }
  }
}
```

The docs Web Payment sample uses `agentCode` `UOB` with `agentChannelCode` `WEBPAY` (non-Malaysian). The Internet / Mobile Banking sample uses `KBANK` / `IBANKING` (Thailand). Malaysian online banking is listed under WEBPAY, not IMBANK.

Source: https://developer.2c2p.com/docs/reference-payment-channels, https://developer.2c2p.com/docs/reference-agent-code-update-for-fpx-payment, https://developer.2c2p.com/docs/direct-api-method-web-payment, https://developer.2c2p.com/docs/direct-api-method-digital-payment-wallet, https://developer.2c2p.com/docs/direct-api-method-qr-payment, https://developer.2c2p.com/docs/direct-api-method-3ds-card-payment, https://developer.2c2p.com/docs/direct-api-method-non-3ds-card-payment, https://developer.2c2p.com/docs/direct-api-method-internet-mobile-banking, https://developer.2c2p.com/docs/direct-api-method-pay-at-counter, https://developer.2c2p.com/docs/direct-api-method-self-service-machines

## Per-Method Guide Summary

Every method guide follows the same steps: Payment Token (check `respCode` `0000`, otherwise stop), optional Payment Option and Payment Option Details, Do Payment, handle `respCode`, then backend notification or Payment Inquiry.

- Non-3DS card: server-to-server; Do Payment returns `2000`; then backend notification or Payment Inquiry.
- 3DS card: Do Payment returns `1001` with an ACS URL in `data`; redirect; result via backend notification and `frontendReturnUrl`.
- Web Payment (FPX): Do Payment returns `1001` with bank URL; redirect.
- QR: Do Payment returns `1005` with `type` and QR image URL in `data`; loop Transaction Status.
- Digital wallet: Do Payment returns `1001` with redirect URL.
- Pay at counter / Self service machine: Do Payment returns a slip URL (sample shows `1001`); customer pays offline; loop Transaction Status.
- Failure or rejection of any step's `respCode` terminates the process.

Source: https://developer.2c2p.com/docs/direct-api-payment-methods, https://developer.2c2p.com/docs/direct-api-method-non-3ds-card-payment, https://developer.2c2p.com/docs/direct-api-method-3ds-card-payment, https://developer.2c2p.com/docs/direct-api-method-web-payment, https://developer.2c2p.com/docs/direct-api-method-qr-payment, https://developer.2c2p.com/docs/direct-api-method-digital-payment-wallet, https://developer.2c2p.com/docs/direct-api-method-pay-at-counter, https://developer.2c2p.com/docs/direct-api-method-self-service-machines, https://developer.2c2p.com/docs/direct-api-method-internet-mobile-banking

## Transaction Status Inquiry

`POST https://sandbox-pgw.2c2p.com/payment/4.5/transactionStatus` (production `https://pgw.2c2p.com/payment/4.5/transactionStatus`; OpenAPI `/payment/4.3/transactionstatus`).

Docs rule: do not use this API to check or acknowledge updated payment status. Use the server-side Payment Inquiry API for that. Transaction Status is for UI display and polling (`1002`, `1003`, `1005`).

Request: base request plus `additionalInfo` (B, O, default `false`; returns more details, V4 UI only).

Response:

| Field | Type | M/C/O | Notes |
| --- | --- | --- | --- |
| `channelCode` | AN 6 | M | |
| `invoiceNo` | AN 50 | C | Only when `respCode` is `2000`. |
| `additionalInfo.merchantDetails` | object | M | `name`, `address`, `email`, `logoUrl`, `showDetailInfo`. |
| `additionalInfo.transactionDetails` | object | M | `dateTime` N 14 (`yyyyMMddHHmmss`), `agentCode`, `channelCode`, `data` (masked PAN, wallet id or account no), `amount` D (12,5), `currencyCode`, `invoiceNo`, `description`, IPP fields (`installmentPeriod`, `payLaterPeriod`, `interestType`, `interestRate`, `monthlyPayment`, `ippAdditionalInfo`, `remainderChargeMode`, `remainderChargeAmount`, `processingAmount`, `processingFixedFee`, `processingPercentage`, `displayProcessingAmount`), `loyaltyPoints`. |
| `additionalInfo.paymentResultDetails` | object | O | `code` (`00` success green, `01` failed red, `02` general grey), `description`, `redirectImmediately`, `autoRedirection` (sample and OpenAPI: `autoRedirect`), `autoRedirectionTimer` ms (sample: `autoRedirectTimer`), `frontendReturnUrl`, `frontendReturnData` (base64), `frontendRedirectMethod`. |
| `payload` | C 255 | C | |
| `rawData` | C 255 | C | Raw acquirer data. |
| `respCode` | C 4 | M | Payment flow code. |
| `respDesc` | C 255 | M | |

Source: https://developer.2c2p.com/docs/api-transaction-status-inquiry, https://developer.2c2p.com/docs/api-transaction-status-inquiry-request-parameter, https://developer.2c2p.com/docs/api-transaction-status-inquiry-response-parameter, https://developer.2c2p.com/reference/post_payment-4-3-transactionstatus

## Card Token Info

`POST /payment/4.3/cardtokeninfo` (OpenAPI). Request: base request.

Response:

| Field | Type | M/C/O | Notes |
| --- | --- | --- | --- |
| `paymentToken` | C 255 | M | |
| `customerToken[]` | array | M | `channelCode` AN 30 O, `token` C 255 M, `accountNo` N 19 M (masked), `expiry` AN 7 M (`MM/YY`), `name` C 50 M, `email` C 255 M, `status` C 2 M (`ED` expired, `EX` expiring within 6 months, `A` available), `iconUrl`, `logoUrl`. OpenAPI also lists `subChannelCode`, `cardBrand`. |
| `respCode`, `respDesc` | | M | |

Pay with a stored card by putting `token` into Do Payment `payment.data.customerToken`. The Payment Token must allow it (`customerToken` array in the token request; see `2c2p-api.md`).

Source: https://developer.2c2p.com/docs/api-card-tokens-information-response-parameter, https://developer.2c2p.com/reference/post_payment-4-3-cardtokeninfo

## Card Installment Plan Info (Visa IPP)

`POST https://sandbox-pgw.2c2p.com/payment/4.5/cardinstallmentplaninfo` (production `https://pgw.2c2p.com/payment/4.5/cardinstallmentplaninfo`). Docs: used only for Visa IPP.

Request:

| Field | Type | M/C/O | Notes |
| --- | --- | --- | --- |
| `paymentToken` | C 255 | M | |
| `securePayToken` | C 255 | M | Encrypted card (card number required for encryption). |
| `bankCode` | C 20 | O | IPP bank code, sample `Visa Insta`. |
| `locale`, `clientID` | | O | |
| `cardNo` | string | O | OpenAPI only. |

Response: same structure as Payment Option Details (`channels[].payment.info.plans[]` with `period`, `interestType`, `interestRate`, `monthlyAmount`, `monthlyInterest`, `currencyCode`, `label`; sample also shows `terms`, `processingFixedFee`, `processingPercentage`, `processingAmount`, `learnMoreUrl`, `termsConsent`, `ippProviderCode`, `registrationStatus`).

Source: https://developer.2c2p.com/docs/api-card-installment-plan-info, https://developer.2c2p.com/docs/api-card-installment-plan-info-request-parameter, https://developer.2c2p.com/docs/api-card-installment-plan-info-response-parameter, https://developer.2c2p.com/reference/post_payment-4-3-cardinstallmentplaninfo

## Loyalty Point Info

`POST https://sandbox-pgw.2c2p.com/payment/4.5/loyaltypointinfo` (production `https://pgw.2c2p.com/payment/4.5/loyaltypointinfo`). Steps: create Payment Token with loyalty info, encrypt card, call this API, then Do Payment with `payment.data.loyaltyPoints`.

Request: `paymentToken` M, `providerId` AN 255 M (sample and OpenAPI spell it `providerID`), `securePayToken` M, `referenceID` O, `clientID` O, `locale` O, `browserDetails` O. OpenAPI also lists `profileID`, `cardNo`, `expiryMonth`, `expiryYear`.

Response: `paymentToken`, `providerId`, `providerName`, `referenceID`, `providerType` (`BANK` or `INDIVIDUAL`), `terms`, `rewards[]` (`sequenceNo`, `id`, `currencyCode`, `totalPoints`, `label`, `point`, `forceToSelectReward`, `name`, `type`, `amount`, `quantity`, `price`), `respCode`, `respDesc`.

Malaysian availability of loyalty providers is not documented in official docs (checked 2026-10-21).

Source: https://developer.2c2p.com/docs/loyalty-point-info-api, https://developer.2c2p.com/docs/loyalty-point-info-request-parameters, https://developer.2c2p.com/docs/loyalty-point-info-response-parameters, https://developer.2c2p.com/reference/post_payment-4-3-loyaltypointinfo

## Other UI Helper Operations (OpenAPI only)

- Cancel Transaction `/payment/4.3/canceltransaction`: base request; response `invoiceNo`, `payload`, `channelCode`, `rawData`, `respCode`, `respDesc`.
- Initialization `/payment/4.3/initialization`: response `initialization.locale[]` (`code`, `name`, `iconUrl`), `respCode`, `respDesc`.
- User Preference `/payment/4.3/userpreference`: base request; response `user.preference` (`name`, `email`, `mobileNo`, `mobileNoPrefix`, `locale`, `currencyCode`, `countryCode`) and `user.paymentChannels[]` (`sequenceNo`, `categoryCode`, `groupCode`, `name`, `channelCode`, `agentCode`, `agentChannelCode`, `iconUrl`, `logoUrl`).
- Payment Notification `/payment/4.3/paymentnotification`: base request plus `plateform` (spelled this way), `recipientID`, `recipientName`; response `respCode`, `respDesc`.
- Exchange Rate `/payment/4.3/exchangerate`: base request plus `bin`; response `fxRates[]` (`name`, `iconUrl`, `amount`, `sequenceNo`, `id`, `fx`, `currencyCode`), `providerCode`, `expire`, `paymentToken`. Use `id` as Do Payment `fxRateID`.
- APM MCC Exchange Rate `/payment/4.3/exchangerate/apmmccexchangerate`: base request; response `fxRates[]` (`expire`, `amount`, `name`, `iconUrl`, `sequenceNo`, `id`, `fx`, `currencyCode`).
- Exchange Rate secure, Payment Instruction, Payment Simulate: JWT `payload` in and out; inner fields not documented in official docs (checked 2026-10-21).
- Apple Pay merchant validation `/payment/4.3/applepay/merchantvalidation`: request `paymentToken` M, `validationUrl`, `locale`, `clientID`, `browserDetails`; response is an Apple merchant session (`epochTimestamp`, `expiresAt`, `merchantSessionIdentifier`, `nonce`, `merchantIdentifier`, `domainName`, `displayName`, `signature`, `operationalAnalyticsIdentifier`, `retries`, `pspId`) plus `respCode`, `respDesc`.

Source: https://developer.2c2p.com/reference/post_payment-4-3-canceltransaction, https://developer.2c2p.com/reference/post_payment-4-3-initialization, https://developer.2c2p.com/reference/post_payment-4-3-userpreference, https://developer.2c2p.com/reference/post_payment-4-3-paymentnotification, https://developer.2c2p.com/reference/post_payment-4-3-exchangerate, https://developer.2c2p.com/reference/post_payment-4-3-exchangerate-apmmccexchangerate, https://developer.2c2p.com/reference/post_payment-4-3-exchangerate-secure, https://developer.2c2p.com/reference/post_payment-4-3-paymentinstruction, https://developer.2c2p.com/reference/post_payment-4-3-paymentsimulate, https://developer.2c2p.com/reference/post_payment-4-3-applepay-merchantvalidation

## Card Encryption With Secure Pay JS (securePayToken)

2C2P's JavaScript SDK encrypts card data in the browser so the merchant server never sees raw PAN or CVV. 2C2P says this avoids the merchant undertaking full PCI-DSS certification.

Script URLs:

- Sandbox: `https://demo2.2c2p.com/SecurePayment/api/pgw-securepay-sdk-2.0.0.min.js`
- Production: `https://t.2c2p.com/SecurePayment/api/pgw-securepay-sdk-2.0.0.min.js`

Form inputs use `data-encrypt` attributes (do not give these inputs a `name`, so raw values are not posted):

| Attribute | Field |
| --- | --- |
| `data-encrypt="cardnumber"` | Card number |
| `data-encrypt="month"` | Expiry month, 2 digits |
| `data-encrypt="year"` | Expiry year, 4 digits |
| `data-encrypt="cvv"` | CVV2/CVC2 |

Two documented ways to encrypt:

1. `My2c2p.onSubmitForm("2c2p-payment-form", function(errCode, errDesc) {...})`: validates, then posts the form to your backend with `encryptedCardInfo`, `maskedCardInfo`, `expMonthCardInfo`, `expYearCardInfo` as form fields.
2. `My2c2p.getEncrypted(My2c2p.APIEnvironment.Sandbox | My2c2p.APIEnvironment.Production, formElement, function(formData, errorCode, errorDesc) {...})`: returns `formData.encryptedCardInfo` (plus masked and expiry fields) so you send it to your backend yourself.

Backend: put `encryptedCardInfo` into Do Payment `payment.data.securePayToken`.

Validation error codes: `0` success, `1` card number required, `2` card number invalid, `3` expiry month required, `4` expiry month must be two numbers, `5` expiry year required, `6` expiry year must be four numbers, `7` card already expired (year), `8` card already expired (month), `9` expiry month invalid, `10` CVV2/CVC2 invalid, `11` invalid card issued month, `12` invalid card issued year.

The encryption algorithm and `encryptedCardInfo` format are not documented in official docs (checked 2026-10-21). Treat the value as opaque, URL-decode it if it arrives form-encoded, and do not try to build it server-side.

Secure Fields (deprecated sample): older pattern loading `https://2c2p-uat-cloudfront.s3-ap-southeast-1.amazonaws.com/2C2PPGW/secureField/my2c2p-secureFields.1.0.0.min.js` and `https://demo2.2c2p.com/SecurePayment/api/my2c2p.1.7.6.min.js` with a form `id="2c2p-payment-form"`; card inputs are injected by 2C2P and `My2c2p.getEncrypted("2c2p-payment-form", cb)` returns `encryptedCardInfo`. The sample code download is marked deprecated; prefer the 2.0.0 SDK above.

Source: https://developer.2c2p.com/docs/encryption-of-card-data-information, https://developer.2c2p.com/docs/using-secure-pay-javascript-library, https://developer.2c2p.com/docs/reference-secure-pay-js-lib, https://developer.2c2p.com/docs/using-securefields

## Hosted Page In iFrame

The Payment Token `webPaymentUrl` can be loaded in an iFrame (pass `iframeMode: true` in the token request, see `2c2p-api.md`). 2C2P recommends the Drop-In UI instead.

- Parent page listens for `message` events; `event.data.paymentResult` has `respCode`, `respDesc`, `respData`.
- `respCode` `2000`: payment completed, go to your result page (then confirm server-side).
- `respCode` `1001`: redirect the top window to `respData` to continue (for example bank or wallet page).
- Parent can trigger submit with `iFrame.contentWindow.postMessage('submit_gcard', '*')`.

Source: https://developer.2c2p.com/docs/using-iframe, https://developer.2c2p.com/docs/response-code-payment-flow

## Drop-In UI (PGW Web SDK)

- JS: `https://pgw-ui.2c2p.com/sdk/js/pgw-sdk-4.2.1.js`
- CSS: `https://pgw-ui.2c2p.com/sdk/css/pgw-sdk-style-4.2.1.css` (merchant may self-host a customised copy).

`PGWSDK.paymentUI(request, callback)` request fields:

| Field | Type | M/C/O | Notes |
| --- | --- | --- | --- |
| `url` | C 255 | M | `webPaymentUrl` from Payment Token. |
| `templateId` | C 50 | O | Custom template id. |
| `mode` | C 20 | O | `Fullscreen` (default), `DropIn`, `Dialog`, `Tab`. |
| `appBar` | BL | O | `Dialog` / `DropIn` only. Default `false`. |
| `cancelConfirmation` | BL | O | `Dialog` / `DropIn` only. Default `false`. |

Callback response: `paymentToken`, `transactionStatusInquiry` (only for `2000`), `responseCode`, `responseDescription`. `0003` means the user cancelled; `2000` means completed (then rely on backend notification or Payment Inquiry).

`PGWSDK.submitPayment(submitType)` (`Dialog` / `DropIn` only): `submit_gcard`, `submit_lcard`, `submit_dpay`, `submit_qr`, `submit_counter`, `submit_ssm`, `submit_webpay`, `submit_imbank`, `submit_gbnpl`, `submit_cancel`.

Source: https://developer.2c2p.com/docs/web-sdk-drop-in-ui

## Apple Pay On The Web

Prerequisites:

- 2C2P merchant ID with Apple Pay enabled; HTTPS on your domain.
- Without an Apple Developer account: host the 2C2P-provided `apple-developer-merchantid-domain-association.txt` at `https://{domain}/.well-known/apple-developer-merchantid-domain-association.txt`.
- With an Apple Developer account: get the `{merchantId}_payment.certSigningRequest` from 2C2P, complete it in the Apple portal, return `apple_pay.cer` to 2C2P, and verify your domain with Apple. You also need your own Merchant Identity Certificate for merchant validation.

Flow:

1. Payment Token with `paymentChannel: ["APPLEPAY"]`; proceed only on `respCode` `0000`.
2. Show the button only if `window.ApplePaySession && ApplePaySession.canMakePayments()`.
3. `new ApplePaySession(3, paymentRequest)`; in `onvalidatemerchant`, your backend posts `{merchantIdentifier, displayName, initiative: "web", initiativeContext: <your domain>}` to Apple's `validationURL` with the Merchant Identity Certificate as TLS client cert and returns Apple's JSON to `completeMerchantValidation`. (OpenAPI also lists a 2C2P-side `/payment/4.3/applepay/merchantvalidation`.)
4. In `onpaymentauthorized`, base64-encode `JSON.stringify(event.payment.token.paymentData)` and send Do Payment with `payment.code.channelCode` `APPLEPAY` and `payment.data.token` set to that string (docs table text calls it `mobilePaymentToken`; the sample uses `payment.data.token`).
5. `session.completePayment(STATUS_SUCCESS | STATUS_FAILURE)` based on the Do Payment result, then settle from backend notification or Payment Inquiry.
6. Apple Pay inside the 2C2P iFrame: post the parent domain to the frame, `{"name":"parent_domain","payload":{"domain":"<YOUR_PARENT_DOMAIN>"}}`, target origin `https://pgw-sandbox-ui.2c2p.com` (sandbox, as written in docs) or `https://pgw-ui.2c2p.com` (production).

Source: https://developer.2c2p.com/docs/direct-api-apple-pay, https://developer.2c2p.com/reference/post_payment-4-3-applepay-merchantvalidation

## Google Pay On The Web

Prerequisites: sign up with Google for a Google Pay merchant ID, complete Google's web integration checklist, and request production access.

- Payment Token with `paymentChannel: ["GOOGLEPAY"]`; proceed only on `0000`.
- Google Pay request: `allowedAuthMethods` `["PAN_ONLY"]` (docs table), `allowedCardNetworks` as agreed with 2C2P (supported: `AMEX`, `DISCOVER`, `JCB`, `MASTERCARD`, `VISA`), `tokenizationSpecification.type` `PAYMENT_GATEWAY`, `parameters.gateway` = gateway identifier from 2C2P (sample `"2c2p"`), `parameters.gatewayMerchantId` = merchant ID from 2C2P, `merchantInfo.merchantId` = your Google merchant ID.
- Capture `paymentData.paymentMethodData.tokenizationData.token` and send Do Payment with `channelCode` `GOOGLEPAY` and `payment.data.token`.
- Encoding: the Do Payment parameter table shows `base64({"channel":"GOOGLEPAY","authMethod":"...","cryptogram":"..."})` for card scheme tokens; the Google Pay guide sample passes the Google token. Confirm the exact encoding with 2C2P.

Source: https://developer.2c2p.com/docs/google-pay, https://developer.2c2p.com/docs/api-do-payment-request-parameter

## Click2Pay (global, one line)

Visa/Mastercard Click to Pay via Cybersource Unified Checkout: Payment Token with `paymentChannel: ["CLICK2PAY"]`, Payment Option Details with `categoryCode` `DPAY`, `groupCode` `CWALLET` and header `X-Merchant-Domain`, read `channels[].payment.info.token` (capture context) and `srcUrl`, render with `Accept(captureContext).unifiedPayments().show(...)`, then Do Payment with `channelCode` `CLICK2PAY` and `payment.data.token`; handle `1001`/`1002`/`1004` by redirecting to `data`. The docs sample response shows a MYR merchant.

Source: https://developer.2c2p.com/docs/direct-api-method-click2pay

## Global-Only Pieces (not needed for most Malaysian merchants)

- Internet banking (IMBANK) agents listed are Thailand, Indonesia and Japan banks; Malaysian banks are under WEBPAY (FPX).
- Self service machines (ATM / KIOSK), deep link, ODD, WPC, crypto, LIPP, WIPP and card loyalty channels list no Malaysian rows in the channel reference except as noted above.
- Airline, 3DS2 risk and account funding fields belong to the Payment Token request (`2c2p-api.md`).

Source: https://developer.2c2p.com/docs/reference-payment-channels

## Security Rules For Direct Integrations

- Create the Payment Token server-side only; the secret key never reaches the browser.
- Prefer `securePayToken` from the 2C2P JS SDK over raw `cardNo` / `securityCode`; never log card fields.
- Treat Do Payment `2000`, Transaction Status, iFrame postMessage and Drop-In callbacks as UI signals only. Settle from a verified backend notification JWT or a server-side Payment Inquiry, checking `invoiceNo`, `amount`, `currencyCode`, `merchantID` and `respCode` `0000`.
- Hide channels where `isDown` is `true` and respect `payment.input` / `validation` from Payment Option Details.
- Payment tokens are valid 30 minutes; create a new token (new `invoiceNo`) for a retry after expiry.

Source: https://developer.2c2p.com/docs/response-code-payment-flow, https://developer.2c2p.com/docs/api-payment-token-response-parameter, https://developer.2c2p.com/docs/api-transaction-status-inquiry, https://developer.2c2p.com/docs/encryption-of-card-data-information
