# Axaipay API Reference

Use this reference for factual Axaipay API guidance. Everything here comes from two official PDFs attached to Axaipay's Freshdesk knowledge base, plus the official plugins Axaipay publishes there. Checked 2026-10-21.

## Source URLs

- eComPay API Technical Documentation v1.18 (article): `https://support.axaipay.com/support/solutions/articles/73000523456-axaipay-ecompay-api-technical-document-v1-18`
- eComPay PDF (37 pages, effective 13 Feb 2025): `https://support.axaipay.com/helpdesk/attachments/73176544580`
- AutoDebit API Documentation v1.6 (article): `https://support.axaipay.com/support/solutions/articles/73000611021-axaipay-autodebit-api-documentation-v1-6`
- AutoDebit PDF (32 pages, effective 23 Nov 2023): `https://support.axaipay.com/helpdesk/attachments/73081657622`
- WooCommerce plugin v2.0.0 (official, Oct 2025): `https://support.axaipay.com/support/solutions/articles/73000530285-axaipay-plugin-for-woocommerce`
- Refund process: `https://support.axaipay.com/support/solutions/articles/73000612883-how-do-i-process-a-refund-through-axaipay-`

Section numbers below (for example "eComPay 1.4") point to the PDF sections.

## Integration Surfaces

- **Hosted payment page** (eComPay 1): browser HTML form POST; the customer picks a method on Axaipay's page.
- **Bypass payment page** (eComPay 1.3, 1.4.3): same form plus `paymentMethod` and `acquirerCode` to jump straight to an FPX bank or e-wallet.
- **QR code transaction** (eComPay 2): server-to-server; Axaipay returns a DuitNow QR or Alipay+ payload you render as a QR on your page.
- **Inquiry and void** (eComPay 3): server-to-server REST.
- **AutoDebit / DirectDebit eMandate** (AutoDebit 1-3): enrolment by form POST (RM1.00 FPX consent payment), recurring debits reported to your backend URL, plus an inquiry API.
- **No-code**: AxaiCart payment links, AutoDebit subscription links (`https://autodebit.axaipay.my/<productCode>`), and plugins for WooCommerce, OpenCart, PrestaShop, VirtueMart.

## Environments

| Purpose | Staging | Production |
|---|---|---|
| Hosted payment page | `https://staging.axaipay.my/gateway/v1/payment` | `https://secured.axaipay.my/gateway/v1/payment` |
| QR generate | `https://api-staging.axaipay.my/trxn/qrcode/generate` | `https://api.axaipay.my/trxn/qrcode/generate` |
| Inquiry | `https://api-staging.axaipay.my/trxn/inquiry` | `https://api.axaipay.my/trxn/inquiry` |
| Void | `https://api-staging.axaipay.my/trxn/void` | `https://api.axaipay.my/trxn/void` |
| AutoDebit enrolment | `https://staging.axaipay.my/gateway/emandate/v1/enrolment` | `https://secured.axaipay.my/gateway/emandate/v1/enrolment` |
| AutoDebit inquiry | `https://api-staging.axaipay.my/emandate/v1/inquiry` | `https://api.axaipay.my/emandate/v1/inquiry` |
| Signature checker | `https://staging.axaipay.my:8888/generate-signature` | n/a |

The PDF HTML samples post to `.../gateway/v1/payment/` (trailing slash); the official WooCommerce plugin posts without it. Both forms appear in official material.

You need a demo (staging) account before integrating; request it from `support@axaipay.com` (eComPay "Know before you code").

## Endpoint Catalogue

| # | Method | URL path | Auth | Purpose |
|---|---|---|---|---|
| 1 | POST (browser form) | `/gateway/v1/payment` on `staging.axaipay.my` / `secured.axaipay.my` | `signature` field | Start a hosted or bypass payment |
| 2 | POST (Axaipay to you) | your `redirectUrl` | `signature` field | Browser return with result |
| 3 | POST (Axaipay to you) | your `backendUrl` | `signature` field | Server callback with result |
| 4 | POST | `/trxn/qrcode/generate` on `api(-staging).axaipay.my` | `x-api-key`, `x-signature` | Create DuitNow QR / Alipay+ QR transaction |
| 5 | POST | `/trxn/inquiry` | `x-api-key`, `x-signature` | Get transaction by `trxnId` or `mchtTrxnId` |
| 6 | POST | `/trxn/void` | `x-api-key`, `x-signature` | Void a transaction by `trxnId` |
| 7 | POST (browser form) | `/gateway/emandate/v1/enrolment` | `signature` field | AutoDebit enrolment, with or without product code |
| 8 | POST (Axaipay to you) | your AutoDebit redirect / backend URL | `signature` field | Enrolment result and each later recurring debit |
| 9 | POST | `/emandate/v1/inquiry` | `x-api-key`, `x-signature` | AutoDebit transaction inquiry |

Not offered as an API in official docs (checked 2026-10-21): refund, capture, tokenised card charge, payout/disbursement, settlement report, merchant-initiated recurring charge, mandate cancel. Refunds go through a form (see Refunds).

## Signature Algorithm (all operations)

Every signature in Axaipay is:

```text
signature = Base64( HMAC-SHA512( key = <SIGNING_KEY>, message = v1 + v2 + ... + vn ) )
```

- Values only, concatenated with no separator, in the order listed for each operation (alphabetical by field name).
- Use UTF-8 bytes of the exact string values. Keep spaces inside values (e.g. `OCBC Bank`, `B2C (Retail Banking)`).
- Optional fields (`backendUrl`, `redirectUrl`) are included only when you send them. The official plugin skips any empty field.
- Request signatures go in the form field `signature` (browser flows) or the header `x-signature` (REST calls).
- Response signatures arrive as `signature` in the POST body or in `data.signature` in JSON.

Every example below was recomputed on 2026-10-21 and matches the digest printed in the PDF, except where flagged.

Pseudo-code:

```text
function sign(values[], signingKey):
    msg = join(values, "")
    return base64(hmac_sha512(signingKey, utf8(msg)))

function verify(post, fieldOrder, signingKey):
    expected = sign([post[f] ?? "" for f in fieldOrder], signingKey)
    return constant_time_equals(expected, post["signature"])
```

PHP (from the PDF):

```php
$digest = hash_hmac('sha512', $string_to_sign, $signing_key, true);
$signature = base64_encode($digest);
```

## 1. Hosted Payment Page (eComPay 1)

Flow: merchant collects order data, browser POSTs the form to Axaipay, customer pays, Axaipay POSTs the result to the Redirect URL (browser) and the Backend URL (server).

At registration the merchant gives Axaipay two default URLs: **Redirect URL** (customer is redirected here with POST) and **Backend URL** (Axaipay POSTs a callback here with the transaction status).

### Request fields (HTML form POST)

| Field | Type | M/O | Sample | Notes |
|---|---|---|---|---|
| `mchtId` | String(255) | M | `iboxfan2021` | Merchant ID from Axaipay |
| `mchtTrxnId` | String(255) | M | `TrxnIbox123` | Your unique transaction ID |
| `signature` | String(255) | M | `C/sGjsyf...` | See request signature |
| `txnAmount` | Decimal(12,2) | M | `4.25` | Ringgit, 2 decimals |
| `customerEmail` | String(255) | M | `abc@gmail.com` | |
| `customerName` | String(255) | M | `Arizka` | |
| `customerPhone` | Numeric(20) | M | `60123456789` | No country code means Malaysian number assumed |
| `orderDescription` | String(255) | M | `Orange Juice` | |
| `redirectUrl` | String(1000) | O | `https://example.com/redirect` | Must start `http://` or `https://`; overrides default |
| `backendUrl` | String(1000) | O | `https://example.com/backend` | Must start `http://` or `https://`; overrides default |

Bypass payment page (both optional, NOT included in the signature):

| Field | Type | Sample | Notes |
|---|---|---|---|
| `paymentMethod` | String(2) | `OB` | `OB` = Online Banking, `EW` = EWallet |
| `acquirerCode` | String(20) | `B2C_TEST` | FPX bank code or e-wallet code from the Codes section. Staging: `B2C_TEST` or `B2B_TEST` |

The official WooCommerce plugin formats `txnAmount` with `number_format(total, 2, '.', '')`, normalises phone to start with `60`, and builds `mchtTrxnId` as `WC<orderId>_<unixTime>`.

### Request signature (eComPay 1.4)

Order: `backendUrl` (if sent), `customerEmail`, `customerName`, `customerPhone`, `mchtId`, `mchtTrxnId`, `orderDescription`, `redirectUrl` (if sent), `txnAmount`.

Example (staging, signing key `dwdefE12324!9293`, no optional URLs):

```text
abc@gmail.comArizka60123456789iboxfan2021TrxnIbox123Orange Juice4.25
=> C/sGjsyfCs4XcqhVmxPGZxgN9CPujmmTFeiT6Ob+/GHvNM07wy+A9A+3PzfsJWhnzaFhYIwBrTWOJMkvXkH4Bw==
```

The PDF shows the header line as `...MchtIdMchtTrxnId...` with capitals; the real field names are `mchtId` and `mchtTrxnId`.

### Sample form

```html
<form method="POST" action="https://staging.axaipay.my/gateway/v1/payment/">
  <input type="hidden" name="customerEmail" value="abc@gmail.com"/>
  <input type="hidden" name="customerName" value="Arizka"/>
  <input type="hidden" name="customerPhone" value="60123456789"/>
  <input type="hidden" name="mchtId" value="iboxfan2021"/>
  <input type="hidden" name="mchtTrxnId" value="TrxnIbox123"/>
  <input type="hidden" name="orderDescription" value="Orange Juice"/>
  <input type="hidden" name="signature" value="C/sGjsyf...H4Bw=="/>
  <input type="hidden" name="txnAmount" value="4.25"/>
  <!-- bypass page only: -->
  <input type="hidden" name="paymentMethod" value="OB"/>
  <input type="hidden" name="acquirerCode" value="B2C_TEST"/>
</form>
```

### Response / callback fields (eComPay 1.5)

Posted to both the Redirect URL and the Backend URL.

| Field | Type | Sample | Notes |
|---|---|---|---|
| `mchtId` | String | `iboxfan2021` | |
| `mchtTrxnId` | String | `TrxnIbox123` | |
| `trxnId` | String | `20220106045513` | Axaipay transaction ID |
| `trxnAmount` | Decimal | `4.25` | MYR |
| `trxnTime` | String | `20220106045513` | `yyyyMMddHHmmss` (inquiry docs say Malaysia time) |
| `trxnStatus` | Integer | `11` | See status enum |
| `trxnAcquirer` | Integer | `21` | See acquirer enum |
| `trxnPaymentMethod` | Integer | `20` | See payment method enum |
| `trxnFpxType` | String | `LCA` | FPX only: `CASA` bank transfer, `LCA` credit/debit card |
| `trxnBankName` | String | `OCBC Bank` | FPX only |
| `trxnFpxMethod` | String | `B2C (Retail Banking)` | FPX only: `B2C (Retail Banking)` or `B2B (Corporate Banking)` |
| `trxnBuyerName` | String | `Arizka` | FPX only, payer name; not signed |
| `signature` | String | | See response signature |

The official WooCommerce plugin notes Axaipay sometimes also sends `mchtTxnId` (no "r") on the redirect; it drops that duplicate before verifying.

### Response signature (eComPay 1.6)

Order: `mchtId`, `mchtTrxnId`, `trxnAcquirer`, `trxnAmount`, `trxnBankName`, `trxnFpxMethod`, `trxnFpxType`, `trxnId`, `trxnPaymentMethod`, `trxnStatus`, `trxnTime`. `trxnBuyerName` is not included.

Example (key `dwdefE12324!9293`):

```text
iboxfan2021TrxnIbox123214.25OCBC BankB2C (Retail Banking)LCA20220106045513201120220106045513
=> Z0kcRCNj/AyqLU1YVsBNfZp7nPbSsRc46g7bET6mz4jSzMA8gCoACO4VUuq++oCW5BNVgQu649vPfHZh8Xrrog==
```

The PDF prints this string with the spaces removed (`OCBCBankB2C(RetailBanking)`), but the printed digest only matches with spaces kept. Keep values exactly as posted.

For non-FPX payments the FPX fields are documented as FPX-only; the docs do not say whether they arrive empty or absent. Treat a missing field as an empty string (the official plugin does this).

### Backend callback handling

- Content: form POST fields (the plugin reads `$_POST`).
- Ack: not documented. The official WooCommerce plugin returns HTTP 200 body `OK` on success, 400 on missing fields, 403 on bad signature.
- Retry policy, timeout and source IPs: not documented in official docs (checked 2026-10-21).
- Status 55 (cancel): the official plugin's redirect handler skips signature verification for status 55, which suggests cancel posts may not carry a valid signature. Do not let an unverified post change order state; confirm with Inquiry.

## 2. QR Code Transaction (eComPay 2)

Server-to-server HTTPS POST. Axaipay returns a QR payload you display.

Headers:

| Header | M/O | Notes |
|---|---|---|
| `Content-Type: application/x-www-form-urlencoded` | M | Body is form-encoded |
| `x-api-key` | M | Merchant API key |
| `x-signature` | M | Request signature |

Body:

| Field | Type | M/O | Sample | Notes |
|---|---|---|---|---|
| `mchtId` | String(255) | M | `iboxfan2021` | |
| `mchtTrxnId` | String(255) | M | `OrderA#1` | |
| `acquirer` | Integer | M | `50` | `16` = Alipay+, `50` = DuitNowQR |
| `txnAmount` | Decimal(12,2) | M | `100.00` | |
| `customerEmail` | String(255) | M | | |
| `customerName` | String(255) | M | | |
| `customerPhone` | Numeric(20) | M | `60123456789` | |
| `orderDescription` | String(255) | M | | |
| `backendUrl` | String(1000) | O | | Status updates POST here; default backend URL if omitted |

Request signature order: `acquirer`, `backendUrl` (if sent), `customerEmail`, `customerName`, `customerPhone`, `mchtId`, `mchtTrxnId`, `orderDescription`, `txnAmount`.

```text
50https://example.com/backendabc@gmail.comJohn Doe60123456789sushiOrderA#1Orange Juice100.00
key wHqA4861VRpjtB0gPH3HW
=> Si8YUPa1Ews8kkmcLI4nmM7iriGzo1oFY4DMp8yPN+ffjN3lo9lJTUJ0oZHqgDABlDGyEZjR0lYRl3QDz0D8qA==
```

Response (HTTP 200, JSON):

```json
{
  "success": true,
  "errorCode": null,
  "message": "QR Code generation is successful",
  "data": {
    "mchtId": "sushi",
    "mchtTrxnId": "OrderA#1",
    "qrCodePayload": "000201010212...6304AAC3",
    "trxnAcquirer": 50,
    "trxnAmount": 100.00,
    "trxnId": "202406041540998",
    "signature": "l07pLq9O..."
  }
}
```

| Field | Type | Notes |
|---|---|---|
| `mchtId`, `mchtTrxnId` | String | Echo |
| `qrCodePayload` | String | EMV QR string to render for the customer |
| `trxnAcquirer` | Integer | Acquirer enum |
| `trxnAmount` | Decimal | MYR |
| `trxnId` | String | Axaipay transaction ID |
| `signature` | String | Response signature |

Response signature order: `mchtId`, `mchtTrxnId`, `qrCodePayload`, `trxnAcquirer`, `trxnAmount`, `trxnId`. Verified: the PDF digest `l07pLq9ORTng0xKF...ImcLFw==` matches with `trxnAmount` rendered `100.00`.

Gotcha: JSON decoding turns `100.00` into a number; re-serialising it can give `100`. Read the raw JSON text, or format with 2 decimals, before signing. Final payment status arrives on the backend URL with the section 1.5 payload, or poll Inquiry.

## 3. Inquiry and Void (eComPay 3)

Headers as in QR (`Content-Type: application/x-www-form-urlencoded`, `x-api-key`, `x-signature`). Method POST.

Body by Axaipay ID (inquiry and void):

| Field | Type | M/O | Sample |
|---|---|---|---|
| `merchantId` | String | M | `sushi` |
| `trxnId` | String | M | `20230314123232562` |

Body by merchant ID (inquiry only; void accepts `trxnId` only):

| Field | Type | M/O | Sample |
|---|---|---|---|
| `mchtTrxnId` | String | M | `40393145` |
| `merchantId` | String | M | `sushi` |

Note the field name is `merchantId` here, not `mchtId`.

Request signatures:

- By `trxnId`: `merchantId + trxnId`. Example `sushi20230314123232562`, key `wHqA4861VRpjtB0gPH3HW` => `URlnYfMillC9cDVbDPdblb7B2/76Ru53CnPwUE0T9nDQm5Zk33gAyrF3PGaXm9JSD0E0CKKDiWMeTZ7zbgr8Ow==` (verified).
- By `mchtTrxnId`: the PDF text and the official WooCommerce plugin both use `mchtTrxnId + merchantId` (`40393145sushi`). **Doc conflict:** the PDF's printed digest `UeSkrOoM...Eim1A==` does not match that string with either sample key; it matches `sushi40393145` (merchantId first) with key `wHqA4861VRpjtB0gPH3HW`. Implement `mchtTrxnId + merchantId` (the documented rule and plugin code), and if staging returns `PY016` confirm the order with Axaipay support.

Inquiry response (HTTP 200, JSON):

```json
{
  "success": true,
  "errorCode": null,
  "message": "Inquiry successful",
  "data": {
    "trxnId": "20230314123232562",
    "mchtId": "sushi",
    "mchtTrxnId": "40393145",
    "trxnStatus": 11,
    "trxnTime": "20230314123233",
    "trxnAmount": 110.00,
    "trxnBankName": "SBI Bank A",
    "trxnPaymentMethod": 20,
    "trxnAcquirer": 21,
    "trxnFpxMethod": "B2C (Retail Banking)",
    "trxnFpxType": "CASA",
    "trxnFpxBuyerName": "...",
    "signature": "4lcfgSTO..."
  }
}
```

Fields are the section 1.5 set with these differences: `trxnFpxMethod` may also be `eMandate`; `trxnFpxType` is `CASA` or `Card` (callback docs say `LCA`); buyer name is `trxnFpxBuyerName` (callback uses `trxnBuyerName`); `trxnTime` is Malaysia time.

Inquiry response signature uses the same 11-field order as section 1.6 (buyer name excluded). Verified: `sushi4039314521110.00SBI Bank AB2C (Retail Banking)CASA20230314123232562201120230314123233` with key `wHqA4861VRpjtB0gPH3HW` gives `4lcfgSTOTTrsXUlFjtsSzf59IGHrU3Kb//3LhZBoMumxGYMV2N4Gd+Vdwt+QLwIHQg35Z+X7TOyFbijTmcoUiw==`.

Error envelope: `success: false`, `errorCode` set, `message` explains; the plugin treats non-200 or `success != true` as failure.

Void response body: not documented beyond sharing the section 3 request format (checked 2026-10-21). Which statuses can be voided and the void cut-off are not documented. After a void, confirm with Inquiry (`trxnStatus = 33`).

## 4. AutoDebit (DirectDebit eMandate)

Consent is an RM1.00 FPX payment by the customer from the account to be debited later. Success means the mandate is active; failure means the customer must enrol again. Per the shopper FAQ, the RM1 is deducted from the first recurring billing, debits run on a fixed date (moved to the next banking day on weekends or holidays), debit cards are not accepted, and customers maintain or cancel at `https://users.axaipay.my/login`.

Both enrolment types POST (browser form) to `/gateway/emandate/v1/enrolment`. You give Axaipay a Redirect URL (gets the status POST and shows the customer a page) and a Backend URL (same payload, plus every future recurring charge result).

### 4a. Enrolment with product code

The merchant first creates an AutoDebit product in the merchant portal; its link is `https://autodebit.axaipay.my/<productCode>`.

| Field | Type | M/O | Sample | Notes |
|---|---|---|---|---|
| `mchtId` | String(50) | M | `iboxfan` | |
| `mchtTrxnId` | String(255) | M | `12345` | |
| `signature` | String(255) | M | | |
| `productCode` | String(255) | M | `71aa54p` | From product link |
| `customerEmail` | String(27) | M | | |
| `customerName` | String(40) | M | | |
| `customerPhone` | Integer(11) | M | `0123456789` | Malaysian number |
| `customerIdentityType` | Integer(1) | M | `3` | 1 New IC, 2 Old IC, 3 Passport, 4 Business Registration, 5 Others |
| `customerIdentityNo` | String(18) | M | `434671` | |

Signature order: `customerEmail`, `customerIdentityNo`, `customerIdentityType`, `customerName`, `customerPhone`, `mchtId`, `mchtTrxnId`, `productCode`.

```text
abc@gmail.com4346713John Doe0123456789iboxfan1234571aa54p   key dwdefE12324!9293
=> HxH9TOdaXg7ZGUwtNZWsbYo9VxpJH03lza4Q68g+vEpdTdZfLV5zOYLARrOYeDtqCu1n13ArSZ6jlYrczTjxsQ==
```

### 4b. Direct enrolment (without product code)

Same as 4a but replace `productCode` with:

| Field | Type | M/O | Sample | Notes |
|---|---|---|---|---|
| `productDescription` | String(255) | M | `Fuji Apple 6packs` | |
| `maxDebitAmount` | Decimal | M | `1000.00` | Max per debit |
| `maxDebitFreq` | Integer | M | `10` | Max number of debits |
| `debitFreqMode` | String(2) | M | `MT` | `WK` weekly, `MT` monthly, `YR` yearly |

Signature order: `customerEmail`, `customerIdentityNo`, `customerIdentityType`, `customerName`, `customerPhone`, `debitFreqMode`, `maxDebitAmount`, `maxDebitFreq`, `mchtId`, `mchtTrxnId`, `productDescription`.

```text
abc@gmail.com4346713John Doe0123456789MT1000.0010iboxfan12345Fuji Apple 6packs   key dwdefE12324!9293
=> rV02ZTNwUc0VVL+UXyQr1JmJyHaBU7XYwiEKefTQWt/7lTdaCBzOL892VWqj/zwqiDV+wi8k/a/O4b2MEcNTlQ==
```

The enrolment link is unique and expires once the RM1.00 is paid. PDF sample-form errors to ignore: it posts `mchtTxnId` (use `mchtTrxnId` from the field table), includes a stray `txnAmount` text input, and the 4b sample reuses the 4a signature. The Java samples in 4b concatenate the wrong fields; follow the field list.

### 4c. Enrolment and recurring callback fields

Sent to redirect and backend for enrolment, and to backend for each recurring charge, with the same structure.

| Field | Type | Notes |
|---|---|---|
| `debitFreqMode` | String(2) | `WK`, `MT`, `YR` |
| `maxDebitAmount` | Decimal | e.g. `25.50` |
| `maxDebitFreq` | Integer | e.g. `2` |
| `mchtId` | String(50) | |
| `mchtTrxnId` | String(255) | Your enrolment transaction ID |
| `productCode` (4a) or `productDescription` (4b) | String(255) | |
| `trxnAmount` | Decimal | Amount actually paid (RM1.00 at enrolment) |
| `trxnBankName` | String | |
| `trxnFpxMethod` | String | `B2C (Retail Banking)` or `B2B (Corporate Banking)` |
| `trxnId` | String(255) | e.g. `EM20230202053432` |
| `trxnRecurringNo` | Integer | 0 = RM1.00 enrolment, then 1, 2, ... |
| `trxnStatus` | Integer | AutoDebit enum (below) |
| `trxnTime` | String | `yyyyMMddHHmmss` |
| `signature` | String | |

Signature order (all 13 fields alphabetical; the PDF's one-line header omits some, the example includes all): `debitFreqMode`, `maxDebitAmount`, `maxDebitFreq`, `mchtId`, `mchtTrxnId`, `productCode` or `productDescription`, `trxnAmount`, `trxnBankName`, `trxnFpxMethod`, `trxnId`, `trxnRecurringNo`, `trxnStatus`, `trxnTime`.

```text
MT25.502iboxfan20211234571aa54p20.00OCBC BankB2C (Retail Banking)EM2023020205343211120230202045513
key dwdefE12324!9293 => n/AE0d20fz15O9TfKQ8DBtStUy2+t3VBpwv60B629QVYCze67IkPO/wzP0n0vK0r4JDAfS/8KfcklOPLw2dsxA==

MT1000.0010iboxfan202112345Fuji Apple 6packs200.00OCBC BankB2C (Retail Banking)EM2023020205343211120230202045513
key dwdefE12324!9293 => lZhn0r3Uk4ZHKhVW7Zu6Osdbkt43KYHyIwPdskpz57HDbzZtT0ziHIXs5kSPMO2aHbKWGv+6w88uzTDR122eSQ==
```

Idempotency key for AutoDebit events: `trxnId` + `trxnRecurringNo`.

### 4d. AutoDebit inquiry

POST `/emandate/v1/inquiry`, headers as section 3.

| Field | Type | M/O | Notes |
|---|---|---|---|
| `merchantId` | String | M | |
| `trxnId` or `mchtTrxnId` | String | M | One of the two |
| `trxnRecurringNo` | Integer | O | Defaults to 0 (the enrolment payment) |

Signatures (both verified against the PDF, key `dwdefE12324!9293`):

- By `trxnId`: `merchantId + trxnId + trxnRecurringNo`. `iboxfan2021EM202302020534321` => `lBQJ+Blis0sB37bOFJmPhg1egyhB7WrY9bAqvSZFJnaUnUqelf0czoPNutQq+Aqvg94NoXLaf4PMyU5Tgc/OGw==`
- By `mchtTrxnId`: `mchtTrxnId + merchantId + trxnRecurringNo`. `12345iboxfan20211` => `gvxXE0zOBYrKrLzVhy/kl0iRsjWY89CUGL3ig9wkFysxP+S6EUoybpeqQ2wWnd4w+CAdOGpKLzqAgAoppcxEKg==`

Whether to append `trxnRecurringNo` when you omit it is not documented; send it explicitly (use `0` for the enrolment).

Response: `{ "success": true, "data": { ...4c fields... , "signature": "..." }, "message": "Inquiry successful" }`. `data` is null when the inquiry fails.

## Codes

### `trxnStatus` (eComPay)

| Code | Meaning | Action |
|---|---|---|
| 1 | Created | Wait |
| 2 | In Processing | Wait |
| 3 | Pending Authorization | Wait, then inquire |
| 11 | Paid | Settle after checks |
| 22 | Failed | Mark failed |
| 23 | Timeout | Mark expired; inquire before reusing stock |
| 33 | Void | Reverse fulfilment |
| 44 | Refund | Record refund |
| 55 | Cancel | Mark cancelled (after verification) |

### `trxnStatus` (AutoDebit)

`0` Created, `1` In Progress, `3` Pending Authorization, `11` Paid, `22` Failed, `23` Timeout, `55` Cancel.

### `trxnAcquirer`

`1` Credit Card, `10` MCash, `11` Boost, `12` GrabPay, `13` Touch 'n Go, `14` WannaPay, `15` Alipay, `16` Alipay+, `17` ShopeePay, `18` WavPay, `21` CIMB, `40` UnionPay, `50` DuitNowQR.

### `trxnPaymentMethod`

`1` Visa, `2` Mastercard, `3` Debit, `10` EWallet, `20` FPX, `30` Instalment, `50` UnionPay, `60` DuitNowQR.

### `acquirerCode` for bypass page (eComPay Appendix C)

FPX B2C: `B2C_TEST` Test Bank, `B2C_PBB0233` Public Bank, `B2C_BKRM0602` Bank Rakyat, `B2C_ABMB0212` Alliance Bank, `B2C_MB2U0227` Maybank2U, `B2C_BIMB0340` Bank Islam, `B2C_BMMB0341` Bank Muamalat, `B2C_KFH0346` Kuwait Finance House, `B2C_ABB0233` Affin Bank, `B2C_RHB0218` RHB Bank, `B2C_OCBC0229` OCBC Bank, `B2C_SCB0216` Standard Chartered, `B2C_HLB0224` Hong Leong Bank, `B2C_MBB0228` Maybank2E, `B2C_UOB0226` UOB Bank, `B2C_BCBB0235` CIMB Clicks, `B2C_AMBB0209` AmBank, `B2C_BSN0601` Bank Simpanan Nasional, `B2C_HSBC0223` HSBC Bank, `B2C_BOCM01` Bank of China, `B2C_AGRO01` Agrobank.

FPX B2B: `B2B_TEST` HostSim, `B2B_PBB0233` Public Bank, `B2B_ABMB0213` Alliance Bank, `B2B_MBB0228` Maybank2E, `B2B_BKRM0602` Bank Rakyat, `B2B_BIMB0340` Bank Islam, `B2B_BMMB0342` Bank Muamalat, `B2B_KFH0346` Kuwait Finance House, `B2B_ABB0232` Affin Bank, `B2B_RHB0218` RHB Bank, `B2B_OCBC0229` OCBC Bank, `B2B_SCB0215` Standard Chartered, `B2B_HLB0224` Hong Leong Bank, `B2B_DBB0199` Deutsche Bank, `B2B_UOB0227` UOB Bank, `B2B_BCBB0235` CIMB Clicks, `B2B_AMBB0208` AmBank, `B2B_HSBC0223` HSBC Bank, `B2B_PBB0234` Public Bank, `B2B_ABB0235` AFFINMAX, `B2B_BNP003` BNP Paribas Malaysia, `B2B_AGRO02` Agrobank.

E-wallet (`paymentMethod=EW`): `10` MCash, `11` Boost, `12` GrabPay, `13` Touch N Go, `14` WannaPay, `17` ShopeePay, `40` UnionPay.

### Error codes (eComPay Appendix B)

| Code | Meaning | Fix |
|---|---|---|
| `PY016` | Invalid signature | Check fields, order, signing key, method |
| `PY022` | Duplicate session for this request | Retry with a new unique `mchtTrxnId` |
| `PY034` | Invalid transaction amount | Amount present and within min/max allowed |
| `PY035` | Invalid merchant transaction ID | Include `mchtTrxnId` |
| `PY036` | Invalid customer name | Include `customerName` |
| `PY037` | Invalid customer email | Include `customerEmail` |
| `PY054` | Invalid backend URL | Valid `http(s)://` URL |
| `PY055` | Invalid redirect URL | Valid `http(s)://` URL |
| `I02` | Invalid order description | Include `orderDescription` |
| `IAP` | Invalid API key | Check `x-api-key` |
| `MAC` | Invalid API account | Check merchant ID |
| `MAP` | Missing API key | Send `x-api-key` header |
| `MSI` | Missing signature | Send `x-signature` header |
| `QRG` | Cannot generate QR now | Retry or contact support |
| `UAP` | Unexpected error | Retry or contact support |
| `UAQ` | Unknown acquirer ID | Check `acquirer` value |

Min and max amounts are referenced by `PY034` but not published.

## Sandbox Test Data (staging only)

Cards (eComPay Appendix A): holder `Test Card`, expiry `08/25`, CVV `123`. The printed expiry is in the past as of 2026-10-21; if staging rejects it, ask support for a current date.

| Scenario | Visa | Mastercard |
|---|---|---|
| Frictionless success | `4100000000000100` | `5100000000000107` |
| Challenge success (password `123456`) | `4100000000005000` | `5100000000005007` |
| Authentication failed (password `111111`) | `4100000000300005` | `5100000000300002` |

FPX staging banks: `B2C_TEST` (Test Bank), `B2B_TEST` (HostSim).

## Refunds

No refund API. The merchant completes the Refund Request Form (PDF, ref AXAI/RRF/20230131 v2.0) and emails it to `support@axaipay.com`. Axaipay processes within 2 working days; the customer may see it up to 14 working days later. The form states transaction fees are not refundable and a refund fee applies per the merchant agreement. Card refunds must go back to the same payment mode; other refunds can be same mode or DuitNow/IBG to a bank account. The WooCommerce plugin's `process_refund` only adds an order note asking for manual processing.

## Not Documented (checked 2026-10-21)

Callback retry schedule, required ack body, callback source IPs or allowlist, rate limits, idempotency headers, min/max amounts, void response body and eligibility, card tokenisation, payouts, settlement reports, webhooks beyond redirect/backend POSTs, currency other than MYR.

## Legacy Fields To Avoid

The OpenCart extension v1.0 (2022, Google Drive download linked from the KB) signs `backendUrl + cancelUrl + channel + customerEmail ...`. Those fields are not in eComPay v1.18; follow the v1.18 field list and the WooCommerce v2.0.0 plugin instead.
