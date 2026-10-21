# Antom Payment Operations

Inquiry, capture, cancel, and refund. Field tables generated from the official API reference pages on 2026-10-21; long descriptions are shortened (ending in `...`).

## Source URLs

- inquiryPayment: `https://docs.antom.com/ac/ams/paymentri_online.md`
- Payment status guide: `https://docs.antom.com/ac/cashierpay/payment_status_desc.md`
- capture (One-time Payments): `https://docs.antom.com/ac/ams/capture.md`
- Capture guide: `https://docs.antom.com/ac/cashierpay/capture.md`
- cancel: `https://docs.antom.com/ac/ams/paymentc_online.md`
- Cancel guide: `https://docs.antom.com/ac/cashierpay/cancel.md`
- refund: `https://docs.antom.com/ac/ams/refund_online.md`
- Refund guide: `https://docs.antom.com/ac/cashierpay/refund.md`
- inquiryRefund: `https://docs.antom.com/ac/ams/ir_online.md`

## inquiryPayment Rules

- Send `paymentRequestId` or `paymentId`. If both are sent, `paymentId` wins.
- `result` is only the call result. Read `paymentStatus`: `SUCCESS` and `FAIL` are final; `PROCESSING` means still in progress. Other values on the page: `CANCELLED`, `PENDING`.
- An unpaid payment closes after the default 14 minutes and becomes `FAIL`.
- `ORDER_NOT_EXIST` can mean the buyer never submitted. The error table says to retry after 15 seconds, three times.
- If inquiryPayment itself returns `U`, call again with the same parameters.
- Poll every 2 seconds until a final status or a notification arrives.
- For capture status, filter `transactions[]` where `transactionType` is `CAPTURE` (and, for manual capture, `transactionRequestId` equals your `captureRequestId`), then read `transactionResult.resultStatus`.

## Capture Rules

- Applies to cards, Apple Pay, and Google Pay. Default `captureMode` `AUTOMATIC` captures right after authorization and sends notifyCapture within seconds.
- `MANUAL`: call capture within 7 days of authorization, or Antom voids the order and releases the funds.
- Only full capture: `captureAmount` must equal `paymentAmount` and use the same currency.
- `captureRequestId` must be new per capture request. `S`: save `captureId` for refunds. `U`: query inquiryPayment or wait for notifyCapture; a retry with the same ID returns the final result. `F`: fix and retry; the same ID keeps returning failure.
- Do not send duplicate captures while `CAPTURE_IN_PROCESS`; that triggers an amount-exceeded error.
- If capture fails after authorization, cancel the authorization to release the buyer's funds.
- Ship only when the capture result is `S`.

## Cancel Rules

- Synchronous only. No cancel notification.
- Full cancellation only. Antom returns the commission.
- Card: within 7 days, and only before capture. Other methods: before 00:15 on T+1 (UTC+8).
- A refunded transaction cannot be cancelled (`PROCESS_FAIL`).
- Cancelling a `FAIL` payment returns `SUCCESS`, but the payment status stays `FAIL`.
- On `U`, retry with the same parameters. After three `U` results, contact Antom support.

## Refund Rules

- Full and multiple partial refunds are supported. Total refunds must not exceed the captured amount.
- Refund only after capture for card flows.
- Result is asynchronous: notifyRefund, or query inquiryRefund. A card refund never returns `S` synchronously.
- `U` with `REFUND_IN_PROCESS`: do not retry; query. `F`: retry with a new `refundRequestId`.
- The refund window depends on the method (FPX 180 days; TNG, Boost, GrabPay MY, ShopeePay MY 365 days). See `antom-codes.md`.
- Use the `paymentId` from notifyPayment. Card retries on Antom-hosted pages create a new `paymentId`.
- Refund by `paymentId` or by `captureId` consistently, or you get `REFUND_BY_TRANSACTION_TYPE_INCONSISTENT`.
- Refunds can also be issued in Antom Dashboard at Transactions > All transactions (single or batch refund).

## inquiryPayment

`POST /v1/payments/inquiryPayment`

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `paymentRequestId` | String | max 64 | The unique ID that is assigned by a merchant to identify a payment request. _paymentRequestId_ and _paymentId_ cannot both be null. If both _paymentRequestId_ and _paymen... |
| `paymentId` | String | max 64 | The unique ID that is assigned by Antom to identify a payment. _paymentRequestId_ and _paymentId_ cannot both be null. A one-to-one correspondence between _paymentId_ and... |
| `merchantAccountId` | String | max 32 | The unique ID to identify a merchant account. Note: Specify this parameter when you use a single client ID across multiple locations. |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | Information about the request calling result. Note: This field doesn't indicate the payment result. This field only indicates whether the [inquiryPayment](https... |
| `result.resultCode` | String, REQUIRED | max 64 | Result code. Possible values are listed in the Result/Error codes table on this page. |
| `result.resultStatus` | String, REQUIRED | values: S,F,U | Result status. Valid values are: - `S`: Indicates that this API is called successfully. - `F`: Indicates that this API call is failed. - `U`: Indicates that the calling s... |
| `result.resultMessage` | String, REQUIRED | max 256 | Result message that explains the result code. |
| `paymentStatus` | String | values: SUCCESS,FAIL,PROCESSING,CANCELLED,PENDING | Indicates the payment result. Valid values are: - `SUCCESS`: Indicates that the payment succeeds. - `FAIL`: Indicates that the payment fails. - `PROCESSING`: Indicates th... |
| `paymentResultCode` | String | max 64 | The result code for different payment statuses. Possible payment result codes are listed in the Payment result codes table on this page. Note: This field is returne... |
| `paymentResultMessage` | String | max 256 | The result message that explains the payment result code. Note: This field is returned when the API is called successfully (the value of _result.resultStatus_ is `S... |
| `paymentRequestId` | String | max 64 | The unique ID that is assigned by a merchant to identify a payment request. Note: This field is returned when the API is called successfully (the value of _result.r... |
| `paymentId` | String | max 64 | The unique ID that is assigned by Antom to identify a payment. Note: This field is returned when the API is called successfully (the value of _result.resultStatus_ ... |
| `paymentAmount` | Amount |  | The payment amount that the merchant requests to receive in the order currency. |
| `paymentAmount.currency` | String, REQUIRED | max 3 | The transaction currency, which is a 3-letter currency code that follows the ISO 4217 standard. (The transaction curre... |
| `paymentAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY ¥100, a 0-decimal currency). Notes: Fo... |
| `paymentCreateTime` | Datetime |  | The date and time when the payment is created. Note: This field is returned when the API is called successfully (the value of _result.resultStatus_ is `S`). - The value f... |
| `paymentTime` | Datetime |  | The date and time when the payment reaches a final state of success. Note: This field is returned only when the payment reaches a final state of success (the value of _... |
| `pspCustomerInfo` | PspCustomerInfo |  | The customer information of the digital wallets. Note: This field is returned when the digital wallets can provide the related information. |
| `pspCustomerInfo.pspName` | String | max 64 | The name of the digital wallets. Note: This field is returned when the digital wallets can provide the related information. |
| `pspCustomerInfo.pspCustomerId` | String | max 64 | The customer ID of the digital wallets. Note: This field is returned when the digital wallets can provide the related information. |
| `pspCustomerInfo.displayCustomerId` | String | max 64 | The customer ID used for display. For example, _loginId_. Note: This field is returned when the digital wallets can provide the related information. |
| `redirectActionForm` | RedirectActionForm |  | Information about the redirection action. Note: This field is returned when the value of _paymentResultCode_ is `PAYMENT_IN_PROCESS`. |
| `redirectActionForm.method` | String, REQUIRED | values: POST,GET | The HTTP method to be used when the merchant initiates a redirection to the redirection URL. Valid values are: - `POST`: Indicates that the request that is sent to the re... |
| `redirectActionForm.parameters` | String | max 2048 | Parameters required for the HTTP method in the key-value pair. Note: This field is returned only when the _method_ is POST. |
| `redirectActionForm.redirectUrl` | URL, REQUIRED | max 2048 | The URL where the user is redirected to. |
| `redirectActionForm.actionFormType` | String, REQUIRED | max 32 | The value is fixed as `RedirectActionForm`. |
| `acquirerReferenceNo` | String | max 64 | The unique ID assigned by the non-Antom acquirer for the transaction. |
| `transactions` | Array |  | Transaction details list. Note: This parameter is returned when a refund or a capture against the transaction exists. |
| `transactions.transactionResult` | Result, REQUIRED |  | The result of the subsequent action against the transaction. |
| `transactions.transactionResult.resultCode` | String, REQUIRED | max 64 | Result code. Possible values are listed in the Transaction result codes table on this page. |
| `transactions.transactionResult.resultStatus` | String, REQUIRED | values: S,F,U | Result status. Valid values are: - `S`: Indicates that the capture is successful. You can proceed with the order and save the Antom side _captureId_ information for futur... |
| `transactions.transactionResult.resultMessage` | String | max 256 | Result message |
| `transactions.transactionId` | String, REQUIRED | max 64 | The unique ID that is assigned by Antom to identify a transaction. - When the transaction type is `REFUND`, the value of this parameter is identical to _refundId_. - When... |
| `transactions.transactionType` | String, REQUIRED | values: REFUND,CAPTURE,INCREMENTAL | Transaction detail type. Valid values are: - `REFUND`: Indicates refund transaction details. - `CAPTURE`: Indicates capture transaction details. - `INCREMENTAL`: Indicate... |
| `transactions.transactionStatus` | String, REQUIRED | values: SUCCESS,FAIL,PROCESSING,CANCELLED | Transaction status. Valid values are: - `SUCCESS`:Transaction is successful. - `FAIL`: Transaction failed. - `PROCESSING`: Transaction is being processed. - `CANCELLED`:T... |
| `transactions.transactionAmount` | Amount, REQUIRED |  | The transaction amount. - When the transaction type is `REFUND`, the value of this parameter is identical to _refundAmount_. - When the transaction type is `CAPTURE`, the... |
| `transactions.transactionAmount.currency` | String, REQUIRED | max 3 | The currency used for the corresponding payment of the refund. The value is a 3-letter currency code that follows the [ISO 4217](https://www.iso.org/iso-4217-currency-cod... |
| `transactions.transactionAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY ¥100, a 0-decimal currency). Notes: Fo... |
| `transactions.transactionRequestId` | String, REQUIRED | max 64 | The unique ID that is assigned by the merchant to identify the transaction request. - When the transaction type is `REFUND`, the value of this parameter is identical to _... |
| `transactions.grossSettlementAmount` | Amount |  | The settlement amount of the capture order. Note: This parameter is returned when the capture is successful. |
| `transactions.grossSettlementAmount.currency` | String, REQUIRED | max 3 | A 3-letter currency code that follows the ISO 4217 standard. |
| `transactions.grossSettlementAmount.value` | Integer, REQUIRED | max 16 | The value of the amount as a positive integer in the smallest currency unit. For example, if the currency is USD and the amount is $1.00, set the value of this parameter ... |
| `transactions.settlementQuote` | Quote |  | The exchange rate between the settlement currency and transaction currency. Note: This parameter is returned when the capture is successful. |
| `transactions.settlementQuote.quoteId` | String | max 64 | The unique ID that is assigned by Antom to identify an exchange rate. |
| `transactions.settlementQuote.guaranteed` | Boolean |  | Indicates whether the exchange rate is locked in advance during payment. Valid values are: - `true`: The exchange rate is locked in advance during payment. - `false`: The... |
| `transactions.settlementQuote.quotePrice` | Decimal, REQUIRED |  | The exchange rate used when a currency conversion between settlement currency and transaction currency occurs. - Value range: 1 - unlimited |
| `transactions.settlementQuote.quoteCurrencyPair` | String, REQUIRED | max 16 | The currency pair corresponding to the exchange rate quote, indicating the exchange relationship between the settlement currency and the transaction currency. Both curren... |
| `transactions.settlementQuote.quoteStartTime` | Datetime |  | Effective time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27T... |
| `transactions.settlementQuote.quoteExpiryTime` | Datetime |  | Expiration time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27... |
| `customsDeclarationAmount` | Amount |  | The total amount for customs declaration. Note: This field is returned only when the payment succeeds and the payment method is AlipayCN. |
| `customsDeclarationAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `customsDeclarationAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY ¥100, a 0-decimal currency). Notes: Fo... |
| `grossSettlementAmount` | Amount |  | The value of this field equals to transaction amount multiplied by the value of _settlementQuote_. This field is returned when the currency exchange is predetermined and ... |
| `grossSettlementAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `grossSettlementAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a zero-decimal currency). Not... |
| `settlementQuote` | Quote |  | The exchange rate between the settlement currency and transaction currency. Note: This field is returned when _grossSettlementAmount_ is returned. |
| `settlementQuote.quoteId` | String | max 64 | The unique ID that is assigned by Antom to identify an exchange rate. |
| `settlementQuote.guaranteed` | Boolean |  | Indicates whether the exchange rate is locked in advance during payment. Valid values are: - `true`: The exchange rate is locked in advance during payment. - `false`: The... |
| `settlementQuote.quotePrice` | Decimal, REQUIRED |  | The exchange rate used when a currency conversion between settlement currency and transaction currency occurs. - Value range: 1 - unlimited |
| `settlementQuote.quoteCurrencyPair` | String, REQUIRED | max 16 | The currency pair corresponding to the exchange rate quote, indicating the exchange relationship between the settlement currency and the transaction currency. Both curren... |
| `settlementQuote.quoteStartTime` | Datetime |  | Effective time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27T... |
| `settlementQuote.quoteExpiryTime` | Datetime |  | Expiration time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27... |
| `paymentResultInfo` | PaymentResultInfo |  | The payment result information. Note: This parameter is returned when the value of _paymentMethodType_ in the [pay (One-time Payments)](https://docs.antom.com/a... |
| `paymentResultInfo.cardNo` | String | max 32 | The masked card number, which just shows part of the card number and can be used to display to the user. Note: This parameter is returned when the value of _payment... |
| `paymentResultInfo.cardBrand` | String | max 256 | The card brand, which can be used to display to the user. Note: This parameter is returned when the value of _paymentMethodType_ in the [pay (One-time Payments)... |
| `paymentResultInfo.cardToken` | String | max 128 | The token of the card, the value of this parameter is used by _paymentMethodId_ in the pay (One-time Payments) API... |
| `paymentResultInfo.issuingCountry` | String | max 2 | The issuing country of the card. The value of this parameter is a 2-letter country code that follows ISO 3166 Country Codes standard... |
| `paymentResultInfo.funding` | String | max 32; values: CREDIT,DEBIT,PREPAID,CHARGE,DEFERRED_DEBIT | The funding type of the card. Valid values are: - `CREDIT`: indicates a credit card. - `DEBIT`: indicates a debit card. - `PREPAID`: indicates a prepaid card. - `CHARGE`:... |
| `paymentResultInfo.paymentMethodRegion` | String | max 6 | The region code that represents the country or region of the payment method. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui/#sear... |
| `paymentResultInfo.threeDSResult` | ThreeDSResult |  | The result of 3D Secure authentication. Note: This parameter is returned when the value of _paymentMethodType_ in the [pay (One-time Payments)](https://docs.ant... |
| `paymentResultInfo.threeDSResult.threeDSVersion` | String, REQUIRED | max 16 | The version of 3D Secure protocol. Valid values are: - `1.0.2` - `2.1.0` - `2.2.0` |
| `paymentResultInfo.threeDSResult.eci` | String, REQUIRED | max 2; values: 02,01,00,04 | Electronic Commerce Indicator (ECI) that is returned by the card scheme. This parameter is used to indicate the type of cardholder identity authentication. Valid values a... |
| `paymentResultInfo.threeDSResult.cavv` | String | max 64 | The cardholder authentication value. The value of this parameter can be Cardholder Authentication Verification Value (CAVV) or Authentication Verification Value (AVV). ... |
| `paymentResultInfo.threeDSResult.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: This parameter is returned for 3D Secure 2.0. |
| `paymentResultInfo.threeDSResult.xid` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: This parameter is returned for 3D Secure 1.0. |
| `paymentResultInfo.threeDSResult.threeDSOffered` | Boolean |  | Indicates whether 3D Secure authentication was triggered for the payment. Valid values are: - `true`: indicates 3D Secure authentication was triggered for the payment. De... |
| `paymentResultInfo.threeDSResult.threeDStransactionStatusReason` | String | max 2 | When a transaction is unsuccessful due to 3D Secure authentication failure, you may receive the transaction status reason code returned by this parameter, indicating the ... |
| `paymentResultInfo.threeDSResult.challengeCancel` | String | max 2; values: 01,02,03,04,05,06,07,08 | Indicator informing the Access Control Server (ACS) and the Directory Server (DS) that the authentication has been cancelled. Possible values are: - `01`: Cardholder sele... |
| `paymentResultInfo.threeDSResult.exemptionType` | String | max 64 | The exemption type that Antom requested during 3DS authentication, specified through the 3DS requestor challenge indicator. Valid values are: - `dataOnly`: Transactions w... |
| `paymentResultInfo.threeDSResult.challenged` | Boolean, REQUIRED |  | Indicates whether a 3DS challenge was triggered. Valid values are: - `true`: The transaction has been challenged, and the user is required to complete the 3D Secure authe... |
| `paymentResultInfo.threeDSResult.paResStatus` | String | max 1; values: Y,N,U,A,C,R,D,I | Payer authentication response status. Indicates the result returned by the issuer or authentication system during the 3DS authentication flow. This parameter describes wh... |
| `paymentResultInfo.threeDSResult.liabilityShift` | Integer | values: 1,0 | Liability shift indicator. Indicates whether liability has been shifted for the transaction, typically in a 3DS authentication scenario. When the cardholder completes val... |
| `paymentResultInfo.avsResultRaw` | String | max 128 | The raw AVS result. See AVS result codes to check the valid values. Note: This parameter is returned when the... |
| `paymentResultInfo.cvvResultRaw` | String | max 128 | The raw Card Verification Value (CVV), Card Security Code (CSC), or Card Verification Code (CVC) result. See [CVV result codes](https://docs.antom.com/ac/ref/risk_methods... |
| `paymentResultInfo.networkTransactionId` | String | max 128 | The unique ID assigned by the card scheme to identify a transaction. The value of this parameter is used by the same parameter of [pay (One-time Payments)](https://do... |
| `paymentResultInfo.cardCategory` | String | max 32; values: CONSUMER,COMMERCIAL | The payment card category. Valid vaules are: - `CONSUMER`: the personal card - `COMMERCIAL`: the business card Note: This parameter is returned when all the followi... |
| `paymentResultInfo.creditPayPlan` | CreditPayPlan |  | The installment plan information for an installment payment. Note: This parameter is returned when the buyer successfully made a payment in installments using a car... |
| `paymentResultInfo.creditPayPlan.installmentNum` | Integer | max 8 | The number of installment payments. Note: This parameter is returned when the buyer successfully made a payment in installments using a card. |
| `paymentResultInfo.creditPayPlan.interval` | String | max 16 | The interval of each installment payment. The valid value is `MONTH`. Note: This parameter is returned when the buyer successfully made a payment in installments us... |
| `paymentResultInfo.creditPayPlan.interestRate` | String | max 16 | The installment interest rate, expressed as a percentage. For example, _interestRate_ = 0.1 means the installment interest rate is 0.1%. Note: This parameter is ret... |
| `paymentResultInfo.cardholderName` | String | max 64 | The cardholder's name. Note: This parameter is returned when the value of _paymentMethodType_ in the [pay (One-time Payments)](https://docs.antom.com/ac/ams/pay... |
| `paymentResultInfo.cardBin` | String | max 8 | The first six digits of the bank card number, used to identify the issuing bank and card type of the bank card. Note: This parameter is returned when the value of _... |
| `paymentResultInfo.lastFour` | String | max 4 | Last 4 digits of the card number. Note: This parameter is returned when the value of _paymentMethodType_ in the [pay (One-time Payments)](https://docs.antom.com... |
| `paymentResultInfo.expiryMonth` | Datetime | max 2 | The month the card expires. Pass in two digits representing the month. For example, if the expiry month is February, the value of this parameter is `02`. Note: This... |
| `paymentResultInfo.expiryYear` | Datetime | max 2 | The year the card expires. Pass in the last two digits of the year number. For example, if the expiry year is 2025, the value of this parameter is `25`. Note: This ... |
| `paymentResultInfo.refusalCodeRaw` | String | max 16 | The response code indicates an issuer’s reason for declining a transaction. Valid values see ISO codes. This parameter is re... |
| `paymentResultInfo.refusalReasonRaw` | String | max 256 | The error message that explains the response code. This parameter is returned when the following conditions are met: - The transaction fails. - The value of _paym... |
| `paymentResultInfo.merchantAdviceCode` | String | max 128 | The Merchant Advice Code (MAC) provides additional information about the type of Mastercard transaction or the reason why the payment failed. If the payment failed, the M... |
| `paymentResultInfo.credentialTypeUsed` | String | max 8; values: PAN,NETWORK_TOKEN | Indicates whether the transaction was processed with the primary account number (PAN) or a network token. - `PAN`: The transaction was processed using the PAN. - `NETWORK... |
| `paymentResultInfo.exemptionRequested` | String | max 64 | The exemption type that Antom requested for in the payment authorization. Valid values are: - `lowValue`: Transactions below €30 may be exempt from Strong Customer Authen... |
| `paymentResultInfo.issuerName` | String | max 256 | The issuing bank. This parameter is returned when the value of _paymentMethodType_ is `CARD`. |
| `paymentResultInfo.authorizationCode` | String |  | The authorization code returned by the payment method upon successful authorization. Note: This parameter is returned when the payment method provides an authorizat... |
| `paymentResultInfo.incrementalAuthorizationAvailable` | Boolean |  | Indicates whether incremental authorization is supported for this pre-authorization. Valid values are: - `true`: Indicates that incremental authorization is supported for... |
| `paymentResultInfo.extendedAuthorizationAvailable` | Boolean |  | Indicates whether extended authorization is supported for this pre-authorization. Valid values are: - `true`: Indicates that extended authorization is supported for this ... |
| `paymentResultInfo.refundOnAuthorizationAvailable` | Boolean |  | Indicates whether a refund can be initiated using _paymentId_ after the pre-authorization succeeds. Valid values are: - `true`: Indicates that subsequent refunds can be i... |
| `paymentResultInfo.fingerprint` | String | max 64 | Card fingerprint. A read-only parameter generated by the Antom system to uniquely identify the same card. The same bank card always generates the same _fingerprint_; a ca... |
| `paymentResultInfo.funding` | String | max 20; values: WALLETS,BNPL_POSTPAID,BNPL_INSTALLMENT | The funding type of the wallet. This parameter is returned when the value of _paymentMethodType_ is `GRABPAY_MY` or `GRABPAY_SG`. Valid values are: - `WALLETS`: indicates... |
| `paymentResultInfo.creditPayPlan` | CreditPayPlan |  | The installment plan information for an installment payment. Note: This parameter is returned when the value of _paymentResultInfo.funding_ is `BNPL_INSTALLMENT`. |
| `paymentResultInfo.creditPayPlan.installmentNum` | Integer | max 8 | The number of installment payments. The value of this parameter is fixed as `4`. Note: This parameter is returned when the value of _paymentResultInfo.funding_ is `... |
| `paymentResultInfo.creditPayPlan.interval` | String | max 16 | The interval of each installment payment. The valid value is `MONTH`. Note: This parameter is returned when the value of _paymentResultInfo.funding_ is `BNPL_INSTAL... |
| `authExpirytime` | Datetime |  | The expiration date and time of the authorization payment. You cannot capture the payment after this time. Notes: - This parameter is returned when the value of... |
| `promotionResult` | Array |  | Promotion result. Note: This parameter is returned when the buyer applied a promotion while placing an order. |
| `promotionResult.promotionType` | String, REQUIRED | max 32 | Promotion type. Valid values are: `DISCOUNT`: a deduction from the regular price. `INTEREST_FREE`: payment in interest-free installments. |
| `promotionResult.discount` | Discount |  | Discount information. Note: This parameter is returned when the buyer applied a promotion while placing an order. |
| `promotionResult.discount.discountName` | String | max 128 | The discount name displayed in the default language. |
| `promotionResult.discount.savingsAmount` | Amount |  | Discount amount. |
| `promotionResult.discount.value` | Integer, REQUIRED |  | The discount amount as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Note:... |
| `promotionResult.discount.currency` | String, REQUIRED |  | A 3-letter currency code that follows the ISO 4217 standard. |
| `paymentMethodType` | String | max 64 | The payment method type that is included in payment method options. See Payment methods to check the valid values. >... |
| `metadata` | String | max 2048 | Your metadata provided during the payment process. This parameter is returned when you provided value to the parameter in the [pay (One-time Payments)](https://docs... |

## capture (One-time Payments)

`POST /v1/payments/capture`

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `captureRequestId` | String, REQUIRED | max 64 | The unique ID that is assigned by the merchant to identify a capture request. Antom uses this field for idempotence control. - This field is an API idempotency field.For ... |
| `paymentId` | String, REQUIRED | max 64 | The unique ID that is assigned by Antom to identify a payment. |
| `captureAmount` | Amount, REQUIRED |  | The capture amount that the merchant requests to receive in the transaction currency. |
| `captureAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `captureAmount.value` | Integer, REQUIRED |  | The value of the amount as a positive integer in the smallest currency unit. For example, if the currency is USD and the amount is $1.00, set the value of this parameter ... |
| `transit` | Transit |  | Trip information, including trip modes, legs of trips and passenger information. |
| `transit.transitType` | String | values: FLIGHT,TRAIN,CRUISE,COACH | Trip modes, valid values are: - `FLIGHT`: indicates that the trip mode is by plane. - `TRAIN`: indicates that the trip mode is by train. - `CRUISE`: indicates that the tr... |
| `transit.agentCode` | String | max 8 | International Air Transport Association (IATA) code of the travel agent handling the reservation. |
| `transit.agentName` | String | max 32 | The name of the travel agency that made the reservation. |
| `transit.ticketNumber` | String | max 15 | A unique identifier for the ticket, typically formatted as IATA 3-letter airline code followed by a 10-digit number assigned by the airline. |
| `transit.ticketIssuerCode` | String | max 2 | The IATA two-word airline code of the ticketing agency. |
| `transit.restrictedTicketIndicator` | String | max 2; values: 0,1 | Indicates whether the ticket is refundable. Valid values include: - `0`: Indicates the ticket is refundable. - `1`: Indicates the ticket is not refundable. |
| `transit.legs` | Array |  | Information about sections of the trip, including departure time, arrival time, departure address, arrival address, transportation company name, carrier code and service ... |
| `transit.legs.departureTime` | Datetime |  | Time of departure for this leg of the trip. - The value follows the ISO 8601 standard format. For example, "2019... |
| `transit.legs.arrivalTime` | Datetime |  | Time of arrival for this leg of the trip. - The value follows the ISO 8601 standard format. For example, "2019-1... |
| `transit.legs.departureAddress` | Object |  | Departure address for this leg of the trip. |
| `transit.legs.region` | String | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `transit.legs.state` | String | max 8 | The state, country, or province. |
| `transit.legs.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `transit.legs.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `transit.legs.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `transit.legs.arrivalAddress` | Object |  | Destination address for this leg of the trip. |
| `transit.legs.region` | String | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `transit.legs.state` | String | max 8 | The state, country, or province. |
| `transit.legs.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `transit.legs.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `transit.legs.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `transit.legs.carrierNo` | String | max 2 | International Air Transport Association (IATA) code is obtained from the Official Airline Guide or its equivalent. |
| `transit.legs.carrierName` | String | max 128 | Company name of the transportation service provider for this leg of the trip. |
| `transit.legs.arrivalAirportCode` | String | max 3 | IATA code for the destination airport for this leg of the trip. |
| `transit.legs.fareBasis` | String | max 16 | Code used to identify the basic rules of segment fares. This code is defined by the airline to identify ticket attributes, such as cabin class (e.g., business class), far... |
| `transit.legs.couponNumber` | String | max 2 | An electronic coupon number used to identify a flight segment in a ticket. Each ticket may contain multiple electronic coupons, each corresponding to an independent fligh... |
| `transit.legs.flightNumber` | String | max 5 | The identifier assigned to a specific flight. Typically 1 to 4 numeric digits, optionally followed by a suffix letter. |
| `transit.passenger` | Object |  | Information about the passenger of the trip. |
| `transit.passenger.passengerName` | Object |  | The name of the passenger. |
| `transit.passenger.firstName` | String | max 32 | First name. |
| `transit.passenger.middleName` | String | max 32 | Middle name. |
| `transit.passenger.lastName` | String | max 32 | Last name. |
| `transit.passenger.fullName` | String | max 128 | Full name. |
| `transit.passenger.passengerCode` | String | max 32 | The unique identifier assigned by the air carrier to a passenger for accurately identifying the passenger in the airline system. |
| `transit.ancillaryData` | Object |  | Information about an ancillary purchase. |
| `transit.ancillaryData.connectedTicketNumber` | String | max 15 | The airline ticket number linked to this ancillary purchase. If the ancillary product is associated with a flight ticket (like a baggage fee), enter that flight ticket nu... |
| `transit.ancillaryData.services` | Array |  | Information about the services included in an ancillary purchase. |
| `transit.ancillaryData.categoryCode` | String | max 4 | Category code for the ancillary services offered. Retrieve the codes from the International Air Transport Association (IATA). |
| `transit.ancillaryData.subCategoryCode` | String | max 4 | Subcategory code for the ancillary service category. Retrieve codes from the International Air Transport Association (IATA). |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | The result of the API call. If this API is successfully called, the capture is successful. |
| `result.resultCode` | String, REQUIRED | max 64 | The result code that indicates the detailed processing result. |
| `result.resultStatus` | String, REQUIRED | values: S,F,U | Result status. Valid values are: - `S`: Indicates that the result status is successful. - `F`: Indicates that the result status is failed. - `U`: Indicates that the resul... |
| `result.resultMessage` | String, REQUIRED | max 256 | Result message that explains the result code. |
| `captureRequestId` | String | max 64 | The unique ID that is assigned by a merchant to identify a capture request. Note: This parameter is returned when the capture status is successful. |
| `captureId` | String | max 64 | The unique ID that is assigned by Antom to identify a capture. Note: This parameter is returned when the capture status is successful. |
| `paymentId` | String | max 64 | The unique ID that is assigned by Antom to identify a payment. Note: This parameter is returned when the capture status is successful. |
| `captureAmount` | Amount |  | The capture amount that the merchant requests to receive in the transaction currency. Note: This parameter is returned when the capture status is successful. |
| `captureAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `captureAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). For detail... |
| `captureTime` | Datetime |  | The time when Antom captures the payment. Note: This parameter is returned when the capture status is successful. - The value follows the [ISO 8601](https://www.iso.org... |
| `acquirerReferenceNo` | String | max 64 | The unique ID assigned by the non-Antom acquirer for the transaction. |

## cancel

`POST /v1/payments/cancel`

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `paymentId` | String | max 64 | The original payment ID of the payment request to be canceled, generated by Antom to uniquely identify the payment when the merchant initiates the original payment. _paym... |
| `paymentRequestId` | String | max 64 | The original payment request ID of the payment request to be canceled, generated by the merchant to uniquely identify a payment request. _paymentRequestId_ and _paymentId... |
| `merchantAccountId` | String | max 32 | The unique ID to identify a merchant account. Note: Specify this parameter when you use a single client ID across multiple locations. |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | Indicates whether this API is called successfully. If this API is successfully called, the payment is canceled successfully. |
| `result.resultCode` | String, REQUIRED | max 64 | Result code. Possible values are listed in the Result/Error codes table on this page. |
| `result.resultStatus` | String, REQUIRED | values: S,F,U | Result status. Valid values are: - `S`: Indicates that the payment is canceled successfully. - `F`: Indicates that the payment failed to be canceled. - `U`: Indicates tha... |
| `result.resultMessage` | String, REQUIRED | max 256 | Result message that explains the result code. |
| `paymentId` | String | max 64 | The unique ID that is assigned by Antom to identify a payment. Note: This field is returned when the cancellation succeeds (the value of _result.resultStatus_ is `S`). |
| `paymentRequestId` | String | max 64 | The unique ID that is assigned by a merchant to identify a payment request. Note: This field is returned when the cancellation succeeds (the value of _result.resultStat... |
| `cancelTime` | Datetime |  | The actual execution completion time of the payment cancellation process, which is the date and time when the payment cancellation succeeds. Note: This field is returne... |

## refund

`POST /v1/payments/refund`

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `refundRequestId` | String, REQUIRED | max 64 | The unique ID assigned by the merchant to identify a refund request. - This field is an API idempotency field.The merchant uses the refundRequestId field for idempotency ... |
| `paymentId` | String, REQUIRED | max 64 | The unique ID assigned by Antom for the original payment to be refunded. |
| `referenceRefundId` | String | max 64 | The unique ID to identify a refund, which is assigned by the merchant that directly provides services or goods to the customer. Note: Specify this field if this value i... |
| `refundAmount` | Amount, REQUIRED |  | The refund amount that is initiated by the merchant. |
| `refundAmount.currency` | String, REQUIRED | max 3 | The currency used for the corresponding payment of the refund. The value is a 3-letter currency code that follows the [ISO 4217](https://www.iso.org/iso-4217-currency-cod... |
| `refundAmount.value` | Integer, REQUIRED |  | The value of the amount as a positive integer in the smallest currency unit. For example, if the currency is USD and the amount is $1.00, set the value of this parameter ... |
| `refundReason` | String | max 256 | The refund reason. Note: Specify this field if you want to provide the refund reason to the user and payment method. |
| `refundNotifyUrl` | String | max 1024 | The URL that is used to receive the refund result notification. The URL must be either specified in the request or set in Antom Dashboard. Note: Specify this field if y... |
| `metadata` | String | max 2048 | This parameter is used for submitting metadata, including information for special use cases. Specify this parameter when you need to add metadata information. |
| `actualRefundAmount` | Amount |  | The refund amount in processing currency, that is, the amount the user actually paid. Note: Specify this parameter when the presentation currency differs from the p... |
| `actualRefundAmount.currency` | String, REQUIRED | max 3 | A 3-letter currency code that follows the ISO 4217 standard. |
| `actualRefundAmount.value` | Integer, REQUIRED |  | A positive integer in the smallest currency unit. Note: For details about the smallest currency unit, see [Smallest unit of the currency](https://docs.antom.com/ac/... |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | Indicates whether this API is called successfully. If this API is successfully called, the refund succeeds. |
| `result.resultCode` | String, REQUIRED | max 64 | Result code. Possible values are listed in the Result/Error codes table on this page. |
| `result.resultStatus` | String, REQUIRED | values: S,F,U | Result status. Valid values are: - `S`: Indicates that the refund succeeded. - `F`: Indicates that the refund failed. - `U`: Indicates that the refund result is unknown. |
| `result.resultMessage` | String, REQUIRED | max 256 | Result message that explains the result code. |
| `refundRequestId` | String | max 64 | The unique ID that is assigned by the merchant to identify a refund request. Note: This field is returned when the refund succeeds (the value of _result.resultStatus_ is ... |
| `refundId` | String | max 64 | The unique ID that is assigned by Antom to identify a refund. A one-to-one correspondence between _refundId_ and _refundRequestId_ exists. Note: This field is returned ... |
| `paymentId` | String | max 64 | The unique ID assigned by Antom for the original payment to be refunded. Note: This field is returned when the refund succeeds (the value of _result.resultStatus_ is `S... |
| `acquirerReferenceNo` | String | max 64 | The unique ID assigned by the non-Antom acquirer for the transaction. |
| `refundAmount` | Amount |  | The refund amount collected by the payer for this refund. Note: This field is returned when the refund succeeds (the value of _result.resultStatus_ is `S`). |
| `refundAmount.currency` | String, REQUIRED | max 3 | The transaction currency that is specified in the contract. The value is a 3-letter currency code that follows the [ISO 4217](https://www.iso.org/iso-4217-currency-codes.... |
| `refundAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY ¥100, a 0-decimal currency). Note: F... |
| `refundTime` | Datetime |  | The date and time when the refund reaches the state of success, failure, or unknown. - The value follows the [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.... |
| `grossSettlementAmount` | Amount |  | The refund settlement amount, which equals the refund amount multiplied by the value of _settlementQuote_. This field is returned when the currency exchange is predetermi... |
| `grossSettlementAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `grossSettlementAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a zero-decimal currency). Note: F... |
| `settlementQuote` | Quote |  | The exchange rate between the settlement currency and transaction currency. This field is returned when _grossSettlementAmount_ is returned. |
| `settlementQuote.quoteCurrencyPair` | String, REQUIRED | max 16 | The exchange rate between settlement currency and transaction currency. Two currencies are separated with a slash and use the 3-letter [ISO-4217](https://www.iso.org/iso-... |
| `settlementQuote.quoteExpiryTime` | Datetime |  | Expiration time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27... |
| `settlementQuote.quoteId` | String | max 64 | The unique ID that is assigned by Alipay to identify an exchange rate. |
| `settlementQuote.quotePrice` | Decimal, REQUIRED |  | The exchange rate used when a currency conversion between settlement currency and transaction currency occurs. - Value range: 1 - unlimited |
| `settlementQuote.quoteStartTime` | Datetime |  | Effective time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27T... |

## inquiryRefund

`POST /v1/payments/inquiryRefund`

Send `refundRequestId` or `refundId`. If both are sent, `refundId` wins. `refundStatus` is `SUCCESS`, `PROCESSING`, or `FAIL`.

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `refundRequestId` | String | max 64 | The unique ID assigned by a merchant to identify a refund request. _refundRequestId_ and _refundId_ cannot both be null. Special characters are not supported. If both _re... |
| `refundId` | String | max 64 | The unique ID assigned by Antom to identify a refund. _refundRequestId_ and _refundId_ cannot both be null. A one-to-one correspondence between _refundId_ and _refundRequ... |
| `merchantAccountId` | String | max 32 | The unique ID to identify a merchant account. Note: Specify this parameter when you use a single client ID across multiple locations. |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | Information about the request call result. Note: This field does not indicate the refund result. This field only indicates whether the inquiryRefund interface is called... |
| `result.resultCode` | String, REQUIRED | max 64 | Result code. Possible values are listed in the Result/Error codes table on this page. |
| `result.resultStatus` | String, REQUIRED | max 1; values: S,F,U | Result status. Valid values are: - `S`: Successful - `F`: Failed - `U`: Unknown - Maximum length: 1 character |
| `result.resultMessage` | String, REQUIRED | max 256 | Result message that explains the result code. |
| `refundId` | String | max 64 | The unique ID assigned by Antom to identify a refund. A one-to-one correspondence between _refundId_ and _refundRequestId_ exists. Note: This field is returned when the... |
| `refundRequestId` | String | max 64 | The unique ID assigned by the merchant to identify a refund request. Note: This field is returned when the API is called successfully (the value of _result.resultStatus... |
| `refundAmount` | Amount |  | The refund amount that is initiated by the merchant. Note: This field is returned when the API is called successfully (the value of _result.resultStatus_ is `S`). |
| `refundAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `refundAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. For example, $1.00 should be charged as 100 cents to charge USD 1.00, or 100 to charge JPY 100, ... |
| `refundTime` | Datetime |  | The date and time when the refund reaches a final state of success on the Antom side. Note: This field is returned when the refund succeeds (the value of _refundStatus_... |
| `refundStatus` | String | max 16; values: SUCCESS,PROCESSING,FAIL | Indicates the final status of the Antom refund. Valid values are: - `SUCCESS`: Indicates the refund succeeded. - `PROCESSING`: Indicates the refund is under processing. -... |
| `grossSettlementAmount` | Amount |  | The refund settlement amount, which equals the refund amount multiplied by the value of _settlementQuote_. Note: This parameter is returned when the exchange rate i... |
| `grossSettlementAmount.currency` | String, REQUIRED | max 3 | The acquiring currency that is specified in the contract. A 3-letter currency code that follows the ISO 4217 standard. |
| `grossSettlementAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a zero-decimal currency). Note: F... |
| `settlementQuote` | Quote |  | The exchange rate between the settlement currency and transaction currency. This field is returned when _grossSettlementAmount_ is returned. |
| `settlementQuote.guaranteed` | Boolean |  | Guaranteed exchange rate available for payment. |
| `settlementQuote.quoteCurrencyPair` | String, REQUIRED | max 16 | The exchange rate between settlement currency and transaction currency. Two currencies are separated with a slash and use the three-letter [ISO-4217](https://www.iso.org/... |
| `settlementQuote.quoteExpiryTime` | Datetime |  | Expiration time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27... |
| `settlementQuote.quoteId` | String | max 64 | The unique ID that is assigned by Antom to identify an exchange rate. |
| `settlementQuote.quotePrice` | Decimal, REQUIRED |  | The exchange rate used when a currency conversion between settlement currency and transaction currency occurs. - Value range: 0 - unlimited |
| `settlementQuote.quoteStartTime` | Datetime |  | Effective time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27T... |
| `arn` | String |  | Acquirer reference number. If the payment method includes an acquirer reference number, this field will have a value. - Value range: [0, +∞) |
| `metadata` | String | max 2048 | Your metadata provided during the refund process. This parameter is returned when you provided value to the parameter in the [refund](https://docs.antom.com/ac/ams/... |
