# Smart Terminal SDK API (Android)

SoftSpaceSDK, ICodecManager, IReader, ReaderTag keys and request objects for the USB Smart Terminal.

Source: `https://developers.softspace.com.my/reference/smart-terminal/android/latest/sdk-api` (checked 2026-10-21). Generated from the page data and condensed; descriptions over about 110 characters are cut with "...". Req: Y = required, C = conditional, blank = optional or not stated.

### Overview

#### Introduction

#### SoftSpaceSDK

`usbReader`

#### ICodecManager
`encodeRequest`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| request | RequestObject |  | A request Object containing details required for the specific action. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| ByteArray |  |  | A `ByteArray` format of request data that the terminal can understand |

`decodeResponse`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| response | String |  | A `String` containing the raw response data. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Map<ReaderTag, String> |  |  | A read-only map where each entry: Key: ReaderTag - acts as a label to indicate what kind of data is stored ... |

`encodeRequestAsString`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| request | RequestObject |  | A request Object containing details required for the specific action. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| String |  |  | A request in `String` format, suitable for transmission over text-based channels. |

`formatResponseAsString`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| response | ByteArray |  | The raw response data in ByteArray form, typically the encoded output received after a transaction or request. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| String |  |  | A `String` that represents the content of the ByteArray. |

`verifyResponse`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| response | String |  | A `String` that contains the encoded response data to be verified. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| VerificationResult<String, VerificationError> |  |  | A wrapper class, indicating the outcome of the validation: On success: Returns the verified response as a s... |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| REF_NO |  |  | Unique reference number assigned to the transaction by the POS. Used for tracking and reconciliation. |
| DEVELOPER_ID |  |  | ID assigned to the developer integrating the SDK. Helps in identifying custom implementations. |
| AMOUNT |  |  | Total transaction amount, including any additional charges, displayed in the relevant currency format. |
| DESCRIPTION |  |  | Description or purpose of the transaction. |
| VOID_REF_ID |  |  | Reference ID for void transactions. |
| APPROVAL_CODE |  |  | Bank approval code for the transaction. |
| CARD_TYPE |  |  | Type of card used in the transaction. |
| APPLICATION_LABEL |  |  | Label of the card application used. |
| CARD_NO |  |  | Masked credit card number. |
| CARD_HOLDER_NAME |  |  | Name of the card holder. |
| TRACE_NO |  |  | Trace number assigned by the bank. |
| TC |  |  | Transaction Certificate. |
| AID |  |  | Application Identifier for the card. |
| CVM_TYPE |  |  | Cardholder Verification Method type. |
| QR_TYPE |  |  | Type of QR code used in the transaction. |
| TRANSACTION_DATE |  |  | Date when the transaction was made. |
| TRANSACTION_TIME |  |  | Time when the transaction was made. |
| TID |  |  | Terminal ID. |
| MID |  |  | Merchant ID. |
| BATCH_NO |  |  | Batch number for the transaction. |
| INVOICE_NO |  |  | Invoice number for the transaction. |
| RREF_NO |  |  | Retrieval Reference Number by Bank. |
| TSI |  |  | Transaction Status Information. |
| TVR |  |  | Terminal Verification Result. |
| DISCOUNT_AMOUNT |  |  | Amount of discount applied to the transaction. |
| ENT_TYPE |  |  | Transaction type. |
| ACTION_CODE |  |  | Action code for the transaction. |
| RESPONSE_CODE |  |  | Response code from the transaction. |
| PAYMENT_TYPE |  |  | Type of payment used. |
| TRANSACTION_STATUS |  |  | Status of the transaction. |
| PRODUCT_TYPE |  |  | Type of product being purchased. |
| AMOUNT_AUTHORIZED |  |  | Amount authorized by the host. |
| PHONE_NUMBER |  |  | Phone number associated with the transaction. |
| EMAIL |  |  | Email address associated with the transaction. |
| TOTAL_TRX_AMOUNT |  |  | Total transaction amount to settle. |
| TOTAL_TRX_COUNT |  |  | Total number of transactions to settle. |
| PRIVATE_FIELD |  |  | Private field for custom data. |
| VOID_TYPE |  |  | Type of void transaction. |
| NETWORK_SELECTION |  |  | Network selection for the transaction. |
| ENTRY_TYPE |  |  | Entry type for the transaction. |
| TRANSACTION_ID |  |  | Unique identifier for the transaction. |
| SALES_TRX_COUNT |  |  | Number of sales transactions. |
| SALES_TRX_AMOUNT |  |  | Total amount of sales transactions. |
| REFUND_TRX_COUNT |  |  | Number of refund transactions. |
| REFUND_TRX_AMOUNT |  |  | Total amount of refund transactions. |
| BIN_LIST |  |  | List of Bank Identification Numbers. |
| E_RECEIPT_REQUEST_ID |  |  | ID for e-receipt request. |
| ORIGINAL_TRANSACTION_ID |  |  | ID of the original transaction. |
| VAT_AMOUNT |  |  | Value Added Tax amount. |
| TAX_ALLOWANCE_AMOUNT |  |  | Tax allowance amount. |
| MERCHANT_UNIQUE_VALUE |  |  | Unique value for the merchant. |
| CAMPAIGN_TYPE |  |  | Type of campaign associated with the transaction. |
| MERCHANT_TAX_ID |  |  | Tax ID of the merchant. |
| ENCRYPTED_PAN |  |  | Encrypted Primary Account Number. |
| EXPIRY_DATE |  |  | Expiry date of the card. |
| BALANCE_AMOUNT |  |  | Balance amount after transaction. |
| QR_SCAN_MODE |  |  | Mode in which the QR code was scanned. |
| IPP_TENURE |  |  | Installment Payment Plan tenure. |
| TOPUP_AMOUNT |  |  | Amount for top-up transaction. |
| TOPUP_PREVIOUS_BALANCE |  |  | Previous balance before top-up. |
| TOPUP_NEW_BALANCE |  |  | New balance after top-up. |
| MEMBER_EXPIRY_DATE |  |  | Expiry date of membership. |
| CASH_OUT_AMOUNT |  |  | Amount withdrawn as cash. |
| TENDER_CODE |  |  | Code indicating the payment tender type. |
| ORIGINAL_AMOUNT |  |  | Original transaction amount before adjustments. |
| PIN_SERIAL_NO |  |  | Serial number for PIN. |
| RELOAD_PIN |  |  | PIN for reload. |
| PIN_RELOAD_TELCO |  |  | Telecommunications operator for PIN reload. |
| PIN_EXPIRY_DATE |  |  | Expiry date of PIN. |
| RECEIPT_TYPE |  |  | Type of receipt. |

#### IReader
`connect`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Flow<ReaderResult<Unit, ConnectionError>> |  |  | A result indicating whether the connection was successful or if it failed |

`sendRequest`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| request | ByteArray |  | Takes a request in the form of a ByteArray. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Flow<ReaderResult<Unit, RequestError>> |  |  | A result indicating whether the request was sent successfully or if it failed during transmission |

`receiveResponse`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Flow<ReaderResult<String, ResponseError>> |  |  | A response as a string if successful, or an error if it fails to get a response. |

`observeConnection`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Flow<ConnectionState> |  |  | Returns a result indicating the current connection status |

`clear`

### Core Modules

#### Card Payment
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A0 - Sale Request |
| referenceNo | String | Y | Unique reference number from 3rd party No duplicates allowed |
| amount | String | Y | Amount in cent unit Example: RM100.50 = 10050 |
| paymentType | String | Y |  |
| privateField | String | C | Refer to Sale Request |
| originalTransactionID | String | C | Original sales transaction ID Only mandatory for `paymentType` = `03` (Sales Completion) |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| ippTenure | String | C | IPP Tenure (months). Only mandatory for `paymentType` = `01` (Installment) |
| developerID | String |  | Unique developer ID assigned by the bank. |
| description | String |  | Description from 3rd party. Suggest to send in the same value as `referenceNo`. |
| productType | String |  |  |
| binList | String |  | BINs are the first 6 digits of a credit card number. Each entry is 7 characters including the comma `,`. Yo... |
| cashOutAmount | String |  | Amount in cent unit. Example: RM100.50 = 10050 |
| vatAmount | String |  | Vat amount for vat rebate for debit card Only applicable if VAT is enabled |
| taxAllowanceAmount | String |  | Amount for tax allowance Only applicable if VAT is enabled |
| merchantUniqueValue | String |  | Merchant unique value: reference information generated from POS for checking back information Only applicab... |
| campaignType | String |  | Campaign type setting by Ministry of Finance Only applicable if VAT is enabled |
| merchantTaxID | String |  | Tax Identification number for Merchant Only applicable if VAT is enabled |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A1: Sale Response |
| responseCode | String | Y |  |
| referenceNo | String | Y | Unique reference number from 3rd party |
| voidRefID | String | Y | Void Ref ID from MPOS. Please keep this ID for `VOID` purpose. |
| transactionID | String |  | Transaction ID from MPOS. Can be used to search for transaction from portal. |
| eReceiptRequestID | String | Y | Request Transaction ID from MPOS. Can be used to send e-Receipt |
| approvalCode | String | Y | Approval Code by bank |
| cardType | String | Y |  |
| applicationLabel | String | Y | Credit Card Application Label |
| cardNo | String | Y | Masked Credit Card number |
| cardHolderName | String | Y | Credit Card holder name |
| traceNo | String | Y | Trace Number by bank |
| transactionCert | String | Y | Transaction Certificate |
| aid | String | Y | Credit card Application ID |
| cvmType | String | Y |  |
| transactionDate | String | Y | Date of transaction made |
| transactionTime | String | Y | Time of transaction made |
| TID | String | Y | Terminal ID |
| MID | String | Y | Merchant ID |
| batchNo | String | Y | Batch Number |
| invoiceNo | String | Y | Unique Invoice number from MPOS |
| rRefNo | String | Y | Retrieval Reference Number by Bank |
| tsi | String | Y | Transaction Status Information |
| tvr | String | Y | Terminal Verification Result |
| entType | String | Y | Transaction Type |
| amountAuthorized | String | Y | Amount Authorized by Host |
| ippTenure | String |  | IPP Tenure (months). Only applicable for `paymentType` = `01` |
| privateField | String (var up to 99) | C | Please Refer to Sale Response |
| tenderCode | String |  | Tender Code |
| originalAmount | String |  | Original Amount Only applicable for Instant Rebate |
| discountAmount | String |  | Discount Amount Only applicable for Instant Rebate |
| cardEntryType | String |  | Only exists if Response Code = 304 (Authorizing) |

#### QR Payment
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A5: QR Payment Request |
| referenceNo | String | Y | Unique reference number from 3rd party No duplicates allowed. |
| amount | String | Y | Amount in cent unit. Example: RM100.50 = 10050 |
| qrType | String | Y | Type of QR transaction performed |
| qrScanMode | String | C | Camera Facing Mode |
| privateField | String | C | Please refer to QR Payment Request |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique developer ID assigned by the bank |
| description | String |  | Description from 3rd party. Suggest to send in the same value as `referenceNo`. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A6: QR Payment Response |
| responseCode | String | Y |  |
| referenceNo | String | Y | Unique Reference number from 3rd party |
| transactionID | String | Y | Transaction ID from MPOS |
| qrType | String | Y |  |
| privateField | String | C | Please Refer to QR Payment Response |
| eReceiptRequestID | string | Y | Request Transaction ID from MPOS. Can be used to send e-Receipt |
| cardNo | String | Y | Masked Credit Card Number |
| traceNo | String | Y | Trace Number by bank |
| transactionDate | String | Y | Date of transaction made |
| transactionTime | String | Y | Time of transaction made |
| TID | String | Y | Terminal ID |
| MID | String | Y | Merchant ID |
| batchNo | String | Y | Batch Number |
| invoiceNo | String | Y | Unique Invoice number from MPOS |
| rRefNo | String | Y | Retrieval Reference Number by Bank |
| entType | String | Y | Transaction Type |
| amountAuthorized | String | Y | Amount Authorized by Host |
| applicationLabel | String |  | Credit Card Application Label |
| tenderCode | String |  | Tender Code |
| merchantCode | String |  | Merchant Code |

#### QR Inquiry
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A15: Inquiry QR Payment Request |
| referenceNo | String | Y | Unique reference number from 3rd party No duplicates allowed. |
| qrType | String | Y |  |
| privateField | String | C | Please Refer to QR Inquiry Payment Request |
| transactionID | String | Y | Transaction ID from MPOS |
| rRefNo | String |  | Retrieval Reference Number by Host |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique developer ID assigned by the bank |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A16: Inquiry QR Payment Response |
| responseCode | String | Y |  |
| referenceNo | String | Y | Unique reference number from 3rd party |
| transactionID | String | Y | Transaction ID from MPOS |
| eReceiptRequestID | String | Y | Request Transaction ID from MPOS. Can be used to send e-Receipt |
| qrType | String | Y |  |
| cardNo | String | Y | Masked Credit Card Number |
| traceNo | String | Y | Trace Number by bank |
| transactionDate | String | Y | Date of transaction made |
| transactionTime | String | Y | Time of transaction made |
| TID | String | Y | Terminal ID |
| MID | String | Y | Merchant ID |
| batchNo | String | Y | Batch Number |
| invoiceNo | String | Y | Unique Invoice number from MPOS |
| rRefNo | String | Y | Retrieval Reference Number by Bank |
| entType | String | Y | Transaction Type |
| amountAuthorized | String | Y | Amount Authorized by Host |
| privateField | String | C | Please Refer to QR Inquiry Payment Response |
| applicationLabel | String |  | Credit Card Application Label |
| tenderCode | String |  | Tender Code |
| merchantCode | String |  | Merchant Code |

#### Void
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A2: Void Request |
| voidRefId | String | Y | Void Ref ID from MPOS |
| voidType | String | Y | Void Type |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique developer ID assigned by the bank. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A3: Void Response |
| responseCode | String | Y |  |
| referenceNo | String |  | Unique reference number from 3rd party |
| tenderCode | String |  | Tender Code |

#### Cancel
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A4: Cancel Request |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A4: Cancel Response |
| responseCode | String | Y |  |

#### Transaction Status
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A9: Transaction Status Request |
| referenceNo | String | Y | Unique reference number from 3rd party No duplicates allowed |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique developer ID assigned by bank |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A10: Transaction Status Response |
| responseCode | String | Y |  |
| referenceNo | String | Y | Unique reference number from 3rd party |
| voidRefID | String | Y | Void Ref ID from MPOS. Please keep this ID for `VOID` purpose. |
| transactionID | String |  | Transaction ID from MPOS. |
| approvalCode | String | Y | Approval code by bank |
| cardType | String | Y |  |
| applicationLabel | String | Y | Credit Card Application Label |
| cardNo | String | Y | Masked Credit Card number |
| cardHolderName | String | Y | Credit Card holder name |
| traceNo | String | Y | Trace Number by bank |
| transactionDate | String | Y | Date of transaction made |
| transactionTime | String | Y | Time of transaction made |
| TID | String | Y | Terminal ID |
| MID | String | Y | Merchant ID |
| batchNo | String | Y | Batch Number |
| invoiceNo | String | Y | Unique Invoice number from MPOS |
| rRefNo | String | Y | Retrieval Reference Number by Bank |
| paymentType | String | Y |  |
| transactionStatus | String | Y |  |
| amountAuthorized | String | Y | Amount Authorized by host |
| privateField | String | C | Please refer to Transaction Status Response |

#### Settlement Info
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A11: Settlement Info Request |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique developer ID assigned by the bank. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A12: Settlement Info Response |
| responseCode | String | Y |  |
| totalTransactionAmount | String | Y | Total transaction amount to settle. `Total Transaction Amount` = `Sales Transaction Amount` - `Refund Trans... |
| totalTransactionCount | String | Y | Total number of transaction to settle. `Total Transaction Count` = `Sales Transaction Count` + `Refund Tran... |
| salesTransactionAmount | String | C | Total sales transaction amount. |
| salesTransactionCount | String | C | Number of sales transaction. |
| refundTransactionAmount | String | C | Total refund transaction amount. |
| refundTransactionCount | String | C | Number of refund transactions. |

#### Settlement
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A13: Settlement Request |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique developer ID assigned by the bank. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A14 - Settlement Response |
| responseCode | String | Y |  |

#### Send E-Receipt
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A34: Send e-Receipt Request |
| eReceiptRequestID | String | Y | E-Receipt Request ID from MPOS. Only applicable for receiptType = 00 (E-receipt) |
| receiptType | String | Y |  |
| phoneNumber | String | Y | 10 digit/11 digit phone number |
| email | String | Y | Valid email address E-Receipt can be sent using either customer Phone Number OR Email <br Settlement Report... |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique developer ID assigned by the bank. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A35: Send e-Receipt Response |
| responseCode | String | Y |  |

#### Membership Info
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A36: Membership Info Request |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A37: Membership Info Response |
| responseCode | String | Y |  |
| cardNo | String | Y | Masked Credit Card number |
| encryptedPan | String | Y | Card encrypted PAN |
| expiryDate | String | Y | Card expiry date |
| balanceAmount | String | Y | Card balance amount |

#### Top Up
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A48: Top Up Request |
| referenceNo | String | Y | Unique reference number from 3rd party. No duplicates allowed. |
| amount | String | Y | Amount in cent unit. Example: RM 100.50 = 10050 |
| privateField | String | C | Please refer to Top Up Request |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique developer ID assigned by the bank. |
| description | String |  | Description from 3rd party. Suggested to send the same value as `referenceNo`. |
| productType | String |  |  |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A49: Top Up Response |
| responseCode | String | Y | Refer to Response Codes |
| referenceNo | String | Y | Unique reference number from 3rd party |
| transactionID | String |  | Transaction ID from MPOS |
| eReceiptRequestID | String | Y | Request Transaction ID from MPOS. Can be used to search for transaction from portal. |
| approvalCode | String | Y | Approval Code by Bank |
| cardType | String | Y |  |
| applicationLabel | String | Y | Credit Card application label |
| cardNo | String | Y | Masked credit card number |
| cardHolderName | String | Y | Credit card holder name |
| traceNo | String | Y | Trace Number by bank |
| transactionCert | String | Y | Transaction Certificate |
| aid | String | Y | Credit card Application ID |
| cvmType | String | Y |  |
| transactionDate | String | Y | Date of transaction made |
| transactionTime | String | Y | Time of transaction made |
| TID | String | Y | Terminal ID |
| MID | String | Y | Merchant ID |
| batchNo | String | Y | Batch Number |
| invoiceNo | String | Y | Unique Invoice number from MPOS |
| rRefNo | String | Y | Retrieval Reference Number by Bank |
| tsi | String | Y | Transaction Status Information |
| tvr | String | Y | Terminal Verification Result |
| entType | String | Y | Transaction Type |
| cardEntryType | String |  | Only exist if `Response Code` is `304` (Authorizing) |
| topUpAmount | String | Y | Amount of Top Up |
| topUpPreviousBalance | String | Y | Balance before Top Up |
| topUpNewBalance | String | Y | Balance after Top Up |
| memberExpiry | String | Y | Member Expiry date |
| privateField | String | C | Please refer to Top Up Response |

#### Card Refund
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A22: Refund Request |
| referenceNo | String | Y | Valid Reference Number |
| amount | String | Y | Amount in cent unit. Example: RM100.50 = 10050 |
| approvalCode | String | Y | Approval Code by Bank |
| privateField | String | C | Please refer to Refund Request |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique Developer ID assigned by the bank. |
| description | String |  | Description from 3rd party. Suggest to send in the same value as `referenceNo`. |
| productType | String |  |  |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A23: Card Refund Response |
| responseCode | String | Y | Refer to Response Codes |
| referenceNo | String | Y | Unique reference number from 3rd party |
| transactionID | String |  | Transaction ID from MPOS |
| eReceiptRequestID | String | Y | Request Transaction ID from MPOS. Can be used to search for transaction from portal. |
| approvalCode | String | Y | Approval Code by Bank |
| cardType | String | Y | Applicable for Card transaction |
| applicationLabel | String | Y | Credit Card application label |
| cardNo | String | Y | Masked credit card number |
| cardHolderName | String | Y | Credit card holder name |
| traceNo | String | Y | Trace Number by bank |
| transactionCert | String | Y | Transaction Certificate |
| aid | String | Y | Credit card Application ID |
| cvmType | String | Y |  |
| transactionDate | String | Y | Date of transaction made |
| transactionTime | String | Y | Time of transaction made |
| TID | String | Y | Terminal ID |
| MID | String | Y | Merchant ID |
| batchNo | String | Y | Batch Number |
| invoiceNo | String | Y | Unique Invoice number from MPOS |
| rRefNo | String | Y | Retrieval Reference Number by Bank |
| tsi | String | Y | Transaction Status Information |
| tvr | String | Y | Terminal Verification Result |
| entType | String | Y | Transaction Type |
| amountAuthorized | String | Y | Amount Authorized by Host |
| cardEntryType | String |  | Only exists if `Response Code` = `304` (Authorizing) |
| privateField | String | C | Please refer to Refund Response |
| Tender Code | String |  | Tender Code |

#### QR Refund
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A24: QR Refund Request |
| referenceNo | String | Y |  |
| amount | String | Y | Amount in cent unit. Example: RM100.50 = 10050 |
| privateField | String | C | A24: QR Refund Request |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique developer ID assigned by the bank |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A25: QR Refund response |
| responseCode | String | Y | Refer to Response Codes |
| referenceNo | String | Y | Unique reference number from 3rd party |
| qrType | String | Y |  |
| traceNo | String | Y | Trace Number by bank |
| transactionDate | String | Y | Date of transaction made |
| transactionTime | String | Y | Time of transaction made |
| TID | String | Y | Terminal ID |
| MID | String | Y | Merchant ID |
| batchNo | String | Y | Batch Number |
| invoiceNo | String | Y | Unique Invoice number from MPOS |
| rRefNo | String | Y | Retrieval Reference Number by Bank |
| amountAuthorized | String | Y | Amount Authorized by Host |
| privateField | String | C | Please refer to QR Refund Response |
| tenderCode | String |  | Tender Code |
| merchantCode | String |  | Merchant Code |

#### Bill Payment
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A50: Bill Payment Request |
| referenceNo | String | Y | Valid Reference Number |
| paymentType | String | Y |  |
| privateField | String | C | Please refer to Bill Payment Request |
| networkSelection | String | C | Specifies whether the terminal or host handles the network connection for the transaction. If unspecified, ... |
| developerID | String |  | Unique Developer ID assigned by the bank. |
| description | String |  | Description from 3rd party. Suggest to send in the same value as `referenceNo`. |
| productType | Strins |  |  |
| binList | String |  | BINs are the first 6 digits of a credit card number. Each entry is 7 characters including the comma `,`. Yo... |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionCode | String |  | A51: Bill Payment response |
| responseCode | String | Y | Refer to Response Codes |
| referenceNo | String | Y | Unique reference number from 3rd party |
| transactionID | String | Y | Transaction ID from MPOS. Can be used to search for transaction from portal. |
| eReceiptRequestID | String | Y | Request Transaction ID from MPOS. Can be used to search for transaction from portal. |
| approvalCode | String | Y | Approval Code by Bank |
| cardType | String | Y | Applicable for Card transaction |
| applicationLabel | String | Y | Credit Card application label |
| cardNo | String | Y | Masked credit card number |
| cardHolderName | String | Y | Credit card holder name |
| traceNo | String | Y | Trace Number by bank |
| transactionCert | String | Y | Transaction Certificate |
| aid | String | Y | Credit card Application ID |
| cvmType | String | Y |  |
| transactionDate | String | Y | Date of transaction made |
| transactionTime | String | Y | Time of transaction made |
| TID | String | Y | Terminal ID |
| MID | String | Y | Merchant ID |
| batchNo | String | Y | Batch Number |
| invoiceNo | String | Y | Unique Invoice number from MPOS |
| rRefNo | String | Y | Retrieval Reference Number by Bank |
| tsi | String | Y | Transaction Status Information |
| tvr | String | Y | Terminal Verification Result |
| entType | String | Y | Transaction Type |
| amountAuthorized | String | Y | Amount Authorized by Host |
| cardEntryType | String |  | Only exists if `Response Code` = `304` (Authorizing) |
| billProvider | String | Y | Bill Payment Service Provider |
| billProductCategory | String | Y | Type of Biller Category |
| billProductName | String | Y | Biller product name |
| billAccountNo | String | Y | Customer Account No (can be mobile number also) |
| privateField | String | C | Please refer to Bill Payment Response |
| pinSerialNo | String | C | PIN Serial Number |
| pinReload | String | C | PIN Reload Number |
| pinReloadTelco | String | C | Telco Name |
| pinExpiryDate | String | C | PIN Expiry Date |
| tenderCode | String |  | Tender Code |


