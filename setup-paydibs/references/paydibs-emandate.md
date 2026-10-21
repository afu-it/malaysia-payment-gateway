# Paydibs E-Mandate And Direct Debit

Added in docs version 4.0 (24 Sep 2026). Checked 2026-10-21. Sources:

- Enrolment: `https://v3api-docs.paydibs.com/docs/e-mandate-feature/enrolment`
- Maintenance: `https://v3api-docs.paydibs.com/docs/e-mandate-feature/maintenance`
- Termination: `https://v3api-docs.paydibs.com/docs/e-mandate-feature/termination`
- Create DDI: `https://v3api-docs.paydibs.com/docs/e-mandate-feature/create-direct-debit-instruction`
- Create DDI transaction: `https://v3api-docs.paydibs.com/docs/e-mandate-feature/create-direct-debit-instruction-transaction`
- Cancel DDI: `https://v3api-docs.paydibs.com/docs/e-mandate-feature/cancel-direct-debit-instruction`
- E-Mandate Sign: `https://v3api-docs.paydibs.com/docs/sign-hashing/e-mandate-sign`
- Direct Debit Sign: `https://v3api-docs.paydibs.com/docs/sign-hashing/direct-debit-sign`

Codes (`CustIdTyp`, `DbtFreq`, e-mandate `Status`, DDI error codes) are in `paydibs-codes.md`.

## How The Pieces Fit

1. Enrol the customer's bank mandate (`EMDT_CRT`). The customer is sent to `MerchantRURL` afterwards and the result also goes to `MerchantCallbackURL`. Your `PymRefNum` identifies the mandate.
2. Optionally change limits or contact data (`EMDT_MTN`) or end the mandate (`EMDT_TMN`).
3. Create a direct debit instruction (DDI) against the mandate with `/PPG/create-ddi`: `Mode=RL` (Paydibs debits on a schedule) or `Mode=FLX` (you trigger each debit). Store the returned `DirectDebitID`.
4. For `FLX`, trigger each debit with `/PPG/create-ddi-txn`.
5. Cancel a DDI with `/PPG/cancel-ddi`: future scheduled and pending DDI transactions are cancelled; processed ones remain.

Every DDI must comply with its mandate: amount not above `MaxDbtAmt`, matching frequency, cycle count within `MaxDbtFreq`, and the mandate active and valid. Non-compliant instructions may be rejected (codes 90175 to 90188).

Transport: all six endpoints are `POST`; samples are JSON bodies. The docs do not say whether e-mandate requests are a browser form post or a server call. Because they carry `MerchantRURL` (browser redirect after the customer completes enrolment at the bank), treat enrolment as browser-facing and confirm the exact content type with Paydibs. The three DDI endpoints return a JSON envelope `{ suc, code, msg, data, traceId, timeStamp }` in the same HTTP session.

Callback: `MerchantCallbackURL` must return HTTP 200; otherwise it is retried up to 5 times. Replace `&` with `;` in both URLs.

## E-Mandate Enrolment (`EMDT_CRT`)

URL: `<payment URL>/PPG/trigger-emandate`, POST.

Request, mandatory:

| Field | Type | Max | Notes |
| --- | --- | --- | --- |
| `TxnType` | A | 8 | `EMDT_CRT` |
| `MerchantID` | AN | 6 | From Paydibs |
| `PymRefNum` | ANS (space) | 20 | Unique merchant reference for this e-mandate |
| `CustName` | ANS (space) | 50 | Full name as per ID document |
| `CustIdTyp` | AN | 1 | ID type code |
| `CustIdNum` | ANS (space) | 20 | ID number |
| `MaxDbtAmt` | N | 12(2) | Max MYR debit amount, `1000.00` format |
| `DbtFreq` | AN | 4 | `DL`/`WK`/`MT`/`YR` |
| `MaxDbtFreq` | N | 3 | Maximum debit frequency (count) |
| `PrpOfPym` | ANS (space) | 27 | Purpose of payment |
| `MerchantRURL` | ANS | 255 | Browser redirect after enrolment |
| `MerchantCallbackURL` | ANS | 255 | Server-to-server result |
| `Sign` | ANS | 128 | Enrolment Sign |

Optional: `EndDt` (ANS 10, ISO 8601 date after today, e.g. `2027-08-31`).
Optional, editable by the customer if left blank: `CustPhone` (N 11), `CustEmail` (ANS 27).

Sample request (docs):

```json
{
  "TxnType": "EMDT_CRT", "MerchantID": "PAYDIBS", "PymRefNum": "EMDT202607220001",
  "CustName": "Wong Chew Shin", "CustIdTyp": "1", "CustIdNum": "000102089281",
  "MaxDbtAmt": "100.00", "DbtFreq": "MT", "MaxDbtFreq": 12, "EndDt": "2027-08-31",
  "PrpOfPym": "Loan Repayment",
  "MerchantRURL": "https://MerchantTestEnv.com/response.aspx",
  "MerchantCallbackURL": "http://MerchantTestEnv.com/callbackresponse.aspx",
  "Sign": "93e9a77b...b7588408"
}
```

## E-Mandate Maintenance (`EMDT_MTN`)

URL: `<payment URL>/PPG/trigger-emandate`, POST.

Mandatory: `TxnType` (`EMDT_MTN`), `MerchantID`, `PymRefNum`, `MerchantRURL`, `MerchantCallbackURL`, `Sign` (Maintenance Sign).
Optional, only populate to modify: `MaxDbtAmt` (N 12(2)), `DbtFreq` (AN 4), `MaxDbtFreq` (N 3).
Optional, editable by the customer if left blank: `CustPhone` (N 11), `CustEmail` (ANS 27).

## E-Mandate Termination (`EMDT_TMN`)

URL: `<payment URL>/PPG/trigger-emandate`, POST.

Mandatory: `TxnType` (`EMDT_TMN`), `MerchantID`, `PymRefNum`, `TmnRsn` (ANS (space) 255, termination reason), `MerchantRURL`, `MerchantCallbackURL`, `Sign` (Termination Sign).

## E-Mandate Response (redirect and callback, all three types)

| Field | Type | Max | Mandatory | Notes |
| --- | --- | --- | --- | --- |
| `TxnType` | A | 8 | Yes | Echo |
| `MerchantID` | AN | 6 | Yes | Echo |
| `PymRefNum` | ANS (space) | 20 | Yes | Echo |
| `MaxDbtAmt` | N | 12(2) | Yes | Current mandate value |
| `DbtFreq` | AN | 4 | Yes | |
| `MaxDbtFreq` | N | 3 | Yes | Sample sends it as a string |
| `PrpOfPym` | ANS (space) | 27 | Yes | |
| `TmnRsn` | ANS (space) | 255 | Termination only | |
| `Status` | A | 4 | Yes | E-mandate status code, `0` = success |
| `Sign` | ANS | 128 | Yes | E-Mandate Response Sign |
| `StatusMsg` | AN | 255 | No | Sample: `SBI Bank A\|MS.N@m3()/PyN . - &B\|Approved\|00` (pipe-delimited bank detail) |

## Create Direct Debit Instruction

URL: `<payment URL>/PPG/create-ddi`, POST.

Request:

| Field | Type | Max | Required | Notes |
| --- | --- | --- | --- | --- |
| `MerchantID` | AN | 6 | Yes | |
| `Sign` | ANS | 128 | Yes | DDI Create Request Sign |
| `Mode` | A | 3 | Yes | `RL` rule-based, `FLX` flexible |
| `PymRefNum` | ANS (space) | 20 | Yes | The e-mandate `PymRefNum` to link |
| `MerchantTxnAmt` | N | 12(2) | If `Mode=RL` | Debit amount |
| `MerchantCurrCode` | A | 3 | If `Mode=RL` | |
| `Freq` | AN | 4 | If `Mode=RL` | `DL`/`WK`/`MT`/`YR` |
| `StDt` | ANS | 10 | If `Mode=RL` | `yyyy-MM-dd`, must be after today |
| `EndDt` | ANS | 10 | If `Mode=RL` | `yyyy-MM-dd`; debit still valid on the end date |
| `MaxCyc` | N | 10 | No | Maximum debit count |

Response `data`:

| Field | Type | Max | Mandatory | Notes |
| --- | --- | --- | --- | --- |
| `MerchantID` | AN | 6 | Yes | |
| `DirectDebitID` | AN | 20 | Yes | Assigned by Paydibs; needed for FLX transactions and cancellation |
| `Status` | A | 10 | Yes | `Success` / `Failed` |
| `Sign` | ANS | 128 | Yes | DDI Create Response Sign |
| `Mode` | A | 3 | Yes | |
| `PymRefNum` | ANS (space) | 20 | Yes | |
| `MerchantTxnAmt` | N | 12(2) | If `Mode=RL` | |
| `MerchantCurrCode` | A | 3 | If `Mode=RL` | |
| `StatusMsg` | AN | 255 | No | |

Samples (docs): RL request `{"MerchantID":"JJTEST","PymRefNum":"JJTESTREF0001","Mode":"RL","MerchantTxnAmt":"100.00","MerchantCurrCode":"MYR","Freq":"MT","StDt":"2026-09-02","EndDt":"2027-09-02","Sign":"1a890202..."}`; FLX request `{"MerchantID":"JJTEST","PymRefNum":"JJTESTREF0002","Mode":"FLX","Sign":"432d3a30..."}`. RL response `{"suc":true,"code":0,"msg":"Success","data":{"MerchantID":"JJTEST","DirectDebitID":"JJTESTDD2609010020","Status":"Success","Sign":"84165f6c...","Mode":"RL","PymRefNum":"JJTESTEMDT2609010101","MerchantTxnAmt":"100.00","MerchantCurrCode":"MYR"},"traceId":"...","timeStamp":"2026-09-01T05:56:26.897936Z"}`.

Possible errors: 90015, 90017, 90025, 90175, 90176, 90178, 90179, 90180, 90182, 90183, 90184, 90187.

## Create Direct Debit Instruction Transaction (FLX)

URL: `<payment URL>/PPG/create-ddi-txn`, POST.

Request (all mandatory): `MerchantID` (AN 6), `Sign` (DDI Create Txn Request Sign), `DirectDebitID` (AN 20), `MerchantOrdID` (ANS (space) 20, unique reference for this debit), `MerchantTxnAmt` (N 12(2)), `MerchantCurrCode` (A 3).

Response `data`: `MerchantID`, `DirectDebitID`, `Status` (`Success`/`Failed`), `Sign` (DDI Create Txn Response Sign), `MerchantOrdID`, `MerchantTxnAmt`, `MerchantCurrCode` (all mandatory); optional `StatusMsg`.

Possible errors: 90015, 90016, 90017, 90025, 90175, 90178, 90179, 90180, 90185, 90186, 90188.

`Status=Success` here means the instruction was accepted. How the final debit result is delivered (callback or query) is not documented (checked 2026-10-21); ask Paydibs.

## Cancel Direct Debit Instruction

URL: `<payment URL>/PPG/cancel-ddi`, POST.

Request: `MerchantID`, `DirectDebitID`, `Sign` (DDI Cancel Request Sign). Response `data`: `MerchantID`, `DirectDebitID`, `Status`, `Sign` (DDI Cancel Response Sign); optional `StatusMsg`. Possible error: 90015.

## Sign Formulas

SHA512, hex, plain concatenation, Merchant Password first. E-mandate pages: optional fields not populated are left blank (empty string). DDI pages: "Any conditional fields with an empty or null value should not be included in the source string." Both produce the same concatenation.

| Operation | String to sign |
| --- | --- |
| Enrolment request | `MerchantPassword + TxnType + MerchantID + PymRefNum + CustName + CustIdTyp + CustIdNum + MaxDbtAmt + DbtFreq + MaxDbtFreq + EndDt + PrpOfPym + MerchantRURL + MerchantCallbackURL` |
| Maintenance request | `MerchantPassword + TxnType + MerchantID + PymRefNum + CustPhone + CustEmail + MaxDbtAmt + DbtFreq + MaxDbtFreq + MerchantRURL + MerchantCallbackURL` |
| Termination request | `MerchantPassword + TxnType + MerchantID + PymRefNum + TmnRsn + MerchantRURL + MerchantCallbackURL` |
| E-mandate response | `MerchantPassword + TxnType + MerchantID + PymRefNum + MaxDbtAmt + DbtFreq + MaxDbtFreq + PrpOfPym + TmnRsn + Status + StatusMsg` |
| Create DDI request | `MerchantPassword + MerchantID + PymRefNum + MerchantTxnAmt + MerchantCurrCode + Mode + Freq + StDt + EndDt + MaxCyc` |
| Create DDI response | `MerchantPassword + data.MerchantID + data.PymRefNum + data.MerchantTxnAmt + data.MerchantCurrCode + data.DirectDebitID + data.Mode + data.Status + data.StatusMsg` |
| Create DDI txn request | `MerchantPassword + MerchantID + MerchantOrdID + MerchantTxnAmt + MerchantCurrCode + DirectDebitID` |
| Create DDI txn response | `MerchantPassword + data.MerchantID + data.MerchantOrdID + data.MerchantTxnAmt + data.MerchantCurrCode + data.DirectDebitID + data.Status + data.StatusMsg` |
| Cancel DDI request | `MerchantPassword + MerchantID + DirectDebitID` |
| Cancel DDI response | `MerchantPassword + data.MerchantID + data.DirectDebitID + data.Status + data.StatusMsg` |

Note the field order differs from the JSON key order (e.g. DDI response puts `DirectDebitID` after the amount fields).

Doc examples reproduced with SHA512 over UTF-8 on 2026-10-21 (all match):

- Enrolment: `abc123EMDT_CRTPAYDIBSEMDT202607220001Wong Chew Shin1000102089281100.00MT122027-08-31Loan Repaymenthttps://MerchantTestEnv.com/response.aspxhttp://MerchantTestEnv.com/callbackresponse.aspx` gives `93e9a77bcd3e...b7588408`.
- Maintenance: `abc123EMDT_MTNPAYDIBSEMDT2026072200010123456789emandatecustomer@mail.com100.00MT12https://MerchantTestEnv.com/response.aspxhttp://MerchantTestEnv.com/callbackresponse.aspx` gives `c380c0236823...113aad15`.
- Termination: `abc123EMDT_TMNPAYDIBSEMDT202607220001Customer has terminated subscription.https://MerchantTestEnv.com/response.aspxhttp://MerchantTestEnv.com/callbackresponse.aspx` gives `ff03e0b78719...0d0a788b`.
- E-mandate response (no `TmnRsn`): `abc123EMDT_CRTPAYDIBSEMDT202607220001139.00MT12Gym Membership0E-Mandate successfully created.` gives `598580c32772...f8857d6b`.
- Create DDI RL: `63P4TMY0I(JJTESTJJTESTREF0001100.00MYRRLMT2026-09-022027-09-02` gives `1a8902028b1d...898dda3b`.
- Create DDI FLX (conditional fields omitted): `63P4TMY0I(JJTESTJJTESTREF0002FLX` gives `432d3a303ace...5ba82c0a8`.
- Create DDI response (no `StatusMsg`): `63P4TMY0I(JJTESTJJTESTEMDT2609010101100.00MYRJJTESTDD2609010020RLSuccess` gives `84165f6c5289...666c9601f`.
- Create DDI txn request: `63P4TMY0I(JJTESTJJTEST20260902ORD006100.00MYRJJTESTDD2609020024` gives `7607f0d04636...444cc9b00`.
- Create DDI txn response: `...JJTESTDD2609020024Success` gives `cfe17f7a921b...d271110c`.
- Cancel request `63P4TMY0I(JJTESTJJTESTDD2609010020` gives `2929feb23bc4...f688504`; response `...Success` gives `57488bb745d5...2a21acff`.

The sample DDI create response echoes a different `PymRefNum` (`JJTESTEMDT2609010101`) than its request (`JJTESTREF0001`); the hash uses the response value. Always sign the response over the values received.

```js
import { createHash } from "node:crypto";
const sha512 = (s) => createHash("sha512").update(s, "utf8").digest("hex");
const v = (x) => (x === undefined || x === null ? "" : String(x));
// Create DDI response check
function verifyCreateDdi(pw, d) {
  const exp = sha512(pw + d.MerchantID + d.PymRefNum + v(d.MerchantTxnAmt) +
    v(d.MerchantCurrCode) + d.DirectDebitID + d.Mode + d.Status + v(d.StatusMsg));
  return exp === String(d.Sign).toLowerCase(); // use a constant-time compare in production
}
```

## Not Documented (checked 2026-10-21)

- Content type of e-mandate requests (form post vs JSON) and whether enrolment is a browser redirect to the bank.
- How the final result of each DDI debit is reported (callback, query, or report).
- Whether a QUERY exists for e-mandate or DDI status.
- Which banks support e-mandate, and test banks for staging.
