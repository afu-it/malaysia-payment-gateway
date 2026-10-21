# RinggitPay API Reference

Use this reference for factual RinggitPay integration guidance. RinggitPay does not publish a developer portal; the facts below come from the official WordPress.org plugin source and two RinggitPay-authored PDFs (see `source-map.md` for the trust caveat). Confirm the latest spec version with RinggitPay before go-live.

Source tags used below:

- [SPEC] RinggitPay "Specification for RinggitPay Integration" v1.13 (01-07-2023), `RinggitPay_Integration_and_Testing_V1.13.pdf`.
- [DD] RinggitPay "Direct Debit (e-Mandate) Integration Guide" API Specification v1.8 (06-09-2023), `RP_DD_API_Integration_Guide_v1.8.pdf`.
- [WP] Official plugin "RinggitPay for WooCommerce" 1.0.4, `https://wordpress.org/plugins/ringgitpay/`, file `class-wc-ringgitpay-gateway.php`.
- [LIVE] HTTP probe on 2026-10-21 (`Allow: POST` header on the endpoints).

## Endpoint Catalogue

Hosted payment page API [SPEC] [WP]:

| Operation | Method | UAT | Production | Purpose |
|---|---|---|---|---|
| Payment request | POST (HTML form, NVP) | `https://ringgitpay.co/payment` | `https://ringgitpay.com/payment` | Send buyer to RinggitPay payment page |
| Transaction enquiry | POST (JSON) | `https://ringgitpay.co/transactionenquiry` | `https://ringgitpay.com/transactionenquiry` | Get current status of an order |
| Direct response | POST to merchant APIURL | merchant URL | merchant URL | Host-to-host result (webhook) |
| Indirect response | browser form POST to merchant RedirectURL or ReturnURL | merchant URL | merchant URL | Browser return with result |

Both RinggitPay endpoints return 405 to GET and advertise `Allow: POST` [LIVE].

Direct Debit (e-Mandate) REST API [DD], base `https://bills2u-api.ringgitpay.co` (UAT) or `https://bills2u-api.ringgitpay.biz` (production):

| Function | Method | Path |
|---|---|---|
| Get token | POST | `https://identity.ringgitpay.co/connect/token` (UAT), `https://identity.ringgitpay.biz/connect/token` (production) |
| DDA create | POST | `/api/v1/mandate/dda` |
| DDA view by PRN | GET | `/api/v1/mandate/dda/getbyprn/{paymentRefNo}` |
| DDA view by id | GET | `/api/v1/mandate/dda/{mandateSubscriptionId}` |
| DDA update | PUT | `/api/v1/mandate/dda` |
| DDA manage (options) | POST | `/api/v1/mandate/dda/manage` |
| DDA debit settings create | POST | `/api/v1/mandate/dda/debitsettings` |
| DDA debit settings view | GET | `/api/v1/mandate/dda/debitsettings/{mandateSubscriptionId}` |
| DDA debit settings delete | DELETE | `/api/v1/mandate/dda/debitsettings/{ddaDebitSettingId}` |
| DDI create | POST | `/api/v1/mandate/ddi/create` |
| DDI view | GET | `/api/v1/mandate/ddi/{sellerOrderNo}` |
| DDI view by DDA PRN | GET | `/api/v1/mandate/ddi/getbyprn/{paymentRefNo}` |
| DDI approve | PUT | `/api/v1/mandate/ddi/{sellerOrderNo}` |
| DDI cancel | DELETE | `/api/v1/mandate/ddi/{sellerOrderNo}` |

Not documented in any official source found (checked 2026-10-21): refund, void, capture, cancel of a hosted payment, payout/disbursement, settlement report, tokenised card recurring, webhook retry schedule, webhook IP allowlist, rate limits.

## Environments And Credentials

- UAT payment host `ringgitpay.co`; production payment host `ringgitpay.com` [SPEC] [WP]. Note the marketing site is `ringgitpay.biz` and `ringgitpay.com` (GET on root) redirects there; the `/payment` path is still the production endpoint.
- Credentials issued after merchant and application registration [SPEC]:
  - Merchant username and password: portal login (view profile, applications, transaction history).
  - `AppId`: unique id for the merchant's application.
  - `RequestKey`: alphanumeric shared secret for request checksums.
  - `ResponseKey`: alphanumeric shared secret for response checksums.
- Keys are in RinggitPay Portal > Merchant Profile (scroll to "RequestKey"/"ResponseKey") or sent by the RinggitPay App Team. If compromised, contact RinggitPay to reset [SPEC]. Portal login link on the website: `https://biller.ringgitpay.biz/`.
- The merchant must give RinggitPay two URLs per application [SPEC]:
  - `APIURL`: server-to-server back-end webhook, "more robust and crucial".
  - `RedirectURL`: browser redirection, "not 100% reliable".
  - Neither may contain an IP address or port number [SPEC go-live checklist].
- `ReturnURL` (optional, per request) takes precedence over RedirectURL; intended for mobile apps/web [SPEC].

## Payment Flow

1. Merchant POSTs an HTML form to `/payment` with the fields below [SPEC].
2. Buyer pays on the RinggitPay page. If the merchant has FPX and cards, FPX loads by default and "Change Payment Method" switches. With one method subscribed it opens directly [SPEC].
3. Production: FPX redirects to the chosen bank; cards go to 3-D Secure [SPEC].
4. RinggitPay sends the DIRECT message to APIURL after it receives the debit status, and the INDIRECT message by browser form POST to RedirectURL/ReturnURL. There is no RinggitPay result screen [SPEC].
5. Incomplete payments stay `PENDING` until FPX/card timeout; deferred updates are sent to APIURL [SPEC].

## Payment Request Fields

POST `application/x-www-form-urlencoded` form, auto-submitted from the browser [SPEC] [WP].

| # | Field | Type | Max | Required | Notes |
|---|---|---|---|---|---|
| 1 | `appId` | C | 20 | Yes | Application ID |
| 2 | `currency` | C | 4 | Yes | Default `MYR`; only MYR supported [WP FAQ] |
| 3 | `amount` | N | 16,2 | Yes | Ringgit with 2 decimals, e.g. `100.00` |
| 4 | `orderId` | C | 20 | Yes | Merchant order id, unique |
| 5 | `checkSum` | C | - | Yes | See checksum section |
| 6 | `buyerEmail` | C | 50 | No | Buyer email |
| 7 | `returnURL` | C | 150 | No | Dynamic redirect, URL-encoded, must stay on your application domain |
| 8 | `accNumber` | N | 20 | If payer verification enabled | Payer bank account number |
| 9 | `accBuyerId` | N | 20 | If payer verification enabled | Payer ID, e.g. IC number |

Sample [SPEC]:

```html
<form method="post" action="https://ringgitpay.co/payment">
  <input type="hidden" name="appId" value="MERCHANTTEST01">
  <input type="hidden" name="currency" value="MYR">
  <input type="hidden" name="amount" value="100.00">
  <input type="hidden" name="orderId" value="ORDERTXN-110">
  <input type="hidden" name="returnURL" value="http%3A%2F%2Fwww.merchantsite.com%2Freceipt">
  <input type="hidden" name="checkSum" value="<SHA256_HEX>">
</form>
```

The official plugin sends only `appId`, `currency`, `amount`, `orderId`, `checkSum` [WP].

## Checksums

All checksums: join values with `|`, append the key last, SHA-256, hex digest [SPEC] [WP].

| Message | String to sign | Key |
|---|---|---|
| Payment request | `appId\|currency\|amount\|orderId\|REQUESTKEY` | RequestKey |
| Payment response (APIURL, RedirectURL) | `rp_appId\|rp_currency\|rp_amount\|rp_statusCode\|rp_orderId\|rp_transactionRef\|RESPONSEKEY` | ResponseKey |
| Enquiry request | `appId\|orderId\|transactionRef\|REQUESTKEY` (no ref: `appId\|orderId\|\|REQUESTKEY`) | RequestKey |
| Enquiry response | same as payment response | ResponseKey |

Rules and gotchas:

- Values in the order of the field table; `buyerEmail`, `returnURL`, `accNumber`, `accBuyerId` are not in the request checksum [SPEC formula].
- `rp_xtraInfo` is explicitly excluded from the response checksum; `rp_paymentMode`, `rp_txnTime`, `rp_remarks`, `rp_paymentCountry` are also not in the formula [SPEC].
- Case: the spec says the request checksum "must be in upper case". The official plugin sends PHP `hash('sha256', ...)` lower-case hex and verifies with `strtolower($rp_checkSum) === hash(...)` [WP]. Send upper case per spec, compare case-insensitively.
- The `amount` in the string must be the exact string posted in the form. Sign `rp_amount` exactly as received.
- The spec sample string shows a stray space (`100.00| ORDERTXN-110`) and the sample response checksum contains non-hex letters; treat both as typos, not as rules.

Pseudo-code:

```text
req_sig  = upper(sha256_hex(app_id + "|" + "MYR" + "|" + amount_2dp + "|" + order_id + "|" + REQUEST_KEY))

expected = sha256_hex(p.rp_appId + "|" + p.rp_currency + "|" + p.rp_amount + "|" +
                      p.rp_statusCode + "|" + p.rp_orderId + "|" + p.rp_transactionRef + "|" + RESPONSE_KEY)
valid    = constant_time_equal(lower(expected), lower(p.rp_checkSum))
```

## Payment Response Fields

Sent as NVP form fields to APIURL (server POST) and RedirectURL/ReturnURL (browser form POST). The plugin reads them from `$_REQUEST` and uses one handler for both [SPEC] [WP].

| # | Field | Type | Max | Notes |
|---|---|---|---|---|
| 1 | `rp_appId` | C | 20 | Application ID |
| 2 | `rp_currency` | C | 4 | `MYR` |
| 3 | `rp_amount` | N | 16,2 | Debit amount |
| 4 | `rp_statusCode` | N | 4 | Prefix `RP` or `IR` (note `RP100` is 5 chars) |
| 5 | `rp_orderId` | C | 20 | Merchant order id |
| 6 | `rp_transactionRef` | N | 15 | RinggitPay transaction reference |
| 7 | `rp_txnTime` | C | 14 | `YYYYMMDDHH24mmSS` in UTC |
| 8 | `rp_paymentMode` | N | 15 | `FPX`, `CC`, `DC`, `NA` |
| 9 | `rp_checkSum` | C | - | Response checksum |
| 10 | `rp_remarks` | C | - | Remarks from channel/gateway, e.g. `Approved` |
| 11 | `rp_xtraInfo` | O | 6 | FPX payer verification result only; not in checksum |
| 12 | `rp_paymentCountry` | C | 30 | FPX `MALAYSIA`; card country from card number on success |

`rp_xtraInfo` (6 digits) [SPEC]:

- Digits 1-2 account type: `00` undetermined, `01` CASA, `02` LCA.
- Digits 3-4 account number match: `00` undetermined, `01` true, `02` false.
- Digits 5-6 buyer ID match: `00` undetermined, `01` true, `02` false.

The sample spells it `rp_xtrainfo` and `rp_xtraInfo` in different places; accept both.

Direct message security: merchants may ask RinggitPay to add static custom headers (API Key, Secret Id, Client Id, Client Secret) to the APIURL call [SPEC v1.13]. Use them as an extra check, not instead of the checksum.

Response handling: the plugin sends `HTTP/1.1 200 OK` first, then processes [WP]. Retry behaviour on non-200 is not documented.

## Transaction Enquiry

`POST https://ringgitpay.{co|com}/transactionenquiry`, JSON body; the response is JSON in the same HTTP exchange with the payment response fields [SPEC] [LIVE].

Request:

| # | Field | Type | Max | Notes |
|---|---|---|---|---|
| 1 | `appId` | C | 20 | Application ID |
| 2 | `orderId` | C | 20 | Merchant order id |
| 3 | `transactionRef` | C | 20 | Optional RinggitPay reference |
| 4 | `checkSum` | C | - | `appId\|orderId\|transactionRef\|REQUESTKEY` |

```json
{"appId":"MERCHANTTEST02","orderId":"ORDERTXN217","transactionRef":"180101121031053","checkSum":"<SHA256_HEX>"}
```

Response sample [SPEC]:

```json
{"rp_appId":"MERCHANTTEST02","rp_currency":"MYR","rp_amount":"541.50","rp_statusCode":"RP00",
 "rp_orderId":"ORDERTXN217","rp_transactionRef":"180101121031053","rp_paymentMode":"FPX",
 "rp_txnTime":"20180101121045","rp_checkSum":"<SHA256_HEX>","rp_remarks":"Approved",
 "rp_xtraInfo":"020102","rp_paymentCountry":"MALAYSIA"}
```

Verify `rp_checkSum` with ResponseKey as for payment responses. Request `Content-Type` is not stated; use `application/json`.

## Status Codes

[SPEC] ("Status codes are regularly updated"; ask RinggitPay for the latest list.)

| Code | Meaning | Handling |
|---|---|---|
| `RP00` | Transaction approved or success | Settle |
| `RP01` | Transaction exception | Failed / investigate |
| `RP02` | Merchant app verification pending | Config issue |
| `RP09` | Transaction pending | Not final; enquire later |
| `RP45` | Duplicate seller order number | Use a new orderId |
| `RP50` | Rejected by Visa/Mastercard | Failed |
| `RP80` | Insufficient fund | Failed |
| `RP91` | Transaction failed | Failed |
| `RP92` | Transaction failed at channel | Failed |
| `RP94` | Amount lower than minimum | Failed |
| `RP95` | Amount higher than maximum | Failed |
| `RP96` | Cancelled by customer | Failed |
| `RP97` | Session timed out | Failed |
| `RP100` | User transaction limit exceeded | Failed |
| `IR10` | Invalid order number | Request error |
| `IR11` | Invalid currency | Request error |
| `IR12` | Invalid amount | Request error |
| `IR13` | Invalid checksum | Request error |
| `IR14` | Invalid buyer email | Request error |
| `IR15` | Invalid ReturnURL | Request error |
| `IR18` | Invalid transaction reference | Request error |
| `IR20` | Merchant not active | Account issue |

Plugin mapping [WP]: `RP00` -> paid (Completed or Processing, configurable), `RP09` -> on hold, anything else -> failed. Store the code and `rp_remarks`; the merchant must show relevant remarks to the payer [SPEC].

## UAT Test Data

FPX [SPEC]:

- Limits: retail (B2C) RM1.00 to RM30,000.00; corporate (B2B) RM2.00 to RM1,000,000.00.
- Retail test banks: `SBI Bank A` = approve/success, `SBI Bank B` = fail. Names may vary; ask RinggitPay.
- FPX simulator login: username `1234`, password `1234` (UAT simulator only).
- B2C: response goes to ReturnURL/RedirectURL and the same to APIURL.
- B2B: all transactions stay `RP09` until an authoriser acts. In UAT, email the RinggitPay transaction reference with `APPROVE` or `DECLINE`; a DIRECT response then goes to APIURL.

Cards (UAT only; cards without 3-D Secure may fail) [SPEC]:

| Brand | Number | 3DS | Type | Country |
|---|---|---|---|---|
| Mastercard | `5123450000000008` | Y | Credit | Malaysia |
| Mastercard | `5111111111111118` | N | NA | United States |
| Mastercard | `2223000000000007` | Y | Credit | Japan |
| Visa | `4508750015741019` | Y | Debit | Malaysia |
| Visa | `4012000033330026` | N | NA | NA |

Expiry simulates result: May-31 approved, May-39 declined, April-27 expired card, August-28 timed out. CVV: `100` match, `101` not processed, `102` no match. Success example: `5123450000000008`, May-31, CVV 100. Leave the simulator fields as they are and click Submit.

Email notifications are sent by the FPX or card platform to the buyer email entered on the payment screen [SPEC].

## Official Plugin Behaviour (WooCommerce) [WP]

- Settings: `appId`, `requestkey`, `responsekey`, `uatorproduction` (`uat`/`production`), `defaultpaymentstatus` (`completed` default, or `processing`).
- Order set to on-hold before redirect; cart emptied.
- Response handler is WooCommerce API hook `rpresponse` (URL `https://<site>/?wc-api=rpresponse`). The plugin sends no callback URL in the request, so this URL has to be registered with RinggitPay as APIURL/RedirectURL (inferred; the plugin readme does not say so).
- Known weaknesses to avoid in custom code: no check of `rp_amount`/`rp_currency`/`rp_appId` against the order; a bad checksum marks the order failed; stock is reduced on every `RP00` hit, so duplicate notifications are not idempotent.

## Direct Debit (e-Mandate) API [DD]

Concepts: DDA (Direct Debit Authorization) = payer mandate with enrolment, maintenance, termination. DDI (Direct Debit Instruction) = an individual debit, only after the DDA is enrolled with the bank.

Auth: OAuth2 token, `POST /connect/token`, `application/x-www-form-urlencoded`, fields `client_id`, `client_secret`, `username`, `password`, `grant_type=password`, `scope=bills2u_api` (from the guide's screenshot). Response has `access_token`, `expires_in`, `token_type: Bearer`, `scope`. Send `Authorization: Bearer <access_token>`. Credentials (authorization link, username, password, client id, client secret, scope) come from RinggitPay. Handle refresh tokens that stop working (revoked access, expiry, password change).

Version: only `v1`. Other versions return 400 `UnsupportedApiVersion`; no version returns 404.

Responses are wrapped: `{"result": {...}, "statusCode": 200}`. HTTP codes: 200 success, 400 bad request, 401 unauthorized, 403 forbidden, 404 not found, 417 exception handling, 500 internal server error.

### DDA create `POST /api/v1/mandate/dda`

| Field | Req | Type | Max | Notes |
|---|---|---|---|---|
| `appId` | M | C | 30 | AppId from RinggitPay |
| `ddMerchantId` | M | C | 10 | Bank direct debit id, `DDxxxxxxxx` (table spells `ddmerchantId`) |
| `frequencyMode` | M | C | 2 | `MT` monthly, `WK` weekly, `DL` daily, `YR` yearly |
| `maxNoOfDebiting` | M | N | | Max DDIs: monthly 1-31, weekly 1-7, daily 1, yearly 365 |
| `maximumDebitAmount` | M | N | 5,2 | Max per DDI |
| `purposeOfPayment` | M | C | 27 | Description |
| `email` | M | C | 50 | Enrolment link sent here |
| `telephoneNo` | M | N | 11 | Phone notifications |
| `expiryDate` | M | | | `yyyy-MM-dd` |
| `paymentRefNo` | M | C | 20 | Unique plan reference (PRN) |
| `idType` | M | N | 1 | 1 new IC, 2 old IC, 3 passport, 4 business registration, 5 others |
| `idNumber` | M | C,N | 18 | Payer ID number |
| `accountHolderName` | M | C | 50 | Payer name |
| `ref1` | M | C | 50 | Extra reference |
| `ref2` | M | C | 50 | Extra reference |

The sample request also sends `"applicationType": "01"` (not in the table). Response `result`: `mandateSubscriptionId`, `planStatus` (`PENDING`), `errorMessage`, `enrolmentLink` (table calls it `elink`), e.g. `https://payer.ringgitpay.co/direct-debit/enroll/6634`.

### DDA view (by PRN or by id)

Returns a collection if several records share a PRN. Fields: `mandateSubscriptionId`, `appId`, `ddmerchantId`, `paymentRefNo`, `idType`, `idNumber`, `email`, `telephoneNo`, `frequencyMode`, `maxNoOfDebiting`, `maximumDebitAmount`, `purposeOfPayment`, `expiryDate`, `accountHolderName`, `ref1`, `ref2`, `planStatus` (`PENDING`/`ACTIVE`/`INACTIVE`), `remarks`, `elink`, `tlink` (termination link), `allowTermination` Y/N, `autoDebit` Y/N, `autoRetrigger` Y/N, `offlinedda` Y/N, `effectiveDate` (`yyyy-MM-dd`), `payerAccountName` (40), `payerAccountNumber` (20) (both when enrolment succeeded), `activityHistory[]`.

`activityHistory[]`: `orderId`, `transactionRef` (30), `transactionTime`, `applicationType` (Enrolment/Maintenance/Termination), `paymentStatus` (`PENDING`/`INPROGRESS`/`SUCCESS`/`FAILED`), `enrpStatus` (`PENDING`/`SUCCESS`/`FAILED`), `remarks`.

### DDA update `PUT /api/v1/mandate/dda`

Only while plan status and payment status are `PENDING`. Fields: `mandateSubscriptionId`, `email`, `telephoneNo`, `paymentRefNo`, `idType`, `idNumber`, `accountHolderName`, `ref1`, `ref2` (all M); sample also sends `appId`, `ddMerchantId`. Response as DDA view.

### DDA manage `POST /api/v1/mandate/dda/manage`

`mandateSubscriptionId` (M), `appId` (M), `paymentRefNo` (M, 20), `allowTermination` Y/N, `terminationReason` (50, required if allowTermination on), `autoDebit` Y/N, `autoRetrigger` Y/N (retrigger failed DDI), `isActive` Y/N. Sample also sends `ddMerchantId`. Returns 200 with the same fields; re-read the DDA to confirm.

### DDA debit settings

- Create `POST /api/v1/mandate/dda/debitsettings`: `mandateSubscriptionId` (M), `appId` (M), `paymentRefNo` (M), `debitamount` (M, 5,2, at most the DDA max), `dayofweek` (Monday..Sunday; required for weekly), `dayofmonth` (1-31; required for monthly and yearly), `month` (1-12; required for yearly). Daily needs none. Number of settings at most `maxNoOfDebiting`. Response 200 with `debitSettingId`.
- View `GET /api/v1/mandate/dda/debitsettings/{mandateSubscriptionId}`: create fields plus `ddaDebitSettingId`.
- Delete `DELETE /api/v1/mandate/dda/debitsettings/{ddaDebitSettingId}`: 200.

### DDI

- Create `POST /api/v1/mandate/ddi/create`: `appId`, `paymentRefNo`, `debitAmount` (5,2), `debitDate` (`yyyy-MM-dd`, today or future). Response `result`: `ddiId`, `appId`, `paymentRefNo`, `sellerOrderNo` (12), `debitAmount`, `debitDate`, `status` (`SCHEDULED`), `remarks`.
- View `GET /api/v1/mandate/ddi/{sellerOrderNo}`: same fields; `status` one of `PENDING`, `SCHEDULED`, `CANCELLED`, `INITIATED`, `SUCCESS`, `FAILED`.
- View by PRN `GET /api/v1/mandate/ddi/getbyprn/{paymentRefNo}`: collection.
- Approve `PUT /api/v1/mandate/ddi/{sellerOrderNo}`: `PENDING` -> `SCHEDULED`; a past debit date is moved to today.
- Cancel `DELETE /api/v1/mandate/ddi/{sellerOrderNo}`: only from `PENDING`/`SCHEDULED`; returns the DDI with `status: CANCELLED`.

DD result notifications to the merchant (webhook) are not described in the DD guide; poll DDA/DDI views or ask RinggitPay.

## Common Errors

- `IR13` invalid checksum: wrong key, wrong field order, amount string differs from posted value, extra spaces.
- `IR15` invalid ReturnURL: not URL-encoded, or outside your application domain.
- `RP45`: reused `orderId`.
- `RP02` or `IR20`: application not verified or merchant inactive; contact RinggitPay.
- No APIURL callback: APIURL not registered with RinggitPay for this AppId, or it uses an IP/port.
- DD API 404: missing `/v1/` in path; 400 `UnsupportedApiVersion`: wrong version.
