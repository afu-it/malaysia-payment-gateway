# Paydibs Codes And Enums

Every enum, code list and data rule published in the Paydibs Merchant Integration API v3 docs (version 4.0, 24 Sep 2026; checked 2026-10-21). Sources: Appendices `https://v3api-docs.paydibs.com/docs/appendices`, Introduction `https://v3api-docs.paydibs.com/docs/introduction`, Revision `https://v3api-docs.paydibs.com/docs/revision`, DDI pages under `https://v3api-docs.paydibs.com/docs/e-mandate-feature/`.

## Data Types (Introduction 1.2)

| Abbreviation | Meaning |
| --- | --- |
| `A` | Alpha |
| `AN` | Alphanumeric |
| `ANS` | Alphanumeric + special character |
| `ANS (with space)` | Alphanumeric + special character + space |
| `N` | Numeric |
| `B` | Boolean |
| `L` | List |
| `DateTime` | Date and time, ISO 8601 |

`N 12(2)` means an amount with exactly 2 decimals, for example `1000.00`.

## Allowed Special Characters

Introduction 1.3 (general fields): `! @ # $ % ^ & * ( ) _ + - = [ ] { } | ; : , . < > ? /`

Introduction 1.4 (E-KYC conditional fields `EkycAccountNumber`, `EkycBuyerId`): `@ / \ ( ) . - _ , &`. Note on the page: `EkycBuyerId` does not support slash (`/`) or ampersand (`&`).

Special characters in `MerchantOrdDesc`, `CustName`, `CustEmail` and `MerchantName` are trimmed or replaced automatically (revision 3.4). `MerchantOrdDesc` does not support Mandarin.

## TxnType (Appendices, plus values used on the API pages)

| TxnType | Meaning | Endpoint |
| --- | --- | --- |
| `PAY` | Payment (also in-store DuitNow QR) | `/PPG/trigger`, `/PPG/offline/trigger-qr` |
| `QUERY` | Status inquiry | `/PPG/trigger` |
| `REVOKE` | Revoke AutoDebit access token or saved card token | `/PPG/trigger` |
| `REFUND` | Submit refund request | `/PPG/refund` |
| `BINDOPT` | Retrieve customer linked accounts/cards | `/PPG/trigger` |
| `MIT` | Merchant-initiated recurring charge | `/PPG/trigger` |
| `EMDT_CRT` | E-Mandate creation (enrolment) | `/PPG/trigger-emandate` |
| `EMDT_MTN` | E-Mandate maintenance | `/PPG/trigger-emandate` |
| `EMDT_TMN` | E-Mandate termination | `/PPG/trigger-emandate` |

The Appendices table lists only PAY, QUERY, REVOKE and REFUND; the other values come from their own API pages. Uppercase is required.

## PTxnStatus (payment status)

| Code | Meaning |
| --- | --- |
| `0` | Payment successful |
| `1` | Payment failed |
| `2` | Payment pending |
| `9` | Payment voided |
| `10` | Refunded |
| `17` | Payment cancelled at payment page |
| `18` | Payment cancelled |
| `19` | Expired |
| `20` | Chargeback |
| `22` | Acquirer bank is under maintenance |
| `50` | Out of per transaction amount limit |
| `51` | Transaction amount is lower than minimum limit |
| `-1` | Transaction not exists/not found |
| `-2` | Internal system error |

## PAYExists (QUERY response)

| Value | Meaning |
| --- | --- |
| `0` | Transaction being queried exists in Paydibs |
| `1` | Transaction being queried does not exist in Paydibs |
| `2` | An error occurred during query processing; merchant can retry the query |

## Currency Codes

`MYR` Malaysia Ringgit, `SGD` Singapore Dollar, `PHP` Philippine Peso, `VND` Vietnamese dong, `CNY` Chinese Yuan Renminbi, `HKD` Hong Kong Dollar, `IDR` Indonesian Rupiah, `KHR` Cambodian Riel, `KRW` South Korean Won, `THB` Thai Baht.

VND, KRW, IDR and KHR have no decimals in ISO 4217 but must still be sent with `.00`. E-Mandate and direct debit amounts are MYR.

## Method Codes

| Code | Method |
| --- | --- |
| `OB` | Online Banking (FPX) |
| `CC` | Credit Card |
| `WA` | E-Wallet |

Note: the MIT success sample shows `"Method": "MC"`, which is not in this list. Treat `Method` in responses as informational and do not reject unknown values.

## Payment Option Code

| Code | Meaning |
| --- | --- |
| `TNGAD` | TNG AutoDebit |

## Token Type Code

| Code | Meaning |
| --- | --- |
| `CARD` | Card |

## FPX E-KYC Result (`EkycResult`, e.g. `020100`)

| Digits | Field | Values |
| --- | --- | --- |
| 1 to 2 | Account Type | `00` undetermined, `01` CASA, `02` LCA |
| 3 to 4 | Account Number match | `00` undetermined, `01` true, `02` false |
| 5 to 6 | Buyer ID match | `00` undetermined, `01` true, `02` false |

## Country And Address Type (Cash on Delivery)

- Country code: `MY` Malaysia.
- Address type: `HME` Home, `OFC` Office, `OTH` Others.

## Recurring Enums

| Field | Values |
| --- | --- |
| `Mode` | `FLX` Flexible, `FIX` Fixed |
| `Src` (schedule source) | `MER` Merchant, `PG` Payment Gateway (Paydibs) |
| `Bhv` (behaviour) | `NONE` verify only, no initial charge, recurring setup only; `IMDT` immediate charge + recurring creation (default) |
| `BillCycTyp` | `FD` Fixed Dates, `RL` Rule |
| `Freq` | `HR` Hours, `WKLY` Weekly, `MTLY` Monthly, `YRLY` Yearly |
| `WkDy` (weekday) | `MON`, `TUE`, `WED`, `THUR`, `FRI`, `SAT`, `SUN` |

## E-Mandate Enums

| Field | Values |
| --- | --- |
| `CustIdTyp` | `1` New IC Number, `2` Old IC Number, `3` Passport Number, `4` Business Registration, `5` Others |
| `DbtFreq` / DDI `Freq` | `DL` Daily, `WK` Weekly, `MT` Monthly, `YR` Yearly |

E-Mandate `Status` (redirect/callback response):

| Code | Meaning |
| --- | --- |
| `0` | E-Mandate application successful |
| `1` | E-Mandate application failed |
| `2` | E-Mandate application pending |
| `22` | Acquirer bank is under maintenance |
| `24` | E-Mandate application cancelled |
| `50` | Out of per transaction amount limit |
| `-1` | E-Mandate application not exists/not found |
| `-2` | Internal system error |

Direct debit `data.Status` (create DDI, create DDI transaction, cancel DDI): `Success` or `Failed`.

## Direct Debit Error Codes

| Code | Message | Returned by |
| --- | --- | --- |
| `90015` | Required field is missing: {0}. | create-ddi, create-ddi-txn, cancel-ddi |
| `90016` | Required field is invalid: {0}. | create-ddi-txn |
| `90017` | Required field is contains invalid characters: {0}. | create-ddi, create-ddi-txn |
| `90025` | Required field is invalid: {0}. | create-ddi, create-ddi-txn |
| `90175` | Direct Debit Amount exceeds the Maximum Debit Amount configured in the E-Mandate. | create-ddi, create-ddi-txn |
| `90176` | Direct Debit Frequency does not match the E-Mandate Frequency. | create-ddi |
| `90178` | Direct Debit max cycle exceed the maximum number of allowed debit transactions. | create-ddi, create-ddi-txn |
| `90179` | E-Mandate is not in Active status. | create-ddi, create-ddi-txn |
| `90180` | No valid E-Mandate found for Direct Debit processing. | create-ddi, create-ddi-txn |
| `90182` | Invalid Start Date. | create-ddi |
| `90183` | End date is before start date: {0}, {1} | create-ddi |
| `90184` | Start date must be after today date: {0} | create-ddi |
| `90185` | No valid Direct Debit found for Direct Debit processing | create-ddi-txn |
| `90186` | Direct Debit ID not in Active status | create-ddi-txn |
| `90187` | E-Mandate ID has active DDI scheduler | create-ddi |
| `90188` | Direct Debit Transaction for this E-Mandate has existed for the past {0}. | create-ddi-txn |

The docs do not say which JSON field carries these codes; the response envelope has `suc`, `code`, `msg`, so `code` is the likely carrier. Confirm on staging.

## Hosted Session SDK Codes

`UpdateSessionStatus`: `ok`, `fields_in_error`, `system_error`, `session_invalid`, `session_expired`, `unsupported`.

SDK error codes (`onError` payload `{ code, message, recoverable }`):

| Code | Recoverable | Meaning |
| --- | --- | --- |
| `session_missing` | Depends | No `paySessId` given to `configure()` |
| `merid_missing` | Depends | Merchant ID missing from the iframe URL fragment |
| `session_invalid` | Yes | Session exists server-side but is malformed |
| `session_expired` | Yes | Session expired; request a new `paySessId` |
| `session_validation_failed` | Yes | Server-side session validation failed |
| `handshake_origin_mismatch` | No | Iframe replied from an unexpected origin; check `environment` |
| `protocol_version_unsupported` | No | SDK file and iframe are incompatible |
| `handshake_timeout` | No | No handshake within `handshakeTimeoutMs` (default 60000 ms) |
| `iframe_destroyed` | No | Iframe unmounted, navigated away or removed |
| `update_in_progress` | No | `updateSession()` called while a previous call is in flight |
| `chokepoint_violation` | No | postMessage payload failed schema validation (corrupted SDK or tampering) |

Calling an `on*` method before `payment.ready` resolves throws `not_ready`.

## Revision History (docs)

| Version | Date | Change |
| --- | --- | --- |
| 1.0 | 19 Apr 2018 | API creation |
| 2.0 | 1 Aug 2018 | Credit card API |
| 2.1 | 28 Aug 2018 | Card test card added (not present in current pages) |
| 2.2 | 2 Nov 2018 | One Click Payment features, URL |
| 2.3 | 4 Jun 2020 | E-wallet payment method |
| 2.4 | 10 Jun 2020 | Currency code update |
| 2.5 | 23 Feb 2021 | Removed CardHolder, CardNo, CardExp, CardCVV2 from PAY request |
| 3.0 | 30 Dec 2024 | New payment checkout API |
| 3.1 | 19 Jun 2025 | E-KYC fields on PAY request, PAY response and QUERY response |
| 3.2 | 29 Jul 2025 | `CustReferenceId`, `TokenType`, `Token` on PAY; `MaskedPan` on response |
| 3.3 | 31 Jul 2025 | Card token revoke, retrieve linked accounts/cards (BINDOPT), new Sign for both |
| 3.4 | 21 Oct 2025 | Special characters trimmed/replaced in four PAY fields |
| 3.5 | 29 Oct 2025 | Refund API |
| 3.6 | 16 Dec 2025 | Cash on Delivery fields on PAY |
| 3.7 | 1 Apr 2026 | Recurring CIT/MIT, MIT Sign, Mode and Src appendices |
| 3.8 | 26 May 2026 | Hosted Payment Session API and Sign |
| 3.9 | 9 Jul 2026 | In-Store merchant API, sidebar restructure |
| 4.0 | 24 Sep 2026 | E-Mandate (enrolment, maintenance, termination), Direct Debit (create DDI, create DDI txn, cancel DDI), Sign for both, ID Type and Debit Frequency appendices |
