# Paydibs Tokens, Linked Accounts And In-Store QR

Checked 2026-10-21 against docs version 4.0. Sources:

- Bind option (retrieve linked accounts/cards): `https://v3api-docs.paydibs.com/docs/merchant-api-reference/bind-option-request`, `.../bind-option-response`
- Revoke token: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/revoke-token-request`, `.../revoke-token-response`
- In-store DuitNow QR: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/in-store-pay-request`, `.../in-store-pay-response`
- Sign: `https://v3api-docs.paydibs.com/docs/sign-hashing/merchant-request-sign`, `.../merchant-response-sign`

## How Card Tokens Are Created

Send `CustReferenceId` (your customer ID, max 100) on a PAY request. If the customer chooses to save the card, the PAY response returns `TokenType=CARD`, `Token` (AN 50) and `MaskedPan`. Next time, send `TokenType=CARD` and `Token` on PAY to charge the saved card (`Token` is part of the PAY Sign). AutoDebit tokens (`PaymentOption=TNGAD`) are created on the Paydibs side; retrieve them with BINDOPT.

## BINDOPT: Retrieve Customer Linked Accounts/Cards

URL: `<payment URL>/PPG/trigger`, POST, server-side. Response is JSON in the same session (no `MerchantRURL`). TLS: production TLS 1.2 only.

Request, always: `TxnType` (A 7, `BINDOPT`), `MerchantID` (AN 6), `Sign` (ANS 128).

AutoDebit variant, mandatory: `CustName` (ANS (space) 50), `CustPhone` (ANS 25), `CustEmail` (ANS 60), `PaymentOption` (A 50, `TNGAD`).
Card variant, mandatory: `CustReferenceId` (ANS (space) 100), `TokenType` (A 4, `CARD`).

Request Sign:

- AutoDebit: `MerchantPassword + MerchantID + CustName + CustPhone + CustEmail + PaymentOption`
- Card: `MerchantPassword + MerchantID + CustReferenceId + TokenType + Token`. Conflict in the docs: the card BINDOPT request table and form sample have no `Token` field, yet the Sign example includes `Token = wXZ7Lb0E4-v7CQj459Nk6Q`. Without a token to send, sign with an empty `Token` and confirm with Paydibs on staging.

Response:

| Field | Type | Max | Notes |
| --- | --- | --- | --- |
| `TxnType` | A | 7 | Echo |
| `MerchantID` | AN | 6 | Echo |
| `BindOptions` | L | - | List of linked accounts/cards |
| `BindOptResult` | B | 5 | `true`/`false` |
| `BindOptDesc` | A | 255 | Result message (absent in success samples) |
| `Sign` | ANS | 128 | Bind option response Sign |
| `CustName`, `CustPhone`, `CustEmail`, `PaymentOption` | | | Echo, AutoDebit variant |
| `CustReferenceId`, `TokenType` | | | Echo, card variant |

`BindOptions[]` items: `Token` (AN 50, AutoDebit access token or saved card token); card only: `MaskedPan` (AN 19, first 6 + last 4, e.g. `512345XXXXXX0008`), `ExpiredMonth` (N 2, `MM`), `ExpiredYear` (N 2, `YY`), `CardHolderName` (ANS (space) 100). Samples also include `DisplayPan` (e.g. `****0008`), not in the field table.

Response Sign: `MerchantPassword + MerchantID + BindOptResult + BindOptDesc` (`BindOptResult` as the text `true`/`false`; `BindOptDesc` empty when absent, an assumption to confirm).

Sample failures: `"BindOptDesc":"No record found","BindOptResult":false`; invalid data returns `"Required field(s) is contains invalid characters"` (the docs sample there has a duplicated `BindOptDesc` key and no Sign).

## REVOKE: Revoke AutoDebit Access Token Or Card Token

URL: `<payment URL>/PPG/trigger`, POST, server-side. JSON response in the same session.

Request, always: `TxnType` (A 7, `REVOKE`), `MerchantID`, `Sign`.
AutoDebit variant: `CustName`, `CustPhone`, `CustEmail`, `PaymentOption` (`TNGAD`).
Card variant: `CustReferenceId`, `TokenType` (`CARD`), `Token` (AN 50, the saved card token from the PAY response).

Request Sign: same two formulas as BINDOPT (AutoDebit: `MerchantPassword + MerchantID + CustName + CustPhone + CustEmail + PaymentOption`; Card: `MerchantPassword + MerchantID + CustReferenceId + TokenType + Token`).

Response fields: `TxnType` (A 7), `MerchantID` (AN 6), `RevokeResult` (B 5), `RevokeDesc` (A 255). The response Sign page gives `MerchantPassword + MerchantID + RevokeResult + RevokeDesc`, but the response field table and all samples omit `Sign`. Verify it when present.

Documented `RevokeDesc` values: `Success`, `Internal Error`, `Required field(s) is contains invalid characters`, `Unable to find Customer's Access Token for the Payment Option`, `Unable to find Customer's Card Token`, `Failed`.

The Hosted Session SDK can also delete saved cards from the iframe (`onSavedCardDelete` reports `{ tokenId, ok, reason? }`); see `paydibs-hosted-session.md`.

## In-Store DuitNow QR PAY

URL: `<payment URL>/PPG/offline/trigger-qr`, POST. TLS 1.2 only in both environments. Returns QR content for your terminal or screen to render.

Request, mandatory:

| Field | Type | Max | Notes |
| --- | --- | --- | --- |
| `TxnType` | A | 7 | `PAY` |
| `MerchantID` | AN | 6 | |
| `MerchantPymtID` | ANS (space) | 20 | Unique; duplicates rejected |
| `MerchantTxnAmt` | N | 12(2) | `10.99` format |
| `MerchantCurrCode` | A | 3 | |
| `AcqBank` | A | 5 | `DNQR` |
| `Sign` | ANS | 128 | In-store Sign |
| `QrPymFlw` | A | 3 | `USM` |

Optional: `PageTimeout` (N 4, seconds; QR validity; default 60; provider may expire earlier).

Sign: `MerchantPassword + TxnType + MerchantID + MerchantPymtID + MerchantTxnAmt + MerchantCurrCode`.

Response:

| Field | Type | Max | Notes |
| --- | --- | --- | --- |
| `suc` | B | 5 | Success indicator |
| `code` | N | 6 | Response code (`0` on success) |
| `msg` | ANS (space) | 20 | Response message |
| `data.qrExpTm` | ANS | 19 | QR expiry, UTC+8, e.g. `2026-06-18T17:07:41` |
| `data.qrCtn` | ANS (space) | 300 | EMV QR payload string to render |
| `traceId` | AN | 32 | For troubleshooting |
| `timeStamp` | ANS | 19 | Response time (sample is UTC with `Z`) |

The in-store response has no Sign. How the merchant learns the final in-store payment result (callback or QUERY by `MerchantPymtID`) is not documented on the in-store pages (checked 2026-10-21); QUERY on `/PPG/trigger` is the documented status API for PAY transactions, so confirm with Paydibs that it covers QR payments.

Terminal API pages (callback, void, settlement and terminal Sign) exist as route names in the docs bundle but are not published in the navigation (checked 2026-10-21). Do not use them until Paydibs publishes them.
