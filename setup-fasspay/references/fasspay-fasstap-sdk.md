# Fasstap SDK API (Android)

Field tables for the Fasstap tap-on-phone SDK. Overview, setup and codes summary are in `fasspay-in-person.md`.

Source: `https://developers.softspace.com.my/reference/fasstap/sdk-integration/android/latest/sdk-api` (checked 2026-10-21). Generated from the page data and condensed; descriptions over about 110 characters are cut with "...". Req: Y = required, C = conditional, blank = optional or not stated.

#### SSMPOSSDK
`init`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Context |  | Context of the caller class. |
| configuration | SSMPOSSDKConfiguration |  | SSMPOSSDKConfiguration object that contains all the configuration information to configure the Fasstap™ MPO... |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| SSMPOSSDK |  |  | Singleton object that represents Fasstap™ MPOS SDK. |

`getInstance`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| SSMPOSSDK |  |  | Singleton object that represents Fasstap™ MPOS SDK. |

`getTransaction`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Transaction |  |  | Transaction object that's used to handle transaction related operations. |

`hasRequiredPermission`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Context |  | Context of the caller class. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| boolean |  |  | Flag indicates if the app has granted with the necessary permissions. |

`requestPermissionIfRequired`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| app | Activity |  | The activity that is requesting for the permission so Android framework could handle the request prompt. Th... |
| code | int |  | The request code that Android's Framework should be forwarding back to main project's `onRequestPermissionR... |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| boolean |  |  | Flag indicates if the permissions have all been granted prior to calling this method. If it is true, Fassta... |

`hasGrantPermission`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| grantResult | int[] |  | The grant results forwarded from `onRequestPermissionsResults`. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| boolean |  |  | Flag indicates if the permissions have all been granted. |

`getCotsId`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| String |  |  | Device ID. |

`getSdkVersion`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| String |  |  | SOFT SPACE Fasstap™ MPOS SDK version. |

`getMPoCSdkVersion`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| String |  |  | SOFT SPACE MPoC SDK version. |

#### SSMPOSSDKConfiguration.Builder
`create`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| SSMPOSSDKConfiguration.Builder |  |  | A new `SSMPOSSDKConfiguration.Builder` that could be used to set configuration values. |

`setConfigurationToken`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| configurationToken | String |  | The configuration token given by SOFT SPACE. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| SSMPOSSDKConfiguration.Builder |  |  | A new `SSMPOSSDKConfiguration.Builder` that could be used to set configuration values. |

`setMPoCActivationToken`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| mPoCActivationToken | String |  | The MPoC activation token to initialize SS MPoC SDK. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| SSMPOSSDKConfiguration.Builder |  |  | A new `SSMPOSSDKConfiguration.Builder` that could be used to set configuration values. |

`setRefreshMPoCToken`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| refreshMPoCToken | Boolean |  | Flag if SDK should refresh the MPoC token. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| SSMPOSSDKConfiguration.Builder |  |  | A new `SSMPOSSDKConfiguration.Builder` that could be used to set configuration values. |

`setOfflineAck`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| offlineAck | Boolean |  | Flag if SDK should acknowledge the offline transactions. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| SSMPOSSDKConfiguration.Builder |  |  | A new `SSMPOSSDKConfiguration.Builder` that could be used to set configuration values. |

`setUniqueID`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| uniqueID | String |  | The uniqueID from SOFT SPACE. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| SSMPOSSDKConfiguration.Builder |  |  | A new `SSMPOSSDKConfiguration.Builder` that could be used to set configuration values. |

`setDeveloperID`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| developerID | String |  | The developerID from SOFT SPACE. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| SSMPOSSDKConfiguration.Builder |  |  | A new `SSMPOSSDKConfiguration.Builder` that could be used to set configuration values. |

#### Transaction
`startTransaction`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that starts the transaction. Please note that Activity class instance is required here instead... |
| trxParams | MPOSTransactionParams |  | The parameters for this transaction like amount. For more information, please refer to MPOSTransactionParams. |
| callback | TransactionEvents |  | The callback implementation class for various transactional and card events. Please refer to TransactionEve... |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the transaction has started successfully. Please take note that this flag does... |

`abortTransaction`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that indicates if the transaction has cancelled successfully. |

`startQRTransaction`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that starts the transaction. Please note that Activity class instance is required here instead... |
| trxParams | MPOSTransactionParams |  | The parameters for this transaction like amount & QR Type. For more information, please refer to MPOSTransa... |
| callback | TransactionEvents |  | The callback implementation class for various transactional and card events. Please refer to TransactionEve... |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the transaction has started successfully. Please take note that this flag does... |

`abortQRTransaction`
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that indicates if the QR transaction has cancelled successfully. |

`voidTransaction`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that voids the transaction. Please note that activity is required here instead of plain context. |
| trxParams | MPOSTransactionParams |  | The parameters for this void transaction like transaction ID. |
| callback | TransactionEvents |  | The callback implementation class for various transactional and card events. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the void transaction has started successfully. Please take note that this flag... |

`refundTransaction`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that refunds the transaction. Please note that activity is required here instead of plain cont... |
| trxParams | MPOSTransactionParams |  | The parameters for this refund transaction like transaction ID or `isCardRequired`. |
| callback | TransactionEvents |  | The callback implementation class for various transactional and card events. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the refund transaction has started successfully. Please take note that this fl... |

`performSettlement`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that performs the settlement. Please note that activity is required here instead of plain cont... |
| trxParams | MPOSTransactionParams |  | The parameters object. |
| callback | TransactionEvents |  | The callback implementation class for various transactional events. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the settlement has started successfully. Please take note that this flag does ... |

`refreshToken`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that initiate the refresh token operation. Please note that activity is required here instead ... |
| callback | TransactionEvents |  | The callback implementation class for various transactional events. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the refresh token operation has started successfully. Please take note that th... |

`sendReceipt`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that performs send receipt. Please note that activity is required here instead of plain context. |
| trxParams | MPOSTransactionParams |  | The parameters for send receipt like email, mobile number. |
| callback | TransactionEvents |  | The callback implementation class for various transactional events. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the send receipt has started successfully. Please take note that this flag doe... |

`uploadSignature`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| trxParams | MPOSTransactionParams |  | The parameter for upload signature is signature. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the upload signature has started successfully. Please take note that this flag... |

`getTransactionStatus`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that gets the transaction status. Please note that activity is required here instead of plain ... |
| trxParams | MPOSTransactionParams |  | The parameters for this get transaction status like transaction ID. |
| callback | TransactionEvents |  | The callback implementation class for various transactional and card events. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the get transaction status has started successfully. Please take note that thi... |

`startOfflineTransaction`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that starts the offline transaction. Please note that Activity class instance is required here... |
| trxParams | MPOSTransactionParams |  | The parameters for this transaction like amount. For more information, please refer to MPOSTransactionParams. |
| callback | TransactionEvents |  | The callback implementation class for various transactional and card events. Please refer to TransactionEve... |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag indicating whether the transaction has successfully started. Please take note that this flag does *n... |

`getOfflineTransaction`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that get the offline transactions. Please note that activity is required here instead of plain... |
| trxParams | MPOSTransactionParams |  | No specific parameters is required. |
| callback | TransactionEvents |  | The callback implementation class for various transactional and card events. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the get offline transaction has started successfully. Please take note that th... |

`submitOfflineTransaction`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| context | Activity |  | The activity that submit the offline transactions. Please note that activity is required here instead of pl... |
| trxParams | MPOSTransactionParams |  | No specific parameters is required. |
| callback | TransactionEvents |  | The callback implementation class for various transactional and card events. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| Boolean |  |  | A flag that is indicating if the submit offline transaction has started successfully. Please take note that... |

`onTransactionResult`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| result | int |  | Result of processing transaction returned by the transaction-processing kernel. For more information about ... |
| transactionOutcome | TransactionOutcome | Y | The transaction outcome object that carries status and details. |
| transactionOutcome.statusCode | String | Y | Status code of the transaction. |
| transactionOutcome.statusMessage | String | Y | Status message of the transaction. |
| transactionOutcome.approvalCode | String | Y | Approval code from Host. |
| transactionOutcome.transactionID | String | Y | Transaction ID from MPOS server, to be used for SendReceipt or Void. |
| transactionOutcome.cardNo | String | Y | Masked PAN number. |
| transactionOutcome.cardType | CardType | Y |  |
| transactionOutcome.cardHolderName | String |  | Card holder name from EMV data. |
| transactionOutcome.aid | String | Y | AID from EMV data. |
| transactionOutcome.transactionDate | String | Y | Transaction Date of the transaction. |
| transactionOutcome.transactionTime | String | Y | Transaction Time of the transaction. |
| transactionOutcome.acquirerID | String |  | Acquirer ID of the transaction. |
| transactionOutcome.applicationCryptogram | String |  | Application Cryptogram of the transaction. |
| transactionOutcome.terminalVerificationResults | String |  | TVR of the transaction. |
| transactionOutcome.transactionStatusInfo | String |  | TSI of the transaction. |
| transactionOutcome.merchantIdentifier | String |  | MID of the transaction. |
| transactionOutcome.terminalIdentifier | String |  | TID of the transaction. |
| transactionOutcome.invoiceNo | String |  | Invoice No of the transaction. |
| transactionOutcome.contactlessCVMType | String |  | Contactless CVM Type of the transaction. |
| transactionOutcome.rrefNo | String |  | RRN of the transaction. |
| transactionOutcome.transactionCert | String |  | TC of the transaction. |
| transactionOutcome.merchantCategoryCode | String |  | MCC of the transaction. |
| transactionOutcome.receiptUrl | String |  | URL for e-receipt of the transaction. |
| transactionOutcome.transactionDateTime | String |  | Full date time of the transaction. |
| transactionOutcome.referenceNo | String |  | Unique reference no of the transaction. |
| transactionOutcome.traceNo | String |  | Trace No of the transaction. |
| transactionOutcome.transactionReference | String |  | Long transaction ID to be used for portal reference. |
| transactionOutcome.isSignRequired | Boolean |  | To determine if the transaction requires signature. |
| transactionOutcome.clientTransactionID | String |  | Offline transaction ID. |
| transactionOutcome.batchID | String |  | Offline batch ID after offline transaction batch has been submitted. |
| transactionOutcome.rid | String |  | RID of a card. |
| transactionOutcome.maskedPAN | String |  | Masked card PAN. |
| transactionOutcome.offlineTransactionList | List<TransactionData |  | Offline transaction listing. |
| transactionOutcome.submitOfflineTransactionList | List<TransactionData |  | Submitted offline transaction listing. |
| transactionOutcome.customField | HashMap |  | Custom field for the transaction. |
| transactionOutcome.supportedCardSchemes | List<CardType |  | List of supported card schemes. |
| transactionOutcome.qrID | String | Y | ID of the QR transaction |
| transactionOutcome.qrData | String | Y | QR content that used to generate QR code. |
| transactionOutcome.qrCode | Bitmap | Y | QR code image of the requested QR type in Bitmap form. |
`onTransactionUIEvent`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| uiEvent | int |  | The event code that raised the UI event. Please refer to TransactionUIEvent for more information. |
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | String | Y | Status code of the transaction. |
| statusMessage | String | Y | Status message of the transaction. |
| approvalCode | String | Y | Approval code from Host. |
| transactionID | String | Y | Transaction ID from MPOS server, to be used for SendReceipt or Void. |
| cardNo | String | Y | Masked PAN number. |
| cardType | CardType | Y |  |
| cardHolderName | String |  | Card holder name from EMV data. |
| aid | String | Y | AID from EMV data. |
| transactionDate | String | Y | Transaction Date of the transaction. |
| transactionTime | String | Y | Transaction Time of the transaction. |
| acquirerID | String |  | Acquirer ID of the transaction. |
| applicationCryptogram | String |  | Application Cryptogram of the transaction. |
| terminalVerificationResults | String |  | TVR of the transaction. |
| transactionStatusInfo | String |  | TSI of the transaction. |
| merchantIdentifier | String |  | MID of the transaction. |
| terminalIdentifier | String |  | TID of the transaction. |
| invoiceNo | String |  | Invoice No of the transaction. |
| contactlessCVMType | String |  | Contactless CVM Type of the transaction. |
| rrefNo | String |  | RRN of the transaction. |
| transactionCert | String |  | TC of the transaction. |
| merchantCategoryCode | String |  | MCC of the transaction. |
| receiptUrl | String |  | URL for e-receipt of the transaction. |
| transactionDateTime | String |  | Full date time of the transaction. |
| referenceNo | String |  | Unique reference no of the transaction. |
| traceNo | String |  | Trace No of the transaction. |
| transactionReference | String |  | Long transaction ID to be used for portal reference. |
| isSignRequired | Boolean |  | To determine if the transaction requires signature. |
| clientTransactionID | String |  | Offline transaction ID. |
| batchID | String |  | Offline batch ID after offline transaction batch has been submitted. |
| rid | String |  | RID of a card. |
| maskedPAN | String |  | Masked card PAN. |
| offlineTransactionList | List<TransactionData |  | Offline transaction listing. |
| submitOfflineTransactionList | List<TransactionData |  | Submitted offline transaction listing. |
| customField | HashMap |  | Custom field for the transaction. |
| supportedCardSchemes | List<CardType |  | List of supported card schemes. |
| qrID | String | Y | ID of the QR transaction |
| qrData | String | Y | QR content that used to generate QR code. |
| qrCode | Bitmap | Y | QR code image of the requested QR type in Bitmap form. |

#### MPOSTransactionParams.Builder
`setCustomField`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| customField | HashMap<String, String |  | The self-defined custom field to be sent to server. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| MPOSTransactionParams.Builder |  |  | The current object of the builder class of `MPOSTransactionParams`. |

`setAmount`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| amount | String |  | The transaction amount. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| MPOSTransactionParams.Builder |  |  | The current object of the builder class of `MPOSTransactionParams`. |

`setReferenceNumber`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| referenceNumber | String |  | The transaction reference number. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| MPOSTransactionParams.Builder |  |  | The current object of the builder class of `MPOSTransactionParams`. |

`setEnableFeedback`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| enableFeedback | boolean |  | The flag to enable feedback configuration in the SDK. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| MPOSTransactionParams.Builder |  |  | The current object of the builder class of `MPOSTransactionParams`. |

`setCardRequired`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| cardRequired | boolean |  | The refund indicator. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| MPOSTransactionParams.Builder |  |  | The current object of the builder class of `MPOSTransactionParams`. |

`setMPOSTransactionID`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| mposTransactionID | String |  | The transaction ID from MPOS server. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| MPOSTransactionParams.Builder |  |  | The current object of the builder class of `MPOSTransactionParams`. |

`setQRType`
Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| qrType | QRType |  | The QR type. |
Returns:
| Field | Type | Req | Notes |
|---|---|---|---|
| MPOSTransactionParams.Builder |  |  | The current object of the builder class of `MPOSTransactionParams`. |

Params:
| Field | Type | Req | Notes |
|---|---|---|---|
| statusCode | String | Y | Status code of the transaction. |
| statusMessage | String | Y | Status message of the transaction. |
| approvalCode | String | Y | Approval code from Host. |
| transactionID | String | Y | Transaction ID from MPOS server, to be used for SendReceipt or Void. |
| cardNo | String | Y | Masked PAN number. |
| cardType | CardType | Y |  |
| cardHolderName | String |  | Card holder name from EMV data. |
| aid | String | Y | AID from EMV data. |
| transactionDate | String | Y | Transaction Date of the transaction. |
| transactionTime | String | Y | Transaction Time of the transaction. |
| acquirerID | String |  | Acquirer ID of the transaction. |
| applicationCryptogram | String |  | Application Cryptogram of the transaction. |
| terminalVerificationResults | String |  | TVR of the transaction. |
| transactionStatusInfo | String |  | TSI of the transaction. |
| merchantIdentifier | String |  | MID of the transaction. |
| terminalIdentifier | String |  | TID of the transaction. |
| invoiceNo | String |  | Invoice No of the transaction. |
| contactlessCVMType | String |  | Contactless CVM Type of the transaction. |
| rrefNo | String |  | RRN of the transaction. |
| transactionCert | String |  | TC of the transaction. |
| merchantCategoryCode | String |  | MCC of the transaction. |
| receiptUrl | String |  | URL for e-receipt of the transaction. |
| transactionDateTime | String |  | Full date time of the transaction. |
| referenceNo | String |  | Unique reference no of the transaction. |
| traceNo | String |  | Trace No of the transaction. |
| transactionReference | String |  | Long transaction ID to be used for portal reference. |
| isSignRequired | Boolean |  | To determine if the transaction requires signature. |
| clientTransactionID | String |  | Offline transaction ID. |
| batchID | String |  | Offline batch ID after offline transaction batch has been submitted. |
| rid | String |  | RID of a card. |
| maskedPAN | String |  | Masked card PAN. |
| offlineTransactionList | List<TransactionData |  | Offline transaction listing. |
| submitOfflineTransactionList | List<TransactionData |  | Submitted offline transaction listing. |
| customField | HashMap |  | Custom field for the transaction. |
| supportedCardSchemes | List<CardType |  | List of supported card schemes. |
| qrID | String | Y | ID of the QR transaction |
| qrData | String | Y | QR content that used to generate QR code. |
| qrCode | Bitmap | Y | QR code image of the requested QR type in Bitmap form. |

