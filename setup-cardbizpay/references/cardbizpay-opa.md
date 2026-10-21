# CardBizPay Open Payment API (OPA)

Card tokenization, merchant-initiated charges, and pre-authorization. Read `cardbizpay-api.md` first for environments, the signature algorithm, and the Merchant API. Source: the Open Payment API guide on `https://cardbizpay.my/developer/open-payment-api/` (checked 2026-10-21).

## Source URLs

All under `https://cardbizpay.my/guide/`: `1-introduction-2/`, `1-1-pre-requisite-of-merchant-integration/`, `2-merchant-integration-2/`, `2-1-url-2/`, `2-2-security-control-2/`, `2-2-1-hash-signature-2/`, `2-2-2-signature-calculation-2/`, `2-3-http-methods-2/`, `2-4-code-table-2/`, `2-4-1-status-code-2/`, `2-4-2-payment-status-code-2/`, `payment-apis/`, `3-1-add-card/`, `3-1-1-request-2/`, `3-1-1-response/`, `3-2-remove-card/`, `3-2-1-request/`, `3-2-1-response/`, `3-3-sale/`, `3-3-1-request-2/`, `3-3-2-response/`, `3-4-pre-authorization/`, `3-4-1-request-2/`, `3-4-2-response/`, `3-5-capture-preauth/`, `3-5-1-request-2/`, `3-5-2-response/`, `3-6-pre-auth-cancellation/`, `3-6-1-request-2/`, `3-6-2-response/`, `3-7-transaction-enquiry/` (titled 3.7 Void), `3-7-1-request-2/`, `3-7-2-response/`, `3-8-refund/`, `3-8-1-request/`, `3-8-2-response/`, `3-9-transaction-enquiry/`, `3-9-1-request/`, `3-9-2-response/`.

## Purpose And Prerequisites

- Lets the merchant hold money on a card when the order is created and charge it at fulfilment, plus charge saved cards.
- Prerequisites: merchant profile from the CardBizPay administrator, Merchant No and Access Key, and a **Notify URL** registered with CardBizPay "to post back API response result in the case of unexpected interruption".
- Base URLs are the same as the Merchant API (staging `https://uat.cardbizpay.com/paymentgateway`, production `https://www.cardbizpay.com/paymentgateway`).
- Same signature algorithm: base64(HMAC-SHA512(secret, values in field order, skipping `Signature` and empty optional fields)). For responses with a Transaction List, **exclude the Transaction List** from the signature.
- `ReqDate` (`yyyyMMddHHmmss`) more than 60 minutes earlier or later than host time is rejected.
- `ReturnUrl` is the browser (frontend) redirect for the hosted page. For RemoveCard, Capture, PreAuthCancel, Void, Refund and TrxnEnq: "If needed a direct response, left this field empty." AddCard, Sale and PreAuth always need it (OTP / hosted step).
- Legend: M mandatory, C conditional, O optional (value can be empty but the parameter must exist).

## Typical Flows

- Save a card: AddCard (hosted, OTP if the subscribed application needs it) -> store `CardTokenId` + `MaskedCardNo` against your `CustId`.
- Charge now: Sale with `CustId` + `CardTokenId` (cardholder completes OTP) -> `PymtStatus` `A`.
- Hold then charge: PreAuth (status `P`, issuer typically holds 7 days) -> Capture (amount <= pre-auth total) or PreAuthCancel (status `T`).
- Undo: Void before settlement (all transactions with that `OrderId` are voided) or Refund after settlement (transactions generally settle the same day).
- Reconcile: TrxnEnq (last 90 days).

## AddCard

`POST [Base URL]/opa/AddCard`. OTP depends on the payment application subscribed on the merchant account.

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | |
| ReqDate | String | 14 | M | +/- 60 min |
| RequestId | String | 50 | M | Merchant request ID |
| CustId | String | 50 | M | Merchant customer ID |
| CustName | String | 300 | M | |
| CustEmail | String | 300 | M | Supply to avoid declines |
| ReturnUrl | String | 300 | M | Browser return |
| Signature | String | - | M | |

Response:

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| RespCode | String | 2 | M | |
| RespMessage | String | 500 | M | |
| Api | String | 30 | M | |
| MerchantNo | String | 16 | M | |
| RequestId | String | 50 | M | |
| CustId | String | 50 | M | |
| CardTokenId | String | 20 | O | Generated token |
| MaskedCardNo | String | 16 | O | `123456xxxxxx5678` |
| Signature | String | - | M | |

## RemoveCard

`POST [Base URL]/opa/RemoveCard`

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | |
| ReqDate | String | 14 | M | |
| CustId | String | 50 | M | |
| CardTokenId | String | 20 | M | |
| ReturnUrl | String | 300 | M | Empty for direct response |
| Signature | String | - | M | |

Response: `RespCode, RespMessage, Api, MerchantNo, Signature`.

## Sale

`POST [Base URL]/opa/Sale`. Authorization and capture in one step; cardholder completes OTP.

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | |
| ReqDate | String | 14 | M | |
| RequestId | String | 50 | M | |
| CustId | String | 50 | M | |
| OrderId | String | 50 | M | |
| CardTokenId | String | 20 | M | |
| Amount | String | 10 | M | Up to 2 decimals, no separators |
| TrxnDesc | String | 150 | O | |
| ReturnUrl | String | 300 | M | |
| Signature | String | - | M | |

Response (`RespCode` `00` = request accepted):

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| RespCode | String | 2 | M | |
| RespMessage | String | 500 | M | |
| Api | String | 30 | M | |
| MerchantNo | String | 16 | M | |
| RequestId | String | 50 | M | |
| CustId | String | 50 | M | |
| OrderId | String | 50 | M | |
| CardTokenId | String | 20 | M | |
| TrxnId | String | 20 | C | Transaction ID |
| PymtStatus | String | 1 | C | Payment Status Code |
| Signature | String | - | M | |

## PreAuth

`POST [Base URL]/opa/PreAuth`. Holds an amount; OTP depends on the subscribed application. Issuers generally hold for 7 days.

Fields: `MerchantNo (16 M), ReqDate (14 M), RequestId (50 M), CustId (50 M), OrderId (50 M), CardTokenId (20 M), Amount (10 M), ReturnUrl (300 M), Signature (M)`.

`Amount` for bidding/auction: first bid is the full amount; later bids by the same bidder are the increment (1st bid RM500 -> `500`; 2nd bid RM600 -> `100`).

Response: same fields and order as the Sale response (`TrxnId` described as unique transaction ID; `PymtStatus` expected `P`).

The official PreAuth HTML example posts to `/opa/AddCard`; that is a doc error, use `/opa/PreAuth`.

## Capture

`POST [Base URL]/opa/Capture`. Capture amount can be equal to or less than the total pre-authorized amount.

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | |
| ReqDate | String | 14 | M | |
| CustId | String | 50 | M | |
| OrderId | String | 50 | M | |
| Amount | String | 10 | M | Total to capture |
| TrxnDesc | String | 150 | O | |
| ReturnUrl | String | 300 | M | Empty for direct response |
| Signature | String | - | M | |

Response (shared shape for Capture, PreAuthCancel, Void):

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| RespCode | String | 2 | M | `00` = accepted |
| RespMessage | String | 500 | M | |
| Api | String | 30 | M | |
| MerchantNo | String | 16 | M | |
| CustId | String | 50 | M | |
| OrderId | String | 50 | M | |
| TrxnCount | Integer | - | M | Number of transactions |
| Transaction List | | | | Repeats per transaction; excluded from signature |
| - TrxnId | String | 20 | C | |
| - RequestId | String | 50 | C | |
| - CardTokenId | String | 20 | C | |
| - Amount | String | 10 | C | Captured amount |
| - PymtStatus | String | 1 | C | |
| Signature | String | - | M | Over the non-list fields only |

Signature order: `RespCode, RespMessage, Api, MerchantNo, CustId, OrderId, TrxnCount`.

The wire encoding of the Transaction List (repeated form fields, indexed names, or JSON) is not documented; the ASP sample just reads `TrxnId`, `RequestId`, `CardTokenId`, `Amount`, `PymtStatus` twice. Confirm in staging.

## PreAuthCancel

`POST [Base URL]/opa/PreAuthCancel`. Releases the hold.

Fields: `MerchantNo (16 M), ReqDate (14 M), CustId (50 M), OrderId (50 M), ReturnUrl (300 M, empty for direct response), Signature (M)`.

Response: Capture response shape; `PymtStatus` expected `T`.

## Void

`POST [Base URL]/opa/Void`. Cancels a previous successful transaction before it settles. All transactions with the same `OrderId` are voided.

Fields: `MerchantNo (16 M), ReqDate (14 M), CustId (50 M), OrderId (50 M, from the successful transaction), Amount (10 M, total captured amount submitted earlier), ReturnUrl (300 M, empty for direct response), Signature (M)`.

Response: Capture response shape; `PymtStatus` expected `V`.

## Refund

`POST [Base URL]/opa/Refund`. Refunds a settled transaction (generally settled the same day as the transaction). Partial refunds allowed; total cannot exceed the transaction amount.

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | |
| ReqDate | String | 14 | M | |
| CustId | String | 50 | M | |
| OrderId | String | 50 | M | From the successful transaction |
| Amount | String | 10 | M | Amount of the successful transaction |
| RefundAmt | String | 10 | M | Must not exceed the transaction amount |
| ReturnUrl | String | 300 | M | Empty for direct response |
| Signature | String | - | M | |

Response:

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| RespCode | String | 2 | M | `00` = accepted |
| RespMessage | String | 500 | M | |
| Api | String | 30 | M | |
| MerchantNo | String | 16 | M | |
| CustId | String | 50 | M | |
| OrderId | String | 50 | M | |
| RefundAmt | String | 10 | M | Requested refund |
| RefundedAmtTtl | String | 10 | M | Total refunded |
| TrxnCount | Integer | - | M | |
| Transaction List | | | | `TrxnId, RequestId, CardTokenId, Amount, RefundedAmt, PymtStatus` (all C); excluded from signature |
| Signature | String | - | M | |

Signature order: `RespCode, RespMessage, Api, MerchantNo, CustId, OrderId, RefundAmt, RefundedAmtTtl, TrxnCount`.

OPA Refund has no `RefundId`; prevent duplicate submissions on your side (store a refund record and lock before calling).

## TrxnEnq

`GET [Base URL]/opa/TrxnEnq` (the official example form uses `method="post"`; confirm which the host accepts). Only the last 90 days are retrievable.

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| MerchantNo | String | 16 | M | |
| ReqDate | String | 14 | M | |
| CustId | String | 50 | O | |
| OrderId | String | 50 | O | |
| TrxnId | String | 20 | O | |
| DateStart | String | 14 | O | Empty = today 0000 hrs |
| DateEnd | String | 14 | O | Empty = now |
| ReturnUrl | String | 300 | M | Empty for direct response |
| Signature | String | - | M | |

Response:

| Field | Type | Size | M/O | Notes |
|---|---|---|---|---|
| RespCode | String | 2 | M | |
| RespMessage | String | 500 | M | |
| Api | String | 30 | M | |
| MerchantNo | String | 16 | M | |
| TrxnCount | Integer | - | M | |
| Transaction List | | | | `TrxnId, TrxnDate, RequestId, CustId, OrderId, CardTokenId, Amount, PymtStatus` (all C); excluded from signature |
| Signature | String | - | M | |

Signature order: `RespCode, RespMessage, Api, MerchantNo, TrxnCount`.

## OPA Status Codes (RespCode)

`00` Approved; `03` Invalid Merchant; `04` Account suspended; `05` Invalid parameters; `06` Invalid data format; `07` Invalid Signature; `08` Invalid card info; `10` Void not allowed; `11` Refund not allowed; `12` Enquiry not allowed; `20` Transaction unsuccessful (payment or pre-auth declined); `21` Overlimit per transaction; `23` No record; `24` Refund amount too large; `25` Refund amount too small; `26` Not same amount (must equal the previous successful transaction); `30` Add card unsuccessful; `31` CustId and CardTokenId not matched; `32` CardTokenId not found; `33` Card Token is not valid (removed, expired or invalid; cardholder must add the card again); `99` Application error.

## OPA Payment Status (PymtStatus)

`A` approved, `D` declined, `P` pre-auth, `T` pre-auth cancelled, `N` pending (cancelled if unpaid within 3 days), `V` voided, `C` cancelled, `Q` partially refunded, `R` fully refunded. (`W` appears only in the Merchant API table.)

## Rules

- Store `CardTokenId` with your `CustId`; never store PAN or CVV. CardBizPay states PCI DSS Level 1 for card storage.
- Handle `31`, `32`, `33` by prompting the cardholder to add the card again.
- Use a unique `RequestId` per AddCard, Sale, PreAuth call.
- Settle a Sale only on a verified response with `RespCode 00` and `PymtStatus A`; for accepted-but-unknown results, use TrxnEnq.
- The Notify URL payload format is not documented beyond "post back API response result"; verify its signature like any response.
