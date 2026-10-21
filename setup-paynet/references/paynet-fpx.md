# PayNet FPX Reference

FPX is PayNet's direct internet-banking debit service. This file is for an approved FPX merchant or TPA connecting to FPX itself. A merchant using a gateway (Billplz, iPay88, Fiuu, and so on) should follow that gateway's skill instead. Checked 2026-10-21.

Sources: `https://docs.paynet.my/docs/fpx/*` (b2c-overview, b2b-1-model, b2b-2-model, bulk-refund, exception-handling, browser-redirection, authentication, key-management, mapping-table, response-codes, resources, uat-test-cases, branding-guidelines) and the official sample source code `source-code-sample.zip` (ASP.NET, Java, PHP) linked from `/docs/fpx/resources`. The docs site does not publish the full AR/AC field specification (lengths, formats); field names and signing order below come from the official PHP sample.

## Models and message types

| Model | `fpx_msgToken` | Who authorises | Notes |
|---|---|---|---|
| B2C | `01` | Individual buyer | Browser flow |
| B2B1 | `02` | Corporate maker submits, authoriser approves later | First direct AC is `99` (pending authorisation), final direct AC `00` |
| B2B2 | `03` (PHP sample) | Batch from merchant system | Host-to-host over MPLS via SSL; first AC `99`, then `00` |

| `fpx_msgType` | Direction | Purpose |
|---|---|---|
| `AR` | Merchant to FPX | Authorisation request (payment) |
| `AC` | FPX to merchant | Authorisation response (direct host-to-host and indirect via browser) |
| `AE` | Merchant to FPX | Status enquiry (debit request exception flow); response is AC fields |
| `BE` | Merchant to FPX | Bank list enquiry |
| `BC` | FPX to merchant | Bank list response |
| `AF` | Merchant to FPX | Bulk refund batch |
| `AB` | FPX to merchant | Bulk refund status (submission `1S`, then `99`, then `00`) |
| `AQ` | Merchant to FPX | Bulk refund status enquiry; response AB |
| `AX` | Merchant to FPX | Merchant cancellation request |
| `AXA` | FPX to merchant | Cancellation response |
| `RR` / `RC` | FPX and acquiring bank | Credit request and credit response (not merchant-facing) |

## B2C flow (from `/docs/fpx/b2c-overview`)

1. Buyer chooses "Pay via FPX" and picks a bank from the list (from BE/BC).
2. Merchant sends AR to FPX via SSL (browser form POST, full window).
3. FPX redirects the buyer to the bank's internet banking login. Buyer logs in and approves with the bank's two-factor method.
4. Bank sends direct AC to FPX. FPX sends a host-to-host direct AC to the merchant, plus emails to merchant and buyer. "FPX will resend numbers of Direct AC whenever Merchant does not acknowledge Direct AC message." Merchant must reply `OK`.
5. Bank redirects the buyer to FPX; FPX redirects the buyer to the merchant with the indirect AC. Merchant shows the receipt.
6. FPX sends RR to the acquiring bank, which credits the merchant and returns RC.

B2B1 adds a maker/authoriser split: direct AC `99` after the maker submits, and a second direct AC `00` after the authoriser approves (steps 14 to 18 of `/docs/fpx/b2b-1-model`).

## Endpoints (UAT, from the official sample)

| Message | UAT URL | Transport |
|---|---|---|
| AR (B2C, B2B1) | `https://uat.mepsfpx.com.my/FPXMain/seller2DReceiver.jsp` | Browser form POST |
| AE | `https://uat.mepsfpx.com.my/FPXMain/sellerNVPTxnStatus.jsp` | Server POST, `key=value&...` |
| BE | `https://uat.mepsfpx.com.my/FPXMain/RetrieveBankList` | Server POST |
| B2B2 AR | `https://uat.mepsfpx.com.my/FPXMain/B2B2Payment` | Server POST |

Production URLs: Not documented in official docs (checked 2026-10-21). PayNet provides them at onboarding. The only production FPX URLs in the docs are `https://www.mepsfpx.com.my/FPXMain/termsAndConditions.jsp` and the DuitNow bridge `https://www.mepsfpx.com.my/FPXMain/r2pReceiver.jsp`. Where the merchant's direct and indirect AC URLs are registered: Not documented in official docs; set during PayNet onboarding.

## AR request fields (form POST)

All values are strings. Names from the official PHP sample `02_fpx_confirm.php`.

| Field | Sample value | Notes |
|---|---|---|
| `fpx_msgType` | `AR` | |
| `fpx_msgToken` | `01` | `01` B2C, `02` B2B1 (`03` in the B2B2 sample) |
| `fpx_sellerExId` | `EX0000xxxx` | Exchange ID assigned by PayNet (also the private key file name in samples) |
| `fpx_sellerExOrderNo` | `YYYYMMDDhhmmss` style | Unique per exchange; duplicates give `XN`/`XO` |
| `fpx_sellerTxnTime` | `20150911160824` | `YYYYMMDDhhmmss` |
| `fpx_sellerOrderNo` | your order number | Duplicates give `45` |
| `fpx_sellerId` | `SE0000xxxx` | Seller ID assigned by PayNet |
| `fpx_sellerBankCode` | `01` | As in sample |
| `fpx_txnCurrency` | `MYR` | |
| `fpx_txnAmount` | `100.00` | Ringgit with decimals, not sen |
| `fpx_buyerEmail` | buyer email or empty | Required to pass if you capture it (browser-redirection page) |
| `fpx_buyerName` | empty | See special characters below |
| `fpx_buyerBankId` | `TEST0021` | From the BC bank list |
| `fpx_buyerBankBranch` | empty | B2B2 sample sets a branch |
| `fpx_buyerAccNo` | empty | B2B2 sample sets an account |
| `fpx_buyerId` | empty | |
| `fpx_makerName` | empty | B2B |
| `fpx_buyerIban` | empty | |
| `fpx_productDesc` | `SampleProduct` | |
| `fpx_version` | `6.0` in sample | Docs BE example shows `7.0`; use the version PayNet assigns |
| `fpx_checkSum` | hex string | See signature |

## Signature (fpx_checkSum)

Rule from `/docs/fpx/authentication` ("FPX Services"): build the source string from all data element values, sorted by data element name in ascending order, separated by `|`; sign with the merchant private key using RSA with SHA-1; the signature is hex. The samples output upper-case hex (`strtoupper(bin2hex(...))`). Empty fields still contribute an empty slot between pipes.

AR and AE string-to-sign (both use the same order in the official sample):

```text
buyerAccNo|buyerBankBranch|buyerBankId|buyerEmail|buyerIban|buyerId|buyerName|makerName|msgToken|msgType|productDesc|sellerBankCode|sellerExId|sellerExOrderNo|sellerId|sellerOrderNo|sellerTxnTime|txnAmount|txnCurrency|version
```

(each item is the value of `fpx_<name>`). BE string-to-sign: `msgToken|msgType|sellerExId|version`, docs sample `01|BE|EX00002200|7.0`.

AC verify string (direct AC, indirect AC and AE response):

```text
buyerBankBranch|buyerBankId|buyerIban|buyerId|buyerName|creditAuthCode|creditAuthNo|debitAuthCode|debitAuthNo|fpxTxnId|fpxTxnTime|makerName|msgToken|msgType|sellerExId|sellerExOrderNo|sellerId|sellerOrderNo|sellerTxnTime|txnAmount|txnCurrency
```

BC verify string: `bankList|msgToken|msgType|sellerExId`.

Pseudo-code:

```text
sign(fields):
  names = sorted keys used for this message (list above)
  s = join("|", [fields["fpx_" + n] or "" for n in names])
  sig = RSA_PKCS1v15_sign(SHA1, merchant_private_key, utf8(s))
  return upper(hex(sig))

verify_ac(post):
  s = join("|", [post["fpx_" + n] or "" for n in AC_NAMES])
  for cert in [fpx_current_cert, fpx_old_cert]:     # skip expired certs
      if RSA_PKCS1v15_verify(SHA1, cert.public_key, utf8(s), unhex(post["fpx_checkSum"])): return true
  return false
```

Use the values exactly as posted (do not reformat `fpx_txnAmount`). Reject the message and do not settle if verification fails (the sample returns error `09`). Certificate errors in the sample: `06` no valid certificate, `07` one certificate found and expired, `08` both expired.

FPX Bank Webview APIs (banks only) use a different scheme: OAuth 2.0 plus an `X-Signature` header holding the RSA-SHA1 signature over key fields (same page).

## Direct and indirect AC handling

- Direct AC (server to server): verify `fpx_checkSum`, store the result, then respond with the body `OK`. FPX resends until acknowledged, so make the handler idempotent on `fpx_sellerExOrderNo` + `fpx_fpxTxnId`.
- Indirect AC (browser POST to your return page): verify `fpx_checkSum` too, but use it only to show the receipt. Settle from the direct AC or AE.
- Status: `fpx_debitAuthCode` decides the receipt. `00` successful, `99` pending authorisation (B2B), `09` pending, any other code failed. Show also transaction date, amount, seller order number, FPX transaction ID (`fpx_fpxTxnId`), buyer bank name and status on the receipt (mandatory receipt fields).
- B2B1/B2B2: `99` is not final. Wait for the second direct AC with `00`, or poll with AE.

## AE status enquiry

Use AE when the direct AC did not arrive ("Debit Request Exception Flow"). Send the same fields as the original AR with `fpx_msgType=AE`, signed with the AR order above, as an `application/x-www-form-urlencoded` POST to the AE URL. The response body is `key=value` pairs joined with `&`, URL-encoded; verify it with the AC verify string. Pass `fpx_buyerEmail` if you captured it. Polling interval and limits: Not documented in official docs (checked 2026-10-21).

The PHP sample disables TLS verification (`CURLOPT_SSL_VERIFYPEER` false). Do not copy that.

## Bank list (BE/BC)

- Send BE every time you display the bank list (browser-redirection page). Response `fpx_bankList` is a comma-separated list of `bankId~status`. Status `B` (blocked) must be shown as "Offline".
- Live bank IDs listed on `/docs/fpx/mapping-table`: ABB0232, ABB0233 (Affin B2C), ABMB0212 (Alliance B2C), ABMB0213 (Alliance B2B), AMBB0208 (AmBank B2B), AMBB0209 (AmBank B2C), BCBB0235 (CIMB), BIMB0340 (Bank Islam), BMMB0341, BMMB0342 (Muamalat B2B), BKRM0602 (Bank Rakyat B2C), BSN0601, DBB0199 (Deutsche), HLB0224, HLB0225 (HLB B2B2), HSBC0223, KFH0346, MB2U0227 (Maybank M2U), MBB0227 (Maybank M2E), MBB0228 (Maybank B2B), OCBC0229, PBB0233 (Public Bank), RHB0218, SCB0215 (SCB B2B), SCB0216 (SCB B2C), TPAGHL, TPAIPAY88, TPAMOLPAY, TPAREVENUE, UOB0226 (UOB B2C), UOB0227 (UOB B2B1), UOB0228 (UOB B2B1 Regional). Always prefer the live BC response over this static list.
- Sandbox-only test banks: `TEST0021` SBI Bank A, `TEST0022` SBI Bank B, `TEST0023` SBI Bank C.
- The mapping table also lists business category codes and MSIC codes (linked to the LHDN MSIC PDF).

## Allowed special characters

`@ / \ ( ) space . - _ , & ’`. For buyer name additionally: `` ` ~ * " ; : ``.

## Merchant page rules (`/docs/fpx/browser-redirection`)

- Redirect with a FORM POST over SSL. Show the bank page in the full browser window. No pop-up, no new tab, no iframe.
- Show the FPX logo, FPX operating hours and the FPX information link; follow the FPX Brand Guideline v1.2 and Merchant UI Guideline v1.0 PDFs (`/docs/fpx/branding-guidelines`).
- Show an FPX Terms and Conditions statement linking to the T&C URL (opens a new window). Clicking "Proceed" means the buyer accepts it.
- Pages must work on major mobile browsers (Safari, Chrome, Android browser, Firefox).
- Do NOT whitelist FPX IPs in your firewall. Use PayNet's URLs for direct and indirect messages.
- Follow FPX minimum and maximum limits, but do not hardcode them (FPX may change them). UAT test script: B2C minimum RM1.00, B2B1 minimum RM2.00 (sandbox values from the UAT test case file).
- Support automatic replacement of the FPX certificate when PayNet issues a new one.

## Keys and certificates (`/docs/fpx/key-management`)

1. Generate a 2048-bit RSA key: `openssl genrsa -out <file_name_key>.key 2048`.
2. Create a CSR: `openssl req -out <file_name_csr>.csr -key <file_name_key>.key -new -sha256` (leave challenge password and optional company name blank).
3. UAT: self-signed certificates are accepted; submit through the Developer Portal project. PayNet approves and uploads it in FPX Webview, and returns your exchange certificate.
4. Production: PayNet approves; the merchant receives the certificate from MSC Trustgate.
5. FPX certificate: download the latest from `/docs/fpx/resources`. Renewal naming: `fpxuat.cer` becomes `fpxuat_current.cer`. The sample checks the current and old certificate files, and renames on rollover. Production file names: Not documented in official docs.
6. Keep the private key in a secure device or secret store. Signed values are hex.

## Bulk refund (AF/AB/AQ)

Flow from `/docs/fpx/bulk-refund`: merchant sends a batch of AF messages host-to-host over the internet via SSL (including buyer bank and buyer bank account); FPX returns AB `1S` (bulk refund successfully submitted); the bank returns direct AB `99`, then `00` when approved; merchant must answer each direct AB with `OK`. AQ asks for bulk refund status and returns a direct AB. AF/AB field list: Not documented in official docs (checked 2026-10-21); a "Bulk Refund Test Script for Merchant V1.4.xlsx" is linked from `/docs/fpx/uat-test-cases`. Bulk refund extended codes B0 to B9 and C1 to C5 are in `paynet-codes.md`.

## Cancellation (AX/AXA)

Merchant sends AX host-to-host, FPX answers with AXA ("Merchant Cancelation Request Status Flow"). Field list: Not documented in official docs. Code `2X` = transaction cancelled by merchant.

## Sandbox tools (sandbox-only)

- FPX Merchant Simulator: `https://simulator.fpx.uat.inet.paynet.my/uatSeller1.10/sellermain.jsp`.
- FPX B2B Bank Simulator: `https://simulator.fpx.uat.inet.paynet.my/UatBuyerBankSim1.7/B2BAuth.jsp`.
- UAT bank simulator login published in the UAT test case file: User ID `1234`, Password `1234` (sandbox-only).
- Message signature checker app `signer-fpx-ui.zip`, Java SDK, and sample code `source-code-sample.zip` on `/docs/fpx/resources`. Treat these as reference; generate your own keys.
- UAT test scripts: Merchant UAT Test Script for B2C and B2B1 v2.2, Bulk Refund v1.4, Payment Account Indicator (eKYC) v1.1.

## Settlement checklist

- Local order exists, `fpx_sellerOrderNo` and `fpx_sellerExOrderNo` match, `fpx_txnAmount` equals the stored amount, `fpx_txnCurrency` is `MYR`, `fpx_sellerExId` and `fpx_sellerId` are yours.
- `fpx_checkSum` verified with a non-expired FPX certificate.
- `fpx_debitAuthCode` is `00`. For `99` or `09` keep the order pending and requery with AE.
- Store `fpx_fpxTxnId`, `fpx_fpxTxnTime`, `fpx_buyerBankId`, `fpx_debitAuthNo`, `fpx_creditAuthCode` for reconciliation.
- FPX settlement reports: see `/docs/operations/reports/fpx` and the Financial Reports API FPX specification (`/docs/operations/financial-reports/financial-reports-api/fpx-specifications`).
