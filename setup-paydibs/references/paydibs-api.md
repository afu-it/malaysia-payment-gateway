# Paydibs API Reference

Use this reference for factual Paydibs Merchant Integration API (v3) guidance. Docs revision checked: version 4.0, 24 Sep 2026 (checked 2026-10-21). Verify environment URLs, enabled payment methods, settlement and fees with Paydibs for the specific merchant account.

## Source URLs

The docs site is a JavaScript single-page app. Page routes (all under `https://v3api-docs.paydibs.com`):

- `/docs/introduction`
- `/docs/revision`
- `/docs/merchant-api-reference/pay-request`
- `/docs/merchant-api-reference/pay-response`
- `/docs/merchant-api-reference/query-request`
- `/docs/merchant-api-reference/query-response`
- `/docs/merchant-api-reference/refund-request`
- `/docs/merchant-api-reference/refund-response`
- `/docs/merchant-api-reference/bind-option-request`, `/bind-option-response`
- `/docs/merchant-api-reference/revoke-token-request`, `/revoke-token-response`
- `/docs/merchant-api-reference/in-store-pay-request`, `/in-store-pay-response`
- `/docs/sign-hashing/merchant-request-sign`
- `/docs/sign-hashing/merchant-response-sign`
- `/docs/hosted-session/initiate`, `/integration-guide`, `/api-reference`
- `/docs/recurring/initiation`, `/subsequent-payments`, `/billing-cycle-rules`
- `/docs/appendices`
- Official SDK file: `https://v3api-docs.paydibs.com/HOSTED_SESSION_SDK/sdk/v1/sdk.js`

## Integration Surfaces

- Hosted checkout: merchant posts a PAY form to Paydibs; customer picks a method on the Paydibs payment page (card, online banking, e-wallet, BNPL).
- Hosted Payment Session SDK: card fields in a Paydibs iframe on the merchant page, then a PAY request with `PaySessId`.
- Recurring: first payment is a customer-initiated PAY (CIT) with recurring fields; later charges are merchant-initiated `MIT` calls, or Paydibs-scheduled by rule/fixed date.
- Saved card token (`TokenType=CARD`), token revoke (`REVOKE`), linked accounts/cards (`BINDOPT`), TNG AutoDebit (`PaymentOption=TNGAD`).
- E-Mandate and Direct Debit (`/PPG/trigger-emandate`, `/PPG/create-ddi`, `/PPG/create-ddi-txn`, `/PPG/cancel-ddi`), added in docs v4.0. Read the e-mandate pages before using.
- In-store DuitNow QR (`/PPG/offline/trigger-qr`).

Default path for a website checkout: PAY form post + `MerchantCallbackURL` + QUERY fallback.

## Companion Reference Files

- `paydibs-codes.md`: every enum and code list (TxnType, PTxnStatus, PAYExists, currencies, method codes, E-KYC result, recurring and e-mandate enums, DDI error codes, SDK error codes, data types, special characters, revision history).
- `paydibs-recurring.md`: CIT fields, billing cycle rules, MIT request/response.
- `paydibs-hosted-session.md`: create-pay-session API and the full card SDK reference.
- `paydibs-tokens-instore.md`: BINDOPT, REVOKE, saved card tokens, in-store DuitNow QR.
- `paydibs-emandate.md`: E-Mandate enrolment/maintenance/termination and Direct Debit instructions.

## Endpoint Catalogue

Every operation in the published docs (31 pages, version 4.0). Host = payment URL issued by Paydibs.

| # | Operation | Method | Path | TxnType / body | Response channel | Details |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Payment (hosted checkout) | GET or POST (browser form) | `/PPG/trigger` | `PAY` | `MerchantRURL` form post + `MerchantCallbackURL` JSON | This file |
| 2 | Payment with hosted session | POST (browser form) | `/PPG/trigger` | `PAY` + `PaySessId` | As above | `paydibs-hosted-session.md` |
| 3 | Recurring first payment (CIT) | POST (browser form) | `/PPG/trigger` | `PAY` + recurring fields | As above, plus `RcurId` | `paydibs-recurring.md` |
| 4 | Recurring subsequent charge (MIT) | Server call | `/PPG/trigger` | `MIT` | JSON same session + callback | `paydibs-recurring.md` |
| 5 | Status inquiry | POST, server | `/PPG/trigger` | `QUERY` | JSON same session | This file |
| 6 | Refund | POST, server | `/PPG/refund` | `REFUND` | JSON same session | This file |
| 7 | Retrieve linked accounts/cards | POST, server | `/PPG/trigger` | `BINDOPT` | JSON same session | `paydibs-tokens-instore.md` |
| 8 | Revoke AutoDebit or card token | POST, server | `/PPG/trigger` | `REVOKE` | JSON same session | `paydibs-tokens-instore.md` |
| 9 | Create hosted payment session | POST JSON, server | `/PPG/create-pay-session` | - | JSON envelope | `paydibs-hosted-session.md` |
| 10 | In-store DuitNow QR | POST | `/PPG/offline/trigger-qr` | `PAY` + `AcqBank=DNQR` | JSON envelope with QR content | `paydibs-tokens-instore.md` |
| 11 | E-Mandate enrolment | POST | `/PPG/trigger-emandate` | `EMDT_CRT` | RURL + callback | `paydibs-emandate.md` |
| 12 | E-Mandate maintenance | POST | `/PPG/trigger-emandate` | `EMDT_MTN` | RURL + callback | `paydibs-emandate.md` |
| 13 | E-Mandate termination | POST | `/PPG/trigger-emandate` | `EMDT_TMN` | RURL + callback | `paydibs-emandate.md` |
| 14 | Create direct debit instruction | POST JSON | `/PPG/create-ddi` | - | JSON envelope | `paydibs-emandate.md` |
| 15 | Create DDI transaction (flexible) | POST JSON | `/PPG/create-ddi-txn` | - | JSON envelope | `paydibs-emandate.md` |
| 16 | Cancel direct debit instruction | POST JSON | `/PPG/cancel-ddi` | - | JSON envelope | `paydibs-emandate.md` |
| 17 | Hosted session iframe (loaded by sdk.js) | - | `/PPG/hosted-payment-iframe` on `stgapi`/`v3api` | - | postMessage | `paydibs-hosted-session.md` |

Not published: void, capture, cancel/void of a card authorisation, partial refund, settlement report and payout APIs (checked 2026-10-21). Terminal API routes (callback, void, settlement) exist in the docs bundle but are not in the navigation; treat them as unavailable.

## Environments

- Introduction 1.1: "A test account is needed for merchant integration. Upon account generated, Merchant ID, Merchant Password and payment URL will be provided."
- Introduction 1.5: "Production URL will be provided via email upon live payment account created."
- Every request sample uses `action="<URL provided by Paydibs>"` plus a path such as `/PPG/trigger`.
- The official Hosted Session SDK (`sdk.js`) maps `environment` to hosts: `stg` = `https://stgapi.paydibs.com`, `prod` = `https://v3api.paydibs.com`. A recurring sample also shows `https://stgapi.paydibs.com/PPGSIM/...`.
- Use the URL Paydibs issues for the account as the source of truth. Store it as `PAYDIBS_BASE_URL` per environment.
- TLS: production allows TLS 1.2 only. Query/refund pages say the test environment is "not limited"; the PAY and hosted-session pages say both allow only TLS 1.2. Use TLS 1.2+ everywhere and test it against production before go-live.

## Authentication

There is no API key header. Every request carries `MerchantID` (6 chars, AN) and a `Sign` computed with the secret Merchant Password. The Password never travels in a request.

## Data Types

`A` alpha, `AN` alphanumeric, `ANS` alphanumeric + special characters, `ANS (with space)`, `N` numeric, `B` boolean, `L` list, `DateTime` ISO 8601. Allowed special characters are listed in Introduction 1.3. For E-KYC fields, `EkycBuyerId` does not support `/` or `&` (full lists in `paydibs-codes.md`).

## Amount Rules

- `MerchantTxnAmt` is `N 12(2)`: major units (ringgit) with exactly 2 decimals, no comma. Valid `1000.00`. Invalid `1,000.00`, `1000`.
- Zero-decimal currencies (VND, KRW, IDR, KHR) still send 2 decimals, always `.00`.
- `MerchantCurrCode`: ISO 4217 3-letter. Listed: MYR, SGD, PHP, VND, CNY, HKD, IDR, KHR, KRW, THB.

## PAY Request (hosted checkout)

- URL: `<payment URL>/PPG/trigger`. Method: HTTP GET or POST (docs). Use a browser form POST.

Mandatory fields:

| Field | Type | Max | Notes |
| --- | --- | --- | --- |
| `TxnType` | A | 7 | `PAY`, uppercase |
| `MerchantID` | AN | 6 | From Paydibs |
| `MerchantPymtID` | ANS (space) | 20 | Unique per payment attempt. Duplicates rejected |
| `MerchantOrdID` | ANS (space) | 20 | Order/invoice no. May repeat across attempts; may equal `MerchantPymtID` |
| `MerchantOrdDesc` | ANS (space) | 100 | No Mandarin; special chars trimmed or replaced |
| `MerchantTxnAmt` | N | 12(2) | See Amount Rules |
| `MerchantCurrCode` | A | 3 | ISO 4217 |
| `MerchantRURL` | ANS | 255 | Browser redirect URL. Replace `&` with `;` |
| `CustIP` | ANS | 40 | Customer IP captured by merchant |
| `CustName` | ANS (space) | 50 | Special chars trimmed or replaced |
| `CustEmail` | ANS | 60 | Special chars trimmed or replaced |
| `CustPhone` | ANS | 25 | |
| `MerchantCallbackURL` | ANS | 255 | Server-to-server URL. Replace `&` with `;`. Listed under mandatory, though the Sign note treats it as optional. Always send it |
| `Sign` | ANS | 128 | SHA512 hex, see Sign section |

Optional: `MerchantName` (25, shown on payment page), `Method` (A 3, preselect `OB`/`CC`/`WA`), `CustReferenceId` (100, required to save a card), `TokenType` (A 4, `CARD`), `Token` (AN 50, saved card token from a previous response; required if `TokenType` set), `PageTimeout` (N 4, seconds left; the page disables inputs at timeout).

Conditional:

- E-KYC (FPX only, when enabled on the channel): `EkycAccountNumber` (40) and/or `EkycBuyerId` (40).
- Cash on Delivery channel: `ReceiverName`, `ReceiverPhoneNumber`, `ReceiverAddress1`, `ReceiverAddress2`, `ReceiverCity`, `ReceiverState`, `ReceiverCountry` (`MY`), `ReceiverPostcode`, `ReceiverAddressType` (`HME`/`OFC`/`OTH`), `ParcelWeight` (kg, max 30, default 1), `ParcelLength`/`ParcelWidth`/`ParcelHeight` (cm, default 1), `ItemCount` (default 1), `ItemDescription`.
- Hosted Payment Session: `PaySessId` (36, UUID from `/PPG/create-pay-session`).
- Recurring CIT: see Recurring section.

Example (from docs):

```html
<form name="frmTestPay" method="post" action="<PAYDIBS_BASE_URL>/PPG/trigger">
  <input type="hidden" name="TxnType" value="PAY">
  <input type="hidden" name="MerchantID" value="<MERCHANT_ID>">
  <input type="hidden" name="MerchantPymtID" value="Test000123">
  <input type="hidden" name="MerchantOrdID" value="Order 123">
  <input type="hidden" name="MerchantOrdDesc" value="Payment for abc">
  <input type="hidden" name="MerchantTxnAmt" value="10.99">
  <input type="hidden" name="MerchantCurrCode" value="MYR">
  <input type="hidden" name="MerchantRURL" value="https://merchant.example/response">
  <input type="hidden" name="CustIP" value="123.123.123.10">
  <input type="hidden" name="CustName" value="Test">
  <input type="hidden" name="CustEmail" value="test@testmail.com">
  <input type="hidden" name="CustPhone" value="60123334567">
  <input type="hidden" name="Sign" value="<SHA512 hex>">
  <input type="hidden" name="MerchantCallbackURL" value="https://merchant.example/callback">
</form>
```

## PAY Response (MerchantRURL and MerchantCallbackURL)

Mandatory: `TxnType`, `Method` (`OB`/`CC`/`WA`), `MerchantID`, `MerchantPymtID`, `MerchantOrdID`, `MerchantTxnAmt`, `MerchantCurrCode` (all echoed), `PTxnID` (AN 30, Paydibs transaction ID), `PTxnStatus` (A 4), `Sign`.
Optional: `PTxnMsg` (255), `AuthCode` (12, bank auth code if any), `BankRefNo` (20), `CustReferenceId`, `TokenType`, `Token` (returned when `CustReferenceId` given and customer saves card), `MaskedPan` (first 6 + last 4, e.g. `512345XXXXXX0008`).
Conditional: `EkycResult` (FPX, e.g. `020100`), `RcurId` (recurring only).

| Field | Type | Max | Section | Notes |
| --- | --- | --- | --- | --- |
| `TxnType` | A | 7 | Mandatory | Echo (`PAY`) |
| `Method` | A | 3 | Mandatory | `OB`, `CC`, `WA` |
| `MerchantID` | AN | 6 | Mandatory | Echo |
| `MerchantPymtID` | ANS (space) | 20 | Mandatory | Echo |
| `MerchantOrdID` | ANS (space) | 20 | Mandatory | Echo |
| `MerchantTxnAmt` | N | 12(2) | Mandatory | Echo |
| `MerchantCurrCode` | A | 3 | Mandatory | Echo |
| `PTxnID` | AN | 30 | Mandatory | Paydibs transaction ID |
| `PTxnStatus` | A | 4 | Mandatory | See PTxnStatus Codes |
| `Sign` | ANS | 128 | Mandatory | Response Sign |
| `EkycResult` | ANS | 50 | Conditional | FPX E-KYC result, decode in `paydibs-codes.md` |
| `RcurId` | AN | 200 | Conditional | Recurring only |
| `PTxnMsg` | AN | 255 | Optional | Explains `PTxnStatus` |
| `AuthCode` | AN | 12 | Optional | Bank authorisation code |
| `BankRefNo` | AN | 20 | Optional | Bank reference |
| `CustReferenceId` | ANS (space) | 100 | Optional | Echo |
| `TokenType` | A | 4 | Optional | `CARD` |
| `Token` | AN | 50 | Optional | Saved card token |
| `MaskedPan` | AN | 19 | Optional | First 6 + last 4 |

Delivery:

- `MerchantRURL`: browser form POST with hidden inputs. Customer-facing only.
- `MerchantCallbackURL`: server-to-server. "Sent with a content type of `application/x-www-form-urlencoded`, and the request body will contain a JSON-formatted string." The merchant must return HTTP 200; otherwise retried up to 5 times. Read the raw body and JSON-parse it; be tolerant if a framework exposes it as a single form key.

Callback body sample (docs):

```json
{"TxnType":"PAY","Method":"OB","MerchantID":"OM","MerchantPymtID":"TestSG20180430009","MerchantOrdID":"OMTest","MerchantTxnAmt":"10.00","MerchantCurrCode":"MYR","PTxnID":"ABC0000TestSG20180430009","PTxnStatus":"0","PTxnMsg":"Success","BankRefNo":"1804301738070889","Sign":"ab0678...8b9e"}
```

## PTxnStatus Codes (Appendices)

| Code | Meaning |
| --- | --- |
| `0` | Payment successful |
| `1` | Payment failed |
| `2` | Payment pending |
| `9` | Payment voided |
| `10` | Refunded |
| `17` | Cancelled at payment page |
| `18` | Payment cancelled |
| `19` | Expired |
| `20` | Chargeback |
| `22` | Acquirer bank under maintenance |
| `50` | Out of per-transaction amount limit |
| `51` | Amount lower than minimum limit |
| `-1` | Transaction not exists/not found |
| `-2` | Internal system error |

Settle only on `0`. Keep `2` open and requery. Treat `9`, `10`, `20` as reversal events on an already-paid order.

## Sign (SHA512)

Rule: concatenate the listed values in order with no separators, Merchant Password first, hash with SHA512, send as a hex string (128 chars). Optional fields not sent are empty strings. `Token` goes in clear form, not URL-encoded.

Request formulas:

| Request | String to sign |
| --- | --- |
| PAY | `MerchantPassword + TxnType + MerchantID + MerchantPymtID + MerchantOrdID + MerchantRURL + MerchantTxnAmt + MerchantCurrCode + CustIP + PageTimeout + MerchantCallbackURL + Token` |
| QUERY | `MerchantPassword + MerchantID + MerchantPymtID + MerchantTxnAmt + MerchantCurrCode` |
| REFUND | `MerchantPassword + MerchantID + MerchantPymtID + MerchantTxnAmt + MerchantCurrCode` |
| MIT | `MerchantPassword + TxnType + MerchantID + MerchantPymtID + MerchantOrdID + MerchantTxnAmt + MerchantCurrCode + RcurId` |
| Hosted session | `MerchantPassword + MerchantID + MerchantPymtID + MerchantTxnAmt + MerchantCurrCode + CustReferenceId` |
| REVOKE / BINDOPT (AutoDebit) | `MerchantPassword + MerchantID + CustName + CustPhone + CustEmail + PaymentOption` |
| REVOKE / BINDOPT (Card) | `MerchantPassword + MerchantID + CustReferenceId + TokenType + Token` |
| In-store DuitNow QR PAY | `MerchantPassword + TxnType + MerchantID + MerchantPymtID + MerchantTxnAmt + MerchantCurrCode` |

Response formulas:

| Response | String to sign |
| --- | --- |
| PAY response / QUERY response | `MerchantPassword + MerchantID + MerchantPymtID + PTxnID + MerchantOrdID + MerchantTxnAmt + MerchantCurrCode + PTxnStatus + AuthCode` |
| REFUND response | `MerchantPassword + MerchantID + MerchantPymtID + MerchantTxnAmt + MerchantCurrCode + RefundResult + RefundDesc + RefundID` |
| MIT response | `MerchantPassword + MerchantID + MerchantPymtID + MitDesc + RecurringID` (skip MitDesc if absent) |
| Hosted session response | `MerchantPassword + MerchantID + MerchantPymtID + MerchantTxnAmt + MerchantCurrCode + PaySessID + PaySessExpTm` |
| Revoke response | `MerchantPassword + MerchantID + RevokeResult + RevokeDesc` |
| Bind option response | `MerchantPassword + MerchantID + BindOptResult + BindOptDesc` |

Verified doc examples (reproduced with SHA512 over UTF-8, lowercase hex, on 2026-10-21):

- Response: `abc123` + `Paydibs` + `INV20180420001` + `Paydibs000000101` + `Order12345678` + `10.99` + `MYR` + `0` + `339900` gives `387e0d6b2651a21c...dc74735` (docs example uses the literal sample MerchantID "Paydibs").
- MIT request: `abc123MITC001TEST1TESTORD110.00MYRAGID100` gives `a0d4000a7593751a...7e08ff6`.
- Hosted session request: `C7(7B3N8MIMEONERtest0525051.00MYRTEST0524CCE` gives `b497585e9652dcb1...6078f28`.
- Also reproduced (MerchantID `Paydibs`, password `abc123`): QUERY request, REFUND request (same string as QUERY), REVOKE/BINDOPT AutoDebit and Card requests, in-store QR request, REVOKE/BINDOPT response (`abc123PaydibstrueSuccess`), REFUND response (`...MYRtrueSuccessRF20250910001`), MIT response (`abc123PaydibsINV20180420001ABCAGID123`).
- The docs' PAY request example hash did not reproduce from its shown source string; follow the formula, and test against staging.

Pseudo-code:

```js
import { createHash, timingSafeEqual } from "node:crypto";
const sha512 = (s) => createHash("sha512").update(s, "utf8").digest("hex");
const v = (x) => (x === undefined || x === null ? "" : String(x));

function payRequestSign(pw, f) {
  return sha512(pw + "PAY" + f.MerchantID + f.MerchantPymtID + f.MerchantOrdID +
    f.MerchantRURL + f.MerchantTxnAmt + f.MerchantCurrCode + f.CustIP +
    v(f.PageTimeout) + v(f.MerchantCallbackURL) + v(f.Token));
}

function verifyPayResponse(pw, r) {
  const expected = sha512(pw + r.MerchantID + r.MerchantPymtID + v(r.PTxnID) +
    r.MerchantOrdID + r.MerchantTxnAmt + r.MerchantCurrCode + r.PTxnStatus + v(r.AuthCode));
  const a = Buffer.from(expected), b = Buffer.from(String(r.Sign || "").toLowerCase());
  return a.length === b.length && timingSafeEqual(a, b);
}
```

Sign over values exactly as sent and received (the `MerchantRURL` with `;` in place of `&`). How `null` JSON values (e.g. `AuthCode: null`) enter the string is not documented; treating them as empty strings is an assumption to confirm on staging.

## QUERY (status inquiry)

- URL: `<payment URL>/PPG/trigger`, server-side POST, `application/x-www-form-urlencoded` (C# sample). Response returns in the same HTTP session as JSON.
- Request: `TxnType=QUERY`, `MerchantID`, `MerchantPymtID`, `MerchantTxnAmt`, `MerchantCurrCode` (same as PAY), `Sign` (QUERY formula). The C# sample shows `Amount=`; the field table says `MerchantTxnAmt`. Use the table.

| Request field | Type | Max | Notes |
| --- | --- | --- | --- |
| `TxnType` | A | 7 | `QUERY` |
| `MerchantID` | AN | 6 | |
| `MerchantPymtID` | ANS (space) | 20 | Of the original PAY |
| `MerchantTxnAmt` | N | 12(2) | As submitted in PAY |
| `MerchantCurrCode` | A | 3 | As submitted in PAY |
| `Sign` | ANS | 128 | QUERY request Sign |

| Response field | Type | Max | Section |
| --- | --- | --- | --- |
| `PAYExists` | N | 1 | Mandatory |
| `MerchantID` | AN | 6 | Mandatory |
| `MerchantPymtID` | ANS (space) | 20 | Mandatory |
| `MerchantOrdID` | ANS (space) | 20 | Mandatory |
| `MerchantTxnAmt` | N | 12(2) | Mandatory |
| `MerchantCurrCode` | A | 3 | Mandatory |
| `PTxnStatus` | A | 4 | Mandatory |
| `Sign` | ANS | 128 | Mandatory |
| `QueryDesc` | AN | 255 | Optional |
| `TxnType` | A | 7 | Optional (`QUERY`) |
| `Method` | A | 3 | Optional (empty when not found) |
| `PTxnID` | AN | 30 | Optional (used in the Sign; empty if absent) |
| `PTxnMsg` | AN | 255 | Optional |
| `BankRefNo` | AN | 20 | Optional |
| `AuthCode` | AN | 12 | Optional |
| `EkycResult` | ANS | 50 | Conditional (FPX E-KYC) |
- Response: PAY response fields plus `PAYExists` (`0` exists, `1` does not exist, `2` processing error, retry), `QueryDesc`, `PTxnMsg`, `BankRefNo`, `AuthCode`, optional `EkycResult`. Not found returns `PTxnStatus` `-1`; bad hash returns `PAYExists` `2`, `QueryDesc` "Hash value mismatched.", `PTxnStatus` `-2`.
- Verify the response Sign with the PAY/QUERY response formula.

## REFUND

- URL: `<payment URL>/PPG/refund`, Method POST, server-side. Response in the same session as JSON.
- Request: `TxnType=REFUND`, `MerchantID`, `MerchantPymtID` (original), `MerchantTxnAmt` and `MerchantCurrCode` (as in the PAY request), `RefundReason` (mandatory per field table, 1000), `Sign` (REFUND formula); optional `RefundRemarks` (1000).
- Response: echoed fields, `RefundResult` (boolean `true`/`false`), `Sign`; plus `RefundDesc` (255), `RefundID` (50, null on failure).

| Request field | Type | Max | Section |
| --- | --- | --- | --- |
| `TxnType` | A | 7 | Mandatory (`REFUND`) |
| `MerchantID` | AN | 6 | Mandatory |
| `MerchantPymtID` | ANS (space) | 20 | Mandatory |
| `MerchantTxnAmt` | N | 12(2) | Mandatory |
| `MerchantCurrCode` | A | 3 | Mandatory |
| `RefundReason` | ANS (space) | 1000 | Mandatory |
| `Sign` | ANS | 128 | Mandatory |
| `RefundRemarks` | ANS (space) | 1000 | Optional |

| Response field | Type | Max | Section |
| --- | --- | --- | --- |
| `TxnType` | A | 7 | Mandatory |
| `MerchantID` | AN | 6 | Mandatory |
| `MerchantPymtID` | ANS (space) | 20 | Mandatory |
| `MerchantTxnAmt` | N | 12(2) | Mandatory |
| `MerchantCurrCode` | A | 3 | Mandatory |
| `RefundResult` | B | 5 | Mandatory |
| `Sign` | ANS | 128 | Mandatory |
| `RefundDesc` | ANS (space) | 255 | Optional |
| `RefundID` | AN | 50 | Optional |

Refund response Sign uses `RefundResult` as the text `true`/`false` and empty strings for absent `RefundDesc`/`RefundID` (null handling is an assumption; confirm on staging).
- Sample failure descriptions: "Refund is only allowed from the next day onwards.", "Transaction not found", "Hash value mismatched."
- Partial refunds: Not documented in official docs (checked 2026-10-21). The request has no separate refund amount field.
- Refund is submitted, not instantly final; requery later (`PTxnStatus` `10` = refunded).

## Hosted Payment Session (card SDK)

Full field tables and the SDK reference: `paydibs-hosted-session.md`.


1. Server: `POST <payment URL>/PPG/create-pay-session` with JSON: `MerchantID`, `MerchantPymtID`, `MerchantTxnAmt`, `MerchantCurrCode`, `Sign` (hosted session formula); optional `RcurIdc` (boolean; if true `CustReferenceId` required), `CustReferenceId`, `PageTimeout` (default 600 s). Values must match the later PAY request.
2. Response: `{ suc, code, msg, data: { MerchantID, MerchantPymtID, MerchantTxnAmt, MerchantCurrCode, PaySessID, PaySessExpTm, Sign, RcurIdc?, PaySessFailMsg }, traceId, timeStamp }`. Verify `data.Sign`.
3. Browser: load `sdk.js` (download from docs and self-host over HTTPS), `HostedSession.configure({ environment: 'stg'|'prod', merId, paySessId })`, `mount('#card-fields')`, await `ready`, then `updateSession()`. Status values: `ok`, `fields_in_error`, `session_invalid`, `session_expired`, `unsupported`, `system_error`.
4. On `ok`, send the PAY request with the same `PaySessId`. Do not resend `CustReferenceId`/`RcurIdc` in PAY; they are stored in the session.

## Recurring

Full CIT/MIT field tables and billing rules: `paydibs-recurring.md`.


- CIT: normal PAY plus recurring fields `RcurIdc=true`, `CustReferenceId`, `Mode` (`FLX` flexible / `FIX` fixed), `Src` (`MER` merchant / `PG` Paydibs schedules), `MaxCyc`, `Bhv` (`NONE` verify only / `IMDT` charge now), `BillCycTyp` (`FD` fixed dates / `RL` rule), `Freq` (`HR`/`WKLY`/`MTLY`/`YRLY`), `Intrvl`, `StDt`, `EndDt`, `DyOfMth`, `WkDy`, `MthOfYrs`, `RcurAmt`, `FixDt`. Successful payment returns `RcurId`; store it.
- MIT: server-side `TxnType=MIT` to `/PPG/trigger` with `MerchantID`, `MerchantPymtID`, `MerchantOrdID`, `MerchantOrdDesc`, `MerchantTxnAmt`, `MerchantCurrCode`, `MerchantCallbackURL`, `RcurId`, `Sign` (MIT formula). JSON response `MITSuccess` true means accepted by acquirer only; the merchant must QUERY for the final result, and the callback also delivers it.
- When `Src=PG`, Paydibs runs later charges by the stored rule; the merchant does not send MIT.

## Not Documented (checked 2026-10-21)

- Public sandbox host, test cards, test FPX banks: not documented in v3 docs (revision 2.1 once mentioned a card test card; it is not in the current pages). Ask Paydibs.
- Callback source IPs or IP allowlist: not documented.
- Rate limits: not documented.
- Callback retry interval: not documented (only "up to 5 times").
- Null-field handling inside Sign strings: not documented.

## Common Errors

- "Hash value mismatched.": wrong field order, wrong Password/environment, amount not `x.xx`, encoded vs clear values, or `&` vs `;` mismatch in URLs.
- Duplicate `MerchantPymtID`: rejected. Generate a new one per attempt.
- Callback retried repeatedly: endpoint did not return HTTP 200.
- Hosted session `session_expired`: create a new session (default 600 s life).
