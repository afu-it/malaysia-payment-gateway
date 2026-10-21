# GoMobi Cards and Payouts

Read `gomobi-api.md` first for hosts and algorithms. Checked 2026-10-21. Docs: `payin/malaysia/cards`, `payin/malaysia/cards/card-payment`, `payin/malaysia/cards/saved-card`, `payout/overview`, `payout/malaysia`, `payout/malaysia/api`, `payout/malaysia/ipn`, `payout/malaysia/status`.

## Cards Overview

- Visa, Mastercard and UnionPay, with 3D Secure.
- Direct card capture puts the merchant in PCI-DSS scope; the hosted payment page removes that burden. Prefer Hosted Checkout unless the merchant is PCI-DSS compliant.
- International cards work only if enabled on the account ("Foreign Card Restricted" otherwise).

## Card Payment and Save Card

Browser HTML form POST to `/UMEzyway/TEzywayReqDetails.jsp` (staging `m-uat.gomobi.io`, production `pay.gomobi.io`). No headers.

| Field | Req | Notes |
|---|---|---|
| mobiApiKey | Yes | `<MOBI_API_KEY>` |
| service | Yes | `UMECOMAPI_SALE_RE` |
| loginId | Yes | `<MOBI_LOGIN_ID>` |
| carddetails | Yes | encrypted `cardNumber|cvv|expiry(YYMM)|amount`, amount 12-digit sen (`000000000500` = RM5.00) |
| nameOnCard | Sample | appears in the sample form |
| ip | Yes | customer IP |
| email | No | |
| orderId | Yes | unique |
| uid | Yes | unique id |
| mobileNo | Yes | with country code |
| umResponseUrl | Yes | where Mobi posts the result |
| mUrl | Yes | response URL |
| subMerchantMid | Yes | sub-merchant id |
| saveCard | Yes | `Yes` / `No` (tokenise) |

Encryption of `carddetails`: the page says "RSA/PKCS1 scheme as per Mobi's encryption guide", while the AES-CBC page's "Cards" sample uses AES/CBC/PKCS5Padding with a key string and an initVector string, base64, then hex. Confirm the method and key material with Mobi before coding (checked 2026-10-21).

Response (form POST to `umResponseUrl`): `responseCode` (`0000` success), `responseMessage`, `responseDescription`, `tid`, `mid`, `rrn`, `aid`, `trxId`, `date`, `time`, `approveCode`, `cardNo` (masked), `cardHolderName`, `amount`, `orderId`, `uid`, `mrn`, `walletId` (card token when saved), `cardBrand`, `cardType`, `mercMdr`, `countryCode`, `currency` (numeric ISO, sample `840`), `mobileNo`.

This response is delivered through the browser. Do not fulfil from it alone; verify the order, amount and `trxId` with Mobi before fulfilment. A card status or IPN endpoint is Not documented in official docs (checked 2026-10-21).

Errors (`0001`): Sub-Merchant MID Missing, account termination, Invalid Recurring, Invalid Card No, Foreign Card Restricted, "We Support VISA, MASTER & UNION PAY Card only", Invalid Transaction!! (PAN encryption error), Transaction Limit Exceeded, DTL Value is Missing. Card response codes: `gomobi-codes.md`.

## Saved Card (WALLET_DIRECT)

`POST /payment/ezywayservice/jsonservice`, header `mobiApiKey`.

```json
{"service":"WALLET_DIRECT","walletId":"<walletId>","amount":"000000001000","mobileNo":"<phone>","mobiApiKey":"<MOBI_API_KEY>","invoiceId":"<unique>"}
```

Response `responseData`: `mercMdr`, `responseMessage`, `CardBrand`, `trxId`, `amount`, `tid`, `mid`, `rrn`, `date`, `time`, `approveCode`, `cardNo`, `cardHolderName`, `cardType`, `txnType` (`EZYWAY`), `invoiceId`.

Errors: Invalid Transaction (inner `responseCode` `12`), Invalid Merchant Details, Invalid Request, Wallet ID Missing.

Store `walletId` as a secret-grade token tied to the customer. Merchant-initiated recurring schedules are Not documented in official docs (checked 2026-10-21); the app must schedule charges itself.

## Payouts (Malaysia)

Asynchronous disbursement to bank accounts and DuitNow-registered e-wallets. Your server IPs must be submitted to and whitelisted by Mobi; requests from other IPs are silently rejected. Reports can be exported from the portal.

### Request payout

`POST /externalapi/payoutservice` (staging `m-uat.gomobi.io`, production `trustpayouts.gomobi.io`), no headers.

| Field | Req | Notes |
|---|---|---|
| service | Yes | `PAYOUT_TXN_REQ` |
| mobiApiKey | Yes | `<MOBI_API_KEY>` |
| businessRegNo | Yes | merchant business registration number (error text says "BRN / IC") |
| bankName | Yes | bank name, or `ewallet` for e-wallets |
| bankAccNo | Yes | account no; e-wallet as `60xxxxxxxxx` |
| customerName | Yes | beneficiary name |
| amount | Yes | MYR `00.00` format, minimum RM10.00 |
| subMID | Yes | sub-merchant id ("if applicable") |
| bicCode | Yes | e.g. `ARBKMYKL` (see `gomobi-codes.md`) |
| payoutType | No | `settlement` for settlement payouts, else blank |
| payoutid | No | your payout id (unique) |
| remarks | No | max 20 chars (typed Number in docs; send a string) |

DuitNow e-wallets listed: TNG, Finexus, BigPay, Boost, Shopee, GX Bank, Merchantrade.

Response: `responseDescription` "On Process", `responseData` `{trxId, amount}`. Not final.

Errors: InvoiceId Exists Already, Sub Merchant not found, Invalid payment type, Transaction not found, Invalid mid, Invalid master mid, MobiKeyDetails not found, Merchant not found. Validation errors: `gomobi-codes.md`.

### Payout IPN

POST `{"payoutEncryptedResponse":"<base64>"}`. Decrypt with 3DES (DESede/ECB/PKCS5Padding). Decrypted:

```json
{"responseCode":"0000","responseMessage":"...","responseDescription":"Payout Processing","responseData":{"trxId":"...","amount":"...","payoutid":"..."},"failureReason":"..."}
```

`responseDescription`: `Registered Successfully Payout`, `Payout Processing`, `Payout Failed`. `responseCode` `0000` or `0001`.

### Payout status

Same URL, `{"service":"PAYOUT_TXN_DETAIL","trxId":"<trxId>","mobiApiKey":"<MOBI_API_KEY>"}` (or `payoutid`). Response: `customerName`, `merchantName`, `trxId`, `amount`, `businessRegNo`, `bankName`, `bankAccNo`, `email`, `payoutDate`, `payoutStatus` (`Paid`, `On Process`, `Pending`, `Failed`), `subMID`.

### Payout safety

- Require an internal approval step and a unique `payoutid` per payout to block duplicates ("PayoutId Already Exists").
- Mark paid only on `payoutStatus` `Paid` from status or a decrypted IPN that matches trxId and amount.
- Balance is shown in the portal menu under "Balance"; "No Available Balance Found" and "In-Sufficient Fund" errors mean the payout wallet is short.
