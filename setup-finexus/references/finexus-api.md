# Finexus PayMaster (MyXaaS) API Reference

Entry point for Finexus online payments. In-person terminals (Kayaaku POS) are in `finexus-kayaaku-pos.md`.
Docs checked 2026-10-21 (MyXaaS developer portal, change log v2.0.8 dated 18 Aug 2026).

## Source URLs

- Get started: https://sandbox.finexusgroup.com/myxaas/docs/get-started
- PayMaster intro: https://sandbox.finexusgroup.com/myxaas/docs/paymaster/intro
- PayMaster API (raw HTTPS, no library): https://sandbox.finexusgroup.com/myxaas/docs/paymaster/api/intro
- UPP payment fields: https://sandbox.finexusgroup.com/myxaas/docs/paymaster/api/upp/set-parameters/payment
- UPP query fields: https://sandbox.finexusgroup.com/myxaas/docs/paymaster/api/upp/set-parameters/query
- UPP reversal fields: https://sandbox.finexusgroup.com/myxaas/docs/paymaster/api/upp/set-parameters/reversal
- DuitNow Pay API: https://sandbox.finexusgroup.com/myxaas/docs/paymaster/api/duitnow-pay/overview
- Hash (API section): https://sandbox.finexusgroup.com/myxaas/docs/paymaster/api/appendix/appendix1
- Hash (MPI section, response): https://sandbox.finexusgroup.com/myxaas/docs/paymaster/appendix/appendix1
- Status codes: https://sandbox.finexusgroup.com/myxaas/docs/paymaster/appendix/appendix3/transaction-status
- Important notes (rate limit): https://sandbox.finexusgroup.com/myxaas/docs/paymaster/important-notes
- Full page list: `source-map.md`.

## Product map

| Name | What it is |
| --- | --- |
| MyXaaS | Finexus developer portal and merchant onboarding platform. |
| PayMaster | Finexus online payment gateway. |
| UPP (Unified Payments Platform) | Hosted payment page. Customer picks card, FPX/DuitNow Pay, or QR on Finexus. Main integration. |
| MPI (Merchant Plug-In) | Official library (Java, .NET, PHP, Python, Node.js, JS test kit) that builds, hashes and parses PayMaster messages for you. |
| PayMaster API | Same messages sent as raw HTTPS without the MPI. |
| DuitNow Pay (OBW) | Online banking redirect (DuitNow online banking, RPP, or FPX) without the UPP page. |
| Dynamic QR | Merchant presented QR per transaction (DuitNow QR, Alipay+). |
| Consumer presented QR (CP QR) | Merchant scans the customer's wallet QR. |
| Static QR | Fixed QR; Finexus pushes a signed JSON notification to the merchant. |
| Kayaaku POS | Card and QR terminals (see `finexus-kayaaku-pos.md`). |

Finexus Cards Sdn Bhd says it is a merchant acquirer and e-money issuer registered with Bank Negara Malaysia (https://www.finexusgroup.com/businesses/payments).

## Environments and hosts

- The docs mask hosts as `https://xxx.finexusgroup.com/...`. Use the host Finexus gives you at onboarding.
- Sandbox host shown in docs examples: `https://sandbox.finexusgroup.com/upp/faces/...`.
- The official MPI config files and the WooCommerce plugin point to a testing host: `https://dev.finexusgroup.com:10501/upp/...` (MPI SOAP query: `https://dev.finexusgroup.com:10501/upp/services/Gateway`).
- Production host: Not documented in official docs (checked 2026-10-21). Get it from Finexus. Do not guess it.
- All payment paths sit under `/upp/faces/`.

## Endpoint catalogue

All are PayMaster messages. MTI = message type indicator (`h001_MTI`). "GET redirect" means the browser is sent to the URL with the fields as query parameters.

| # | Operation | Method | Path | Request MTI | Response MTI | How the response arrives |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | UPP payment (hosted page) | GET redirect | `/upp/faces/upp/payment.xhtml` | 0280 | 0290 | Browser GET redirect to success / unsuccessful / cancel URL |
| 2 | UPP and DuitNow payment query | POST | `/upp/faces/upp/requery0780.xhtml` | 0780 | 0790 | To `f284_RURL_UPPPQ`, else same session |
| 3 | MPI query (SOAP) | POST SOAP | `/upp/services/Gateway` op `UPPPaymentQuery` | Query0780Req | Query0790Resp | SOAP response (MPI does this for you) |
| 4 | Reversal / void / refund (card, QR) | GET redirect | `/upp/faces/reversal0400.xhtml` | 0400 | 0410 | To `f272_RURL_CCR` |
| 5 | DuitNow Pay refund (by RRN) | GET redirect | `/upp/faces/reversal0400.xhtml` with `f003_ProcCode=993000` | 0400 | 0410 | To `f272_RURL_CCR` |
| 6 | DuitNow Pay bank listing | GET | `/upp/faces/banklisting.xhtml` | 0360 | 0370 | Response message |
| 7 | DuitNow Pay payment (OBW) | GET redirect | `/upp/faces/duitnowpaypayment.xhtml` | 0880 | 0890 | Browser GET redirect to `f275_RURL_DD` |
| 8 | Dynamic QR generate payload | POST, server side, follow redirect | `/upp/faces/generateQrPayload.xhtml` | 0180 | 0190 | Response body, then MT0250 callback |
| 9 | Dynamic QR payment callback | POST from Finexus | merchant `SuccRespURL` | 0250 | merchant returns HTTP 200 | Server to server |
| 10 | Consumer presented QR payment | GET | `/upp/faces/qrPushPayment.xhtml` | 0240 | 0250 | Response message |
| 11 | Card payment / recurring charge (token) | GET redirect | `/upp/faces/ccpayment.xhtml` | 0200 | 0210 | To `f265`/`f266`/`f267` URLs |
| 12 | Pre-auth (card) | UPP with `f362_PreAuthFlg=Y` or `ccpayment.xhtml` MT0100 | as row 1 or 11 | 0280 / 0100 | 0290 / 0110 | Browser redirect |
| 13 | Pre-auth completion (C13) | POST (MPI) | `/upp/faces/preauthcomplete.xhtml` | 0220 | 0230 | Response message |
| 14 | Void pre-auth (C12) | GET | `/upp/faces/ccpayment.xhtml`, `f003_ProcCode=023000` | 0100 | 0110 | Response |
| 15 | Void completion (C14) | POST | `/upp/faces/preauthcomplete.xhtml`, `f003_ProcCode=023000` | 0220 | 0230 | Response |
| 16 | Static QR push notification | POST from Finexus, JSON | merchant primary and secondary callback URLs | n/a | merchant JSON reply | Server to server, RSA signed |
| 17 | PAN profile maintenance (SOAP, WSDL only) | SOAP | `/upp/services/Gateway` op `PANProfileMaintenanceQuery` | 0600 | 0610 | No field docs. Not documented in official docs (checked 2026-10-21) beyond the WSDL. |
| 18 | PAN list query (SOAP, WSDL only) | SOAP | `/upp/services/Gateway` op `PANListQuery` | 0620 | 0630 | Same as above. |

Not documented in official docs (checked 2026-10-21): settlement report API, payout API, webhook registration API, refund status API separate from query. The OpenAPI spec page (`/myxaas/docs/openapi-spec`) is behind a client certificate and could not be read.

### MPI PaymentID map (PaymasterEntryPoint.json, PHP MPI 3.0.3)

The MPI picks MTI, processing code (`f003_ProcCode`) and POS condition (`f008_POSCond`) from a PaymentID.

| PaymentID | Use | MTI | f003 | f008 |
| --- | --- | --- | --- | --- |
| C01 | Card verification | 0200 | 003000 | 51 |
| C02 | Card e-commerce sale | 0200 | 003000 | 59 |
| C03 | Card MOTO sale | 0200 | 003000 | 08 |
| C04 | Card recurring sale | 0200 | 003000 | 73 |
| C05 / C06 / C07 | Refund (ecom / MOTO / recurring) | 0400 | 993000 | 59 / 08 / 73 |
| C08 / C09 / C10 | Void (ecom / MOTO / recurring) | 0200 | 023000 | 59 / 08 / 73 |
| C11 | Pre-auth | 0100 | 303000 | 06 |
| C12 | Void pre-auth | 0100 | 023000 | 06 |
| C13 | Pre-auth completion | 0220 | 003000 | 06 |
| C14 | Void completion | 0220 | 023000 | 06 |
| C15 / C16 / C17 | Reversal (ecom / MOTO / recurring) | 0400 | 220000 | 59 / 08 / 73 |
| C18 | Reversal of pre-auth | 0400 | 303000 | 59 |
| C19 / C20 / C21 | Reversal of refund | 0400 | 203000 | 59 / 08 / 73 |
| D01 | Direct debit (FPX / DuitNow Pay) sale | 0260 | 001000 | |
| U01 | UPP payment | 0280 | | |
| U02 | UPP query | 0780 | | |
| Q01 | Dynamic QR payload | 0180 | 000000 | 67 |
| Q02 | QR sale | 0240 | 003000 | 67 |
| Q03 | Consumer presented QR | 0240 | 003000 | 69 (per CP QR page) |
| B01 | Bank listing | 0360 | | |

Processing codes (Appendix 4): 000000 QR payload generation, 001000 normal sales, 003000 card verification / sale / QR sale, 023000 void, 203000 reversal for refund, 220000 reversal, 303000 pre-authorization, 993000 refund.

## Message format

- Fields are `name=value` pairs: header `h001`-`h004`, body `fNNN_Name`, trailer `t001_SHT`, `t002_SHV`.
- Header: `h001_MTI` (4), `h002_VNO` version (`06` for UPP and DuitNow Pay; `01` for bank listing), `h003_TDT` YYYYMMDD, `h004_TTM` HHMMSSss (8 digits, e.g. `12432905`).
- Trailer: `t001_SHT=SH2` (HMAC-SHA256). Docs also mention MD5 HMAC; use SH2. `t002_SHV` is 64 hex chars.
- Values are URL-encoded for transport. The hash is computed over the raw (decoded) values.
- The MPI replaces `&` with `^` inside return URL values (f265, f266, f267, f272, f273, f275, f276, f280, f281, f282, f284) so the URL does not break the field list. If you build messages yourself, avoid `&` in return URLs or URL-encode them.

## Amount rules

- `f007_TxnAmt` is 12 digits, zero padded, no decimal point. Decimals are implied by `f019_ExpTxnAmt` (`2` for MYR). RM 1.00 = `000000000100`; RM 570.80 = `000000057080`.
- `f010_CurrCode=458` (MYR, ISO 4217 numeric). `f253_CntyCode=MY`. `f260_ServID=FNX`.
- MPI entity setters take decimal strings (e.g. `"1280.99"`) and pad for you.
- `f247_OrgTxnAmt` / `f248_OrgCurrCode` / `f287_ExpOrgTxnAmt` carry an original foreign amount if used.

## Signature (Secure Hash, t002_SHV)

Shared secret: the merchant `SecretKey` from Finexus (PayMaster Gateway). Same key signs requests and verifies responses.

Exact string to sign:

```
message = <every header and body field as name=value, joined with "&", in message order,
           trailer fields t001_SHT and t002_SHV removed, empty fields kept as "name=">
input   = message + SecretKey          (secret key appended directly, no separator)
t002_SHV = UPPERCASE( HEX( HMAC_SHA256( key = SecretKey, data = input ) ) )
```

Pseudo-code:

```
function finexusHash(fields /* ordered list of [name, value] */, secretKey):
    msg = join("&", [name + "=" + value for (name, value) in fields
                     if name not in ("t001_SHT", "t002_SHV", "f289_PortalCustId", "f289_CustId", "f290_Signature")])
    return upper(hex(hmac_sha256(key = utf8(secretKey), data = utf8(msg + secretKey))))

# request
fields.append(["t001_SHT", "SH2"]); fields.append(["t002_SHV", finexusHash(fields, KEY)])

# response (browser redirect query string, or query/reversal response)
raw = query string up to (not including) "&t001_SHT="
msg = url_decode(raw)            # "+" becomes a space, %XX decoded, keep field order exactly as received
expected = upper(hex(hmac_sha256(KEY, msg + KEY)))
ok = constant_time_equals(expected, received t002_SHV)
```

Official sources for this method:
- API Appendix 1: "hexadecimal-encoded SHA-256 HMAC ... of the concatenation of all fields in the Message Header and Data Content (excluding trailer fields), followed by the secret key".
- Official WooCommerce plugin `MessageHashing.php`: `strtoupper(hash_hmac("sha256", $message.$secretKey, $secretKey))`.

Test vectors (docs, sample key `00112233445566778899AABBCCDDEEFF`, sandbox example only):
- Request 0880 `h001_MTI=0880&h002_VNO=06&h003_TDT=20260824&h004_TTM=12432905&f001_MID=120567801561128&f003_ProcCode=000000&f006_TxnDtTm=20010831124329&f007_TxnAmt=000000057080&f010_CurrCode=458&f019_ExpTxnAmt=2&f249_TxnCh=WEB&f253_CntyCode=MY&f256_FICode=RPP&f260_ServID=FNX&f263_MRN=2012121912345&f264_Locale=en&f268_CHName=Thomson&f275_RURL_DD=https://www.megastore.com/dd/done.jsp&f278_EMailAddr=thomson8899@yahoo.com&f279_HP=60128888888&f285_IPAddr=11.11.88.88&f352_AcqBank=FNXSMYNB` gives `6C825DC0E39F2917A8BC34A45F5BAF6EDDF7353446C19D3CC6296E6F3A962892` (reproduced).
- Response 0890 example on the same page gives `0978A771F4F3395B00455BFBD7ADF732A09DBCFF65DE773009940DAA113248FC` (reproduced).
- The older MPI-section 0290 example (`E73CED...` in text, `FC63B3...` in the string) does not reproduce with the sample key. Use the API-section vectors for tests.

Field order and exclusions:
- API Appendix 1 says "ascending field-ID order". The MPI schemas (which build the real messages) place some fields out of numeric order, e.g. UPP 0280: `... f287, f350, f354, f325, f339, f344, f347, f362, f363, f364, f365`; card: `... f288, f350, f354, f352, f353, f325, f339, f340 ...`. Use the order in the field tables below (they follow the MPI schema). Confirm with Finexus if a hash fails on optional fields.
- Not hashed: `f289_PortalCustId` / `f289_CustId` and `f290_Signature` (MPI schema `Hash: false`).
- For responses, hash the fields in the order received. Reject on mismatch. MPI reports a bad hash as `TxnStatDetCde=5015`.
- The WooCommerce plugin compares hashes with `==`. Use a constant-time compare.

Optional message encryption: the PHP MPI and WooCommerce plugin support AES-256-CTR (`Encryption=true` in `PaymasterEntryPoint.json`, default `false`), sending `encryptedData=<base64url>&iv=<hex>`. Not described in the HTML docs; only enable if Finexus asks.

## 1. UPP payment request (MTI 0280)

`GET https://<host>/upp/faces/upp/payment.xhtml?<fields>` (send the browser here, or auto-submit a GET form).

Mandatory:

| Field | Len | Value / meaning |
| --- | --- | --- |
| h001_MTI | 4 | `0280` |
| h002_VNO | 2 | `06` |
| h003_TDT | 8 | YYYYMMDD |
| h004_TTM | 8 | HHMMSSss |
| f001_MID | 15 | Merchant ID `<MERCHANT_ID>` |
| f007_TxnAmt | 12 | Amount, implied decimals |
| f010_CurrCode | 3 | `458` |
| f019_ExpTxnAmt | 1 | `2` |
| f249_TxnCh | 3 | `WEB` |
| f253_CntyCode | 2 | `MY` |
| f260_ServID | 3 | `FNX` |
| f263_MRN | 1-40 | Merchant reference number, unique per payment |
| f264_Locale | 2 | `en` |
| f325_ECommMercInd | 1 | `0` / `1` 3DS / `2` non-3DS MOTO |
| f339_TokenFlg | 1 | `Y` save card token / `N` |
| f362_PreAuthFlg | 1 | `N` sale / `Y` pre-auth / `A` |
| t001_SHT | 3 | `SH2` |
| t002_SHV | 64 | Hash |

Optional (hash order as listed): `f247_OrgTxnAmt`, `f248_OrgCurrCode`, `f250_CCProcFee` (MPI `f250_CCProcFeeAmt`), `f251_DDProcFee` (MPI `f251_DDProcFeeAmt`), `f252_PromoCode` (`01` Visa, `02` Mastercard, `49` issuer, `60` BIN list, `50` DD only, `51` DD bank), `f255_IssCode` (pipe separated), `f256_FICode`, `f261_HostID` (8), `f262_SessID` (1-256), `f270_ORN` order ref (2-34), `f271_ODesc` description, `f278_EMailAddr`, `f279_HP` phone, `f280_RURL_UPPPS` success URL, `f281_RURL_UPPPU` unsuccessful URL, `f282_RURL_UPPPC` cancel URL, `f285_IPAddr` (7-39), `f287_ExpOrgTxnAmt`, `f289_PortalCustId` (not hashed), `f290_Signature` (not hashed), `f350_CrdTyp` (`VSA`, `MST`, `CUP`, `AP`), `f354_TID`, `f344_MercCustId` (customer id for saved cards), `f347_TokenShrtNm` (required when `f339_TokenFlg=Y`), `f363_InvNum`, `f364_Fee` (TLV: tag 4, length 2, value), `f365_POSEnvFlg` (`C`, `I`, `R`).

Return URLs can also be set in MPI properties (`SuccRespURL`, `UnsuccRespURL`, `CancelRespURL`).

## 2. UPP payment response (MTI 0290)

Sent as a browser GET redirect to the success, unsuccessful or cancel URL. It is browser data: verify the hash, then confirm by query before fulfilling.

Fields in order: `h001_MTI` (0290), `h002_VNO`, `h003_TDT`, `h004_TTM`, `f001_MID`, `f004_PAN` (masked), `f005_ExpDate` (YYMM), `f006_TxnDtTm`, `f007_TxnAmt`, `f009_RespCode` (card result, Appendix 3), `f010_CurrCode`, `f011_AuthIDResp`, `f019_ExpTxnAmt`, `f023_RRN`, `f024_OrgRespCode`, `f254_DDRespCode` (FPX / DuitNow Pay result), `f256_FICode`, `f257_PGRN` (gateway ref), `f258_TxnStatDetCde` (`0000` success), `f259_TxnStatMsg`, `f260_ServID`, `f261_HostID`, `f262_SessID`, `f263_MRN`, `f270_ORN`, `f277_DDRN` (direct debit ref), `f283_UPP_PM` (`00` credit card, `01` debit card, `02` FPX / DuitNow Pay, `03` QR, `99` unknown or error), `f286_OrgDDRespCode`, `f350_CrdTyp`, `f352_AcqBank`, `f353_IPPTenure`, `f325_ECommMercInd`, `f339_TokenFlg`, `f344_MercCustId`, `f346_Token`, `f347_TokenShrtNm`, `f348_MaskPAN`, `f365_POSEnvFlg`, `t001_SHT`, `t002_SHV`.

Success test: hash valid, `f263_MRN` and `f007_TxnAmt` and `f010_CurrCode` match your order, and the result code for the mode in `f283_UPP_PM` is `00` (`f009_RespCode` for cards, `f254_DDRespCode` for FPX / DuitNow Pay), with `f258_TxnStatDetCde=0000`. `22` means pending: keep the order pending and query.

## 3. UPP / DuitNow query (MTI 0780 / 0790)

`POST https://<host>/upp/faces/upp/requery0780.xhtml?<fields>`

Request: `h001_MTI=0780`, `h002_VNO=06` (one example shows `03`; use `06`), `h003_TDT`, `h004_TTM`, `f001_MID`, `f260_ServID=FNX`, `f263_MRN` (same as the payment), optional `f284_RURL_UPPPQ` (1-512; if omitted the result returns to the same session), `t001_SHT`, `t002_SHV`.

Response 0790: `h001`-`h004`, `f001_MID`, `f004_PAN`, `f006_TxnDtTm`, `f007_TxnAmt`, `f009_RespCode`, `f010_CurrCode`, `f011_AuthIDResp`, `f019_ExpTxnAmt`, `f023_RRN`, `f024_OrgRespCode`, `f247_OrgTxnAmt`, `f248_OrgCurrCode`, `f249_TxnCh`, `f254_DDRespCode`, `f256_FICode`, `f257_PGRN`, `f260_ServID`, `f261_HostID`, `f262_SessID`, `f263_MRN`, `f270_ORN`, `f274_QRespCode`, `f277_DDRN`, `f283_UPP_PM`, `f286_OrgDDRespCode`, `f287_ExpOrgTxnAmt`, `t001_SHT`, `t002_SHV`; optional `f258_TxnStatDetCde`, `f259_TxnStatMsg`, `f350_CrdTyp` (`VSA`, `MST`, `CUP`, `AMX`, `JCB`, `DNR`, `WLT`, `FPX`, ...), `f398_CrdProduct` (`C` credit, `D` debit, `P` prepaid).

`f274_QRespCode` (Appendix 3 query status): `00` found, details returned; `12` transaction does not exist; `30` invalid query message; `63` not permitted (security violation); `96` temporary problem, retry later. Only read the transaction result when `f274_QRespCode=00`.

Polling rule (UPP and DuitNow Pay pages): if no response 30 seconds after the payment, query every 5 seconds for up to 30 minutes, then stop.

MPI alternative: MPI `U02` uses SOAP `UPPPaymentQuery` at `query_gateway` (`.../upp/services/Gateway`) and returns a response entity with the same data.

## 4. Reversal, void and refund (MTI 0400 / 0410)

`GET https://<host>/upp/faces/reversal0400.xhtml?<fields>`

| Field | Rule |
| --- | --- |
| h001_MTI | `0400` |
| f001_MID | MID |
| f003_ProcCode | `220000` reversal; `993000` refund; `203000` reversal of refund; `303000` reversal of pre-auth |
| f004_PAN | Conditional (card) |
| f006_TxnDtTm | Date time |
| f007_TxnAmt | Same as original approved amount |
| f008_POSCond | `59` (ecom) |
| f010_CurrCode, f019_ExpTxnAmt | `458`, `2` |
| f011_AuthIDResp | Required to reverse an approved transaction; omit when reversing a timeout |
| f023_RRN, f257_PGRN | Conditional, from original |
| f260_ServID | `FNX` |
| f263_MRN | Original MRN (reversal). For DuitNow Pay refund use a NEW MRN plus the original `f023_RRN`. |
| Optional | `f261_HostID`, `f262_SessID`, `f264_Locale`, `f270_ORN`, `f272_RURL_CCR` return URL, `f354_TID` |

Response 0410: `f001`, `f003`, `f004`, `f006`, `f007`, `f009_RespCode`, `f010`, `f011`, `f019`, `f257`, `f258_TxnStatDetCde`, `f259_TxnStatMsg`, `f260`, `f261`, `f262`, `f263`, `f270`, trailer.

`f258_TxnStatDetCde` for reversal:

| Code | Class | Meaning |
| --- | --- | --- |
| 0000 | Success | Reversal approved |
| 0021 | Success | Already reversed |
| 0076 | Success | Reversal for non-existing transaction acknowledged |
| 0022 | Retry | In progress, query or retry later |
| 0091 | Retry | Acquirer resources unavailable |
| 0096 | Retry | PayMaster resources unavailable |
| 0013 | Fatal | Invalid reversal amount |
| 0014 | Fatal | Authorization ID unmatched |
| 0030 | Fatal | Invalid message format |
| 0090 | Fatal | Acquirer rejected reversal |

Rules: no partial reversal; amount cannot exceed the approved amount; may be rejected after settlement cut-off. Void / reversal (C15) is for same day before settlement. Refund (C05) is for after settlement or beyond one day. DuitNow QR supports refund only. Partial refund: Not documented in official docs (checked 2026-10-21).

## 5. DuitNow Pay (online banking) API

### Bank listing (MTI 0360 / 0370)

`GET https://<host>/upp/faces/banklisting.xhtml?` with `h001_MTI=0360`, `h002_VNO=01`, `h003_TDT`, `h004_TTM`, `f001_MID`, `f010_CurrCode=458`, `f249_TxnCh=B2C`, `f256_FICode` (`RPP` DuitNow or `FPX`), `f260_ServID=FNX`, trailer. MPI also sends `f262_SessID`.

Response 0370: `f001`, `f249`, `f256`, `f258_TxnStatDetCde`, `f259_TxnStatMsg`, `f260`, `f351_AcqBankList` (comma separated entries such as `BIMBMYKL~Bank+Islam~A~RET~https...`; the docs define only bank code and name; other parts are undocumented), trailer.

Sample bank codes (Appendix 2, use in `f352_AcqBank`): ABB0233 Affin Bank, ABMB0212 Alliance Bank, ACFBMYK1 ACF Bank, AGRO01 Agro Bank, AMBB0209 AmBank, BCBB0235 CIMB, BIMB0340 Bank Islam, BIMBMYKL Bank Islam, BKCHMYKL Bank of China, BKRM0602 Bank Rakyat, BMMB0341 Bank Muamalat, BSN0601 BSN, CIBBMYKL CIMB, DEUTMYKL Deutsche Bank, FNXSMYNB Finexus, HLB0224 / HLBBMYKL Hong Leong, HSBC0223 HSBC, KFH0346 KFH, MB2U0227 / MBBEMYKL Maybank, MFBBMYKL Alliance, OCBC0229 / OCBCMYKL OCBC, PBB0233 / PBBEMYKL Public Bank, PCBCMYKL CCBM, PHBMMYKL Affin, RHB0218 RHB, SCB0216 Standard Chartered, UOB0226 / UOVBMYKL UOB. Always use the live bank list.

### Payment (MTI 0880 / 0890)

`GET https://<host>/upp/faces/duitnowpaypayment.xhtml?<fields>`

Mandatory: `h001_MTI=0880`, `h002_VNO=06`, `h003_TDT`, `h004_TTM`, `f001_MID`, `f003_ProcCode=000000`, `f006_TxnDtTm` (YYYYMMDDhhmmss), `f007_TxnAmt`, `f010_CurrCode`, `f019_ExpTxnAmt`, `f249_TxnCh=WEB`, `f253_CntyCode=MY`, `f256_FICode=RPP`, `f260_ServID`, `f263_MRN`, `f264_Locale`, `f268_CHName` (1-100), trailer.

Optional: `f247`, `f248`, `f261`, `f262`, `f270_ORN`, `f271_ODesc`, `f275_RURL_DD` return URL, `f276_URL_VMPS` view payment status URL, `f278_EMailAddr`, `f279_HP`, `f285_IPAddr` (conditional), `f287`, `f352_AcqBank` (bank code from listing), `f400_AddData`, `f340_MercPromoCde`, `f341_MercPromoAmt`, `f342_PromoAmtAcqCst`, `f343_PromoAmtMercCst`, `f363_InvNum`, `f364_Fee`.

Response 0890 (browser GET redirect to `f275_RURL_DD`): `f003`, `f006`, `f007`, `f010`, `f019`, `f254_DDRespCode` (examine this; `00` success), `f256`, `f257_PGRN`, `f258`, `f260`, `f263`, `f270`, `f277_DDRN`, `f286_OrgDDRespCode`, `f407_PayerAccType`, optional `f259`, `f261`, `f262`, `f269_IssName`, trailer.

### Refund

`reversal0400.xhtml` with `f003_ProcCode=993000`, a new `f263_MRN`, `f023_RRN` of the original, other fields as section 4.

## 6. Card payment and recurring (MTI 0200 / 0210)

`GET https://<host>/upp/faces/ccpayment.xhtml?<fields>` (MPI `CCPayment`). Used for C04 recurring charges with a saved token.

Request fields in hash order: `f001_MID`, `f003_ProcCode` (`003000`), `f004_PAN` (card number or saved token), `f005_ExpDate`, `f006_TxnDtTm`, `f007_TxnAmt`, `f008_POSCond` (`73` recurring, `59` ecom, `08` MOTO), `f010_CurrCode`, `f012_CVV2`, `f014_3DXID`, `f015_3DARC`, `f016_3DCAVVLen`, `f017_3DCAVV`, `f019_ExpTxnAmt`, `f022_ECI` (`12` in recurring example), `f247`, `f248`, `f249_TxnCh`, `f260_ServID`, `f261`, `f262`, `f263_MRN`, `f264_Locale`, `f265_RURL_CCPS`, `f266_RURL_CCPU`, `f267_RURL_CCPC`, `f268_CHName`, `f269_IssName`, `f270_ORN`, `f271_ODesc`, `f278`, `f279`, `f285`, `f287`, `f288_IssCntrCde`, `f350_CrdTyp`, `f354_TID`, `f352_AcqBank`, `f353_IPPTenure`, `f325_ECommMercInd`, `f339_TokenFlg` (`Y`), `f340`-`f343` promo, `f344_MercCustId`, `f347_TokenShrtNm`, `f363_InvNum`, `f364_Fee`, `f365_POSEnvFlg`.

Response 0210: `f001`, `f003`, `f004`, `f005`, `f006`, `f007`, `f009_RespCode`, `f010`, `f011_AuthIDResp`, `f013_CVV2RsltCde`, `f019`, `f023_RRN`, `f024`, `f257_PGRN`, `f258`, `f259`, `f260`-`f263`, `f270`, `f354`, `f325`, `f339`, `f340`-`f344`, `f346_Token`, `f347`, `f348_MaskPAN`, `f350`, `f365`.

Token flow: first payment through UPP with `f339_TokenFlg=Y`, `f344_MercCustId`, `f347_TokenShrtNm`; store `f346_Token` from the response (it is a card token, keep it server side). Later charges send the token as `f004_PAN` with C04. Billing schedule: Not documented in official docs (checked 2026-10-21); the merchant schedules charges. Card data entry outside UPP needs PCI DSS scope; prefer UPP.

## 7. Pre-auth

- Authorize: UPP with `f362_PreAuthFlg=Y` (save-card flow adds `f339_TokenFlg=Y` and `f344_MercCustId`), or MT0100 `f003_ProcCode=303000` on `ccpayment.xhtml` (fields as MT0200).
- Complete (C13): MT0220 `f003=003000`, `f008=06` POST to `/upp/faces/preauthcomplete.xhtml` (one docs page lists `payment.xhtml`; the MPI uses `preauthcomplete.xhtml`). MT0220 fields: as MT0200 up to `f288`, then `f339`, `f365`. Response MT0230.
- Void pre-auth (C12): MT0100 `f003=023000`. Void completion (C14): MT0220 `f003=023000`. Reversal of pre-auth (C18): MT0400 `f003=303000`.
- Hold duration and partial capture rules: Not documented in official docs (checked 2026-10-21).

## 8. Dynamic QR (merchant presented)

Generate (server side): `POST https://<host>/upp/faces/generateQrPayload.xhtml?<fields>`, follow the redirect. MTI 0180.

Request: `f001_MID`, `f003_ProcCode=000000`, `f004_PAN` (sender ID), `f007_TxnAmt`, `f008_POSCond=67`, `f010_CurrCode`, `f019_ExpTxnAmt`, `f256_FICode`, `f260_ServID`, `f263_MRN`, `f265_RURL_CCPS` (callback URL), `f279_HP`, `f285_IPAddr`, `f289_CustId` (not hashed), `f354_TID`, `f366_PayloadInd=12`, `f367_PANInd` (`26` DuitNow QR, `97` Alipay+), `f368_AcctInd`, `f369_DevID`.

Response 0190: `f001`, `f004`, `f007`, `f009_RespCode`, `f010`, `f011`, `f019`, `f023_RRN`, `f258`, `f259`, `f263`, `f370_QRPayload` (render as QR; replace `+` with a space first), `f371_BarcodePayload`.

Callback MT0250, POSTed to `SuccRespURL`: `f001`, `f004`, `f007`, `f009_RespCode`, `f010`, `f011`, `f019`, `f023_RRN`, `f258`, `f259`, `f263`, `f370`, `f375_MrkUpFee`, `f376_DiscAmt`, `f377_DiscNote`, `f379_QrcVocNo`; docs also list `f350_CrdTyp` (e.g. `PN`), `f378_TxnId`, `f354_TID`, `addAmt`, and the trailer. Verify the hash, then return HTTP 200 OK.

Timing: if no callback after 30 seconds, query every 5 seconds for up to 5 minutes (the flow table says 3 minutes). Brand rules (design page): show "Powered by Finexus"; show an expiry countdown of 5 minutes for DuitNow QR and 3 minutes for Alipay+; Alipay+ QR printed size 25.4 to 30.5 mm.

## 9. Consumer presented QR (MTI 0240 / 0250)

`GET https://<host>/upp/faces/qrPushPayment.xhtml?<fields>` (PaymentID Q03). Fields: `f001`, `f003_ProcCode=003000`, `f004_PAN`, `f007`, `f008_POSCond` (69 on the CP QR page), `f010`, `f011`, `f019`, `f023`, `f247`, `f248`, `f253`, `f256`, `f260`, `f263`, `f265`, `f279`, `f285`, `f289_CustId` (not hashed), `f297_BussName`, `f304_BusZip`, `f305_BusStcde`, `f354_TID`, `f366_PayloadInd`, `f367_PANInd`, `f368_AcctInd` (`2`), `f369_DevID`, `f370_QRPayload` (scanned customer code), `f372_TxnFlag=CP`, `f373_MercPAN`, `f374_MCC`, `f378_TxnId`. Response 0250 as above. Refund uses C05 with the `RetrievalRefNo`.

## 10. Static QR push notification

Finexus POSTs JSON to the merchant primary callback URL (secondary as backup). Example endpoint in docs: `POST https://xxx/sb/PushMessageRequest`.

```
{ "message": { "ACQ", "deviceBrandModel", "deviceID", "lang" (EN/MS/CH), "trxRefNo" (30),
               "trxCurr" (MYR), "trxAmt", "trxDateTms" (YYYYMMDDhhmmssSSS),
               "trxPymtBrand" (AL Alipay+, AP Alipay, DN DuitNow; example shows PN),
               "MID", "TID" },
  "signMsg": "<base64 RSA SHA256withRSA PKCS#1 v1.5 signature by Finexus>" }
```

Merchant reply: `{"message":{"reqStat":..., "errDesc":...}, "signMsg":"<merchant signature>"}` within 5 seconds. De-duplicate on `trxRefNo`. Keys: merchant creates an RSA 2048 key (keytool JKS, SHA256withRSA), sends a CSR to Finexus, gets a signed cert and the Finexus public key; repeat for production. The exact bytes signed (`message` JSON serialization) are not specified beyond the examples; confirm with Finexus.

## Status codes (Appendix 3, `f009_RespCode`)

00 Approved; L0 Approved special condition; 01 / 02 Refer to issuer; 03 Invalid merchant; 04 / 07 Pick up card; 05 Do not honour; 06 Error; 11 Voided; 12 Invalid transaction; 13 Invalid amount; 14 Invalid card number; 19 Re-enter; 21 No action taken; 22 Pending (in progress); 23 Invalid fee; 25 Terminated card; 30 Format error; 31 Bank ID not found; 41 Lost card; 43 Stolen card; 44 PIN change required; 45 Card not activated; 51 Insufficient funds; 52 No checking account; 53 No savings account; 54 Expired card; 55 Invalid PIN; 56 Invalid card; 57 Not permitted to cardholder; 58 Invalid transaction; 59 Suspected fraud; 61 Over limit; 62 Restricted card; 63 Security violation; 68 Timeout (no reply from acquirer); 75 PIN tries exceeded; 76 Previous message not found; 77 Repeat / reversal inconsistent; 78 Card not activated yet; 80 Issuer unavailable; 83 Unable to verify PIN; 88 Call issuer; 91 Issuer / switch error; 92 Routing not found; 93 Violation of law; 94 Duplicate transaction; 95 Reconciliation mismatch; 96 System malfunction; 99 Unknown; P1 Failed 3D authentication; N7 CVV2 failure.

Treat `00` (and `L0`) as paid only after hash and amount checks. Treat `22` and `68` as unknown: query, do not fail the order yet. `f258_TxnStatDetCde=5015` from the MPI means invalid secure hash.

## Rate limit

Production blocks an IP that sends too many requests: blocked for 1 minute, plus 1 minute for each further attempt. The limit value is not published. Back off on query loops.

## Sandbox data

- Sandbox access comes after the MyXaaS application is approved (email with sandbox merchant business portal access).
- Sample secret key in docs (`00112233445566778899AABBCCDDEEFF`) and sample MIDs are examples for hash tests only.
- Test card numbers, test bank logins: Not documented in official docs (checked 2026-10-21). Ask Finexus.

## Plugins (official)

- WooCommerce plugin 4.0.5 (`woocommerce-plugin-4.0.5.zip`, linked from docs): settings Enable, Environment Mode (sandbox / production), Debug; properties `PayMasterUrl`, `RecurringUrl`, `CheckoutUrl`, `debug_log`. Recurring needs the paid WooCommerce Subscriptions extension. Warning: the plugin's return handler marks orders from a `result_mode=succ` query parameter in the merchant return URL and does not verify the hash there. Do not copy this pattern; verify the hash and query.
- OpenCart 4.0.3.2 plugin (`finexus_payment_gateway.ocmod.zip`): SandboxMode, MerchantID, HashKey, order statuses.
- Google Pay: through UPP when enabled by Finexus; Google Pay terms apply.
- Secure payment link: the merchant hosts its own unique link that maps to a UPP payment URL. No Finexus link API is documented.

## MPI quick reference

- Versions: Java 3.0.2 (JDK 8u162+), .NET Core 2.1, PHP 3.0.3 (PHP 7.4, `PayMaster.phar`), Python 3.8.5 (jproperties, zeep, pycryptodomex), Node.js 12+ (properties-reader), JavaScript test kit 1.2.0.
- Download path: `https://sandbox.finexusgroup.com/myxaas/assets/files/<Lang>-PayMaster-MPI-<ver>-<hash>.zip` (links on the UPP download-plugin page).
- Properties files: `UPPPayment`, `CCPayment`, `DDPayment`, `QRPayment`, `BankListing` (`.properties`, or `.ini` for PHP). Keys: `VersionNo`, `ServiceID`, `MerchantID`, `TxnChannel`, `LangLocale`, `SuccRespURL`, `UnsuccRespURL`, `CancelRespURL`, `QueryRespURL`, `ReversalRespURL`, `SHAlgorithmType`, `SecretKey`, `PaymasterEntryPoint`, `UPPSchema` / `CCSchema` / `FPXSchema` / `QRSchema` / `BankListingSchema`, `query_gateway`.
- Keep `SecretKey` out of committed properties files; load it from an environment variable or secret store.
