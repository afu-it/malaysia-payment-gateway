# Antom Result And Error Codes

Copied from the "Result/Error codes" table of each official API page on 2026-10-21. Long cells are shortened (ending in `...`). `Status` is `result.resultStatus`: `S` success, `F` failed, `U` unknown or in process.

General rules from the docs:

- `U` is not a final failure. Follow the "Further action" column.
- `UNKNOWN_EXCEPTION` and `REQUEST_TRAFFIC_EXCEED_LIMIT` (`U`): call again; contact support if it persists.
- `KEY_NOT_FOUND`, `ACCESS_DENIED`, `NO_INTERFACE_DEF`: check keys, client ID, and endpoint path (including the sandbox path). The signature guide notes that a response returning `SIGNATURE_INVALID` carries no signature header.
- `REPEAT_REQ_INCONSISTENT`: same request ID with different amount, currency, or method.

## pay (One-time Payments)

Source: `https://docs.antom.com/ac/ams/payment_cashier.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| SUCCESS | S | Success | The payment is successful, no further action is needed. |
| ACCESS_DENIED | F | Access is denied. | Contact Antom Technical Support for detailed reasons. |
| ACCOUNT_CLOSED | F | The account is closed.  Please verify the account number for accuracy and do not attempt to process the same PAN or token again. | Contact the issuer immediately or use another card to pay the transaction. |
| AUTHENTICATION_REQUIRED | F | 3D Secure authentication is required. | Reinitiate the payment and redirect the user to perform 3D Secure authentication. |
| BLOCKED_BY_CARDHOLDER | F | This card has been locked by the user. | Contact the issuer immediately or use another card to pay the transaction. |
| CARD_EXPIRED | F | Card has expired. | Check whether the value of paymentMethod.paymentMethodMetaData.expiryYear or paymentMethod.paymentMethodMetaData.expiryMonth is correct. If not correct, pass in the correct value. If correct, contact Antom Technical Supp... |
| CARD_NOT_SUPPORTED | F | The card used for the transaction is not supported. | Use another card to pay the transaction. |
| CURRENCY_NOT_SUPPORT | F | The currency is not supported. | Contact Antom Technical Support for detailed reasons. |
| DOMESTIC_DEBIT_TRANSACTION_NOT_ALLOWED | F | Domestic debit transaction not allowed. (Regional use only) | Contact the issuer immediately or use another card to pay the transaction. |
| DO_NOT_HONOR | F | The payment is declined by the issuing bank. | Retry the payment using a different card or contact the issuing bank. |
| EXPIRED_CODE | F | The payment code is expired. | The user needs to refresh the payment code. |
| FRAUD_REJECT | F | The transaction cannot be further processed because of risk control. If the user has already paid for the transaction, the transaction will be refunded. | Contact Antom Technical Support when one of the following conditions is met:You want to make an appeal. The user does not re |
| IDENTITY_AGE_NOT_ENOUGH | F | The buyer's age does not meet the requirements. | Please confirm your identification details and try again. |
| IDENTITY_VERIFY_FAILED | F | Identity verification timed out or the result couldn't be retrieved. | Please confirm your identification details and try again. |
| INSTALLMENT_NOT_SUPPORTED | F | This card or merchant does not support installments. | This card does not support installment, please change a card to pay. |
| INVALID_ACCESS_TOKEN | F | The access token is expired, revoked, or does not exist. | Check whether accessToken is correct. If not correct, pass in the correct value. If correct, contact Antom Technical Support for detailed reasons. |
| INVALID_AMOUNT | F | The transaction was declined by the issuing bank due to various reasons. For example, the specified amount is invalid or exceeds the maximum amount limit. | Contact Antom Technical Support for detailed reasons. |
| INVALID_CARD_NUMBER | F | The number of the card used for the transaction is invalid. | Check whether the value of paymentMethod.paymentMethodMetaData.cardno is correct:If not correct, pass in the correct value and try again. If correct, this card is not supported by the transaction, use another |
| INVALID_CONTRACT | F | The parameter values in the contract do not match those in the current transaction. | Check whether the parameter values in the contract match those in the current transaction. If the values match, contact Antom Technical Support to troubleshoot the issue. |
| INVALID_CVV | F | The CVV code is invalid. | Prompt the user to check if the entered CVV is correct. |
| INVALID_IDENTITY_STATUS | F | Abnormal identity status. | Please confirm your identification details and try again. |
| INVALID_INSTALLMENT_PLAN | F | The number or the interval of installment plans is incorrect. | Please adjust the number of installments and try again. |
| INVALID_MERCHANT_STATUS | F | The merchant status is abnormal because restrictions exist. | Contact Antom Technical Support for detailed reasons. |
| INVALID_PAYMENT_CODE | F | The payment code cannot be accepted by Antom. | Choose other payment methods. Contact Antom Technical Support if the payment method is supported. |
| INVALID_PAYMENT_METHOD_META_DATA | F | The payment method metadata is invalid. | Check whether payment method metadata is correct. If correct, contact Antom Technical Support. |
| KEY_NOT_FOUND | F | The private key or public key of Antom or the merchant is not found. | Check whether the private key or public key exists. If not, upload the private key in Antom Developer Center. |
| MERCHANT_KYB_NOT_QUALIFIED | F | The payment failed because of the merchant's KYB status. The merchant is either not KYB compliant, or the KYB status is not qualified for this transaction. | Contact Antom Technical Support for detailed reasons. |
| MERCHANT_NOT_REGISTERED | F | The merchant is not registered. | Please register the merchant by using the registration interface. Contact Antom Technical Support if failed to call the registration interface. |
| NO_INTERFACE_DEF | F | API is not defined. | Check whether the URL is correct. Please refer to the endpoint in the API documentation. |
| NO_PAY_OPTIONS | F | No payment options are available. | Contact Antom Technical Support for detailed reasons. |
| ORDER_IS_CANCELED | F | The request you initiated has the same paymentRequestId as the previously paid transaction, which is canceled. | Use a new paymentRequestId to initiate the payment again. |
| ORDER_IS_CLOSED | F | The order associated with the paymentRequestId in your request is closed due to payment failure or payment timeout, etc. | Use a new paymentRequestId to initiate a payment. |
| ORDER_NOT_EXIST | F | The order does not exist. | Check whether paymentId is correct. |
| ORDER_STATUS_INVALID | F | The transaction cannot be further processed because the order status is invalid. | Check the order status of the transaction or contact Antom Technical Support for detailed reasons. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PAYMENT_AMOUNT_EXCEED_LIMIT | F | The payment amount is greater than the maximum amount allowed by the contract or payment method. | Check whether the payment amount exceeds the limit or use a lower amount and try again. Contact Antom technical support to know the specific limitation. |
| PAYMENT_COUNT_EXCEED_LIMIT | F | The maximum number of payments exceeds the limit that is specified by the payment method. | Contact Antom Technical Support to know the specific limitation. |
| PAYMENT_IN_PROCESS | U | The payment is being processed. | Get any of the URLs (appLinkUrl,  normalUrl, schemeUrl) and open the cashier page. If no URL is returned, call the pay API again with a new paymentRequestId value. If the issue persists, contact Antom Technical Support. |
| PAYMENT_METHOD_SYSTEM_ERROR | F | Payment method system error. | Issuer system error, please contact the issuer. |
| PAYMENT_NOT_QUALIFIED | F | The merchant is not qualified to pay because the merchant is not registered, does not have a contract for Tokenized Payment, or is forbidden to make a payment. | Contact Antom Technical Support for detailed reasons. |
| PICKUP_CARD | F | The card issuer requests that the merchant retain the card.  This typically arises in cases of suspected counterfeit or stolen cards. | Contact the issuer immediately or use another card to pay the transaction. |
| PROCESS_FAIL | F | A general business failure occurred. | It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| REPEAT_REQ_INCONSISTENT | F | The amount or currency is different from the previous request. | Ensure all the fields in the requests are the same or use a new paymentRequestId to initiate the payment again. |
| REQUEST_TRAFFIC_EXCEED_LIMIT | U | The request traffic exceeds the limit. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |
| RESTRICTED_CARD | F | The card issuer has imposed restrictions on where the card can be used. | Contact the issuer immediately or use another card to pay the transaction. |
| RISK_REJECT | F | The transaction cannot be further processed because of risk control. If the user has already paid for the transaction, the transaction will be refunded. | If the user does not receive the refund within two weeks, contact Antom Technical Support. |
| SETTLE_CONTRACT_NOT_MATCH | F | No matched settlement contract can be found. | Check the following for a solution: Specify one settlement currency from the multiple currencies that the merchant signed up for. Check if the settlement currency is specified in the settlement contracts. The merchant d |
| SYSTEM_ERROR | F | A system error occurred. | Do not retry, and contact Antom Technical Support for more details. |
| UNKNOWN_EXCEPTION | U | An API call has failed, which is caused by unknown reasons. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |
| USER_AMOUNT_EXCEED_LIMIT | F | The payment amount exceeds the user payment limit. | Create a new payment by using an amount less than or equal to the account's available balance, or contact Antom Technical Support. |
| USER_BALANCE_NOT_ENOUGH | F | The payment cannot be completed because the user balance in the corresponding payment method is not enough. | Please top up the account or choose other payment methods. |
| USER_KYC_NOT_QUALIFIED | F | The payment failed because of the user's KYC status. The user is either not KYC compliant, or the KYC status is not qualified for this transaction (for example, limitations on the payment amount or product information). | Complete the KYC verification first. |
| USER_AUTHENTICATION_NOT_FINISHED | F | User authentication failed due to a timeout.  This error code is returned when authentication is not completed after the 3D Secure page is launched during payment. | Re-initiate a payment. |
| USER_NOT_EXIST | F | The user does not exist on the wallet side. | Contact Antom Technical Support for detailed reasons. |
| USER_NOT_SUBMITTED | F | Payment timeout due to user's failure to submit. When using the landing page/SDK integration, a timeout will be triggered if the user fails to submit after launching the payment details fill-in page. | Re-initiate a payment. |
| USER_PAYMENT_VERIFICATION_FAILED | F | User fails to pass the payment verification in the methods like OTP, PIN, and so on. | Contact Antom Technical Support to know the specific reasons. |
| USER_STATUS_ABNORMAL | F | The user status is abnormal on the wallet side. | Contact Antom Technical Support to know the specific reasons. |
| VERIFY_TIMES_EXCEED_LIMIT | F | The current verification code failed to pass the payment verification too many times. | The user must get a new verification code. |
| VERIFY_UNMATCHED | F | The verification code is invalid. | The user must get a new verification code. |
| SELECTED_CARD_BRAND_NOT_AVAILABLE | F | The card brand that the user selected to pay is not available. | The card brand that the user selected to pay is not available. |
| PAYMENT_PROHIBITED | F | The payment cannot be processed because the goods are prohibited from sale in the country. | You are not allowed to appeal against this transaction. |
| INVALID_EXPIRATION_DATE | F | The value of paymentMethod.paymentMethodMetaData.expiryYear or paymentMethod.paymentMethodMetaData.expiryDate is invalid. | Check whether the value of paymentMethod.paymentMethodMetaData.expiryYear or paymentMethod.paymentMethodMetaData.expiryDate is correct:If not c |
| INVALID_CPF | F | The provided CPF number is invalid. | Please confirm your identification details and try again. |
| REFERRAL_ISSUER | F | The transaction was declined by the card issuer. | The cardholder should contact their issuer for clarification and retry after resolution. |
| TRANSACTION_NOT_PERMITTED | F | The card issuer declined the transaction on this card/account. | Use an alternative payment method or contact the issuer. |
| INVALID_PIN | F | The PIN is incorrect or invalid. | Ensure the correct PIN is entered with valid format/length. |
| LIFECYCLE | F | Declined due to invalid card data. | Verify card details (number/expiry/CVV). |
| LAW_VIOLATION | F | The issuing bank declined the transaction. | Contact the issuer for details. |
| PIN_REQURED | F | PIN is mandatory but not provided. | Reattempt with a valid PIN. |
| BLOCKED_FIRST_USED | F | When a card is either temporarily blocked, has not been activated for first use, or is subject to special conditions that prevent processing (e.g., new cardholder not activated). | Activate the card or check with the issuer. |
| CANNOT_VERIFY_PIN | F | PIN verification failed. | Retry without PIN or contact issuer. |
| LOST_CARD | F | Lost card, pick up card (fraud account) | Contact the issuer immediately. |
| INVALID_MCC | F | Invalid MCC. | Do not retry, and contact Antom Technical Support for more details. |
| INVALID_ISSUER | F | It is not possible to reach out to the shopper's issuer for transaction authorization. | Do not retry. Please contract your issuer to confirm the card status. |
| STOLEN_CARD | F | Stolen card, pick up (fraud account). | Contact the issuer immediately. |
| SECURITY_REJECT | F | The transaction was declined due to a security issue detected by the card issuer. | Verify Information: Ensure that all card details, including the CVV/CVC and billing address, are entered correctly.Contact Issuer: Reach out to the bank or financial institution that issued th |
| INVALID_TRANSACTION | F | Invalid transaction. | Do not retry, and contact Antom Technical Support for more details. |
| SURCHARGE_NOT_ALLOWED | F | Surcharge not permitted on this card. | Contact the issuer. |
| EXCEEDS_PREAUTHORIZED_AMOUNT_LIMIT | F | Transaction exceeds preauthorized approval limit. | Reduce amount or request higher limit. |
| STOP_RECURRING_PAYMENT | F | The buyer has requested to cancel or stop their subscription. | The cardholder should contact their issuer for clarification and can either try again after resolving the issue. |
| NO_SELECTED_ACCOUNT | F | Used when the cardholder has selected specific account but does not have. | May reattempt using correct account type. |
| POLICY | F | Policy restrictions. | The cardholder should contact their issuer for clarification. |
| RE_ENTER_TRANSACTION | F | Transaction temporarily cannot be processed. | The cardholder should contact their issuer for clarification and may attempt again once the issue is resolved. |
| TRANSACTION_NOT_ALLOWED_AT_TERMINAL | F | Transaction is not allowed at terminal. | The cardholder should contact their issuer for clarification and can either try again after resolving the issue. |

## createPaymentSession (One-time Payments)

Source: `https://docs.antom.com/ac/ams/session_cashier.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| SUCCESS | S | Success | The payment session is successfully created. No further action is needed. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PROCESS_FAIL | F | A general business failure occurred. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| NO_PAY_OPTIONS | F | No payment options are available. | Contact Antom Technical Support for detailed reasons. |
| CARD_NOT_SUPPORTED | F | The card used for the transaction is not supported. | Use another card to pay the transaction. |
| UNKNOWN_EXCEPTION | U | An API call has failed, which is caused by unknown reasons. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |

## consult (One-time Payments)

Source: `https://docs.antom.com/ac/ams/consult.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| SUCCESS | S | Success | The consultation is successful. No further action is needed. |
| CURRENCY_NOT_SUPPORT | F | The currency is not supported. | Change the currency used in the request. If the issue persists, contact Antom Technical Support for detailed reasons. |
| MERCHANT_NOT_REGISTERED | F | The merchant is not registered. | Please register the merchant by using the registration interface. Contact Antom Technical Support if failed to call the registration interface. |
| NO_PAY_OPTIONS | F | No payment options are available. | Contact Antom Technical Support for detailed reasons. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PROCESS_FAIL | F | A general business failure occurred. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| SETTLE_CONTRACT_NOT_MATCH | F | No matched settlement contract can be found. | Check the following for a solution:   Specify one settlement currency from the multiple currencies that the merchant signed up for.  Check if the settlement currency is specified in the settlement contracts.  The mercha |
| UNKNOWN_EXCEPTION | U | An API call has failed, which is caused by unknown reasons. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |

## inquiryPayment

Source: `https://docs.antom.com/ac/ams/paymentri_online.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| USER_NOT_SUBMITTED | F | Payment timeout due to user's failure to submit. When using the landing page/SDK integration, a timeout will be triggered if the user fails to submit after launching the payment details fill-in page. | Re-initiate a payment. |
| USER_AUTHENTICATION_NOT_FINISHED | F | User authentication failed due to a timeout.  This error code is returned when authentication is not completed after the 3D Secure page is launched during payment. | Re-initiate a payment. |
| PICKUP_CARD | F | The card issuer requests that the merchant retain the card.  This typically arises in cases of suspected counterfeit or stolen cards. | Contact the issuer immediately or use another card to pay the transaction. |
| DOMESTIC_DEBIT_TRANSACTION_NOT_ALLOWED | F | Domestic debit transaction not allowed. (Regional use only) | Contact the issuer immediately or use another card to pay the transaction. |
| BLOCKED_BY_CARDHOLDER | F | This card has been locked by the user. | Contact the issuer immediately or use another card to pay the transaction. |
| RESTRICTED_CARD | F | The card issuer has imposed restrictions on where the card can be used. | Contact the issuer immediately or use another card to pay the transaction. |
| CARD_EXPIRED | F | Card has expired. | Check whether the value of paymentMethod.paymentMethodMetaData.expiryYear or paymentMethod.paymentMethodMetaData.expiryMonth is correct. If not correct, pass in the correct value. If correct, contact Antom Technical Supp... |
| ACCOUNT_CLOSED | F | The account is closed.  Please verify the account number for accuracy and do not attempt to process the same PAN or token again. | Contact the issuer immediately or use another card to pay the transaction. |
| INVALID_CVV | F | The CVV code is invalid. | Prompt the user to check if the entered CVV is correct. |
| INSTALLMENT_NOT_SUPPORTED | F | This card or merchant does not support installments. | This card does not support installment, please change a card to pay. |
| PAYMENT_METHOD_SYSTEM_ERROR | F | Payment method system error. | Issuer system error, please contact the issuer. |
| INVALID_INSTALLMENT_PLAN | F | The number or the interval of installment plans is incorrect. | Please adjust the number of installments and try again. |
| SUCCESS | S | Success | The API is called successfully. Obtain the order status by paymentStatus. |
| ACCESS_DENIED | F | Access is denied. | Contact Antom Technical Support for detailed reasons. |
| INVALID_API | F | The called API is invalid or not active. | Contact Antom Technical Support to resolve the issue. |
| KEY_NOT_FOUND | F | The private key or public key of Antomy or the merchant is not found. | Check whether the private key or public key exists. If not, upload the private key in Antom Developer Center. |
| NO_INTERFACE_DEF | F | API is not defined. | Check whether the URL is correct. Please refer to the endpoint in the API documentation. |
| ORDER_NOT_EXIST | F | The order does not exist. | Check whether paymentId is correct. If correct, contact Antom Technical Support for specific reasons. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PAYMENT_IN_PROCESS | U | The payment is being processed. | For One-time Payments, the request you initiated has the same paymentRequestId as that of the existed transaction, which might be a successful or in-process transaction. Check whether redirectActionForm.redirectUrl is re... |
| PROCESS_FAIL | F | A general business failure occurred. | It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| SYSTEM_ERROR | F | A system error occurred. | Do not retry, and contact Antom Technical Support for more details. |
| REQUEST_TRAFFIC_EXCEED_LIMIT | U | The request traffic exceeds the limit. | Call the API again to resolve the issue. If not resolved, contact Antom Technical Support. |
| UNKNOWN_EXCEPTION | U | An API call has failed, which is caused by unknown reasons. | Call the API again to resolve the issue. If not resolved, contact Antom Technical Support. |
| INVALID_CPF | F | The provided CPF number is invalid. | Please confirm your identification details and try again. |
| REFERRAL_ISSUER | F | The transaction was declined by the card issuer. | The cardholder should contact their issuer for clarification and retry after resolution. |
| TRANSACTION_NOT_PERMITTED | F | The card issuer declined the transaction on this card/account. | Use an alternative payment method or contact the issuer. |
| INVALID_PIN | F | The PIN is incorrect or invalid. | Ensure the correct PIN is entered with valid format/length. |
| LIFECYCLE | F | Declined due to invalid card data. | Verify card details (number/expiry/CVV). |
| LAW_VIOLATION | F | The issuing bank declined the transaction. | Contact the issuer for details. |
| PIN_REQURED | F | PIN is mandatory but not provided. | Reattempt with a valid PIN. |
| LOST_CARD | F | Lost card, pick up card (fraud account). | Contact the issuer immediately. |
| STOLEN_CARD | F | Stolen card, pick up (fraud account). | Contact the issuer immediately. |
| SECURITY_REJECT | F | The transaction was declined due to a security issue detected by the card issuer. | Verify Information: Ensure that all card details, including the CVV/CVC and billing address, are entered correctly.Contact Issuer: Reach out to the bank or financial institution that issued th |
| BLOCKED_FIRST_USED | F | When a card is either temporarily blocked, has not been activated for first use, or is subject to special conditions that prevent processing (e.g., new cardholder not activated). | Activate the card or check with the issuer. |
| CANNOT_VERIFY_PIN | F | PIN verification failed. | Retry without PIN or contact issuer. |
| SURCHARGE_NOT_ALLOWED | F | Surcharge not permitted on this card. | Contact the issuer. |
| EXCEEDS_PREAUTHORIZED_AMOUNT_LIMIT | F | Transaction exceeds preauthorized approval limit. | Reduce amount or request higher limit. |
| STOP_RECURRING_PAYMENT | F | The buyer has requested to cancel or stop their subscription. | The cardholder should contact their issuer for clarification and can either try again after resolving the issue. |
| POLICY | F | Policy restrictions. | The cardholder should contact their issuer for clarification. |
| INVALID_TRANSACTION | F | Invalid transaction. | Do not retry, and contact Antom Technical Support for more details. |
| RE_ENTER_TRANSACTION | F | Transaction temporarily cannot be processed. | The cardholder should contact their issuer for clarification and may attempt again once the issue is resolved. |
| TRANSACTION_NOT_ALLOWED_AT_TERMINAL | F | Transaction is not allowed at terminal. | The cardholder should contact their issuer for clarification and can either try again after resolving the issue. |
| INVALID_MCC | F | Invalid MCC. | Do not retry, and contact Antom Technical Support for more details. |
| NO_SELECTED_ACCOUNT | F | Used when the cardholder has selected specific account but does not have. | May reattempt using correct account type. |
| INVALID_ISSUER | F | It is not possible to reach out to the shopper's issuer for transaction authorization. | Do not retry. Please contract your issuer to confirm the card status. |
| SUCCESS | S | Success | The payment is successful, no further action is needed. |
| ACCESS_DENIED | F | Access is denied. | Contact Antom Technical Support for detailed reasons. |
| CURRENCY_NOT_SUPPORT | F | The currency is not supported. | Contact Antom Technical Support for detailed reasons. |
| FRAUD_REJECT | F | The transaction cannot be further processed because of risk control. If the user has already paid for the transaction, the transaction will be refunded. | Contact Antom Technical Support when one of the following conditions is met:  You want to make an appeal.  The user does no |
| INVALID_API | F | The called API is invalid or not active. | Contact Antom Technical Support to resolve the issue. |
| INVALID_CARD | F | The card is invalid. Maybe the credit card number cannot be identified, the card has no corresponding issuing bank, or the card number is in the wrong format. | Use a new card to initiate a payment or contact the issuing bank. |
| INVALID_EXPIRY_DATE_FORMAT | F | The format of expiryYear or expiryMonth is wrong. | Check the format of the passed parameters expiryYear and expiryMonth. |
| ISSUER_REJECTS_TRANSACTION | F | The issuing bank rejects the transaction. | Use a new card to initiate a payment or contact the issuing bank. |
| INVALID_ACCESS_TOKEN | F | The access token is expired, revoked, or does not exist. | Check whether the accessToken is expired, revoked, or does not exist. Re-sign the contract and re-initiate the authorization signing process. |
| INVALID_MERCHANT_STATUS | F | The merchant status is abnormal because restrictions exist. | Contact Antom Technical Support for detailed reasons. |
| KEY_NOT_FOUND | F | The private key or public key of Antom or the merchant is not found. | Check whether the private key or public key exists. If not, upload the private key in Antom Developer Center. |
| MERCHANT_KYB_NOT_QUALIFIED | F | The payment failed because of the merchant's KYB status. The merchant is either not KYB compliant, or the KYB status is not qualified for this transaction. | Contact Antom Technical Support for detailed reasons. |
| NO_INTERFACE_DEF | F | API is not defined. | Check whether the URL is correct. Please refer to the endpoint in the API documentation. |
| NO_PAY_OPTIONS | F | No payment options are available. | Contact Antom Technical Support for detailed reasons. |
| ORDER_IS_CLOSED | F | The request you initiated has the same paymentRequestId as that of the existed transaction, which is closed. | Use a new paymentRequestId to initiate the payment again. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PAYMENT_AMOUNT_EXCEED_LIMIT | F | The payment amount is greater than the maximum amount allowed by the contract or payment method. | Check whether the payment amount exceeds the limit or use a lower amount and try again. Contact Antom technical support to know the specific limitation. |
| PAYMENT_COUNT_EXCEED_LIMIT | F | The maximum number of payments exceeds the limit that is specified by the payment method. | Contact Antom Technical Support to know the specific limitation. |
| PAYMENT_NOT_QUALIFIED | F | The merchant is not qualified to pay because the merchant is not registered, does not have a contract for Tokenized Payment, or is forbidden to make a payment. | Contact Antom Technical Support for detailed reasons. |
| PROCESS_FAIL | F | A general business failure occurred. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| RISK_REJECT | F | The transaction cannot be further processed because of risk control. If the user has already paid for the transaction, the transaction will be refunded. | If the user does not receive the refund within two weeks, contact Antom Technical Support. |
| SUSPECTED_CARD | F | The card is suspected of fraud. For example, the card is stolen or restricted. | Use a new card to initiate a payment or contact the issuing bank. |
| SUSPECTED_RISK | F | The transaction cannot be further processed because of suspected security issues. You can retry the transaction after one working day.  If the transaction is not secure and the user has already paid, the transaction will... | Contact Antom Technical Support when one |
| SYSTEM_ERROR | F | A system error occurred. | Do not retry, and contact Antom Technical Support for more details. |
| USER_AMOUNT_EXCEED_LIMIT | F | The payment amount exceeds the user payment limit. | Create a new payment by using an amount less than or equal to the account's available balance, or contact Antom Technical Support. |
| USER_BALANCE_NOT_ENOUGH | F | The payment cannot be completed because the user balance in the corresponding payment method is not enough. | Please top up the account or choose other payment methods. |
| USER_KYC_NOT_QUALIFIED | F | The payment failed because of the user's KYC status. The user is either not KYC compliant, or the KYC status is not qualified for this transaction (for example, limitations on the payment amount or product information). | Complete the KYC verification first. |
| USER_PAYMENT_VERIFICATION_FAILED | F | The user is restricted from payment on the payment method side. | Contact Antom Technical Support to know the specific reasons. |
| USER_STATUS_ABNORMAL | F | The user status is abnormal on the payment method side. | Contact Antom Technical Support to know the specific reasons. |
| PAYMENT_IN_PROCESS | U | The payment is being processed. | For One-time Payments, the request you initiated has the same paymentRequestId as that of the existed transaction, which might be a successful or in-process transaction. Check whether redirectActionForm.redirectUrl is re... |
| UNKNOWN_EXCEPTION | U | An API call has failed, which is caused by unknown reasons. | For One-time Payments, call the pay API again with a new paymentRequestId to resolve the issue. If not resolved, contact Antom Technical Support.For Tokenized Payment, call the pay API again or call the inquiryPa |
| CARD_NOT_SUPPORTED | F | The card used for the transaction is not supported. | Use another card to pay the transaction. |
| INVALID_EXPIRATION_DATE | F | The value of paymentMethod.paymentMethodMetaData.expiryYear or paymentMethod.paymentMethodMetaData.expiryDate is invalid. | Check whether the value of paymentMethod.paymentMethodMetaData.expiryYear or paymentMethod.paymentMethodMetaData.expiryDate is correct:  If not |
| INVALID_CARD_NUMBER | F | The number of the card used for the transaction is invalid. | Check whether the value of paymentMethod.paymentMethodMetaData.cardNo specified in the pay (One-time Payments) API is correct:If not correct, pass in the correct value and try again. If correct, this card is n |
| DO_NOT_HONOR | F | The payment is declined by the issuing bank. | Retry the payment using a different card or contact the issuing bank. |
| SUCCESS | S | Success | The API is called successfully. Obtain the refund status from refundStatus. |
| ACCESS_DENIED | F | Access is denied. | Contact Antom Technical Support for detailed reasons. |
| INVALID_API | F | The called API is invalid or not active. | Contact Antom Technical Support to resolve the issue. |
| CLIENT_INVALID | F | The client ID is invalid. Antom has restrictions on client ID. | Check whether the client ID is correct, or contact Antom Technical Support for detailed reasons. |
| KEY_NOT_FOUND | F | The private key or public key of Antom or the merchant is not found. | Check whether the private key or public key exists. If not, upload the private key in Antom Developer Center. |
| NO_INTERFACE_DEF | F | API is not defined. | Check whether the URL is correct. Please refer to the endpoint in the API documentation. |
| ORDER_NOT_EXIST | F | The order does not exist. | Call the API again after 15 seconds. If no results are returned after three retries, the order has not been placed. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PROCESS_FAIL | F | A general business failure occurred. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| SYSTEM_ERROR | F | A system error occurred. | Do not retry, and contact Antom Technical Support for more details. |
| REQUEST_TRAFFIC_EXCEED_LIMIT | U | The request traffic exceeds the limit. | Call the API again to resolve the issue. If not resolved, contact Antom Technical Support. |
| UNKNOWN_EXCEPTION | U | An API call has failed, which is caused by unknown reasons. | Call the API again to resolve the issue. If not resolved, contact Antom Technical Support. |
| PAYMENT_IN_PROCESS | U | The payment is being processed. | For One-time Payments, the request you initiated has the same paymentRequestId as that of the existed transaction, which might be a successful or in-process transaction. Check whether redirectActionForm.redirectUrl is re... |
| AUTHENTICATION_REQUIRED | F | 3D Secure authentication is required. | Reinitiate the payment and redirect the user to perform 3D Secure authentication. |
| PAYMENT_PROHIBITED | F | The payment cannot be processed because the goods are prohibited from sale in the country. | You are not allowed to appeal against this transaction. |
| INVALID_AMOUNT | F | The transaction was declined by the issuing bank due to various reasons. For example, the specified amount is invalid or exceeds the maximum amount limit. | Contact Antom Technical Support for detailed reasons. |

## notifyPayment (result codes in notifications)

Source: `https://docs.antom.com/ac/ams/paymentrn_online.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| USER_NOT_SUBMITTED | F | Payment timeout due to user's failure to submit. When using the landing page/SDK integration, a timeout will be triggered if the user fails to submit after launching the payment details fill-in page. | Re-initiate a payment. |
| USER_AUTHENTICATION_NOT_FINISHED | F | User authentication failed due to a timeout.  This error code is returned when authentication is not completed after the 3D Secure page is launched during payment. | Re-initiate a payment. |
| PICKUP_CARD | F | The card issuer requests that the merchant retain the card.  This typically arises in cases of suspected counterfeit or stolen cards. | Contact the issuer immediately or use another card to pay the transaction. |
| DOMESTIC_DEBIT_TRANSACTION_NOT_ALLOWED | F | Domestic debit transaction not allowed. (Regional use only) | Contact the issuer immediately or use another card to pay the transaction. |
| BLOCKED_BY_CARDHOLDER | F | This card has been locked by the user. | Contact the issuer immediately or use another card to pay the transaction. |
| RESTRICTED_CARD | F | The card issuer has imposed restrictions on where the card can be used. | Contact the issuer immediately or use another card to pay the transaction. |
| CARD_EXPIRED | F | Card has expired. | Check whether the value of paymentMethod.paymentMethodMetaData.expiryYear or paymentMethod.paymentMethodMetaData.expiryMonth is correct. If not correct, pass in the correct value. If correct, contact Antom Technical Supp... |
| ACCOUNT_CLOSED | F | The account is closed.  Please verify the account number for accuracy and do not attempt to process the same PAN or token again. | Contact the issuer immediately or use another card to pay the transaction. |
| INVALID_CVV | F | The CVV code is invalid. | Prompt the user to check if the entered CVV is correct. |
| INSTALLMENT_NOT_SUPPORTED | F | This card or merchant does not support installments. | This card does not support installment, please change a card to pay. |
| PAYMENT_METHOD_SYSTEM_ERROR | F | Payment method system error. | Issuer system error, please contact the issuer. |
| INVALID_INSTALLMENT_PLAN | F | The number or the interval of installment plans is incorrect. | Please adjust the number of installments and try again. |
| SUCCESS | S | Success | The payment is successful, no further action is needed. |
| ACCESS_DENIED | F | Access is denied. | Contact Antom Technical Support for detailed reasons. |
| INVALID_API | F | The called API is invalid or not active. | Contact Antom Technical Support to resolve the issue. |
| CURRENCY_NOT_SUPPORT | F | The currency is not supported. | Contact Antom Technical Support for detailed reasons. |
| EXPIRED_CODE | F | The payment code is expired. | The user needs to refresh the payment code. |
| FRAUD_REJECT | F | The transaction cannot be further processed because of risk control. If the user has already paid for the transaction, the transaction will be refunded. | Contact Antom Technical Support when one of the following conditions is met:  You want to make an appeal.  The user does no |
| INVALID_ACCESS_TOKEN | F | The access token is expired, revoked, or does not exist. | Check whether the accessToken is expired, revoked, or does not exist. Re-sign the contract and re-initiate the authorization signing process. |
| INVALID_CONTRACT | F | The parameter values in the contract do not match those in the current transaction. | Check whether the parameter values in the contract match those in the current transaction. If the values match, contact Antom Technical Support to troubleshoot the issue. |
| INVALID_MERCHANT_STATUS | F | The merchant status is abnormal because restrictions exist. | Contact Antom Technical Support for detailed reasons. |
| KEY_NOT_FOUND | F | The private key or public key of Antom or the merchant is not found. | Check whether the private key or public key exists. If not, upload the private key in Antom Developer Center. |
| MERCHANT_BALANCE_NOT_ENOUGH | F | The merchant balance is not enough. | Call the interface again after the merchant has sufficient balance. |
| NO_INTERFACE_DEF | F | API is not defined. | Check whether the URL is correct. Please refer to the endpoint in the API documentation. |
| NO_PAY_OPTIONS | F | No payment methods are available. | Contact Antom Technical Support for detailed reasons. |
| ORDER_IS_CLOSED | F | The transaction is closed and cannot be paid again. | Use a new paymentRequestId to initiate a payment. |
| ORDER_NOT_EXIST | F | The order does not exist. | Check whether paymentId is correct. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PAYMENT_AMOUNT_EXCEED_LIMIT | F | The payment amount is greater than the maximum amount allowed by the contract or payment method. | Check whether the payment amount exceeds the limit or use a lower amount and try again. Contact Antom technical support to know the specific limitation. |
| PAYMENT_COUNT_EXCEED_LIMIT | F | The maximum number of payments exceeds the limit that is specified by the payment method. | Contact Antom Technical Support to know the specific limitation. |
| PAYMENT_NOT_QUALIFIED | F | The merchant is not qualified to pay because the merchant is not registered, does not have a contract for Tokenized Payment, or is forbidden to make a payment. | Contact Antom Technical Support for detailed reasons. |
| PROCESS_FAIL | F | A general business failure occurred. | It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| RISK_REJECT | F | The transaction cannot be further processed because of risk control. If the user has already paid for the transaction, the transaction will be refunded. | If the user does not receive the refund within two weeks, contact Antom Technical Support. |
| SUSPECTED_RISK | F | The transaction cannot be further processed because of suspected security issues. You can retry the transaction after one working day.  If the transaction is not secure and the user has already paid, the transaction will... | Contact Antom Technical Support when one |
| SYSTEM_ERROR | F | A system error occurred. | Do not retry, and contact Antom Technical Support for more details. |
| USER_AMOUNT_EXCEED_LIMIT | F | The payment amount exceeds the user payment limit. | Create a new payment by using an amount less than or equal to the account's available balance, or contact Antom Technical Support. |
| USER_BALANCE_NOT_ENOUGH | F | The payment cannot be completed because the user balance in the corresponding payment method is insufficient. | Please top up the account or choose other payment methods. |
| USER_KYC_NOT_QUALIFIED | F | The payment failed because of the user's KYC status. The user is either not KYC compliant, or the KYC status is not qualified for this transaction (for example, limitations on the payment amount or product information). | Complete the KYC verification first. |
| USER_NOT_EXIST | F | The user does not exist on the wallet side. | Contact Antom Technical Support for detailed reasons. |
| USER_PAYMENT_VERIFICATION_FAILED | F | The user is restricted from payment on the payment method side. | Contact Antom Technical Support to know the specific reasons. |
| USER_STATUS_ABNORMAL | F | The user status is abnormal on the payment method side. | Contact Antom Technical Support to know the specific reasons. |
| VERIFY_TIMES_EXCEED_LIMIT | F | The current verification code failed to pass the payment verification too many times. | The user must get a new verification code. |
| VERIFY_UNMATCHED | F | The verification code is invalid. | The user must get a new verification code. |
| AUTHENTICATION_REQUIRED | F | 3D Secure authentication is required. | Reinitiate the payment and redirect the user to perform 3D Secure authentication. |
| PAYMENT_PROHIBITED | F | The payment cannot be processed because the goods are prohibited from sale in the country. | You are not allowed to appeal against this transaction. |
| CARD_NOT_SUPPORTED | F | The card used for the transaction is not supported. | Use another card to pay the transaction. |
| INVALID_EXPIRATION_DATE | F | The value of paymentMethod.paymentMethodMetaData.expiryYear or paymentMethod.paymentMethodMetaData.expiryDate is invalid. | Check whether the value of paymentMethod.paymentMethodMetaData.expiryYear or paymentMethod.paymentMethodMetaData.expiryDate is correct:  If not |
| INVALID_CARD_NUMBER | F | The number of the card used for the transaction is invalid. | Check whether the value of paymentMethod.paymentMethodMetaData.cardno is correct:  If not correct, pass in the correct value and try again.  If correct, this card is not supported by the transaction, use anoth |
| DO_NOT_HONOR | F | The payment is declined by the issuing bank. | Retry the payment using a different card or contact the issuing bank. |
| INVALID_AMOUNT | F | The transaction was declined by the issuing bank due to various reasons. For example, the specified amount is invalid or exceeds the maximum amount limit. | Contact Antom Technical Support for detailed reasons. |
| INVALID_CPF | F | The provided CPF number is invalid. | Please confirm your identification details and try again. |
| REFERRAL_ISSUER | F | The transaction was declined by the card issuer. | The cardholder should contact their issuer for clarification and retry after resolution. |
| TRANSACTION_NOT_PERMITTED | F | The card issuer declined the transaction on this card/account. | Use an alternative payment method or contact the issuer. |
| LIFECYCLE | F | Declined due to invalid card data. | Verify card details (number/expiry/CVV). |
| LAW_VIOLATION | F | The issuing bank declined the transaction. | Contact the issuer for details. |
| PIN_REQURED | F | PIN is mandatory but not provided. | Reattempt with a valid PIN. |
| LOST_CARD | F | Lost card, pick up card (fraud account). | Contact the issuer immediately. |
| STOLEN_CARD | F | Stolen card, pick up (fraud account). | Contact the issuer immediately. |
| SECURITY_REJECT | F | The transaction was declined due to a security issue detected by the card issuer. | Verify Information: Ensure that all card details, including the CVV/CVC and billing address, are entered correctly.Contact Issuer: Reach out to the bank or financial institution that issued th |
| BLOCKED_FIRST_USED | F | When a card is either temporarily blocked, has not been activated for first use, or is subject to special conditions that prevent processing (e.g., new cardholder not activated). | Activate the card or check with the issuer. |
| CANNOT_VERIFY_PIN | F | PIN verification failed. | Retry without PIN or contact issuer. |
| SURCHARGE_NOT_ALLOWED | F | Surcharge not permitted on this card. | Contact the issuer. |
| EXCEEDS_PREAUTHORIZED_AMOUNT_LIMIT | F | Transaction exceeds preauthorized approval limit. | Reduce amount or request higher limit. |
| STOP_RECURRING_PAYMENT | F | The buyer has requested to cancel or stop their subscription. | The cardholder should contact their issuer for clarification and can either try again after resolving the issue. |
| POLICY | F | Policy restrictions. | The cardholder should contact their issuer for clarification. |
| INVALID_TRANSACTION | F | Invalid transaction. | Do not retry, and contact Antom Technical Support for more details. |
| RE_ENTER_TRANSACTION | F | Transaction temporarily cannot be processed. | The cardholder should contact their issuer for clarification and may attempt again once the issue is resolved. |
| TRANSACTION_NOT_ALLOWED_AT_TERMINAL | F | Transaction is not allowed at terminal. | The cardholder should contact their issuer for clarification and can either try again after resolving the issue. |
| INVALID_MCC | F | Invalid MCC. | Do not retry, and contact Antom Technical Support for more details. |
| NO_SELECTED_ACCOUNT | F | Used when the cardholder has selected specific account but does not have. | May reattempt using correct account type. |
| INVALID_ISSUER | F | It is not possible to reach out to the buyer's issuer for transaction authorization. | Do not retry. Please contract your issuer to confirm the card status. |

## capture (One-time Payments)

Source: `https://docs.antom.com/ac/ams/capture.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| SUCCESS | S | Success | The capture is successful, no further action is needed. |
| ACCESS_DENIED | F | The access is denied. | Contact Antom Technical Support for detailed reasons. |
| AUTH_CANCELLED | F | The payment authorization is canceled. | Use a new paymentRequestId to initiate a payment. |
| AUTH_EXPIRED | F | The payment authorization is expired. | Use a new paymentRequestId to initiate a payment. |
| AUTH_NOT_FOUND | F | The payment authorization is not found. | Contact Antom Technical Support for detailed reasons. |
| CAPTURE_AMOUNT_EXCEED_AUTH_LIMIT | F | The total capture amount exceeds the limit of the authorized payment amount. | Create a new capture request by using an amount less than or equal to the authorized payment amount, or contact Antom Technical Support. |
| CAPTURE_IN_PROCESS | U | The capture is processing. | Wait for the notification from Antom or inquiry about the capture result. |
| CURRENCY_NOT_SUPPORT | F | The currency is not supported. | Check the currency used in the request, such as the capture currency. If the issue persists, contact Antom Technical Support for detailed reasons. |
| MULTI_CAPTURE_NOT_SUPPORTED | F | The transaction does not support multiple captures. | Contact Antom Technical Support for detailed reasons. |
| INVALID_CONTRACT | F | The parameter values in the contract do not match those in the current transaction. | Check whether the parameter values in the contract match those in the current transaction. If the values match, contact Antom Technical Support to troubleshoot the issue. |
| NO_PAY_OPTIONS | F | No payment methods are available. | Contact Antom Technical Support for detailed reasons. |
| ORDER_IS_CANCELED | F | The transaction is canceled. | Use a new paymentRequestId to initiate a payment. |
| ORDER_STATUS_INVALID | F | The transaction status is abnormal. The payment cannot be captured. | Check the transaction status.  If the values match, contact Antom Technical Support to troubleshoot the issue. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PAYMENT_COUNT_EXCEED_LIMIT | F | The number of captures exceeds the limit that is specified by the payment method. | Contact Antom Technical Support to know the specific limit. |
| PROCESS_FAIL | F | The capture failed. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| REPEAT_REQ_INCONSISTENT | F | The submitted request exists, and the parameter values of this request are not consistent with those of the existing request. | Ensure all the fields in the requests are the same or use a new paymentRequestId to initiate a payment. |
| RISK_REJECT | F | The request is rejected because of risk control. | Prompt the user that the request is rejected because of failed risk control. |
| USER_AMOUNT_EXCEED_LIMIT | F | The capture amount exceeds the user payment limit. | Create a new capture by using an amount less than or equal to the user payment limit, or contact the user or the issuing bank. |
| USER_BALANCE_NOT_ENOUGH | F | The user's balance is insufficient for the capture. | Contact Antom Technical Support for detailed reasons. |
| USER_NOT_EXIST | F | The user's account does not exist on the payment method side. | Contact the user or the issuing bank. |
| USER_STATUS_ABNORMAL | F | The user's account status is abnormal on the payment method side. | Contact Antom Technical Support to know the specific reasons. |
| UNKNOWN_EXCEPTION | U | The API call has failed, which is caused by unknown reasons. | Call the interface again to resolve the issue. If the problem persists, contact Antom Technical Support. |

## notifyCapture

Source: `https://docs.antom.com/ac/ams/notify_capture.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| SUCCESS | S | Success | The capture is successful, no further action is needed. |
| ACCESS_DENIED | F | The access is denied. | Contact Antom Technical Support for detailed reasons. |
| AUTH_CANCELLED | F | The payment authorization is canceled. | Use a new paymentRequestId to initiate a payment. |
| AUTH_EXPIRED | F | The payment authorization is expired. | Use a new paymentRequestId to initiate a payment. |
| AUTH_NOT_FOUND | F | The payment authorization is not found. | Contact Antom Technical Support for detailed reasons. |
| CAPTURE_AMOUNT_EXCEED_AUTH_LIMIT | F | The total capture amount exceeds the limit of the authorized payment amount. | Create a new capture request by using an amount less than or equal to the authorized payment amount, or contact Antom Technical Support. |
| CURRENCY_NOT_SUPPORT | F | The currency is not supported. | Check the currency used in the request, such as the capture currency. If the issue persists, contact Antom Technical Support for detailed reasons. |
| INVALID_CONTRACT | F | The parameter values in the contract do not match those in the current transaction. | Check whether the parameter values in the contract match those in the current transaction. If the values match, contact Antom Technical Support to troubleshoot the issue. |
| MULTI_CAPTURE_NOT_SUPPORTED | F | The transaction does not support multiple captures. | Contact Antom Technical Support for detailed reasons. |
| NO_PAY_OPTIONS | F | No payment methods are available. | Contact Antom Technical Support for detailed reasons. |
| ORDER_IS_CANCELED | F | The transaction is canceled. | Use a new paymentRequestId to initiate a payment. |
| ORDER_STATUS_INVALID | F | The transaction status is abnormal. The payment cannot be captured. | Check the transaction status.  If the values match, contact Antom Technical Support to troubleshoot the issue. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PAYMENT_COUNT_EXCEED_LIMIT | F | The number of captures exceeds the limit that is specified by the payment method. | Contact Antom Technical Support to know the specific limit. |
| PROCESS_FAIL | F | The capture failed. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| REPEAT_REQ_INCONSISTENT | F | The submitted request exists, and the parameter values of this request are not consistent with those of the existing request. | Ensure all the fields in the requests are the same or use a new paymentRequestId to initiate a payment. |
| RISK_REJECT | F | The request is rejected because of risk control. | Prompt the user that the request is rejected because of failed risk control. |
| USER_AMOUNT_EXCEED_LIMIT | F | The capture amount exceeds the user payment limit. | Create a new capture by using an amount less than or equal to the user payment limit, or contact the user or the issuing bank. |
| USER_BALANCE_NOT_ENOUGH | F | The user's balance is insufficient for the capture. | Contact Antom Technical Support for detailed reasons. |
| USER_NOT_EXIST | F | The user's account does not exist on the payment method side. | Contact the user or the issuing bank. |
| USER_STATUS_ABNORMAL | F | The user's account status is abnormal on the payment method side. | Contact Antom Technical Support to know the specific reasons. |

## cancel

Source: `https://docs.antom.com/ac/ams/paymentc_online.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| SUCCESS | S | Success | The order is canceled successfully. |
| ACCESS_DENIED | F | Access is denied. | Contact Antom Technical Support for detailed reasons. |
| INVALID_API | F | The called API is invalid or not active. | Contact Antom Technical Support to resolve the issue. |
| CANCEL_WINDOW_EXCEED | F | The cancellation date exceeds the period during which cancellation is supported. The period is agreed upon in the contract. | The transaction cannot be canceled anymore. |
| KEY_NOT_FOUND | F | The private key or public key of Antom or the merchant is not found. | Check whether the private key or public key exists. If not, upload the private key in Antom Developer Center. |
| MERCHANT_BALANCE_NOT_ENOUGH | F | The merchant balance is insufficient. | Call the interface again after the merchant has sufficient balance. |
| NO_INTERFACE_DEF | F | API is not defined. | Check whether the URL is correct. Please refer to the endpoint in the API documentation. |
| ORDER_NOT_EXIST | F | The order does not exist. | Check whether paymentId is correct. If correct, contact Antom Technical Support for specific reasons. |
| ORDER_STATUS_INVALID | F | The transaction cannot be further processed because the order status is invalid. | Check the order status of the transaction or contact Antom Technical Support for detailed reasons. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PROCESS_FAIL | F | A general business failure occurred. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| SYSTEM_ERROR | F | A system error occurred. | Do not retry, and contact Antom Technical Support for more details. |
| REQUEST_TRAFFIC_EXCEED_LIMIT | U | The request traffic exceeds the limit. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |
| PAYMENT_METHOD_NOT_SUPPORTED | F | The payment method does not support canceling or refunding transactions if the payment status is successful. | Do not retry. Contact Antom Technical Support to get more information. |
| UNKNOWN_EXCEPTION | U | An API call has failed, which is caused by unknown reasons. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |
| USER_NOT_EXIST | F | The user does not exist on the payment method side. | Contact Antom Technical Support for detailed reasons. |
| USER_STATUS_ABNORMAL | F | The user status is abnormal on the payment method side. | Contact Antom Technical Support to know the specific reasons. |

## refund

Source: `https://docs.antom.com/ac/ams/refund_online.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| SUCCESS | S | Success | The refund is successful, no further action is needed. |
| ACCESS_DENIED | F | Access is denied. | Contact Antom Technical Support for detailed reasons. |
| INVALID_API | F | The called API is invalid or not active. | Contact Antom Technical Support to resolve the issue. |
| CURRENCY_NOT_SUPPORT | F | The currency is not supported. | Contact Antom Technical Support for detailed reasons. |
| INVALID_MERCHANT_STATUS | F | The merchant status is abnormal because restrictions exist. | Contact Antom Technical Support for detailed reasons. |
| KEY_NOT_FOUND | F | The private key or public key of Antom or the merchant is not found. | Check whether the private key or public key exists. If not, upload the private key in Antom Dashboard. |
| MERCHANT_BALANCE_NOT_ENOUGH | F | The merchant balance is insufficient. | Call the interface again after the merchant has sufficient balance. The refundRequestId field needs to be changed when you try again. |
| MULTIPLE_REFUNDS_NOT_SUPPORTED | F | Multiple refunds are not supported because restrictions exist in the contract. | Check whether multiple refunds exist. Do not call the refund interface anymore. For each refund, only call the interface once. |
| NO_INTERFACE_DEF | F | API is not defined. | Check whether the URL is correct. Please refer to the endpoint in the API documentation. |
| ORDER_IS_CLOSED | F | The request you initiated has the same paymentRequestId as that of the existed transaction, which is closed. | Use a new paymentRequestId to initiate the refund again. |
| ORDER_NOT_EXIST | F | The order does not exist. | Check whether paymentId is correct. If correct, contact Antom Technical Support for specific reasons. |
| ORDER_STATUS_INVALID | F | The order status is invalid. The transaction is under process or the transaction is failed. | Check the order status and take corresponding actions. Wait if the transaction is under process. Do not initiate the refund if the transaction failed. Contact Antom Technical Su |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PROCESS_FAIL | F | A general business failure occurred. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| REFUND_AMOUNT_EXCEED | F | The total refund amount exceeds the payment amount or the refund amount is less than the minimum refund amount. | Confirm whether the total refund amount exceeds the payment amount or whether the refund amount is less than the minimum refund amount. Create a new refund |
| REFUND_WINDOW_EXCEED | F | The refund date exceeds the refundable period that is agreed in the contract. | Confirm whether the refund date exceeds the refundable period. Check the refundable period in the contract or contact Antom Technical Support for the specific refundable period. |
| REPEAT_REQ_INCONSISTENT | F | The amount or currency is different from the previous request. | Ensure all the fields in the requests are the same or use a new paymentRequestId to initiate the refund again. |
| SYSTEM_ERROR | F | A system error occurred. | Do not retry, and contact Antom Technical Support for more details. |
| REFUND_IN_PROCESS | U | The refund is being processed. | Wait for the asynchronous notification or call the inquiryRefund interface to query the final refund status. Do not retry the refund request. |
| REFUND_NOT_SUPPORT | F | Refunds are not supported for this transaction. | Check the transaction status and the refund policy in the contract. If the transaction is successful and the contract allows refunds, contact Antom Technical Support. |
| REQUEST_TRAFFIC_EXCEED_LIMIT | U | The request traffic exceeds the limit. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |
| PARTIAL_REFUND_NOT_SUPPORTED | F | A partial refund is not supported for this transaction. | Make a full refund or contact the buyer for an offline refund. |
| PAYMENT_METHOD_NOT_SUPPORTED | F | The payment method does not support canceling or refunding transactions if the payment status is successful. | Do not retry. Contact Antom Technical Support to get more information. |
| ORDER_IS_CANCELED | F | The transaction is canceled. | You cannot refund the transaction because it is canceled. |
| REFUND_BY_TRANSACTION_TYPE_INCONSISTENT | F | refund by transaction type inconsistent | Stop this refund immediately, and check whether subsequent refund requests use the same refund dimension as the initial refund, that is, refund consistently by either paymentId or captureId. Correct the request |
| REFUND_AFTER_DISPUTE_NOT_SUPPORTED | F | Refund is not supported after dispute. | A refund cannot be initiated while the chargeback is in progress. Please wait for the chargeback result notification. |
| UNKNOWN_EXCEPTION | U | An API call has failed, which is caused by unknown reasons. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |

## inquiryRefund

Source: `https://docs.antom.com/ac/ams/ir_online.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| SUCCESS | S | Success | The interface is called successfully. Obtain the refund status from refundStatus. |
| ACCESS_DENIED | F | Access is denied. | Contact Antom Technical Support for detailed reasons. |
| INVALID_API | F | The called API is invalid or not active. | Contact Antom Technical Support to resolve the issue. |
| KEY_NOT_FOUND | F | The private key or public key of Antom or the merchant is not found. | Check whether the private key or public key exists. If not, upload the private key in Antom Developer Center. |
| NO_INTERFACE_DEF | F | API is not defined. | Check whether the URL is correct. Please refer to the endpoint in the API documentation. |
| ORDER_NOT_EXIST | F | The order does not exist. | Call the interface again after 15 seconds. If no results are returned after three retries, the order has not been placed. |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PROCESS_FAIL | F | A general business failure occurred. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| SYSTEM_ERROR | F | A system error occurred. | Do not retry, and contact Antom Technical Support for more details. |
| REQUEST_TRAFFIC_EXCEED_LIMIT | U | The request traffic exceeds the limit. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |
| UNKNOWN_EXCEPTION | U | An API call has failed, which is caused by unknown reasons. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |
| SUCCESS | S | Success | The refund is successful, no further action is needed. |
| ACCESS_DENIED | F | Access is denied. | Contact Antom Technical Support for detailed reasons. |
| API_INVALID | F | The API name or API call format is used incorrectly. | Check whether the API name, HTTP method, or request format is correct when sending the request. |
| CURRENCY_NOT_SUPPORT | F | The currency is not supported. | Contact Antom Technical Support for detailed reasons. |
| INVALID_MERCHANT_STATUS | F | The merchant status is abnormal because restrictions exist. | Contact Antom Technical Support for detailed reasons. |
| KEY_NOT_FOUND | F | The private key or public key of Antom or the merchant is not found. | Check whether the private key or public key exists. If not, upload the private key in Antom Developer Center. |
| MERCHANT_BALANCE_NOT_ENOUGH | F | The merchant balance is insufficient. | Call the interface again after the merchant has sufficient balance. The refundRequestId field needs to be changed when you try again. |
| MULTIPLE_REFUNDS_NOT_SUPPORTED | F | Multiple refunds are not supported because restrictions exist in the contract. | Check whether multiple refunds exist. Do not call the refund interface anymore. For each refund, only call the interface once. |
| NO_INTERFACE_DEF | F | API is not defined. | Check whether the URL is correct. Please refer to the endpoint in the API documentation. |
| ORDER_IS_CLOSED | F | The request you initiated has the same paymentRequestId as that of the existed transaction, which is closed. | Use a new paymentRequestId to initiate the refund again. |
| ORDER_NOT_EXIST | F | The order does not exist. | Check whether paymentId is correct. If correct, contact Antom Technical Support for specific reasons. |
| ORDER_STATUS_INVALID | F | The order status is invalid. The transaction is under process or the transaction is failed. | Check the order status and take corresponding actions. Wait if the transaction is under process. Do not initiate the refund if the transaction failed. Contact Antom Technical Su |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PROCESS_FAIL | F | A general business failure occurred. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| REFUND_AMOUNT_EXCEED | F | The total refund amount exceeds the payment amount. | Confirm whether the total refund amount exceeds the payment amount. Create a new refund by using an amount less than or equal to the payment amount, or contact Antom Technical Support. |
| REFUND_WINDOW_EXCEED | F | The refund date exceeds the refundable period that is agreed in the contract. | Confirm whether the refund date exceeds the refundable period. Check the refundable period in the contract or contact Antom Technical Support for the specific refundable period. |
| REPEAT_REQ_INCONSISTENT | F | The amount or currency is different from the previous request. | Ensure all the fields in the requests are the same or use a new paymentRequestId to initiate the refund again. |
| RISK_REJECT | F | The request is rejected because of the risk control. | Prompt the user that the request is rejected because the risk control failed. |
| SYSTEM_ERROR | F | A system error occurred. | Do not retry, and contact Antom Technical Support for more details. |
| REQUEST_TRAFFIC_EXCEED_LIMIT | U | The request traffic exceeds the limit. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |
| UNKNOWN_EXCEPTION | U | An API call has failed, which is caused by unknown reasons. | Call the interface again to resolve the issue. If not resolved, contact Antom Technical Support. |

## notifyRefund

Source: `https://docs.antom.com/ac/ams/notify_refund.md`

| Code | Status | Message | Further action |
| --- | --- | --- | --- |
| SUCCESS | S | Success | The refund is successful, no further action is needed. |
| ACCESS_DENIED | F | Access is denied. | Contact Antom Technical Support for detailed reasons. |
| CLIENT_INVALID | F | The client ID is invalid. Antom has restrictions on client ID. | Check whether the client ID is correct, or contact Antom Technical Support for detailed reasons. |
| CURRENCY_NOT_SUPPORT | F | The currency is not supported. | Contact Antom Technical Support for detailed reasons. |
| INVALID_API | F | The called API is invalid or not active. | Contact Antom Technical Support to resolve the issue. |
| INVALID_CONTRACT | F | The parameter values in the contract do not match those in the current transaction. | Check whether the parameter values in the contract match those in the current transaction. If the values match, contact Antom Technical Support to troubleshoot the issue. |
| INVALID_MERCHANT_STATUS | F | The merchant status is abnormal because restrictions exist. | Contact Antom Technical Support for detailed reasons. |
| INVALID_SIGNATURE | F | The signature is not validated. The private key used to sign a request does not match the public key of Antom Dashboard. | Check whether the private key used to sign a request matches the public key of Antom Dashboard. The following signature references are useful:The signa |
| KEY_NOT_FOUND | F | The private key or public key of Antom or the merchant is not found. | Check whether the private key or public key exists. If not, upload the private key in Antom Dashboard. |
| MEDIA_TYPE_NOT_ACCEPTABLE | F | The server does not implement the media type that is acceptable to the client. | Check whether the media type is correct and use a media type that is accepted by Antom. |
| MERCHANT_BALANCE_NOT_ENOUGH | F | The merchant balance is insufficient. | Call the interface again after the merchant has sufficient balance. The refundRequestId field needs to be changed when you try again. |
| MERCHANT_NOT_REGISTERED | F | The merchant is not registered. | Please register the merchant by using the registration interface. Contact Antom Technical Support if failed to call the registration interface. |
| METHOD_NOT_SUPPORTED | F | The server does not implement the requested HTTP method. Only the POST method is supported. | Ensure the HTTP method is POST. |
| MULTIPLE_REFUNDS_NOT_SUPPORTED | F | Multiple refunds are not supported because restrictions exist in the contract. | Check whether multiple refunds exist. Do not call the refund interface anymore. For each refund, only call the interface once. |
| NO_INTERFACE_DEF | F | API is not defined. | Check whether the URL is correct. Please refer to the endpoint in the API documentation. |
| ORDER_IS_CLOSED | F | The transaction is closed and cannot be paid again. | Use a new paymentRequestId to initiate the refund again. |
| ORDER_NOT_EXIST | F | The order does not exist. | Check whether paymentId is correct. If correct, contact Antom Technical Support for specific reasons. |
| ORDER_STATUS_INVALID | F | The order status is invalid. The transaction is under process or the transaction failed. | Check the order status and take corresponding actions. Wait if the transaction is under process. Do not initiate the refund if the transaction failed. Contact Antom Technical Suppo |
| PARAM_ILLEGAL | F | The required parameters are not passed, or illegal parameters exist. For example, a non-numeric input, an invalid date, or the length and type of the parameter are wrong. | Check and verify whether the required request fields (including the header fields and body fields) of the |
| PROCESS_FAIL | F | A general business failure occurred. | Do not retry. Human intervention is usually needed. It is recommended that you contact Antom Technical Support to troubleshoot the issue. |
| REFUND_AMOUNT_EXCEED | F | The total refund amount exceeds the payment amount or the refund amount is less than the minimum refund amount. | Confirm whether the total refund amount exceeds the payment amount or if the refund amount is below the minimum refund amount. Create a new refund using an a |
| REFUND_WINDOW_EXCEED | F | The refund date exceeds the refundable period that is agreed in the contract. | Confirm whether the refund date exceeds the refundable period. Check the refundable period in the contract or contact Antom Technical Support for the specific refundable period. |
| REPEAT_REQ_INCONSISTENT | F | The amount or currency is different from the previous request. | Ensure all the fields in the requests are the same or use a new paymentRequestId to initiate the refund again. |
| RISK_REJECT | F | The request is rejected because of the risk control. | Prompt the user that the request is rejected because the risk control failed. |
| SYSTEM_ERROR | F | A system error occurred. | Do not retry, and contact Antom Technical Support for more details. |
| REFUND_BY_TRANSACTION_TYPE_INCONSISTENT | F | refund by transaction type inconsistent | Stop this refund immediately, and check whether subsequent refund requests use the same refund dimension as the initial refund, that is, refund consistently by either paymentId or captureId. Correct the request |
| USER_IDENTITY_FROZEN_BY_CHANNEL | F | The user's account has been frozen by the payment method. |  |
