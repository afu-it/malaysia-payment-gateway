# PayCools API Reference

Use this reference for factual PayCools (Malaysia) Open API guidance. It is distilled from the full public developer docs at `https://docs.paycools.com.my/en/` (25 pages, checked 2026-10-21; the Chinese mirror at `/zh/` was read as a cross-check). Confirm the base URL, UAT environment, enabled channels, and pricing with PayCools when those details matter.

## Source URLs

- Docs index: `https://docs.paycools.com.my/en/llms.txt`
- Merchant Unified API Param (request envelope, RSA signing): `https://docs.paycools.com.my/en/9367977m0`
- Unified API Param (notification envelope, RSA verify): `https://docs.paycools.com.my/en/9367979m0`
- API Code Collection: `https://docs.paycools.com.my/en/9367978m0`
- WebPay: `https://docs.paycools.com.my/en/507514603e0`, `/en/507514604e0`, `/en/507514605e0`, `/en/9368011m0`
- QR Code: `https://docs.paycools.com.my/en/507514599e0`, `/en/507514600e0`, `/en/507514601e0`, `/en/507514602e0`, `/en/9368008m0`
- Refund: `https://docs.paycools.com.my/en/507514606e0`, `/en/507514607e0`, `/en/507514608e0`, `/en/9368019m0`, `/en/9368021m0`
- Disbursement: `https://docs.paycools.com.my/en/508846511e0`, `/en/508846512e0`, `/en/507514596e0`, `/en/507514597e0`, `/en/507514598e0`, `/en/9368003m0`, `/en/9392778m0`
- Account: `https://docs.paycools.com.my/en/507514595e0`

Every page is also available as Markdown by appending `.md` to the URL.

## Integration Surfaces

- **WebPay** (hosted redirect): FPX, DuitNow RPP online banking, cards, Touch 'n Go, Boost, GrabPay, MCash. You choose the channel up front with `channelCode`; PayCools returns a `payUrl`.
- **DuitNow dynamic QR**: you render `qrCodeContent` (EMV QR string) yourself.
- **Refunds** for WebPay channels (full and partial).
- **Disbursement** (payouts) to Malaysian banks and e-wallets, with account validation (bank account or DuitNow proxy) first.
- **Balance inquiry**.
- Payment Links and the merchant dashboard exist as products on `https://www.paycools.com.my` but have no public API docs.

Not documented in official docs (checked 2026-10-21): recurring billing, card tokenisation, capture/void, order cancel, settlement reports, payment link API, hosted checkout with method selection by PayCools.

## Environments

- API base URL: Not documented in official docs (checked 2026-10-21). Every OpenAPI block has `servers: []`. Get the production and UAT base URLs from PayCools during onboarding and store as `PAYCOOLS_API_BASE_URL`.
- Sandbox/UAT: Not documented. The WebPay detail example returns a `uat.pbebank.com` bank URL, which suggests a UAT flow exists, but no host or test credentials are published. Ask PayCools.
- Paths below are relative to the base URL, for example `{PAYCOOLS_API_BASE_URL}/open-api/webpay/generate`.

## Endpoint Catalogue

Merchant calls PayCools (all `POST`, `Content-Type: application/json`, body `{appId, sign, param}`):

| # | Method | Path | Purpose |
| --- | --- | --- | --- |
| 1.1 | POST | `/open-api/account/balance` | Merchant balance inquiry |
| 2.1 | POST | `/open-api/inquiry-account` | Payout account validation (bank or DuitNow proxy) |
| 2.2 | POST | `/open-api/inquiry-account/query` | Account validation order status |
| 2.3 | POST | `/open-api/disbursement` | Create payout |
| 2.4 | POST | `/open-api/disbursement/check` | Payout status |
| 3.1 | POST | `/open-api/qr/generate` | Create DuitNow dynamic QR |
| 3.2 | POST | `/open-api/qr/detail` | QR order detail/status |
| 3.3 | POST | `/open-api/qr/paymentList` | Payments made on a QR within a date range |
| 4.1 | POST | `/open-api/webpay/generate` | Create WebPay order, returns `payUrl` |
| 4.2 | POST | `/open-api/webpay/detail` | WebPay order status |
| 5.1 | POST | `/open-api/payment/refund` | Create refund |
| 5.2 | POST | `/open-api/refund/payments` | Query refunds |

PayCools calls the merchant (merchant implements; body `{sign, param}`; the docs' paths are placeholders, PayCools calls the URL you pass):

| # | Method | Placeholder path | Called on URL | Purpose |
| --- | --- | --- | --- | --- |
| 2.5 | POST | `/callbacks/disbursement` | Not stated (no notify field in 2.3; configured with PayCools) | Payout result |
| 3.4 | POST | `/callbacks/qrcode-payment` | QR `notifyUrl` | QR payment result |
| 4.3 | POST | `/callbacks/webpay-payment` | WebPay `notifyUrl` | WebPay payment result |
| 5.3 | POST | `/callbacks/refund` | `refundCallbackUrl` | Refund result |

## Request Envelope (merchant to PayCools)

| Field | Required | Type | Notes |
| --- | --- | --- | --- |
| `appId` | Yes | string | Merchant APP ID, provided by PayCools. |
| `sign` | Yes | string | Base64 RSA signature of `param`. |
| `param` | Yes | string | The business parameters, serialized to a JSON string. |

```json
{
  "appId": "<APP_ID>",
  "sign": "<BASE64_SIGNATURE>",
  "param": "{\"timestamp\":1676017834369,\"mchOrderId\":\"ORDER0001\"}"
}
```

Steps documented: build the business params for the endpoint, format them as JSON and use that string as `param`, sign `param` with the merchant RSA private key and use the result as `sign`.

## Response Envelope

```json
{ "code": 10000, "message": "Success", "data": { } }
```

`code` 10000 is success. On errors `data` may be `null`. The payout pages show success as `"code": 1000, "message": "success"` in their examples, which conflicts with the code table; treat `10000` as success and log anything else for review.

## Signature

### Algorithm (verified)

- Algorithm: `SHA256withRSA` (RSASSA-PKCS1-v1_5 with SHA-256).
- Key size: 4096 bits (docs: "RSA key length should be 4096").
- String to sign: the exact `param` string, byte for byte. No sorting, no concatenation with other fields, no `appId`.
- Output: standard Base64 of the signature bytes.
- Merchant private key format: Base64 of PKCS#8 DER (`PKCS8EncodedKeySpec`). Wrap in `-----BEGIN PRIVATE KEY-----` for PEM-based libraries.
- Public key format: Base64 of X.509 SubjectPublicKeyInfo DER (`X509EncodedKeySpec`). Wrap in `-----BEGIN PUBLIC KEY-----`.
- Encoding: the Java sample uses `param.getBytes()` (platform default charset). Use UTF-8 explicitly and keep `param` ASCII-safe where possible.

The docs' own example (public key, `param`, expected `sign`) was checked with `openssl dgst -sha256 -verify` on 2026-10-21 and returned `Verified OK`, confirming the method above. The docs publish an example key pair; never use it for real traffic.

### Directions

| Direction | Signed by | Verified with |
| --- | --- | --- |
| Merchant to PayCools (Open API calls) | Merchant private key | Merchant public key, which you give PayCools |
| PayCools to merchant (notifications) | PayCools private key | PayCools public key, which you request from the PayCools administrator |

If PayCools fails to verify a request it refuses it (`21001 OPI_SIGN_ERROR`).

### Pseudo-code (Node.js)

```js
import crypto from "node:crypto";

const toPem = (b64, label) =>
  `-----BEGIN ${label}-----\n${b64.match(/.{1,64}/g).join("\n")}\n-----END ${label}-----\n`;

const privateKey = toPem(process.env.PAYCOOLS_PRIVATE_KEY, "PRIVATE KEY");          // PKCS#8
const paycoolsPublicKey = toPem(process.env.PAYCOOLS_PLATFORM_PUBLIC_KEY, "PUBLIC KEY");

export async function callPaycools(path, params) {
  const param = JSON.stringify({ ...params, timestamp: Date.now() }); // serialize ONCE
  const sign = crypto.sign("sha256", Buffer.from(param, "utf8"), privateKey).toString("base64");
  const res = await fetch(process.env.PAYCOOLS_API_BASE_URL + path, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ appId: process.env.PAYCOOLS_APP_ID, sign, param }),
  });
  const body = await res.json();
  if (body.code !== 10000) throw new Error(`PayCools ${body.code} ${body.message}`);
  return body.data;
}

export function verifyNotification(rawBody) {
  const { sign, param } = JSON.parse(rawBody);          // outer envelope only
  if (typeof sign !== "string" || typeof param !== "string") return null;
  const ok = crypto.verify("sha256", Buffer.from(param, "utf8"), paycoolsPublicKey,
                           Buffer.from(sign, "base64"));
  return ok ? JSON.parse(param) : null;                  // parse param only after verify
}
```

Python equivalent: `cryptography` `private_key.sign(param.encode(), padding.PKCS1v15(), hashes.SHA256())`, Base64-encode; verify with `public_key.verify(base64.b64decode(sign), param.encode(), padding.PKCS1v15(), hashes.SHA256())`.

## WebPay (hosted redirect)

### 4.1 Create: `POST /open-api/webpay/generate`

`param` fields:

| Field | Required | Type | Length | Notes |
| --- | --- | --- | --- | --- |
| `timestamp` | Yes | long | - | Current timestamp. |
| `channelCode` | Yes | string | - | From the WebPay channel code list below. |
| `mchOrderId` | Yes | string | 1-32 | Merchant order ID, alphanumeric, max 32. Must be unique. |
| `amount` | Yes | integer | - | Amount in cents. |
| `notifyUrl` | Yes | string | 1-256 | Merchant payment notification URL. |
| `redirectUrl` | Yes | string | 1-512 | Browser redirect URL after payment. |
| `customerName` | Yes | string | 1-128 | Customer name (docs also say max length 100). |
| `email` | Yes | string | 1-128 | Customer email. |
| `mobile` | Yes | string | 10-11 | Customer mobile number. The Chinese page adds: starts with `01`, max 10 digits. |
| `remark` | No | string | 1-256 | Remark. |
| `expireSeconds` | No | integer | - | Expiry in seconds, 30 to 3600, default 3600. |

Docs inconsistency: the example `param` uses `callbackUrl` instead of `notifyUrl`. The field table (EN and ZH) requires `notifyUrl`. Send `notifyUrl`; confirm with PayCools if notifications do not arrive.

Example `param` (adapted from the docs' example: `callbackUrl` replaced by `notifyUrl`, placeholder URLs and mobile):

```json
{"mchOrderId":"EXAMPLE00000001","amount":1100,"customerName":"Jonathan","channelCode":"RPP_PUBLIC_URL","email":"example@gmail.com","mobile":"0123456789","notifyUrl":"https://merchant.example/paycools/notify","redirectUrl":"https://merchant.example/return","timestamp":1676017834369,"expireSeconds":300}
```

Response `data`:

| Field | Notes |
| --- | --- |
| `payUrl` | Redirect the customer here (it can be a bank, card processor, or wallet URL). |
| `transactionId` | PayCools transaction ID. Store it. |
| `mchOrderId` | Echo of your order ID. |
| `expiresTime` | `yyyy-MM-dd HH:mm:ss`. Timezone not stated. |

### 4.2 Status: `POST /open-api/webpay/detail`

`param`: `timestamp` (long, required), `mchOrderId` (string, required).

Response `data`:

| Field | Notes |
| --- | --- |
| `mchOrderId`, `transactionId` | Identifiers. |
| `payUrl` | The payment URL. |
| `transactionStatus` | `PENDING`, `COMPLETED`, `FAILED`. |
| `amount` | Cents. |
| `email`, `mobile`, `remark` | As sent. |
| `callbackUrl`, `redirectUrl` | URLs on the order (the detail response names the notify URL `callbackUrl`). |
| `channelCode` | Channel used. |
| `createTime`, `returnTime` | `yyyy-MM-dd HH:mm:ss`; `returnTime` is null until finished. |
| `cardNo`, `cardScheme`, `cardIssuer` | Masked card number and scheme for card payments. |

### 4.3 Notification (merchant implements, sent to `notifyUrl`)

Sent when a payment transaction completes. Body `{sign, param}`. `param` fields:

| Field | Required | Type | Notes |
| --- | --- | --- | --- |
| `eventName` | Y | string | `order.payment.success` or `order.payment.failed`. |
| `mchOrderId` | Y | string | Your order ID. |
| `transactionId` | Y | string | PayCools transaction ID. |
| `amount` | Y | integer | Amount (cents, same unit as create). |
| `transactionStatus` | Y | string | `PENDING`, `COMPLETED`, `FAILED`. |
| `createTime` | Y | string | Payment create time. |
| `returnTime` | Y | string | Payment finish time. |
| `channelCode` | Y | string | Channel code. |
| `remark` | Y | string | Remark. |
| `failedCode` | N | integer | Failure code. |
| `failedMessage` | N | string | Failure message. |

Expected merchant response: `{"code":10000,"message":"Success"}`.

The docs' example `param` also contains a `"sign"` key inside `param`. Do not rely on it; verify the outer `sign` over the `param` string.

### WebPay channel codes

| Group | Codes |
| --- | --- |
| E-wallets | `BOOST_URL` (Boost), `GRPY_URL` (GrabPay), `TOUCHNGO_URL` (Touch 'n Go), `MCASH_URL` (MCash) |
| Card | `CARD_URL` |
| FPX | `FPX_AFFIN_URL`, `FPX_ALLIANCE_URL`, `FPX_AGRO_URL`, `FPX_AMBANK_URL`, `FPX_CIMB_URL`, `FPX_BANK_ISLAM_URL`, `FPX_KERJASAMA_RAKYAT_URL`, `FPX_MUAMALAT_URL`, `FPX_SIMPANAN_NASIONAL_URL`, `FPX_HONG_LEONG_URL`, `FPX_HSBC_URL`, `FPX_KUWAIT_URL`, `FPX_MAYBANK_URL`, `FPX_OCBC_URL`, `FPX_PUBLIC_URL`, `FPX_RHB_URL`, `FPX_STANDARD_CHARTERED_URL`, `FPX_UOB_URL`, `FPX_BOC_URL` |
| DuitNow RPP (online banking) | `RPP_CIMB_URL`, `RPP_FINEXUS_URL` (Finexus kayaaku), `RPP_HONG_LEONG_URL`, `RPP_OCBC_URL`, `RPP_PUBLIC_URL`, `RPP_AFFIN_URL`, `RPP_UOB_URL`, `RPP_MBSB_URL`, `RPP_AMBANK_URL`, `RPP_MUAMALAT_URL`, `RPP_SIMPANAN_NASIONAL_URL`, `RPP_HSBC_URL`, `RPP_RHB_URL` |

Because the bank or wallet is fixed per order, your checkout page must show the method/bank picker and pass the chosen code. Channel availability depends on your merchant account (`21005 Merchant channel is unavailable`).

## DuitNow QR

### QR channel codes

| `channelCode` | Notes |
| --- | --- |
| `DUITNOW_DYNAMIC_QR` | Dynamic DuitNow QR. |
| `DUITNOW_STATIC_QR` | Static DuitNow QR. Cannot be created by merchants via Open API for now. |

### 3.1 Create: `POST /open-api/qr/generate`

| Field | Required | Type | Length | Notes |
| --- | --- | --- | --- | --- |
| `timestamp` | Yes | long | - | Current timestamp. |
| `channelCode` | Yes | string | - | `DUITNOW_DYNAMIC_QR`. |
| `mchOrderId` | Yes | string | 1-32 | Alphanumeric, unique. |
| `amount` | Yes | integer | - | Cents. |
| `notifyUrl` | Yes | string | 1-256 | Notification URL. |
| `customerName` | No | string | 1-128 | Max length 100. |
| `email` | No | string | 1-128 | |
| `mobile` | No | string | 10-11 | |
| `remark` | No | string | 1-256 | |
| `expireSeconds` | No | integer | - | 30 to 3600, default 3600. |

Response `data`: `mchOrderId`, `transactionId`, `qrCodeContent` (EMV DuitNow QR payload string to render as a QR image), `channelCode`, `notifyUrl`, `remark`, `qrStatus` (example `ACTIVE`; other values not documented), `transactionStatus` (`PENDING` at creation), `qrLink` (null in example), `createTime`, `updateTime`, `expiresTime`.

### 3.2 Detail: `POST /open-api/qr/detail`

`param`: `timestamp`, `mchOrderId`. Response `data`: same fields as create (`mchOrderId`, `transactionId`, `qrCodeContent`, `channelCode`, `notifyUrl`, `remark`, `qrStatus`, `transactionStatus`, `qrLink`, `createTime`, `updateTime`).

### 3.3 Payment list: `POST /open-api/qr/paymentList`

| Field | Required | Type | Notes |
| --- | --- | --- | --- |
| `timestamp` | Yes | long | |
| `mchOrderId` | Yes | string | QR order ID. |
| `startDate` | Yes | string | `yyyy-MM-dd`. |
| `endDate` | Yes | string | `yyyy-MM-dd`. |

Response `data` is an array of: `transactionId`, `channelCode`, `transactionCreateTime`, `transactionReturnTime`, `transactionAmount` (cents), `transactionStatus`, `remark`.

### 3.4 Notification (merchant implements, sent to QR `notifyUrl`)

Sent when a payment occurs on the QR. `param` fields: `eventName` (`qrcode.payment.success`, `qrcode.payment.failed`), `mchOrderId`, `transactionId`, `amount`, `transactionStatus` (`PENDING`, `COMPLETED`, `FAILED`), `createTime`, `returnTime`, `channelCode`, `remark`, and optional `failedCode` (integer), `failedMessage`. All non-failure fields are marked required, though the docs' example omits `eventName` and `mchOrderId`; requery `qr/detail` if either is missing. Reply `{"code":10000,"message":"Success"}`.

## Refunds

Refund support (reference only; "the actual refund request should prevail"): full and partial refunds for every `FPX_*_URL` channel, `TOUCHNGO_URL`, `BOOST_URL`, `MCASH_URL`, `GRPY_URL`, and `CARD_URL`. For refunds, the transaction fee is not returned. `RPP_*` and DuitNow QR channels are not in the refund support list.

### 5.1 Create: `POST /open-api/payment/refund`

| Field | Required | Type | Length | Notes |
| --- | --- | --- | --- | --- |
| `mchRefundOrderId` | Y | string | 1-32 | Unique merchant refund ID, letters + numbers. |
| `originalTransactionId` | Y | string | 1-32 | PayCools `transactionId` of the original payment. |
| `refundAmount` | Y | long | - | Cents. |
| `refundReason` | N | string | 1-200 | |
| `refundCallbackUrl` | Y | string | 1-256 | Refund notification URL. |
| `timestamp` | Y | long | - | Example shows a 10-digit value (seconds); other pages use milliseconds. Confirm with PayCools. |

Response `data`: `mchRefundOrderId`, `refundTransactionId`, `refundCreateTime`, `refundAmount`, `refundStatus` (`PENDING` at creation).

### 5.2 Query: `POST /open-api/refund/payments`

`param`: `originalTransactionId` (N) or `refundTransactionId` (N), one of the two; `timestamp` (Y).

Response `data` is an array of: `mchRefundOrderId`, `originalTransactionId`, `refundTransactionId`, `refundAmount`, `feeRefunded`, `refundStatus`, `refundCreateTime`, `refundReturnTime`, and on failure `failedCode` (example `-10010`), `failedMessage` (example `Channel Refund Failed`).

### 5.3 Notification (merchant implements, sent to `refundCallbackUrl`)

| Field | Required | Type | Notes |
| --- | --- | --- | --- |
| `eventName` | Y | string | `payment.refund.success`, `payment.refund.failed`, `payment.refund.cancel`. |
| `mchRefundOrderId` | Y | string | |
| `refundTransactionId` | Y | string | |
| `originalTransactionId` | Y | string | |
| `refundAmount` | Y | int | Cents. |
| `feeRefunded` | Y | int | |
| `refundStatus` | Y | string | See enum. |
| `refundCreateTime` | Y | string | |
| `refundReturnTime` | Y | string | |
| `refundReason` | Y | string | |
| `failedCode` | N | int | On failure. |
| `failedMessage` | N | string | On failure (docs type says int; treat as string). |

### Refund status enum

| Status | Meaning |
| --- | --- |
| `PENDING` | Awaiting processing |
| `PROCESS` | Processing |
| `COMPLETED` | Successful |
| `FAILED` | Failed |
| `CANCELLED` | Cancelled |

The query and notification examples show `COMPLETE` (no D). Map both `COMPLETE` and `COMPLETED` to success, and key off `eventName` too.

## Disbursement (payouts)

### 2.1 Account validation: `POST /open-api/inquiry-account`

| Field | Required | Type | Notes |
| --- | --- | --- | --- |
| `timestamp` | Yes | long | Milliseconds (examples show seconds; confirm). |
| `mchOrderId` | Yes | string (10-32) | Unique validation order ID. |
| `accountBankCode` | Conditional | string (1-20) | Required when `accountType=BANK`; optional for `DUITNOWPROXY`. Bank codes from the payout bank list. |
| `accountNo` | Yes | string (1-50) | Bank account, mobile number, or DuitNow ID. |
| `accountType` | Yes | string | `BANK` or `DUITNOWPROXY`. |
| `accountName` | No | string (1-128) | Payee name. |
| `proxyType` | Conditional | string | Required when `accountType=DUITNOWPROXY`. |
| `passportCountryCode` | Conditional | string (3) | Required when `proxyType=PSPT`. ISO 3166-1 alpha-3. |

DuitNow `proxyType` values: `NRIC` (alphanumeric, no hyphens), `PSPT` (alpha-3 country code + passport number), `BREG` (business registration, no hyphens), `MBNO` (country code + mobile, max 15 digits, e.g. `6010xxxxxxx`), `ARMN` (military/police ID, no hyphens).

Response `data`: `transactionId`, `mchOrderId`, `accountStatus` (`VALID`, `INVALID`, `UNKNOWN`: query again later), `accountName` (actual holder name), `accountNo`, `accountBankCode`, `accountType` (`BANK`, `VA`, `EWALLET`, `UNKNOWN`), `failedCode`, `failedMessage`, `orderStatus` (`PENDING`, `SUCCESS`, `FAILED`), `accountNameMatchResult` (`MATCH`, `NOT_MATCH`: risk, `UNKNOWN`), `returnTime`.

Validation failure codes: `-20001` Invalid parameters, `-20002` Account abnormal, `-20003` Balance insufficient, `-20004` Channel error, `-20005` System error, `-20006` Unknown error.

### 2.2 Validation status: `POST /open-api/inquiry-account/query`

`param`: `mchOrderId` (1-32), `timestamp` (ms). Response `data`: same fields as 2.1. Not found returns `21006`.

### 2.3 Create payout: `POST /open-api/disbursement`

| Field | Required | Type | Length | Notes |
| --- | --- | --- | --- | --- |
| `timestamp` | Y | long | - | Current timestamp (ms). |
| `mchOrderId` | Y | string | 10-32 | Unique merchant order ID. |
| `idempotencyId` | Y | string | 10-32 | Idempotency key (docs: "letter + space, 10-30 characters"). Duplicate returns `21015`. |
| `amount` | Y | long | - | Cents. |
| `accountNo` | Y | string | 1-50 | Beneficiary account. |
| `accountName` | Y | string | 1-128 | Payee name. |
| `payMethodCode` | Y | string | - | `BANK` or `EWALLET`. |
| `transferCode` | Y | string | - | Bank or e-wallet code from the lists below. |
| `accountType` | Y | string | - | `MOBILE_NUMBER` or `BANK_ACCOUNT` (the docs' example omits it). |
| `remark` | N | string | 1-256 | |

Response `data`: `transactionId`, `transactionStatus` (`PENDING`), `transactionTime`. Insufficient balance returns `21014`.

### 2.4 Payout status: `POST /open-api/disbursement/check`

`param`: `mchOrderId` (10-32), `timestamp` (ms). Response `data`: `mchOrderId`, `transactionId`, `amount`, `transactionStatus` (`PENDING`, `COMPLETED`, `FAILED`), `createTime`, `returnTime`, `remark`, `failedCode`, `failedMessage`.

### 2.5 Payout notification (merchant implements)

`param`: `eventName` (`disbursement.success`, `disbursement.failed`), `mchOrderId`, `transactionId`, `amount`, `transactionStatus`, `createTime`, `returnTime`, `remark`, optional `failedCode`, `failedMessage`. The create request has no notify URL field, so the payout callback URL is presumably configured with PayCools; confirm during onboarding.

### Payout bank `transferCode` list

`AGOB` Agrobank, `ARBK` AmBank, `BIMB` Bank Islam, `BKCH` Bank of China, `BKRM` Bank Rakyat, `BMMB` Bank Muamalat, `BNPA` BNP Paribas, `BOFA` Bank of America, `BSNA` BSN, `CHAS` JP Morgan Chase, `CIMB` CIMB, `CITI` Citibank, `DEUT` Deutsche Bank, `HBMB` HSBC, `HLBB` Hong Leong Bank, `KFHO` Kuwait Finance House, `MFBB` Alliance Bank, `MHCB` Mizuho, `OCBC` OCBC, `PBBE` Public Bank, `PHBM` Affin Bank, `RHBB` RHB, `SCBL` Standard Chartered, `SMBC` Sumitomo Mitsui, `UOVB` UOB, `AFBQ` MBSB Bank, `AIBB_SPI` Affin Islamic, `ALIBB_SPI` Alliance Islamic, `RJHIC` Al-Rajhi, `ABB_SPI` AmIslamic, `PCBC` China Construction Bank, `CIMBI_SPI` CIMB Islamic, `HLIB_SPI` Hong Leong Islamic, `HAMB_SPI` HSBC Amanah, `ICBK` ICBC, `MLBBE` Maybank, `MBIS_SPI` Maybank Islamic, `OABB_SPI` OCBC Al-Amin, `PIBE_SPI` Public Islamic, `RHBA_SPI` RHB Islamic, `SCSR` Standard Chartered Saadiq, `BKKB` Bangkok Bank, `BOTKM` MUFG Bank.

### Payout e-wallet `transferCode` list

`TNG` Touch 'n Go, `FINEXUS` Finexus Cards, `BGP` BigPay, `BOOST` Boost, `Shopee` Shopee, `GXB` GX Bank, `MASB` Merchantrade.

## Account

### 1.1 Balance: `POST /open-api/account/balance`

`param`: `timestamp` (long, required). Response `data`: `balance` (example `3688`; unit not stated, presumably cents like other amounts).

## Response Codes

| Code | Inner code | Message |
| --- | --- | --- |
| 10000 | SUCCESS | Success |
| 21000 | OPI_GENERAL_ERROR | Service error, please contact administrator |
| 21001 | OPI_SIGN_ERROR | Request sign error |
| 21002 | OPI_PARAM_ERROR | Request param error |
| 21003 | OPI_APPID_ERROR | Request appId error |
| 21004 | OPI_MCH_ERROR | Merchant is unavailable |
| 21005 | OPI_MCH_CHANNEL_UNAVAILABLE_ERROR | Merchant channel is unavailable |
| 21006 | OPI_MCH_TX_NOT_FOUND | Merchant transaction not found |
| 21007 | OPI_DUPLICATE_MCH_ORDER_ID | Duplicate merchant order id |
| 21008 | OPI_TIMESTAMP_ERROR | Request timestamp error |
| 21009 | OPI_TIMESTAMP_EXPIRED_ERROR | Request timestamp error (expired) |
| 21010 | OPI_AMOUNT_EXCEED_ERROR | The amount exceeds the limit |
| 21011 | OPI_NETWORK_ERROR | Network error |
| 21012 | OPI_BANK_CODE_ERROR | Request bank code error |
| 21013 | OPI_EWALLET_CODE_ERROR | Request wallet code error |
| 21014 | OPI_BALANCE_ERROR | Merchant balance insufficient |
| 21015 | OPI_DUPLICATE_IDEMPOTENCY_ID | Duplicate idempotencyId |
| 21016 | OPI_MCH_IP_ERROR | Merchant ip forbidden |

`21008`/`21009` mean the `timestamp` is wrong or too old: keep server clocks in sync and generate `timestamp` per request. The allowed skew is not documented. `21016` means your calling IP is not allowed; ask PayCools to allowlist your egress IPs.

## Amount and Data Rules

- All amounts are integers in cents: RM 11.00 is `1100`.
- `mchOrderId`: alphanumeric, max 32 (payout and validation pages say 10-32).
- Dates in responses: `yyyy-MM-dd HH:mm:ss`; timezone not documented (assume Malaysia time, confirm).
- Minimum and maximum amounts: not documented; `21010` signals a limit breach.

## Settlement Flow (recommended)

1. Insert local payment `pending` with `mchOrderId`, amount (cents), `channelCode`.
2. Call create; store `transactionId`, `payUrl` or `qrCodeContent`, `expiresTime`.
3. Redirect or show QR. On `redirectUrl`, show "confirming payment" and poll your own backend.
4. On notification: verify `sign`, parse `param`, look up by `mchOrderId`, check `transactionId`, `amount`, `channelCode`, then move `pending` to `paid` only if `transactionStatus=COMPLETED`, inside a transaction with a unique constraint. Reply `{"code":10000,"message":"Success"}` after durable recording.
5. If no notification by expiry, call the detail endpoint; treat `PENDING` past `expiresTime` as unpaid but keep it open for a late notification.

## Not Documented (checked 2026-10-21)

- API base URLs (production and UAT), sandbox test banks/cards/wallets.
- Notification retry schedule, timeout, and whether a non-10000 reply triggers retry.
- PayCools notification source IPs.
- Rate limits.
- Allowed `timestamp` skew and unit (ms vs s) on the refund and account validation pages.
- Recurring, tokenisation, capture/void, cancel, settlement reports, payment link API.
