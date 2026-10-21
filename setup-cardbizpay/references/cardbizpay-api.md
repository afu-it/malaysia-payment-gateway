# CardBizPay API Reference

Use this reference for factual CardBizPay API guidance. It distils the official developer guide published as `guide` posts on `cardbizpay.my` (checked 2026-10-21). The Open Payment API (card tokens, pre-auth) is in `cardbizpay-opa.md`. Confirm anything marked "Not documented" with CardBizPay in staging.

## Source URLs

- Developer hub: `https://cardbizpay.my/developer/`
- Merchant API page: `https://cardbizpay.my/developer/merchant-api/` (the page body is currently empty; the Merchant API sections live as guide posts, listed below)
- Open Payment API page: `https://cardbizpay.my/developer/open-payment-api/`
- Guide index (WordPress REST, 134 posts): `https://cardbizpay.my/wp-json/wp/v2/guide?per_page=100`
- Merchant API sections: `https://cardbizpay.my/guide/1-introduction/`, `/guide/1-1-pre-requisite/`, `/guide/1-2-transaction-flow-diagram/`, `/guide/1-3-pages-preview/`, `/guide/2-merchant-integration/`, `/guide/2-1-url/`, `/guide/2-2-security-control/`, `/guide/2-2-1-hash-signature/`, `/guide/2-2-2-signature-calculation/`, `/guide/2-3-http-methods/`, `/guide/2-4-code-table/`, `/guide/2-4-1-status-code/`, `/guide/2-4-2-payment-status-code/`, `/guide/3-payment-apis/`, `/guide/3-1-payment/`, `/guide/3-1-1-request/`, `/guide/3-1-2-response/`, `/guide/3-2-payment-enquiry/`, `/guide/3-3-2-request/` (titled 3.2.1), `/guide/3-2-2-reponse/`, `/guide/3-3-void-payment/`, `/guide/3-3-1-request/`, `/guide/3-3-2-reponse/`, `/guide/3-4-void-payment-enquiry/`, `/guide/3-4-1-request/`, `/guide/3-4-2-reponse/`, `/guide/3-5-refund-payment/`, `/guide/3-5-1-request/`, `/guide/3-5-2-reponse/`, `/guide/3-6-refund-status-enquiry/`, `/guide/3-6-1-request/`, `/guide/3-6-2-reponse/`, `/guide/3-7-mdr-enquiry/`, `/guide/3-7-1-request/`, `/guide/3-7-2-reponse/` (all under `https://cardbizpay.my`)

## Products

- **Merchant API** (`/Payment/*`): hosted CardBizPay payment page for cards, FPX (B2C/B2B), e-Wallet, DuitNow; saved-card option on the page keyed by `CustId`; enquiry, void, refund, MDR (fee) enquiry.
- **Open Payment API** (`/opa/*`): card tokenization (add/remove card), token sale, pre-authorization, capture, pre-auth cancel, void, refund, transaction enquiry. See `cardbizpay-opa.md`.
- No-code: Payment Links and eMOTO (mail/telephone order) in the Merchant Portal. Shopping cart plugins are mentioned on the pricing page, but no plugin is published on WordPress.org or GitHub (checked 2026-10-21).

API availability is per merchant profile: "API access may be enabled or disabled based on the configuration set by the CardBizPay system administrator."

## Environments

All requests are HTTP form submissions to:

| Environment | Base URL |
|---|---|
| Staging | `https://uat.cardbizpay.com/paymentgateway` |
| Production | `https://www.cardbizpay.com/paymentgateway` |

Merchant Portal: `https://cardbizpay.com/Merchant` (linked from cardbizpay.my).

## Endpoint Catalogue

| API name | Method | Path | Purpose | Family |
|---|---|---|---|---|
| Pymt | POST | `/Payment/Pymt` | Redirect buyer to hosted payment page | Merchant |
| PymtEnq | GET | `/Payment/PymtEnq` | Payment status enquiry | Merchant |
| PymtVoid | POST | `/Payment/PymtVoid` | Void (same-day cancel, 100%) | Merchant |
| PymtVoidEnq | GET | `/Payment/PymtVoidEnq` | Void status enquiry | Merchant |
| Refund | POST | `/Payment/Refund` | Full or partial refund | Merchant |
| RefundEnq | GET | `/Payment/RefundEnq` | Refund status enquiry | Merchant |
| MdrEnq | GET | `/Payment/MdrEnq` | Processing fee (MDR) of a transaction | Merchant |
| AddCard | POST | `/opa/AddCard` | Register a card, get `CardTokenId` | OPA |
| RemoveCard | POST | `/opa/RemoveCard` | Delete a card token | OPA |
| Sale | POST | `/opa/Sale` | Charge a token (auth + capture) | OPA |
| PreAuth | POST | `/opa/PreAuth` | Hold amount on a token | OPA |
| Capture | POST | `/opa/Capture` | Capture a pre-auth | OPA |
| PreAuthCancel | POST | `/opa/PreAuthCancel` | Release a pre-auth | OPA |
| Void | POST | `/opa/Void` | Void a captured transaction before settlement | OPA |
| Refund | POST | `/opa/Refund` | Refund a settled transaction | OPA |
| TrxnEnq | GET | `/opa/TrxnEnq` | Transaction enquiry (last 90 days) | OPA |

Full path = Base URL + Path, for example `https://uat.cardbizpay.com/paymentgateway/Payment/Pymt`.

No other endpoints are documented (no recurring schedule API, payout, settlement report, or webhook registration API). Recurring/subscription billing is marketed; the documented building block is the OPA token `Sale`.

## Signature (Security Control)

Every request and every response carries `Signature`. CardBizPay authenticates each request with it; the merchant must (i) check the payment amount from CardBizPay and (ii) compare CardBizPay's signature with its own computed one. Each merchant gets a unique Secret Key shared only with CardBizPay.

Algorithm (sections 2.2.1 and 2.2.2):

1. Take the fields of the request or response **in the order of the documented field table**.
2. Drop `Signature`. Drop any optional field whose value is empty.
3. Concatenate the remaining **values only** (no names, no separators) into one string.
4. Compute HMAC-SHA512 over that string, keyed with the Secret Key.
5. Base64-encode the raw 64-byte digest.

Official example (Merchant API guide, reproduces exactly):

```text
MerchantNo = "1000000000000001", OrderId = "M123456789", Amount = "10.50"
string to sign: 1000000000000001M12345678910.50
key:            Ws123456Ws987654
signature:      ++lLP7XD4fK3t7PsBz50LdVU35yutApDvZdmlltNXuBsrQnF0B/ed1uxAjSN+3anRjJWrDelYCZoWbsyTzAuMg==
```

The Open Payment API copy of the same example prints a different result (`x3v2wH3a...`) that does not reproduce with the stated inputs; treat the Merchant API value above as the correct vector.

Pseudo-code:

```text
def sign(fields_in_doc_order, optional_names, secret):
    parts = []
    for name, value in fields_in_doc_order:
        if name == "Signature": continue
        if name in optional_names and value == "": continue
        parts.append(value)
    mac = HMAC(key=utf8(secret), msg=utf8("".join(parts)), hash=SHA512).digest()
    return base64(mac)

def verify(posted_form, response_table, secret):
    fields = [(n, posted_form.get(n, "")) for n in response_table]
    expected = sign(fields, optional_names_of(response_table), secret)
    return constant_time_equal(expected, posted_form["Signature"])
```

Notes:

- Character encoding is not documented; UTF-8 is the safe assumption for ASCII-only values. Test non-ASCII names (`CustName`, `TrxnDesc`) in staging.
- Mandatory fields that are empty are not explicitly addressed; mandatory fields should never be empty.
- For responses, hash the values as received. Do not trim, reformat amounts, or decode differently from what was posted.
- In OPA responses, the repeating "Transaction List" is excluded from the signature.

## Payment (Pymt): hosted payment page

Flow (section 1.2): merchant posts the form to CardBizPay; CardBizPay validates parameters and signature; on error the payment page shows the error; otherwise the buyer picks a method (and, for cards, a saved card plus CVV or a new card, optionally saved). CardBizPay sends the payment to the bank host, then sends an HTTP POST with the response fields to the merchant's Payment Return URL ("Merchant Callback Page"). A cancelled payment is also posted back.

Request: `POST [Base URL]/Payment/Pymt` (browser form post).

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | Assigned by CardBizPay |
| ReqDate | String | 14 | M | `yyyyMMddHHmmss` |
| OrderId | String | 25 | M | Unique merchant order number |
| TrxnAmount | String | 10 | M | Up to 2 decimals, no thousand separators |
| TrxnDesc | String | 500 | M | Transaction description |
| PymtMethodSelect | String | 50 | O | Empty = all subscribed methods; `Card`, `FPX` (B2B and B2C), `FPX B2C`, `FPX B2B`, `e-Wallet`, `DuitNow`; comma-separated, for example `Card, FPX` |
| CustId | String | 50 | O | Unique per buyer; key for saving and retrieving cards |
| CustName | String | 300 | M | Buyer name |
| CustEmail | String | 300 | M | Supply to avoid declines |
| CustMobile | String | 20 | O | Required if e-Wallet is subscribed |
| CustIdentityNo | String | 40 | O | Payer ID for FPX identity verification; no `/`, `&`, `'` |
| Signature | String | - | M | See Signature |

Legend: M mandatory; O optional, value can be empty but the parameter must exist.

Signing order: `MerchantNo, ReqDate, OrderId, TrxnAmount, TrxnDesc, PymtMethodSelect, CustId, CustName, CustEmail, CustMobile, CustIdentityNo` (skip empty optionals).

The official HTML example has stray spaces inside some `name` attributes (`" ReqDate"`, `" PymtMethodSelect"`); use the exact names from the table.

Response (posted to the Payment Return URL as form fields):

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| RespCode | String | 2 | M | See Status Codes |
| RespMessage | String | 500 | M | |
| Api | String | 30 | M | API name |
| MerchantNo | String | 16 | M | |
| OrderId | String | 25 | M | |
| TrxnAmount | String | 10 | M | |
| PymtMethod | String | 100 | O | For example `Credit / Debit Cards`, `FPX`, `e-Wallet - Boost`, `e-Wallet - AliPay` |
| TrxnId | String | 20 | O | Gateway transaction ID, unique, needed for void and refund |
| TrxnDate | String | 14 | O | `yyyyMMddHHmmss` |
| BankAccName | String | 50 | O | FPX: buyer bank account name |
| BankBranch | String | 30 | O | FPX: buyer bank branch code |
| MaskedCardNo | String | 16 | O | `123456xxxxxx7890`; visibility depends on merchant settings |
| StatusCode | String | 1 | O | Payment Status Code |
| StatusDesc | String | 50 | O | |
| Signature | String | - | M | |

Verify order: `RespCode, RespMessage, Api, MerchantNo, OrderId, TrxnAmount, PymtMethod, TrxnId, TrxnDate, BankAccName, BankBranch, MaskedCardNo, StatusCode, StatusDesc`.

Paid only when: signature valid, `RespCode == "00"`, `StatusCode == "A"`, `MerchantNo` is yours, `OrderId` matches a local pending order, `TrxnAmount` equals the expected amount.

## Payment Enquiry (PymtEnq)

Use when there is no response after a period of time, a connection problem, or other problems.

Request: `GET [Base URL]/Payment/PymtEnq`

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | |
| ReqDate | String | 14 | M | `yyyyMMddHHmmss` |
| OrderId | String | 25 | M | Same as Pymt |
| TrxnAmount | String | 10 | M | Same value as Pymt |
| Signature | String | - | M | |

Response: same fields and order as the Pymt response (section 3.2.2).

Whether PymtEnq answers directly in the HTTP response or posts to the Return URL is not documented (only MdrEnq is stated to answer directly, with a JSON example). Test in staging and handle both.

## Void (PymtVoid)

Cancels a payment on the **same day** it was made; 100% returned to the payer.

Request: `POST [Base URL]/Payment/PymtVoid`

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | |
| ReqDate | String | 14 | M | |
| OrderId | String | 25 | M | Same as Pymt |
| TrxnAmount | String | 10 | M | Same value as Pymt |
| TrxnId | String | 20 | M | From the approved payment |
| Signature | String | - | M | |

Response (section 3.3.2):

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| RespCode | String | 2 | M | |
| RespMessage | String | 500 | M | |
| Api | String | 30 | M | |
| MerchantNo | String | 16 | M | |
| OrderId | String | 25 | M | |
| TrxnAmount | String | 10 | M | Voided amount |
| TrxnId | String | 20 | O | Original payment transaction ID |
| TrxnDate | String | 14 | O | Void date |
| StatusCode | String | 1 | O | Expect `V` |
| StatusDesc | String | 50 | O | |
| TrxnIdNew | String | 20 | O | Void transaction ID |
| Signature | String | - | M | |

## Void Enquiry (PymtVoidEnq)

Request: `GET [Base URL]/Payment/PymtVoidEnq` with the same fields as PymtVoid (`MerchantNo, ReqDate, OrderId, TrxnAmount, TrxnId, Signature`). Response: same fields and order as the PymtVoid response.

## Refund

Returns funds to the original method (card, FPX, e-Wallet). Full refund or multiple partial refunds up to the paid amount.

Request: `POST [Base URL]/Payment/Refund`

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | |
| ReqDate | String | 14 | M | |
| OrderId | String | 25 | M | Original order |
| TrxnAmount | String | 10 | M | Original amount, same value as Pymt |
| TrxnId | String | 20 | M | From the approved payment |
| RefundId | String | 20 | M | Unique per refund request; needed for RefundEnq |
| RefundAmount | String | 10 | M | Less than or equal to the paid amount |
| Remark | String | 300 | O | Reason |
| Signature | String | - | M | |

Response (section 3.5.2), verify order:

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| RespCode | String | 2 | M | |
| RespMessage | String | 500 | M | |
| Api | String | 30 | M | |
| MerchantNo | String | 16 | M | |
| OrderId | String | 25 | M | |
| TrxnAmount | String | 10 | M | Original amount |
| TrxnId | String | 20 | M | Original payment ID |
| RefundId | String | 20 | M | |
| TrxnDate | String | 14 | O | |
| StatusCode | String | 1 | O | Expect `Q` or `R` |
| StatusDesc | String | 50 | O | |
| TrxnIdNew | String | 20 | O | Refund transaction ID |
| AmtRemain | String | - | O | Remaining paid amount after refund |
| AmtRefunded | String | - | O | Total refunded amount |
| Signature | String | - | M | |

The ASP sample reads `TrxnDate` right after `MerchantNo` and again later; follow the table order above.

## Refund Enquiry (RefundEnq)

Request: `GET [Base URL]/Payment/RefundEnq`

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | |
| ReqDate | String | 14 | M | |
| OrderId | String | 25 | M | Original order |
| TrxnAmount | String | 10 | M | Original amount |
| TrxnId | String | 20 | M | |
| RefundId | String | 20 | M | Original refund ID |
| RefundAmount | String | 10 | M | |
| Signature | String | - | M | |

Response: same fields and order as the Refund response.

## MDR Enquiry (MdrEnq)

Returns the processing fee of a transaction. Response is returned directly to the requester, not posted to the Return URL.

Request: `GET [Base URL]/Payment/MdrEnq` with `MerchantNo, ReqDate, OrderId, TrxnAmount, Signature` (same rules as PymtEnq).

Response (JSON in the official example):

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| RespCode | String | 2 | M | |
| RespMessage | String | 500 | M | |
| Api | String | 30 | M | `MdrEnq` |
| MerchantNo | String | 16 | M | |
| OrderId | String | 25 | M | |
| MdrDesc | String | 50 | M | For example `Normal Rate` |
| MdrRatePerc | String | 10 | O | Percentage, 4 decimals, `1.5000` |
| MdrRateAmt | String | 10 | O | Flat fee, 2 decimals, `0.70` |
| MdrTtlAmt | String | 10 | O | Fee charged |
| Signature | String | - | M | |

## Status Codes (RespCode)

Merchant API table (updated 2025-12-12):

| Code | Message | Meaning |
|---|---|---|
| 00 | Approved | Success or request without error |
| 03 | Invalid Merchant | Merchant number does not exist |
| 04 | Account suspended | |
| 05 | Invalid parameters | One or more parameters invalid or empty |
| 06 | Invalid data format | |
| 07 | Invalid Signature | |
| 08 | Invalid card info | Encrypted card number could not be decrypted or invalid |
| 09 | Payment not allowed | Payment not allowed for this merchant |
| 10 | Void not allowed | |
| 11 | Refund not allowed | |
| 12 | Enquiry not allowed | |
| 13 | Payment has made before | Order already paid |
| 14 | No payment method subscribed | |
| 15 | Selected payment method not subscribed | `PymtMethodSelect` contains an unsubscribed method |
| 20 | Transaction unsuccessful | Payment, void or refund declined |
| 21 | Overlimit per transaction | |
| 22 | Duplicate order number | `OrderId` must be unique |
| 23 | No record | |
| 24 | Refund amount too large | Larger than remaining paid amount |
| 25 | Refund amount too small | Below minimum limit |
| 99 | Application error | Server error or misconfiguration |

OPA adds `26`, `30` to `33`; see `cardbizpay-opa.md`.

## Payment Status Codes (StatusCode)

| Code | Meaning | Remark |
|---|---|---|
| A | Payment Approved | Final paid |
| D | Payment Declined | Final failed |
| N | Pending for payment | Cancelled if not paid within 3 days |
| W | Awaiting for status | Not final; requery |
| V | Payment voided | |
| C | Payment cancelled | |
| Q | Payment partially refunded | |
| R | Payment fully refunded | |

## Amounts, Dates, Limits

- Amount strings in ringgit, up to 2 decimals, no thousand separators. No currency parameter; MYR is implied, multi-currency is not documented.
- Per-transaction limits exist (`21`) and refunds have a minimum (`25`); values are per merchant and not published.
- Payment page session: 15 minutes.
- Rate limits and IP allowlists: Not documented in official docs (checked 2026-10-21).
- Sandbox test cards, FPX test banks, and test credentials: Not documented publicly; staging credentials come from CardBizPay.

## Return URL, Callback, Notify

- Merchant API: the Payment Return URL (also called Callback URL) is given to the CardBizPay administrator at onboarding and configured on the merchant profile. Changes go through the administrator, or through the Merchant Portal if you have a login. It is not a request field.
- Open Payment API: a Notify URL is registered the same way and receives the API result "in the case of unexpected interruption". Per-request `ReturnUrl` drives the browser.
- Whether the Return URL post is a browser auto-post or a server-to-server call, retry behaviour, and expected acknowledgement are not documented. Build the handler to accept both, verify the signature, and requery `PymtEnq` before final settlement when in doubt.

## Doc Defects To Confirm

- Open Payment API signature example output does not reproduce; the Merchant API one does.
- Stray spaces in example `name` attributes (`" ReqDate"`, `" TrxnId"`, `" RefundId"`).
- Example HTML is not valid (missing `>` after the Signature input, curly quotes); do not copy it verbatim.
- PymtEnq section is published under slug `3-3-2-request` with title 3.2.1.
- The Merchant API page currently shows no section links; the sections are still published as guide posts.

## Common Errors

- `07` Invalid Signature: wrong field order, included an empty optional field, hashed hex instead of base64, wrong key or environment, amount string differs from the one posted.
- `05` Invalid parameters: an optional parameter omitted entirely (it must exist, even empty) or a mandatory field empty.
- `15`: `PymtMethodSelect` names a method not subscribed on the profile.
- `22` or `13`: `OrderId` reused.
- `12`, `10`, `11`: enquiry, void or refund not enabled on the profile; ask CardBizPay.
