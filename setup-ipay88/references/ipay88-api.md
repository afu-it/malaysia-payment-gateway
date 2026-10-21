# iPay88 Malaysia API Reference

Source: iPay88's own Technical Specification v1.6.4.4 (11 Sep 2024), not published on an iPay88 domain; obtained from a community mirror (`https://github.com/karyamedia/ipay88/blob/master/iPay88%20iPay88%20Technical%20API%20v1.6.4.4.pdf`). Merchants receive the current spec from iPay88 Support; confirm against it.

The document title is "Online Payment Switching Gateway (OPSG) Technical Specification, For Malaysia Only, Version 1.6.4.4". Section numbers below (2.3, 3.1, ...) refer to it. The request signature rule is also confirmed on iPay88's official tester page, `https://payment.ipay88.com.my/epayment/testing/TestSignatureIpay88.asp` (checked 2026-10-21), which is the authority for the request signature.

## Source URLs

- Spec v1.6.4.4 (mirror): `https://raw.githubusercontent.com/karyamedia/ipay88/master/iPay88%20iPay88%20Technical%20API%20v1.6.4.4.pdf`
- Official signature tester: `https://payment.ipay88.com.my/epayment/testing/TestSignatureIpay88.asp`
- Official payment page host: `https://payment.ipay88.com.my/epayment/entry.asp`
- Official re-query host: `https://payment.ipay88.com.my/epayment/enquiry.asp`
- Official webservice description (WSDL, served by iPay88): `https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx?WSDL`
- Official webservice operation page (SOAP 1.1, SOAP 1.2, HTTP GET, HTTP POST samples): `https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx?op=TxDetailsInquiryCardInfo`
- iPay88's own support FAQ "For Web Developer" (www.ipay88.com, now offline; archived copy): `https://web.archive.org/web/2024/https://www.ipay88.com/support/`

Source priority: the official pages on `payment.ipay88.com.my` (tester, WSDL) win over the mirrored spec where they differ. The mirrored spec is iPay88-authored but was not obtained from iPay88; treat it as "iPay88 text, unverified copy".

## Integration Model

iPay88 OPSG is a redirect (HTML form POST) gateway, not a JSON REST API.

1. Merchant checkout page (the registered "Request URL") POSTs hidden form fields to `entry.asp`.
2. Customer picks or confirms the payment method on the iPay88 page, then pays at the bank, card page or e-wallet.
3. iPay88 POSTs the result with a Signature to the merchant ResponseURL (through the browser).
4. For successful payments only, iPay88 also POSTs the same fields server to server to the BackendURL, at the same time as the ResponseURL post.
5. Merchant verifies the signature and amount, can re-query, then updates the order once.

Service hours: 7x24, excluding host downtime (section 1.6).

## Environments

| Item | Value |
| --- | --- |
| Payment page | `https://payment.ipay88.com.my/epayment/entry.asp` (sample form uses `https://payment.ipay88.com.my/ePayment/entry.asp`) |
| Re-query | `https://payment.ipay88.com.my/epayment/enquiry.asp` |
| Webservice requery | `https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx?op=TxDetailsInquiryCardInfo` |
| Signature tester | `https://payment.ipay88.com.my/epayment/testing/TestSignatureIpay88.asp` |
| Merchant report | `https://payment.ipay88.com.my/ePayment/report` |

Sandbox: no separate sandbox host or public test credentials are documented in the Malaysia spec (checked 2026-10-21). The spec says test transactions must come from the registered Request URL and use MYR 1.00, and that "any valid credit card" can be used for testing (FAQ 4). `https://sandbox.ipay88.com.my/` answers but is not mentioned in the spec; do not use it unless iPay88 Support tells you to.

Localhost is not allowed as a Request URL (section 1.3).

## Authentication

There is no API key header. Authentication is:

- `MerchantCode`: public identifier issued by iPay88.
- Merchant Key: shared secret issued by iPay88, used as the HMAC key and as the first part of the string to sign. Never sent as a field. iPay88 re-issues it only if it suspects compromise or fraud (section 3.2).
- Registered Request URL (referrer check) and whitelisted server IPs for ResponseURL and BackendURL (section 1.3). A request from a non-registered referrer fails with "Permission not allow".

## Endpoint Catalogue

| # | Direction | Method | URL | Purpose | Section |
| --- | --- | --- | --- | --- | --- |
| 1 | Merchant browser to iPay88 | POST (form) | `/epayment/entry.asp` | Payment request; opens the iPay88 payment page | 2.1, 2.3, 2.4 |
| 2 | iPay88 to merchant (browser) | POST (form) | merchant `ResponseURL` | Payment result, success or fail | 2.5, 2.6 |
| 3 | iPay88 to merchant (server) | POST (form) | merchant `BackendURL` | Backend post, success only; must answer `RECEIVEOK` | 2.7, 2.10 |
| 4 | Merchant server to iPay88 | POST (spec text) / GET (spec sample) | `/epayment/enquiry.asp` | Re-query status, plain-text reply | 2.11 |
| 5 | Merchant server to iPay88 | SOAP 1.1 POST | `/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx` (SOAPAction `https://www.mobile88.com/epayment/webservice/TxDetailsInquiryCardInfo`) | Webservice requery with card details | 2.12, WSDL |
| 5b | Merchant server to iPay88 | SOAP 1.2 POST | same `.asmx` URL, `Content-Type: application/soap+xml` | Same operation, SOAP 1.2 binding | WSDL |
| 5c | Merchant server to iPay88 | HTTP POST (form) | `/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx/TxDetailsInquiryCardInfo` | Same operation, form fields in, plain `TransactionInquiry` XML out | WSDL |
| 5d | Merchant server to iPay88 | HTTP GET | same path as 5c with a query string | Same operation (binding declared in WSDL) | WSDL |
| 6 | Developer | GET | `/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx?WSDL` | Machine-readable service description | WSDL |
| 7 | Developer | Browser | `/epayment/testing/TestSignatureIpay88.asp` | Check a generated signature | 3.1, 3.2 |
| 8 | Merchant | Browser login | `/ePayment/report` | Online transaction report | 4.2 |

All paths are on `https://payment.ipay88.com.my`. This is every operation in the spec and on the official host (checked 2026-10-21). The `.asmx` service lists exactly one operation, `TxDetailsInquiryCardInfo`.

Not publicly documented (checked 2026-10-21): refund API, void, capture, cancel, recurring or subscription, card tokenisation, payout, settlement report API, sandbox host. iPay88's own product page for Recurring Payment (archived `https://web.archive.org/web/2024/https://www.ipay88.com/ecommerce/recurring-payment/`) says recurring, tokenisation, a mobile SDK and a server-to-server "external API" exist, but no spec for them is public. The WSDL response carries refund, capture, void and token fields (section 5), which shows these features exist on the platform, but there is no public operation to start them. Do not invent them; ask iPay88 Support.

## 1. Payment Request (entry.asp)

HTTPS POST of form fields. Legend: M mandatory, O optional. For O fields "value can be empty but parameter must exist", so always send every field.

| Field | Type | Size | M/O | Notes |
| --- | --- | --- | --- | --- |
| `MerchantCode` | String | 20 | M | Merchant Code from iPay88. |
| `PaymentId` | Integer | | O | Payment method ID. Values in iPay88 "Appendix I.pdf" (MYR gateway) and "Appendix II.pdf" (multi-currency). Empty in the spec sample. |
| `RefNo` | String | 30 | M | Unique merchant transaction number. |
| `Amount` | Currency | | M | Two decimals with thousand separators, for example `1,278.99`. |
| `Currency` | String | 5 | M | For example `MYR`. Allowed values in Appendix I / II. |
| `ProdDesc` | String | 100 | M | Product description. |
| `UserName` | String | 100 | M | Customer name. |
| `UserEmail` | String | 100 | M | Customer email; receives the receipt. |
| `UserContact` | String | 20 | M | Customer contact number. |
| `Remark` | String | 100 | O | Merchant remarks. |
| `Lang` | String | 20 | O | Encoding: `ISO-8859-1` English, `UTF-8` Unicode, `GD18030` Chinese Simplified (spelling as in spec), `BIG5` Chinese Traditional. |
| `SignatureType` | String | 10 | M | `HMACSHA512`. |
| `Signature` | String | 128 | M | HMAC-SHA512 request signature (section 3.1 below). |
| `ResponseURL` | String | 200 | M | Merchant response page. |
| `BackendURL` | String | 200 | M | Merchant backend post page. |
| `Optional` | JSON object | | O | `{"carddetails":"Y"}` returns card details in the response. `{"MDRValues":"Y"}` returns finance charge values. Both: `{"carddetails":"Y","MDRValues":"Y"}`. |
| `Appdeeplink` | String | 100 | O | Merchant mobile app deeplink, mobile app only (section 2.8). Sample form uses lowercase `appdeeplink`. |
| `Xfield1` | String | 100 | O | Custom field. |

Sample (spec 2.4, abbreviated; the spec says it is reference only):

```html
<form method="post" name="ePayment" action="https://payment.ipay88.com.my/ePayment/entry.asp">
  <input type="hidden" name="MerchantCode" value="<MERCHANT_CODE>">
  <input type="hidden" name="PaymentId" value="">
  <input type="hidden" name="RefNo" value="A00000001">
  <input type="hidden" name="Amount" value="1.00">
  <input type="hidden" name="Currency" value="MYR">
  <input type="hidden" name="ProdDesc" value="Photo Print">
  <input type="hidden" name="UserName" value="John Tan">
  <input type="hidden" name="UserEmail" value="john@example.com">
  <input type="hidden" name="UserContact" value="0126500100">
  <input type="hidden" name="Remark" value="">
  <input type="hidden" name="Lang" value="UTF-8">
  <input type="hidden" name="SignatureType" value="HMACSHA512">
  <input type="hidden" name="Signature" value="<128_HEX_CHARS>">
  <input type="hidden" name="ResponseURL" value="https://shop.example.com/payment/ipay88/response">
  <input type="hidden" name="BackendURL" value="https://shop.example.com/payment/ipay88/backend">
  <input type="hidden" name="Optional" value='{"carddetails":"Y","MDRValues":"Y"}'>
  <input type="hidden" name="appdeeplink" value="">
  <input type="hidden" name="Xfield1" value="">
</form>
```

HTML-escape the `Optional` JSON (the spec sample nests double quotes inside a double-quoted attribute, which breaks the attribute; use single quotes or `&quot;`).

### PaymentId values

The numeric PaymentId list is not in the v1.6.4.4 spec. It lives in separate iPay88 files "Appendix I.pdf" (MYR) and "Appendix II.pdf" (multi-currency) that iPay88 Support sends with the spec. The only value shown in the spec is `PaymentId = "2"` in the response signature example and the webservice sample (a VISA card transaction). Rules:

- Leave `PaymentId` empty to let the customer choose a method on the iPay88 page.
- Send a specific ID only for methods iPay88 has enabled on your account; otherwise you get "Payment not allowed".
- Store the `PaymentId` returned in the response; it is part of the response signature.
- Do not copy PaymentIDs from iPay88 Indonesia's "API 2.0" document (`https://ipay88.co.id/docs/pdf/iPay88_API_2.0.9_Technical_Documentation.pdf`, section 8.4 "Malaysia Payment Method", 4-digit IDs such as `4210` Boost). That is a different iPay88 gateway (Indonesia, JSON API 2.0); nothing says its IDs are valid on the Malaysia OPSG.
- Community lists of Malaysia PaymentIds exist (for example on Scribd and in GitHub libraries) but are not iPay88-published, can be years old, and are not reproduced here.

## 2. Payment Response (ResponseURL and BackendURL)

HTTPS POST from iPay88. Same fields on both URLs (section 2.7).

| Field | Type | Size | M/O | Notes |
| --- | --- | --- | --- | --- |
| `MerchantCode` | String | 20 | M | |
| `PaymentId` | Integer | | M | Method used. |
| `RefNo` | String | 30 | M | Your transaction number. |
| `Amount` | Currency | | M | Two decimals with thousand separators, for example `1,278.99`. |
| `Currency` | String | 5 | M | |
| `Remark` | String | 100 | O | |
| `TransId` | String | 30 | O | iPay88 transaction ID (for example `T099959151521`). |
| `AuthCode` | String | 20 | O | Bank approval code. |
| `Status` | String | 1 | M | `1` success, `0` fail. |
| `ErrDesc` | String | 100 | O | Status description (values in Appendix I / II). |
| `Signature` | String | 128 | M | HMAC-SHA512 response signature (section 3.2 below). |
| `CCName` | String | 200 | O | Card or account holder name. |
| `CCNo` | String | 16 | O | Cards only. Masked, first six and last four, for example `492159xxxxxx4941`. |
| `S_bankname` | String | 100 | O | Cards only. Issuing bank name. |
| `S_country` | String | 100 | O | Cards only. Issuing country. |
| `TranDate` | Datetime | 20 | O | Transaction date. |
| `Optional` | JSON object | | O | `{"carddetails":{"paymenttype":"LOCAL DEBIT CARD","cardtype":"VISA","expirydate":"07/23"}}` and/or `{"MDRValues":{"finance":"0.00","GSTAmount":"0.00","total":"0.00"}}`. The ASP/PHP samples read it as lowercase `optional`. |
| `Xfield1` | String | 100 | O | Custom field. |

The PHP sample also reads `Xfield2` and `Xfield3`, which are not in the field table. Read them defensively; do not depend on them.

The spec notes the response signature is present only when the transaction is not in error, that is for approved and declined transactions (section 3.2).

### ResponseURL handling

- Browser-delivered: can be lost (closed browser, timeout) or forged.
- Security controls from section 3.0: check `HTTP_REFERER` is `https://payment.ipay88.com.my` (only meaningful over SSL), check the amount matches yours, and compare the Signature with your own.
- Show the customer a result page. You may settle here only after full verification, and only once.

### BackendURL handling (Backend Post, section 2.7)

- Server to server, independent of the browser.
- Sent ONLY for successful payments. No backend post for failed payments.
- iPay88 posts to ResponseURL and BackendURL simultaneously after success. Decide which one updates the order so the status is not updated more than once.
- Must perform the same checks as the response page (signature and so on).
- Must not contain session-related code, so it works after the user logs out or the session expires.
- After successfully recording the payment, output exactly `RECEIVEOK` with no HTML tags and nothing else (PHP `echo "RECEIVEOK";`).
- If `RECEIVEOK` is not detected, iPay88 retries the backend post up to 3 times at different intervals.
- The backend page does not replace the response page; keep both.

Pseudo-code for both routes:

```text
fields = parse form body
expected = hmac_sha512_hex(key=MERCHANT_KEY,
           MERCHANT_KEY + fields.MerchantCode + fields.PaymentId + fields.RefNo
           + strip(fields.Amount, ".", ",") + fields.Currency + fields.Status)
if not constant_time_equal(lower(expected), lower(fields.Signature)): reject (do not print RECEIVEOK)
order = find_order(fields.RefNo)
check fields.MerchantCode == MERCHANT_CODE
check normalize(fields.Amount) == order.amount and fields.Currency == order.currency
if fields.Status == "1":
    optional: requery enquiry.asp and require "00"
    in one transaction: if order.status != "paid": mark paid, store TransId, PaymentId, AuthCode
    backend route: print "RECEIVEOK"
else:
    mark failed only if still pending (response route only; backend never receives fails)
```

## 3. Signatures (section 3.0 to 3.3)

Algorithm: HMAC-SHA512, key = Merchant Key (UTF-8 bytes), message = concatenated fields (UTF-8), output = lowercase hex (`b.ToString("x2")` in the .NET sample), 128 characters. PHP: `hash_hmac('sha512', $toEncrypt, $merchantKey)`.

SHA256 deprecation: spec v1.6.4.4 changelog (Ahmad Amin, 11 Sep 2024) says "Update from SHA256 to HMACSHA512 for Request and Response". Older specs (v1.6.1 SHA1, v1.6.2 to v1.6.4.3 SHA256) and libraries that send `SignatureType=SHA256` are out of date. The official tester still offers SHA1 and SHA256 options; do not take that as permission to use them.

Amount rule for both signatures: remove "." and "," from the Amount string before hashing. `1.00` becomes `100`; `1,278.99` becomes `127899`. Hash the same Amount string you posted or received. Note the tester page JavaScript uses a single `.replace(".", "")` and `.replace(",", "")`, which strips only the first comma; for amounts with two or more thousand separators (1,000,000.00 and above) remove all of them per the spec and confirm with iPay88.

### 3.1 Request signature

Authority (official tester page, `updateSignature()`):

```text
string = MerchantKey + MerchantCode + RefNo + Amount(no "." or ",") + Currency
Signature = hex(HMAC_SHA512(key = MerchantKey, string))
```

Spec v1.6.4.4 section 3.1 lists a sixth field and says the order is `MerchantKey & MerchantCode & RefNo & Amount & Currency & Xfield1`.

Worked examples (key `apple`, MerchantCode `M00003`, RefNo `A00000001`, Amount `1.00`, Currency `MYR`; recomputed with openssl on 2026-10-21 and matching the spec):

| Variant | String to sign | Signature |
| --- | --- | --- |
| Xfield1 empty (tester form; spec 2.4 sample) | `appleM00003A00000001100MYR` | `c41859fed9b4ac135990cc10737ff3a37ad1e0eaa3e0f2235a652172fe3465413449b2e73acb782bb14a1b2e32792e91dc03fd2da355432de86df6730ed70e9a` |
| Xfield1 = `xx11` (spec 3.1 example) | `appleM00003A00000001100MYRxx11` | `d8e0b9807bdd10527267f6433f9487155f56d42fdd23d3d76f2315bf72fdb50806e81e5a42491c64eb368c9e9165364fa3b011dcc81963b2fd465cfcdae77288` |

### Open question: Xfield1 in the request signature

- The tester page form builds the string without Xfield1. The spec 3.1 text and example append Xfield1.
- When Xfield1 is empty (or not used), both variants give the same string, so they agree. This is the safe default: send `Xfield1` as an empty field.
- If you need a non-empty Xfield1: build `MerchantKey+MerchantCode+RefNo+Amount+Currency+Xfield1`, paste it into the tester page "Full string to hash (optional)" box with your Merchant Key in the key box and `HMAC-SHA512` selected, then run a MYR 1.00 test. If iPay88 answers "Signature not match", drop Xfield1 from the string and ask iPay88 Support which rule applies to your account.

### 3.2 Response signature

```text
string = MerchantKey + MerchantCode + PaymentId + RefNo + Amount(no "." or ",") + Currency + Status
Signature = hex(HMAC_SHA512(key = MerchantKey, string))
```

Example (spec 3.2): key `apple`, PaymentId `2`, Status `1` gives string `appleM000032A00000001100MYR1` and signature `2b8bcaf66881a9f02a7a2b470cbdf958a86d92f2169774de1062db0376545b061c98f78827db0dbe8aacf908982d5777f577370e19658649a9794c7168b5a5d7` (recomputed, matches). Xfield1 is not part of the response signature.

Short Node.js example:

```js
import crypto from "node:crypto";
const strip = (a) => String(a).replace(/[.,]/g, "");
const hmac = (key, s) => crypto.createHmac("sha512", key).update(s, "utf8").digest("hex");

function requestSignature(key, code, refNo, amount, currency, xfield1 = "") {
  return hmac(key, key + code + refNo + strip(amount) + currency + xfield1);
}
function responseValid(key, f) {
  const exp = hmac(key, key + f.MerchantCode + f.PaymentId + f.RefNo + strip(f.Amount) + f.Currency + f.Status);
  const got = String(f.Signature || "").toLowerCase();
  return got.length === exp.length && crypto.timingSafeEqual(Buffer.from(exp), Buffer.from(got));
}
```

The spec mentions a "Sample Code to Generate HMACSHA512 Signature" folder (C#, Java, JavaScript, PHP, VB.Net) shipped with the spec; it is not in the mirror.

## 4. Re-query (enquiry.asp, section 2.11)

The spec text says HTTPS POST; its PHP sample sends a GET with a query string. Both carry:

| Field | Type | Size | M/O | Notes |
| --- | --- | --- | --- | --- |
| `MerchantCode` | String | 20 | M | |
| `RefNo` | String | 30 | M | Your transaction number / order ID. |
| `Amount` | Currency | | M | Two decimals, same value as the payment. |

Plain-text replies:

| Reply | Meaning |
| --- | --- |
| `00` | Successful payment |
| `Invalid parameters` | Parameters passed in are incorrect |
| `Record not found` | Record cannot be found |
| `Incorrect amount` | Amount differs |
| `Payment fail` | Payment failed |
| `M88Admin` | Payment status updated by iPay88 Admin (Fail) |

Trim whitespace and compare the whole body. Treat only `00` as paid. A GET with a query string was observed to work on 2026-10-21 (unknown RefNo returned `Record not found`, HTTP 200), matching the spec's PHP sample. URL-encode `Amount` if it contains a comma.

## 5. Webservice requery (SOAP, section 2.12)

URL: `https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx?op=TxDetailsInquiryCardInfo` (the PDF wraps it as `TxDe tailsInquiryCardInfo`; there is no space). Use it after an Appdeeplink return or when you need card details.

Request fields:

| Field | Type | Size | M/O | Notes |
| --- | --- | --- | --- | --- |
| `MerchantCode` | String | 20 | M | |
| `ReferenceNo` | String | 50 | M | Your RefNo (note the different field name). |
| `Amount` | Currency | | M | Two decimals. |
| `Version` | String | 2 | M | Use `4`. |

```xml
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <TxDetailsInquiryCardInfo xmlns="https://www.mobile88.com/epayment/webservice">
      <MerchantCode><MERCHANT_CODE></MerchantCode>
      <ReferenceNo>A00000001</ReferenceNo>
      <Amount>1.00</Amount>
      <Version>4</Version>
    </TxDetailsInquiryCardInfo>
  </soap:Body>
</soap:Envelope>
```

### Transport details (official WSDL and operation page, checked 2026-10-21)

The spec omits these; they come from the service description iPay88 serves at `TxDetailsInquiry.asmx?WSDL`.

| Item | Value |
| --- | --- |
| Endpoint (all bindings) | `https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx` (the `?op=...` suffix in the spec is the help page, not needed when posting) |
| Service / operation | `Transaction_Inquiry_CardDetails` / `TxDetailsInquiryCardInfo`, document/literal |
| Target namespace | `https://www.mobile88.com/epayment/webservice` |
| SOAP 1.1 headers | `Content-Type: text/xml; charset=utf-8` and `SOAPAction: "https://www.mobile88.com/epayment/webservice/TxDetailsInquiryCardInfo"` (with the double quotes) |
| SOAP 1.2 | `Content-Type: application/soap+xml; charset=utf-8`, same action |
| HTTP POST | `POST .../TxDetailsInquiry.asmx/TxDetailsInquiryCardInfo`, `application/x-www-form-urlencoded` body `MerchantCode=...&ReferenceNo=...&Amount=...&Version=4` |
| HTTP GET | `GET .../TxDetailsInquiry.asmx/TxDetailsInquiryCardInfo?MerchantCode=...&ReferenceNo=...&Amount=...&Version=4` (declared in the WSDL; not probed) |
| Request element types | All four inputs are `xs:string`, `minOccurs=0` in the WSDL; the spec still marks all four mandatory. |

Response wrapping differs by binding:

- SOAP: `soap:Envelope/soap:Body/TxDetailsInquiryCardInfoResponse/TxDetailsInquiryCardInfoResult/<fields>`.
- HTTP GET/POST: bare `<TransactionInquiry xmlns="https://www.mobile88.com/epayment/webservice">` with the fields as children. The spec 2.12.2 sample shows this bare form even though it pairs it with a SOAP request.

Parse by local element name inside the namespace, and accept both shapes.

Minimal call (HTTP POST binding, placeholders):

```bash
curl -s https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx/TxDetailsInquiryCardInfo \
  --data-urlencode "MerchantCode=<MERCHANT_CODE>" --data-urlencode "ReferenceNo=<REF_NO>" \
  --data-urlencode "Amount=1.00" --data-urlencode "Version=4"
```

Observed behaviour (unknown merchant and RefNo, checked 2026-10-21, both SOAP and HTTP POST): HTTP 200 with `Status` empty and `Errdesc` = `Record not found`; `MerchantCode` is absent. So "no row" is not an HTTP error and not `Status=0`. Treat empty `Status` as "unknown, not paid".

### Response fields (spec 2.12 table)

Root `TransactionInquiry` (or `TxDetailsInquiryCardInfoResult` under SOAP), namespace `https://www.mobile88.com/epayment/webservice`:

| Field | Type | Size | M/O | Notes |
| --- | --- | --- | --- | --- |
| `MerchantCode` | String | 20 | M | |
| `PaymentId` | Integer | | M | |
| `RefNo` | String | 50 | M | |
| `Amount` | Currency | | M | Two decimals with thousand separators. |
| `Currency` | String | 5 | M | |
| `Remark` | String | 100 | O | |
| `TransId` | String | 30 | O | iPay88 transaction ID. |
| `AuthCode` | String | 50 | O | Bank approval code. |
| `Status` | String | 2 | M | `1` Success, `0` Fail, `6` Payment Pending, `20` Authorised. |
| `Errdesc` | String | 100 | O | Status description. |
| `CCName` | String | 200 | O | |
| `Creditcardno` | String | 16 | O | Masked card number. |
| `CardType` | String | 20 | O | For example `VISA`, `MASTERCARD`, `AMEX`. |
| `S_bankname` | String | 100 | O | Issuing bank. |
| `S_country` | String | 100 | O | Issuing country, for example `MY`. |
| `BankMID` | String | 100 | O | Bank MID that processed the transaction. |
| `TranDate` | Datetime | 20 | O | For example `2020-02-25`. |

### Additional response elements declared in the WSDL only

The WSDL `TransactionInquiry` type declares these extra elements (all `xs:string`, optional). The spec does not describe them, so their values and formats are not documented. Read them if present; never base a paid/unpaid decision on them.

| Group | Elements |
| --- | --- |
| Refund | `RefundTranx` (one `RefundTransaction`), `Refund` (array of `RefundTransaction`), `RefundStatus` |
| Capture (pre-auth) | `CaptureTranx` (one `CaptureTransaction`), `Capture` (array of `CaptureTransaction`), `CaptureStatus`, `CapturedTransId` |
| Void | `VoidStatus` |
| Card / 3-D Secure | `ThreeDStatus`, `CardCategory`, `U_bankname`, `U_country`, `Iplocation` |
| DCC (currency conversion) | `DCCStatus`, `OriginalAmount`, `OriginalCurrency`, `SettlementAmount`, `DCCConversionRate`, `SettlementCurrency` |
| Other status | `TransStatus`, `ResultCode`, `Discount`, `PaymentType` |
| Dates and names | `Transactiondate`, `Creationdate`, `OrderID`, `CustomerName`, `CollectorName`, `CollectorID`, `ProdDesc` |
| Token | `Tokenid`, `BindCardErrDesc` |
| Custom / extra | `Xfield1`, `Xfield2`, `optional` (lowercase), `Signature` |

`RefundTransaction` children: `Status`, `Description`, `Amount`, `RefundType`, `xField1`, `xField2`, `CreateDate`.
`CaptureTransaction` children: `CapturePayId`, `CaptureId`, `CaptureType`, `CaptureAmount`, `CaptureCreateDate`, `CaptureStatus`.

Element order in the WSDL sequence starts with `RefundTranx`, `CaptureTranx`, then `MerchantCode` ... `Errdesc`, `Refund`, `Capture`, ...; a strict schema-validating client must accept that order. The spec field table and sample carry no Signature, and the WSDL `Signature` element has no documented string-to-sign; do not try to verify it. Trust the webservice reply because you called iPay88 over HTTPS. Settle only on `Status` `1`. Keep `6` pending and re-check later. The spec does not explain when `20` (Authorised) is returned or whether it is capturable; do not treat it as paid without iPay88 confirmation.

## 6. Status values

| Context | Values |
| --- | --- |
| ResponseURL / BackendURL `Status` | `1` success, `0` fail |
| enquiry.asp reply | `00`, `Invalid parameters`, `Record not found`, `Incorrect amount`, `Payment fail`, `M88Admin` |
| Webservice `Status` | `1`, `0`, `6` pending, `20` authorised; empty with `Errdesc` `Record not found` when no transaction matches (observed) |
| Payment request `SignatureType` | `HMACSHA512` (current). The official tester also lists legacy `SHA1`, `SHA256`. |
| `Lang` | `ISO-8859-1`, `UTF-8`, `GD18030`, `BIG5` |
| `Optional` request keys | `carddetails`, `MDRValues` (value `Y`) |

## 7. Error messages on the payment page (FAQ 6 and 7)

| Message | Cause |
| --- | --- |
| `Duplicate reference number` | RefNo already used by a successful payment. |
| `Invalid merchant code` | Merchant code missing or wrong. |
| `Invalid parameters` | A posted parameter is invalid or empty. |
| `Overlimit per transaction` | Amount exceeds the per-transaction limit on the account. |
| `Payment not allowed` | Requested payment method not enabled for the account. |
| `Permission not allow` | Referrer URL differs from the registered Request URL. |
| `Signature not match` | Request signature generated incorrectly (section 3.1). |
| `Status not approved` | Merchant account suspended or not active. |
| `The return page URL not exist` | Wrong merchant code, Request URL not registered, or ResponseURL not set. |
| `Fail (Card issuing bank do not honor the transaction)` (ErrDesc) | Card issuer declined; customer should ask the bank whether the card works online. |

## 8. Appdeeplink and e-wallets (sections 2.8, 2.9)

- `Appdeeplink` is only for a merchant mobile app paying with app-to-app e-wallets such as TNG eWallet and Boost. Do not send it for mobile browser checkouts.
- After payment the e-wallet app opens your deeplink with no response data. Call the webservice requery to get status; iPay88 also sends the success backend post to BackendURL.
- ShopeePay in an Android WebView fails with `ERR_UNKNOWN_URL_SCHEME`. Override `shouldOverrideUrlLoading` and open URLs starting with `shopeemy://` or `https://wsa.wallet.airpay.com.my/` with an external intent outside the WebView. Mobile web only flows are not affected.

## 9. Amount and currency rules

- Amounts are ringgit with two decimals, not sen. Format `1.00`, `1,278.99`.
- Signature uses the amount with "." and "," removed.
- Compare amounts numerically after removing commas; do not compare raw strings with different formatting.
- Test amounts (FAQ 5): MYR, USD, AUD, CAD, EUR, GBP, SGD `1.00`; HKD `2.50`; IDR `3000.00`; INR, PHP, THB, TWD `15.00`. Non-MYR currencies need the multi-currency gateway (Appendix II).
- Currency codes offered by the official signature tester dropdown (checked 2026-10-21): `MYR`, `CNY`, `USD`, `GBP`, `THB`, `CAD`, `SGD`, `AUD`, `EUR`, `HKD`, `IDR`, `PHP`. The authoritative per-account list is Appendix I / II. iPay88's support FAQ (archived) lists the same test amounts as the spec.

## 10. Reports and email

- Online report at `https://payment.ipay88.com.my/ePayment/report` with a login from iPay88; this is the primary channel to confirm status besides the APIs.
- Customer receipt and merchant notification emails come from `no-reply@ipay88.com.my`. iPay88 does not guarantee email delivery; never use email as a payment trigger.

## 11. Not documented (checked 2026-10-21)

- No public operation for refunds, voids, captures for `20` Authorised, recurring payments, tokenisation or payouts. iPay88 markets recurring payments, tokenisation, a mobile SDK and an "external API" (archived ipay88.com Recurring Payment page), and the WSDL response has refund, capture, void and token fields, but the request side of those features is not published. Older iPay88 materials mention a separate "iPay88 Recurring Payment" product with its own application form.
- Also not published: rate limits, iPay88 outbound IP list (for allowlisting their posts to you), sandbox host and test credentials, the PaymentId table, the `Errdesc` value list (Appendix I / II), the meaning of the WSDL-only response elements.
- Ask iPay88 Support (`support@ipay88.com.my`) for each of these.
