# AsiaPay (PayDollar) API Reference

Use this reference for factual AsiaPay/PayDollar integration guidance. Facts come from the official developer portal `https://developer.asiapay.com/integration-guide/...` (checked 2026-10-21), the official PDF guides on `paydollar.com` and `github.com/asiapay-lib`, and the official WooCommerce plugin source. Page paths below are relative to `https://developer.asiapay.com/integration-guide`. Full URLs are in `source-map.md`.

The developer portal mostly lists test URLs. Production URLs come from the official PDF guides (PayGate Integration Guide v3.67, Merchant API OnlineRefund v1.2, Membership and MemberPay v2.2) and the official WooCommerce plugin. Confirm them with AsiaPay before go-live.

## Platform Overview

- PayDollar is AsiaPay's gateway. SiamPay (Thailand) and PesoPay (Philippines) use the same API on `siampay.com` / `pesopay.com`. Malaysian merchants use PayDollar.
- Connection methods (`/introduction`, `/web-integration`):
  - Client Post Through Browser: merchant posts a form to the PayDollar hosted payment page. Simplest; PayDollar collects card data and handles 3DS.
  - Direct Client Side Connection: merchant page collects card data and posts it from the browser to PayDollar. Needs SSL and puts the merchant in PCI scope.
  - Server Side Direct Connection: merchant server posts payment data and reads the result synchronously. Used for IVR, apps, app-to-app wallets and wallet direct integrations.
  - Mobile: PaySDK for Android and iOS, eWallet Direct Connection, and the Tokenization Flow (PaySDK REST API with JWE).
- Result delivery: browser redirect to `successUrl` / `failUrl` / `cancelUrl` (display only) plus the server-to-server datafeed (the only safe status source).
- Merchant API: server-side form POST for Query, Capture, Void, RequestRefund, Cancel, QueryRequestLog, OnlineReversal, and the settlement report.

## Endpoint Catalogue

Test base: `https://test.paydollar.com/b2cDemo`. Production base: `https://www.paydollar.com/b2c2`.

| Method | Path (append to base) | Purpose | Source |
| --- | --- | --- | --- |
| POST (browser form) | `/eng/payment/payForm.jsp` | Client Post Through Browser: hosted payment page | `/web-integration/client-post-through-browser/sample-code`; live URL from PDF v3.67 |
| POST (browser form) | `/eng/dPayment/payComp.jsp` | Direct Client Side Connection (card data from merchant page) | `/web-integration/direct-client-side-connection/sample-code`; live URL from PDF v3.67 |
| POST (server) | `/eng/directPay/payComp.jsp` | Server Side Direct Connection and eWallet Direct | `/web-integration/server-side-direct-connection/sample-code`; live URL from PDF v3.67 |
| POST (PayDollar to merchant) | merchant "Return Value Link" | Datafeed (payment result callback) | `/data-feed-handling` |
| POST (server) | `/eng/merchant/api/orderApi.jsp` | Merchant API: `Capture`, `Void`, `RequestRefund`, `Query`, `QueryRequestLog`, `Cancel` | `/functions-of-merchant-api/introduction`; live URL from PDF v3.67 |
| POST (server) | `/eng/merchant/api/onlineApi.jsp` | Merchant API: `OnlineReversal` (void, online refund, partial refund) | `/functions-of-merchant-api/void-online-refund-or-partial-refund`; live URL from OnlineRefund PDF v1.2 |
| POST (server) | `/GenTxnXML` | Settlement report request (XML) | `/functions-of-merchant-api/settlement-report-request`; live URL from PDF v3.67 |
| POST (server) | `/eng/merchant/api/MembershipApi.jsp` | Membership API: `Query`, `Add`, `Update`, `Delete` member | MemberPay PDF v2.2 |
| POST (server) | `/eng/merchant/api/MemberPayApi.jsp` | MemberPay API: `QueryAccount`, `Add`, `Update`, `Delete`, `ChangePassword`, `GenerateToken`, `ResetStaticToken` | MemberPay PDF v2.2 |
| GET (browser) | `/eng/merchant/index.jsp` | Merchant administration portal (test). Live: `https://admin.paydollar.com/b2c2/eng/merchant/index.jsp` | `/mobile-integration/tokenization-flow/get-started-with-paydollar-api` |
| POST JSON | `https://uat4.paydollar.com/paysdk-api/auth` | PaySDK API: get access token (JWE) | `/mobile-integration/tokenization-flow/get-started-with-paydollar-api` |
| POST JSON | `https://uat4.paydollar.com/paysdk-api/order/initiatetoken` | PaySDK API: create payment token for the app | `/mobile-integration/tokenization-flow/initiate-payment-token` |
| POST JSON | `https://uat4.paydollar.com/paysdk-api/query/txquery` | PaySDK API: query transaction | `/mobile-integration/tokenization-flow/query-api` |
| POST JSON | `https://uat4.paydollar.com/paysdk-api/tx/refund` | PaySDK API: refund | `/mobile-integration/tokenization-flow/refund-api` |

Production hosts for the PaySDK REST API (`paysdk-api`): Not documented in official docs (checked 2026-10-21). Ask AsiaPay.

MCP exchange-rate enquiry (`MCPMultiRateEnquiry`) is referenced but its endpoint is not documented in the public portal (checked 2026-10-21).

## Authentication Model

- Payment requests: no API key. Integrity comes from `secureHash` (see below) once AsiaPay enables Secure Hash on the account. Without it, any party can post a form with your `merchantId`.
- Merchant API: `merchantId`, `loginId` (Text 30) and `password` (Text 15) in the POST body. AsiaPay issues the API login on request (`/functions-of-merchant-api/introduction`). Membership/MemberPay APIs use `merchantApiId` and `password` (Text 20).
- PaySDK REST API: exchange master merchant ID + merchant ID (`user` = `"<masterId>-<merchantId>"`) and a JWE built with the AsiaPay RSA public key for a bearer token. Client secret and RSA public key are in merchant admin > Profile > Account Information > Payment Account Information.
- Security page (`/security/backend-configuration`): AsiaPay can whitelist your API server IPs and the domain allowed to iframe its payment page.

## Common Request Rules

- Content type `application/x-www-form-urlencoded`, UTF-8 encoding, case-sensitive field names (`/web-integration/client-post-through-browser/integration-parameters`).
- `amount` is `Number(12,2)`: a decimal ringgit amount with up to 2 decimals. It is not sen. Example: `amount=3000` means 3000.00 in the account currency.
- `currCode` is the ISO 4217 numeric code as text. MYR is `"458"`.
- One merchant ID accepts one currency. Error "CurrCode is incorrect" means the currency does not match the merchant ID (`/faq/common-problems`).
- Error "Your payment service is not active" means test merchant ID used on production URL or the reverse. Test IDs are 6 digits; production IDs are 4 or 8 digits.
- `orderRef` is Text(35). Duplicate references can be rejected by fraud rules (PRC `-8`, SRC `2007` / `2008`), so use one unique `orderRef` per attempt you want to track, or expect retries under the same reference.

### Currency codes (currCode / Cur)

`344` HKD, `840` USD, `702` SGD, `156` CNY, `392` JPY, `901` TWD, `036` AUD, `978` EUR, `826` GBP, `124` CAD, `446` MOP, `608` PHP, `764` THB, `458` MYR, `360` IDR, `410` KRW, `682` SAR, `554` NZD, `784` AED, `096` BND, `704` VND, `356` INR.

### Language codes (lang)

`E` English, `C` Traditional Chinese, `X` Simplified Chinese, `J` Japanese, `T` Thai, `F` French, `G` German, `R` Russian, `S` Spanish, `V` Vietnamese. Server Side Direct documents only `E`.

## Client Post Through Browser (hosted page)

Test URL `https://test.paydollar.com/b2cDemo/eng/payment/payForm.jsp`. Live URL `https://www.paydollar.com/b2c2/eng/payment/payForm.jsp`.

Browser requirements in docs: HTML 4.0, JavaScript 1.3, IE 11+, Firefox 27+, Chrome 30+, Safari 7+. Do not put the post-submit flow in frames; some issuers' 3DS pages need full-page cookies (`/faq/3-d-secure-authentication`). Iframe embedding requires AsiaPay to enable your domain (`/security/backend-configuration`).

### Required fields

| Field | Type | Notes |
| --- | --- | --- |
| `merchantId` | Number | Merchant ID issued by AsiaPay. |
| `orderRef` | Text(35) | Merchant order reference. |
| `currCode` | Text(3) | Numeric currency code. `458` for MYR. With `mpsMode=SCP` use foreign currency; with `MCP` use base currency. |
| `amount` | Number(12,2) | Total to charge. SCP: foreign amount; MCP: base amount. |
| `lang` | Text(1) | Payment page language (codes above). |
| `cancelUrl` | Text(300) | Redirect on customer cancel. Display only. |
| `failUrl` | Text(300) | Redirect on rejection. Display only. |
| `successUrl` | Text(300) | Redirect on acceptance. Display only. |
| `payType` | Text(1) | `N` normal sale, `H` hold (authorise only). Capture holds within 14 days. Hold not available for 99BILL, ALIPAY, CHINAPAY, PAYPAL, PPS, TENPAY, WECHAT, MEPS, OCTOPUS. |
| `payMethod` | Text | Payment method code, or `ALL` for every method on the account. List below. |
| `departureDate` | Text(10) | `yyyy-MM-dd`. Only required for `SLICEPAY`. |

The redirect URLs receive only `Ref` (merchant order reference) for display (`/web-integration/client-post-through-browser/3ds-2-0/redirect-url`). A customer can call them directly with any `Ref`, so they prove nothing.

### payMethod codes (Client Post)

Malaysia-relevant codes in the documented list: `ALL`, `CC` (credit card), `VISA`, `Master`, `AMEX`, `JCB`, `Diners`, `CHINAPAY` / `UPOP` (UnionPay), `FPX` (FPX net banking Malaysia), `DuitNow`, `TouchnGo`, `GRABPAY`, `SHOPEEPAY`, `ALIPAY`, `ALIPAYPLUS`, `WECHATONL`, `PAYPAL`, `ATOME`, `M2U`, `CIMBCLICK`, `MEPS`, `CLICKTOPAY`, `APPLEPAY`, `GOOGLE`, `SAMSUNG`.

Full documented list: `ALIPAY`, `ALIPAYHKONL`, `ALIPAYPLUS`, `AMEX`, `ATOME`, `AU`, `BANKTRANSFERJPN`, `BillDesk`, `BPI`, `BPM`, `BoCPay`, `Brankas`, `CC`, `CHINAPAY`, `CIMBCLICK`, `CLICKTOPAY`, `DANA`, `Diners`, `DuitNow`, `ENETS`, `ENETSBANKING`, `ENETSQR`, `Eximbay`, `FPX`, `FPS`, `GCash`, `GRABPAY`, `HANDSIN`, `HOOLAH`, `humm`, `IBANKING`, `IOUPay`, `JCB`, `JKOPAY`, `KONBINI`, `KREDIVO`, `KRUNGSRIONLINE`, `KTB`, `LINEPAY`, `M2U`, `Master`, `MERPAY`, `MEPS`, `Momopay`, `MP`, `NAPAS`, `NICEPAY`, `NTTDOCOMO`, `OCTOPUS`, `OnePay`, `OVO`, `PACE`, `PAIDY`, `PAYEASY`, `PAYID`, `PayMe`, `PayMaya`, `PAYPAL`, `PAYPAY`, `PAYU`, `PAYSBUY`, `POLI`, `PPS`, `QRIS`, `SCB`, `SGQR`, `SHOPEEPAY`, `SLICEPAY`, `SOFTBANK`, `TendoPay`, `TENPAY`, `TMB`, `TOSS`, `TouchnGo`, `TRUEMONEY`, `UOB`, `UPOP`, `VCO`, `VISA`, `WECHAT` (deprecated), `WECHATONL`, `WELEND`, `ZIPPAY`, `99BILL`, plus device wallets `APPLEPAY`, `GOOGLE`, `SAMSUNG`.

The Malaysia method table (`/payment-methods-by-regions-countries`) also lists BOOST, Scan & Pay by MAE, ShopBack Pay and PayLater by Grab, and asiapay.com.my lists SPayLater. No `payMethod` code is documented for these (checked 2026-10-21). Use `payMethod=ALL` and confirm codes with AsiaPay.

### Optional fields (Client Post)

Connect to payment page (`.../optional-parameters/connect-to-payment-page`):

| Field | Type | Notes |
| --- | --- | --- |
| `secureHash` | Text(40) | Request hash. Mandatory once Secure Hash is enabled. |
| `remark` | Text(50) | Free data returned in datafeed, not shown to payer. |
| `deviceMode` | Text | `auto` (default), `pc`, `mobile`. |
| `redirect` | Number | Seconds before auto-redirect from PayDollar result page. `0` skip result page, `-1` disable auto-redirect, `-2` skip only on success. |
| `oriCountry`, `destCountry` | Number(3) | Numeric country codes. |
| `print` | Text(2) `no` | Disable print on result page. |
| `failRetry` | Text(2) `no` | Disable retry after rejection. |
| `timeoutValue` | Number | OCTOPUS only, 180 to 7200 seconds. |
| `hiddenAmount` | Text(1) `T`/`F` | Show or hide amount. Approved merchants only. |
| `expDateCheck` | Number | Card must not expire within N months (1 to 192). Approved merchants only. |
| `payMode` | Text(2) | `EC` e-commerce (default), `MO` mail order, `RC` recurring. |
| `orderPhoneNo` | Text(30) | Order phone number. |
| `appUrlScheme` | Text | Universal/App Link to return to merchant app (Web-2-App) for ALIPAYHKONL, BoCPay, GrabPay, JKOPay, LINEPAY, OCTOPUS (iOS), PayMe, WECHATONL. |
| `deeplink` | Text(1) | `0` to `3`, Web-2-App mode for ALIPAYHKONL. |
| `oaflag` | Text(1) | `T` to use WeChat JSAPI (WECHATONL). |
| `orderItems` | Text | `ID^Description^Qty^SubTotal|...` |

Other optional groups (field names; see pages under `.../optional-parameters/`):

- Installment: `installment_service` (T/F), `installment_period` (months; `|`-separated only for WELEND), `installOnly` (T/F).
- Airline: `airline_service` (T/F), `airline_ticketNumber` (Text 13).
- MCP / MPS: `mpsMode` (`NIL`, `SCP`, `DCC`, `MCP`), `multiRateID` (Text 12), `mpsExRate` (Number 10,4), `foreignCurrCode` (Text 3), `foreignAmount` (Number 12,2).
- Promotion: `promotion` (T/F), `promotionCode` (Text 8), `promotionRuleCode` (Text 8), `promotionOriginalAmt` (Number 12,2).
- Shipping address: `shippingAddress1`, `shippingAddress2` (Text 240), `shippingCity`, `shippingState` (Text 100), `shippingPostalCode` (Text 50), `shippingCountryCode` (Text 2).
- Customer details: `customer_firstName`, `customer_lastName` (Text 50), `customer_email`, `customer_account_id`, `customer_phone` (Text 100), `customer_country_code` (Text 2), `customer_create_date` (Text 14, `yyyyMMddHHmmss`).
- OTA fulfilment: `ota_type`, `ota_service_start_date`, `ota_service_end_date`, `ota_change_guest_t_f`, `ota_start_country`, `ota_start_city`, `ota_start_zipcode`.
- Merchant custom data (risk): `cd_string_one`, `cd_string_two` (IATA airport code, mandatory for airlines), `cd_int_one`.
- Schedule Payment: `appId` (`SP`), `schType` (Day/Month/Year), `schStatus` (Active/Suspended), `nSch`, `sMonth`, `sDay`, `sYear`, `eMonth`, `eDay`, `eYear`.
- Protect Pay (refund protection): `customer_firstName`, `customer_lastName`, `protect_enable` (`T`/`F`/`0`), `numberOfTickets`, `eventTravelDateTime` (dd/mm/yyyy), `ticketType`, `salesTax`.

### 3DS 2.0 fields (Client Post and Direct Client Side)

Visa made 7 fields mandatory from 12 August 2024 (browser IP, screen height and width, cardholder email, name, phone; device IP for in-app). For Client Post the hosted page collects missing data; Direct Client Side must send the fields; Server Side Direct needs no change (`/web-integration/direct-client-side-connection/3ds-2-0`).

- Basic: `threeDSTransType` (`01` goods/service default, `03`, `10`, `11`, `28`), `threeDSCustomerEmail` (Text 254), `threeDSMobilePhoneCountryCode` (Text 3) + `threeDSMobilePhoneNumber` (Text 15), `threeDSHomePhoneCountryCode` + `threeDSHomePhoneNumber`, `threeDSWorkPhoneCountryCode` + `threeDSWorkPhoneNumber`, `threeDSIsFirstTimeItemOrder` (T/F), `threeDSChallengePreference` (`01` no preference default, `02` no challenge, `03` challenge requested, `04` challenge mandated, `05` to `08` no-challenge exemptions, `09` whitelist prompt). Choosing a "no challenge" option shifts chargeback liability to the merchant.
- Recurring / installment: `threeDSRecurringFrequency` (Text 4, min days between payments), `threeDSRecurringExpiry` (Text 8, `YYYYMMDD`).
- Billing: `threeDSBillingCity`, `threeDSBillingCountryCode`, `threeDSBillingLine1`, `threeDSBillingLine2`, `threeDSBillingLine3`, `threeDSBillingPostalCode`, `threeDSBillingState`.
- Shipping: `threeDSDeliveryEmail`, `threeDSDeliveryTime`, `threeDSIsAddrMatch`, `threeDSShippingCity`, `threeDSShippingCountryCode`, `threeDSShippingDetails`, `threeDSShippingLine1`, `threeDSShippingLine2`, `threeDSShippingLine3`, `threeDSShippingPostalCode`, `threeDSShippingState`.
- Gift card: `threeDSGiftCardAmount`, `threeDSGiftCardCount`, `threeDSGiftCardCurr`.
- Pre-order: `threeDSPreOrderReadyDate`, `threeDSPreOrderReason`.
- Account info: `threeDSAcctAgeInd`, `threeDSAcctCardProvisionAttempt`, `threeDSAcctCreateDate`, `threeDSAcctIsShippingAcctNameSame`, `threeDSAcctIsSuspiciousAcct`, `threeDSAcctLastChangeDate`, `threeDSAcctLastChangeInd`, `threeDSAcctNumTransDay`, `threeDSAcctNumTransYear` (Client Post only), `threeDSAcctPaymentAcctDate`, `threeDSAcctPaymentAcctInd`, `threeDSAcctPurchaseCount`, `threeDSAcctPwChangeDate`, `threeDSAcctPwChangeInd`, `threeDSAcctShippingAddrLastChangeDate`, `threeDSAcctShippingAddrLastChangeInd`.
- Account authentication: `threeDSAcctAuthMethod`, `threeDSAcctAuthTimestamp`.
- Pay token: `threeDSPayTokenInd` (`T` when card data was de-tokenised).

Value formats for the account, address and indicator fields are on the matching portal pages; the official PDF says country codes there are ISO 3166-1 numeric.

### Sample form

```html
<form method="post" action="https://test.paydollar.com/b2cDemo/eng/payment/payForm.jsp">
  <input type="hidden" name="merchantId" value="<MERCHANT_ID>">
  <input type="hidden" name="orderRef" value="ORD-10001">
  <input type="hidden" name="currCode" value="458">
  <input type="hidden" name="amount" value="150.00">
  <input type="hidden" name="payType" value="N">
  <input type="hidden" name="payMethod" value="ALL">
  <input type="hidden" name="lang" value="E">
  <input type="hidden" name="successUrl" value="https://shop.example/pay/success">
  <input type="hidden" name="failUrl" value="https://shop.example/pay/fail">
  <input type="hidden" name="cancelUrl" value="https://shop.example/pay/cancel">
  <input type="hidden" name="secureHash" value="<SERVER_COMPUTED_HASH>">
</form>
```

## Direct Client Side Connection

Test `https://test.paydollar.com/b2cDemo/eng/dPayment/payComp.jsp`, live `https://www.paydollar.com/b2c2/eng/dPayment/payComp.jsp`. Requires a 128-bit SSL certificate on the merchant site. Result comes back by redirect plus datafeed.

Required fields: `orderRef` Text(35), `amount` Number(12,2), `currCode` Text(3), `lang` Text(1), `merchantId` Number, `pMethod` Text (note: `pMethod`, not `payMethod`; values `VISA`, `Master`, `Diners`, `JCB`, `AMEX`, `PPS`, `PAYPAL`, `CHINAPAY`, `ALIPAY`, `TENPAY`, `99BILL`, `MEPS`, `OCTOPUS`, `NOVAPAY`, `ENETS`, `MYCLEAR`, `POLI`, `UPOP`, `UPOP-GNETE`, `UPOP-DNA`, `FUIOU`, `SCB`, `KRUNGSRIONLINE`, `KTB`, `UOB`, `TMB`, `IBANKING`, `BPM`, `GCash`, `BancNet`, `SMARTMONEY`, `M2U`, `CIMBCLICK`, `WECHAT`, `ONEPAY`, `VCO`, `WELEND`), `epMonth` Number(2), `epYear` Number(4), `cardNo` Text(16), `securityCode` Text(4), `cardHolder` Text(20) (card fields mandatory for card payment), `failUrl`, `successUrl`, `errorUrl` Text(300), `payType` (`N`/`H`).

Optional: `remark`, `oriCountry`, `destCountry`, `secureHash`, `timeoutValue`, `payMode`, `orderPhoneNo`; installment (`installment_service`, `installment_period`); airline; MCP (`mpsMode`, `multiRateID`, `mpsExRate`, `foreignCurrCode`, `foreignAmount`); billing (`billingFirstName`, `billingLastName` Text 60, `billingStreet1`, `billingStreet2` Text 40, `billingCity` Text 50, `billingState` Text 2, `billingPostalCode` Text 10, `billingCountry` Text 2, `billingEmail` Text 255, `custIPAddress` Text 15); third-party eWallet (`eWalletService`, `eWalletBrand` `MP`/`VCO`/`ECO`, `eWalletInd`, `vcoCallid`); card data encryption (`encryptMode=WEB`, `encryptAlgo=AES/CBC/PKCS5Padding`, `encryptedData` over `cardNo|expDate|cardHolder|securityCode`, partner only); 3DS 2.0 fields above.

## Server Side Direct Connection

Test `https://test.paydollar.com/b2cDemo/eng/directPay/payComp.jsp`, live `https://www.paydollar.com/b2c2/eng/directPay/payComp.jsp`. The merchant server posts and reads a synchronous `&`-joined result string. Needs SSL and the acquiring bank's approval. Card data handling puts the merchant in PCI scope.

Required fields: `orderRef` Text(35), `amount` Number(12,2), `currCode` Text(3), `lang` (`E`), `merchantId`, `pMethod` (cards `VISA`, `Master`, `Diners`, `JCB`, `AMEX`; `VCO`; wallets `ALIPAYAPP`, `ALIPAYCNAPP`, `ALIPAYHKAPP`, `WECHATAPP`, `WECHATONL`, `WECHATMP`, `FPS`, `OCTOPUS`, `PayMe`, `BoCPayAPP`), `epMonth`, `epYear`, `cardNo`, `cardHolder`, `securityCode` (card fields for card payments), `payType` (`N`/`H`).

Optional: `remark`, `secureHash`, `payMode` (`EC`, `MO`, `RC`, `RT` retail), `orderPhoneNo`, installment, airline, billing (as above), MCP, third-party eWallet, device wallet (`eWalletService=T`, `eWalletBrand` `APPLEPAY`/`SAMSUNG`/`GOOGLE`, `eWalletPaymentData` Base64 token), WeChat MiniProgram `oAuth` (Open ID), card data encryption (`encryptMode` `WEB` or `POS`, `encryptAlgo`, `encryptedTrack1Data`, `encryptedTrack2Data`, `encryptedEmvData`, `encryptedData`).

COF and MIT (`.../optional-parameters/cof-and-mit`): `appId` Text(5) `MIT` / `MITS` (initial / subsequent merchant-initiated), `COF` / `COFS` (card on file), `REC` / `RECS` (recurring); `appRef` Text(30) master agreement ID.

Merchant-side 3D (own MPI, `.../integration-parameters/3d-transaction`): `vbvTransaction` (T/F; non-3D needs bank approval), `vbvCHReturnCode`, `vbvPAReturnCode`, `vbvTransTime`, `vbvTransAuth` (CAVV Base64), `vbvTransECI`, `vbvCAVVAlgo`, `vbvXID`, `vbvMerchantID`, `vbvAcquirerBin`, `vbvTransStatus`; 3DS 2.0: `threeDSVersion`, `threeDSCustomerIP`, `threeDSAresTransStatus`, `threeDSDsTransId` (mandatory for 3DS 2.0), `threeDSServerTransId`, `threeDSAcsTransId`, `threeDSSdkTransId`.

Response string fields (`.../3ds-2-0/return-parameter`, eWallet `.../return-parameters`): `src`, `prc`, `Ord`, `Holder`, `successcode` (`0` succeeded, `1` failed, `2` waiting/pending for app-to-app), `Ref`, `PayRef`, `Amt`, `Cur`, `AuthId`, `TxTime`, `errMsg`, `encryptedEmvResponse` (EMV only).

```text
successcode=0&Ref=Test&PayRef=4780&Amt=1.0&Cur=344&prc=0&src=0&Ord=6697090&Holder=edward&AuthId=123456&TxTime=2003-10-07 17:48:02.0&errMsg=Transaction completed
```

For wallet direct flows `errMsg` carries wallet data: WeChat App `appId|nonceStr|package|partnerId|prepayId|timeStamp|sign|end`; WeChat QR and FPS QR a Base64 QR token; Octopus a Base64 `octopus://` URI; Alipay the SDK order string. The final result still arrives by datafeed or Query.

## Secure Hash

Source: `/transaction-security-by-secure-hash/introduction`, `/manual-secure-hashing`, `/enable-secure-hash`, `/client-library`.

- AsiaPay enables the function on request (email `service@paydollar.com`). The Secure Hash Secret is in merchant admin > Profile > Payment Information.
- Algorithm: SHA-1 (default, 40 hex chars) or SHA-256, as configured on the account.
- Once enabled, every transaction request without a valid hash is dropped and an email goes to the operation contact.
- The secret changes every 2 years. The official WooCommerce plugin splits the datafeed `secureHash` on `,` and accepts a match on any part.
- A client library (Java, PHP, ASP, ASP.NET) with `generateSecureHash` and `verifyDatafeed` is in merchant admin > Support > Developer Corner.

### Request hash (string to sign)

```text
merchantId|orderRef|currCode|amount|payType|secureHashSecret
```

Documented example (verified with `shasum`): secret `gMAVIEGVpqHvxoNEqbrZRuBDFT1B0icW`, string `56100908|1280204670187|344|10|N|gMAVIEGVpqHvxoNEqbrZRuBDFT1B0icW`, SHA-1 `13068c0ef09139ea711d36bde16785a2d30b9a30`.

### Datafeed hash (string to verify)

```text
src|prc|successcode|Ref|PayRef|Cur|Amt|payerAuth|secureHashSecret
```

Use the values exactly as posted in the datafeed (for example `Amt=80.00`).

### Pseudo-code

```js
import crypto from "node:crypto";

const algo = process.env.ASIAPAY_HASH_ALGO === "sha256" ? "sha256" : "sha1";
const secret = process.env.ASIAPAY_SECURE_HASH_SECRET;
const sha = (s) => crypto.createHash(algo).update(s, "utf8").digest("hex");

function requestHash({ merchantId, orderRef, currCode, amount, payType }) {
  // amount must be the exact string placed in the form
  return sha([merchantId, orderRef, currCode, amount, payType, secret].join("|"));
}

function datafeedValid(p /* raw POST fields as strings */) {
  const expected = sha([p.src, p.prc, p.successcode, p.Ref, p.PayRef,
                        p.Cur, p.Amt, p.payerAuth ?? "", secret].join("|"));
  return String(p.secureHash || "").split(",").some((h) =>
    h.length === expected.length &&
    crypto.timingSafeEqual(Buffer.from(h.toLowerCase()), Buffer.from(expected)));
}
```

How an empty `payerAuth` is represented for non-card methods is not documented (checked 2026-10-21); the pseudo-code uses an empty string. Log the raw body and confirm in the test environment.

## Datafeed (server callback)

Source: `/data-feed-handling`, `/data-feed-setup`, `/output-parameters/*`, `/sample-data-feed-page`, `/data-feed-validation`, `/faq/data-feed`, `/exceptional-transaction-handling/*`.

- Enable in merchant admin > Profile > Payment Options ("Return Value Link"). PayDollar POSTs form fields to that URL after each transaction.
- The page must print `OK` (plain, no HTML) as acknowledgement, first, then do its own processing. If `OK` is not read, PayDollar assumes the datafeed is lost.
- Only port 80 or 443. The URL must be reachable from the internet with a TLS chain PayDollar trusts (otherwise `SSLHandshakeException` in "Data Feed Return").
- Optional "Auto retry failed data feed": resend immediately after a failure, then once more 15 minutes later. Optional "Data Feed Failure Alert by Email".
- Check delivery in Transaction Details with columns "Data Feed Ind." (`T`/`F`) and "Data Feed Return".
- Source IPs (PDF v3.67 FAQ 13): test `203.105.61.183` to `203.105.61.218`; production `103.149.149.1` to `103.149.149.62`.
- Validate before fulfilment: `MerchantId`, `Ref` exists, `Amt`, `Cur`. Do not accept on `successcode=0` alone.
- Incomplete 3DS: status stays `Pending_3D` (`payerAuth=P`) until a scheduled job sets `Rejected` with PRC/SRC `3`/`9999` and sends a datafeed. Incomplete alternative payments stay `Pending` until a job sets `Rejected` and sends a datafeed.
- Customers who press Back and retry reuse the same `orderRef`, so one order can get a success datafeed followed by failure datafeeds. Ignore failures after a success.
- If your system fails after acknowledging, AsiaPay suggests voiding the transaction through the Merchant API.

### Datafeed fields

| Field | Type | Notes |
| --- | --- | --- |
| `src` | Text | Secondary bank response code. |
| `prc` | Text | Primary response code. |
| `Ord` | Text(40) | Bank reference. |
| `Holder` | Text | Payment account holder name. |
| `successcode` | Number | `0` succeeded, `1` failure, `2` pending, others error. |
| `Ref` | Text | Merchant `orderRef`. |
| `PayRef` | Number | PayDollar payment reference (unique across PayDollar). |
| `Amt` | Number(12,2) | Amount. |
| `Cur` | Text(3) | Currency code. |
| `remark` | Text(200) | Echo of request `remark`. |
| `AuthId` | Text | Approval code. |
| `eci` | Text(2) | ECI. Visa/JCB/Amex `05` authenticated, `06` attempted/not enrolled, `07` not 3D. Mastercard `02`, `01`, `00`. Empty when rejected by PayAlert. |
| `payerAuth` | Text(1) | 3DS 1: `Y`, `N`, `P` pending, `A` not enrolled, `U` not processed. 3DS 2: `Y`, `N`, `U`, `A`, `C`, `R`. |
| `sourceIp` | Text(15) | Payer IP. |
| `ipCountry` | Text(3) | Payer country; `*` suffix for high-risk. |
| `payMethod` | Text(20) | e.g. `VISA`, `Master`, `FPX`. |
| `cardIssuingCountry` | Text(3) | `--` if unknown. |
| `channelType` | Text(3) | `SPC` client post, `DPC` direct client, `DPS` server direct, `SCH` schedule, `DPL` payment link, `MOT` MOTO, `RTL` retail, `BPP` batch. |
| `secureHash` | Text(40) | Datafeed hash (may be comma-separated list). |
| `AlertCode` | Text(50) | e.g. `R14` IP country differs from issuing country, `R9` high-risk country. |
| `MerchantId` | Number | Merchant ID. |
| `airline_ticketNumber` | Text | Airline only. |
| `TxTime` | Text | `YYYY-MM-DD HH:MI:SS.0`. |

Extra groups: card info for approved merchants (`expMonth`, `expYear`, `panFull` AES-256 encrypted, `panFirst6`, `panFirst4`, `panLast4`, `accountHash`, `accountHashAlgo`); MPS (`mpsAmt`, `mpsCur`, `mpsForeignAmt`, `mpsForeignCur`, `mpsRate`); SchedulePay (`mSchPayId`, `dSchPayId`); MemberPay (`mpMemberId`, `mpLatestStaticToken`); third-party eWallet (`isEwallet`, `eWalletBrand`, `eWalletInd`); Promopay (`promotionCode`, `promotionRuleCode`, `promotionOriginalAmt`); installment (`period`, `installmentProvider`, `firstPayAmt`, `eachPayAmt`, `lastPayAmt`); rewards (`netAmtAfterRewards`, `rewardsRedeemTotalAmt`, `rewardsRedeemType`, `rewardsRedeemProgram` and related); 3DS (`threeDSVersion`: empty, `1.0`, `2.1.0`, `2.2.0`).

Sample:

```text
prc=0&src=0&Ord=12345678&Ref=8e88729be67a4722aa34537db4d0ae99&PayRef=11868042&successcode=0&Amt=80.00&Cur=840&Holder=testing+card&AuthId=868042&AlertCode=&remark=&eci=07&payerAuth=U&sourceIp=203.184.197.67&ipCountry=HK&payMethod=VISA&TxTime=2023-11-22+09%3A23%3A14.0
```

## Merchant API (orderApi.jsp)

Test `https://test.paydollar.com/b2cDemo/eng/merchant/api/orderApi.jsp`, live `https://www.paydollar.com/b2c2/eng/merchant/api/orderApi.jsp`. Server-side form POST. Common inputs: `merchantId` (Number), `loginId` (Text 30), `password` (Text 15), `actionType`. Non-Query actions return an `&`-joined string with `resultCode` (`0` success, `-1` failed), `orderStatus`, `ref`, `payRef`, `amt`, `cur`, `errMsg`.

| actionType | Extra inputs | Purpose and rules |
| --- | --- | --- |
| `Capture` | `payRef` (Text 35), `amount` (Number 12,2, at most the authorised amount) | Capture a `payType=H` authorisation. Sample: `resultCode=0&orderStatus=Accepted&...&errMsg=Capture Successfully.` |
| `Void` | `payRef` | Void an accepted payment before the settlement cut-off. Returns `orderStatus=Voided`. |
| `RequestRefund` | `payRef`; optional `amount` (at most original) | Request refund of an accepted payment within 14 days. Returns `orderStatus=RequestRefund`; AsiaPay processes it later. |
| `Query` | `orderRef` or `payRef` (conditional, one required) | Query status; returns XML. |
| `QueryRequestLog` | optional `orderRef`, `periodType` (`M`/`D`), `queryDate` (`MMYYYY` or `DDMMYYYY`) | Payment page request log; returns XML (`requestDate`, `sessionId`, `ref`, `cur`, `amt`, `payerIp`, `ipCountry`, `deviceBrowser`, `deviceOS`, `deviceMoblie`, `channelType`, `pageCode`). |
| `Cancel` | `payRef`; optional `forceCancel` (`T`/`F` default) | Cancel a transaction in pending status; `forceCancel=T` cancels regardless of acquirer status. Returns e.g. `orderStatus=Rejected`. |

`pageCode` values: `CPF` method selection, `CPF2` card input, `CPV` 3D verify, `CPVR` 3D result, `CPC` result, `DPC` processing, `DPR` result redirect, `CPFD` DCC selection.

### Query output (XML `<records><record>...`)

`orderStatus`, `ref`, `payRef`, `mpsMode`, `amt`, `cur`, `prc`, `src`, `ord`, `holder`, `sourceIp`, `ipCountry`, `payMethod`, `cardIssuingCountry`, `airline_ticketNumber`, `errMsg`, `txTime`, `successcode` (only for `Accepted`, `Authorized`, `Rejected`, `Pending`, `Pending_feedback`), `remark`, `authId`, `eci` (string `null` when absent), `payerAuth`, `channelType` (adds `MOB` mobile), `MerchantId`, `alertCode`; card info (`panFull` or `panMasked`, `panFirst4`, `panLast4`, `panFirst6`, `expMonth`, `expYear`, `accountHash`, `accountHashAlgo`); MPS (`mpsAmt`, `mpsCur`, `mpsForeignAmt`, `mpsForeignCur`, `mpsRate`); installment (`installment_period`, `installment_provider`, `installment_firstPayAmt`, `installment_eachPayAmt`, `installment_lastPayAmt`); SchedulePay (`mSchPayId`, `dSchPayId`); MemberPay (`mpMemberId`); eWallet (`isEWallet`, `eWalletBrand`); Promopay (`promotionCode`, `promotionRuleCode`, `promotionOriginalAmt`); rewards fields. A query can return several `<record>` elements for one `orderRef` (retries): pick the record whose `payRef` you track, or any `Accepted` one.

## Online Void / Refund (onlineApi.jsp)

Test `https://test.paydollar.com/b2cDemo/eng/merchant/api/onlineApi.jsp`, live `https://www.paydollar.com/b2c2/eng/merchant/api/onlineApi.jsp`.

Inputs: `merchantId`, `loginId`, `password`, `actionType=OnlineReversal`, `payRef` (Text 35), `amount` (up to 2 decimals; optional for void). NICEPAY virtual account only: `RefundAcctNo`, `RefundBankCd`, `RefundAcctNm`.

Return: `resultCode` (`0`/`-1`), `orderStatus`, `ref`, `payRef`, `amt` (Number 15,5, processed amount), `cur`, `errMsg`.

| Case | Criteria | New orderStatus | errMsg |
| --- | --- | --- | --- |
| Void | Accepted, not yet settled | `Voided` | Transaction was voided successfully |
| Void | payRef not found | no change | The transaction is not existed |
| Void | Not accepted before | no change | The transaction is not available for void/refund. |
| Online refund | Accepted and settled, no prior partial refund, amount equals transaction amount | `Refunded` | The transaction was refunded successfully |
| Online refund | Amount larger than transaction | no change | The refund amount is larger than transaction amount |
| Partial refund | Settled, amount less than transaction (or equal but a partial refund already happened) | `PartialRefunded` | Transaction was partially refunded successfully |
| Partial refund | Amount larger than remaining | no change | The refund amount is larger than transaction amount |
| Partial refund | Amount larger than today's sales amount | no change | The transaction is not available for void/refund. |

## Settlement Report (GenTxnXML)

Test `https://test.paydollar.com/b2cDemo/GenTxnXML`, live `https://www.paydollar.com/b2c2/GenTxnXML`.

Inputs: `merchantId`, `loginId`, `password`, `startDate` and `endDate` (Number 14, `DDMMYYYYhhmmss`), optional `queryType` (`O` by authorisation/sales time, default; `S` by settlement time, only `settleflag=T`).

XML output per record: `authdate`, `capturedate`, `batchid`, `settledate`, `payref`, `merref`, `authid`, `cur`, `amt`, `surcharge`, `merRequestAmt`, `orderstatus`, `terminal`, `bankmid`, `settleflag` (`T`/`F`), `src`, `prc`, `bank`, `bankref`, `traceno`, `accountno`, `currency`, `remark`, `errMsg`, `originalamt`.

## MemberPay (card on file tokens)

From the official MemberPay guide v2.2 (`github.com/asiapay-lib/integration_guide`):

- Membership API `.../merchant/api/MembershipApi.jsp`: `actionType` `Query`, `Add`, `Update`, `Delete` with `merchantId`, `merchantApiId`, `password`, `memberId` (Text 50) and member fields (`firstName`, `lastName`, `memberGroupId`, `status` `A`/`I`/`D`, `email`, phones, optional card `account`, `expYear`, `expMonth`, `holderName`). XML response with `responsecode`, `responsemessage`, optional `statictoken`.
- MemberPay API `.../merchant/api/MemberPayApi.jsp`: `QueryAccount`, `Add`, `Update`, `Delete`, `ChangePassword`, `GenerateToken` (inputs `memberId`, `orderRef`, `currCode`, `amount`, plus `payPassword` in Password Mode or decrypted `staticToken` in Static Token Mode, optional `accountId`; output `token`, `generatetime` GMT `YYYYMMDDHH24MISS`, `timeout` seconds), `ResetStaticToken`.
- Pay with a stored member by adding `memberPay_service=T`, `memberPay_memberId` and `memberPay_token` (one-time token) to the Client Post, Direct Client or Server Direct request. Client Post can also send `memberPay_firstName`, `memberPay_lastName`, `memberPay_email`, phone and address fields to register the member.

For merchant-initiated or recurring card charges through Server Side Direct, use the COF/MIT fields (`appId`, `appRef`). Schedule Payment (`appId=SP` with `schType` etc.) sets up AsiaPay-run schedules. Recurring for FPX or e-wallets is not documented (checked 2026-10-21).

## PaySDK REST API (Tokenization Flow)

Source: `/mobile-integration/tokenization-flow/*`. Test host `https://uat4.paydollar.com/paysdk-api`.

- Every request body is JWE: `alg=RSA-OAEP-256`, `enc=A128GCM`, random `kid` UUID, AES-128 key, `AES/GCM/NoPadding`, encrypted with the AsiaPay RSA public key. Standard request body: `data` (JWE of the raw JSON) and optional `hash` (MD5). The docs show an MD5 value but do not state exactly which bytes are hashed (checked 2026-10-21).
- `POST /auth`: body `user` (`"<masterMerchantId>-<merchantId>"`), `auth` (JWE of `{"time":..., "device":{...}, "appId":...}`), `device` (`app`, `id` required; `lang`, `tz`, `os`, `model`, `ver` optional), `time` (ms). Response `token` (bearer), `permission` (`D` direct, `H` host). Send `Authorization: Bearer <token>` afterwards.
- `POST /order/initiatetoken`: `amount`, `currCode`, `merchantId`, `orderRef`, `memberpayId`, `payGate` (`PAYDOLLAR`), `payRef`, `payMethod` (`ALL` or list), `payType` (`N`/`H`), `lang` required; `appUrlScheme`, `email`, `orderType` (`web`/`app`) optional. Response `responseCode` (`0`/`-1`), `responseMessage`, `paymentToken` (pass to the app, which calls `paySDK.process()`).
- `POST /query/txquery`: `merchantId`, `orderRef`, `payRef`. Response `responseCode` and `detail[]` with Query-style fields.
- `POST /tx/refund`: `amount`, `merchantId`, `payRef`, `environment` (`paydollar`). Response `responseCode`, `responseMessage`, `successCode`.
- Success codes: `0` completed, `1` failed, `2` processing, `3` refund completed, `4` refund failed, `99` system error. HTTP: `200`, `400` (JWT/hash validation failed), `401`, `500`, `502`, `504`.

Native PaySDK (Android `PaySDK-x.y.z.aar`, iOS `AP_PaySDK` via framework or CocoaPods) needs your bundle ID / application ID registered with AsiaPay and the SDK RSA public key in `paysdk.properties` (Android assets) or `paysdk.plist` (iOS). `EnvBase.EnvType` is `SANDBOX` or `PRODUCTION`, `PayGate` `PAYDOLLAR`, `PayChannel` `WEBVIEW` or `DIRECT`. App results are UX only; settle from datafeed or server Query.

## Status Values

Order statuses (`/appendix-a/payment-status-classification`): `Accepted` completed, `Accepted_Adj` partially refunded (remaining amount), `Authorized`, `Authorized_Adj` partially captured, `Captured`, `Capturing`, `ChargeBack`, `PartialChargeBack`, `PartialRefunded`, `Pending` processing, `Pending_feedback`, `Pending_3D`, `Refunded` (settled transaction), `Rejected` failed, `RequestPartialRefund`, `RequestRefund`, `Reversal-Auth`, `Voided` (non-settled), `Voiding`.

Treat as paid: `Accepted`, `Captured` (and `Authorized` only for hold flows awaiting capture). Treat as final failure: `Rejected`, `Voided`, `Reversal-Auth`. Everything with `Pending` is not final.

## Response Codes

Primary response code `prc` (`/appendix-a/paydollar-siampay-pesopay-payment-response-code`): `0` success, `1` rejected by payment bank, `3` payer authentication (3D) failure, `9` other reason, `-1` input parameters incorrect, `-2` server access error, `-8` PayDollar internal or fraud check, `-9` host access error, `-10` PaySDK error.

Other pairs: `0/0` success, `3/any` payer authentication fail, `-1/-1` input error, `-2/-2` server access error, `-9/-9` host access error, `-10/any` PaySDK error.

Bank codes with `prc=1`: `src` `01`, `02`, `05`, `51` bank decline; `14`, `54` input error; `41`, `43` lost/stolen card; `03`, `04`, `12`, `13`, `19`, `25`, `30`, `31`, `55`, `58`, `76`, `77`, `78`, `80`, `89`, `91`, `94`, `95`, `96`, `2000` other.

PayDollar codes with `prc=-8`: `999` other, `1000` skipped, `2000` blacklist, `2001` card blacklisted by system, `2002` card blacklisted by merchant, `2003` IP blacklisted by system, `2004` IP blacklisted by merchant, `2005` invalid cardholder name, `2006` same card more than 6 times a day, `2007` / `2008` duplicate merchant reference, `2011` other, `2012` card verification failed, `2013` card already registered, `2014` high-risk country, `2016` same payer IP over daily limit, `2017` invalid card number, `2018` multi-card attempt, `2019` issuing bank mismatch, `2020` single transaction limit, `2021` daily limit, `2022` monthly limit, `2023` invalid channel type, `2099` non-testing card, `2031` / `2032` / `2033` system rejected (TN/TA/TR), `2035` payer card country blacklisted by merchant, `2037` payer country blacklisted by merchant, `2039` special country restriction, `2041` invalid installment period, `2043` unverified member, `2045` invalid coupon, `2047` cancelled by user, `2049` e-wallet process not completed, `2050` invalid card for installment, `2051` invalid expiry date, `2053` cancelled transaction, `2054` rejected by 3D settings, `2055` card issuing and payer IP country mismatch.

## Sandbox Test Cards

From `/appendix-a/3ds-testing-cards` (sandbox only; expiry `07/2030`; to force failure use a different expiry or security code):

| Scenario | Visa | Mastercard | JCB | Amex | Others |
| --- | --- | --- | --- | --- | --- |
| 3DS 2.2 frictionless | 4333880000200019 / 123 | 5422880000200018 / 123 | 3518910000200013 / 123 | 371891000200017 / 1234 | CUP 6220230626000057 / 1234; Discover 6011000300002569 / 1234 |
| 3DS 2.2 challenge (OTP 123456) | 4333880000200027 / 123 | 5422880000200026 / 123 | 3518910000200021 / 123 | 371891000200025 / 1234 | CUP 6220230626000073 / 123; Discover 6011000300002585 / 123 |
| No 3DS | 4918914107195005 / 123 | 5258950000000020 / 123 | 3528010000010021 / 123 | 378416000100026 / 1234 | |
| 3DS reject (transStatus R) | 4333880000200001 | 5422880000200034 | 3518910000200005 | 371891000200009 | |
| 3DS failed (transStatus N) | 4333880000200092 | 5422880000200091 | 3518910000200039 | 371891000200033 | |

Test FPX banks, DuitNow or e-wallet simulators: Not documented in official docs (checked 2026-10-21).

## Rate Limits

Not documented in official docs (checked 2026-10-21). Fraud rules can reject repeated cards, IPs and references (`prc=-8` codes above).

## Common Errors and Gotchas

- "Your payment service is not active": merchant ID and URL environment mismatch.
- "CurrCode is incorrect": currency not enabled for this merchant ID; one currency per merchant ID.
- Request dropped silently with email to operations: missing or wrong `secureHash` after Secure Hash was enabled. Check algorithm, field order, amount string and `payType`.
- Datafeed shows "F" in "Data Feed Ind.": URL wrong, not on 80/443, TLS chain untrusted, or `OK` not printed.
- `pMethod` (direct connections) vs `payMethod` (client post): different field names.
- `successUrl` is reachable by anyone and only carries `Ref`.
- Hold (`payType=H`) not available for several alternative methods and must be captured within 14 days.
- `RequestRefund` is asynchronous (`orderStatus=RequestRefund`); `OnlineReversal` returns the final status.
- The Merchant API `password` is limited to 15 characters (Text 15) on orderApi.
