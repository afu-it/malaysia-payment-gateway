# Antom Notifications

How Antom sends asynchronous results and how to verify and acknowledge them. Field tables generated from the official reference pages on 2026-10-21; long descriptions are shortened (ending in `...`).

## Source URLs

- Notifications guide: `https://docs.antom.com/ac/cashierpay/notifications.md`
- Notification URL setup and testing: `https://docs.antom.com/ac/merchant_service/notification.md`
- notifyPayment: `https://docs.antom.com/ac/ams/paymentrn_online.md`
- notifyCapture (One-time Payments): `https://docs.antom.com/ac/ams/notify_capture.md`
- notifyRefund: `https://docs.antom.com/ac/ams/notify_refund.md`
- Signature guide: `https://docs.antom.com/ac/ams/digital_signature.md`

## Notification URL

- Set per request with `paymentNotifyUrl` (pay, createPaymentSession) and `refundNotifyUrl` (refund), or once in Antom Dashboard > Developer > Notification URL.
- When both are set, the API value wins (notification URL page and APM guide).
- Exception: the capture guide says the Dashboard value wins for capture notifications, and a separate capture URL cannot be passed through the API.
- The Dashboard accepts fixed URLs only, with no order parameters.
- Dashboard notification types: notifyAuthorization, notifyVaulting, notifyPayment (One-time and Subscription), notifyCapture, notifySubscription, notifyDispute, notifyRefund.
- Dashboard > Developer > Notification URL > Test sends a simulated notification and checks your reply.

## Handling Steps

1. Read the raw body bytes before any JSON parsing.
2. Read `client-id`, `request-time`, `signature` headers.
3. Verify `POST <http-uri>\n<client-id>.<request-time>.<raw-body>` with `<ANTOM_PUBLIC_KEY>` (see `antom-api.md`).
4. Reject on failure. On success, look up the local order by `paymentRequestId` and check `paymentId`, `paymentAmount`, currency, and `notifyType`.
5. Process idempotently. Record each processed notification.
6. Reply with the fixed body, whether the payment succeeded or failed:

```json
{"result":{"resultCode":"SUCCESS","resultStatus":"S","resultMessage":"success"}}
```

No signature is required on the reply.

## Timing And Retries

- A successful payment usually notifies within 3 to 5 seconds. An unpaid order notifies after it closes (default 14 minutes).
- No notification is sent when pay returns `F` synchronously, or when the buyer never submits on Checkout Page before the session expires.
- Missing or wrong replies trigger resends within 24 hours at 0s, 2 min, 10 min, 10 min, 1 h, 2 h, 6 h, 15 h. The notification URL page and APM guide say up to eight times; another page says seven. Plan for eight.
- Delivery is not guaranteed and some wallets send nothing on failure. Keep an inquiryPayment fallback.

## What To Ship On

| Method | Ship when |
| --- | --- |
| APMs (FPX, TNG, Boost, GrabPay MY, ShopeePay MY) | notifyPayment `notifyType` `PAYMENT_RESULT` with `result.resultStatus` `S` |
| Cards, Apple Pay, Google Pay | notifyCapture `CAPTURE_RESULT` with `result.resultStatus` `S` |

`notifyType` values on notifyPayment: `PAYMENT_RESULT`, `PAYMENT_PENDING`, `UPDATE_AMOUNT_RESULT`. `result.resultStatus` is `S` or `F` only.

## notifyPayment (One-time Payments)

Antom sends this to your notification URL.

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `notifyType` | String, REQUIRED | values: PAYMENT_RESULT,PAYMENT_PENDING,UPDATE_AMOUNT_RESULT | Type of the payment status notification. Valid values are: - `PAYMENT_RESULT`: Indicates that the notification is about the payment result. - `PAYMENT_PENDING`: Indicates... |
| `result` | Result, REQUIRED |  | Details about the payment result, such as payment status, result code, and result message. |
| `result.resultCode` | String, REQUIRED | max 64 | Result code. The possible result codes are listed in the Result/Error codes table on this page. Not all the result codes listed in the table can be returned by every paym... |
| `result.resultStatus` | String, REQUIRED | values: S,F | Payment status. Valid values are: - `S`: indicates the payment is successful when _notifyType_ is `PAYMENT_RESULT`; indicates the payment is being processed when _notifyT... |
| `result.resultMessage` | String, REQUIRED | max 256 | Result message that explains the result code. |
| `paymentRequestId` | String, REQUIRED | max 64 | The unique ID that is assigned by a merchant to identify a payment request. |
| `paymentId` | String, REQUIRED | max 64 | The unique ID that is assigned by Antom to identify a payment. A one-to-one correspondence between _paymentId_ and _paymentRequestId_ exists. |
| `paymentAmount` | Amount, REQUIRED |  | The payment amount that the merchant requests to receive in the order currency. |
| `paymentAmount.currency` | String, REQUIRED | max 3 | The transaction currency, which is a 3-letter currency code that follows the ISO 4217 standard. |
| `paymentAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Notes: For... |
| `paymentCreateTime` | Datetime, REQUIRED |  | The date and time when the payment is created. - The value follows the ISO 8601 standard format. For example, "2... |
| `paymentTime` | Datetime |  | The date and time when the payment reaches a final state of success. Note: This field is returned only when the payment reaches a final state of success (the value ... |
| `pspCustomerInfo` | PspCustomerInfo |  | Information about the customer of the digital wallets. Note: This field is returned when the digital wallets can provide the related information. |
| `pspCustomerInfo.pspName` | String | max 64 | The name of the digital wallets. Note: This field is returned when the digital wallets can provide the related information. |
| `pspCustomerInfo.pspCustomerId` | String | max 64 | The customer ID of the digital wallets. Note: This field is returned when the digital wallets can provide the related information. |
| `pspCustomerInfo.displayCustomerId` | String | max 64 | The customer ID used for display. For example, _loginId_. Note: This field is returned when the digital wallets can provide the related information. |
| `customsDeclarationAmount` | Amount |  | The total amount for customs declaration. Note: This field is returned only when the payment succeeds and the wallet is AlipayCN. |
| `customsDeclarationAmount.currency` | String, REQUIRED | max 3 | The currency used for declaration. The value is a 3-letter currency code that follows the ISO 4217 standard. |
| `customsDeclarationAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Notes: For... |
| `grossSettlementAmount` | Amount |  | The value of this field equals to transaction amount multiplied by the value of _settlementQuote_. This field is returned when the currency exchange is predetermined and ... |
| `grossSettlementAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `grossSettlementAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a zero-decimal currency). Not... |
| `settlementQuote` | Quote |  | The exchange rate between the settlement currency and transaction currency. Note: This field is returned when _grossSettlementAmount_ is returned. |
| `settlementQuote.guaranteed` | Boolean |  | Guaranteed exchange rate available for payment. |
| `settlementQuote.quoteCurrencyPair` | String, REQUIRED | max 16 | The exchange rate between settlement currency and transaction currency. Two currencies are separated with a slash and use the three-letter [ISO-4217](https://www.iso.org/... |
| `settlementQuote.quoteExpiryTime` | Datetime |  | Expiration time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27... |
| `settlementQuote.quoteId` | String | max 64 | The unique ID that is assigned by Antom to identify an exchange rate. |
| `settlementQuote.quotePrice` | Decimal, REQUIRED |  | The exchange rate used when a currency conversion between settlement currency and transaction currency occurs. - Value range: 1 - unlimited |
| `settlementQuote.quoteStartTime` | Datetime |  | Effective time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27T... |
| `acquirerReferenceNo` | String | max 64 | The unique ID assigned by the non-Antom acquirer for the transaction. |
| `paymentResultInfo` | PaymentResultInfo object |  | The payment result information. Note: This parameter is returned when the value of _paymentMethodType_ in the [pay (One-time Payments)](https://docs.antom.com/a... |
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
| `paymentResultInfo.threeDSResult.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: This parameter is returned for 3D Secure 2. |
| `paymentResultInfo.threeDSResult.xid` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. This parameter is returned for 3D Secure 1.0. |
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
| `paymentResultInfo.cardCategory` | string | max 32; values: CONSUMER,COMMERCIAL | The payment card category. Valid vaules are: - `CONSUMER`: the personal card - `COMMERCIAL`: the business card Note: This parameter is returned when all the followi... |
| `paymentResultInfo.creditPayPlan` | CreditPayPlan |  | The installment plan information for an installment payment. Note: This parameter is returned when the buyer successfully made a payment in installments using a car... |
| `paymentResultInfo.creditPayPlan.installmentNum` | Integer | max 8 | The number of installment payments. Note: This parameter is returned when the buyer successfully made a payment in installments using a card. |
| `paymentResultInfo.creditPayPlan.interval` | String | max 16 | The interval of each installment payment. The valid value is `MONTH`. Note: This parameter is returned when the buyer successfully made a payment in installments us... |
| `paymentResultInfo.creditPayPlan.interestRate` | String | max 16 | The installment interest rate, expressed as a percentage. For example, _interestRate_ = 0.1 means the installment interest rate is 0.1%. Note: This parameter is ret... |
| `paymentResultInfo.cardholderName` | String | max 64 | The cardholder's name. Note: This parameter is returned when the value of _paymentMethodType_ in the [pay (One-time Payments)](https://docs.antom.com/ac/ams/pay... |
| `paymentResultInfo.cardBin` | String | max 8 | The first six digits of the bank card number, used to identify the issuing bank and card type of the bank card. Note: This parameter is returned when the value of _... |
| `paymentResultInfo.lastFour` | String | max 4 | Last 4 digits of the card number. Note: This parameter is returned when the value of _paymentMethodType_ in the [pay (One-time Payments)](https://docs.antom.com... |
| `paymentResultInfo.expiryMonth` | String | max 2 | The month the card expires. Pass in two digits representing the month. For example, if the expiry month is February, the value of this parameter is `02`. Note: This... |
| `paymentResultInfo.expiryYear` | String | max 2 | The year the card expires. Pass in the last two digits of the year number. For example, if the expiry year is 2025, the value of this parameter is `25`. Note: This ... |
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
| `promotionResult` | Array |  | Promotion result. Note: This parameter is returned when the buyer applied a promotion while placing an order. |
| `promotionResult.promotionType` | String, REQUIRED | max 32 | Promotion type. Valid values are: `DISCOUNT`: a deduction from the regular price. `INTEREST_FREE`: payment in interest-free installments. |
| `promotionResult.discount` | Discount |  | Discount information. Note: This parameter is returned when the buyer applied a promotion while placing an order. |
| `promotionResult.discount.discountName` | String | max 128 | The discount name displayed in the default language. |
| `promotionResult.discount.savingsAmount` | Amount |  | Discount amount. |
| `promotionResult.discount.value` | Integer, REQUIRED |  | The discount amount as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Note:... |
| `promotionResult.discount.currency` | String, REQUIRED |  | A 3-letter currency code that follows the ISO 4217 standard. |
| `paymentMethodType` | String | max 64 | The payment method type that is included in payment method options. See Payment methods to check the valid valu... |
| `metadata` | String | max 2048 | Your metadata provided during the payment process. This parameter is returned when you provided value to the parameter in the [pay (One-time Payments)](https://docs... |
| `authExpiryTime` | Datetime |  | The expiration date and time of the authorization payment. You cannot capture the payment after this time. Notes: - This parameter is returned when the value of... |
| `updateRequestId` | String | max 64 | The request ID obtained after calling the updateAmount API. Note: This parameter is returned when the value of... |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | A fixed value, which is sent to Antom to acknowledge that the notification is received. |
| `result.resultCode` | String, REQUIRED | max 64 | The value is fixed as `SUCCESS`. |
| `result.resultStatus` | String, REQUIRED |  | The value is fixed as `S`. |
| `result.resultMessage` | String, REQUIRED | max 256 | The value is fixed as `success`. |

## notifyCapture (One-time Payments)

`notifyType` is `CAPTURE_RESULT`. Save `captureId` for refunds.

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | Details about the payment capture result, such as capture status, result code, and result message. |
| `result.resultCode` | String, REQUIRED | max 64 | The result code that indicates the detailed processing result. |
| `result.resultStatus` | String, REQUIRED | values: S,F | The capture status. Valid values are: - `S`: Indicates that the capture status is successful. - `F`: Indicates that the capture status is failed. |
| `result.resultMessage` | String | max 256 | The result message that describes the result code in detail. |
| `notifyType` | String, REQUIRED |  | The type of notification. The notification type of this API is `CAPTURE_RESULT`. |
| `captureRequestId` | String, REQUIRED | max 64 | The unique ID that is assigned by the merchant to identify a capture request. |
| `paymentId` | String, REQUIRED | max 64 | The unique ID that is assigned by Antom to identify a payment. |
| `captureId` | String, REQUIRED | max 64 | The unique ID assigned by Antom to identify a capture. |
| `captureAmount` | Amount, REQUIRED |  | The capture amount that the merchant requests to receive in the transaction currency. |
| `captureAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `captureAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). For detail... |
| `captureTime` | Datetime |  | The time when Antom captures the payment. |
| `acquirerReferenceNo` | String | max 64 | The unique ID assigned by the non-Antom acquirer for the transaction. |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | A fixed value, which is sent to Antom to acknowledge that the notification is received. |
| `result.resultCode` | String, REQUIRED | max 64 | Result code. The value is fixed as `SUCCESS`. |
| `result.resultStatus` | String, REQUIRED |  | Result status. The value is fixed as `S`, which means the notification is received successfully. |
| `result.resultMessage` | String, REQUIRED | max 256 | Result message that explains the result code. The value is fixed as `success`. |

## notifyRefund

`notifyType` is `REFUND_RESULT`. `refundStatus` is `SUCCESS` or `FAIL`.

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `notifyType` | String, REQUIRED | values: REFUND_RESULT | The type of the refund status notification. The valid value is: - `REFUND_RESULT`: Indicates that the notification is about the refund result. |
| `result` | Result, REQUIRED |  | The details about the refund result, such as refund status, result code, and result message. |
| `result.resultCode` | String, REQUIRED | max 64 | Result code. Possible values are listed in the Result/Error codes table on this page. |
| `result.resultStatus` | String, REQUIRED | values: S,F | Refund status. Valid values are: - `S`: Indicates that the refund has been accepted successfully by the channel. - `F`: Indicates that the refund failed. |
| `result.resultMessage` | String, REQUIRED | max 256 | Result message that explains the result code. |
| `refundStatus` | String, REQUIRED | values: SUCCESS,FAIL | Indicates the refund result. Valid values are: - `SUCCESS`: Indicates that the refund succeeded. - `FAIL`: Indicates that the refund failed. |
| `refundRequestId` | String, REQUIRED | max 64 | The unique ID assigned by a merchant to identify a refund request. |
| `refundId` | String, REQUIRED | max 64 | The unique ID that is assigned by Antom to identify a refund. A one-to-one correspondence between _paymentId_ and _paymentRequestId_ exists. |
| `refundAmount` | Amount, REQUIRED |  | The refund amount that is initiated by the merchant. |
| `refundAmount.currency` | String, REQUIRED | max 3 | The currency used for the corresponding payment of the refund. The value is a 3-letter currency code that follows the [ISO 4217](https://www.iso.org/iso-4217-currency-cod... |
| `refundAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY ¥100, a 0-decimal currency). Notes: Fo... |
| `refundTime` | Datetime |  | The date and time when the refund reaches a final state of success or failure. Note: This field is returned when the refund succeeds (the value of _result.resultStatus_... |
| `grossSettlementAmount` | Amount |  | The refund settlement amount, which equals the refund amount multiplied by the value of _settlementQuote_. This field is returned when the currency exchange is predetermi... |
| `grossSettlementAmount.currency` | String, REQUIRED | max 3 | The acquiring currency that is specified in the contract. A 3-letter currency code that follows the ISO 4217 standard. |
| `grossSettlementAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a zero-decimal currency). Note: F... |
| `settlementQuote` | Quote |  | The exchange rate between the settlement currency and transaction currency. This field is returned when _grossSettlementAmount_ is returned. |
| `settlementQuote.guaranteed` | Boolean |  | Guaranteed exchange rate available for payment. |
| `settlementQuote.quoteCurrencyPair` | String, REQUIRED | max 16 | The exchange rate between settlement currency and transaction currency. Two currencies are separated with a slash and use the three-letter [ISO-4217](https://www.iso.org/... |
| `settlementQuote.quoteExpiryTime` | Datetime |  | Expiration time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27... |
| `settlementQuote.quoteId` | String | max 64 | The unique ID that is assigned by Antom to identify an exchange rate. |
| `settlementQuote.quotePrice` | Decimal, REQUIRED |  | The exchange rate used when a currency conversion between settlement currency and transaction currency occurs. - Value range: 0 - unlimited |
| `settlementQuote.quoteStartTime` | Datetime |  | Effective time of the exchange rate. - The value follows the ISO 8601 standard format. For example, "2019-11-27T... |
| `arn` | String |  | Acquirer reference number. If the payment method used includes an aquirer reference number, this field will have a value. - Value range: [0, +∞) |
| `metadata` | String | max 2048 | Your metadata provided during the refund process. This parameter is returned when you provided value to the parameter in the [refund](https://docs.antom.com/ac/ams/... |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | A fixed value, which is sent to Antom to acknowledge that the notification is received. |
| `result.resultCode` | String, REQUIRED | max 64 | The value is fixed as `SUCCESS`. |
| `result.resultStatus` | String, REQUIRED |  | The value is fixed as `S`. |
| `result.resultMessage` | String, REQUIRED | max 256 | The value is fixed as `success`. |
