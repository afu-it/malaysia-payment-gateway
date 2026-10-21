# Gkash Offline and In-Store Payments Reference

Use this file for in-store or server-driven QR, SoftPOS terminal, and Soundbox flows. Online checkout lives in `gkash-api.md`. All facts below come from the official docs (checked 2026-10-21) unless marked otherwise:

- E-wallet payments: `https://doc.gkash.my/ewallet-payments`
- Terminal payments: `https://doc.gkash.my/terminal-payments`
- Soundbox payments: `https://doc.gkash.my/soundbox-payments`

All URLs below are staging (`https://api-staging.pay.asia`). The production host for these paths is not stated on any doc page (checked 2026-10-21); see the Environments section of `gkash-api.md` and confirm with Gkash.

## Two Signature Families (read first)

The offline pages use two different signature conventions. Do not mix them.

| Family | Endpoints | String to sign | Case rule | Output |
| --- | --- | --- | --- | --- |
| A: payment API | `/api/payment/submit` (QR scan, QR display, Soundbox) | `key;CID;v_cartid;v_amount;v_currency` | Uppercase the whole string before hashing | Lowercase hex |
| A: response/callback verify | Sync response and callback of the above | `key;CID;POID;cartid;amount;currency;status` | Uppercase the whole string before hashing | Lowercase hex |
| B: merchant API (`/apim/merchant/*`) | `RemotePay` | `key;Amount;Currency;ReferenceNo;TerminalId` | Do NOT uppercase the input (see note) | UPPERCASE hex |
| B | `RemoteCancel`, `SoundboxCancel` | `key;TerminalId` | See note | UPPERCASE hex |
| B | `PublishInvoice` | `key;ReferenceNo;TerminalId;InvoiceValue` | Do NOT uppercase the input (see note) | UPPERCASE hex |

Amount rule is the same everywhere: 2 decimals, commas and dot removed (`100.00` -> `10000`, `1` -> `100`, `0.1` -> `010`).

Note on family B case handling (verified locally by recomputing SHA512, 2026-10-21):

- `RemotePay` text says "Hash the resultant string with SHA512. Convert the resultant string to uppercase." (hash first, then uppercase the hex). The documented vector matches that: `SHA512("SIGNATUREKEY9999;10000;MYR;merchant-reference-712894;M161-TD-999")` = `03F4F438...4FF14` in uppercase hex. Uppercasing the input first gives a different hash (`371420aa...`).
- `PublishInvoice` and `RemoteCancel` text says "Convert the resultant string to uppercase. Hash ...", but the `PublishInvoice` vector (`AACAB685...3409`) equals SHA512 of the NON-uppercased input, output as uppercase hex. Uppercasing the input gives `7332a202...`, which does not match. The `RemoteCancel` vector (`ED17C82C...2E62`) is SHA512 of `SIGNATUREKEY9999;M161-TD-999`, already uppercase, so both readings agree there.
- The `signature` example inside the `RemoteCancel` and `SoundboxCancel` parameter tables (`D0E18286...`) does not equal the worked vector `ED17C82C...`; treat the table value as a placeholder.
- Conclusion: for family B, hash the raw string and send uppercase hex. Confirm with Gkash if your signature key contains lowercase letters, because the docs text and vectors disagree.

Pseudo-code:

```ts
import { createHash } from "node:crypto";
const sigB = (parts: string[]) =>
  createHash("sha512").update(parts.join(";"), "utf8").digest("hex").toUpperCase();

sigB([KEY, "10000", "MYR", referenceNo, terminalId]);           // RemotePay
sigB([KEY, terminalId]);                                          // RemoteCancel, SoundboxCancel
sigB([KEY, referenceNo, terminalId, invoiceValue]);               // PublishInvoice
```

## 1. Customer Presented QR (merchant scans the buyer's e-wallet QR)

```text
POST https://api-staging.pay.asia/api/payment/submit
Content-Type: application/x-www-form-urlencoded
```

Request:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `version` | String | Yes | Constant `1.5.5`. |
| `CID` | String | Yes | Merchant ID, for example `M161-U-999`. |
| `v_cartid` | String | Yes | Unique merchant reference. Special characters: `-`, `_`, `.`. |
| `v_currency` | String | Yes | `MYR` only. |
| `v_amount` | String | Yes | For example `100.00`. |
| `signature` | String | Yes | Family A request signature, lowercase hex. |
| `QRCode` | String | Yes | The scanned QR value. |
| `terminalID` | String | Yes | Merchant terminal ID, for example `M161-TD-999`. |
| `callbackurl` | String | No | Server-to-server result URL. |
| `v_firstname`, `v_lastname`, `v_billemail`, `v_billstreet`, `v_billpost`, `v_billcity`, `v_billstate`, `v_billcountry`, `v_billphone` | String | No | Buyer and billing details, as in web payments. |
| `v_shipstreet`, `v_shippost`, `v_shipcity`, `v_shipstate`, `v_shipcountry` | String | No | Shipping details. |
| `clientip`, `v_productdesc` | String | No | Buyer IP, product description. |

Synchronous JSON response (HTTP 200):

| Field | Required | Notes |
| --- | --- | --- |
| `CID` | Yes | Merchant ID. |
| `POID` | Yes | Gkash transaction reference. |
| `status` | Yes | `88 - Transferred`, `66 - Failed`, `11 - Pending`. |
| `cartid`, `currency`, `amount` | Yes | Echoed. |
| `signature` | Yes | Docs: "It is mandatory to verify the signature sent by Gkash", family A verify formula. |
| `description` | No | Informational only. Example `SUCCESS`. |
| `PaymentType` | No | For example `TNG QR SCAN`, `GrabPay QR SCAN`. |

The example response signature (`7f152d80...`) does not match the documented verify formula for the example data (verified locally, 2026-10-21). Confirm final state with the callback or `POST /api/payment/query` (see `gkash-api.md`).

## 2. Merchant Presented QR (merchant displays a dynamic QR)

```text
POST https://api-staging.pay.asia/api/payment/submit
Content-Type: application/x-www-form-urlencoded
```

Request: the same fields as customer presented QR, minus `QRCode`, plus:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `terminalID` | String | Yes | Merchant terminal ID. |
| `paymentid` | String | Yes | Payment method ID, see table below. |
| `OpenId` | String | Conditional | WeChat OpenId. Only for WeChat Pay Mini Program (`46171`). |

`paymentid` values:

| Code | Method |
| --- | --- |
| `95` | TNG QR DISPLAY |
| `72` | GRABPAY QR DISPLAY |
| `69` | BOOST QR DISPLAY |
| `86` | MAYBANK QR DISPLAY |
| `46148` | WECHAT QR DISPLAY |
| `84` | ALIPAY QR DISPLAY |
| `98` | GKASH QR DISPLAY |
| `46118` | SHOPEEPAY QR DISPLAY |
| `46119` | DUITNOW QR DISPLAY |
| `46113` | ATOME QR DISPLAY |
| `46122` | MCASH QR DISPLAY |
| `46143` | SARAWAKPAY QR DISPLAY |
| `46171` | WECHAT PAY MINI PROGRAM |

Synchronous JSON response: same fields as customer presented QR. `status` is `11 - Pending` or `66 - Failed`. `PaymentType` examples: `DUITNOW DYNAMIC QR DISPLAY`, `TNG QR DISPLAY`.

- `description` carries the QR payload: render the value after `DecodedQRText:` as the QR image (example `DecodedQRText:https://qr.tngdigital.com.my/a/...`).
- For WeChat Pay Mini Program, `description` holds a JSON object (`appId`, `timeStamp`, `nonceStr`, `package`, `signType`, `paySign`) to pass to `wx.requestPayment`.
- Docs vector for the pending response: `SHA512("SIGNATUREKEY9999;M161-U-999;M161-PO-999999;MERCHANT-REFERENCE-712893;10000;MYR;11 - PENDING")` = `13498ca0...d7a4`.
- The final result is posted to `callbackurl` (same callback fields and `OK` rule as web payments).

## 3. Static DuitNow QR

```text
POST https://api-staging.pay.asia/apim/merchant/GetDuitNowStaticQR
Content-Type: application/json
```

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `token` | String | Yes | "Token from Userlogin". |

Response: `{ "qr": "<EMVCo DuitNow QR string>" }`. Render `qr` as a QR image.

The "Userlogin" call that issues `token` is not documented on any public doc page (checked 2026-10-21). Ask Gkash for it.

## 4. SoftPOS Terminal Payment (remote pay to the Gkash SoftPOS app)

Enable remote payment in the Gkash SoftPOS app settings first.

```text
POST https://api-staging.pay.asia/apim/merchant/RemotePay
Content-Type: application/json
```

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `Amount` | String | Yes | For example `100.00`. |
| `Currency` | String | Yes | `MYR` only. |
| `ReferenceNo` | String | Yes | Unique merchant reference. Special characters: `-`, `_`, `.`. |
| `TerminalId` | String | Yes | For example `M161-TD-999`. |
| `PaymentType` | Int | Yes | See table below. |
| `Email` | String | No | Buyer email. |
| `MobileNo` | String | No | Buyer phone with country code. |
| `PreAuth` | Bool | No | Docs example `false`. (The doc description cell is a copy error reading "The terminal Id of merchant account".) |
| `callbackurl` | String | No | Table name is `callbackurl`; the cURL example sends `CallbackURL`. Confirm the casing with Gkash. |
| `signature` | String | Yes | Family B: `key;Amount;Currency;ReferenceNo;TerminalId`, uppercase hex. |

`PaymentType` (RemotePay):

| Value | Method |
| --- | --- |
| `0` | E-Wallet Scan |
| `1` | Tap-to-Phone |
| `2` | Card Payment (EMV Reader) |
| `3` | MAYBANK QR DISPLAY |
| `4` | GrabPay QR DISPLAY |
| `5` | Touch'N go QR DISPLAY |
| `6` | GKASH QR DISPLAY |
| `7` | Boost QR DISPLAY |
| `8` | Wechat QR DISPLAY |
| `9` | ShopeePay QR DISPLAY |
| `10` | AliPay QR DISPLAY |
| `11` | Atome QR DISPLAY |
| `12` | MCash QR DISPLAY |
| `13` | DuitNow QR DISPLAY |
| `14` | Finexus DuitNow QR DISPLAY |
| `15` | SarawakPay QR DISPLAY |

Response (HTTP 200 JSON): `message` (`Request successful` or `Request failed`), `referenceNo`. This only confirms the request reached the terminal, not payment.

Callback: form POST to `callbackurl` with the same fields as the web payment callback (`CID`, `POID`, `status`, `cartid`, `currency`, `amount`, `signature`, optional `epkey`, `SubscriptionRemID`, `SubscriptionCount`, `description`, `PaymentType`). Verify with the family A verify formula and reply body `OK`.

Cancel an ongoing terminal payment:

```text
POST https://api-staging.pay.asia/apim/merchant/RemoteCancel
Content-Type: application/json
{ "TerminalId": "M161-TD-999", "signature": "<UPPERCASE SHA512 of key;TerminalId>" }
```

Response: `message` (`Request successful` / `Request failed`).

## 5. Soundbox Payment (dynamic DuitNow QR on a Gkash Soundbox)

```text
POST https://api-staging.pay.asia/api/payment/submit
```

The doc header says `Content-Type: application/json`, but the cURL example sends `application/x-www-form-urlencoded` like the other `/api/payment/submit` calls. Confirm with Gkash; the form encoding matches every other submit example.

Request: the merchant presented QR fields (family A signature, lowercase hex) with:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `terminalID` | String | Yes | Soundbox terminal ID. |
| `paymentid` | String | Yes | Only `46119` (DUITNOW QR DISPLAY). |
| `isSoundBox` | Bool | Yes | `true`. |

Response: `message` (`Request successful` / `Request failed`). On success the Soundbox shows the DuitNow QR with the amount. (The doc's example response block shows signature pseudo-code instead of JSON; a doc error.) The result is posted to `callbackurl` with the standard callback fields; verify and reply `OK`.

Cancel:

```text
POST https://api-staging.pay.asia/apim/merchant/SoundboxCancel
Content-Type: application/json
{ "TerminalId": "M161-TD-999", "signature": "<UPPERCASE SHA512 of key;TerminalId>" }
```

Response: `message`.

## 6. Display e-Invoice QR on Soundbox

```text
POST https://api-staging.pay.asia/apim/merchant/PublishInvoice
Content-Type: application/json
```

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `TerminalId` | String | Yes | Soundbox terminal ID. |
| `InvoiceValue` | String | Yes | The e-Invoice QR value (for example a MyInvois URL). |
| `ReferenceNo` | String | Yes | Unique merchant reference. |
| `Signature` | String | Yes | Capital `S`. Family B: `key;ReferenceNo;TerminalId;InvoiceValue`, uppercase hex. |

Documented vector: `SHA512("SIGNATUREKEY9999;merchant-reference-712893;M161-TD-999;https://myeinvois.com.my/#/S_04a3f1f3-d537-47c3-91b7-2688ad27bbcb/O_4db98620-9067-4807-b300")` = `AACAB685B3B0E64AE630DB2EF31163F2CFDB890174C204EC704B875BC0ACF6BCC505479A2A0481F5AE23119517C4A1877D0A4DF65C8A3261059BA95E1D4B3409`.

Response: `message` (`Request successful` / `Request failed`), `referenceNo`.

## Local SoftPOS Integrations (official GitHub, not cloud APIs)

Gkash also publishes local or on-device integrations that talk to the SoftPOS app, not to `api.gkash.my`. They use a client certificate (`.pfx`) and SoftPOS terminal login. Listed for completeness; read the repo when a merchant needs them:

- `https://github.com/gkashmy/gkash-softpos-local-web-api`: .NET 6 local web API on `http://localhost:5010` with `POST /api/Gkash/Login`, `POST /api/Gkash/RequestPayment` (`Amount`, `TerminalId`, `Email`, `MobileNo`, `ReferenceNo`, `PaymentType` 0 to 13, `PreAuth`, `CallbackURL`), `GET /api/Gkash/CancelTransaction?TerminalId=`, `GET /api/Gkash/QueryStatus?ReferenceNo=`, `GET /api/Gkash/QueryCardAndDuitNowStatus?ReferenceNo=`. For DuitNow QR it appends `-QR` to the reference number.
- `https://github.com/gkashmy/gkash-client-tcp-softpos-android`: Android SDK `io.github.gkashmy:gkash-softpos-sdk` (TCP to SoftPOS).
- `https://github.com/gkashmy/gkash-client-tcp-softpos-windows`: Windows Forms sample using the `ClientSocketConnection` NuGet package; requires "Server mode" in Gkash Business settings.
- `https://github.com/gkashmy/client-socket-connection`: socket library (no README).
