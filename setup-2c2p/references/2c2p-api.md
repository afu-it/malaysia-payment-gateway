# 2C2P PGW API Reference (entry point)

Use this file first for any 2C2P (PGW v4.x) server integration: environments, JWT signing, the full endpoint catalogue, the hosted redirect flow (Payment Token), backend and frontend notifications, Payment Inquiry, settlement rules and sandbox data. Topic files hold the rest:

| File | Covers |
| --- | --- |
| `2c2p-direct-api.md` | Own checkout UI: Payment Option, Payment Option Details, Do Payment, Transaction Status, Secure Pay JS card encryption, iFrame, Drop-In UI, Apple Pay, Google Pay, Malaysian channel shapes for Do Payment. |
| `2c2p-maintenance.md` | Refund, void, settle (capture), maintenance inquiry, refund status, refund notification, recurring plan maintenance, card token and customer token maintenance, FX rate inquiry, JWE + JWS key setup, legacy PKCS7/hash versions. |
| `2c2p-codes.md` | All payment response codes, flow codes, maintenance result and status codes, Malaysian channel, agent and scheme codes, FPX code migration, cut-off times, test cards. |
| `2c2p-mobile-sdk.md` | Android, iOS, Flutter and React Native PGW SDK. |
| `2c2p-quickpay-payout.md` | QuickPay payment links (HMAC-SHA1) and Payout / disbursement API (JWT). |
| `2c2p-batch-recon.md` | SFTP batch files (authorization, refund, token, QuickPay, promo), reconciliation report columns, SoftPOS, plugins, SNAP (Indonesia, out of scope). |

Checked against the official docs at `https://developer.2c2p.com` on 2026-10-21. Index used: `https://developer.2c2p.com/llms.txt`.

## Environments

| Surface | Sandbox | Production |
| --- | --- | --- |
| PGW API (Payment Token, Inquiry, Direct API) | `https://sandbox-pgw.2c2p.com` | `https://pgw.2c2p.com` |
| Hosted payment page UI | `https://sandbox-pgw-ui.2c2p.com` | `https://pgw-ui.2c2p.com` |
| Payment Maintenance (refund, void, settle) | `https://demo2.2c2p.com/PaymentAction/2.0/action` | `https://t.2c2p.com/PaymentAction/2.0/action` |
| Customer Token Maintenance | `https://sandbox-pgw.2c2p.com/customertoken/3.0/CustomerToken/...` | `https://pgw.2c2p.com/customertoken/3.0/CustomerToken/...` |
| QuickPay | `https://demo2.2c2p.com/QuickPay/DirectAPI` | `https://t.2c2p.com/QuickPay/DirectAPI` |
| Payout | `https://sandbox-pgw.2c2p.com/payouts/api/v1.2/...` | `https://pgw.2c2p.com/payouts/api/v1.2/...` |

Indonesia has its own production hosts (`https://pgw.dp.alipay.com/`, `https://pgwcore.dp.alipay.com/PaymentAction/2.0/action`, `pgw-id.2c2p.com` for payouts). Malaysian merchants use the hosts in the table.

Version note: the narrative pages show `/payment/4.5/<operation>` (camelCase, for example `/payment/4.5/paymentToken`). The sandbox setup page uses `/payment/4.3/paymentToken`. The OpenAPI reference is titled "PGW Payment API 4.3" and lists lowercase paths such as `/payment/4.3/paymenttoken`. The v4.3.0 changelog calls 4.3.0 the "main document version" and versioned doc sets also exist for 4.5.0 and 4.0.2. Pick one version, use it for every call in a flow, and confirm with 2C2P which version your merchant profile uses. Path case sensitivity: Not documented in official docs (checked 2026-10-21).

Source: https://developer.2c2p.com/docs/reference-environment-guide, https://developer.2c2p.com/docs/sandbox-setup, https://developer.2c2p.com/docs/api-payment-token, https://developer.2c2p.com/reference/post_payment-4-3-paymenttoken, https://developer.2c2p.com/changelog/payment-v43

## Authentication And Signatures

2C2P uses four different security schemes. Use the right one per API.

| API family | Scheme | Key material |
| --- | --- | --- |
| PGW Payment API (Payment Token, Payment Inquiry, backend notification, Payment Instruction, Payment Simulate, Exchange Rate secure) | JWT, HS256 (HMAC SHA-256) | Merchant Secret Key ("SHA Key") |
| PGW Direct API UI calls (Do Payment, Payment Option, Transaction Status and similar) | Plain JSON keyed by `paymentToken`; no signature (see `2c2p-direct-api.md`) | none |
| Payment Maintenance, Customer Token Maintenance, Recurring Maintenance, Card Token Maintenance, FX Rate inquiry | JWE (RSA-OAEP + A256GCM) wrapped in JWS (PS256) | 2C2P public key + merchant RSA key pair |
| QuickPay | HMAC-SHA1 hash field (`hashValue`), Base64 JSON body | Merchant Secret Key |
| Payout | JWT HS256 `{"payload"}`; payout notification carries an `x-signature` header | Merchant Secret Key |

### PGW JWT (HS256), exact procedure

1. Build the request JSON (for example the Payment Token fields below).
2. Create a JWT with header `{"alg":"HS256","typ":"JWT"}` and the request JSON as the JWT claims (payload).
3. Sign with HMAC SHA-256 using the merchant Secret Key. The official C#, Java and PHP samples pass the Secret Key string exactly as issued (the 64-character value); they do not hex-decode it.
4. POST `{"payload":"<jwt>"}` with `Content-Type: application/json`.
5. The response body is `{"payload":"<jwt>"}`. Verify the HS256 signature with the same Secret Key, then read the claims. If the response is not a JWT (for example a plain `respCode` error), treat it as a failure.

JWT signing input is the standard `base64url(header) + "." + base64url(payload)`. No extra claims (`iat`, `exp`) are required by the docs.

```js
// Node.js, server side only. Library: any RFC 7519 JWT library, for example jsonwebtoken.
const jwt = require("jsonwebtoken");
const SECRET = process.env.TWOC2P_SECRET_KEY; // never hard-code

async function pgwCall(path, body) {
  const token = jwt.sign(body, SECRET, { algorithm: "HS256", noTimestamp: true });
  const res = await fetch(`${process.env.TWOC2P_PGW_BASE}${path}`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ payload: token }),
  });
  const json = await res.json();
  if (!json.payload) throw new Error(`2C2P error ${json.respCode} ${json.respDesc}`);
  return jwt.verify(json.payload, SECRET, { algorithms: ["HS256"] }); // throws if forged
}
```

`noTimestamp` keeps the payload identical to the documented samples; adding `iat` is not documented either way.

Source: https://developer.2c2p.com/docs/json-web-tokens-jwt, https://developer.2c2p.com/recipes/prepare-request-payload-with-jwt, https://developer.2c2p.com/recipes/process-response-payload-with-jwt, https://developer.2c2p.com/docs/sandbox-setup

## Endpoint Catalogue

Every documented operation. All are `POST` unless stated. "PGW host" means `https://sandbox-pgw.2c2p.com` or `https://pgw.2c2p.com`.

### PGW Payment API (JWT or plain JSON)

| Operation | Path (docs pages, 4.5) | Path (OpenAPI 4.3) | Body | Purpose | Detail |
| --- | --- | --- | --- | --- | --- |
| Payment Token | `/payment/4.5/paymentToken` | `/payment/4.3/paymenttoken` | JWT | Create a payment session; returns `paymentToken` and `webPaymentUrl`. | this file |
| Payment Inquiry | `/payment/4.5/paymentInquiry` | `/payment/4.3/paymentinquiry` | JWT | Server-side authoritative payment result. | this file |
| Payment Option | `/payment/4.5/paymentOption` | `/payment/4.3/paymentoption` | JSON | List enabled channel categories for a token. | direct-api |
| Payment Option Details | `/payment/4.5/paymentOptionDetails` | `/payment/4.3/paymentoptiondetails` | JSON | Channels, required inputs, IPP plans. | direct-api |
| Do Payment | `/payment/4.5/payment` | `/payment/4.3/payment` | JSON | Submit the chosen channel and data. | direct-api |
| Transaction Status | `/payment/4.5/transactionStatus` | `/payment/4.3/transactionstatus` | JSON | UI polling only. Not for settlement. | direct-api |
| Card Installment Plan Info | `/payment/4.5/cardinstallmentplaninfo` | `/payment/4.3/cardinstallmentplaninfo` | JSON | Visa IPP plans for a card. | direct-api |
| Loyalty Point Info | `/payment/4.5/loyaltypointinfo` | `/payment/4.3/loyaltypointinfo` | JSON | Card loyalty balance. | direct-api |
| Card Token Info | none | `/payment/4.3/cardtokeninfo` | JSON | Stored tokens for the token's customer. | direct-api |
| Cancel Transaction | none | `/payment/4.3/canceltransaction` | JSON | Cancel in-progress payment for a token. | direct-api |
| Initialization | none | `/payment/4.3/initialization` | JSON | Supported locales. | direct-api |
| User Preference | none | `/payment/4.3/userpreference` | JSON | Saved preference and recent channels. | direct-api |
| Payment Notification | none | `/payment/4.3/paymentnotification` | JSON | Share result to a messaging platform. | direct-api |
| Exchange Rate | none | `/payment/4.3/exchangerate` | JSON | FX rate for a card BIN. | direct-api |
| Exchange Rate (secure) | none | `/payment/4.3/exchangerate/secure` | JWT | Server-side variant. | direct-api |
| APM MCC Exchange Rate | none | `/payment/4.3/exchangerate/apmmccexchangerate` | JSON | APM multi-currency rates. | direct-api |
| Payment Instruction | none | `/payment/4.3/paymentinstruction` | JWT | Fields not documented in official docs (checked 2026-10-21). | direct-api |
| Payment Simulate | none | `/payment/4.3/paymentsimulate` | JWT | Fields not documented in official docs (checked 2026-10-21). | direct-api |
| Apple Pay merchant validation | none | `/payment/4.3/applepay/merchantvalidation` | JSON | Apple Pay merchant session. | direct-api |

### Notifications and redirects (2C2P calls you)

| Operation | Method and target | Body | Detail |
| --- | --- | --- | --- |
| Backend notification | `POST` to `backendReturnUrl` (or merchant portal default) | JSON `{"payload":"<JWT HS256>"}` | this file |
| Frontend return | Browser form `POST` to `frontendReturnUrl` (or portal default) | `application/x-www-form-urlencoded`, field `paymentResponse=<base64url JSON>`, not signed | this file |
| Refund notification | `POST` to `notifyURL` from the refund request | `notifyResponse=<JWS of JWE>` | maintenance |
| Payout notification | `POST` to `notificationUrl` | JSON, headers `x-signature`, `x-notify-event: payout` | quickpay-payout |
| SoftPOS notification | `POST` to endpoint registered at onboarding | JSON envelope with JWT `payload` | batch-recon |

### Payment Maintenance and token maintenance (JWE + JWS)

| Operation | Endpoint | Message | Detail |
| --- | --- | --- | --- |
| Inquiry (`processType` I) | `/PaymentAction/2.0/action` | `PaymentProcessRequest` v4.6 XML | maintenance |
| Refund (R) | same | same | maintenance |
| Refund status (RS) | same | same | maintenance |
| Void / cancel (V) | same | same | maintenance |
| Settle / capture (S) | same | same | maintenance |
| Recurring plan inquiry, update, cancel (I, U, C) | same | `RecurringMaintenanceRequest` v2.4 XML | maintenance |
| Card token add, update, delete, inquiry (A, U, D, I) | same | `MaintenanceRequest` v2.3 XML | maintenance |
| FX rate inquiry, FX rate list inquiry | same | `FxRateRequest`, `FxRateListRequest` v2.1/2.2 XML | maintenance |
| Customer token add / update / get / delete | `/customertoken/3.0/CustomerToken/add`, `/update`, `/get`, `/delete` on PGW host | JSON | maintenance |
| Legacy Payment Action (PKCS7) | `/PaymentActionV2/PaymentAction.aspx` on demo2 / t.2c2p.com | XML, HMAC hash | maintenance |
| Legacy Payment Process (Base64) | `/PaymentActionV2/PaymentProcess.aspx` | XML, HMAC hash | maintenance |

### QuickPay, Payout, SoftPOS, batch, SNAP

| Family | Operations | Detail |
| --- | --- | --- |
| QuickPay DirectAPI (`/QuickPay/DirectAPI`) | Generate (`GenerateQPReq`), Query (`QPQueryReq`), Update (`QPUpdateReq`), Delete (`QPDeleteReq`) | quickpay-payout |
| QuickPay DeliveryAPI (`/QuickPay/DeliveryAPI`) | Generate and Send (`GenerateSendQPReq`), Send (`QPSendReq`) | quickpay-payout |
| Payout | `/payouts/api/v1.2/payout/create`, `/payouts/api/v1.2/payout/inquiry`, `/payouts/api/v1.1/beneficiary/register`, `/payouts/api/v1.1/payout/beneficiary/inquiry`, `/payouts/api/v1.1/merchants/providers`, `/payouts/api/v1.1/merchants/source-of-fund` | quickpay-payout |
| SoftPOS Pay Server (LAN, on device) | `POST /api/v1/activation`, `POST /api/v1/trans`, `GET /api/v1/cancel`, `GET /api/v1/trans/status/{posMessageId}` | batch-recon |
| Batch SFTP | `/input` (pickup), `/output` (drop) on the 2C2P SFTP server | batch-recon |
| SNAP (Indonesia only) | `/snap/v1.0/access-token/b2b` and `/snap/v{version}/...` debit, VA, QR, account binding | batch-recon (listed, out of scope) |

Source: https://developer.2c2p.com/llms.txt, https://developer.2c2p.com/reference/post_payment-4-3-paymenttoken, https://developer.2c2p.com/docs/payment-maintenance-how-it-works, https://developer.2c2p.com/docs/customer-token-maintenance, https://developer.2c2p.com/docs/reference-environment-guide

## Hosted Redirect Flow (recommended default)

1. Create a local order with status `pending`, a unique `invoiceNo`, expected amount and currency.
2. Server: call Payment Token (JWT). Proceed only if `respCode` is `0000`; otherwise stop and show an error.
3. Redirect the browser to `webPaymentUrl` (hosted page on `pgw-ui`). The token is valid for 30 minutes.
4. 2C2P POSTs the backend notification to `backendReturnUrl`. If you do not send `backendReturnUrl`, the merchant portal default is used.
5. 2C2P redirects the browser with a form POST to `frontendReturnUrl` (or the portal default) carrying `paymentResponse` (base64url JSON with `invoiceNo`, `channelCode`, `respCode`, `respDesc`). This is UX only.
6. If you do not run a backend notification handler, you must call Payment Inquiry to get the result. Calling Payment Inquiry on the frontend return as well is a safe pattern.
7. Settle the order only from a verified backend notification or a Payment Inquiry response (see Settlement Checklist).

Iframe mode (`iframeMode: true`): the hosted page posts a browser `postMessage` `paymentResult` with `respCode`, `respDesc`, `respData`. `2000` means completed (then check your backend status), `1001` means redirect to `respData`. 2C2P recommends the Drop-In UI over raw iframes (see `2c2p-direct-api.md`).

Source: https://developer.2c2p.com/docs/redirect-api-how-it-works, https://developer.2c2p.com/docs/redirect-api-integrate-with-payment, https://developer.2c2p.com/docs/using-iframe

## Payment Token Request

`POST {PGW host}/payment/4.5/paymentToken` (or the 4.3 path). JWT HS256. M = mandatory, O = optional, C = conditional.

Core fields:

| Field | Type | M/O | Notes |
| --- | --- | --- | --- |
| `merchantID` | AN 25 | M | Merchant ID from 2C2P. |
| `invoiceNo` | AN 50 | M | Unique merchant order number. Max 20 alphanumeric for QR payments; 12 numerals for Myanmar APM. Reuse returns 9015 / 5005. |
| `description` | C 250 | M | HTML-encode special characters; avoid unnecessary ones. |
| `amount` | D (12,5) | M | Major units with decimals per ISO 4217, for example `150.00` for RM150. Not sen. IDR uses 0 decimals. |
| `currencyCode` | A 3 | M | ISO 4217 alpha, `MYR`. If empty the merchant base currency is used. |
| `idempotencyID` | C 100 | O | Merchant value to recognise retries of the same request. |
| `paymentChannel` | Array AN 1-6 | O | Restrict channels by group code, for example `["CC"]`, `["IPP"]`, `["QR"]`; for 123 channels use the group code column. Empty means all enabled channels. |
| `agentChannel` | Array AN 1-6 | O | For 123 channels, for example `["WEBPAY"]`. |
| `request3DS` | A 1 | O | `Y` enable 3DS (default), `F` force, `N` disable. |
| `paymentExpiry` | C 19 | O | `yyyy-MM-dd HH:mm:ss`; payment after expiry is rejected. |
| `locale` | C 10 | O | Page and API language (see Initialization API). |
| `frontendReturnUrl` | NT | O | Browser return URL. |
| `backendReturnUrl` | NT | O | Server notification URL (ntext since the 2 Apr 2025 change). |
| `userDefined1` to `userDefined5` | C 150 | O | Echoed back in notification and inquiry. |
| `statementDescriptor` | AN 20 | O | 5 to 20 characters; must not contain `<`, `>`, `\`, `'`, `"`, `*`. |
| `promotionCode` | AN 20 | O | Promotion code. |
| `paymentRouteID` | C 255 | O | Custom routing rule. |
| `immediatePayment` | B | O | Trigger payment immediately. |
| `iframeMode` | B | O | Hosted page runs inside an iframe. |
| `nonceStr` | C 32 | O | Random nonce. |
| `clientIP` | AN 45 | O | Customer IP; taken from request header if empty. |
| `userAgent` | C 2048 | O | Payer browser user-agent. |
| `schemeReturnUrl` | C 512 | O | App return scheme for mobile. |
| `appBundleID` | C 255 | C | Required with `schemeReturnUrl`; include when Apple Pay or Google Pay run in an app WebView. |
| `clientAppID` | AN 45 | O | Used by mobile SDK. |
| `allowCustomerNote` | B | O | Let customer add notes. |
| `transactionMode` | AN 20 | O | For manual-settle merchants, for example pre-authorisation (`PREAUTH` in samples). |
| `childMerchantID` | AN 25 | C | Sub account of `merchantID`. |

Tokenisation and card-on-file:

| Field | Type | M/O | Notes |
| --- | --- | --- | --- |
| `tokenize` | B | O | Show "save card" on the page; token returned when approved. |
| `customerToken` | Array C 255 | O | Registered tokens to offer for payment. |
| `customerTokenOnly` | B | O | Allow only stored tokens. |
| `tokenizeOnly` | B | O | `true` tokenise without authorisation; `false` with authorisation (default). |
| `storeCredentials` | A 1 | O | `F` first payment, `S` subsequent, `N` not used (acquirer must support COF). |
| `transactionInitiator` | C 1 | O | `C` customer (default), `M` merchant. |
| `previousPaymentID` | C 255 | C | Required if `transactionInitiator` is `M`; send the `schemePaymentID` from the first CIT. |
| `externalToken` | C max | O | External token for specific channels. |
| `scaExemptionIndicator` | C 255 | O | `authentication_outage`, `delegated_authentication`, `low_value`, `low_risk`, `secure_corporate_payment`, `trusted_merchant`, `recurring_payment`, `out_of_sca_scope` (MIT), `transaction_risk_assessment`, `other`, `none` (default). |
| `allow3DSUpgrade` | A 1 | O | With `request3DS=Y`, allow upgrade to 3DS if exemption fails. |
| `requestReauthentication` | B | O | Request re-authentication. |
| `protocolVersion` | AN 10 | O | 3DS protocol, default `2.1.0`. |
| `eci`, `cavv`, `dsTransactionId` | N 2, AN 40, C 36 | C | External 3DS results; send together. |

CIT: `transactionInitiator=C`, `request3DS` `Y`/`F` (or `N`), `storeCredentials` `F` (first) or `S`, `tokenize` if using 2C2P tokens; store `schemePaymentID` and `customerToken` from the notification. MIT: `transactionInitiator=M`, `request3DS=N`, `storeCredentials=S`, `previousPaymentID=<schemePaymentID>`, `customerToken`. Old subscriptions without a `schemePaymentID` need a one-time 0-amount CIT non-3DS authorisation (acquirer dependent; tell the cardholder first).

Instalment (IPP) and recurring (RPP):

| Field | Type | M/O | Notes |
| --- | --- | --- | --- |
| `interestType` | A 1 | O | `A` all (default), `C` customer pays, `M` merchant pays. |
| `installmentPeriodFilter` | Array N 2 | O | Allowed tenors, for example `[3,6]`. |
| `installmentBankFilter` | Array | O | Allowed IPP banks. |
| `productCode` | AN 50 | O | Instalment product code. |
| `recurring` | B | O | Enable RPP. |
| `invoicePrefix` | AN 15 | C | Mandatory if recurring; 2C2P appends 5 digits per cycle. |
| `recurringAmount` | D (12,5) | O | Defaults to `amount`. |
| `allowAccumulate` | B | C | Mandatory if recurring; accumulate failed amounts. |
| `maxAccumulateAmount` | D (12,5) | C | Mandatory if recurring; plan terminates when exceeded. |
| `recurringInterval` | N 3 | C | Days between charges. Use this or `chargeOnDate`, not both. |
| `recurringCount` | N 5 | C | Mandatory if recurring; `0` = until terminated. |
| `chargeNextDate` | N 8 | O | `ddMMyyyy`; needed with `recurringInterval`. |
| `chargeOnDate` | N 4 | C | `ddMM` day-of-month charging. |

Other objects (all optional): `fxProviderCode`, `fxRateId`, `originalAmount` (multi-currency); `externalSubMerchantID`; `subMerchants[]` (`merchantID` AN 25, `invoiceNo` AN 20, `amount` D 12,5, `description` C 255, `loyaltyPoints`, `originalAmount`); `paymentItems[]` (`code`, `name`, `quantity`, `price`; max 90 records); `uiParams.userInfo` (`name` C 50, `email` C 150, `mobileNo` N 15, `countryCode` A 2, `mobileNoPrefix` N 2, `currencyCode` A 3); `customerAddress.billing` / `.shipping` (`address1..3`, `city`, `state`, `postalCode` AN 10, `countryCode` A 2); `airlinePassengers` (airline data); `3DSecure2Params` (`payer`, `order`, `deviceChannel`, `browserInfo` with `browserAcceptHeader`, `browserIP`, `browserJavaEnabled`, `browserLanguage`, `browserColorDepth`, `browserScreenHeight`, `browserScreenWidth`, `browserTZ`, `browserUserAgent`, `browserJavaScriptEnabled`); `loyaltyPoints`; `browserDetails`; `defaultSettlementCurrencyMerchantID`; `settlementCurrencyMerchantID`; `accountFunding` (sender and recipient identity, for business codes that require it); `riskInfo` (`customer_id`, `shipping_method`, `customData`).

Minimal Malaysian request (claims before signing):

```json
{
  "merchantID": "<MERCHANT_ID>",
  "invoiceNo": "INV20261021001",
  "description": "Order 1001",
  "amount": 150.00,
  "currencyCode": "MYR",
  "frontendReturnUrl": "https://shop.example.my/2c2p/return",
  "backendReturnUrl": "https://shop.example.my/2c2p/notify",
  "userDefined1": "<LOCAL_ORDER_ID>"
}
```

Source: https://developer.2c2p.com/docs/api-payment-token, https://developer.2c2p.com/docs/api-payment-token-request-parameter, https://developer.2c2p.com/docs/reference-card-on-file-cit-mit, https://developer.2c2p.com/docs/reference-convert-existing-transactions-to-card-on-file-model

## Payment Token Response

Decoded JWT claims:

| Field | Type | M/O | Notes |
| --- | --- | --- | --- |
| `paymentToken` | C 255 | M | Valid for 30 minutes. Used by Direct API and SDK. |
| `webPaymentUrl` | C 255 | O | Hosted page URL to redirect to. |
| `respCode` | N 4 | M | `0000` success; see `2c2p-codes.md`. |
| `respDesc` | C 255 | M | Description. |

Source: https://developer.2c2p.com/docs/api-payment-token-response-parameter

## Backend Notification (server to server)

2C2P POSTs JSON `{"payload":"<JWT>"}` to `backendReturnUrl`. Verify HS256 with the Secret Key, then read the claims. The decoded claims use the same structure as the Payment Inquiry response.

| Field | Type | Notes |
| --- | --- | --- |
| `merchantID` | C 25 | Must equal your merchant ID. |
| `childMerchantID` | AN 25 | Sub account, if used. |
| `invoiceNo` | AN 50 | Your order number; match to the local order. |
| `amount` | D (12,5) | Compare to the expected amount. |
| `currencyCode` | A 3 | Compare to the expected currency. |
| `transactionDateTime` | N 14 | `yyyyMMddHHmmss`. |
| `agentCode` | AN 30 | Agent that processed the payment (see agent codes). |
| `channelCode` | AN 30 | Channel code. |
| `approvalCode` | C 6 | Mandatory for card payments. |
| `referenceNo` | AN 50 | Host reference number. |
| `tranRef` | AN 28 | 2C2P routing reference. |
| `accountNo` | N 19 | Masked card, first 6 and last 4. |
| `customerToken` / `customerTokenExpiry` | AN 20 / AN 8 | Stored card token and expiry `yyyyMMdd` if tokenised (older versions name it `cardToken`). |
| `cardType` | C 20 | `PREPAID`, `DEBIT`, `CREDIT`. |
| `issuerCountry` / `issuerBank` | A 2 / C 200 | Card issuer. |
| `eci` | C 2 | Card ECI. |
| `installmentPeriod`, `interestType`, `interestRate`, `installmentMerchantAbsorbRate` | | IPP details. |
| `recurringUniqueID` | N 20 | RPP plan ID; keep it for recurring maintenance. |
| `recurringSequenceNo` | N 10 | Cycle number. |
| `fxAmount`, `fxRate`, `fxCurrencyCode` | | MCP details. |
| `userDefined1` to `userDefined5` | C 150 | Echo of your values. |
| `acquirerReferenceNo`, `acquirerMerchantId` | C 50 | Acquirer references. |
| `idempotencyID` | C 100 | Echo. |
| `paymentScheme` | C 30 | Scheme code, for example `VI`, `MA`, `DU`, `TG`. |
| `loyaltyPoints` | Array | Loyalty redemption details. |
| `paymentID` | C 255 | 2C2P payment ID, for example `ccpp_12345678`. |
| `schemePaymentID` | C 255 | Network payment ID; store for MIT `previousPaymentID`. |
| `acquirerResponseCode` | N 2 | ISO 8583 response code when available. |
| `respCode` | C 4 | `0000` success, `0001` pending, others failed (see codes file). |
| `respDesc` | C 255 | Description. |

Acknowledgement format, retry schedule and source IP ranges for the PGW backend notification: Not documented in official docs (checked 2026-10-21). Return HTTP 200 quickly and process idempotently; reconcile with Payment Inquiry for anything you did not receive.

Source: https://developer.2c2p.com/docs/api-payment-response-backend, https://developer.2c2p.com/docs/api-payment-response-back-end-parameter

## Frontend Return (browser)

Form POST, `Content-Type: application/x-www-form-urlencoded`, single field `paymentResponse` whose value is base64url-encoded JSON:

| Field | Type | Notes |
| --- | --- | --- |
| `invoiceNo` | AN 50 | Order number. |
| `channelCode` | C 255 | Channel used. |
| `respCode` | N 4 | Payment flow code (`2000` = completed, check backend or call inquiry). |
| `respDesc` | C 255 | Description. |

The page header claims JWT, but the documented sample is plain base64url JSON with no signature. Never settle from it.

Source: https://developer.2c2p.com/docs/api-payment-response-frontend, https://developer.2c2p.com/docs/api-payment-response-front-end-parameter

## Payment Inquiry

`POST {PGW host}/payment/4.5/paymentInquiry` (or `/payment/4.3/paymentinquiry`). JWT HS256 request and response.

Request:

| Field | Type | M/O | Notes |
| --- | --- | --- | --- |
| `merchantID` | AN 25 | M | |
| `paymentToken` | C 255 | C | Either `paymentToken` or `invoiceNo` is required. |
| `invoiceNo` | AN 50 | C | Either `paymentToken` or `invoiceNo` is required. |
| `childMerchantID` | AN 25 | C | Sub account. |
| `locale` | C 10 | O | Response language. |

Response: same fields as the backend notification table (including `loyaltyPoints`, `paymentID`, `schemePaymentID`, `respCode`, `respDesc`).

Transaction Status (`/payment/4.5/transactionStatus`) is a different, unsigned UI polling call; 2C2P says not to use it to confirm payment. Use Payment Inquiry from the server.

Source: https://developer.2c2p.com/docs/api-payment-inquiry, https://developer.2c2p.com/docs/api-payment-inquiry-request-parameter, https://developer.2c2p.com/docs/api-payment-inquiry-response-parameter, https://developer.2c2p.com/docs/api-transaction-status-inquiry

## Status Values To Act On

| Source | Value | Meaning | Action |
| --- | --- | --- | --- |
| Payment Token | `0000` | Token created | Redirect. Anything else: stop. |
| Notification / Inquiry `respCode` | `0000` | Successful | Settle after checks. |
| | `0001` | Pending (for example payslip, async APM) | Keep pending; wait for notification or re-inquire. |
| | `0003` | Cancelled | Mark failed or cancelled. |
| | `0004` | Soft decline, needs 3DS | Retry with 3DS. |
| | `2001` | In progress | Re-inquire later. |
| | `2002` | Not found | Not paid (yet); do not settle. |
| | `4xxx`, `5xxx`, `9xxx` | Declined or error | Mark failed (see codes file). |
| Frontend / Do Payment flow | `1000` to `1005`, `2000` | Next UI step | See flow table in `2c2p-codes.md`. |

Source: https://developer.2c2p.com/docs/response-code-payment, https://developer.2c2p.com/docs/response-code-payment-flow

## Amount Rules

- `amount` is a decimal in major currency units with up to 5 decimals in the type, limited to the ISO 4217 decimals of the currency. For MYR send ringgit with 2 decimals (`150.00`), never sen.
- Maintenance `actionAmount` is 2-decimal format (`25.00`).
- Recurring maintenance `amount` and `maxAccumulateAmount` are 12-digit zero-padded with 2 implied decimals and no decimal point: the docs sample returns `000000009990` for 99.90. One docs inquiry sample sends `99.90` instead, so send the 12-digit form for updates as the parameter table says.
- Batch refund files use 12-digit amounts without a decimal point.

Source: https://developer.2c2p.com/docs/api-payment-token-request-parameter, https://developer.2c2p.com/docs/payment-maintenance-refund-guide, https://developer.2c2p.com/docs/api-parameters-2, https://developer.2c2p.com/docs/batch-services-refund

## Settlement Checklist (mark paid only when all pass)

1. Body is `{"payload": ...}` and the JWT verifies with HS256 and the Secret Key for this environment. Reject `alg` other than HS256.
2. `merchantID` (and `childMerchantID` if used) equals yours.
3. `invoiceNo` maps to a local pending order.
4. `amount` and `currencyCode` equal the stored expected values.
5. `respCode` is `0000`.
6. The order is not already settled (idempotent update keyed on `invoiceNo`, plus `paymentID` or `tranRef`).
7. If anything is missing or late, call Payment Inquiry and apply the same checks.
8. Store `paymentID`, `schemePaymentID`, `customerToken`, `recurringUniqueID`, `referenceNo`, `approvalCode` for refunds, MIT and reconciliation.

```js
// Express-style backend notification handler (sketch)
app.post("/2c2p/notify", express.json(), async (req, res) => {
  let claims;
  try { claims = jwt.verify(req.body.payload, SECRET, { algorithms: ["HS256"] }); }
  catch { return res.status(400).end(); }
  const order = await db.orders.findByInvoice(claims.invoiceNo);
  const ok = order && claims.merchantID === MERCHANT_ID &&
    claims.currencyCode === order.currency &&
    Number(claims.amount).toFixed(2) === Number(order.amount).toFixed(2) &&
    claims.respCode === "0000";
  if (ok) await db.orders.markPaidOnce(order.id, claims.paymentID ?? claims.tranRef);
  else if (order) await db.orders.recordAttempt(order.id, claims.respCode);
  res.status(200).end();
});
```

## Sandbox Data (sandbox only)

- Published demo merchant for Malaysia: Merchant ID `JT07`, currency MYR / 458, Secret Key `9E936798778E8E21ABA8E7B620EF631E0C957BB1ADA14E4960022307A9726A09`. Public demo values for sandbox only; never use in production. Singapore `JT01` is used in most samples.
- Test cards and 1-2-3 test login: see `2c2p-codes.md`.
- FPX sandbox test banks: agent codes `MYSBIA`, `MYSBIB`, `MYSBIC`.
- Go-live: get production Merchant ID and Secret Key from 2C2P, switch the base URL to `https://pgw.2c2p.com`, point `backendReturnUrl` at production, run a small live transaction.

Source: https://developer.2c2p.com/docs/sandbox, https://developer.2c2p.com/docs/sandbox-setup, https://developer.2c2p.com/docs/reference-test-information-my

## Not Documented (checked 2026-10-21)

- Rate limits for PGW APIs.
- Source IP ranges or allowlists for backend notifications.
- Backend notification retry schedule and required response body.
- Whether URL paths are case-sensitive.
