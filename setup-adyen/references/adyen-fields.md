# Adyen Field Reference (generated from official OpenAPI)

Generated on 2026-10-21 from Adyen's official OpenAPI specifications in `https://github.com/Adyen/adyen-openapi` (`json/CheckoutService-v72.json`, `json/Webhooks-v1.json`, `json/TokenizationNotification-v1.json`, `json/DisputeService-v30.json`). The docs site uses Checkout API v72 in its examples.

How to read: `Req` = `yes` when the OpenAPI schema marks the field required. Notes are the first sentence of the official field description, with enum values and length limits from the schema. A type that names another schema (for example `Amount`) has its own table below when it matters for MY payments. Full descriptions: `https://docs.adyen.com/api-explorer/Checkout/latest/overview`.

Rules that the tables do not show (from the docs pages, see `adyen-checkout.md`):

- `amount.value` is an integer in minor units. MYR has 2 decimals, so RM 10.00 is `1000`.
- `reference` max 80 characters on v72; GrabPay needs max 32 characters, `[A-Za-z0-9_-]` only.
- `returnUrl` max 1024 characters, URL-encode non-ASCII, no personal data in it.
- `metadata` on v72: key max 20 characters, value max 80 characters.
- `captureDelayHours` max 672 on v72.

Contents: Sessions, Payment methods, Payments, Payment details, Actions, Modifications (captures, refunds, cancels, reversals, amount updates), Payment links, Stored payment methods (tokens), shared objects, Standard webhook, Tokenization webhook, Disputes API.

## Checkout API v72

### `CreateCheckoutSessionRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `accountInfo` | AccountInfo |  | Shopper account information for 3D Secure 2. |
| `additionalAmount` | Amount |  | If you want a BIN or card verification request to use a non-zero value, assign this value to `additionalAmount` (while the amount must be still set to 0 to trigger BIN or card verification). |
| `additionalData` | object |  | This field contains additional data, which may be required for a particular payment request. |
| `allowedPaymentMethods` | array of string |  | List of payment methods to be presented to the shopper. |
| `amount` | Amount | yes | The amount of the payment. |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `authenticationData` | AuthenticationData |  | Configuration data for 3DS payments. |
| `billingAddress` | BillingAddress |  | The address where to send the invoice. |
| `blockedPaymentMethods` | array of string |  | List of payment methods to be hidden from the shopper. |
| `captureDelayHours` | integer (int32) |  | The delay between the authorisation and scheduled auto-capture, specified in hours. |
| `channel` | string |  | The platform where a payment transaction takes place. Values: `iOS`, `Android`, `Web`. |
| `company` | Company |  | Information regarding the company. |
| `countryCode` | string |  | The shopper's two-letter country code. |
| `dateOfBirth` | string (date) |  | The shopper's date of birth. |
| `deliverAt` | string (date-time) |  | The date and time when the purchased goods should be delivered. |
| `deliveryAddress` | DeliveryAddress |  | The address where the purchased goods should be delivered. |
| `enableOneClick` | boolean |  | When true and `shopperReference` is provided, the shopper will be asked if the payment details should be stored for future one-click payments. |
| `enablePayOut` | boolean |  | When true and `shopperReference` is provided, the payment details will be tokenized for payouts. |
| `enableRecurring` | boolean |  | When true and `shopperReference` is provided, the payment details will be stored for recurring payments where the shopper is not present, such as subscription or automatic top-up payments. |
| `enhancedSchemeData` | EnhancedSchemeData |  | Enhanced scheme data that may be required for processing the payment and/or interchange savings can apply. |
| `expiresAt` | string (date-time) |  | The date the session expires in ISO8601 format. |
| `fundOrigin` | FundOrigin |  | The person or entity funding the money. |
| `fundRecipient` | FundRecipient |  | the person or entity receiving the money |
| `installmentOptions` | object |  | A set of key-value pairs that specifies the installment options available per payment method. |
| `lineItems` | array of LineItem |  | Price and product information about the purchased items, to be included on the invoice sent to the shopper. |
| `mandate` | Mandate |  | The mandate details to initiate recurring transaction. |
| `mcc` | string |  | The merchant category code (MCC) is a four-digit number, which relates to a particular market segment. |
| `merchantAccount` | string | yes | The merchant account identifier, with which you want to process the transaction. |
| `merchantOrderReference` | string |  | This reference allows linking multiple transactions to each other for reporting purposes (i.e. |
| `metadata` | object |  | Metadata consists of entries, each of which includes a key and a value. |
| `mode` | string |  | Indicates the type of front end integration. Values: `embedded`, `hosted`. |
| `mpiData` | ThreeDSecureData |  | Authentication data produced by an MPI (Mastercard SecureCode, Visa Secure, or Cartes Bancaires). |
| `payable` | boolean |  | Indicates if the session is payable. |
| `platformChargebackLogic` | PlatformChargebackLogic |  | Defines how to book chargebacks when using Adyen for Platforms. |
| `recurringExpiry` | string |  | Date after which no further authorisations shall be performed. |
| `recurringFrequency` | string |  | Minimum number of days between authorisations. |
| `recurringProcessingModel` | string |  | Defines a recurring payment type. Values: `CardOnFile`, `Subscription`, `UnscheduledCardOnFile`. |
| `redirectFromIssuerMethod` | string |  | Specifies the redirect method (GET or POST) when redirecting back from the issuer. |
| `redirectToIssuerMethod` | string |  | Specifies the redirect method (GET or POST) when redirecting to the issuer. |
| `reference` | string | yes | The reference to uniquely identify a payment. |
| `returnUrl` | string | yes | The URL to return to in case of a redirection. (maxLength 1024) |
| `riskData` | RiskData |  | Any risk-related settings to apply to the payment. |
| `shopperConversionId` | string |  | Use this if you made a `/paymentMethods` request to get the payment methods for the shopper's checkout session. (maxLength 256) |
| `shopperEmail` | string |  | The shopper's email address. |
| `shopperIP` | string |  | The shopper's IP address. |
| `shopperInteraction` | string |  | Specifies the sales channel, through which the shopper gives their card details, and whether the shopper is a returning customer. Values: `Ecommerce`, `ContAuth`, `Moto`, `POS`. |
| `shopperLocale` | string |  | The language for the payment. |
| `shopperName` | ShopperName |  | The shopper's full name. |
| `shopperReference` | string |  | Your reference to uniquely identify this shopper, for example user ID or account ID. (maxLength 256, minLength 3) |
| `shopperStatement` | string |  | The text to be shown on the shopper's bank statement. |
| `showInstallmentAmount` | boolean |  | Set to true to show the payment amount per installment. |
| `showRemovePaymentMethodButton` | boolean |  | Set to **true** to show a button that lets the shopper remove a stored payment method. |
| `socialSecurityNumber` | string |  | The shopper's social security number. |
| `splitCardFundingSources` | boolean |  | Boolean value indicating whether the card payment method should be split into separate debit and credit options. |
| `splits` | array of Split |  | An array of objects specifying how to split a payment when using Adyen for Platforms, Classic Platforms integration, or Issuing. |
| `store` | string |  | Required for Adyen for Platforms integrations if you are a platform model. |
| `storeFiltrationMode` | string |  | Specifies how payment methods should be filtered based on the 'store' parameter:   - 'exclusive': Only payment methods belonging to the specified 'store' are returned. Values: `exclusive`, `inclusive`, `skipFilter`. |
| `storePaymentMethod` | boolean |  | When true and `shopperReference` is provided, the payment details will be stored for future recurring payments. |
| `storePaymentMethodMode` | string |  | Indicates if the details of the payment method will be stored for the shopper. Values: `askForConsent`, `disabled`, `enabled`. |
| `telephoneNumber` | string |  | The shopper's telephone number. |
| `themeId` | string |  | Sets a custom theme for Hosted Checkout. |
| `thirdPartyTokenRedundancyInfo` | ThirdPartyTokenRedundancyInfo |  | Configuration for creating redundant payment tokens with third-party token vaults using the Adyen Forward API. |
| `threeDS2RequestData` | CheckoutSessionThreeDS2RequestData |  | Request fields for 3D Secure 2. |
| `threeDSAuthenticationOnly` | boolean |  | Deprecated. Required to trigger the authentication-only flow. |
| `trustedShopper` | boolean |  | Set to true if the payment should be routed to a trusted MID. |

### `CreateCheckoutSessionResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `accountInfo` | AccountInfo |  | Shopper account information for 3D Secure 2. |
| `additionalAmount` | Amount |  | If you want a BIN or card verification request to use a non-zero value, assign this value to `additionalAmount` (while the amount must be still set to 0 to trigger BIN or card verification). |
| `additionalData` | object |  | This field contains additional data, which may be required for a particular payment request. |
| `allowedPaymentMethods` | array of string |  | List of payment methods to be presented to the shopper. |
| `amount` | Amount | yes | The amount of the payment. |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `authenticationData` | AuthenticationData |  | Configuration data for 3DS payments. |
| `billingAddress` | BillingAddress |  | The address where to send the invoice. |
| `blockedPaymentMethods` | array of string |  | List of payment methods to be hidden from the shopper. |
| `captureDelayHours` | integer (int32) |  | The delay between the authorisation and scheduled auto-capture, specified in hours. |
| `channel` | string |  | The platform where a payment transaction takes place. Values: `iOS`, `Android`, `Web`. |
| `company` | Company |  | Information regarding the company. |
| `countryCode` | string |  | The shopper's two-letter country code. |
| `dateOfBirth` | string (date-time) |  | The shopper's date of birth in ISO8601 format. |
| `deliverAt` | string (date-time) |  | The date and time when the purchased goods should be delivered. |
| `deliveryAddress` | DeliveryAddress |  | The address where the purchased goods should be delivered. |
| `enableOneClick` | boolean |  | When true and `shopperReference` is provided, the shopper will be asked if the payment details should be stored for future one-click payments. |
| `enablePayOut` | boolean |  | When true and `shopperReference` is provided, the payment details will be tokenized for payouts. |
| `enableRecurring` | boolean |  | When true and `shopperReference` is provided, the payment details will be stored for recurring payments where the shopper is not present, such as subscription or automatic top-up payments. |
| `enhancedSchemeData` | EnhancedSchemeData |  | Enhanced scheme data that may be required for processing the payment and/or interchange savings can apply. |
| `expiresAt` | string (date-time) | yes | The date the session expires in ISO8601 format. |
| `fundOrigin` | FundOrigin |  | The person or entity funding the money. |
| `fundRecipient` | FundRecipient |  | the person or entity receiving the money |
| `id` | string | yes | A unique identifier of the session. |
| `installmentOptions` | object |  | A set of key-value pairs that specifies the installment options available per payment method. |
| `lineItems` | array of LineItem |  | Price and product information about the purchased items, to be included on the invoice sent to the shopper. |
| `mandate` | Mandate |  | The mandate details to initiate recurring transaction. |
| `mcc` | string |  | The merchant category code (MCC) is a four-digit number, which relates to a particular market segment. |
| `merchantAccount` | string | yes | The merchant account identifier, with which you want to process the transaction. |
| `merchantOrderReference` | string |  | This reference allows linking multiple transactions to each other for reporting purposes (i.e. |
| `metadata` | object |  | Metadata consists of entries, each of which includes a key and a value. |
| `mode` | string |  | Indicates the type of front end integration. Values: `embedded`, `hosted`. |
| `mpiData` | ThreeDSecureData |  | Authentication data produced by an MPI (Mastercard SecureCode, Visa Secure, or Cartes Bancaires). |
| `payable` | boolean |  | Indicates if the session is payable. |
| `platformChargebackLogic` | PlatformChargebackLogic |  | Defines how to book chargebacks when using Adyen for Platforms. |
| `recurringExpiry` | string |  | Date after which no further authorisations shall be performed. |
| `recurringFrequency` | string |  | Minimum number of days between authorisations. |
| `recurringProcessingModel` | string |  | Defines a recurring payment type. Values: `CardOnFile`, `Subscription`, `UnscheduledCardOnFile`. |
| `redirectFromIssuerMethod` | string |  | Specifies the redirect method (GET or POST) when redirecting back from the issuer. |
| `redirectToIssuerMethod` | string |  | Specifies the redirect method (GET or POST) when redirecting to the issuer. |
| `reference` | string | yes | The reference to uniquely identify a payment. |
| `returnUrl` | string | yes | The URL to return to in case of a redirection. (maxLength 1024) |
| `riskData` | RiskData |  | Any risk-related settings to apply to the payment. |
| `sessionData` | string |  | The payment session data you need to pass to your front end. |
| `shopperEmail` | string |  | The shopper's email address. |
| `shopperIP` | string |  | The shopper's IP address. |
| `shopperInteraction` | string |  | Specifies the sales channel, through which the shopper gives their card details, and whether the shopper is a returning customer. Values: `Ecommerce`, `ContAuth`, `Moto`, `POS`. |
| `shopperLocale` | string |  | The language for the payment. |
| `shopperName` | ShopperName |  | The shopper's full name. |
| `shopperReference` | string |  | Your reference to uniquely identify this shopper, for example user ID or account ID. (maxLength 256, minLength 3) |
| `shopperStatement` | string |  | The text to be shown on the shopper's bank statement. |
| `showInstallmentAmount` | boolean |  | Set to true to show the payment amount per installment. |
| `showRemovePaymentMethodButton` | boolean |  | Set to **true** to show a button that lets the shopper remove a stored payment method. |
| `socialSecurityNumber` | string |  | The shopper's social security number. |
| `splitCardFundingSources` | boolean |  | Boolean value indicating whether the card payment method should be split into separate debit and credit options. |
| `splits` | array of Split |  | An array of objects specifying how to split a payment when using Adyen for Platforms, Classic Platforms integration, or Issuing. |
| `store` | string |  | Required for Adyen for Platforms integrations if you are a platform model. |
| `storeFiltrationMode` | string |  | Specifies how payment methods should be filtered based on the 'store' parameter:   - 'exclusive': Only payment methods belonging to the specified 'store' are returned. Values: `exclusive`, `inclusive`, `skipFilter`. |
| `storePaymentMethod` | boolean |  | When true and `shopperReference` is provided, the payment details will be stored for future recurring payments. |
| `storePaymentMethodMode` | string |  | Indicates if the details of the payment method will be stored for the shopper. Values: `askForConsent`, `disabled`, `enabled`. |
| `telephoneNumber` | string |  | The shopper's telephone number. |
| `themeId` | string |  | Sets a custom theme for Hosted Checkout. |
| `thirdPartyTokenRedundancyInfo` | ThirdPartyTokenRedundancyInfo |  | Configuration for creating redundant payment tokens with third-party token vaults using the Adyen Forward API. |
| `threeDS2RequestData` | CheckoutSessionThreeDS2RequestData |  | Request fields for 3D Secure 2. |
| `threeDSAuthenticationOnly` | boolean |  | Deprecated. Required to trigger the authentication-only flow. |
| `trustedShopper` | boolean |  | Set to true if the payment should be routed to a trusted MID. |
| `url` | string |  | The URL for the Hosted Checkout page. |

### `SessionResultResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `additionalData` | object |  | Contains additional information about the payment. |
| `id` | string |  | A unique identifier of the session. |
| `payments` | array of Payment |  | A list of all authorised payments done for this session. |
| `reference` | string |  | The unique reference that you provided in the original `/sessions` request. |
| `status` | string |  | The status of the session. Values: `active`, `canceled`, `completed`, `expired`, `paymentPending`, `refused`. |

### `CheckoutSessionPatchSessionRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `amount` | SessionAmountUpdate | yes | The amount to update for the payment session. |
| `payable` | boolean |  | Indicates if the session is payable. |
| `sessionData` | string | yes | The encoded payment session data from the `beforeSubmit` callback from Drop-in or the Component. (minLength 1) |

### `PaymentMethodsRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `additionalData` | object |  | This field contains additional data, which may be required for a particular payment request. |
| `allowedPaymentMethods` | array of string |  | List of payment methods to be presented to the shopper. |
| `amount` | Amount |  | The amount information for the transaction (in minor units). |
| `blockedPaymentMethods` | array of string |  | List of payment methods to be hidden from the shopper. |
| `browserInfo` | BrowserInfo |  | The shopper's browser information. |
| `channel` | string |  | The platform where a payment transaction takes place. Values: `iOS`, `Android`, `Web`. |
| `countryCode` | string |  | The shopper country code. |
| `merchantAccount` | string | yes | The merchant account identifier, with which you want to process the transaction. |
| `order` | EncryptedOrderData |  | The order information required for partial payments. |
| `shopperConversionId` | string |  | A unique ID to connect the shopper to a single checkout session that uses multiple API requests. (maxLength 256) |
| `shopperEmail` | string |  | The shopper's email address. |
| `shopperIP` | string |  | The shopper's IP address. |
| `shopperLocale` | string |  | The language for the payment. |
| `shopperReference` | string |  | Required for recurring payments. |
| `splitCardFundingSources` | boolean |  | Boolean value indicating whether the card payment method should be split into separate debit and credit options. |
| `store` | string |  | Required for Adyen for Platforms integrations if you are a platform model. (maxLength 16, minLength 1) |
| `storeFiltrationMode` | string |  | Specifies how payment methods should be filtered based on the `store` parameter:   - **exclusive**: Only payment methods belonging to the specified `store` are returned. Values: `exclusive`, `inclusive`, `skipFilter`. |
| `telephoneNumber` | string |  | The shopper's telephone number. |

### `PaymentMethodsResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `paymentMethods` | array of PaymentMethod |  | Detailed list of payment methods required to generate payment forms. |
| `storedPaymentMethods` | array of StoredPaymentMethod |  | List of all stored payment methods. |

### `PaymentMethod`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `apps` | array of PaymentMethodUPIApps |  | A list of apps for this payment method. |
| `brand` | string |  | Brand for the selected gift card. |
| `brands` | array of string |  | List of possible brands. |
| `configuration` | object |  | The configuration of the payment method. |
| `fundingSource` | string |  | The funding source of the payment method. Values: `credit`, `debit`, `prepaid`. |
| `group` | PaymentMethodGroup |  | The group where this payment method belongs to. |
| `inputDetails` | array of InputDetail |  | Deprecated. All input details to be provided to complete the payment with this payment method. |
| `issuers` | array of PaymentMethodIssuer |  | A list of issuers for this payment method. |
| `name` | string |  | The displayable name of this payment method. |
| `promoted` | boolean |  | Indicates whether this payment method should be promoted or not. |
| `type` | string |  | The unique payment method code. |

### `StoredPaymentMethod`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `bankAccountNumber` | string |  | The bank account number (without separators). |
| `bankLocationId` | string |  | The location id of the bank. |
| `brand` | string |  | The brand of the card. |
| `cashtag` | string |  | The shopper’s Cash App Pay Cashtag. |
| `expiryMonth` | string |  | The two-digit month when the card expires |
| `expiryYear` | string |  | The last two digits of the year the card expires. |
| `externalToken` | string |  | The token issued by an external tokenization service representing the shopper's payment method |
| `holderName` | string |  | The name of the payment method holder. |
| `iban` | string |  | The IBAN of the bank account. |
| `id` | string |  | A unique identifier of this stored payment method. |
| `label` | string |  | The shopper’s issuer account label |
| `lastFour` | string |  | The last four digits of the PAN. |
| `name` | string |  | The display name of the stored payment method. |
| `networkTxReference` | string |  | Returned in the response if you are not tokenizing with Adyen and are using the Merchant-initiated transactions (MIT) framework from Mastercard or Visa. |
| `ownerName` | string |  | The name of the bank account holder. |
| `shopperEmail` | string |  | The shopper’s email address. |
| `supportedRecurringProcessingModels` | array of string |  | The supported recurring processing models for this stored payment method. |
| `supportedShopperInteractions` | array of string |  | The supported shopper interactions for this stored payment method. |
| `type` | string |  | The type of payment method. |

### `PaymentRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `accountInfo` | AccountInfo |  | Shopper account information for 3D Secure 2. |
| `additionalAmount` | Amount |  | If you want a BIN or card verification request to use a non-zero value, assign this value to `additionalAmount` (while the amount must be still set to 0 to trigger BIN or card verification). |
| `additionalData` | object |  | This field contains additional data, which may be required for a particular payment request. |
| `amount` | Amount | yes | The amount information for the transaction (in minor units). |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `authenticationData` | AuthenticationData |  | Data for 3DS authentication. |
| `bankAccount` | CheckoutBankAccount |  | The details of the bank account, from which the payment should be made. |
| `billingAddress` | BillingAddress |  | The address where to send the invoice. |
| `browserInfo` | BrowserInfo |  | The shopper's browser information. |
| `captureDelayHours` | integer (int32) |  | The delay between the authorization and automatic capture of the payment, specified in hours. |
| `channel` | string |  | The platform where a payment transaction takes place. Values: `iOS`, `Android`, `Web`. |
| `checkoutAttemptId` | string |  | Checkout attempt ID that corresponds to the Id generated by the client SDK for tracking user payment journey. (maxLength 256) |
| `company` | Company |  | Information regarding the company. |
| `countryCode` | string |  | The shopper country code. (maxLength 100) |
| `dateOfBirth` | string (date-time) |  | The shopper's date of birth. |
| `dccQuote` | ForexQuote |  | The forex quote as returned in the response of the forex service. |
| `deliverAt` | string (date-time) |  | The date and time the purchased goods should be delivered. |
| `deliveryAddress` | DeliveryAddress |  | The address where the purchased goods should be delivered. |
| `deliveryDate` | string (date-time) |  | Deprecated. The date and time the purchased goods should be delivered. |
| `deviceFingerprint` | string |  | A string containing the shopper's device fingerprint. (maxLength 5000) |
| `enableOneClick` | boolean |  | When true and `shopperReference` is provided, the shopper will be asked if the payment details should be stored for future one-click payments. |
| `enablePayOut` | boolean |  | When true and `shopperReference` is provided, the payment details will be tokenized for payouts. |
| `enableRecurring` | boolean |  | When true and `shopperReference` is provided, the payment details will be stored for recurring payments where the shopper is not present, such as subscription or automatic top-up payments. |
| `enhancedSchemeData` | EnhancedSchemeData |  | Enhanced scheme data that may be required for processing the payment and/or interchange savings can apply. |
| `entityType` | string |  | The type of the entity the payment is processed for. Values: `NaturalPerson`, `CompanyName`. |
| `fraudOffset` | integer (int32) |  | An integer value that is added to the normal fraud score. |
| `fundOrigin` | FundOrigin |  | The person or entity funding the money. |
| `fundRecipient` | FundRecipient |  | the person or entity receiving the money |
| `industryUsage` | string |  | The reason for the amount update. Values: `delayedCharge`, `installment`, `noShow`. |
| `installments` | Installments |  | Contains installment settings. |
| `lineItems` | array of LineItem |  | Price and product information about the purchased items, to be included on the invoice sent to the shopper. |
| `localizedShopperStatement` | object |  | The `localizedShopperStatement` field lets you use dynamic values for your shopper statement in a local character set. |
| `mandate` | Mandate |  | The mandate details to initiate recurring transaction. |
| `mcc` | string |  | The merchant category code (MCC) is a four-digit number, which relates to a particular market segment. |
| `merchantAccount` | string | yes | The merchant account identifier, with which you want to process the transaction. |
| `merchantOrderReference` | string |  | You can use this reference to link multiple transactions to one another (for example, to track order authorization rate). (maxLength 1000) |
| `merchantRiskIndicator` | MerchantRiskIndicator |  | Additional risk fields for 3D Secure 2. |
| `metadata` | object |  | Metadata consists of entries, each of which includes a key and a value. |
| `mpiData` | ThreeDSecureData |  | Authentication data produced by an MPI (Mastercard SecureCode, Visa Secure, or Cartes Bancaires). |
| `opi` | OpiRequest |  | Configuration for Oracle Payment Interface (OPI) integrations. |
| `order` | EncryptedOrderData |  | The order information required for partial payments. |
| `orderReference` | string |  | When you are doing multiple partial (gift card) payments, this is the `pspReference` of the first payment. |
| `origin` | string |  | Required for browser-based (`channel` **Web**) 3D Secure 2 transactions.Set this to the origin URL of the page where you are rendering the Drop-in/Component. (maxLength 80) |
| `paymentMethod` | one of the *Details objects (for MY: `CardDetails` type `scheme`, `MolPayDetails` type `molpay_ebanking_fpx_MY`, `ApplePayDetails`, `GooglePayDetails`, or `StoredPaymentMethodDetails`/generic details with `type`) | yes | The type and required details of a payment method to use. |
| `paymentValidations` | PaymentValidations |  | The object that you can use to enable payment validations for a transaction. |
| `platformChargebackLogic` | PlatformChargebackLogic |  | Defines how to book chargebacks when using Adyen for Platforms. |
| `recurringExpiry` | string |  | Date after which no further authorisations shall be performed. |
| `recurringFrequency` | string |  | Minimum number of days between authorisations. |
| `recurringProcessingModel` | string |  | Defines a recurring payment type. Values: `CardOnFile`, `Subscription`, `UnscheduledCardOnFile`. |
| `redirectFromIssuerMethod` | string |  | Specifies the redirect method (GET or POST) when redirecting back from the issuer. |
| `redirectToIssuerMethod` | string |  | Specifies the redirect method (GET or POST) when redirecting to the issuer. |
| `reference` | string | yes | The reference to uniquely identify a payment. (maxLength 80) |
| `returnUrl` | string | yes | The URL to return to in case of a redirection. (maxLength 1024) |
| `riskData` | RiskData |  | Contains risk data, such as client-side data, used to identify risk for a transaction. |
| `sessionValidity` | string |  | The date and time until when the session remains valid, in ISO 8601 format. |
| `shopperConversionId` | string |  | A unique ID to connect the shopper to a single checkout session that uses multiple API requests. (maxLength 256) |
| `shopperEmail` | string |  | The shopper's email address, in valid RFC-5322 format. (maxLength 256) |
| `shopperIP` | string |  | The shopper's IP address. (maxLength 50) |
| `shopperInteraction` | string |  | Specifies the sales channel, through which the shopper gives their card details, and whether the shopper is a returning customer. Values: `Ecommerce`, `ContAuth`, `Moto`, `POS`. |
| `shopperLocale` | string |  | The language for the payment. |
| `shopperName` | ShopperName |  | The shopper's full name. |
| `shopperReference` | string |  | Required for recurring payments. (maxLength 256, minLength 3) |
| `shopperStatement` | string |  | The text to be shown on the shopper's bank statement. (maxLength 10000) |
| `shopperTaxInfo` | ShopperTaxInfo |  | The tax info of the shopper |
| `socialSecurityNumber` | string |  | The shopper's social security number. (maxLength 50) |
| `splits` | array of Split |  | An array of objects specifying how to split a payment when using Adyen for Platforms, Classic Platforms integration, or Issuing. |
| `store` | string |  | Required for Adyen for Platforms integrations if you are a platform model. (maxLength 64, minLength 1) |
| `storePaymentMethod` | boolean |  | When true and `shopperReference` is provided, the payment details will be stored for future recurring payments. |
| `subMerchants` | array of SubMerchantInfo |  | This field contains additional information on the submerchant, who is onboarded to an acquirer through a payment facilitator or aggregator |
| `surcharge` | Surcharge |  | The surcharge amount to apply to the transaction, in minor units. |
| `telephoneNumber` | string |  | The shopper's telephone number. (maxLength 64) |
| `thirdPartyTokenRedundancyInfo` | ThirdPartyTokenRedundancyInfo |  | Configuration for creating redundant payment tokens with third-party token vaults using the Adyen Forward API. |
| `threeDS2RequestData` | ThreeDS2RequestFields |  | Request fields for 3D Secure 2. |
| `threeDSAuthenticationOnly` | boolean |  | Deprecated. Required to trigger the authentication-only flow. |
| `trustedShopper` | boolean |  | Set to true if the payment should be routed to a trusted MID. |

### `PaymentResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `action` | CheckoutAwaitAction or CheckoutBankTransferAction or CheckoutDelegatedAuthenticationAction or CheckoutNativeRedirectAction or CheckoutQrCodeAction or CheckoutRedirectAction or CheckoutSDKAction or CheckoutThreeDS2Action or CheckoutVoucherAction |  | Action to be taken for completing the payment. |
| `additionalData` | object |  | Contains additional information about the payment. |
| `amount` | Amount |  | Authorised amount in the transaction. |
| `donationToken` | string |  | Donation Token containing payment details for Adyen Giving. |
| `fraudResult` | FraudResult |  | The fraud result properties of the payment. |
| `merchantReference` | string |  | The reference to uniquely identify a payment. |
| `opi` | OpiResponse |  | Oracle Payment Interface (OPI) response fields such as `transToken` and `issuerId`. |
| `order` | CheckoutOrderResponse |  | Contains updated information regarding the order in case order information was provided in the request. |
| `paymentMethod` | ResponsePaymentMethod |  | Details about the payment method used in the transaction. |
| `paymentValidations` | PaymentValidationsResponse |  | The object that contains the validation outcomes. |
| `pspReference` | string |  | Adyen's 16-character string reference associated with the transaction/request. |
| `refusalReason` | string |  | If the payment's authorisation is refused or an error occurs during authorisation, this field holds Adyen's mapped reason for the refusal or a description of the error. |
| `refusalReasonCode` | string |  | Code that specifies the refusal reason. |
| `resultCode` | string |  | The result of the payment. Values: `AuthenticationFinished`, `AuthenticationNotRequired`, `Authorised`, `Cancelled`, `ChallengeShopper`, `Error`, `IdentifyShopper`, `PartiallyAuthorised`, `Pending`, `PresentToShopper`, `Received`, `RedirectShopper`, `Refused`, `Success`. |
| `threeDS2ResponseData` | ThreeDS2ResponseData |  | Response of the 3D Secure 2 authentication. |
| `threeDS2Result` | ThreeDS2Result |  | Result of the 3D Secure 2 authentication. |
| `threeDSPaymentData` | string |  | When non-empty, contains a value that you must submit to the `/payments/details` endpoint as `paymentData`. |

### `PaymentDetailsRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `authenticationData` | DetailsRequestAuthenticationData |  | Data for 3DS authentication. |
| `details` | PaymentCompletionDetails | yes | Use this collection to submit the details that were returned as a result of the `/payments` call. |
| `paymentData` | string |  | Encoded payment data. (maxLength 200000) |
| `threeDSAuthenticationOnly` | boolean |  | Deprecated. Change the `authenticationOnly` indicator originally set in the `/payments` request. |

### `PaymentDetailsResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `action` | CheckoutThreeDS2Action |  | Action to be taken for completing the payment. |
| `additionalData` | object |  | Contains additional information about the payment. |
| `amount` | Amount |  | Authorised amount in the transaction. |
| `donationToken` | string |  | Donation Token containing payment details for Adyen Giving. |
| `fraudResult` | FraudResult |  | The fraud result properties of the payment. |
| `merchantReference` | string |  | The reference used during the /payments request. |
| `opi` | OpiResponse |  | Oracle Payment Interface (OPI) response fields such as `transToken` and `issuerId`. |
| `order` | CheckoutOrderResponse |  | Contains updated information regarding the order in case order information was provided in the request. |
| `paymentMethod` | ResponsePaymentMethod |  | Details about the payment method used in the transaction. |
| `paymentValidations` | PaymentValidationsResponse |  | The object that contains the validation outcomes. |
| `pspReference` | string |  | Adyen's 16-character string reference associated with the transaction/request. |
| `refusalReason` | string |  | If the payment's authorisation is refused or an error occurs during authorisation, this field holds Adyen's mapped reason for the refusal or a description of the error. |
| `refusalReasonCode` | string |  | Code that specifies the refusal reason. |
| `resultCode` | string |  | The result of the payment. Values: `AuthenticationFinished`, `AuthenticationNotRequired`, `Authorised`, `Cancelled`, `ChallengeShopper`, `Error`, `IdentifyShopper`, `PartiallyAuthorised`, `Pending`, `PresentToShopper`, `Received`, `RedirectShopper`, `Refused`, `Success`. |
| `shopperLocale` | string |  | The shopperLocale. |
| `threeDS2ResponseData` | ThreeDS2ResponseData |  | Response of the 3D Secure 2 authentication. |
| `threeDS2Result` | ThreeDS2Result |  | Result of the 3D Secure 2 authentication. |
| `threeDSPaymentData` | string |  | When non-empty, contains a value that you must submit to the `/payments/details` endpoint as `paymentData`. |

### `PaymentCompletionDetails`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `MD` | string |  | A payment session identifier returned by the card issuer. (maxLength 20000) |
| `PaReq` | string |  | (3D) Payment Authentication Request data for the card issuer. (maxLength 20000) |
| `PaRes` | string |  | (3D) Payment Authentication Response data by the card issuer. (maxLength 20000) |
| `authorization_token` | string |  |  |
| `billingToken` | string |  | PayPal-generated token for recurring payments. |
| `cupsecureplus.smscode` | string |  | The SMS verification code collected from the shopper. |
| `facilitatorAccessToken` | string |  | PayPal-generated third party access token. |
| `oneTimePasscode` | string |  | A random number sent to the mobile phone number of the shopper to verify the payment. |
| `orderID` | string |  | PayPal-assigned ID for the order. |
| `payerID` | string |  | PayPal-assigned ID for the payer (shopper). |
| `payload` | string |  | Payload appended to the `returnURL` as a result of the redirect. (maxLength 20000) |
| `paymentID` | string |  | PayPal-generated ID for the payment. |
| `paymentStatus` | string |  | Value passed from the WeChat MiniProgram `wx.requestPayment` **complete** callback. |
| `redirectResult` | string |  | The result of the redirect as appended to the `returnURL`. (maxLength 20000) |
| `resultCode` | string |  | Value you received from the WeChat Pay SDK. |
| `returnUrlQueryString` | string |  | The query string as appended to the `returnURL` when using direct issuer links . (maxLength 20000) |
| `threeDSResult` | string |  | Base64-encoded string returned by the Component after the challenge flow. (maxLength 50000) |
| `threeds2.challengeResult` | string |  | Base64-encoded string returned by the Component after the challenge flow. (maxLength 50000) |
| `threeds2.fingerprint` | string |  | Base64-encoded string returned by the Component after the challenge flow. (maxLength 100000) |
| `vaultToken` | string |  | PayPalv2-generated token for recurring payments. |

### `ResponseAdditionalDataCommon`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `acquirerAccountCode` | string |  | The name of the Adyen acquirer account. |
| `acquirerCode` | string |  | The name of the acquirer processing the payment request. |
| `acquirerReference` | string |  | The reference number that can be used for reconciliation in case a non-Adyen acquirer is used for settlement. |
| `alias` | string |  | The Adyen alias of the card. |
| `aliasType` | string |  | The type of the card alias. |
| `authCode` | string |  | Authorisation code: * When the payment is authorised successfully, this field holds the authorisation code for the payment. |
| `authorisationMid` | string |  | Merchant ID known by the acquirer. |
| `authorisedAmountCurrency` | string |  | The currency of the authorised amount, as a three-character ISO currency code. |
| `authorisedAmountValue` | string |  | Value of the amount authorised. |
| `avsResult` | string |  | The AVS result code of the payment, which provides information about the outcome of the AVS check. |
| `avsResultRaw` | string |  | Raw AVS result received from the acquirer, where available. |
| `bic` | string |  | BIC of a bank account. |
| `coBrandedWith` | string |  | Includes the co-branded card information. |
| `cvcResult` | string |  | The result of CVC verification. |
| `cvcResultRaw` | string |  | The raw result of CVC verification. |
| `dsTransID` | string |  | Supported for 3D Secure 2. |
| `eci` | string |  | The Electronic Commerce Indicator returned from the schemes for the 3DS payment session. |
| `expiryDate` | string |  | The expiry date on the card. |
| `extraCostsCurrency` | string |  | The currency of the extra amount charged due to additional amounts set in the skin used in the HPP payment request. |
| `extraCostsValue` | string |  | The value of the extra amount charged due to additional amounts set in the skin used in the HPP payment request. |
| `fraudCheck-[itemNr]-[FraudCheckname]` | string |  | The fraud score due to a particular fraud check. |
| `fraudManualReview` | string |  | Indicates if the payment is sent to manual review. |
| `fraudResultType` | string |  | The fraud result properties of the payment. Values: `AMBER`, `GREEN`, `RED`. |
| `fraudRiskLevel` | string |  | The risk level of the transaction as classified by the machine learning fraud risk rule. Values: `veryLow`, `low`, `medium`, `high`, `veryHigh`. |
| `fundingSource` | string |  | Information regarding the funding type of the card. |
| `fundsAvailability` | string |  | Indicates availability of funds. |
| `inferredRefusalReason` | string |  | Provides the more granular indication of why a transaction was refused. |
| `isCardCommercial` | string |  | Indicates if the card is used for business purposes only. |
| `issuerCountry` | string |  | The issuing country of the card based on the BIN list that Adyen maintains. |
| `liabilityShift` | string |  | A Boolean value indicating whether a liability shift was offered for this payment. |
| `mcBankNetReferenceNumber` | string |  | The `mcBankNetReferenceNumber`, is a minimum of six characters and a maximum of nine characters long. |
| `merchantAdviceCode` | string |  | The Merchant Advice Code (MAC) can be returned by Mastercard issuers for refused payments. |
| `merchantReference` | string |  | The reference provided for the transaction. |
| `networkProcessingMode` | string |  | Indicates the processing flow. |
| `networkTxReference` | string |  | Returned in the response if you are not tokenizing with Adyen and are using the Merchant-initiated transactions (MIT) framework from Mastercard or Visa. |
| `ownerName` | string |  | The owner name of a bank account. |
| `paymentAccountReference` | string |  | The Payment Account Reference (PAR) value links a network token with the underlying primary account number (PAN). |
| `paymentMethod` | string |  | The payment method used in the transaction. |
| `paymentMethodVariant` | string |  | The Adyen sub-variant of the payment method used for the payment request. |
| `payoutEligible` | string |  | Indicates whether a payout is eligible or not for this card. |
| `realtimeAccountUpdaterStatus` | string |  | The response code from the Real Time Account Updater service. |
| `receiptFreeText` | string |  | Message to be displayed on the terminal. |
| `recurring.contractTypes` | string |  | The recurring contract types applicable to the transaction. |
| `recurring.firstPspReference` | string |  | The `pspReference`, of the first recurring payment that created the recurring detail. |
| `recurring.recurringDetailReference` | string |  | Deprecated. The reference that uniquely identifies the recurring transaction. |
| `recurring.shopperReference` | string |  | Deprecated. The provided reference of the shopper for a recurring transaction. |
| `recurringProcessingModel` | string |  | The processing model used for the recurring transaction. Values: `CardOnFile`, `Subscription`, `UnscheduledCardOnFile`. |
| `referred` | string |  | If the payment is referred, this field is set to true. |
| `refusalReasonRaw` | string |  | Raw refusal reason received from the acquirer, where available. |
| `requestAmount` | string |  | The amount of the payment request. |
| `requestCurrencyCode` | string |  | The currency of the payment request. |
| `shopperInteraction` | string |  | The shopper interaction type of the payment request. |
| `shopperReference` | string |  | The shopperReference passed in the payment request. |
| `terminalId` | string |  | The terminal ID used in a point-of-sale payment. |
| `threeDAuthenticated` | string |  | A Boolean value indicating whether 3DS authentication was completed on this payment. |
| `threeDAuthenticatedResponse` | string |  | The raw 3DS authentication result from the card issuer. |
| `threeDOffered` | string |  | A Boolean value indicating whether 3DS was offered for this payment. |
| `threeDOfferedResponse` | string |  | The raw enrollment result from the 3DS directory services of the card schemes. |
| `threeDSVersion` | string |  | The 3D Secure 2 version. |
| `tokenization.shopperReference` | string |  | The reference for the shopper that you sent when tokenizing the payment details. |
| `tokenization.store.operationType` | string |  | The operation performed on the token. Values: `created`, `updated`, `alreadyExisting`. |
| `tokenization.storedPaymentMethodId` | string |  | The reference that uniquely identifies tokenized payment details. |
| `transactionLinkId` | string |  | Returned in the response for Mastercard payments. |
| `visaTransactionId` | string |  | The `visaTransactionId`, has a fixed length of 15 numeric characters. |
| `xid` | string |  | The 3DS transaction ID of the 3DS session sent in notifications. |

### `Amount`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `currency` | string | yes | The three-character ISO currency code of the amount. (maxLength 3, minLength 3) |
| `value` | integer (int64) | yes | The numeric value of the amount, in minor units. |

### `Address`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `city` | string | yes | The name of the city. (maxLength 3000) |
| `country` | string | yes | The two-character ISO-3166-1 alpha-2 country code. |
| `houseNumberOrName` | string | yes | The number or name of the house. (maxLength 3000) |
| `postalCode` | string | yes | A maximum of five digits for an address in the US, or a maximum of ten characters for an address in all other countries. |
| `stateOrProvince` | string |  | The two-character ISO 3166-2 state or province code. |
| `street` | string | yes | The name of the street. (maxLength 3000) |

### `BrowserInfo`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `acceptHeader` | string | yes | The accept header value of the shopper's browser. |
| `colorDepth` | integer (int32) | yes | The color depth of the shopper's browser in bits per pixel. |
| `javaEnabled` | boolean | yes | Boolean value indicating if the shopper's browser is able to execute Java. |
| `javaScriptEnabled` | boolean |  | Boolean value indicating if the shopper's browser is able to execute JavaScript. |
| `language` | string | yes | The `navigator.language` value of the shopper's browser (as defined in IETF BCP 47). |
| `screenHeight` | integer (int32) | yes | The total height of the shopper's device screen in pixels. |
| `screenWidth` | integer (int32) | yes | The total width of the shopper's device screen in pixels. |
| `timeZoneOffset` | integer (int32) | yes | Time difference between UTC time and the shopper's browser local time, in minutes. |
| `userAgent` | string | yes | The user agent value of the shopper's browser. |

### `Name`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `firstName` | string | yes | The first name. (maxLength 80) |
| `lastName` | string | yes | The last name. (maxLength 80) |

### `CheckoutRedirectAction`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `data` | object |  | When the redirect URL must be accessed via POST, use this data to post to the redirect URL. |
| `method` | string |  | Specifies the HTTP method, for example GET or POST. |
| `paymentMethodType` | string |  | Specifies the payment method. |
| `type` | string | yes | **redirect** Values: `redirect`. |
| `url` | string |  | Specifies the URL to redirect to. |

### `CheckoutThreeDS2Action`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `authorisationToken` | string |  | A token needed to authorise a payment. |
| `paymentData` | string |  | Encoded payment data. |
| `paymentMethodType` | string |  | Specifies the payment method. |
| `subtype` | string |  | A subtype of the token. |
| `token` | string |  | A token to pass to the 3DS2 Component to get the fingerprint. |
| `type` | string | yes | **threeDS2** Values: `threeDS2`. |
| `url` | string |  | Specifies the URL to redirect to. |

### `CheckoutQrCodeAction`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `expiresAt` | string |  | Expiry time of the QR code. |
| `paymentData` | string |  | Encoded payment data. |
| `paymentMethodType` | string |  | Specifies the payment method. |
| `qrCodeData` | string |  | The contents of the QR code as a UTF8 string. |
| `type` | string | yes | **qrCode** Values: `qrCode`. |
| `url` | string |  | Specifies the URL to redirect to. |

### `CheckoutAwaitAction`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `paymentData` | string |  | Encoded payment data. |
| `paymentMethodType` | string |  | Specifies the payment method. |
| `type` | string | yes | **await** Values: `await`. |
| `url` | string |  | Specifies the URL to redirect to. |

### `CheckoutSDKAction`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `paymentData` | string |  | Encoded payment data. |
| `paymentMethodType` | string |  | Specifies the payment method. |
| `sdkData` | object |  | The data to pass to the SDK. |
| `type` | string | yes | The type of the action. Values: `sdk`, `wechatpaySDK`. |
| `url` | string |  | Specifies the URL to redirect to. |

### `CheckoutVoucherAction`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `alternativeReference` | string |  | The voucher alternative reference code. |
| `collectionInstitutionNumber` | string |  | A collection institution number (store number) for Econtext Pay-Easy ATM. |
| `downloadUrl` | string |  | The URL to download the voucher. |
| `entity` | string |  | An entity number of Multibanco. |
| `expiresAt` | string |  | The date time of the voucher expiry. |
| `initialAmount` | Amount |  | The initial amount. |
| `instructionsUrl` | string |  | The URL to the detailed instructions to make payment using the voucher. |
| `issuer` | string |  | The issuer of the voucher. |
| `maskedTelephoneNumber` | string |  | The shopper telephone number (partially masked). |
| `merchantName` | string |  | The merchant name. |
| `merchantReference` | string |  | The merchant reference. |
| `passCreationToken` | string |  | A Base64-encoded token containing all properties of the voucher. |
| `paymentData` | string |  | Encoded payment data. |
| `paymentMethodType` | string |  | Specifies the payment method. |
| `reference` | string |  | The voucher reference code. |
| `shopperEmail` | string |  | The shopper email. |
| `shopperName` | string |  | The shopper name. |
| `surcharge` | Amount |  | The surcharge amount. |
| `totalAmount` | Amount |  | The total amount (initial plus surcharge amount). |
| `type` | string | yes | **voucher** Values: `voucher`. |
| `url` | string |  | Specifies the URL to redirect to. |

### `CheckoutNativeRedirectAction`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `data` | object |  | When the redirect URL must be accessed via POST, use this data to post to the redirect URL. |
| `method` | string |  | Specifies the HTTP method, for example GET or POST. |
| `nativeRedirectData` | string |  | Native SDK's redirect data containing the direct issuer link and state data that must be submitted to the /v1/nativeRedirect/redirectResult. |
| `paymentMethodType` | string |  | Specifies the payment method. |
| `type` | string | yes | **nativeRedirect** Values: `nativeRedirect`. |
| `url` | string |  | Specifies the URL to redirect to. |

### `PaymentCaptureRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `amount` | Amount | yes | The amount that you want to capture. |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `enhancedSchemeData` | EnhancedSchemeData |  | Enhanced scheme data that may be required for processing the payment. |
| `lineItems` | array of LineItem |  | Price and product information of the refunded items, required for partial refunds. |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `platformChargebackLogic` | PlatformChargebackLogic |  | Defines how to book chargebacks when using Adyen for Platforms. |
| `reference` | string |  | Your reference for the capture request. |
| `splits` | array of Split |  | An array of objects specifying how the amount should be split between accounts when using Adyen for Platforms. |
| `subMerchants` | array of SubMerchantInfo |  | A List of sub-merchants. |

### `PaymentCaptureResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `amount` | Amount | yes | The captured amount. |
| `lineItems` | array of LineItem |  | Price and product information of the refunded items, required for partial refunds. |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `paymentPspReference` | string | yes | The `pspReference` of the payment to capture. |
| `platformChargebackLogic` | PlatformChargebackLogic |  | Defines how to book chargebacks when using Adyen for Platforms. |
| `pspReference` | string | yes | Adyen's 16-character reference associated with the capture request. |
| `reference` | string |  | Your reference for the capture request. |
| `splits` | array of Split |  | An array of objects specifying how the amount should be split between accounts when using Adyen for Platforms. |
| `status` | string | yes | The status of your request. Values: `received`. |
| `subMerchants` | array of SubMerchantInfo |  | List of sub-merchants. |

### `PaymentRefundRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `amount` | Amount | yes | The amount that you want to refund. |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `capturePspReference` | string |  | This is only available for PayPal refunds. |
| `enhancedSchemeData` | EnhancedSchemeData |  | Enhanced scheme data that may be required for processing the payment. |
| `lineItems` | array of LineItem |  | Price and product information of the refunded items, required for partial refunds. |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `merchantRefundReason` | string |  | The reason for the refund request. Values: `FRAUD`, `CUSTOMER REQUEST`, `RETURN`, `DUPLICATE`, `OTHER`. |
| `reference` | string |  | Your reference for the refund request. |
| `splits` | array of Split |  | An array of objects specifying how the amount should be split between accounts when using Adyen for Platforms. |
| `store` | string |  | The online store or physical store that is processing the refund. |

### `PaymentRefundResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `amount` | Amount | yes | The refund amount. |
| `capturePspReference` | string |  | This is only available for PayPal refunds. |
| `lineItems` | array of LineItem |  | Price and product information of the refunded items, required for partial refunds. |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `merchantRefundReason` | string |  | Your reason for the refund request. Values: `FRAUD`, `CUSTOMER REQUEST`, `RETURN`, `DUPLICATE`, `OTHER`. |
| `paymentPspReference` | string | yes | The `pspReference` of the payment to refund. |
| `pspReference` | string | yes | Adyen's 16-character reference associated with the refund request. |
| `reference` | string |  | Your reference for the refund request. |
| `splits` | array of Split |  | An array of objects specifying how the amount should be split between accounts when using Adyen for Platforms. |
| `status` | string | yes | The status of your request. Values: `received`. |
| `store` | string |  | The online store or physical store that is processing the refund. |

### `PaymentCancelRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `reference` | string |  | Your reference for the cancel request. |

### `PaymentCancelResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `paymentPspReference` | string | yes | The `pspReference` of the payment to cancel. |
| `pspReference` | string | yes | Adyen's 16-character reference associated with the cancel request. |
| `reference` | string |  | Your reference for the cancel request. |
| `status` | string | yes | The status of your request. Values: `received`. |

### `StandalonePaymentCancelRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `paymentReference` | string | yes | The `reference` of the payment that you want to cancel. |
| `reference` | string |  | Your reference for the cancel request. |

### `StandalonePaymentCancelResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `paymentReference` | string | yes | The `reference` of the payment to cancel. |
| `pspReference` | string | yes | Adyen's 16-character reference associated with the cancel request. |
| `reference` | string |  | Your reference for the cancel request. |
| `status` | string | yes | The status of your request. Values: `received`. |

### `PaymentReversalRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `reference` | string |  | Your reference for the reversal request. |

### `PaymentReversalResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `paymentPspReference` | string | yes | The `pspReference` of the payment to reverse. |
| `pspReference` | string | yes | Adyen's 16-character reference associated with the reversal request. |
| `reference` | string |  | Your reference for the reversal request. |
| `status` | string | yes | The status of your request. Values: `received`. |

### `PaymentAmountUpdateRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `adjustAuthType` | string |  | The type of adjustment. Values: `cardholderInitiatedTransaction`, `merchantInitiatedTransaction`. |
| `adjustAuthorisationData` | string |  | The required data to make a synchronous authorization adjustment. |
| `amount` | Amount | yes | The updated amount. |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `industryUsage` | string |  | The reason for the amount update. Values: `delayedCharge`, `installment`, `noShow`. |
| `lineItems` | array of LineItem |  | Price and product information of the refunded items, required for partial refunds. |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `mpiData` | ThreeDSecureData |  | Authentication data from a merchant plug-in (MPI) like Mastercard SecureCode, Visa Secure, or Cartes Bancaires. |
| `reference` | string |  | Your reference for the amount update request. |
| `splits` | array of Split |  | An array of objects specifying how the amount should be split between accounts when using Adyen for Platforms. |

### `PaymentAmountUpdateResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `adjustAuthorisationData` | string |  | The data blob for subsequent synchronous adjust authorisation calls. |
| `amount` | Amount | yes | The updated amount. |
| `industryUsage` | string |  | The reason for the amount update. Values: `delayedCharge`, `installment`, `noShow`. |
| `lineItems` | array of LineItem |  | Price and product information of the refunded items, required for partial refunds. |
| `merchantAccount` | string | yes | The merchant account that is used to process the payment. |
| `paymentPspReference` | string | yes | The `pspReference` of the payment to update. |
| `pspReference` | string | yes | Adyen's 16-character reference associated with the amount update request. |
| `reference` | string | yes | Your reference for the amount update request. |
| `splits` | array of Split |  | An array of objects specifying how the amount should be split between accounts when using Adyen for Platforms. |
| `status` | string | yes | The status of your request. Values: `authorised`, `received`, `refused`. |

### `PaymentLinkRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `allowedPaymentMethods` | array of string |  | List of payment methods to be presented to the shopper. |
| `amount` | Amount | yes | The payment amount and currency. |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `billingAddress` | Address |  | The address where to send the invoice. |
| `blockedPaymentMethods` | array of string |  | List of payment methods to be hidden from the shopper. |
| `captureDelayHours` | integer (int32) |  | The delay between the authorisation and scheduled auto-capture, specified in hours. |
| `countryCode` | string |  | The shopper's two-letter country code. (maxLength 100) |
| `dateOfBirth` | string (date) |  | The shopper's date of birth. |
| `deliverAt` | string (date-time) |  | The date and time when the purchased goods should be delivered. |
| `deliveryAddress` | Address |  | The address where the purchased goods should be delivered. |
| `description` | string |  | A short description visible on the payment page. |
| `expiresAt` | string (date-time) |  | The date when the payment link expires. |
| `fundOrigin` | FundOrigin |  | The person or entity funding the money. |
| `fundRecipient` | FundRecipient |  | the person or entity receiving the money |
| `installmentOptions` | object |  | A set of key-value pairs that specifies the installment options available per payment method. |
| `lineItems` | array of LineItem |  | Price and product information about the purchased items, to be included on the invoice sent to the shopper. |
| `manualCapture` | boolean |  | Indicates if the payment must be captured manually. |
| `mcc` | string |  | The merchant category code (MCC) is a four-digit number, which relates to a particular market segment. (maxLength 16) |
| `merchantAccount` | string | yes | The merchant account identifier for which the payment link is created. |
| `merchantOrderReference` | string |  | This reference allows linking multiple transactions to each other for reporting purposes (for example, order auth-rate). (maxLength 1000) |
| `metadata` | object |  | Metadata consists of entries, each of which includes a key and a value. (maxLength 80) |
| `platformChargebackLogic` | PlatformChargebackLogic |  | Dictates the behavior of how a potential chargeback should be booked when using Adyen Platforms. |
| `recurringProcessingModel` | string |  | Defines a recurring payment type. Values: `CardOnFile`, `Subscription`, `UnscheduledCardOnFile`. (maxLength 50) |
| `reference` | string | yes | A reference that is used to uniquely identify the payment in future communications about the payment status. |
| `requiredShopperFields` | array of string |  | List of fields that the shopper has to provide on the payment page before completing the payment. Values: `billingAddress`, `deliveryAddress`, `shopperEmail`, `shopperName`, `telephoneNumber`. |
| `returnUrl` | string |  | Website URL used for redirection after payment is completed. (maxLength 8000) |
| `reusable` | boolean |  | Indicates whether the payment link can be reused for multiple payments. |
| `riskData` | RiskData |  | Any risk-related settings to apply to the payment. |
| `shopperEmail` | string |  | The shopper's email address. (maxLength 500) |
| `shopperLocale` | string |  | The language to be used in the payment page, specified by a combination of a language and country code. (maxLength 32) |
| `shopperName` | Name |  | The shopper's full name. |
| `shopperReference` | string |  | Your reference to uniquely identify this shopper, for example user ID or account ID. (maxLength 256, minLength 3) |
| `shopperStatement` | string |  | The text to be shown on the shopper's bank statement. (maxLength 10000) |
| `showRemovePaymentMethodButton` | boolean |  | Set to **false** to hide the button that lets the shopper remove a stored payment method. |
| `socialSecurityNumber` | string |  | The shopper's social security number. (maxLength 32) |
| `splitCardFundingSources` | boolean |  | Boolean value indicating whether the card payment method should be split into separate debit and credit options. |
| `splits` | array of Split |  | An array of objects specifying how to split a payment when using Adyen for Platforms, Classic Platforms integration, or Issuing. |
| `store` | string |  | The physical store, for which this payment is processed. |
| `storePaymentMethodMode` | string |  | Indicates if the details of the payment method will be stored for the shopper. Values: `askForConsent`, `disabled`, `enabled`. |
| `telephoneNumber` | string |  | The shopper's telephone number. (maxLength 32) |
| `themeId` | string |  | A theme to customize the appearance of the payment page. |
| `threeDS2RequestData` | CheckoutSessionThreeDS2RequestData |  | The cardholder phone number need to be part of the authentication message for payment data. |

### `PaymentLinkResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `allowedPaymentMethods` | array of string |  | List of payment methods to be presented to the shopper. |
| `amount` | Amount | yes | The payment amount and currency. |
| `applicationInfo` | ApplicationInfo |  | Information about your application. |
| `billingAddress` | Address |  | The address where to send the invoice. |
| `blockedPaymentMethods` | array of string |  | List of payment methods to be hidden from the shopper. |
| `captureDelayHours` | integer (int32) |  | The delay between the authorisation and scheduled auto-capture, specified in hours. |
| `countryCode` | string |  | The shopper's two-letter country code. (maxLength 100) |
| `dateOfBirth` | string (date) |  | The shopper's date of birth. |
| `deliverAt` | string (date-time) |  | The date and time when the purchased goods should be delivered. |
| `deliveryAddress` | Address |  | The address where the purchased goods should be delivered. |
| `description` | string |  | A short description visible on the payment page. |
| `expiresAt` | string (date-time) |  | The date when the payment link expires. |
| `fundOrigin` | FundOrigin |  | The person or entity funding the money. |
| `fundRecipient` | FundRecipient |  | the person or entity receiving the money |
| `id` | string | yes | A unique identifier of the payment link. |
| `installmentOptions` | object |  | A set of key-value pairs that specifies the installment options available per payment method. |
| `lineItems` | array of LineItem |  | Price and product information about the purchased items, to be included on the invoice sent to the shopper. |
| `manualCapture` | boolean |  | Indicates if the payment must be captured manually. |
| `mcc` | string |  | The merchant category code (MCC) is a four-digit number, which relates to a particular market segment. (maxLength 16) |
| `merchantAccount` | string | yes | The merchant account identifier for which the payment link is created. |
| `merchantOrderReference` | string |  | This reference allows linking multiple transactions to each other for reporting purposes (for example, order auth-rate). (maxLength 1000) |
| `metadata` | object |  | Metadata consists of entries, each of which includes a key and a value. (maxLength 80) |
| `platformChargebackLogic` | PlatformChargebackLogic |  | Dictates the behavior of how a potential chargeback should be booked when using Adyen Platforms. |
| `recurringProcessingModel` | string |  | Defines a recurring payment type. Values: `CardOnFile`, `Subscription`, `UnscheduledCardOnFile`. (maxLength 50) |
| `reference` | string | yes | A reference that is used to uniquely identify the payment in future communications about the payment status. |
| `requiredShopperFields` | array of string |  | List of fields that the shopper has to provide on the payment page before completing the payment. Values: `billingAddress`, `deliveryAddress`, `shopperEmail`, `shopperName`, `telephoneNumber`. |
| `returnUrl` | string |  | Website URL used for redirection after payment is completed. (maxLength 8000) |
| `reusable` | boolean |  | Indicates whether the payment link can be reused for multiple payments. |
| `riskData` | RiskData |  | Any risk-related settings to apply to the payment. |
| `shopperEmail` | string |  | The shopper's email address. (maxLength 500) |
| `shopperLocale` | string |  | The language to be used in the payment page, specified by a combination of a language and country code. (maxLength 32) |
| `shopperName` | Name |  | The shopper's full name. |
| `shopperReference` | string |  | Your reference to uniquely identify this shopper, for example user ID or account ID. (maxLength 256, minLength 3) |
| `shopperStatement` | string |  | The text to be shown on the shopper's bank statement. (maxLength 10000) |
| `showRemovePaymentMethodButton` | boolean |  | Set to **false** to hide the button that lets the shopper remove a stored payment method. |
| `socialSecurityNumber` | string |  | The shopper's social security number. (maxLength 32) |
| `splitCardFundingSources` | boolean |  | Boolean value indicating whether the card payment method should be split into separate debit and credit options. |
| `splits` | array of Split |  | An array of objects specifying how to split a payment when using Adyen for Platforms, Classic Platforms integration, or Issuing. |
| `status` | string | yes | Status of the payment link. Values: `active`, `completed`, `expired`, `paid`, `paymentPending`. |
| `store` | string |  | The physical store, for which this payment is processed. |
| `storePaymentMethodMode` | string |  | Indicates if the details of the payment method will be stored for the shopper. Values: `askForConsent`, `disabled`, `enabled`. |
| `telephoneNumber` | string |  | The shopper's telephone number. (maxLength 32) |
| `themeId` | string |  | A theme to customize the appearance of the payment page. |
| `threeDS2RequestData` | CheckoutSessionThreeDS2RequestData |  | The cardholder phone number need to be part of the authentication message for payment data. |
| `updatedAt` | string (date-time) |  | The date when the payment link status was updated. |
| `url` | string | yes | The URL at which the shopper can complete the payment. |

### `UpdatePaymentLinkRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `status` | string | yes | Status of the payment link. Values: `expired`. |

### `StoredPaymentMethodRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `merchantAccount` | string | yes | The merchant account identifier, with which you want to process the transaction. |
| `opi` | OpiRequest |  | Configuration for Oracle Payment Interface (OPI) integrations. |
| `paymentMethod` | PaymentMethodToStore | yes | Contains the information required to store a payment method. |
| `recurringProcessingModel` | string | yes | Defines a recurring payment type. Values: `CardOnFile`, `Subscription`, `UnscheduledCardOnFile`. |
| `shopperEmail` | string |  | The shopper's email address. |
| `shopperIP` | string |  | The IP address of a shopper. |
| `shopperReference` | string | yes | A unique identifier for the shopper (for example, user ID or account ID). |

### `StoredPaymentMethodResource`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `alias` | string |  | The alias of the credit card number. |
| `aliasType` | string |  | The alias type of the credit card number. |
| `billingAddress` | Address |  | The billing address associated with the stored payment method. |
| `brand` | string |  | The brand of the card. |
| `cardBin` | string |  | The bank identification number (BIN) of the card. |
| `createdAt` | string (date-time) |  | The date when the recurring details were created. |
| `expiryMonth` | string |  | The month the card expires. |
| `expiryYear` | string |  | The last two digits of the year the card expires. |
| `externalResponseCode` | string |  | The response code returned by an external system (for example after a provisioning operation). |
| `externalTokenReference` | string |  | The token reference of a linked token in an external system (for example a network token reference). |
| `firstPspReference` | string |  | The PSP reference of the first payment that created this recurring detail. |
| `holderName` | string |  | The unique payment method code. |
| `iban` | string |  | The IBAN of the bank account. |
| `id` | string |  | A unique identifier of this stored payment method. |
| `issuerName` | string |  | The name of the issuer of token or card. |
| `lastFour` | string |  | The last four digits of the PAN. |
| `mandate` | TokenMandate |  | Mandate details for the stored payment method. |
| `name` | string |  | The display name of the stored payment method. |
| `networkTxReference` | string |  | Returned in the response if you are not tokenizing with Adyen and are using the Merchant-initiated transactions (MIT) framework from Mastercard or Visa. |
| `opi` | OpiResponse |  | Oracle Payment Interface (OPI) response fields such as `transToken` and `issuerId`. |
| `ownerName` | string |  | The name of the bank account holder. |
| `shopperEmail` | string |  | The shopper’s email address. |
| `shopperReference` | string |  | Your reference to uniquely identify this shopper, for example user ID or account ID. (maxLength 256, minLength 3) |
| `supportedRecurringProcessingModels` | array of string |  | Defines a recurring payment type. |
| `type` | string |  | The type of payment method. |

### `ListStoredPaymentMethodsResponse`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `merchantAccount` | string |  | Your merchant account. |
| `shopperReference` | string |  | Your reference to uniquely identify this shopper, for example user ID or account ID. |
| `storedPaymentMethods` | array of StoredPaymentMethodResource |  | List of all stored payment methods. |

### `LineItem`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `amountExcludingTax` | integer (int64) |  | Item amount excluding the tax, in minor units. |
| `amountIncludingTax` | integer (int64) |  | Item amount including the tax, in minor units. |
| `brand` | string |  | Brand of the item. |
| `color` | string |  | Color of the item. |
| `description` | string |  | Description of the line item. (maxLength 10000) |
| `id` | string |  | ID of the line item. |
| `imageUrl` | string |  | Link to the picture of the purchased item. |
| `itemCategory` | string |  | Item category, used by the payment methods PayPal and Ratepay. |
| `manufacturer` | string |  | Manufacturer of the item. |
| `marketplaceSellerId` | string |  | Marketplace seller id. |
| `productUrl` | string |  | Link to the purchased item. |
| `quantity` | integer (int64) |  | Number of items. |
| `receiverEmail` | string |  | Email associated with the given product in the basket (usually in electronic gift cards). |
| `returnShippingCompany` | string |  | Shipping company handling the return of the item. |
| `returnTrackingNumber` | string |  | Tracking number for the return of the item. |
| `returnTrackingUri` | string |  | Tracking URI for the return of the item. |
| `riskCategory` | string |  | An optional, free-text category for the item to be used in the risk evaluation. |
| `shippingCompany` | string |  | Shipping company handling the delivery of the item. |
| `shippingMethod` | string |  | Shipping method used to deliver the item. |
| `size` | string |  | Size of the item. |
| `sku` | string |  | Stock keeping unit. |
| `taxAmount` | integer (int64) |  | Tax amount, in minor units. |
| `taxPercentage` | integer (int64) |  | Tax percentage, represented as a basis point integer. |
| `trackingNumber` | string |  | Tracking number for the delivery of the item. |
| `trackingUri` | string |  | Tracking URI for the delivery of the item. |
| `upc` | string |  | Universal Product Code. |

### `ThreeDS2RequestFields`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `acctInfo` | AcctInfo |  | Additional information about the cardholder’s account provided by the 3DS Requestor. |
| `acctType` | string |  | Indicates the type of account. Values: `01`, `02`, `03`. (maxLength 2, minLength 2) |
| `acquirerBIN` | string |  | Required for authentication-only integration. |
| `acquirerMerchantID` | string |  | Required for authentication-only integration. |
| `addrMatch` | string |  | Indicates whether the cardholder shipping Address and cardholder billing address are the same. Values: `Y`, `N`. (maxLength 1, minLength 1) |
| `authenticationOnly` | boolean |  | Deprecated. If set to true, you will only perform the 3D Secure 2 authentication, and not the payment authorisation. |
| `challengeIndicator` | string |  | Deprecated. Possibility to specify a preference for receiving a challenge from the issuer. Values: `noPreference`, `requestNoChallenge`, `requestChallenge`, `requestChallengeAsMandate`. |
| `deviceRenderOptions` | DeviceRenderOptions |  | Display options for the 3D Secure 2 SDK. |
| `homePhone` | Phone |  | The home phone number provided by the cardholder. |
| `mcc` | string |  | Required for merchants that have been enrolled for 3D Secure 2 by another party than Adyen, mostly authentication-only integrations. |
| `merchantName` | string |  | Required for authentication-only integration. |
| `messageVersion` | string |  | The `messageVersion` value indicating the 3D Secure 2 protocol version. |
| `mobilePhone` | Phone |  | The mobile phone number provided by the cardholder. |
| `notificationURL` | string |  | URL to where the issuer should send the `CRes`. |
| `payTokenInd` | boolean |  | Value **true** indicates that the transaction was de-tokenised prior to being received by the ACS. |
| `paymentAuthenticationUseCase` | string |  | Indicates the type of payment for which an authentication is requested (message extension) |
| `purchaseInstalData` | string |  | Indicates the maximum number of authorisations permitted for instalment payments. (maxLength 3, minLength 1) |
| `recurringExpiry` | string |  | Date after which no further authorisations shall be performed. |
| `recurringFrequency` | string |  | Indicates the minimum number of days between authorisations. (maxLength 4) |
| `sdkAppID` | string |  | The `sdkAppID` value as received from the 3D Secure 2 SDK. |
| `sdkEphemPubKey` | SDKEphemPubKey |  | The `sdkEphemPubKey` value as received from the 3D Secure 2 SDK. |
| `sdkMaxTimeout` | integer (int32) |  | The maximum amount of time in minutes for the 3D Secure 2 authentication process. |
| `sdkReferenceNumber` | string |  | The `sdkReferenceNumber` value as received from the 3D Secure 2 SDK. |
| `sdkTransID` | string |  | The `sdkTransID` value as received from the 3D Secure 2 SDK. |
| `threeDSCompInd` | string |  | Completion indicator for the device fingerprinting. |
| `threeDSRequestorAuthenticationInd` | string |  | Indicates the type of Authentication request. |
| `threeDSRequestorAuthenticationInfo` | ThreeDSRequestorAuthenticationInfo |  | Information about how the 3DS Requestor authenticated the cardholder before or during the transaction |
| `threeDSRequestorChallengeInd` | string |  | Indicates whether a challenge is requested for this transaction. Values: `01`, `02`, `03`, `04`, `05`, `06`. |
| `threeDSRequestorID` | string |  | Required for authentication-only integration for Visa. |
| `threeDSRequestorName` | string |  | Required for authentication-only integration for Visa. |
| `threeDSRequestorPriorAuthenticationInfo` | ThreeDSRequestorPriorAuthenticationInfo |  | Information about how the 3DS Requestor authenticated the cardholder as part of a previous 3DS transaction. |
| `threeDSRequestorURL` | string |  | URL of the (customer service) website that will be shown to the shopper in case of technical errors during the 3D Secure 2 process. |
| `transType` | string |  | Identifies the type of transaction being authenticated. Values: `01`, `03`, `10`, `11`, `28`. (maxLength 2, minLength 2) |
| `transactionType` | string |  | Identify the type of the transaction being authenticated. Values: `goodsOrServicePurchase`, `checkAcceptance`, `accountFunding`, `quasiCashTransaction`, `prepaidActivationAndLoad`. |
| `whiteListStatus` | string |  | The `whiteListStatus` value returned from a previous 3D Secure 2 transaction, only applicable for 3D Secure 2 protocol version 2.2.0. |
| `workPhone` | Phone |  | The work phone number provided by the cardholder. |

### `AuthenticationData`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `attemptAuthentication` | string |  | Indicates when 3D Secure authentication should be attempted. Values: `always`, `never`. |
| `authenticationOnly` | boolean |  | Required to trigger the authentication-only flow. |
| `threeDSRequestData` | ThreeDSRequestData |  | Object with additional parameters for the 3D Secure authentication flow. |

### `ThreeDSRequestData`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `challengeWindowSize` | string |  | Dimensions of the 3DS2 challenge window to be displayed to the cardholder. Values: `01`, `02`, `03`, `04`, `05`. |
| `dataOnly` | string |  | Required to trigger the data-only flow. Values: `false`, `true`. |
| `nativeThreeDS` | string |  | Indicates if native 3D Secure authentication should be triggered when available. Values: `preferred`, `disabled`. |
| `threeDSVersion` | string |  | The version of 3D Secure to use. Values: `2.1.0`, `2.2.0`. |

### `CardDetails`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `billingSequenceNumber` | string |  | The sequence number for the debit. |
| `brand` | string |  | Secondary brand of the card. |
| `checkoutAttemptId` | string |  | The checkout attempt identifier. |
| `cupsecureplus.smscode` | string |  | Deprecated.  |
| `cvc` | string |  | The card verification code. |
| `encryptedCard` | string |  | Only include this for JSON Web Encryption (JWE) implementations. (maxLength 40000) |
| `encryptedCardNumber` | string |  | The encrypted card number. (maxLength 15000) |
| `encryptedExpiryMonth` | string |  | The encrypted card expiry month. (maxLength 15000) |
| `encryptedExpiryYear` | string |  | The encrypted card expiry year. (maxLength 15000) |
| `encryptedPassword` | string |  | This field contains an encrypted, one-time password or an authentication code provided by the cardholder. (maxLength 15000) |
| `encryptedSecurityCode` | string |  | The encrypted card verification code. (maxLength 15000) |
| `expiryMonth` | string |  | The card expiry month. |
| `expiryYear` | string |  | The card expiry year. |
| `fastlaneData` | string |  | The encoded fastlane data blob |
| `fundingSource` | string |  | The funding source that should be used when multiple sources are available. Values: `credit`, `debit`, `prepaid`. |
| `holderName` | string |  | The name of the card holder. (maxLength 15000) |
| `networkPaymentReference` | string |  | The transaction identifier from card schemes. |
| `number` | string |  | The card number. |
| `recurringDetailReference` | string |  | Deprecated. This is the `recurringDetailReference` returned in the response when you created the token. |
| `sdkData` | string |  | Base64-encoded JSON object containing SDK related parameters required by the SDK (maxLength 50000) |
| `shopperNotificationReference` | string |  | The `shopperNotificationReference` returned in the response when you requested to notify the shopper. |
| `srcCorrelationId` | string |  | An identifier used for the Click to Pay transaction. |
| `srcDigitalCardId` | string |  | The SRC reference for the Click to Pay token. |
| `srcScheme` | string |  | The scheme that is being used for Click to Pay. |
| `srcTokenReference` | string |  | The reference for the Click to Pay token. |
| `storedPaymentMethodId` | string |  | This is the `recurringDetailReference` returned in the response when you created the token. (maxLength 64) |
| `threeDS2SdkVersion` | string |  | Required for mobile integrations. (maxLength 12) |
| `type` | string |  | Default payment method details. Values: `bcmc`, `scheme`, `networkToken`, `giftcard`, `card`, `clicktopay`. |

### `MolPayDetails`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `checkoutAttemptId` | string |  | The checkout attempt identifier. |
| `issuer` | string | yes | The shopper's bank. |
| `sdkData` | string |  | Base64-encoded JSON object containing SDK related parameters required by the SDK (maxLength 50000) |
| `type` | string | yes | **molpay** Values: `molpay_ebanking_fpx_MY`, `molpay_ebanking_TH`. |

### `ApplePayDetails`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `applePayToken` | string | yes | The stringified and base64 encoded `paymentData` you retrieved from the Apple framework. (maxLength 10000) |
| `checkoutAttemptId` | string |  | The checkout attempt identifier. |
| `fundingSource` | string |  | The funding source that should be used when multiple sources are available. Values: `credit`, `debit`, `prepaid`. |
| `recurringDetailReference` | string |  | Deprecated. This is the `recurringDetailReference` returned in the response when you created the token. |
| `sdkData` | string |  | Base64-encoded JSON object containing SDK related parameters required by the SDK (maxLength 50000) |
| `storedPaymentMethodId` | string |  | This is the `recurringDetailReference` returned in the response when you created the token. (maxLength 64) |
| `type` | string |  | **applepay** Values: `applepay`. |

### `GooglePayDetails`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `checkoutAttemptId` | string |  | The checkout attempt identifier. |
| `fundingSource` | string |  | The funding source that should be used when multiple sources are available. Values: `credit`, `debit`, `prepaid`. |
| `googlePayCardNetwork` | string |  | The selected payment card network. |
| `googlePayToken` | string | yes | The `token` that you obtained from the Google Pay API `PaymentData` response. (maxLength 10000) |
| `recurringDetailReference` | string |  | Deprecated. This is the `recurringDetailReference` returned in the response when you created the token. |
| `sdkData` | string |  | Base64-encoded JSON object containing SDK related parameters required by the SDK (maxLength 50000) |
| `storedPaymentMethodId` | string |  | This is the `recurringDetailReference` returned in the response when you created the token. (maxLength 64) |
| `threeDS2SdkVersion` | string |  | Required for mobile integrations. (maxLength 12) |
| `type` | string |  | **googlepay**, **paywithgoogle** Values: `googlepay`. |

### `ServiceError`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `additionalData` | object |  | Contains additional information about the payment. |
| `errorCode` | string |  | The error code mapped to the error message. |
| `errorType` | string |  | The category of the error. |
| `message` | string |  | A short explanation of the issue. |
| `pspReference` | string |  | The PSP reference of the payment. |
| `status` | integer (int32) |  | The HTTP response status. |

### `CheckoutForwardRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `amount` | Amount |  | The amount of the forwarded payment. |
| `baseUrl` | string | yes | The base URL of the third party API, where Adyen will send the request to forward the payment details. |
| `merchantAccount` | string | yes | Your merchant account. |
| `merchantReference` | string |  | Merchant defined payment reference. |
| `options` | CheckoutForwardRequestOptions |  | The customizations that can be applied when making a forward request. |
| `paymentMethod` | CheckoutForwardRequestCard |  | The card details. |
| `request` | CheckoutOutgoingForwardRequest | yes | The details of the request that you want to forward to the third-party. |
| `shopperReference` | string | yes | Your reference to uniquely identify this shopper, for example user ID or account ID. |
| `storedPaymentMethodId` | string |  | The unique identifier of the token that you want to forward to the third party. |

### `Split`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `account` | string |  | The unique identifier of the account to which the split amount is booked. |
| `amount` | SplitAmount |  | The amount of the split item. |
| `description` | string |  | Your description for the split item. |
| `reference` | string |  | Your unique reference for the part of the payment booked to the specified `account`. |
| `type` | string | yes | The part of the payment you want to book to the specified `account`. Values: `AcquiringFees`, `AdyenCommission`, `AdyenFees`, `AdyenMarkup`, `BalanceAccount`, `Commission`, `Default`, `Interchange`, `MarketPlace`, `PaymentFee`, `Remainder`, `SchemeFee`, `Surcharge`, `Tip`, `TopUp`, `VAT`. |


## Standard webhook (Webhooks v1)

### `NotificationRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `live` | string |  | Informs about the origin of the notification. |
| `notificationItems` | array of NotificationRequestItemWrapper |  | A container object for the details included in the notification. |

### `NotificationRequestItem`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `additionalData` | NotificationAdditionalData |  | A generic container for extra fields. |
| `amount` | Amount | yes | The payment amount. |
| `eventCode` | string | yes | The type of event the notification item is for. 43 enum values. |
| `eventDate` | string (date-time) | yes | The time when the event was generated. |
| `merchantAccountCode` | string | yes | The merchant account identifier used in the transaction the notification item is for. |
| `merchantReference` | string | yes | Your reference to uniquely identify the payment. |
| `originalReference` | string |  | For modifications, this field corresponds to the payment request assigned to the original payment. |
| `paymentMethod` | string |  | The payment method used in the transaction. |
| `pspReference` | string | yes | Adyen's 16-character unique reference associated with the transaction or request. |
| `reason` | string |  | If `success` = `false`, then this includes a short message with an explanation for the refusal. |
| `success` | string | yes | Informs about the outcome of the event (`eventCode`) the notification is for. |


## Recurring tokens life cycle webhook (TokenizationNotification v1)

### `TokenizationCreatedDetailsNotificationRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `createdAt` | string (date-time) | yes | The date and time when the event happened, in ISO 8601 extended format. |
| `data` | RecurringTokenStoreOperation | yes | Contains event details. |
| `environment` | string | yes | The environment from which the webhook originated. Values: `test`, `live`. |
| `eventId` | string | yes | The PSP reference of the event that triggered the webhook. |
| `type` | string | yes | The type of webhook. Values: `recurring.token.created`. |
| `version` | string |  | The version of this entity. |

### `RecurringToken`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `merchantAccount` | string | yes | The identifier of the merchant account related to the event that triggered the webhook. |
| `shopperReference` | string | yes | Your unique shopper reference that is associated with the `storedPaymentMethodId`. |
| `storedPaymentMethodId` | string | yes | The ID of the token. |
| `type` | string | yes | The type of the payment method. |

### `RecurringTokenStoreOperation`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `merchantAccount` | string | yes | The identifier of the merchant account related to the event that triggered the webhook. |
| `operation` | string | yes | A text description that provides details about the operation, intended for audit purposes. |
| `shopperReference` | string | yes | Your unique shopper reference that is associated with the `storedPaymentMethodId`. |
| `storedPaymentMethodId` | string | yes | The ID of the token. |
| `type` | string | yes | The type of the payment method. |


## Disputes API v30 (`https://ca-test.adyen.com/ca/services/DisputeService/v30`)

### `AcceptDisputeRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `disputePspReference` | string | yes | The PSP reference assigned to the dispute. |
| `merchantAccountCode` | string | yes | The merchant account identifier, for which you want to process the dispute transaction. |

### `DefendDisputeRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `defenseReasonCode` | string | yes | The defense reason code that was selected to defend this dispute. |
| `disputePspReference` | string | yes | The PSP reference assigned to the dispute. |
| `merchantAccountCode` | string | yes | The merchant account identifier, for which you want to process the dispute transaction. |

### `DefenseReasonsRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `disputePspReference` | string | yes | The PSP reference assigned to the dispute. |
| `merchantAccountCode` | string | yes | The merchant account identifier, for which you want to process the dispute transaction. |

### `SupplyDefenseDocumentRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `defenseDocuments` | array of DefenseDocument | yes | An array containing a list of the defense documents. |
| `disputePspReference` | string | yes | The PSP reference assigned to the dispute. |
| `merchantAccountCode` | string | yes | The merchant account identifier, for which you want to process the dispute transaction. |

### `DefenseDocument`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `content` | string (byte) | yes | The content of the defense document. |
| `contentType` | string | yes | The content type of the defense document. |
| `defenseDocumentTypeCode` | string | yes | The document type code of the defense document. |

### `DeleteDefenseDocumentRequest`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `defenseDocumentType` | string | yes | The document type code of the defense document. |
| `disputePspReference` | string | yes | The PSP reference assigned to the dispute. |
| `merchantAccountCode` | string | yes | The merchant account identifier, for which you want to process the dispute transaction. |

