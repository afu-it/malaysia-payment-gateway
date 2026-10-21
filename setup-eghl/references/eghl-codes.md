# eGHL Codes And Enums

Every enum that appears in eGHL's official public sources (Bitbucket SDKs and plugins, checked 2026-10-21). The full merchant API specification may list more values; it is not public. `eghl-api.md` is the entry point.

## TransactionType

| Value | Meaning | Source |
| --- | --- | --- |
| `SALE` | One-off payment via hosted page | all plugins, `eGHL_HASH.php` |
| `AUTH` | Card authorisation, capture later | merchantapi README, Omnipay |
| `QUERY` | Status requery | merchantapi `Query.php`, paramvalidator |
| `REFUND` | Online refund (cards) | merchantapi `Refund.php`, paramvalidator |
| `RSALE` | Reversal / void | merchantapi `APIBase.php`, paramvalidator |
| `CAPTURE` | Capture an `AUTH` | merchantapi `Capture.php`, paramvalidator |
| `SETTLE` | Settlement request (fields only, see `eghl-api.md`) | paramvalidator `SettlementRequest.php` |
| `SOPBL` | Forex rate list (JSON) | Magento `Helper/Data.php` |
| `SOP`, `SOP2`, `SOPCARD` | Hash orders only; purpose not published | `eGHL_HASH.php` |

## PymtMethod

| Value | Label in official plugins | Source |
| --- | --- | --- |
| `ANY` | All Payment Methods (shopper picks on eGHL page) | Magento `PayMethods.php`, WooCommerce, OpenCart, WHMCS, PrestaShop |
| `CC` | Credit Card | Magento `PayMethods.php`, merchantapi README |
| `DD` | Direct Debit (online banking / FPX) | Magento `PayMethods.php`, merchantapi README |
| `WA` | e-Wallet | Magento `PayMethods.php` |

Bank codes, e-wallet codes, and `IssuingBank` values are not published; `IssuingBank` is free text (AN 30) in the validator.

## TxnStatus

| Value | Meaning | Where it appears |
| --- | --- | --- |
| `0` | Success | payment response, all API responses |
| `1` | Failed (`TxnMessage` `Buyer cancelled` = shopper cancelled) | payment response, API responses |
| `2` | Pending (QUERY: pending at bank when `TxnID` and `BankRefNo` or `AuthCode` are present) | payment response, API responses |
| `9` | Reversed | QUERY |
| `10` | Refunded | QUERY |
| `15` | Authorised (awaiting capture) | QUERY |
| `16` | Captured | QUERY |
| `31` | Reversal pending | QUERY |
| `-1` | Transaction not found | QUERY, REFUND, RSALE |
| `-2` | System error | QUERY, REFUND, RSALE |

Plugin-internal values (never sent by eGHL): WooCommerce and OpenCart set `3`, PrestaShop sets `-1`, to mark a hash mismatch locally. Do not confuse PrestaShop's local `-1` with eGHL's "not found".

## TxnExists (QUERY)

| Value | Meaning |
| --- | --- |
| `0` | Transaction exists |

Other values are not published.

## TxnMessage

Free text. The only value official code branches on is `Buyer cancelled` (OpenCart, Magento).

## TokenType

| Value | Meaning | Source |
| --- | --- | --- |
| `OCP` | Card remembered by ADAPTIS if the shopper agrees ("Enable OCP") | WooCommerce `init_form_fields`, `get_eghl_args` |

## CurrencyCode

ISO 4217 alphabetic, 3 letters. The WooCommerce plugin enables itself only for: `CNY`, `IDR`, `AUD`, `BRL`, `CAD`, `MXN`, `NZD`, `HKD`, `SGD`, `USD`, `EUR`, `JPY`, `TRY`, `NOK`, `CZK`, `DKK`, `HUF`, `ILS`, `MYR`, `PHP`, `PLN`, `SEK`, `CHF`, `TWD`, `THB`, `GBP`, `RMB`, `RUB`. Which ones your account accepts depends on the contract.

## Field Data Types (paramvalidator)

| Code | Meaning | Check applied |
| --- | --- | --- |
| `A` | Alphabetic | PHP `ctype_alpha` |
| `N` | Numeric | PHP `is_numeric` (amounts additionally need exactly 2 decimals, no `,`) |
| `AN` | Alphanumeric | length only (e.g. `PaymentID` `130#1547432197655ef9` in the README) |
| `ANS` | Alphanumeric with symbols | length only |

Lengths are counted with `mb_strlen` (characters, not bytes).

## Callback And Return Acknowledgement Bodies

| Body | Used by | When |
| --- | --- | --- |
| `OK` | all plugins | Callback processed (also for duplicates) |
| `ERROR` | PrestaShop, Magento | Hash mismatch or amount mismatch (fraud) |
| `Fail` | WHMCS | Failed status |
| `Error` | OpenCart | Callback with local status `3` |

Send `OK` once your durable write succeeds. Behaviour of eGHL on other bodies is not published.

## Validator Error Messages (client-side, paramvalidator)

These are thrown by the official PHP validator before sending, not returned by eGHL. Useful when porting the checks:

- `Param '<name>' is required`
- `Param '<name>' exceeds maxlength i.e. <n>`
- `Param '<name>' expects data type to be Alphabetic|Numeric|AlphaNumeric`
- `Amount [<v>] must be 2 decimal places`
- `',' Sign must be excluded in amount [<v>]`
- `The Amount [<v>] must be numeric`
- `URL [<v>] is invalid` (the validator converts `;` back to `&` to validate, then stores with `;`)
- `Email [<v>] is invalid`
- `Value must be QUERY|CAPTURE|RSALE|REFUND|SETTLE`
- `Parameter Token is required if parameter TokenType is defined`

## eGHL Gateway Error Codes

Not documented in official public sources (checked 2026-10-21). Use `TxnStatus`, `TxnMessage`, and QUERY; ask the account manager for the error code table.
