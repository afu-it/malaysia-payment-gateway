# PayNet DuitNow QR Reference

DuitNow QR is Malaysia's national QR standard (DuitNow National Quick Response Code Standard, Merchant-Presented Mode, V1.5, based on EMV QR Code Specification for Payment Systems). This file covers building and parsing a merchant-presented DuitNow QR string, and the acquirer's RPP QR webhooks. A merchant gets its QR ID and Acquirer ID from its acquirer; PayNet does not issue QR codes to merchants directly. Checked 2026-10-21.

Sources: `/docs/duitnow-qr/merchant-presented-qr/qr-generation-specification/merchant-presented-mode`, `/docs/duitnow-qr/merchant-presented-qr/qr-data-object`, `/docs/duitnow-qr/merchant-presented-qr/examples`, `/docs/duitnow-qr/merchant-presented-qr/appendix-a`, `/docs/duitnow-qr/merchant-presented-qr/merchant-presented-mode-domestic-qr`, `/docs/duitnow-qr/merchant-presented-qr/cross-border-inbound-qr`, `/docs/duitnow-qr/merchant-presented-qr/cross-border-outbound-qr`, `/docs/duitnow-qr/consumer-presented-qr/*`, and the QR OpenAPI specs (all on `https://docs.paynet.my`).

## String format

- Each data object is `ID` (2 digits) + `Length` (2 digits, decimal count of characters) + `Value`.
- Payload Format Indicator (ID `00`) must be first; CRC (ID `63`) must be last.
- Formats: `N` numeric, `ANS` alphanumeric special (Common Character Set, ISO/IEC 8859 common part, Appendix A), `S` string (any character set, used in templates `62` and `64`).
- Processing of the QR by the payer app and the resulting network messages are out of scope of the QR spec.

## Root data objects

| Name | ID | Format | Length | Presence | Rules |
|---|---|---|---|---|---|
| Payload Format Indicator | `00` | N | 02 | M | `02` |
| Point of Initiation Method | `01` | N | 02 | M | `11` static, `12` dynamic |
| Merchant Account Information | `26` (`27` RFU) | ANS | up to 99 | M | Template below |
| Merchant Category Code | `52` | N | 04 | M | ISO 18245; `0000` for P2P QR; `6010` for POS cash-out QR |
| Transaction Currency | `53` | N | 03 | M | `458` (MYR) |
| Transaction Amount | `54` | ANS (digits and `.`) | up to 13 | C | Omit to let the payer enter the amount |
| Tip or Convenience Indicator | `55` | N | 02 | O | `01` prompt for tip, `02` flat fee (needs `56`), `03` percentage fee (needs `57`) |
| Convenience Fee Fixed | `56` | ANS | up to 13 | C | Only when `55` = `02` |
| Convenience Fee Percentage | `57` | ANS | up to 05 | C | Only when `55` = `03`; `0.01` to `99.99`; `0.06` means 0.06% of the amount |
| Country Code | `58` | ANS | 02 | M | `MY` |
| Merchant Name | `59` | ANS | up to 25 | M | |
| Merchant City | `60` | ANS | up to 15 | M | `MY` if not applicable |
| Postal Code | `61` | ANS | 05 | O | 5 digits |
| Additional Data Field Template | `62` | S | up to 99 | C | Required for JomPAY QR; each sub-tag up to 30 chars |
| Merchant Information, Language Template | `64` | S | up to 99 | O | Sub-tags `00` language (ISO 639, M), `01` alternate merchant name (up to 25, M), `02` alternate city (up to 15, O) |
| CRC | `63` | ANS | 04 | M | See CRC |
| Data Integrity Check | `82` | ANS | 99 | O | Hash (SHA-256 per ISO/IEC 10118-3) for integrity |

Amount rules (`54` and `56`): not zero; digits `0`-`9` and at most one `.`; the `.` may be present with no decimals; decimals should match the ISO 4217 exponent. Valid: `98.73`, `98`, `98.`, `0.1`, `0.01`. Invalid: `98,73`, `3 705`, `3,705`.

## Merchant Account Information (ID `26`)

| Name | Sub-ID | Format | Length | Presence | Notes |
|---|---|---|---|---|---|
| Application Identifier | `00` | ANS | 14 | M | `A0000006150001` (Malaysia AID) |
| Acquirer ID | `01` | ANS | up to 06 | M | From the Acquirer ID table below |
| QR ID | `02` | AN | up to 28 | M | Merchant or recipient ID assigned by the acquirer; keep it stable across dynamic QRs; for JomPAY QR use the biller code |
| Merchant Descriptor | `03` | ANS | up to 20 | O | Assigned by the acquirer |
| Mobile Number | `04` | ANS | up to 15 | O | Merchant mobile number |

## Additional Data Field Template (ID `62`)

| Name | Sub-ID | Format | Length | Notes |
|---|---|---|---|---|
| Bill Number | `01` | ANS | up to 25 | |
| Mobile Number | `02` | ANS | up to 25 | |
| Store Label | `03` | ANS | up to 25 | |
| Loyalty Number | `04` | ANS | up to 25 | |
| Reference Label | `05` | ANS | up to 25 | Merchant or acquirer transaction reference |
| Customer Label | `06` | ANS | up to 25 | |
| Terminal Label | `07` | ANS | up to 25 | |
| Purpose of Transaction | `08` | ANS | up to 25 | |
| Additional Consumer Data Request | `09` | A | up to 03 | Any of `A` address, `M` mobile, `E` email, each once (e.g. `0903AME`) |
| Merchant Tax ID | `10` | ANS | up to 15 | TIN |
| Merchant Channel | `11` | ANS | up to 3 | 3 characters: media, location, presence |
| RRN | `90` | ANS | up to 20 | Ref-1; required for JomPAY QR |
| RRN2 | `91` | ANS | up to 30 | Ref-2; optional for JomPAY QR |
| Geo Coordinates | `92` | ANS | up to 35 | Merchant latitude and longitude |
| Payment System Specific | `50`-`89`, `92`-`99` | S | var | RFU |

If `62` is present it must contain at least one sub-tag. For sub-tags `01` to `08`, a value of `***` means the payer app must ask the payer (example `0603***` prompts for Customer Label).

Merchant Channel characters: first (media) `0` merchant sticker, `1` bill/invoice, `2` magazine/poster, `3` other print, `4` merchant POS/POI screen, `5` website, `6` app, `7` other screen; second (location) `0` at merchant premises, `1` not at premises, `2` remote commerce, `3` others; third (presence) `0` attended, `1` unattended, `2` semi-attended (self-checkout), `3` others.

## CRC (ID `63`)

ISO/IEC 13239 CRC-16 with polynomial `0x1021` and initial value `0xFFFF` (CRC-16/CCITT-FALSE), computed over every data object including IDs and lengths, in order, plus the CRC's own ID and length `6304`, but not its value. Encode the 2-byte result as 4 uppercase hex characters (`007B` becomes `6304007B`).

```python
def duitnow_crc(payload_without_crc_value: str) -> str:
    # payload must already end with "6304"
    crc = 0xFFFF
    for byte in payload_without_crc_value.encode("ascii"):
        crc ^= byte << 8
        for _ in range(8):
            crc = ((crc << 1) ^ 0x1021) & 0xFFFF if crc & 0x8000 else (crc << 1) & 0xFFFF
    return f"{crc:04X}"
```

This function reproduces the CRCs of the official examples below (checked 2026-10-21).

## Official examples (sandbox illustration values)

Example 1, retail POS static QR, Affin Bank acquirer, amount 10.00:

```text
00020201021126410014A000000615000101065016640209123456789520499995303458540510.005802MY5909QRCSDNBHD6005BANGI6304343F
```

Breakdown: `000202` version, `010211` static, `2641` + `0014A0000006150001` AID + `0106501664` Acquirer ID 501664 + `0209123456789` QR ID, `52049999` MCC, `5303458` MYR, `540510.00` amount, `5802MY`, `5909QRCSDNBHD`, `6005BANGI`, `6304343F` CRC.

Example 2, P2P (MCC `0000`): `...52040000...5909AUSERNAME6005BANGI63043A23`.

Example 3, POS with Data Integrity Check `8264f879ac6266d8d6da081fc1c17160021119747b190615c21076b3fc0fce56c6eb` before CRC `6304B623`. The docs do not say what input that hash covers.

Example 4, JomPAY bill payment (Acquirer ID `898989`, QR ID = biller code `1234`, MCC `1111`, no amount, postal code, `62` with RRN):

```text
00020201021126360014A00000061500010106898989020412345204111153034585802MY5903TNB6002KL610559200621790130124007552406823231d4db4506dc9852b1c21c74be148b8363045DFB
```

## Acquirer IDs (ID `26` sub-ID `01`)

| Acquirer ID | Participant | BIC |
|---|---|---|
| 501664 | Affin Bank Berhad | PHBMMYKL |
| 504374 | Alliance Bank Malaysia Berhad | MFBBMYKL |
| 432134 | Al Rajhi Banking & Investment Corporation (Malaysia) Berhad | RJHIMYKL |
| 564169 | Ambank Malaysia Berhad | ARBKMYKL |
| 603346 | Bank Islam Malaysia Berhad | BIMBMYKL |
| 589267 | Bank Kerjasama Rakyat Malaysia Berhad | BKRMMYKL |
| 564167 | Bank Muamalat Malaysia Berhad | BMMBMYKL |
| 629188 | Bank of America (M) Berhad | BOFAMY2X |
| 629152 | Bank of China (M) Berhad | BKCHMYKL |
| 589373 | Bank Pertanian Malaysia Berhad (Agrobank) | AGOBMYKL |
| 420709 | Bank Simpanan Nasional | BSNAMYK1 |
| 629196 | MUFG Bank (Malaysia) Berhad | BOTKMYKX |
| 629204 | BNP Paribas Malaysia Berhad | BNPAMYKL |
| 629261 | China Construction Bank (Malaysia) Berhad | PCBCMYKL |
| 501854 | CIMB Bank Berhad | CIBBMYKL |
| 589170 | Citibank Berhad | CITIMYKL |
| 629246 | Deutsche Bank (Malaysia) Berhad | DEUTMYKL |
| 588830 | Hong Leong Bank Berhad | HLBBMYKL |
| 589836 | HSBC Bank Malaysia Berhad | HBMBMYKL |
| 629253 | Industrial and Commercial Bank of China (M) Berhad | ICBKMYKL |
| 629212 | JP Morgan Chase Bank Berhad | CHASMYKX |
| 639406 | Kuwait Finance House (Malaysia) Berhad | KFHOMYKL |
| 588734 | Maybank Berhad | MBBEMYKL |
| 432310 | MBSB Bank Berhad | AFBQMYKL |
| 629220 | Mizuho Bank (Malaysia) Berhad | MHCBMYKA |
| 504324 | OCBC Bank Berhad | OCBCMYKL |
| 564162 | Public Bank Berhad | PBBEMYKL |
| 564160 | RHB Bank Berhad | RHBBMYKL |
| 539981 | Standard Chartered Bank Malaysia Berhad | SCBLMYKX |
| 629238 | Sumitomo Mitsui Banking Corporation (M) Berhad | SMBCMYKL |
| 519469 | United Overseas Bank Berhad | UOVBMYKL |
| 629295 | AEON Bank (M) Berhad | ACDBMYK2 |
| 629303 | Boost Bank Berhad | BOBEMYK2 |
| 629279 | GX Bank Berhad (GXBank) | GXSPMYKL |
| 629311 | KAF Digital Bank Berhad | KAFBMYK2 |
| 629287 | YTL Digital Bank Berhad (Ryt Bank) | SCCHMYKL |
| 898989 | JomPAY | n/a |
| 890335 | Bangkok Bank Berhad (onboarding in progress) | BKKBMYKL |
| 890012 | BigPay Malaysia Sdn Bhd | BGPYMYNB |
| 890061 | Axiata Digital eCode Sdn Bhd | BOSTMYNB |
| 890236 | Beez Fintech Sdn Bhd | BEEZMYNB |
| 890160 | Razorpay Curlec Sdn Bhd | CRLCMYNB |
| 890145 | Fass Payment Solutions Sdn Bhd | FSPYMYNB |
| 890020 | Fave Asia Technologies Sdn Bhd | FAVEMYNB |
| 890038 | Finexus Cards Sdn Bhd | FNXSMYNB |
| 890103 | GHL Cardpay Sdn Bhd | GHLCMYNB |
| 890046 | GPay Network (M) Sdn Bhd | GRABMYNB |
| 890079 | iPay88 (M) Sdn Bhd | IPAYMYNB |
| 890228 | Koperasi Co-opbank Pertama Malaysia Berhad | KCPMMYK1 |
| 890111 | Merchantrade Asia Sdn Bhd | MASBMYNB |
| 890210 | MobilityOne Sdn Bhd | MBLOMYNB |
| 890194 | Payex PLT | PAYXMYNB |
| 890087 | Razer Merchant Services Sdn Bhd | RZMSMYNB |
| 890095 | Revenue Solution Sdn Bhd | RSSBMYNB |
| 890129 | Setel Pay Sdn Bhd | SVSBMYNB |
| 890004 | ShopeePay Malaysia Sdn. Bhd. | ARPYMYNB |
| 890202 | SiliconNet Technologies Sdn Bhd | SPAYMYNB |
| 890053 | TNG Digital Sdn Bhd | TNGDMYNB |
| 890251 | UniPin (M) Sdn Bhd | UNPNMYNB |
| 890319 | Wannapay Sdn Bhd | WANNMYNB |
| 890285 | 2C2P System Sdn Bhd (onboarding in progress) | C2P2MYNB |
| 890293 | Ampersand Pay Sdn Bhd (onboarding in progress) | APSBMYNB |
| 890244 | Boost Connect Sdn Bhd (onboarding in progress) | BCNTMYNB |
| 890178 | Instapay Technologies Sdn. Bhd | INSTMYNB |
| 890301 | ManagePay Systems Sdn Bhd | MPAYMYNB |
| 890327 | MRuncit Commerce Sdn Bhd | MRCTMYNB |
| 890269 | Paydibs Sdn Bhd | PDBSMYNB |
| 890350 | Wise Payments Malaysia Sdn Bhd | WISEMYNB |
| 890368 | Cardbiz Payment Services Sdn Bhd | CBSBMYNB |
| 890376 | Gkash Sdn Bhd | GKSBMYNB |

## Merchant-presented QR payment flow (domestic)

From `/docs/duitnow-qr/merchant-presented-qr/merchant-presented-mode-domestic-qr`:

1. Payer scans the QR in the issuer app. The issuer parses AID, Acquirer ID, merchant name, QR ID and amount. If the AID is PayNet's and the merchant is on-us, the issuer does not route to RPP.
2. Off-us: issuer sends Account Enquiry (520; 521 cross-border) to RPP, which calls the acquirer webhook `POST /webhooks/v3/accounts/enquire-xc`.
3. The acquirer validates the QR string (hash validation, key-field comparison such as merchant account info, MCC, merchant name, amount, or whole-string comparison) and, for dynamic QR, the validity period. It returns merchant name, QR category and accepted sources of funds.
4. Issuer shows the merchant name, payer confirms, issuer debits and sends QR Payment (030 POS, 031 cross-border POS, 040 P2P). RPP calls the acquirer `POST /webhooks/v3/payments/transfer-xc`.
5. Acquirer credits the merchant and answers `ACSP`. U300 rejects a QR payment without a prior QR enquiry.

The acquirer is responsible for notifying the merchant. A merchant should rely on its acquirer's notification or API, not on the payer's screen.

Consumer-presented QR (CPM): the payer app shows a QR, the merchant scans it, and the acquirer sends a debit request (240 domestic) or pre-authorisation (880) then post-authorisation debit (250) to the issuer; cancellation is 872. The new-site CPM issuer spec publishes webhooks `/webhooks/v3/debits/authorize` and `/webhooks/v3/debits/initiate`; the old-site acquirer spec shows `POST /v3/payments/cpmqr`. The new CPM acquirer spec lists only `/v3/transactions/enquire` and is in "specification review".

Cross-border: outbound (Malaysian payers abroad) uses issuer APIs `/v3/accounts/enquire-xc-cross-border` and `/v3/payments/transfer-xc-cross-border`; inbound (foreign payers at Malaysian merchants) uses the same acquirer webhooks with the `qr.crossborder` object (FX rate up to 4 decimals, `fxRateReference`, `foreignCurrencyAmount`, `convertedAmount`).

## Acquirer QR webhook field tables

#### QR Account Enquiry webhook (520/521) request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see `paynet-rpp-v3.md`) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `debtor` | object | Y | Debtor information |
| `debtor.name` | string | Y | max 140 ; Name of debtor |
| `debtor.id` | string | Y | max 35 ; Debtor Identifier refers to the ID used to register the customer. For details, please refer to the list of identification types provided here. Debtor ID is masked, displaying only the last 4 characters. |
| `debtorAccount` | object | Y | Information on the debtor account |
| `debtorAccount.id` | string | Y | max 34 ; Debtor account number |
| `debtorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, DEFAULT ; Account type |
| `debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor's resident status |
| `debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `debtorAgent` | object | Y |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the Originating FI |
| `creditorAccount` | object | Y | Information on the creditor account. |
| `creditorAccount.id` | string | Y | max 34 ; The QR ID extracted from PayNet QR. Default to ‘99999999999’ if QR is not parse-able or Application ID does not belong to PayNet. |
| `creditorAccount.type` | string |  | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, DEFAULT ; Creditor account type. |
| `creditorAgent` | object | Y |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the Receiving FI. |
| `qr` | object | Y | This contains information about the QR Payment. |
| `qr.code` | string | Y | max 2000 ; QR Code string |

#### QR Account Enquiry response the acquirer returns (200)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see `paynet-rpp-v3.md`) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.originalBusinessMessageId` | string | Y | max 35 ; Business Message ID provided in the request. |
| `data` | object | Y |  |
| `data.qr` | object | Y | This contains information about the QR payment. |
| `data.qr.crossborder` | object |  | Details required only for a cross border QR transaction, can be omitted for a domestic QR transaction. |
| `data.qr.crossborder.fxRate` | number |  | The exchange rate from MYR 1.00 to the foreign currency, can have up to 4 decimals. |
| `data.qr.crossborder.qrType` | string |  | max 20 ; enum: STATIC, DYNAMIC ; Type of the QR Code |
| `data.qr.crossborder.foreignCurrencyAmount` | object |  |  |
| `data.qr.crossborder.foreignCurrencyAmount.currency` | string |  | max 3 ; Foreign currency code |
| `data.qr.crossborder.foreignCurrencyAmount.value` | number |  | Value of the transaction in foreign currency, can have up to 2 decimals. |
| `data.qr.crossborder.fxRateReference` | string |  | max 35 ; The FX rate reference number used by PayNet. |
| `data.qr.crossborder.convertedAmount` | object |  |  |
| `data.qr.crossborder.convertedAmount.currency` | string |  | max 3 ; Currency code of the transaction |
| `data.qr.crossborder.convertedAmount.value` | number |  | Value of the transaction in MYR, can have up to 2 decimals. For static QR, PayNet always return MYR 1.00. For dynamic QR, PayNet returns the amount in MYR, by converting the foreign currency amount using FX rate. |
| `data.qr.crossborder.merchantCategoryCode` | string |  | max 35 ; Merchant category code provided by the Acquirer. |
| `data.qr.category` | string | Y | max 20 ; enum: POINT_OF_SALES, PERSON_TO_PERSON ; Category of QR transaction |
| `data.qr.acceptedSourceOfFunds` | array | Y | Accepted sources of fund or line of credit ; items enum: CASA, CREDIT_CARD, WALLET |
| `data.qr.promoCode` | string |  | max 35 ; Promotional code for discount |
| `data.creditor` | object | Y |  |
| `data.creditor.name` | string | Y | max 140 ; Creditor name |
| `data.creditorAccount` | object | Y | Information on the creditor account |
| `data.creditorAccount.id` | string | Y | max 34 ; Creditor Account number |
| `data.creditorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, LOAN, DEFAULT ; Creditor Account type |
| `data.creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Creditor's resident status |
| `data.creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `data.creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `data.creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `data.creditorAccount.customerCategory` | string |  | max 20 ; enum: RET, COR, OTH ; Customer category of creditor account |
| `resp` | object | Y |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, ACSP, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |
| `resp.reason.name` | string | Y | max 120 ; Name of the corresponding response code |
| `resp.reason.code` | string | Y | max 20 ; Response code of the request |
| `resp.reason.description` | string | Y | max 2000 ; Description of the response code |
| `resp.reason.details` | string |  | max 2000 ; Additional information related to the status of the request |
| `resp.reason.additionalCode` | string |  | max 20 ; Additional response code for PayNet troubleshooting |

#### QR Payment webhook (030/031/040) request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.transactionId` | string | Y | Transaction Identifier. Use the businessMessageId of this request. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see `paynet-rpp-v3.md`) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.possibleDuplicate` | boolean |  | Indicates a possible duplicate of a request that was sent in an earlier attempt. To be set to TRUE when re-sending a request to PayNet. |
| `appHeader.copyDuplicate` | string |  | max 4 ; enum: CODU, COPY, EXPY ; OFI uses CODU (copy of a duplicate) during the first attempt of a request, and uses COPY (copy) when retrying the same request. RPP uses COPY to indicate a SAF (Store-and-Forward) and EXPY (expired retry) to indicate a retry reque... |
| `interbankSettlementAmount` | object | Y |  |
| `interbankSettlementAmount.value` | number | Y | Value of the transaction in MYR, supports up to 2 decimal places. Special handling for Cross Border transaction: For static Cross Border QR, value is calculated by converting the foreign currency amount using FX rate in Cross Bord... |
| `interbankSettlementAmount.currency` | string |  | max 3 ; Currency of the transaction. Default to MYR. |
| `qr` | object | Y |  |
| `qr.crossborder` | object |  | Details required only for a cross border QR transaction, can be omitted for a domestic QR transaction. |
| `qr.crossborder.fxRate` | number |  | The exchange rate from MYR 1.00 to the foreign currency, can have up to 4 decimals. This value can be taken from `data.qr.crossborder.fxRate` in QR Enquiry (521) response. |
| `qr.crossborder.qrType` | string |  | max 20 ; enum: STATIC, DYNAMIC ; Type of the QR Code This value can be taken from `data.qr.crossborder.qrType` in QR Enquiry (521) response. |
| `qr.crossborder.foreignCurrencyAmount` | object |  |  |
| `qr.crossborder.foreignCurrencyAmount.currency` | string |  | max 3 ; Foreign currency code. This value can be taken from `data.qr.crossborder.foreignCurrencyAmount.currency` in QR Enquiry (521) response. |
| `qr.crossborder.foreignCurrencyAmount.value` | number |  | Value of the transaction in foreign currency, can have up to 2 decimals. For static QR, this value is based on merchant input. For dynamic QR, this value can be taken from `data.qr.crossborder.foreignCurrencyAmount.value` in QR En... |
| `qr.crossborder.fxRateReference` | string |  | max 35 ; The FX rate reference number used by PayNet. This value can be found in `data.qr.crossborder.fxRateReference` in Cross Border QR Enquiry (521) response. |
| `qr.crossborder.convertedAmount` | object |  |  |
| `qr.crossborder.convertedAmount.currency` | string | Y | max 3 ; Currency code of the transaction |
| `qr.crossborder.convertedAmount.value` | number | Y | Value of the transaction in MYR, can have up to 2 decimals. This value must be the same as interbankSettlementAmount. |
| `qr.crossborder.merchantCategoryCode` | string |  | max 35 ; Merchant category code provided by the Acquirer. This value can be found in `data.qr.crossborder.merchantCategoryCode` in Cross Border QR Enquiry (521) response. |
| `qr.code` | string | Y | max 2000 ; QR Code string |
| `qr.category` | string | Y | max 20 ; enum: POINT_OF_SALES, PERSON_TO_PERSON ; Category of QR Transaction. This value can be taken from `data.qr.category` in QR Enquiry (520/521) response. |
| `qr.acceptedSourceOfFunds` | array | Y | Accepted sources of fund or line of credit. This value can be taken from `data.qr.acceptedSourceOfFunds[]` in QR Enquiry (520/521) response. ; items enum: CASA, CREDIT_CARD, WALLET |
| `qr.promoCode` | string |  | max 35 ; Promotional code for discount. This value can be taken from `data.qr.promoCode` in QR Enquiry (520/521) response. |
| `debtor` | object | Y | Debtor information |
| `debtor.name` | string | Y | max 140 ; Name of debtor |
| `debtor.id` | string | Y | max 35 ; Debtor Identifier refers to the ID used to register the customer. For details, please refer to the list of identification types provided here. Debtor ID is masked, displaying only the last 4 characters. |
| `debtorAccount` | object | Y | Debtor account information |
| `debtorAccount.id` | string | Y | max 34 ; Debtor account number |
| `debtorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, DEFAULT ; Debtor account type |
| `debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor's resident status |
| `debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `debtorAgent` | object | Y |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI. |
| `creditor` | object | Y | Information of the creditor, who is the recipient of funds. |
| `creditor.name` | string | Y | max 140 ; Name of the creditor returned by the Acquirer. This value can be found in `data.creditor.name` in QR Enquiry (520/521) response. |
| `creditorAccount` | object | Y | Information of the receiving account |
| `creditorAccount.id` | string | Y | max 34 ; The QR ID extracted from PayNet QR. Default to ‘99999999999’ if QR is not parse-able or Application ID does not belong to PayNet. |
| `creditorAgent` | object | Y |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the Receiving Financial Institution. |
| `recipientReference` | string | Y | max 140 ; Reference note for creditor. For allowable special character details, please refer to the list provided here |
| `paymentDescription` | string |  | max 140 ; Description for the transaction. |
| `settlementCycleNumber` | string | Y | max 35 ; Settlement cycle number set by RPP |
| `interbankSettlementDate` | string | Y | Settlement date of the transaction. Expressed in Malaysian local date (YYYY-MM-DD). |

#### QR Payment response the acquirer returns (200)

| Field | Type | Req | Notes |
|---|---|---|---|
| `data` | object | Y |  |
| `data.qr` | object |  |  |
| `data.qr.crossborder` | object |  | Details required only for a cross border QR transaction, can be omitted for a domestic QR transaction. |
| `data.qr.crossborder.fxRate` | number |  | The exchange rate from MYR 1.00 to the foreign currency, can have up to 4 decimals. |
| `data.qr.crossborder.qrType` | string |  | max 20 ; enum: STATIC, DYNAMIC ; Type of the QR Code |
| `data.qr.crossborder.foreignCurrencyAmount` | object |  |  |
| `data.qr.crossborder.foreignCurrencyAmount.currency` | string |  | max 3 ; Foreign currency code. |
| `data.qr.crossborder.foreignCurrencyAmount.value` | number |  | Value of the transaction in foreign currency, can have up to 2 decimals. |
| `data.qr.crossborder.fxRateReference` | string |  | max 35 ; The FX rate reference number used by PayNet. |
| `data.qr.crossborder.convertedAmount` | object |  |  |
| `data.qr.crossborder.convertedAmount.currency` | string | Y | max 3 ; Currency code of the transaction |
| `data.qr.crossborder.convertedAmount.value` | number | Y | Value of the transaction in MYR, can have up to 2 decimals. |
| `data.qr.crossborder.merchantCategoryCode` | string |  | max 35 ; Merchant category code provided by the Acquirer. |
| `data.qr.category` | string |  | max 20 ; enum: POINT_OF_SALES, PERSON_TO_PERSON ; Category of QR Transaction. This value can be taken from `data.qr.category` in QR Enquiry (520/521) response. |
| `data.qr.acceptedSourceOfFunds` | array |  | Accepted sources of fund or line of credit. This value can be taken from `data.qr.acceptedSourceOfFunds[]` in QR Enquiry (520/521) response. ; items enum: CASA, CREDIT_CARD, WALLET |
| `data.qr.promoCode` | string |  | max 35 ; Promotional code for discount. This value can be taken from `data.qr.promoCode` in QR Enquiry (520/521) response. |
| `data.settlementCycleNumber` | string |  | max 35 ; Settlement cycle number set by RPP |
| `data.interbankSettlementDate` | string | Y | Settlement date of the transaction. Expressed in Malaysian local date (YYYY-MM-DD). |
| `data.creditor` | object | Y | Information on the creditor |
| `data.creditor.name` | string | Y | max 140 ; Name of the creditor |
| `data.creditorAccount` | object | Y | Information on the receiving account |
| `data.creditorAccount.id` | string | Y | max 34 ; Creditor account number |
| `data.creditorAccount.type` | string |  | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, DEFAULT ; Creditor account type |
| `data.creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `data.creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `data.creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `data.creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `data.creditorAccount.customerCategory` | string |  | max 20 ; enum: RET, COR, OTH ; Customer category returned by the RFI |
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Message Identifier |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.originalBusinessMessageId` | string | Y | max 35 ; Message Identifier from the request. |
| `appHeader.transactionId` | string | Y | max 35 ; Transaction Identifier from the request. |
| `resp` | object | Y |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, ACSP, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |
| `resp.reason.name` | string | Y | max 120 ; Name of the corresponding response code |
| `resp.reason.code` | string | Y | max 20 ; Response code of the request |
| `resp.reason.description` | string | Y | max 2000 ; Description of the response code |
| `resp.reason.details` | string |  | max 2000 ; Additional information related to the status of the request |
| `resp.reason.additionalCode` | string |  | max 20 ; Additional response code for PayNet troubleshooting |


Signing, headers, error blocks and repeat handling for these webhooks are the same as in `paynet-rpp-v3.md`.
