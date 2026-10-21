# Paydee API Reference

Use this reference for factual Paydee Payment Acceptance Gateway (MPI) guidance. Primary source is the official spec PDF v5.0.0 (1 December 2025) linked from `https://developer.paydee.co/`. Where the developer site, Swagger JSON, or message-table XLSX disagree with the PDF, this file says so under "Documented conflicts". Checked 2026-10-21.

Full code tables (response codes, referral codes, 3DS statuses) live in `references/paydee-codes.md`.

## Source URLs

- `https://developer.paydee.co/` (Getting Started, same content as `/api/get-started`)
- `https://developer.paydee.co/api/use-cases`
- `https://developer.paydee.co/api/message-type`
- `https://developer.paydee.co/download/Paydee_Payment_Acceptance_Gateway_API_Spec_v5.0.0.pdf`
- `https://developer.paydee.co/download/Paydee_Payment_Acceptance_Gateway_API_Spec_v4.0.0-2.pdf` (previous version)
- `https://developer.paydee.co/download/DevBox.v2.5.xlsx` (message table)
- `https://developer.paydee.co/static/sales_v2_4.json` and `https://developer.paydee.co/static/preauth_v2_4.json` (OpenAPI 3.0, "3DS Merchant Interface" v2.4)
- `https://developer.paydee.co/api/mac-validation` (login-gated; only a login form is public)

## Endpoint Catalogue

All paths are relative to `https://<endpoint>`. The spec does not list any required HTTP headers (section 2.1 lists none). The message-type page shows `content-type: application/x-www-form-urlencoded` for MPI requests.

| Method | Path | Body | Purpose |
|---|---|---|---|
| POST | `/mkReq` | JSON | Key exchange. Must be called first for every transaction ID. Exchanges per-session RSA public keys. |
| POST | `/mercReq` | Form data | Hosted payment page. Paydee renders checkout. Trans types `SALES`, `INITRECURR`, `PRERECURR` (PDF); site also routes `PREAUTH`, `INITINSTL` through hosted or non-hosted. |
| POST | `/mpReq` | Form data | Non-hosted. Merchant-built checkout for `SALES`, plus server operations `INQ`, `VSALES`, `REFUND`, `RECURR`, `TERMINATE` (PDF), and `INSTL`, `SALESCOMPL`, `PREAUTH`, `INITINSTL` (developer site). |
| GET | `/channels` | JSON (per spec) | Get Payment Channels: list of FPX banks and e-wallets with Active/Blocked status (added v4.0.0). |

Transaction types (`MPI_TRANS_TYPE`) across all sources:

| Value | Meaning | Triggered by | Needs `MPI_ORI_TRXN_ID` |
|---|---|---|---|
| `SALES` | Sale / purchase, renders checkout | Cardholder browser | No |
| `VSALES` | Void an unsettled sale | Merchant server | Yes |
| `REFUND` | Refund a settled sale, partial allowed | Merchant server | Yes |
| `INQ` | Inquiry of current status | Merchant server | Yes |
| `PRERECURR` | Pre-recurring, validates card, not settled (non-chargeable) | Cardholder browser | No |
| `INITRECURR` | Initiate recurring, settled (chargeable) | Cardholder browser | No |
| `RECURR` | Charge a recurring cycle | Merchant server | Yes (the INITRECURR/PRERECURR ID) |
| `INITINSTL` | Initiate installment plan | Cardholder browser | No |
| `INSTL` | Charge an installment | Merchant server | Yes (the INITINSTL ID) |
| `TERMINATE` | End recurring or installment programme | Merchant server | Yes |
| `PREAUTH` | Reserve amount without settlement (cards) | Cardholder browser | No |
| `SALESCOMPL` | Complete a PREAUTH with final amount | Merchant server | Yes (the PREAUTH ID) |

The Swagger enum spells pre-recurring as `RERECUUR`; that is a typo. Use `PRERECURR` as in the PDF and XLSX.

## Environments

- Test server (from the official Swagger `servers` entry, "MPI Proxy Server"): `https://devlink.paydee.co/mpi`. The developer site says test endpoint URLs are "locked" and shown after sign-in, so confirm the exact URL in the portal.
- Production: "Production endpoints require authenticated credentials to unlock." Not published.
- XLSX note: in the UAT environment Paydee appends a 2-character suffix to your transaction ID.

## Integration Flow

1. Create local pending payment with a new transaction ID `T`.
2. Generate an RSA-2048 key pair for this session (site: "Generate a new pair of RSA keys" for every new payment session; Swagger: "A new pair of RSA keys is generated for each payment session").
3. `POST /mkReq` with `purchaseId = T`. Store Paydee's returned `pubKey` with the payment.
4. Build the `MPI_*` request with `MPI_TRXN_ID = T`, sign `MPI_MAC` with the session private key.
5. Hosted: browser form-post to `/mercReq`. Non-hosted SALES: call `/channels` first, then post to `/mpReq` with `MPI_PAYMENT_CHANNEL_ID`; the payer's browser is redirected to the selected channel, then back to the merchant status page using the redirect URL.
6. Response arrives "in Form Data through redirect and callback URL" (hosted) or as JSON/STRING/HTML (non-hosted, see `MPI_RESPONSE_TYPE`). Verify `MPI_MAC` with Paydee's session `pubKey`.
7. Follow-up operation on `T` (INQ, VSALES, REFUND, RECURR, INSTL, TERMINATE, SALESCOMPL): new transaction ID `T2`, new `/mkReq` with `purchaseId = T2`, then `/mpReq` with `MPI_TRXN_ID = T2` and `MPI_ORI_TRXN_ID = T`.

Callback and redirect URL: no request field carries a return or callback URL in any public source. Not documented in official docs (checked 2026-10-21); confirm with Paydee how they are registered (likely at onboarding).

## Key Exchange: `POST /mkReq`

Request (JSON):

| Field | Type (size) | Presence | Notes |
|---|---|---|---|
| `merchantId` | AN(15) | M | Merchant ID assigned by the acquirer. |
| `purchaseId` | AN(20) | M | Transaction ID, unique per merchant, minimum 6 digits. Existing (merchantId, purchaseId) pair = duplicate, invalid. |
| `pubKey` | AN(392) | M | Merchant RSA-2048 public key, Base64Url encoded (samples begin `MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A`, i.e. X.509 SubjectPublicKeyInfo DER). |
| `mac` | ANS(344) | C | Key-exchange MAC. Mandatory only if the merchant enrolled key-exchange MAC verification. |
| `paymentMethod` | A(8) | O | `CC`, `EWALLETS`, `FPX`, `TRD` (Trade Credit), `DNQR` (DuitNow QR). Default: customer selects; the chosen method then returns in `MPI_CUST_PAYMENT_METHOD`. |

Response (JSON):

| Field | Type (size) | Presence | Notes |
|---|---|---|---|
| `merchantId` | AN(15) | ME | Echo. |
| `purchaseId` | AN(20) | ME | Echo. |
| `pubKey` | AN(392) | C | Paydee server RSA-2048 public key for this session, Base64Url. Present if `errorCode` is `000`. |
| `errorCode` | AN(3) | M | `000` success; see codes file. |
| `errorDesc` | AN(100) | O | Present if not `000`. |

## Hosted Payment Page: `POST /mercReq`

Request (form data). Presence: M mandatory, O optional, C conditional, C1 mandatory where a billing address exists, C2 at least one of the group.

| Field | Type (size) | Presence | Notes |
|---|---|---|---|
| `MPI_TRANS_TYPE` | AN(10) | M | `SALES`, `INITRECURR`, `PRERECURR`. |
| `MPI_MERC_ID` | N(15) | M | Merchant ID. |
| `MPI_PURCH_AMT` | N(19) | M | Minor units, no decimal point. MYR 200.00 = `20000`. |
| `MPI_PURCH_CURR` | N(3) | M | ISO 4217 numeric, MYR = `458`. |
| `MPI_TRXN_ID` | N(20) | M | Same as `purchaseId` in `/mkReq`. |
| `MPI_PURCH_DATE` | N(14) | M | `yyyyMMddHHmmss` 24h. |
| `MPI_ADDR_MATCH` | A(1) | O | `Y` shipping equals billing, `N` not. |
| `MPI_BILL_ADDR_CITY` | AN(50) | C1 | |
| `MPI_BILL_ADDR_STATE` | AN(3) | C1 | ISO 3166-2 subdivision code (sample `05`). |
| `MPI_BILL_ADDR_CNTRY` | N(3) | C1 | ISO 3166-1 numeric (MY = `458`). |
| `MPI_BILL_ADDR_POSTCODE` | N(16) | C1 | |
| `MPI_BILL_ADDR_LINE1`..`LINE3` | AN(50) each | C1 | |
| `MPI_SHIP_ADDR_CITY` | AN(50) | O | |
| `MPI_SHIP_ADDR_STATE` | AN(3) | O | ISO 3166-2. |
| `MPI_SHIP_ADDR_CNTRY` | AN(3) | O | ISO 3166-1 numeric. |
| `MPI_SHIP_ADDR_POSTCODE` | AN(16) | O | |
| `MPI_SHIP_ADDR_LINE1`..`LINE3` | AN(50) each | O | |
| `MPI_EMAIL` | AN(254) | M | Required to offer Trade Credit. |
| `MPI_HOME_PHONE_CC` / `MPI_HOME_PHONE` | N(3) / AN(15) | C2 | ITU E.164. |
| `MPI_MOBILE_PHONE_CC` / `MPI_MOBILE_PHONE` | N(3) / AN(15) | C2 | |
| `MPI_WORK_PHONE_CC` / `MPI_WORK_PHONE` | N(3) / AN(15) | C2 | |
| `MPI_MAC` | AN(344) | M | Request MAC, see "MAC signing". |
| `MPI_TTL_QUANTITY` | N | C | Sum of item quantities; mandatory if `MPI_LINE_ITEM` present. |
| `MPI_LINE_ITEM` | List | O | Items, see below. |
| `MPI_ADDITIONAL_INFO_IND` | A(1) | O | `Y` returns 3DS detail fields; default `N`. |
| `MPI_RECURR_TYPE` | AN(30) | C | `day` or `month`. Mandatory for `INITRECURR`. |
| `MPI_RECURR_COUNT` | N(3) | C | Interval of recurrence. Mandatory for `INITRECURR`. |
| `MPI_RECURR_TOTAL` | N(3) | C | Total occurrences. Mandatory for `INITRECURR`. |
| `MPI_RECURR_DAY` | N(2) | C | Pre-defined billing day. Optional for `PRERECURR`, not applicable for `INITRECURR`. |

`MPI_LINE_ITEM` format: per item `MPI_ITEM_ID;MPI_ITEM_REMARK;MPI_ITEM_QUANTITY;MPI_ITEM_AMOUNT;MPI_ITEM_CURRENCY` (ID AN(10) M, remark AN(50) O, quantity N(10) M integer, amount N(12) M minor units, currency N(3) M). Sample: `1;store A;1;20000;458`. How multiple items are separated is not shown in the spec.

Response (form data via redirect and callback URL):

| Field | Type (size) | Presence | Notes |
|---|---|---|---|
| `MPI_MERC_ID` | AN(15) | ME | Echo. |
| `MPI_TRXN_ID` | AN(20) | ME | Echo. |
| `MPI_MAC` | AN(344) | M | Server response MAC. |
| `MPI_ERROR_CODE` | AN(10) | M | `000` success. |
| `MPI_ERROR_DESC` | AN(100) | C | Present if not `000`. |
| `MPI_APPR_CODE` | AN(6) | C | Cards only, when `000`. |
| `MPI_RRN` | AN(12) | C | Cards only, when `000`. |
| `MPI_BIN` | N(8) | O | First 8 card digits. |
| `MPI_REFERRAL_CODE` | ANS(100) | O | For `512`, `579`, `PAG100` (and `PAG200`/`PAG300` per code tables). |
| `MPI_ECI_VALUE` | N(2) | C | Cards; only if `MPI_ADDITIONAL_INFO_IND=Y`. |
| `MPI_CHALLENGE_MANDATED_IND` | A(1) | C | `Y`/`N`; only if additional info. |
| `MPI_CHALLENGE_IND` | A(1) | C | `Y` challenged / `N`; only if additional info. |
| `MPI_AUTH_STATUS` | A(1) | C | 3DS transStatus; only if additional info. |
| `MPI_STATUS_REASON` | N(2) | C | 3DS reason; only if additional info. |
| `MPI_STATUS_REASON_DESC` | AN(100) | C | Only if additional info. |
| `MPI_STATUS` | N(3) | C | Payment status (table below). |
| `MPI_CUST_PAYMENT_METHOD` | A(8) | C | Selected method (`CC`, `EWALLETS`, `FPX`, `TRD`, `DNQR`). |
| `MPI_REF_ID` | AN(20) | M | Gateway-generated unique ID per payment request. |
| `MPI_BUYER_BANK` | AN(100) | C | FPX only, buyer bank name. |

## Non-Hosted: `POST /mpReq`

Request fields are the hosted fields above (same types; `MPI_TRANS_TYPE` takes `INQ`, `VSALES`, `REFUND`, `RECURR`, `TERMINATE`, `SALES` in the PDF), plus:

| Field | Type (size) | Presence | Notes |
|---|---|---|---|
| `MPI_ORI_TRXN_ID` | N(20) | C | Original transaction ID. Not needed for `SALES`; required for VSALES/REFUND/INQ/RECURR/INSTL/TERMINATE/SALESCOMPL. |
| `MPI_RESPONSE_TYPE` | A(10) | O | `JSON`, `STRING` (form-data string); default HTML posted to callback URL. For `SALES` an HTML page with redirection is returned. |
| `MPI_PAYMENT_CHANNEL_ID` | AN(30) | C | Channel ID from `/channels` (e.g. `TEST002`). Mandatory for `SALES`. |
| `MPI_PAN` | N(16) | C | Card number, no spaces (developer site/XLSX/Swagger only; PCI DSS scope). |
| `MPI_PAN_EXP` | N(4) | C | `MMYY`. |
| `MPI_CVV2` | N | C | |
| `MPI_CARD_HOLDER_NAME` | AN | M/C | Visa rule #2 requires it for card SALES/PREAUTH/INIT*. The PDF v1.0.2 removed it from the hosted page. |

The PDF's non-hosted request table does not list the recurring fields; the developer site and XLSX list them for PRE/INIT RECURR and INITINSTL (see "Recurring and installment").

Response fields: same as hosted response, minus `MPI_REF_ID` and `MPI_BUYER_BANK`. 3DS detail fields and `MPI_STATUS` are returned for `INQ` (when `MPI_ADDITIONAL_INFO_IND=Y`). Special `MPI_ERROR_CODE` meanings:

- For `INQ`, the code is the status of the original transaction: `000` successful, `001` failed, `002` cancelled, `003` timeout/expired, `004` in processing. Any other code means the inquiry message itself failed.
- For `REFUND`: `004` refund in progress; run another INQ and read `MPI_STATUS`.
- Developer site: VOID/REFUND may reply code `00`, which only means "received and will be processed". Perform INQ about 30 minutes later; `MPI_STATUS` shows the reversal result.

## Get Payment Channels: `GET /channels`

Request (JSON): `MPI_MERC_ID` N(15) M, `MPI_TRXN_ID` N(20) M (the `/mkReq` purchaseId), `MPI_MAC` AN(344) M.

Response (JSON): `MPI_MERC_ID`, `MPI_TRXN_ID`, `MPI_MAC`, `MPI_ERROR_CODE`, `MPI_ERROR_DESC`, `MPI_PAYMENT_CHANNEL_LIST` = array of `{ "MPI_PAYMENT_METHOD": "FPX", "MPI_PAYMENT_CHANNEL": ["PBB001=A", "TEST002=A", "TEST003=B"] }`. `A` active, `B` blocked. Blocked channels must be shown disabled and not selectable. Retrieve the list frequently for non-hosted flows.

## MAC Signing And Verification

Algorithm (spec 2.13): SHA256withRSA (RSA PKCS#1 v1.5 with SHA-256, the Java `SHA256withRSA` name), output Base64Url encoded. Sign with your own private key; the peer verifies with your public key. Keys are RSA-2048.

String-to-sign = the field values concatenated in the order below with no separators. Absent fields contribute nothing (the request sample omits empty `MPI_ORI_TRXN_ID`).

Key exchange request `mac` (only if enrolled; signed with the enrolled long-term key, verified with the public key from the enrollment file):

1. `merchantId` 2. `purchaseId` 3. `pubKey` 4. `paymentMethod`

Hosted and non-hosted request `MPI_MAC` (merchant session private key):

1. `MPI_TRANS_TYPE` 2. `MPI_MERC_ID` 3. `MPI_TRXN_ID` 4. `MPI_ORI_TRXN_ID` 5. `MPI_PURCH_DATE` 6. `MPI_PURCH_CURR` 7. `MPI_PURCH_AMT` 8. `MPI_ADDR_MATCH` 9. `MPI_BILL_ADDR_CITY` 10. `MPI_BILL_ADDR_STATE` 11. `MPI_BILL_ADDR_CNTRY` 12. `MPI_BILL_ADDR_POSTCODE` 13. `MPI_BILL_ADDR_LINE1` 14. `MPI_BILL_ADDR_LINE2` 15. `MPI_BILL_ADDR_LINE3` 16. `MPI_SHIP_ADDR_CITY` 17. `MPI_SHIP_ADDR_STATE` 18. `MPI_SHIP_ADDR_CNTRY` 19. `MPI_SHIP_ADDR_POSTCODE` 20. `MPI_SHIP_ADDR_LINE1` 21. `MPI_SHIP_ADDR_LINE2` 22. `MPI_SHIP_ADDR_LINE3` 23. `MPI_EMAIL` 24. `MPI_HOME_PHONE` 25. `MPI_HOME_PHONE_CC` 26. `MPI_WORK_PHONE` 27. `MPI_WORK_PHONE_CC` 28. `MPI_MOBILE_PHONE` 29. `MPI_MOBILE_PHONE_CC` 30. `MPI_TTL_QUANTITY` 31. `MPI_LINE_ITEM` (each item as `ID;REMARK;QTY;AMOUNT;CURRENCY`, `;` kept) 32. `MPI_RESPONSE_TYPE` 33. `MPI_ADDITIONAL_INFO_IND` 34. `MPI_RECURR_TYPE` 35. `MPI_RECURR_COUNT` 36. `MPI_RECURR_TOTAL` 37. `MPI_RECURR_DAY` 38. `MPI_PAYMENT_CHANNEL_ID`

Note: phone order in the MAC is number then country code (`MPI_HOME_PHONE`, then `MPI_HOME_PHONE_CC`), and work comes before mobile. Card fields (`MPI_PAN` etc.) are not in the MAC list.

Spec sample before signing (shows values with spaces removed, `;` kept):

```text
SALES6000000000000016487047256202204051530304582000YSeremban0545872120No6JlnKwongTungTamanBaiduriSeremban0545872120No6JlnKwongTungTamanBaiduritest@email.com641134506123454506321565601911;storeA;1;2000;458JSONYday3512TEST002
```

Hosted and non-hosted response `MPI_MAC` (verify with Paydee's session public key from `/mkReq`):

1. `MPI_MERC_ID` 2. `MPI_TRXN_ID` 3. `MPI_ERROR_CODE` 4. `MPI_APPR_CODE` 5. `MPI_RRN` 6. `MPI_BIN` 7. `MPI_REFERRAL_CODE` 8. `MPI_ECI_VALUE` 9. `MPI_CHALLENGE_IND` 10. `MPI_CHALLENGE_MANDATED_IND` 11. `MPI_AUTH_STATUS` 12. `MPI_STATUS_REASON` 13. `MPI_STATUS_REASON_DESC` 14. `MPI_STATUS` 15. `MPI_CUST_PAYMENT_METHOD`

Sample: `6000000000000016487047256001194705208900055026410000001YYY01CardAuthenticationfailed-1CC` (desc "Card authentication failed" appears with spaces removed). `MPI_REF_ID` and `MPI_BUYER_BANK` are not in the MAC, so do not trust them without INQ.

Get channels request `MPI_MAC`: `MPI_MERC_ID` + `MPI_TRXN_ID`.

Get channels response `MPI_MAC`: `MPI_MERC_ID` + `MPI_TRXN_ID` + `MPI_ERROR_CODE` + for each list entry `MPI_PAYMENT_METHOD` followed by each `MPI_PAYMENT_CHANNEL` string. Sample: `6000000000000016487047258001FPXPBB001=ATEST002=ATEST002=AEWALLETSBOOST=AGRABPAY=BSHOPEEPAY=A`.

Pseudo-code (Python `cryptography`):

```python
import base64
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import rsa, padding

def b64url(b): return base64.urlsafe_b64encode(b).decode()  # sample MACs keep '=' padding
def b64url_dec(s): return base64.urlsafe_b64decode(s + "=" * (-len(s) % 4))

priv = rsa.generate_private_key(public_exponent=65537, key_size=2048)  # per session
pub_spki = priv.public_key().public_bytes(serialization.Encoding.DER,
            serialization.PublicFormat.SubjectPublicKeyInfo)
pubKey = b64url(pub_spki).rstrip("=")  # enrollment spec: URL-safe, '=' omitted

REQ_ORDER = ["MPI_TRANS_TYPE","MPI_MERC_ID","MPI_TRXN_ID","MPI_ORI_TRXN_ID","MPI_PURCH_DATE",
  "MPI_PURCH_CURR","MPI_PURCH_AMT","MPI_ADDR_MATCH", ...]  # full 38-field list above

def sign(fields, order):
    s = "".join(fields.get(k, "") for k in order)  # confirm whitespace rule with Paydee
    return b64url(priv.sign(s.encode("utf-8"), padding.PKCS1v15(), hashes.SHA256()))

def verify(fields, order, paydee_pub_b64url):
    pub = serialization.load_der_public_key(b64url_dec(paydee_pub_b64url))
    s = "".join(fields.get(k, "") for k in order)
    pub.verify(b64url_dec(fields["MPI_MAC"]), s.encode("utf-8"),
               padding.PKCS1v15(), hashes.SHA256())  # raises on mismatch
```

Text encoding (UTF-8), padding scheme name, and whitespace stripping are not stated in the spec; the PKCS#1 v1.5 choice follows from the `SHA256withRSA` name. Test against the sandbox: `5A0` = MAC verification failed.

## Enrollment File (key-exchange MAC)

To enroll a long-term public key for key-exchange MAC verification, submit a file line: `<merchantId max 15>|<Base64 URL-safe public key max 392>` where URL-safe means `+` to `-`, `/` to `_`, `=` omitted. Example shape: `000000000000001|MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...IDAQAB`. The site calls this "Step 00 (One Time Setup) Upload a public RSA key to Paydee's MPI Server".

## Payment Status (`MPI_STATUS`)

| Value | Meaning |
|---|---|
| `0` | Sales Requested |
| `1` | Payment Success |
| `2` | Payment Cancelled |
| `3` | Reverse Requested |
| `4` | Reverse Success |
| `5` | Refund Requested |
| `6` | Refund Success |
| `7` | Payment Settled |
| `8` | Recurring Payment Terminated |
| `-1` | Payment Failed |
| `-2` | Reverse Failed |
| `-3` | Refund Failed |
| `-4` | Sales Request Failed |

Settlement rule: `MPI_ERROR_CODE=000` and `MPI_STATUS` in (`1`, `7`) with matching IDs and amount.

## Void And Refund

- VSALES voids an unsettled sale (never appears on statements, no processing fee). REFUND reverses a settled sale; partial refunds allowed. Both are async; confirm with INQ.
- Void/Refund reference (PDF v5 section 2.3, cut-off for void, credit to buyer):

| Channel | Supports | Void cut-off | Credit within |
|---|---|---|---|
| Credit/Debit card | Void, Refund | 11:30pm GMT+8 | 2 business days |
| Boost | Void, Refund | 11:59pm GMT+8 | 1 business day |
| TnG eWallet | Void, Refund | 11:59pm GMT+8 | 1 business day |
| GrabPay | Void, Refund | 11:59pm GMT+8 | 1 business day |
| Maybank QR Push | Refund only per site (manual) | none | 7 business days |
| Alipay | Void, Refund | 11:59pm GMT+8 | 1 business day |
| ShopeePay | Void, Refund | 11:59pm GMT+8 | 1 business day |
| UnionPay | Void, Refund | not stated | not stated |
| Online Banking (FPX) | listed, but refund function removed in v3.0.3 | not stated | not stated |
| DuitNow QR | Void, Refund | 11:59pm GMT+8 | 1 business day |

E-wallet processing is "Auto" per the developer site; Maybank QR Push is "Manual".

## PreAuth And Sales Complete

`PREAUTH` (browser) authenticates the cardholder and reserves an amount without settlement (cards only). `SALESCOMPL` (server, `MPI_ORI_TRXN_ID` = PREAUTH ID) charges the final amount. If the final amount exceeds the PREAUTH amount by 20%, it is rejected; completing for less is allowed. Paydee suggests PREAUTH a pre-agreed amount plus expected extras, or complete and start a new PREAUTH.

## Recurring And Installment (Split Payment, cards)

- `PRERECURR`: validates the card (charges RM1.00 or a merchant-set amount), not settled, so the amount is returned. `INITRECURR`: settled (end of day, 11 p.m.). Both from the cardholder browser with 3DS.
- `RECURR` / `INSTL`: merchant server triggers each cycle with `MPI_ORI_TRXN_ID` = the INITRECURR/PRERECURR/INITINSTL transaction ID. No cardholder interaction.
- `TERMINATE`: ends the programme.
- Control parameters (site): Frequency (days between payments), Expiry, Count, Maximum Amount, Total Amount, Terminate.
- Recurring errors `PAG06` to `PAG13` (start date, expired, cycle already paid, not initialised, original not success, terminated, cancelled, expired).

## Documented Conflicts (confirm with Paydee)

1. Recurring field names differ by source:
   - PDF v5 (hosted): `MPI_RECURR_TYPE` (`day`/`month`), `MPI_RECURR_COUNT`, `MPI_RECURR_TOTAL`, `MPI_RECURR_DAY`. These are the ones in the MAC order.
   - Developer site message table: `MPI_RECURRING_FREQUENCY`, `MPI_RECURR_EXPIRY`, `MPI_RECURR_MAX_CNT`, `MPI_RECURR_MAX_AMT`, `MPI_RECURR_MAX_TOTAL`.
   - XLSX v2.5: `MPI_RECURRING_FREQUENCY` (days, e.g. `30`), `MPI_RECURR_EXPIRY` (`yyyymmdd`), `MPI_RECURR_MAX_AMT`, `MPI_RECURR_MAX_TOTAL`.
   - Swagger v2.4: `MPI_RECURRING_FREQUENCY`, `MPI_RECURRING_EXPIRY`, `MPI_PURCHASE_INSTAL_DATA`, `MPI_RECURRING_MAX_PURCHASE_AMOUNT`, `MPI_RECURRING_TOTAL_PURCHASE_AMOUNT`.
   Prefer the PDF v5 names for `INITRECURR`/`PRERECURR`; ask Paydee which names apply to `INITINSTL`.
2. Response MAC: the ordered list has 15 fields including `MPI_REFERRAL_CODE` with `MPI_CHALLENGE_IND` before `MPI_CHALLENGE_MANDATED_IND`; the index table under it has 14 entries without referral code. Use the 15-field list; the sample is consistent with it when referral code is empty.
3. Whitespace: both MAC samples show values with internal spaces removed, but the spec never says to strip them. Test both against the sandbox.
4. Request MAC sample amount `2000` differs from the sample request `20000`; samples are illustrative, not consistent test vectors.
5. Transaction ID type: PDF types `MPI_TRXN_ID` as `N(20)`; `purchaseId` is `AN(20)`; XLSX example is `de20121212121212`. Use digits only to be safe.
6. XLSX describes `MPI_PURCH_DATE` as `yyyymmddd` with example `20121212`; the PDF says `yyyyMMddHHmmss`. Follow the PDF.
7. Void/refund ack code: PDF says REFUND returns `004` in progress; developer site says `00`. Treat either as non-final.
8. `/channels` is documented as GET with a JSON body; confirm whether Paydee also accepts POST.

## Test Data (sandbox only, published on the developer site)

Cardholder name `Test User`, expiry `2608` (MMYY), CVV `123`.

| Scenario | Visa | Mastercard |
|---|---|---|
| Success, frictionless (ARes transStatus Y) | `4100000000000100` | `5100000000000107` |
| Success, challenge (ARes C, RRes Y), OTP `123456` | `4100000000005000` | `5100000000005007` |
| Failed (ARes R) | `4100000000300005` | `5100000000300002` |

Swagger example merchant ID `000000000000051` and FPX test channel `TEST002` appear in samples; your real test merchant ID comes from Paydee.

## Not Documented (checked 2026-10-21)

Rate limits, IP allowlists for callbacks, callback retry policy, callback URL registration, required HTTP headers, settlement report API, payout API, tokenisation beyond recurring, and an FPX/e-wallet sandbox simulator. The sample code repo `https://github.com/fin-project/sample-mpi-py` linked from the site returns 404, and the hosted sample `https://mpi-sample-py.onrender.com` returned 503.
