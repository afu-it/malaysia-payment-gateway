# 2C2P QuickPay (Payment Links) And Payout Reference

QuickPay creates hosted payment links (v2.4, Base64 JSON, HMAC-SHA1). Payout disburses funds to bank accounts or wallets (v1.2, JWT HS256). Entry point: `2c2p-api.md`. Checked against the official docs on 2026-10-21.

## QuickPay Endpoints

| API | Sandbox | Live |
| --- | --- | --- |
| DirectAPI (Generate, Query, Update, Delete) | `https://demo2.2c2p.com/QuickPay/DirectAPI` | `https://t.2c2p.com/QuickPay/DirectAPI` |
| DeliveryAPI (Generate and Send, Send) | `https://demo2.2c2p.com/QuickPay/DeliveryAPI` | `https://t.2c2p.com/QuickPay/DeliveryAPI` |

Indonesia live uses `https://pgwcore.dp.alipay.com/QuickPay/...`.

Wire format: build JSON wrapped in the message name (for example `{"GenerateQPReq": {...}}`), Base64-encode the whole JSON, `POST` it with `Content-Type: text/plain`. The response is Base64 JSON (for example `{"GenerateQPRes": {...}}`); decode it, then verify its `hashValue`.

| Operation | Request object | Response object | Endpoint |
| --- | --- | --- | --- |
| Generate link | `GenerateQPReq` | `GenerateQPRes` | DirectAPI |
| Query link | `QPQueryReq` | `QPQueryRes` | DirectAPI |
| Update link | `QPUpdateReq` | `QPUpdateRes` | DirectAPI |
| Delete link | `QPDeleteReq` | `QPDeleteRes` | DirectAPI |
| Generate and send | `GenerateSendQPReq` | `GenerateSendQPRes` | DeliveryAPI |
| Send existing link | `QPSendReq` | `QPSendRes` | DeliveryAPI |

Source: https://developer.2c2p.com/docs/quickpay-how-it-works, https://developer.2c2p.com/docs/api-quickpay-generate-link, https://developer.2c2p.com/docs/api-quickpay-generate-and-send-link

## QuickPay Hash (`hashValue`)

`hashValue` = HMAC-SHA1 over the plain concatenation (no separators) of the listed field values, keyed with the merchant Secret Key, hex encoded. Docs samples show both lowercase (requests) and uppercase (responses) hex, so compare case-insensitively. Empty optional fields contribute an empty string. Use the exact order:

| Message | String to sign |
| --- | --- |
| Generate request | `version + timeStamp + merchantID + orderIdPrefix + description + currency + amount + allowMultiplePayment + maxTransaction + expiry + userData1 + userData2 + userData3 + userData4 + userData5 + promotion + categoryId + resultUrl1 + resultUrl2 + paymentOption + ippInterestType + paymentExpiry + request3DS + enableStoreCard + recurring + recurringAmount + allowAccumulate + maxAccumulateAmount + recurringInterval + recurringCount + chargeNextDate + chargeOnDate + useStoreCardOnly + storeCardUniqueID + ippPeriodFilter` |
| Generate response | `version + timeStamp + qpID + orderIdPrefix + currency + amount + expiry + url + resCode + resDesc` |
| Query request | `version + timeStamp + merchantID + qpID` |
| Query response | `version + timeStamp + qpID + orderIdPrefix + description + currency + amount + allowMultiplePayment + maxTransaction + expiry + userData1 + userData2 + userData3 + userData4 + userData5 + promotion + categoryId + resultUrl1 + resultUrl2 + paymentOption + ippInterestType + paymentExpiry + url + currentApproved + clickCount + request3DS + enableStoreCard + recurring + recurringAmount + allowAccumulate + maxAccumulateAmount + recurringInterval + recurringCount + chargeNextDate + chargeOnDate + useStoreCardOnly + storeCardUniqueID + IPPPeriodFilter + resCode + resDesc` |
| Update request | `version + timeStamp + merchantID + qpID + description + currency + amount + allowMultiplePayment + maxTransaction + expiry + userData1..5 + promotion + categoryId + resultUrl1 + resultUrl2 + paymentOption + ippInterestType + paymentExpiry + request3DS + enableStoreCard + recurring + recurringAmount + allowAccumulate + maxAccumulateAmount + recurringInterval + recurringCount + chargeNextDate + chargeOnDate + useStoreCardOnly + storeCardUniqueID + ippPeriodFilter` |
| Update response, Delete response | `version + timeStamp + qpID + resCode + resDesc` |
| Delete request | `version + timeStamp + merchantID + qpID` |
| Generate and Send request | Generate order up to `paymentExpiry`, then `toEmails + ccEmails + bccEmails + emailSubject + emailMessage + toMobiles + smsMessage`, then `request3DS + enableStoreCard + recurring + recurringAmount + allowAccumulate + maxAccumulateAmount + recurringInterval + recurringCount + chargeNextDate + chargeOnDate + useStoreCardOnly + storeCardUniqueID + ippPeriodFilter + receiverName + customerName + customerPhone + customerEmail + sendNotification + recipientEmail + itemize + items + locale` (the docs print a stray `+ +` before `receiverName`; how `items` is serialised into the string is not documented in official docs, checked 2026-10-21) |
| Generate and Send response | `version + timestamp + qpID + orderIdPrefix + currency + amount + expiry + url + successEmails + successMobiles + failedEmails + failedMobiles + resCode + resDesc` |
| Send request | `version + timeStamp + merchantID + qpID + toEmails + ccEmails + bccEmails + emailSubject + emailMessage + toMobiles + smsMessage` |
| Send response | `version + timeStamp + qpID + successEmails + successMobiles + failedEmails + failedMobiles + resCode + resDesc` |

```js
const crypto = require("crypto");
const GEN_ORDER = ["version","timeStamp","merchantID","orderIdPrefix","description","currency","amount",
  "allowMultiplePayment","maxTransaction","expiry","userData1","userData2","userData3","userData4","userData5",
  "promotion","categoryId","resultUrl1","resultUrl2","paymentOption","ippInterestType","paymentExpiry","request3DS",
  "enableStoreCard","recurring","recurringAmount","allowAccumulate","maxAccumulateAmount","recurringInterval",
  "recurringCount","chargeNextDate","chargeOnDate","useStoreCardOnly","storeCardUniqueID","ippPeriodFilter"];
const qpHash = (obj, order, key) =>
  crypto.createHmac("sha1", key).update(order.map(k => obj[k] ?? "").join("")).digest("hex");
// req.hashValue = qpHash(req, GEN_ORDER, process.env.TWOC2P_SECRET_KEY);
// body = Buffer.from(JSON.stringify({ GenerateQPReq: req })).toString("base64");
```

Source: https://developer.2c2p.com/docs/api-quickpay-generate-link-request-parameter, https://developer.2c2p.com/docs/api-quickpay-generate-link-response-parameter, https://developer.2c2p.com/docs/api-quickpay-query-request-parameter, https://developer.2c2p.com/docs/api-quickpay-query-response-parameter, https://developer.2c2p.com/docs/api-quickpay-update-request-parameter, https://developer.2c2p.com/docs/api-quickpay-delete-request-parameter, https://developer.2c2p.com/docs/api-quickpay-generate-and-send-link-request-parameter, https://developer.2c2p.com/docs/api-quickpay-send-link-request-parameter

## QuickPay Generate Request (`GenerateQPReq`)

| Field | Type | M/O | Notes |
| --- | --- | --- | --- |
| `version` | AN 3 | M | `2.4`. |
| `timeStamp` | AN 14 | M | `yyyyMMddHHmmss`. |
| `merchantID` | AN 15 | M | |
| `orderIdPrefix` | AN 50 | M | Alphanumeric plus `_` and `-`. Max 45 if multiple payment, 40 if multiple payment and recurring. |
| `description` | AN 150 | M | Alphanumeric plus `.`, `_`, `#`, `-`. |
| `currency` | AN 3 | M | ISO 4217 alpha, `MYR`. |
| `amount` | D 13 | M | Major units, `100.00`. |
| `allowMultiplePayment` | C 1 | O | `Y` / `N` (default). |
| `maxTransaction` | N 4 | O | 1 to 1000 approved payments. |
| `expiry` | AN 19 | M | `yyyy-MM-dd HH:mm:ss` (a changelog entry says `yyyy-MM-dd HH:mm`; the parameter table and sample use seconds). |
| `userData1` to `userData5` | C 150 | O | |
| `promotion` | AN 20 | O | Promotion code. |
| `categoryId` | AN 20 | O | Reporting category. |
| `resultUrl1` | AN 255 | O | Frontend return URL. |
| `resultUrl2` | AN 255 | O | Backend return URL. |
| `paymentOption` | C 1 | O | `F` full amount only, `I` instalment only, `C` card only, `A` all. |
| `ippInterestType` | C 1 | O | `M`, `C`, `A`. |
| `paymentExpiry` | AN 19 | O | 123 (online banking / counter) payment expiry; default from merchant config. |
| `request3DS` | C 1 | O | `Y`, `N`, `F`. |
| `enableStoreCard` | C 1 | O | `Y` / `N` tokenisation. |
| `recurring` | C 1 | O | `Y` / `N` RPP. |
| `recurringAmount`, `allowAccumulate`, `maxAccumulateAmount`, `recurringInterval` (1 to 365), `recurringCount`, `chargeNextDate` (`ddMMyyyy`), `chargeOnDate` (`ddMM`) | | C | RPP fields, same rules as Payment Token. |
| `ippPeriodFilter` | C 200 | C | Allowed tenors. |
| `useStoreCardOnly` | C 1 | O | Pay with given stored card only. |
| `storeCardUniqueID` | C 20 | C | Stored card ID. |
| `locale` | | O | Page language. |
| `hashValue` | AN 150 | M | See hash table. |

Response (`GenerateQPRes`): `version`, `timeStamp`, `qpID` (N 10), `orderIdPrefix`, `currency`, `amount`, `expiry`, `url` (the payment link), `resCode` (`000` success), `resDesc`, `hashValue`.

Query (`QPQueryReq`: `version`, `timeStamp`, `merchantID`, `qpID`, `hashValue`) returns the link settings plus `url`, `currentApproved`, `clickCount`. Update (`QPUpdateReq`) sends `qpID` plus the editable Generate fields. Delete (`QPDeleteReq`: `version`, `timeStamp`, `merchantID`, `qpID`, `hashValue`).

Generate and Send / Send add delivery fields: `toEmails`, `ccEmails`, `bccEmails`, `emailSubject` and `emailMessage` (mandatory with `toEmails`), `toMobiles`, `smsMessage` (mandatory with `toMobiles`, 60 characters), plus `receiverName`, `customerName`, `customerPhone`, `customerEmail`, `sendNotification`, `recipientEmail`, `itemize`, `items[]` (`code`, `name`, `price`, `quantity`). One channel (email or SMS) per call. Responses add `successEmails`, `successMobiles`, `failedEmails`, `failedMobiles`.

Payment result for a QuickPay link: the format 2C2P posts to `resultUrl2` is not described on the QuickPay pages (checked 2026-10-21). Confirm the order status with Payment Inquiry or the maintenance inquiry before fulfilling, and match on the order ID built from `orderIdPrefix`.

QuickPay response codes: `000` success; `001` invalid MID; `002` invalid QuickPay ID; `003` invalid amount; `004` invalid currency; `005` invalid description; `006` invalid expiry date; `007` invalid payment option; `008` invalid IPP interest type; `009` invalid payment expiry date; `010` invalid allowMultiplePayment; `011` invalid max transactions; `012` invalid order ID prefix; `013` order ID in use; `014` max transaction must exceed pending plus approved; `015` invalid request version; `016` invalid request3DS; `017` invalid enableStoreCard; `018` invalid recurring flag; `019` invalid recurring amount; `020` invalid allowAccumulate; `021` invalid accumulate amount; `022` invalid recurring interval; `023` invalid recurring count; `024` invalid charge next date; `025` invalid charge on date; `026` provide charge on date or recurring interval; `027` provide only one of them; `101` provide one or more emails / mobiles; `102` invalid email; `103` invalid BCC email; `104` invalid mobile; `105` invalid CC email; `106` invalid email subject; `107` invalid email message; `108` invalid SMS message; `997` invalid request message; `998` empty compulsory value; `999` system down.

Source: https://developer.2c2p.com/docs/api-quickpay-generate-link-request-parameter, https://developer.2c2p.com/docs/api-quickpay-query, https://developer.2c2p.com/docs/api-quickpay-update, https://developer.2c2p.com/docs/api-quickpay-delete, https://developer.2c2p.com/docs/api-quickpay-send-link, https://developer.2c2p.com/docs/response-code-quickpay

## Payout Endpoints (JWT HS256)

All `POST {"payload":"<JWT>"}` with `Content-Type: application/json`, JWT HS256 with the Secret Key, response `{"payload":"<JWT>"}`. Payout must be enabled on the account (code `8115` otherwise).

| Operation | Sandbox | Production |
| --- | --- | --- |
| Create payout | `https://sandbox-pgw.2c2p.com/payouts/api/v1.2/payout/create` | `https://pgw.2c2p.com/payouts/api/v1.2/payout/create` |
| Payout inquiry | `https://sandbox-pgw.2c2p.com/payouts/api/v1.2/payout/inquiry` | `https://pgw.2c2p.com/payouts/api/v1.2/payout/inquiry` |
| Beneficiary registration | `https://sandbox-pgw.2c2p.com/payouts/api/v1.1/beneficiary/register` | `https://pgw.2c2p.com/payouts/api/v1.1/beneficiary/register` |
| Beneficiary inquiry | `https://sandbox-pgw.2c2p.com/payouts/api/v1.1/payout/beneficiary/inquiry` | `https://pgw.2c2p.com/payouts/api/v1.1/payout/beneficiary/inquiry` |
| Preferred provider list | `https://sandbox-pgw.2c2p.com/payouts/api/v1.1/merchants/providers` | `https://pgw.2c2p.com/payouts/api/v1.1/merchants/providers` |
| Source of fund list | `https://sandbox-pgw.2c2p.com/payouts/api/v1.1/merchants/source-of-fund` | `https://pgw.2c2p.com/payouts/api/v1.1/merchants/source-of-fund` |

The environment guide still lists v1.1 create / inquiry paths; the v1.2 changelog adds the v1.2 paths. Indonesia production uses `pgw-id.2c2p.com`.

Flow: merchant sends payout, 2C2P validates and forwards to the bank, an immediate response comes back (`8101` accepted), the bank disburses, then 2C2P sends a payout notification (`8100` disbursed, `8102` rejected).

Source: https://developer.2c2p.com/docs/payout-how-it-works, https://developer.2c2p.com/docs/create-payout, https://developer.2c2p.com/docs/payout-inquiry, https://developer.2c2p.com/docs/beneficiary-registration, https://developer.2c2p.com/docs/beneficiary-inquiry, https://developer.2c2p.com/docs/payout-preferred-provider, https://developer.2c2p.com/docs/payout-source-of-fund, https://developer.2c2p.com/changelog/add-new-payout-api-12

## Create Payout Request

| Field | Type | M/O | Notes |
| --- | --- | --- | --- |
| `merchantID` | AN 25 | M | |
| `beneficiaryName` | A 150 | M | |
| `amount` | D 16,2 | M | 2-decimal format. |
| `beneficiaryBankCode` | AN 20 | M | Payout bank code (Malaysian list below). |
| `requestID` | AN 40 | M | UUID. Idempotency key; duplicates rejected (`8105`). |
| `UTR` | AN 22 | O | Reference sent to the bank; generated as `{PayoutDate:YYMMDD}{Sequence:N16}` if blank. |
| `payoutDate` | C 10 | O | `dd/MM/yyyy`; default today + 2 (T+2). |
| `userDefined1` to `userDefined5` | C 150 | O | |
| `preferredSofProfile` | AN 10 | O | Source of fund code. |
| `beneficiaryId` | AN 50 | C | Registered beneficiary. |
| `beneficiaryType` | C 50 | C | `DEFAULT`, `BANKACC`, `MOBILE`, `IDN`, `CIDN`, `PPN`, `APN`. |
| `beneficiaryTypeValue` | C 50 | C | Account number, mobile or ID matching the type. |
| `preferredProvider` | C 50 | C | Wallet provider when a mobile number maps to several wallets (Malaysia and Thailand only). |
| `notificationUrl` | C 255 | O | Payout notification callback. |

## Payout Response, Inquiry Response And Notification

Fields: `version` (`1.2`), `merchantID`, `requestID`, `UTR`, `payoutDate` (`yyyy-MM-dd`; the notification sample uses ISO date-time), `amount`, `currency`, `beneficiaryName`, `beneficiaryBankName`, `beneficiaryBankCode`, `bankReferenceNo`, `userDefined1..5`, `respCode`, `respDesc`, `remarks` (rejection reason), `preferredSofProfile`, `beneficiaryId`, `beneficiaryType`, `beneficiaryTypeValue`, `preferredProvider`.

Inquiry request: `merchantID` (M) plus `requestID` or `UTR`.

Notification: HTTP POST to `notificationUrl`, plain-text JSON body (not JWT), headers `x-signature` (described as an "hmac256 signature", 64 uppercase hex characters in the sample) and `x-notify-event: payout`. The signing input and key for `x-signature` are not documented in official docs (checked 2026-10-21). Do not trust the notification body alone: call Payout Inquiry (JWT verified) with the `requestID` before booking the result.

Beneficiary registration request: `merchantID` M, `requestID` M, `idCard` C, `accountName` M, `bankCode` C, `emailAddress` O, `mobileNo` C, `companyId` O, `beneficiaryType` C (adds `CBANKACC` company bank account), `beneficiaryTypeValue` C, `notificationUrl` O, `preferredProvider` C, `dateOfBirth` (`DD/MM/YYYY`) and `nationality` (A2) for individuals, `dateOfIncorporation` and `countryOfIncorporation` for `CBANKACC` / `CIDN`. Result codes `8180` to `8183`.

## Payout Response Codes

`8100` disbursement success; `8101` request accepted; `8102` rejected by bank; `8103` invalid merchant; `8104` invalid request ID; `8105` duplicate request ID; `8106` invalid UTR; `8107` invalid payout date; `8108` invalid amount; `8109` invalid beneficiary name; `8110` invalid beneficiary account no; `8111` invalid beneficiary bank code; `8112` invalid beneficiary mobile; `8113` invalid user defined property; `8114` cannot find payout provider; `8115` payout not enabled; `8116` insufficient balance; `8117` invalid request; `8118` invalid ID card; `8119` invalid SoF; `8120` invalid beneficiary ID; `8121` invalid registered beneficiary status; `8122` invalid company ID; `8123` invalid regency code; `8124` invalid beneficiary email; `8125` invalid notification URL; `8126` rejected (refunded); `8127` duplicate UTR; `8128` invalid passport number; `8129` invalid army / police number; `8130` duplicate beneficiary entry; `8131` invalid preferred provider; `8132` invalid account status; `8133` invalid account type; `8134` invalid beneficiary type; `8135` invalid use of preferred provider; `8136` invalid currency code; `8180` beneficiary registration success; `8181` registration accepted; `8182` duplicate registration; `8183` registration rejected; `8197` payout malfunction; `8198` payout timeout; `8199` internal server error; `8200` interface error; `8201` bank timeout; `8401` customer payout not found; `8402` payout not found; `8403` merchant not found; `8404` to `8412` batch file errors (content, read, empty, missing fields, file name, size, duplicate transaction ID, bank code, extension); `8500` CDD pass; `8501` CDD fail; `8502` KYC manual process pending.

Source: https://developer.2c2p.com/docs/payout-request-parameter, https://developer.2c2p.com/docs/payout-response-notification, https://developer.2c2p.com/docs/payout-inquiry-request-parameter, https://developer.2c2p.com/docs/payout-inquiry-response-parameter, https://developer.2c2p.com/docs/beneficiary-registration-request-parameter, https://developer.2c2p.com/docs/payout-response-code

## Malaysian Payout Bank Codes (`beneficiaryBankCode`)

Source: https://developer.2c2p.com/docs/payout-bank-code (Malaysia section).

| Bank Code | Bank Name |
| --- | --- |
| MFBBMYKL | Alliance Bank Malaysia Berhad |
| ALSRMYKL | Alliance Islamic Bank |
| MBAMMYKL | Alliance Investment Bank Berhad - Conventional |
| MBAMMY21 | Alliance Investment Bank Berhad SPI |
| ISCAMYK1 | ABRAR DISCOUNTS BHD |
| ISCA9999 | ABRAR DISCOUNTS BHD SPI |
| PHBMMYKL | AFFIN BANK BHD |
| PAMBMYK1 | AFFIN INVESTMENT BANK BHD |
| AIBBMYKL | AFFIN ISLAMIC BANK BHD |
| AGOBMYKL | AGROBANK |
| AGOBMY21 | AGROBANK SPI |
| RJHIMYKL | AL-RAJHI BANK MALAYSIA BHD |
| ARBKMYKL | AMBANK BHD |
| AMMBMYKL | AMINVESTMENT BANK BHD |
| AMMBMY21 | AMINVESTMENT BANK BHD SPI |
| AISLMYKL | AMISLAMIC BANK BHD |
| AFBQMYKL | ASIAN FINANCE BANK BHD |
| BKKBMYKL | BANGKOK BANK BHD |
| BIMBMYKL | BANK ISLAM BHD |
| BKRMMYK1 | BANK KERJASAMA RAKYAT |
| BKRMMYKL | BANK KERJASAMA RAKYAT - NEW |
| BMMBMYKL | BANK MUAMALAT MALAYSIA BHD |
| BNMAMYKL | BANK NEGARA MALAYSIA |
| BOFAMY2X | BANK OF AMERICA BHD |
| BKCHMYKL | BANK OF CHINA MALAYSIA BHD |
| BOTKMYKX | BANK OF TOKYO-MITSUBISHI UFJ |
| BOTKMY21 | BANK OF TOKYO-MITSUBISHI UFJ SPI |
| PEMBMYK1 | BANK PEMBANGUNAN |
| PEMBMY21 | BANK PEMBANGUNAN SPI |
| BSNAMYK1 | BANK SIMPANAN NASIONAL |
| BSNAMY21 | BANK SIMPANAN NASIONAL SPI |
| BNPA9999 | BNP PARIBAS MALAYSIA BHD |
| BNPAMYKL | BNP PARIBAS MALAYSIA BHD |
| BNPAMY21 | BNP PARIBAS MALAYSIA BHD SPI |
| CAGAMYKL | CAGAMAS BHD |
| CAGAMY21 | CAGAMAS BHD SPI |
| CIBBMYKL | CIMB BANK BHD |
| COIMMYKL | CIMB INVESTMENT BANK BHD |
| COIMMY21 | CIMB INVESTMENT BANK BHD SPI |
| CTBBMYKL | CIMB ISLAMIC BANK BHD |
| CITIMYKL | CITIBANK BHD |
| CITIMYM1 | CITIBANK BHD SPI |
| DEUTMYKL | DEUTSCHE BANK MALAYSIA BHD |
| DEUTMY21 | DEUTSCHE BANK MALAYSIA BHD SPI |
| AVSSMYK1 | ECM LIBRA INVESTMENT BANK BHD |
| EXMBMY21 | EXPORT IMPORT BANK BHD (EXIM) |
| EXMB9999 | EXPORT IMPORT BANK BHD SPI |
| HLBBMYKL | HONG LEONG BANK BHD |
| HLIVMYKL | HONG LEONG INVESTMENT BANK BHD |
| HLIBMYKL | HONG LEONG ISLAMIC BANK BHD |
| HBMBMYKL | HSBC Bank |
| HBMB9999 | HSBC AMANAH |
| HMABMYKL | HSBC AMANAH MALAYSIA BHD |
| HDSBMY2P | AFFIN HWANGDBS INVESTMENT BANK BHD |
| IMBMYKL | INDIA INTERNATIONAL BANK MALAYSIA BHD |
| ICBKMYKL | INDUSTRIAL AND COMM BANK OF CHINA MALAYSIA |
| CHASMYKX | JP MORGAN CHASE BANK BHD |
| KAFBMYKL | KAF INVESTMENT BANK BHD |
| KAFBMY21 | KAF INVESTMENT BANK BHD SPI |
| KKENMYK1 | KENANGA INVESTMENT BANK BHD |
| KKENMY21 | KENANGA INVESTMENT BANK BHD  SPI |
| KWAPMYK1 | KUMPULAN WANG  PERSARAAN DIPERBADANKAN (KWAP) |
| KWSPMYK1 | KUMPULAN WANG SIMPANAN PEKERJA (KWSP) |
| KFHOMYKL | KUWAIT FINANCE HOUSE |
| MBBEMYKL | MALAYAN BANKING BHD |
| MEPSMYK1 | MEPS |
| MBEAMYKL | MAYBANK INVESTMENT BANK BHD |
| MBEAMY21 | MAYBANK INVESTMENT BANK BHD SPI |
| MBISMYKL | MAYBANK ISLAMIC BHD |
| MIDFMYK1 | MIDF AMANAH INVESTMENT |
| MIDFMY21 | MIDF AMANAH INVESTMENT SPI |
| MIMBMYKL | MIMB INVESTMENT BANK BHD |
| MHCBMYKA | MIZUHO BANK (M) BHD |
| NBADMYKL | NATIONAL BANK OF ABU DHABI |
| OABBMYKL | OCBC AL-AMIN BANK BHD |
| OCBCMYKL | OCBC BANK MALAYSIA BHD |
| OSKIMYKL | OSK INVESTMENT BANK BHD |
| OSKI9999 | OSK INVESTMENT BANK BHD SPI |
| PBBEMYKL | PUBLIC BANK BHD |
| SMBBMYK1 | PUBLIC INVESTMENT BANK BHD |
| PIBEMYK1 | PUBLIC ISLAMIC BANK BHD |
| RHBBMYKL | RHB BANK |
| RHBMMYK1 | RHB INVESTMENT BANK |
| RHBAMYKL | RHB ISLAMIC BANK |
| MSMEMYK1 | SME BANK |
| MSMEMY21 | SME BANK SPI |
| SCBLMYKX | STANDARD CHARTERED BANK BHD |
| SCSRMYKK | STANDARD CHARTERED SAADIQ BHD |
| SMBCMYKL | SUMITOMO MITSUI BANK BHD |
| NOSCMYKL | THE BANK OF NOVA SCOTIA BHD |
| ABNAMYKL | THE ROYAL BANK OF SCOTLAND BHD |
| UOVBMYKL | UNITED OVERSEAS BANK MALAYSIA BHD |
| UOVBMY21 | UNITED OVERSEAS BANK MALAYSIA BHD SPI |
| PCBCMYKL | CHINE CONSTRUCTION BANK (MALAYSIA) BERHAD |
| ABNAMY2A | THE ROYAL BANK OF SCOTLAND PLC LABUAN BRANCH |
| BISLMYKA | BANK ISLAM MALAYSIA BERHAD LABUAN OFFSHORE BRANCH |
| BNPAMYKA | BNP PARIBAS |
| BOTKMYKA | BANK OF TOKYO-MITSUBISHI UFJ, LTD., THE |
| CHASMY2A | JPMORGAN CHASE BANK, N.A., LABUAN BRANCH |
| CHASMYL1 | J.P. MORGAN CHASE BANK BERHAD |
| CIBBMY2L | CIMB BANK BERHAD, LABUAN OFFSHORE BRANCH |
| CIBBMYKA | CIMB BANK (L) LIMITED |
| HSBCMYKA | HONGKONG AND SHANGHAI BANKING CORPORATION LTD.,THE |
| MBBEMY2L | MAYBANK INTERNATIONAL LABUAN BRANCH |
| MBBEMYKA | MAYBANK INTERNATIONAL (L) LTD. |
| MHCBMY2L | MIZUHO BANK, LTD., LABUAN BRANCH |
| NOSCMY2L | BANK OF NOVA SCOTIA, THE |
| OCBCMYKA | OVERSEA-CHINESE BANKING CORP. LTD, LABUAN BRANCH |
| PBLLMYKA | PUBLIC BANK (L) LTD |
| RHBBMYKA | RHB BANK (L) LTD |
| SMBCMYKA | SUMITOMO MITSUI BANKING CORPORATION |
| UOVBMY2L | UNITED OVERSEAS BANK LIMITED LABUAN BRANCH |
| PCBCMY22 | CHINA CONSTRUCTION BANK CORPORATION LABUAN BRANCH |
