# eGHL API Reference

Use this reference for factual eGHL (ADAPTIS) API guidance. The full eGHL Merchant API specification is given to merchants by eGHL/ADAPTIS and is not public; everything below is taken from eGHL's own public Bitbucket SDKs and plugins (checked 2026-10-21). Where a detail is missing, it is marked "Not documented in official public sources".

## Source URLs

- `https://bitbucket.org/eghl/merchantapi/src/master/README.md` (Query, Refund, Reversal, Capture)
- `https://bitbucket.org/eghl/merchantapi/src/master/src/core/APIBase.php` (URLs, POST, response parse, response hash)
- `https://bitbucket.org/eghl/merchantapi/src/master/src/Query.php` (TxnStatus meanings)
- `https://bitbucket.org/eghl/eghl_hash/src/master/eGHL_HASH.php` (hash field lists and algorithm)
- `https://bitbucket.org/eghl/eghl-omnipay/src/master/` (README, `src/Gateway.php`, `src/Message/*.php`)
- `https://bitbucket.org/eghl/paramvalidator/src/master/src/PaymentRequest.php` and `src/core/validatorBase.php` (field types, lengths, amount/URL rules)
- `https://bitbucket.org/eghl/adaptis-woocommerce/src/main/` (`adaptis-woocommerce-8.2.4.zip`)
- `https://bitbucket.org/eghl/adaptis-whmcs/src/main/` (`eGHL-whmcs-6.x-7.x-20250709.zip`)
- `https://bitbucket.org/eghl/adaptis-magento/src/main/` (`eghl-magento-2.1.7.zip`: `Helper/Data.php` for SOPBL, `Block/Terminal.php`, `Controller/Index/ResponseHandler.gte_m230.php`, `Model/Config/Source/*.php`)
- `https://bitbucket.org/eghl/paramvalidator/src/master/src/` (`QueryRequest.php`, `RefundRequest.php`, `ReversalRequest.php`, `CaptureRequest.php`, `SettlementRequest.php`)
- `https://bitbucket.org/eghl/adaptis-opencart/src/main/` (`ghl.ocmod.zip`: `catalog/controller/payment/ghl.php`)
- `https://bitbucket.org/eghl/adaptis-prestashop-1.6.x/src/main/` (`eghl-eghl-prestashop1.6.x-20250523.zip`: `modules/eghl/eghl.php`, `eGHLResponseHandler.php`)

## Integration Surfaces

- Hosted payment page: browser form POST to `Payment.aspx` (this reference).
- Server-to-server Query, Refund, Reversal, Capture: form POST to the same `Payment.aspx`.
- Official plugins: WooCommerce, Magento 2, OpenCart 3, WHMCS, PrestaShop 1.6 (see `account-setup.md`).
- Official PHP libraries: `e-ghl/merchant-api`, `e-ghl/omnipay`, `e-ghl/param-validator` on Packagist.
- ADAPTIS marketing also lists Email Payment Link, Recurring Payment, and Easy Payment Plan (instalment). Their API details are not documented in official public sources.

Enums (PymtMethod, TransactionType, TxnStatus, currencies, data types, acknowledgement bodies, validator error messages) are collected in `eghl-codes.md`.

## Endpoint Catalogue

eGHL has one gateway endpoint per environment (`Payment.aspx`, see Environments). Every operation is selected by the `TransactionType` field. This is every operation that appears in the official public sources (checked 2026-10-21):

| # | Operation | Method / direction | `TransactionType` | Purpose | Hash | Source |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Sale (hosted payment page) | Browser form POST (WooCommerce uses GET query string) to `Payment.aspx` | `SALE` | Take a one-off payment; eGHL shows the method/bank picker | SALE `HashValue` | paramvalidator `PaymentRequest.php`, all plugins |
| 2 | Authorise | Browser form POST to `Payment.aspx` | `AUTH` | Card authorisation only, capture later | SALE `HashValue` | merchantapi README (Capture section), Omnipay |
| 3 | Payment return | eGHL to `MerchantReturnURL` via shopper browser (POST or GET) | echoes `SALE`/`AUTH` | Show result to shopper | `HashValue2` (verify) | plugins, Omnipay `CompletePurchaseRequest.php` |
| 4 | Payment callback | eGHL server to `MerchantCallBackURL` (POST or GET) | echoes `SALE`/`AUTH` | Server-to-server result; reply `OK` | `HashValue2` (verify) | plugins |
| 5 | Approval / unapproval URLs | eGHL to `MerchantApprovalURL` / `MerchantUnApprovalURL` | n/a | Optional success / failure landing URLs | in SALE hash | eGHL_HASH.php, paramvalidator |
| 6 | Query (requery) | Server POST `application/x-www-form-urlencoded`, URL-encoded response | `QUERY` | Read status of an existing transaction | API request / response `HashValue` | merchantapi `Query.php` |
| 7 | Refund | Server POST, URL-encoded response | `REFUND` | Full or partial online refund (cards only) | API request / response `HashValue` | merchantapi `Refund.php`, README |
| 8 | Reversal (void) | Server POST, URL-encoded response | `RSALE` | Void before bank settlement | API request / response `HashValue` | merchantapi `Reversal.php`, README |
| 9 | Capture | Server POST, URL-encoded response | `CAPTURE` | Capture an `AUTH` transaction | API request / response `HashValue` | merchantapi `Capture.php`, README |
| 10 | Settlement | Server POST (assumed, same endpoint) | `SETTLE` | Field list only; purpose and hash not published | not published | paramvalidator `SettlementRequest.php` |
| 11 | Forex rate list | Server GET `Payment.aspx?...`, JSON response | `SOPBL` | Exchange rates for a base currency (Magento uses it to check foreign-currency responses) | SOPBL `HashValue` | Magento `Helper/Data.php`, eGHL_HASH.php |
| 12 | `SOP`, `SOP2`, `SOPCARD` | not published | `SOP`, `SOP2`, `SOPCARD` | Only their hash field orders exist in `eGHL_HASH.php`; no request/response fields or purpose published | see hash table | eGHL_HASH.php |

Operations 10 and 12 must not be built from this skill alone: ask the account manager for the merchant API specification. No REST/JSON payment API, no webhook signature header, and no other host is published.

## Environments

Live:

```text
https://securepay.e-ghl.com/IPG/Payment.aspx
```

Test (UAT):

```text
https://pay.e-ghl.com/IPGSG/payment.aspx
```

The SDKs use these two exact URLs (path case varies between `payment.aspx`, `Payment.aspx`, and `ipgsg` across official plugins; all appear in official code). The Magento plugin also lists a Philippines test URL (`http://test2ph.ghl.com:86/IPGSG/Payment.aspx`), which is not for Malaysia, and Magento and PrestaShop list `http://localhost:8080/IPGLocal/Payment.aspx` ("Localhost"), an internal developer option; never use it.

There is no separate REST API host and no API key header. Authentication is the hash computed with the Merchant Password.

## Credentials

- `ServiceID`: Merchant Service ID, 3 alphanumeric characters, issued by eGHL/ADAPTIS.
- Merchant Password: shared secret used only inside the hash. Never sent as a field.
- Test and live use different credentials. The SDK READMEs publish the shared demo test pair `ServiceID=SIT`, password `sit12345` for the test URL only. Use your own issued test credentials where possible and never use the demo pair in live.

## Hash Algorithm (HashValue / HashValue2)

From `eGHL_HASH.php`:

```text
hash = lowercase_hex( SHA256( MerchantPassword + v1 + v2 + ... + vn ) )
```

- Plain concatenation, no separators, no URL encoding.
- Values are taken exactly as sent or received. An optional field that is absent contributes an empty string. A missing mandatory field makes the library refuse to hash.
- Compare received hashes case-insensitively (`strcasecmp` in WooCommerce and Magento plugins) and in constant time.

Field order by message:

| Message | Hash field | Order (password first, then) |
| --- | --- | --- |
| SALE / AUTH payment request | `HashValue` | `ServiceID`*, `PaymentID`*, `MerchantReturnURL`*, `MerchantApprovalURL`, `MerchantUnApprovalURL`, `MerchantCallBackURL`, `Amount`*, `CurrencyCode`*, `CustIP`*, `PageTimeout`, `CardNo`, `Token` |
| Payment response (return and callback) | `HashValue2` | `TxnID`*, `ServiceID`*, `PaymentID`*, `TxnStatus`*, `Amount`*, `CurrencyCode`*, `AuthCode`, `OrderNumber`*, `Param6`, `Param7` |
| QUERY / REFUND / RSALE / CAPTURE request | `HashValue` | `ServiceID`*, `PaymentID`*, `Amount`*, `CurrencyCode`* |
| QUERY / REFUND / RSALE / CAPTURE response | `HashValue` | `TxnID`*, `ServiceID`*, `PaymentID`*, `TxnStatus`*, `Amount`*, `CurrencyCode`*, `AuthCode` |
| SOPBL forex request | `HashValue` | `ServiceID`*, `DateTime`*, `CurrencyCode`* |
| SOP (purpose not published) | `HashValue` | `ServiceID`*, `PaymentID`*, `CustIP`* |
| SOP2 (purpose not published) | `HashValue` | `ServiceID`*, `PaymentID`*, `CustEmail`*, `CustPhone`, `Token`*, `CustIP`* |
| SOPCARD (purpose not published) | `HashValue` | `ServiceID`*, `CustEmail`, `CustPhone`, `Token`*, `DateTime`* |

`*` = required by the hash library. The library keys the API request list by `QUERY`, `REVERSAL`, `CAPTURE`, `REFUND` (the merchantapi SDK passes `REVERSAL` for the hash but sends `TransactionType=RSALE`). No hash order is published for `SETTLE`.

The payment response also carries a `HashValue` field, but every official plugin and the Omnipay library verify `HashValue2` for return and callback. Use `HashValue2`.

The WHMCS plugin states the HashValue2 string directly:

```php
hash('sha256', $merchantPassword.$TxnID.$ServiceID.$PaymentID.$TxnStatus.str_replace(',','',$Amount).$CurrencyCode.$AuthCode.$OrderNumber.$Param6.$Param7);
```

Pseudo-code:

```js
import crypto from "node:crypto";

const sha = (s) => crypto.createHash("sha256").update(s, "utf8").digest("hex");
const v = (x) => (x == null ? "" : String(x));

function saleHash(pw, p) {
  return sha(pw + [p.ServiceID, p.PaymentID, p.MerchantReturnURL, p.MerchantApprovalURL,
    p.MerchantUnApprovalURL, p.MerchantCallBackURL, p.Amount, p.CurrencyCode, p.CustIP,
    p.PageTimeout, p.CardNo, p.Token].map(v).join(""));
}

function responseHash2(pw, r) {
  return sha(pw + [r.TxnID, r.ServiceID, r.PaymentID, r.TxnStatus, r.Amount, r.CurrencyCode,
    r.AuthCode, r.OrderNumber, r.Param6, r.Param7].map(v).join(""));
}

function apiRequestHash(pw, p) { // QUERY, REFUND, RSALE, CAPTURE
  return sha(pw + [p.ServiceID, p.PaymentID, p.Amount, p.CurrencyCode].map(v).join(""));
}

function apiResponseHash(pw, r) {
  return sha(pw + [r.TxnID, r.ServiceID, r.PaymentID, r.TxnStatus, r.Amount, r.CurrencyCode,
    r.AuthCode].map(v).join(""));
}
```

Gotchas:

- Hash the URL values in the exact form you send them (after replacing `&` with `;`).
- Hash `Amount` exactly as sent, for example `10.00`.
- Send `PageTimeout` in the form only if it was included in the hash, and with the same value.
- The WooCommerce plugin replaces `:` with `=>` in received values before hashing. This is plugin-specific and not described elsewhere; hash raw received values and only investigate this if a value containing `:` fails to match.

## Payment Request (SALE)

Browser form POST (auto-submit) to the payment URL. The WooCommerce plugin sends the same fields as a GET query string; POST is the pattern used by Omnipay, WHMCS, and OpenCart.

Fields from `paramvalidator/src/PaymentRequest.php` (type A = alphabetic, N = numeric, AN = alphanumeric, ANS = alphanumeric plus symbols):

| Field | Type | Max | Required | Notes |
| --- | --- | --- | --- | --- |
| `TransactionType` | A | 7 | yes | `SALE`; `AUTH` to authorise a card for later `CAPTURE` |
| `PymtMethod` | A | 3 | yes | `ANY`, `CC`, `DD`, `WA` (values used in official plugins) |
| `ServiceID` | AN | 3 | yes | Merchant Service ID |
| `PaymentID` | AN | 20 | yes | Unique per attempt |
| `OrderNumber` | AN | 40 | yes | Your order/invoice; may repeat across attempts |
| `PaymentDesc` | AN | 100 | yes | Plugins strip HTML and truncate to 100 |
| `MerchantReturnURL` | AN | 255 | yes | Browser return; `&` written as `;` |
| `Amount` | N | 15 | yes | 2 decimals, no commas |
| `CurrencyCode` | A | 3 | yes | ISO code, `MYR` |
| `HashValue` | AN | 64 | yes | SALE hash above |
| `CustIP` | AN | 45 | yes | Shopper IP (WooCommerce truncates to 20) |
| `CustName` | AN | 50 | yes | |
| `CustEmail` | AN | 60 | yes | Valid email |
| `CustPhone` | AN | 25 | yes | |
| `MerchantCallBackURL` | AN | 255 | no (strongly recommended) | Server-to-server notification |
| `MerchantApprovalURL` | AN | 255 | no | |
| `MerchantUnApprovalURL` | AN | 255 | no | |
| `PageTimeout` | N | 4 | no | Seconds; plugins default 600 to 780 |
| `MerchantName` | AN | 25 | no | |
| `LanguageCode` | A | 2 | no | PrestaShop sends `en`; Magento sends the store locale's language part (e.g. `en`). Full list not published |
| `B4TaxAmt`, `TaxAmt` | N | 15 | no | 2 decimals |
| `CustMAC` | AN | 50 | no | |
| `IssuingBank` | AN | 30 | no | |
| `CardHolder`, `CardNo`, `CardExp`, `CardCVV2` | AN/N | 30/19/6/4 | no | Merchant-hosted card fields; PCI scope, avoid unless approved |
| `BillAddr` / `ShipAddr` | ANS | 250 | no | WooCommerce sends `None` when empty |
| `BillPostal` / `ShipPostal` | AN | 15 | no | |
| `BillCity` / `ShipCity` | ANS | 100 | no | |
| `BillRegion` / `ShipRegion` | ANS | 30 | no | |
| `BillCountry` / `ShipCountry` | A | 2 | no | ISO 3166 alpha-2 |
| `SessionID` | AN | 100 | no | |
| `TokenType` | A | 3 | no | `OCP` used by WooCommerce "remember card" option |
| `Token` | ANS | 50 | if `TokenType` set (validator) | The validator rejects `TokenType` without `Token`, but WooCommerce 8.2.4 sends `TokenType=OCP` with no `Token`; for OCP follow the plugin |
| `Param6`, `Param7` | ANS | 50 | no | Echoed back and included in `HashValue2` |
| `EPPMonth` | N | 2 | no | Instalment months |
| `PromoCode` | AN | 100 | no | |

Plugins also replace `&` with ` and ` and `+` with `,` in addresses before sending.

Example form:

```html
<form id="eghl" method="post" action="https://pay.e-ghl.com/IPGSG/payment.aspx">
  <input type="hidden" name="TransactionType" value="SALE">
  <input type="hidden" name="PymtMethod" value="ANY">
  <input type="hidden" name="ServiceID" value="<SERVICE_ID>">
  <input type="hidden" name="PaymentID" value="<UNIQUE_PAYMENT_ID>">
  <input type="hidden" name="OrderNumber" value="ORD-1001">
  <input type="hidden" name="PaymentDesc" value="Order ORD-1001">
  <input type="hidden" name="MerchantReturnURL" value="https://shop.example/eghl/return">
  <input type="hidden" name="MerchantCallBackURL" value="https://shop.example/eghl/callback">
  <input type="hidden" name="Amount" value="10.00">
  <input type="hidden" name="CurrencyCode" value="MYR">
  <input type="hidden" name="CustIP" value="203.0.113.10">
  <input type="hidden" name="CustName" value="Ali Ahmad">
  <input type="hidden" name="CustEmail" value="ali@example.com">
  <input type="hidden" name="CustPhone" value="60123456789">
  <input type="hidden" name="PageTimeout" value="600">
  <input type="hidden" name="HashValue" value="<SALE_HASH>">
</form>
<script>document.getElementById("eghl").submit();</script>
```

## Payment Response (Return and Callback)

eGHL sends the same parameter set to `MerchantReturnURL` (via the shopper's browser) and to `MerchantCallBackURL` (server to server). Official plugins read them from `$_REQUEST`, so accept both POST and GET.

Fields read by official code (Omnipay `CompletePurchaseRequest.php` list, plus `RespCode` read by the PrestaShop handler). Types and lengths of response fields are not published; meanings below are only what the official code shows.

| Field | In `HashValue2` | Notes from official code |
| --- | --- | --- |
| `TransactionType` | no | `SALE` (or `AUTH`); Magento rejects anything other than `SALE` |
| `PymtMethod` | no | Method actually used (`CC`, `DD`, `WA`, ...) |
| `ServiceID` | yes (2nd) | Must equal your Service ID |
| `PaymentID` | yes (3rd) | Your request value |
| `OrderNumber` | yes (8th) | Your request value |
| `Amount` | yes (5th) | 2 decimals; WHMCS strips `,` before hashing |
| `CurrencyCode` | yes (6th) | May differ from the request currency, see Multi-Currency below |
| `TxnID` | yes (1st) | eGHL transaction ID; store it |
| `TxnStatus` | yes (4th) | `0` / `1` / `2`, see `eghl-codes.md` |
| `AuthCode` | yes (7th, optional) | Bank/card approval code; may be empty |
| `Param6`, `Param7` | yes (9th, 10th, optional) | Echo of your request values |
| `TxnMessage` | no | Text; `Buyer cancelled` means shopper cancelled |
| `IssuingBank` | no | Bank / issuer name |
| `HashValue` | n/a | Shorter response hash; plugins ignore it on return/callback |
| `HashValue2` | n/a | Verify this one |
| `RespCode` | no | Read but not used by PrestaShop; meaning not published |
| `TokenType`, `Token` | no | Card token returned when tokenisation (e.g. `OCP`) is used |
| `CardHolder`, `CardNoMask`, `CardExp`, `CardType` | no | Card details, card number masked |
| `SettleTAID`, `TID` | no | Settlement terminal / terminal IDs; meaning not further published |
| `EPPMonth`, `EPP_YN` | no | Instalment months / instalment flag |
| `PromoCode`, `PromoOriAmt` | no | Promo applied and original amount before promo |

### Multi-Currency Responses

The Magento 2.1.7 plugin accepts a response whose `CurrencyCode` differs from the order currency: it calls SOPBL (below) for rates with the order currency as base, converts `Amount / rate`, and treats a difference above `0.1` as fraud. If your account does not enable shopper currency choice, simply reject any currency mismatch. If it does, ask the account manager how foreign-currency settlement works before accepting.

Handler steps:

1. Recompute `HashValue2` and compare. Mismatch means reject (plugins force status to failed).
2. Load the local payment by `PaymentID` (or `OrderNumber`); check `ServiceID`, `Amount`, `CurrencyCode`, and `TransactionType` (`SALE`, as the Magento plugin checks).
3. Apply status idempotently: `0` paid, `2` pending (requery later), `1` failed or cancelled. Never downgrade an already paid order.
4. Callback only: respond with HTTP 200 body `OK` (official plugins: `die('OK')` / raw `OK`; Magento also returns `OK` for duplicates).
5. Return only: redirect the shopper to a success, pending, or failed page.

Callback retry schedule, source IPs, and timeout are not documented in official public sources.

Acknowledgement bodies seen in official plugins: `OK` after processing (all plugins, including duplicates and, in WooCommerce, even when the order is not found); `ERROR` on hash mismatch (PrestaShop, Magento fraud path); `Fail` (WHMCS on failed status). Only `OK` is consistently used as the success acknowledgement; what eGHL does on a non-`OK` reply is not published.

Patterns in official plugins NOT to copy:

- PrestaShop 1.6 `eGHLResponseHandler.php` takes `merchantPassword` and `merchantServiceID` from `$_REQUEST` if present, so a forged request can supply its own password and pass the hash check. Always load the password from server config only.
- WooCommerce 8.2.4 only logs an amount mismatch and still completes the order. Reject instead.
- OpenCart's `callback()` action answers `OK` without verifying anything; it settles only in the return handler. Verify and settle in the callback.
- Magento uses a MySQL `GET_LOCK` per order around settlement. Copy this idea: serialise return and callback for the same order.

## TxnStatus Values

| Value | Meaning | Source |
| --- | --- | --- |
| `0` | Success | APIBase, Omnipay, plugins |
| `1` | Failed (`TxnMessage` `Buyer cancelled` = shopper cancelled) | APIBase, Omnipay, Magento |
| `2` | Pending (Query: pending at bank if `TxnID` and `BankRefNo` or `AuthCode` present) | APIBase, Query.php |
| `9` | Reversed | Query.php |
| `10` | Refunded | Query.php |
| `15` | Authorised (AUTH, awaiting capture) | Query.php |
| `16` | Captured | Query.php |
| `31` | Reversal pending | Query.php |
| `-1` | Transaction not found | Query/Refund/Reversal |
| `-2` | System error | Query/Refund/Reversal |

Query also returns `TxnExists`: `0` means the transaction exists.

## Server-To-Server API Request Fields (QUERY, REFUND, RSALE, CAPTURE)

From `paramvalidator/src/{Query,Refund,Reversal,Capture}Request.php`. All four take exactly these fields:

| Field | Type | Max | Required | QUERY | REFUND | RSALE | CAPTURE |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `TransactionType` | A | 7 / 6 / 5 / 7 | yes | `QUERY` | `REFUND` | `RSALE` | `CAPTURE` |
| `PymtMethod` | A | 3 | yes | original method | `CC` only | original method | `CC` |
| `ServiceID` | AN | 3 | yes | yours | yours | yours | yours |
| `PaymentID` | AN | 20 | yes | original | original | original | original |
| `Amount` | N | 15 | yes | original, 2 decimals | refund amount, 2 decimals, up to original | original (no partial) | amount to capture, 2 decimals |
| `CurrencyCode` | A | 3 | yes | original | original | original | original |
| `HashValue` | AN | 64 | yes | `sha256(pw + ServiceID + PaymentID + Amount + CurrencyCode)` | same | same | same |

Whether CAPTURE accepts a partial amount is not documented in official public sources.

## Server-To-Server API Response Fields

The body is URL-encoded (`parse_str` in `APIBase.php`). The merchantapi SDK verifies `HashValue` with a strict (case-sensitive) compare; plugins compare case-insensitively.

| Field | Used by | Notes |
| --- | --- | --- |
| `TxnStatus` | all | See `eghl-codes.md`; `0` success, `1` fail, `2` pending, query-only states, `-1` not found, `-2` system error |
| `TxnExists` | QUERY | `0` = transaction exists |
| `TxnID` | all, in hash | eGHL transaction ID |
| `ServiceID`, `PaymentID`, `Amount`, `CurrencyCode` | all, in hash | Echo of the request |
| `AuthCode` | all, in hash (optional) | |
| `BankRefNo` | QUERY | Bank reference; with `TxnID` and `TxnStatus=2` means pending at bank |
| `TxnMessage` | all | Text |
| `HashValue` | all | Verify before use |

The SDK aborts on any non-200 HTTP status. Other response fields, if any, are not published.

## Query (Status Requery)

Server-side form POST (`application/x-www-form-urlencoded`) to the payment URL:

| Field | Value |
| --- | --- |
| `TransactionType` | `QUERY` |
| `PymtMethod` | Method of the original payment, e.g. `CC`, `DD`, `WA` |
| `ServiceID` | Your Service ID |
| `PaymentID` | Original `PaymentID` |
| `Amount` | Original amount, 2 decimals |
| `CurrencyCode` | Original currency |
| `HashValue` | `sha256(pw + ServiceID + PaymentID + Amount + CurrencyCode)` |

The transaction must match all of `PymtMethod`, `ServiceID`, `PaymentID`, `Amount`, `CurrencyCode`. The response body is URL-encoded (`parse_str` in APIBase). Verify its `HashValue` with the API response order before trusting it. Response fields used by the SDK: `TxnExists`, `TxnStatus`, `TxnID`, `BankRefNo`, `AuthCode`, `TxnMessage`, `HashValue`.

## Refund

Same POST with `TransactionType=REFUND`, same five fields and request hash.

- Only `PymtMethod=CC` transactions are refundable online.
- Partial refund allowed; amount must not exceed the original.
- Query first: `TxnStatus=10` means already refunded.
- Success is `TxnStatus=0`; pending `2`.

## Reversal (Void)

Same POST with `TransactionType=RSALE`.

- Only before bank settlement; eGHL advises within one hour of the sale.
- Reversal of a settled transaction returns failed; partial reversal returns not found.
- Query first: `9` reversed, `31` reversal pending.

## Capture

Same POST with `TransactionType=CAPTURE` for a card payment sent with `TransactionType=AUTH`.

- Query first: must be `15` (authorised), not `16` (captured).

## Settlement Request

`paramvalidator/src/SettlementRequest.php` defines:

| Field | Type | Max | Required | Notes |
| --- | --- | --- | --- | --- |
| `TransactionType` | A | 6 | yes | `SETTLE` |
| `ServiceID` | AN | 3 | yes | |
| `SettleTAID` | N | 10 | yes | Also returned on payment responses |
| `SettleAmount` | N | 15 | yes | 2 decimals |
| `SettleTxnCount` | N | 3 | yes | |
| `HashValue` | AN | 64 | yes | Hash order not published |

Its purpose, response, and hash order are not documented in official public sources; do not implement without the merchant API spec.

## Forex Rates (SOPBL)

From the Magento 2.1.7 plugin (`Helper/Data.php`, `getForexData`):

- `GET <payment URL>?ServiceID=<SERVICE_ID>&TransactionType=SOPBL&HashValue=<hash>&sdk=<label>&CurrencyCode=<BASE>&DateTime=<unix seconds>`
- `HashValue = sha256(pw + ServiceID + DateTime + CurrencyCode)`.
- `sdk` is a free label (Magento sends `Magento 2.x`); it is not hashed.
- Response is JSON; the plugin reads `Forex`, an array of objects with `curr` (currency code) and `rate`. Converted base amount = foreign `Amount / rate`.
- No other response fields, rate limits, or error format are published.

## Amount Rules

- Major unit (ringgit), exactly 2 decimals, no thousands separator: `123.10`, not `123.1`, `1,000.00`, or `12310`.
- Official plugins format with `sprintf("%.02f")` or `number_format($a, 2, '.', '')`.
- Store expected amounts as integer sen locally and format only at the boundary.

## Recurring, Tokenisation, Instalments

- `TokenType`/`Token` and `TokenType=OCP` (card remembered with shopper consent) appear in official code.
- `EPPMonth`, `EPP_YN`, `PromoCode`, `PromoOriAmt` relate to instalment and promo.
- Request/response details for recurring and token charges are not documented in official public sources.

## Sandbox Test Data

- Test URL and demo credentials above. Test cards and test bank accounts are not documented in official public sources; ask your account manager for UAT data.

## Rate Limits, IP Allowlists, Errors

- Rate limits: not documented in official public sources.
- Callback source IPs: not documented in official public sources.
- Error codes beyond `TxnStatus` and `TxnMessage` text: not documented in official public sources.

## Common Errors

- Hash mismatch on request: wrong field order, `&` not replaced by `;`, amount not 2 decimals, `PageTimeout` sent but not hashed, test password on live URL.
- Hash mismatch on response: omitted `OrderNumber`/`Param6`/`Param7`, amount reformatted before hashing, wrong environment password.
- Duplicate `PaymentID`: generate a new one for every attempt.
- Callback not acknowledged: endpoint returns HTML, redirects, or errors instead of `OK`.
- Refund rejected: original method was not `CC`, or amount exceeds original.
