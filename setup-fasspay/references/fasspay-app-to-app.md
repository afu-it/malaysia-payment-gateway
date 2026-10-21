# Fasstap App-to-App API (Android)

Request and response objects for handing a transaction to the Fasstap app. Per-field setter and getter blocks are omitted because they repeat the ThirdPartyIntegrationRequestVO and ThirdPartyIntegrationResponseVO tables.

Source: `https://developers.softspace.com.my/reference/fasstap/app-to-app-integration/android/latest/sdk-api` (checked 2026-10-21). Generated from the page data and condensed; descriptions over about 110 characters are cut with "...". Req: Y = required, C = conditional, blank = optional or not stated.

### Overview

#### Introduction

#### Action Type
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| Registration |  |  | Refers to Registration. |
| GetToken |  |  | Refers to Get Token. |
| Payment |  |  | Refers to Payment. |
| SalesHistory |  |  | Refers to Sales History. |
| GetTransactionStatus |  |  | Refers to Get Transaction Status. |
| VoidPayment |  |  | Refers to Void Payment. |
| CreditSettlement |  |  | Refers to Credit Settlement. |
| Refund |  |  | Refers to Refund. |
| RecurringPayment |  |  | Refers to Recurring Payment. |
| QRPayment |  |  | Refers to QR Payment. |
| QRRefund |  |  | Refers to QR Refund. |
| QRCheckStatus |  |  | Refers to QR Check Status. |
| InstallmentPayment |  |  | Refers to Installment Payment. |
| QuickChip |  |  | Refers to Quick Chip. |
| QuickChipCompletion |  |  | Refers to Quick Chip Completion. |
| Undefined |  |  | Fallbacks to Payment if not specified. |

#### Request
`requestMPOSIntegration`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Context | Y | Context of the caller class. |
| requestVO | ThirdPartyIntegrationRequestVO | Y | Value object class that holding payment information set from 3rd party application. |
| requestVO.actionType | ActionType |  | Enum value (ActionType) of actionType. |
| requestVO.mposUrl | string | Y | Fasstap mobile application package name. |
| requestVO.returnUrl | string | Y | 3rd party application package name. |
| requestVO.developerId | string | Y | 3rd party application developer ID. |
| requestVO.currency | string |  | currency code |
| requestVO.amount | string |  | amount Mandatory for payment. |
| requestVO.customerMobileNo | string |  | mobile number |
| requestVO.customerEmail | string |  | email address |
| requestVO.thirdPartyUserID | string | Y | 3rd party user ID |
| requestVO.returnIntentName | string | Y | 3rd party application intent name. |
| requestVO.referenceNo | string |  | 3rd party application reference number. Mandatory for payment. |
| requestVO.integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |
| requestVO.returnTimeout | number |  | return timeout in seconds. |
| requestVO.sourceApplication | String |  | 3rd party application source. |
| requestVO.authToken | String |  | 3rd party application auth token. |
| requestVO.transactionToken | String |  | 3rd party application transactionToken. |
| requestVO.statusUrl | String |  | 3rd party application transaction status url. |
| requestVO.valueSeparator | String |  | 3rd party application value separator. |
| requestVO.thirdPartyDescriptionVO | ThirdPartyDescriptionVO |  | Value object of class ThirdPartyDescriptionVO that holding description of payment transaction. |
| requestVO.thirdPartyDescriptionVO.title | string | Y | Title of the description. |
| requestVO.thirdPartyDescriptionVO.description | string | Y | Description value. |
| requestVO.mposAutoLogOut | Boolean |  | Flag to determine whether the MPOS application should logout after each transaction. |
| requestVO.https | Boolean |  | flag to indicate whether the request is from HTTPS. |
| requestVO.refundReferenceNo | String |  | 3rd party application refund reference number. |
| requestVO.thirdPartyRecurringPaymentVO | ThirdPartyRecurringPaymentVO |  | Value object of class ThirdPartyRecurringPaymentVO that holding recurring detail. |
| requestVO.thirdPartyRecurringPaymentVO.referenceId | string |  | Recurring payment reference id. |
| requestVO.thirdPartyRecurringPaymentVO.frequency | frequencyEnum |  | Recurring payment frequency. |
| requestVO.thirdPartyRecurringPaymentVO.duration | string |  | Recurring payment duration. |
| requestVO.transactionUniqueID | String |  | 3rd party application transaction ID. |
| requestVO.transactionReaderSN | String |  | 3rd party application transactions reader serial number. |
| requestVO.thirdPartyVoidPayment | Boolean |  | Flag to indicate whether the 3rd party application allowed to be voided. |
| requestVO.qrID | String |  | 3rd party application QR transaction. |
| requestVO.qrType | QRType |  | Enum value (QRType) of QR type. optional for QR Payment Method, only applies to Merchant Present QR |
| requestVO.nfcType | NFCType |  | Enum value (NFCType) of NFC type. |
| requestVO.merchantData | String |  | 3rd party application merchant data. |
| requestVO.thirdPartyBundleID | String |  | 3rd party application's bundle identification number. |
| requestVO.thirdPartySignature | String |  | 3rd party application's signature. |
| requestVO.timeout | Long |  | return timeout in seconds. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | Enum value (ActionType) of actionType. |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| currency | string |  | currency code |
| amount | string |  | amount Mandatory for payment. |
| customerMobileNo | string |  | mobile number |
| customerEmail | string |  | email address |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string |  | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |
| returnTimeout | number |  | return timeout in seconds. |
| sourceApplication | String |  | 3rd party application source. |
| authToken | String |  | 3rd party application auth token. |
| transactionToken | String |  | 3rd party application transactionToken. |
| statusUrl | String |  | 3rd party application transaction status url. |
| valueSeparator | String |  | 3rd party application value separator. |
| thirdPartyDescriptionVO | ThirdPartyDescriptionVO |  | Value object of class ThirdPartyDescriptionVO that holding description of payment transaction. |
| thirdPartyDescriptionVO.title | string | Y | Title of the description. |
| thirdPartyDescriptionVO.description | string | Y | Description value. |
| mposAutoLogOut | Boolean |  | Flag to determine whether the MPOS application should logout after each transaction. |
| https | Boolean |  | flag to indicate whether the request is from HTTPS. |
| refundReferenceNo | String |  | 3rd party application refund reference number. |
| thirdPartyRecurringPaymentVO | ThirdPartyRecurringPaymentVO |  | Value object of class ThirdPartyRecurringPaymentVO that holding recurring detail. |
| thirdPartyRecurringPaymentVO.referenceId | string |  | Recurring payment reference id. |
| thirdPartyRecurringPaymentVO.frequency | frequencyEnum |  | Recurring payment frequency. |
| thirdPartyRecurringPaymentVO.duration | string |  | Recurring payment duration. |
| transactionUniqueID | String |  | 3rd party application transaction ID. |
| transactionReaderSN | String |  | 3rd party application transactions reader serial number. |
| thirdPartyVoidPayment | Boolean |  | Flag to indicate whether the 3rd party application allowed to be voided. |
| qrID | String |  | 3rd party application QR transaction. |
| qrType | QRType |  | Enum value (QRType) of QR type. optional for QR Payment Method, only applies to Merchant Present QR |
| nfcType | NFCType |  | Enum value (NFCType) of NFC type. |
| merchantData | String |  | 3rd party application merchant data. |
| thirdPartyBundleID | String |  | 3rd party application's bundle identification number. |
| thirdPartySignature | String |  | 3rd party application's signature. |
| timeout | Long |  | return timeout in seconds. |

#### Response
`processResponseMPOSIntegration`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| active3rdPartyAppActivity | Activity | Y | Current active activity running on 3rd party application when it's to process response back from Fasstap mo... |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| ThirdPartyIntegrationResponseVO |  |  | TODO |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| approvalCode | string |  | approval code of payment transaction, given if payment transaction is approved and success. |
| transactionID | string |  | Identifier number of payment transaction. |
| traceNo | string |  | Trace number of payment transaction. |
| cardNo | string |  | 16-digit credit/debit card number that used to perform payment transaction. Only show first 6 last 4 digits... |
| cardType | string |  | Type of credit/debit card that used to perform payment transaction. |
| cardHolderName | string |  | Holder name of credit/debit card that used to perform payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |
| thirdPartyRecurringPaymentVO | ThirdPartyRecurringPaymentVO |  | Recurring payment reference id, duration and frequency. Example: <br referenceId = thirdPartyRecurringPayme... |
| thirdPartyRecurringPaymentVO.referenceId | string |  | Recurring payment reference id. |
| thirdPartyRecurringPaymentVO.frequency | frequencyEnum |  | Recurring payment frequency. |
| thirdPartyRecurringPaymentVO.duration | string |  | Recurring payment duration. |
| currency | String |  | Currency code of payment transaction. |
| errorType | String |  | Type of error that occured during the operation. |
| amount | String |  | Amount of payment transaction. |
| thirdPartyUrl | String |  | Endpoint url of the 3rd party service handling the transaction. |
| thirdPartyAppIntentName | String |  | Intent name that is used to communicate with the 3rd party application. |
| transactionRequestID | String |  | Unique identifier for the original transaction request. |
| transactionDetailID | String |  | Unique identification numver for the transaction information. |
| applicationLabel | String |  | Name of the application. |
| uuid | String |  | Unique identification number for the transaction. |
| transactionUniqueID | String |  | Unique identification number for the transaction. |
| transactionReaderSN | String |  | Serial numver of the card reader or terminal use for the transaction. |
| customerField | String |  | General field to store custom metadata needed. |
| refundReferenceNo | String |  | Reference number of the refunded transaction. |
| integrationType | IntegrationType |  | Identification number of the QR transaction. |
| qrID | string |  | Original QR code ID. |
| rppID | String |  | Recurring payment plan transaction identification number. |
| rppOriID | String |  | Original recurring payment plan transaction identification number. |
| qrType | string |  | Type of QR transaction. |
| nfcType | NFCType |  | Type of NFC. |
| merchantData | String |  | Metadata to merchant. |
| encryptedPan | String |  | Encrypted primary account number for secure transmission. |
| expiryDate | String |  | Expiry date information of the card. |
| balanceAmount | String |  | Remaining balance for partial payment or prepaid method. |
| timeout | Long |  | Maximum time to wait before the operation timeout. |
| thirdPartyEppPackageVO | ThirdPartyEppPackageVO |  | Nested object containing information about the easy payment plan. |
| thirdPartyEppPackageVO.eppPackageID | String |  | Unique identifier for the easy payment plan. |
| thirdPartyEppPackageVO.eppPackageName | String |  | Name for the installment package. |
| thirdPartyEppPackageVO.finalPaymentAmount | String |  | Total amount to be paid overtime of the installment plan |
| transactionStatus | String |  |  |

### Core Modules

#### Get Token
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeGetToken` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Registration
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeRegistration` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Payment
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypePayment` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| amount | string | C | amount Mandatory for payment. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| approvalCode | string |  | approval code of payment transaction, given if payment transaction is approved and success. |
| transactionID | string |  | Identifier number of payment transaction. |
| cardNo | string |  | 16-digit credit/debit card number that used to perform payment transaction. Only show first 6 last 4 digits... |
| cardType | string |  | Type of credit/debit card that used to perform payment transaction. |
| cardHolderName | string |  | Holder name of credit/debit card that used to perform payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Void Payment
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeVoidPayment` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Refund
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeRefund` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| amount | string | C | amount Mandatory for payment. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Installment Payment
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeInstallmentPayment` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| amount | string | C | amount Mandatory for payment. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| approvalCode | string |  | approval code of payment transaction, given if payment transaction is approved and success. |
| transactionID | string |  | Identifier number of payment transaction. |
| cardNo | string |  | 16-digit credit/debit card number that used to perform payment transaction. Only show first 6 last 4 digits... |
| cardType | string |  | Type of credit/debit card that used to perform payment transaction. |
| cardHolderName | string |  | Holder name of credit/debit card that used to perform payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Recurring Payment
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeRecurringPayment` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| amount | string | C | amount Mandatory for payment. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| approvalCode | string |  | approval code of payment transaction, given if payment transaction is approved and success. |
| transactionID | string |  | Identifier number of payment transaction. |
| cardNo | string |  | 16-digit credit/debit card number that used to perform payment transaction. Only show first 6 last 4 digits... |
| cardType | string |  | Type of credit/debit card that used to perform payment transaction. |
| cardHolderName | string |  | Holder name of credit/debit card that used to perform payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Quick Chip
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeQuickChip` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| amount | string | C | amount Mandatory for payment. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| approvalCode | string |  | approval code of payment transaction, given if payment transaction is approved and success. |
| transactionID | string |  | Identifier number of payment transaction. |
| cardNo | string |  | 16-digit credit/debit card number that used to perform payment transaction. Only show first 6 last 4 digits... |
| cardType | string |  | Type of credit/debit card that used to perform payment transaction. |
| cardHolderName | string |  | Holder name of credit/debit card that used to perform payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Quick Chip Completion
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeQuickChipCompletion` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| amount | string | C | amount Mandatory for payment. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| approvalCode | string |  | approval code of payment transaction, given if payment transaction is approved and success. |
| transactionID | string |  | Identifier number of payment transaction. |
| cardNo | string |  | 16-digit credit/debit card number that used to perform payment transaction. Only show first 6 last 4 digits... |
| cardType | string |  | Type of credit/debit card that used to perform payment transaction. |
| cardHolderName | string |  | Holder name of credit/debit card that used to perform payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Get Transaction Status
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeGetTransactionStatus` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |

#### QR Payment
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeQRPayment` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| amount | string | C | amount Mandatory for payment. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |
| qrType | QRType | C | Enum value (QRType) of QR type. optional for QR Payment Method, only applies to Merchant Present QR |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| actionType | ActionType |  | Type of 3rd party action. |
| qrID | string |  | Original QR code ID. |
| qrType | string |  | Type of QR transaction. |

#### QR Refund
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeQRRefund` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |

#### QR Check Status
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeQRCheckStatus` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| referenceNo | string | C | 3rd party application reference number. Mandatory for payment. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| referenceNo | string |  | Unique reference number for the integration. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Sales History
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeSalesHistory` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| actionType | ActionType |  | Type of 3rd party action. |

#### Credit Settlement
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| actionType | ActionType |  | `ActionType.ActionTypeCreditSettlement` |
| mposUrl | string | Y | Fasstap mobile application package name. |
| returnUrl | string | Y | 3rd party application package name. |
| developerId | string | Y | 3rd party application developer ID. |
| thirdPartyUserID | string | Y | 3rd party user ID |
| returnIntentName | string | Y | 3rd party application intent name. |
| integrationType | IntegrationType | Y | Enum value (IntegrationType) of integrationType. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | string |  | Status code of payment transaction. |
| statusMessage | string |  | Status message of payment transaction. |
| actionType | ActionType |  | Type of 3rd party action. |


