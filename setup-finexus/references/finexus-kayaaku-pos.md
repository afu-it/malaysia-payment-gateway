# Finexus Kayaaku POS (in-person) Reference

Use only for card or QR payments on a Finexus terminal (mPOS, Android EDC, UCAT unattended terminal). Online checkout is in `finexus-api.md`.
Docs checked 2026-10-21. Start at https://sandbox.finexusgroup.com/myxaas/docs/kayaakupos/http/overview (120 Kayaaku POS pages, see `source-map.md`).

## Integration modes

| Mode | Device | Transport | Notes |
| --- | --- | --- | --- |
| App-to-App | Android EDC with KPOS installed | Android Intent / bound service on the same device | Partner app must pass penetration testing by an independent certified assessor. |
| Bluetooth | mPOS / EDC | Bluetooth from ECR | Uses the same parameter set as App-to-App (MsgVer, PmtType, TxnID, AmtTxn ...). |
| USB | mPOS only | USB serial cable | QR payments do not use the mPOS device. |
| HTTP (HTTPS) | EDC | HTTPS to Finexus host, JWE body | Enable on terminal: Settings > ECR / Cash Register / POS Setting > HTTPs/MQTTs Integration. |
| Pre-auth via HTTPS | EDC | HTTPS `thirdparty/*`, RSA signed JSON | EV chargers, parking, cashierless stores, petrol pumps. |
| UCAT HTTP / UCAT pre-auth HTTP / UCAT pre-auth EV HTTP | UCAT | HTTPS, same as HTTP mode | Parking (ANPR), vending, AI refrigerator. |
| UCAT pre-auth serial cable | UCAT | Serial: STX + DATA + XOR checksum (2 hex) + ETX (Appendix 5) | |

## App-to-App and Bluetooth

- Intent actions: `com.finexuscards.yippiepos.xxx.SALES`, `.VOID`, `.REFUND`, `.SALESCOMPLETION`, and `com.finexuscards.yippiepos.ENQUIRY`. `xxx` is environment (docs show `sandbox`). Bound service: `com.finexuscards.kayaakupos.service` with permission `com.finexuscards.kayaakupos.BIND_SERVICE`.
- Both KPOS and the merchant app must be installed on the same device to use Intents.
- Void: same day, before settlement. Refund: after settlement or beyond one day. DuitNow QR: refund only.

Sales request fields: `MsgVer` ("1.0"), `PmtType` (`00` all methods, `01` card, `04` DuitNow QR merchant presented, `08` Alipay+ merchant presented), `CallerDeviceType` (`01` mobile, `03` kiosk), `CallerDeviceVer`, `TxnID` (unique, up to 50, UUID suggested), `LocalTxnDTTime` (YYYYMMDDhhmmss), `Email`, `Pwd` (KPOS login), `AmtTxn` (12 digits, implied decimals, e.g. `000000010000` = RM 100.00), `CrcyTxn` (`458`), `TxnTyp` (`STX`, required for Bluetooth / serial). Optional: `Description`, `OptInPrintReceipt`, `OptInSendEReceipt`, `OptInEReceipt`, `OptInPmtAckmnt` (Y / N, default Y), `PaymentTimeout` (5-300 s).

Sales response fields: `MsgVer`, `TxnID`, `ResultCode` (examine this, Appendix 3), `RespCd` (host, Appendix 2), `OrgRespCd`, `RRN`, `STAN`, `AuthIdResp`, `MRN`, `TxnDTTime` (GMT+8), `MaskedPAN` (first 6 and last 4), `PaymentScheme` (`AF` Alipay, `JC` JCB, `MC` Mastercard, `PN` DuitNow, `UP` UnionPay, `VS` Visa).

Status enquiry for Bluetooth / USB / pre-auth: `POST https://xxx.finexusgroup.com/mqpos/sandbox/host/thirdparty/transactionDetailsEnquiry`.

## HTTP mode

### Endpoints

| Operation | Method | URL | Request MTI | Response MTI |
| --- | --- | --- | --- | --- |
| Submit payment (and e-invoice) | POST | `https://xxx.finexusgroup.com/mqpos/sandbox/host/api/submit` | 0011 | 0012 |
| Cancel (void) | POST | `.../host/api/cancel` | 0013 | 0014 |
| Refund | POST | `.../host/api/refund` | 0015 | 0016 |
| Status enquiry | POST | `.../host/api/paymentInquiry` | 0017 | 0018 |
| Payment notification (Finexus to merchant) | POST | merchant `paymentNotiUrl` (docs: `https://xxx/xxx/paymentNotification`) | 0025 | 0026 (merchant reply) |
| Abort submitted payment | POST | `.../host/api/submit/abort` | 0031 | 0032 |
| Print receipt | POST | `.../host/api/printReceipt` | 0033 | 0034 |

`xxx` and `sandbox` are placeholders; production host comes from Finexus. Not documented in official docs (checked 2026-10-21): the production path.

### Transport and security

- Headers: `Content-Type: text/plain`, `Accept: application/json`, `Signature: <url-encoded base64 ECDSA signature>`, `sourceSystem: <system ID from Finexus>`.
- Body: 4-digit MTI followed directly by a JWE compact string, e.g. `0011eyJlcGsiOnsia3R5...`.
- JWE: `alg=dir`, `enc=A256CBC-HS512`. Content key from ECDH on P-256 between your EC private key and the Finexus POS public key, Concat KDF (SHA-256, key length 512 bits, `partyVInfo="1.0"`); the ephemeral public key goes in the `epk` header (with `kid`).
- Signature: `SHA512withECDSA` over the UTF-8 string `MTI + JWE` (no separator), base64, then URL-encode. Verify a response over its full body (MTI + JWE) with the Finexus public key.
- Key exchange: send Finexus your public key and the Device Serial Number (Settings > System > About > Status). Finexus returns its public key and your `sourceSystem`. Each merchant needs its own key.
- Keys: Appendix 4 generates the encryption key on `prime256v1`; the key exchange page generates a signing key on `secp521r1`. Use what Finexus asks for.
- Inconsistencies to confirm with Finexus: the PHP sample signs `MTI + "\n" + JWE` while Java, Node.js, Python and .NET sign `MTI + JWE`; the .NET sample signs DER but verifies `Raw`.

### Submit payment (0011) body

`msgVer` ("1.0"), `billerType` (`PS` POS system, `AM` automated machine, `KP` KPOS), `callerDeviceType` (2, e.g. `02` Android EDC, `03` kiosk, `05`, `17` car park, `21`), `callerDeviceVer`, `sequenceNo` (16, unique per calendar day, restarts at 1; recommended `[TID][MTI][running no]`), `sellerId` (6, from Finexus for e-invoice; `000000` if none), `mid` (15), `isPaymentReq` (Y / N), `payment` object, `isInvoiceReq` (Y / N), `invoice` object (conditional).

`payment` object: `mrn` (40, unique), `requiredPayment`, `paymentMethod` (`00` all, `01` card, `04` DuitNow QR, `08` Alipay+), `currency` (`458`), `amount` (minor units, `1000` = RM 10.00), `description`, `paymentNotiUrl` and `paymentNotiUrl2` (HTTPS only), `paymentTime` (ISO 8601), `email`, `deviceSN`, `printPaymentReceipt`, `nonBlocking` (default Y), `paymentTimeout` (5-900 s, default 300), `paymentType` (`STX` sale, `SLC` sales completion, `VIS` Visa instalment), `htr`.

Response (0012) and enquiry response (0018): `sequenceNo`, `resultCode` (`00000` success), `advMsg`, `authIdResp`, `paymentScheme` (`AF`, `JC`, `MC`, `MD`, `PN`, `UP`, `VS`), `maskedPAN`, `htr`, `paymentMethod`, `paymentType`, `visDetails` {`numOfInstalment`, `instalmentFrequency`, `instalmentAmount`, `mdrFee`}, `receiptInfo` {`acqCode` (FINEXUS), `acqMid`, `acqTid`, `stan`, `rrn`, `authIdResp`, `aid`, `appLabel`, `arqc`}.

### Payment notification (0025, Finexus to merchant)

Body fields: `msgVer`, `billerType`, `callerDeviceType`, `callerDeviceVer`, `sequenceNo`, `mrn`, `resultCode`, `authIdResp`, `paymentScheme`, `maskedPAN`, `htr`, `paymentMethod`, `advMsg`, `paymentType`, `mid`, `tid`, `currency`, `amount`, `visDetails`, `receiptInfo`, `invoice` (for `KP` + kiosk).
Merchant reply (0026, encrypted and signed): `sequenceNo` (echo), `resultCode`. Verify the `Signature` header before trusting the notification. Merchant result codes `PS001`-`PS016` (e.g. `PS001` invalid signature, `PS014` duplicate notification). De-duplicate on `mrn`.

### Status enquiry (0017)

`msgVer`, `billerType`, `callerDeviceType`, `callerDeviceVer`, `sequenceNo`, `mid`, `mrn` (50). Use when no response or notification arrives.

### Cancel (0013) and refund (0015)

Common: `msgVer`, `billerType`, `callerDeviceType`, `callerDeviceVer`, `sequenceNo`, `sellerId`, `mid`, `mrn` (original), `currency` (conditional), `invoice` (credit note / refund note), plus:
- Cancel: `cancelMrn` (new unique), `refundAmount` (conditional), `cancelReason` (300). Response adds `issuedInvoiceInd` (`C` credit note), `transactionAction` (`C` cancelled), `receiptInfo`.
- Refund: `refundMrn` (new unique), `refundAmt` (conditional), `refundReason` (300). Response: `sequenceNo`, `resultCode`, `advMsg`, `receiptInfo`.

### Abort (0031) and print receipt (0033)

`msgVer`, `billerType`, `callerDeviceType`, `callerDeviceVer`, `sequenceNo`, (`sellerId` for abort), `mid`, `mrn`, `deviceSN`. Response: `sequenceNo`, `resultCode`, `advMsg`.

### Invoice object (LHDN e-invoice)

Header: `invNo` (22), `oriInvNo` (cancel / refund), `invDt` (yyyymmdd), `invTm` (hhmmss), `invCur` and `taxCur` (`458` only), `billFreq`, `billStartDt`, `billEndDt`, `totalNet`, `totalDisc`, `totalFeeCharge`, `totalExclTax` (= totalNet - totalDisc + totalFeeCharge), `totalTax`, `totalInclTax`, `pmtMode`, `supplierAcctNo`, `prePmtAmt`, `prePmtDt`, `prePmtTm`, `prePmtRefNo`, `roundAmt` (can be negative, minor units), `totalPayable` (= totalInclTax - prePmtAmt + roundAmt), `billRefNo`, `items[]`, `addAllowCharge[]`, `shippingRecipient`, `denyIndividualEInvoice` (default N), `eSignature` and `pinCode` (FIDU QR, required when individual e-invoice allowed).
Item: `oriInvNo`, `classCde` (Appendix 6, `001`-`043`), `desc`, `uom`, `unitPrice`, `qty`, `subtotal`, `discRate`, `discAmt`, `feeRate`, `feeAmt`, `totalExclTax`, `taxTyp` (`06` not applicable), `taxRate`, `taxExemptInd`, `taxExemptDtl`, `taxExemptAmt`, `taxAmt`, `totalInclTax`, `prodTariffCde`, `oriCountry` (e.g. `MYS`), `origin*` fields, `givenItemId` (required if configured in KBIZ).
Allowance / charge: `allowChargeTyp` (`C` fee), `allowChargeDesc`, `allowChargeAmt`. Shipping recipient: `name`, `addr1`-`addr3`, `state` (Appendix 7, `01`-`17`, e.g. `14` W.P. Kuala Lumpur), `city`, `postcode`, `country` (`MYS`), `tin`, `idType` (NRIC, PASSPORT, BRN, ARMY), `idValue`.
Customers claim the e-invoice through the FIDU app by scanning the FIDU QR on the receipt (Appendix 8).

## Pre-auth via HTTPS (thirdparty)

| Operation | URL |
| --- | --- |
| Pre-auth or sale | `POST .../mqpos/sandbox/host/thirdparty/paymentAuthorization` |
| Sales completion | `POST .../thirdparty/saleCompletion` |
| Void | `POST .../thirdparty/void` |
| Refund | `POST .../thirdparty/refund` |
| Enquiry | `POST .../thirdparty/transactionDetailsEnquiry` |

Request JSON: `MsgVer`, `PmtType` (`01` card), `IsPreauthInd` (Y / N, card only), `CallerDeviceType` (`04` web), `Email`, `DeviceSN`, `CallerDeviceVer`, `TxnID`, `LocalTxnDTTime`, `AmtTxn`, `CrcyTxn`, `MID`, `sourceSystem`, `sequenceNo`, `signedMessage`; optional `Description`, `CallerDeviceId`, `OptInPrintReceipt`, `OptInSendEReceipt`, `OptInPmtAckmnt`, `PaymentTimeout` (5-900 s).
Response: `MsgVer`, `TxnID`, `ResultCode`, `RespCd`, `OrgRespCd`, `RRN`, `STAN`, `AuthIdResp`, `MRN`, `TxnDTTime`, `MaskedPAN`, `PaymentScheme`, `HTR`, `AdvMsg`, `sourceSystem`, `sequenceNo`, `signedMessage`.
Signature: `SHA256withRSA` (PKCS#1 v1.5) over `sourceSystem + "\n" + <compact JSON of the request without signedMessage>`, base64, URL-encode, put in `signedMessage`. Key: RSA 2048 (`openssl genrsa -out privatekey.pem 2048`). The field order in the JSON must match what you send; samples differ between languages, so serialize once and sign that exact string.

## Codes (Kayaaku POS appendices)

- HTTP `resultCode`: `00000` successful. Merchant reply codes `PS001`-`PS016`. Terminal codes `KA001`-`KA099` (e.g. `KA005` cancel, `KA020` read card timed out, `KA021` over contactless limit, `KA025` no internet). Host codes `KP001`-`KP099` (e.g. `KP050` invalid transaction status, `KP063` status unknown, `KP078` transaction ID exists, `KP090` invalid SHV). Switch codes `SW001`-`SW100` mirror ISO card response codes.
- App-to-App `ResultCode` numeric (e.g. `21` over contactless limit, `22` email mismatch, `23` device processing, `24` reader connection failed, `25` server error or timeout, `99` exception).
- Transaction status: `ST` settled, `UT` unsettled, `VT` voided, `DC` declined, `PT` pending TC, `IN` in progress, `AC` approved completion, `PA` pre-auth.
- Treat "status unknown", "in progress" and timeouts as unknown: run an enquiry, do not mark failed.
