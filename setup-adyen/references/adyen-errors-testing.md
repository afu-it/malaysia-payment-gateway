# Adyen Errors, Refusals and Sandbox Testing

Checked 2026-10-21. All test data below is **sandbox only** (published by Adyen for everyone). It does not work in live.

## Source URLs

- `https://docs.adyen.com/development-resources/test-cards-and-credentials/test-card-numbers`
- `https://docs.adyen.com/development-resources/testing/3d-secure-2-authentication`
- `https://docs.adyen.com/development-resources/testing/result-codes`
- `https://docs.adyen.com/development-resources/testing/tokenization`
- `https://docs.adyen.com/development-resources/error-codes`
- `https://docs.adyen.com/development-resources/refusal-reasons`
- `https://docs.adyen.com/development-resources/raw-acquirer-responses`
- `https://docs.adyen.com/development-resources/response-handling`
- `https://docs.adyen.com/online-payments/go-live-checklist` (end-to-end test list)

## Error Response Shape

Source: `https://docs.adyen.com/development-resources/response-handling`

```json
{ "status": 403, "errorCode": "901", "message": "Invalid Merchant Account", "errorType": "security", "pspReference": "<PSP_REFERENCE>" }
```

- `errorType`: `validation`, `security`, `configuration`, `internal`.
- HTTP 200 with `resultCode: "Refused"` is a decline, not an API error.
- HTTP status (docs table): 400 malformed request, do not repeat unchanged; 401 missing or invalid credentials; 403 insufficient permission (role or merchant account access, for example `901`, `010`), do not repeat; 404 wrong URL; 408 timeout, can retry; 409 already processed or in progress; 422 validation error (`errorCode` explains); 429 too many requests (`705`), exponential backoff; 500 / 504 server or upstream error, retry with the same `idempotency-key` (check `transient-error: true`).
- Idempotency: `704` request already processed or in progress (409/422); `703` idempotency store unavailable (503). Source: `https://docs.adyen.com/development-resources/api-idempotency`

## Refusal Reasons (`refusalReasonCode` / `refusalReason`)

Source: `https://docs.adyen.com/development-resources/refusal-reasons`. Do not show these to shoppers word for word; they reveal fraud logic.

| Code | Reason | Code | Reason |
| --- | --- | --- | --- |
| 0 | (none; ignore if `Authorised`) | 22 | FRAUD-CANCELLED |
| 2 | Refused | 23 | Transaction Not Permitted |
| 3 | Referral | 24 | CVC Declined |
| 4 | Acquirer Error | 25 | Restricted Card |
| 5 | Blocked Card | 26 | Revocation Of Auth |
| 6 | Expired Card | 27 | Declined Non Generic |
| 7 | Invalid Amount | 28 | Withdrawal amount exceeded |
| 8 | Invalid Card Number | 29 | Withdrawal count exceeded |
| 9 | Issuer Unavailable | 31 | Issuer Suspected Fraud |
| 10 | Not supported | 32 | AVS Declined |
| 11 | 3D Not Authenticated | 33 | Card requires online pin |
| 12 | Not enough balance | 34 / 35 | No checking / savings account |
| 14 | Acquirer Fraud | 36 | Mobile pin required |
| 15 | Cancelled | 37 | Contactless fallback |
| 16 | Shopper Cancelled | 38 | Authentication required (retry with 3DS) |
| 17 | Invalid Pin | 39 | RReq not received from DS |
| 18 | Pin tries exceeded | 40 | Current AID is in Penalty Box |
| 19 | Pin validation not possible | 41 | CVM Required Restart Payment |
| 20 | FRAUD (risk score 100 or more) | 42 | 3DS Authentication Error |
| 21 | Not Submitted | 46 | Blocked by Adyen to prevent excessive retry fees |
| | | 50 | Token Revoked |

Raw acquirer codes: enable **Add Raw Acquirer Result** to get `additionalData.refusalReasonRaw`. Source: `https://docs.adyen.com/development-resources/raw-acquirer-responses`

## Test Cards

Source: `https://docs.adyen.com/development-resources/test-cards-and-credentials/test-card-numbers`

Expiry **03/2030**, CVC **737** (Amex CID **7373**) unless noted. Any holder name.

| Brand | Number |
| --- | --- |
| Visa | `4111 1111 4555 1142`, `4166 6766 6766 6746`, `4988 4388 4388 4305` |
| Mastercard | `5555 5555 5555 4444`, `2222 4000 7000 0005` |
| American Express | `3700 0000 0000 002` |
| JCB | `3569 9900 1009 5841` |
| UnionPay | `6243 0300 0000 0001` (expiry 12/2029) |
| Diners | `3600 6666 3333 44` |
| Discover | `6011 6011 6011 6611` |

API-only raw test values (no encryption, test only): `encryptedCardNumber: "test_4111111111111111"`, `encryptedExpiryMonth: "test_03"`, `encryptedExpiryYear: "test_2030"`, `encryptedSecurityCode: "test_737"`.

## 3D Secure 2 Test Data

Source: `https://docs.adyen.com/development-resources/testing/3d-secure-2-authentication`

- Challenge password: web `password`; native app flow `1234`. `NoLiabilityShift` as password simulates no liability shift.
- 3DS2 cards (expiry 03/2030): Visa `4917 6100 0000 0000`, Mastercard `5454 5454 5454 5454`, Amex `3714 4963 5398 431` (CID 7373), JCB `3566 1111 1111 1113`, UnionPay `6250 9470 0000 0014` (CVC 123).
- Challenge flow: `4212 3456 7891 0006`. Frictionless: `5201 2815 0512 9736`.
- `transStatus` cards: Y `5201281505129736`, N `5201281262435268`, A `5201285093823592`, U `5201282828366351`, R `5201286496816589`.

## Trigger Result Codes (sandbox)

Source: `https://docs.adyen.com/development-resources/testing/result-codes`

Set `holderName` (or `additionalData.RequestedTestAcquirerResponseCode`) on a test card payment:

| holderName / code | Result |
| --- | --- |
| `APPROVED` (1) | Authorised |
| `DECLINED` (2) | Refused |
| `REFERRAL` (3), `ERROR` (4), `BLOCK_CARD` (5), `CARD_EXPIRED` (6), `INVALID_AMOUNT` (7), `INVALID_CARD_NUMBER` (8), `ISSUER_UNAVAILABLE` (9), `NOT_SUPPORTED` (10), `NOT_3D_AUTHENTICATED` (11), `NOT_ENOUGH_BALANCE` (12) | Refused with that reason |
| `PENDING` (13) | Received |
| `ACQUIRER_FRAUD` (14), `CANCELLED` (15), `SHOPPER_CANCELLED` (16), `INVALID_PIN` (17), `PIN_TRIES_EXCEEDED` (18), `PIN_VALIDATION_NOT_POSSIBLE` (19), `FRAUD` (20), `NOT_SUBMITTED` (21) | Refused |
| `FRAUD_CANCELLED` (22) | Cancelled |
| `TRANSACTION_NOT_PERMITTED` (23) to `ERPS_BLOCK` (46), for example `CVC_DECLINED` (24), `RESTRICTED_CARD` (25), `AUTHENTICATION_REQUIRED` (38), `THREED_SECURE_AUTHENTICATION_ERROR` (42) | Refused |
| `UNKNOWN` (0) | Error |

Modification failures: `holderName: "capture failed"`, `holderName: "refund failed"`.

## Malaysian Local Methods In Sandbox

Alipay sandbox wallet (sandbox only, published at `https://docs.adyen.com/development-resources/test-cards-and-credentials/alternative-payment-method-credentials`): username `forex_1698906295803@alitest.com`, PIN `111111`, payment PIN `111111`.

Adyen Payment Simulator (on once the method is added in the test Customer Area) for FPX, DuitNow, Touch 'n Go, GrabPay, Boost, Atome, Alipay, Alipay+. WeChat Pay has no test platform. Details per method: `adyen-payment-methods-my.md`.

## Go-Live Test List

Source: `https://docs.adyen.com/online-payments/go-live-checklist`

1. One successful payment per method with real details (small amounts).
2. A `Refused` payment, and a `FRAUD` refusal (risk score above 100).
3. A refund and a partial refund; a capture if using manual capture.
4. Webhooks received and acknowledged for every event code you use.
5. Real iOS and Android devices; secure context (HTTPS or `localhost`).

## Error Codes (full list of headings)

Source: `https://docs.adyen.com/development-resources/error-codes`. Each code page section has the cause and fix; this list is the index.

### Generic error codes

- `000` Multiple messages
- `010` Not allowed
- `100` Required object 'amount' is not provided
- `101` Invalid card number
- `102` Unable to determine variant
- `103` CVC is not the right length
- `104` Billing address problem
- `105` Invalid paRes from issuer
- `106` This session was already used previously
- `107` Recurring is not enabled
- `108` Invalid bankaccount number
- `109` Invalid variant
- `110` BankDetails missing
- `111` Invalid BankCountryCode specified
- `112` This bank country is not supported
- `113` No InvoiceLines provided
- `114` Received an incorrect InvoiceLine
- `115` Total amount is not the same as the sum of the lines
- `116` Invalid date of birth
- `117` Invalid billing address
- `118` Invalid delivery address
- `119` Invalid shopper name
- `120` Required field 'shopperEmail' is not provided
- `121` Required field 'shopperReference' is not provided
- `122` Required field 'telephoneNumber' is not provided.
- `124` Invalid PhoneNumber
- `125` Invalid recurring contract specified
- `126` Bank Account or Bank Location Id not valid or missing
- `127` Required field 'accountHolderName' is not provided
- `128` Required field 'card.holderName' is not provided
- `129` Expiry Date Invalid
- `130` Reference Missing
- `131` Required field 'billingAddress.city' is not provided
- `132` Required field 'billingAddress.street' is not provided
- `133` Required field 'billingAddress.houseNumberOrName' is not provided
- `134` Billing address problem (Country)
- `135` Required field 'billingAddress.stateOrProvince' is not provided
- `136` Failed to retrieve OpenInvoiceLines
- `137` Field 'amount' is not valid
- `138` Field 'currency' is not valid
- `139` Recurring requires 'shopperEmail' and 'shopperReference'
- `140` Invalid expiryMonth[1..12] / expiryYear[>2000], or before now
- `141` Invalid expiryMonth[1..12] / expiryYear[>2000]
- `142` Bank Name or Bank Location not valid or missing
- `143` Submitted total iDEAL merchantReturnUrl length is {0}, but max size is {1} for this request
- `144` Invalid startMonth[1..12] / startYear[>2000], or in the future
- `145` Invalid issuer countryCode
- `146` Invalid social security number
- `147` Required field 'deliveryAddress.city' is not provided
- `148` Required field 'deliveryAddress.street' is not provided
- `149` Required field 'deliveryAddress.houseNumberOrName' is not provided
- `150` Delivery address problem (Country)
- `151` Required field 'deliveryAddress.stateOrProvince' is not provided
- `152` Invalid number of installments
- `153` Invalid CVC
- `154` No additional data specified
- `155` No acquirer specified
- `156` No authorisation mid specified
- `157` No fields specified
- `158` Required field {0} not provided
- `159` Invalid number of requests
- `160` Not allowed to store Payout Details
- `161` Invalid iban
- `162` Inconsistent iban
- `163` Invalid bic
- `164` Auto capture delay invalid or out of range
- `165` MandateId does not match pattern
- `166` Amount not allowed for this operation
- `167` Original pspReference required for this operation
- `168` AuthorisationCode required for this operation
- `170` Generation Date required but missing
- `171` Unable to parse Generation Date
- `172` Encrypted data used outside of valid time period
- `173` Unable to load Private Key for decryption
- `174` Unable to decrypt data
- `175` Unable to parse JSON data
- `180` Invalid shopperReference
- `181` Invalid shopperEmail
- `182` Invalid selected brand
- `183` Invalid recurring contract
- `184` Invalid recurring detail name
- `185` Invalid additionalData
- `186` Missing additionalData field
- `187` Invalid additionalData field
- `188` Invalid pspEchoData
- `189` Invalid shopperStatement
- `190` Invalid shopper IP
- `191` No params specified
- `192` Invalid field {0}
- `193` Bin Details not found for the given card number
- `194` Billing address missing
- `195` Could not find an account with this key: {0}
- `196` Invalid Mcc
- `198` Reference may not exceed 79 characters
- `199` The cryptographic operation could not proceed, no key configured
- `200` Invalid country code
- `203` Invalid Bank account holder name
- `205` Missing or invalid networkTxReference
- `213` Required field 'socialSecurityNumber' is not provided.
- `217` Field 'shopperInteraction' is missing or invalid
- `218` Recurring shopper lookup failed
- `600` No InvoiceProject provided
- `601` No InvoiceBatch provided
- `602` No creditorAccount specified
- `603` No projectCode specified
- `604` No creditorAccount found
- `605` No project found
- `606` Unable to create InvoiceProject
- `607` InvoiceBatch already exists
- `608` Unable to create InvoiceBatch
- `609` InvoiceBatch validity period exceeded
- `690` Error while storing debtor
- `691` Error while storing invoice
- `692` Error while checking if invoice already exists for creditorAccount
- `693` Error while searching invoices
- `694` No Invoice Configuration configured for creditAccount
- `695` Invalid Invoice Configuration configured for creditAccount
- `700` No method specified
- `701` Server could not process request
- `702` Problem parsing request
- `703` Required resource temporarily unavailable
- `704` Request already processed or in progress
- `705` Rate limited
- `800` Contract not found
- `801` Too many PaymentDetails defined
- `802` Invalid contract
- `803` PaymentDetail not found
- `803_3` No active payment details found for shopper reference
- `804` Failed to disable
- `805` RecurringDetailReference not available for provided recurring-contract
- `806` No applicable contractTypes left for this payment-method
- `807` Invalid combination of shopper interaction and recurring-contract
- `820` CVC is required for OneClick card payments
- 901 Invalid Merchant Account
- `902` Invalid or empty request data
- `903` Internal error
- `904` Unable To Process
- `905` Payment details are not supported
- `905_1` Could not find an acquirer account for the provided txvariant ({0}), currency ({1}), and action ({2})
- `905_2` No specified acquirer account found for '{0}' for specified acquirer '{1}' with variant '{2}' for unit '{3}' for Action '{4}'
- `905_3` Could not find an acquirer account for the provided currency ({0})
- `905_4` Cashback and Cashout are not allowed for the configured acquirer account
- `905_5` No acquirer account active and configured for Klarna
- `905_6` Both 'KlarnaPayments' and 'Klarna' platforms have been configured for the merchant account. Only one of two is allowed
- `906` Invalid Request: Original pspReference is invalid for this environment
- `907` Payment details are not supported for this country/ MCC combination
- `908` Invalid request
- `910` Invalid Store
- `912` The TX Variant does not support the redemption type
- `916` The transaction amount exceeds the allowed limit for this type of card
- `918` Required object 'card' is not provided
- `919` The 'additionalAmount' currency should be the same currency as the payment
- `920` Total payment amount should be equal or greater than zero
- `921_1` Object 'mpiData' is missing, required due to threeDReference
- `921_2` Object 'mpiData' is not valid. Reason: 'authenticationResponse' missing
- `921_2` Object 'mpiData' is not valid. Reason: 'directoryResponse' mismatch
- `922` Account updater connection only supported for Visa & MasterCard
- `923` No registered account for AccountUpdater
- `924` Not allowed to change the shopperInteraction
- `926` Invalid iDEAL issuer provided
- `927` Missing payment details
- `928` Invalid forex type specified
- `929` Invalid base currency specified
- `930` Invalid target currency specified
- `931` There is no merchant with the code {0}
- `950` Invalid AcquirerAccount
- `951` Configuration Error (acquirerIdentification)
- `952` Configuration Error (acquirerPassword)
- `953` Configuration Error (apiKey)
- `954` Configuration Error (redirectUrl)
- `955` Configuration Error (AcquirerAccountData)
- `956` Configuration Error (currencyCode)
- `957` Configuration Error (terminalId)
- `958` Configuration Error (serialNumber)
- `959` Configuration Error (password)
- `960` Configuration Error (projectId)
- `961` Configuration Error (merchantCategoryCode)
- `962` Configuration Error (merchantName)
- `963` Invalid company registration number
- `964` Invalid company name
- `965` Missing company details
- `966` Configuration Error (privateKeyAlias)
- `967` Configuration Error (publicKeyAlias)
- `1000` Card number cannot be specified for Incontrol virtual card requests
- `1001` Recurring not allowed for Incontrol virtual card requests
- `1002` Invalid Authorisation Type supplied
- `2000` Bank account details do not meet instruction format
- `29_001` Field '{0}' may not exceed {1} characters.
- `29_003` Field 'fraudOffset' must be between -999 and 999
- `29_100` The request is larger than the allowed maximum of {0} bytes

### Checkout error codes

- `14_002` paymentData has not been provided in the request
- `14_003` Invalid paymentData
- `14_004` Missing payment method details
- `14_005` Invalid payment method details
- `14_006` paymentMethod object has not been provided in the request
- `14_007` Invalid payment method data
- `14_008` Session has expired
- `14_010` The request contains no sdkVersion although the channel is set to Web
- `14_011` The provided channel is conflicting the provided parameters
- `14_012` The provided SDK token could not be parsed
- `14_013` For HTML Response; origin has to be provided
- `14_014` This end point requires the version to be specified
- `14_015` Token is missing
- `14_016` Invalid sdkVersion provided
- `14_017` The provided SDK Token has an invalid timestamp
- `14_018` Invalid payload provided
- `14_019` The request does not contain sdkVersion or token
- `14_020` No issuer selected
- `14_022` Missing uniqueTerminalId
- `14_023` Installments were not configured in setup request
- `14_024` Invalid open invoice request
- `14_026` The selected flow is invalid
- `14_027` The provided returnUrlQueryString is invalid
- `14_028` 3D Auth Data is incomplete
- `14_029` CVC is required for this payment but not provided
- `14_030` Return URL is missing
- `14_031` Bank account is missing
- `14_032` Provide either threeds2.fingerprint or threeds2.challengeResult
- `14_033` The provided fingerprint is invalid
- `14_034` The provided fingerprint is invalid. Field affected: **FIELD_NAME**
- `14_035` 'origin' or 'notificationURL' must be provided
- `14_036` Missing required field 'channel'
- `14_037` Provided orderData is invalid
- `14_0378` Provided recurringExpiry format is invalid. Required format: yyyy-MM-dd'T'HH:mm:ssX, for example 2019-12-31T23:59:59+02:00
- `14_0379` expiresAt is greater than the allowed date, max allowed: **VALUE**
- `14_0380` PaymentData and MD belong to different requests, please submit the correct MD
- `14_0381` Missing or invalid clientKey/originKey
- `14_0382` Missing or invalid checkoutAttemptId
- `14_0383` Missing or invalid shopperStage
- `14_0384` Missing or invalid details for provided shopperStage
- `14_0385` PaymentLink is expired
- `14_0386` Hosted Session is expired
- `14_0387` PaymentLink is already paid
- `14_0388` Field 'description' is too long
- `14_0389` Pos to PayByLink QR Code is expired
- `14_0390` Missing required field 'redirectResult'
- `14_0391` Invalid redirectResult provided
- `14_0392` Payment details are incomplete. Please provide both PaRes and MD.
- `14_0393` storingMethod.type **VALUE** is invalid
- `14_0394` Required field recurringProcessingModel is not provided
- `14_0395` Storing method is incomplete
- `14_0396` Push account receipt is invalid
- `14_0397` Order Data is expired
- `14_0398` Order Data Does Not Have Access To MerchantAccount
- `14_0399` Invalid field 'pushAccountData'
- `14_0400` The provisioning request was declined
- `14_0401` Authentication of a 3DS2 payment has failed
- `14_0402` Missing 3DS2 fingerprint
- `14_0403` Missing delegated authentication result
- `14_0404` Client data is not allowed to be passed during payment link creation.
- `14_0405` Donation token or Donation Original Psp Reference or Both are missing
- `14_0406` Parent PSP Ref missing for donation.
- `14_0407` Missing required field 'donationAccount'
- `14_0408` There are no payment methods available for the given parameters.
- `14_0410` Unable to generate a redirect url for the 3DS transaction.
- `14_0411` The gift card provided for the order has 0 balance.
- `14_0412` The field 'paymentMethod.type' is required for non-recurring payments.
- `14_0413` The 'paymentMethod.type' could not be retrieved from recurring details.
- `14_0414` The provided 3DS2 result is invalid.
- `14_0415` Missing required field 'threeDSResult'
- `14_0416` Unable to retrieve the total weight of a package.
- `14_0417` The carbon credit project id is not configured.
- `14_0418` Invalid response from the terminal or missing parameters
- `14_0419` Donation Token is expired.
- `14_0420` Both installmentOptions.values and installmentOptions.maxValue can't be specified in the request
- `14_0421` Invalid platform for Voucher Pass.
- `14_0422` The provided session identifier or data is invalid.
- `14_0423` The session has expired.
- `14_0424` A product has a negative quantity.
- `14_0425` Terminal with uniqueTerminalId '{0}' was not found
- `14_0426` Terminal with uniqueTerminalId '{0}' is not deployed
- `14_0427` Insufficient permissions for terminal with uniqueTerminalId '{0}'
- `14_0428` Merchant account is missing Pay by Link permission(s)
- `14_0429` Donations are not supported for this payment method.
- `14_0430` Provided merchant unknown
- `14_0431` Unknown terminal
- `14_0432` Unknown terminal
- `14_0433` A theme and terms of service must be configured to send emails to the shopper.
- `14_0434` An internal error occurred when sending the link to the shopper.
- `14_0435` The theme name is already in use.
- `14_0436` Field **FIELD_NAME** is too long.
- `14_0438` A default theme must be configured for the account.
- `14_0439` This theme is default. Please select another default theme before deleting it.
- `14_0444` No terms and condtions were configured for this account.
- `14_0445` The specified theme does not exist.
- `14_0446` The file specified for the field **FIELD_NAME** must be a valid JPG/JPEG image.
- `14_0447` no `bin` or `encryptedBin` provided.
- `14_0448` Provided paymentData and MD don't belong to the same payment.
- `14_0449` The field **FIELD_NAME** is not supported when mode: **MODE**.
- `14_0450` The provided request cannot be processed as the session is in **STATE** state.
- `14_0451` Subset of options was not specified for field **FIELD_NAME**.
- `14_0452` Subset of options was specified for field **FIELD_NAME** but all options were selected.
- `14_0453` No options are expected when fillMode is empty for field **FIELD_NAME**.
- `14_0454` Options mode was not specified for field **FIELD_NAME**.
- `14_0455` The fill mode of the field **FIELD_NAME** must be mandatory.
- `14_0456` Country **COUNTRY_CODE** is invalid.
- `14_0457` Options mode for **FIELD_NAME** should be subset.
- `14_0458` **FIELD_NAME** with flexible validity should not contain quantity or unit.
- `14_0459` **FIELD_NAME** with fixed validity should contain quantity and unit.
- `14_0460` The provided redirect data is invalid.
- `14_0461` The provided redirectId is invalid.
- `14_0462` The provided return query string is invalid.
- `14_0463` Invalid field 'recurringProcessingModel'
- `14_0464` The provided request cannot be processed as the sessionID in the returnURL is different from the current sessionID.
- `14_0465` Missing or invalid field 'sessionId'
- `14_0466` Invalid field 'sessionId'
- `14_0467` Missing or invalid field 'Forward-Url'
- `14_0468` Metadata value size exceeds limit
- `14_0469` Option not supported for new shopper field **FIELD_NAME**.
- `14_0470` Invalid paymentMethodData. Contains both raw and encrypted card details.
- `14_0471` Field 'expireAfterRefusal' is not allowed.
- `14_0472` qrCodeExpiresAt is greater than the allowed date, max allowed: **VALUE**
- `14_0473` The details have expired.
- `14_0474` Klarna lineItems fields `taxAmount` and `amountExcludingTax` are missing or not valid
- `14_0475` shopperInteraction Moto not allowed due to PCI regulations
- `14_0476` `amount.value` cannot be less than the `amount.value` of the original request.
- `14_0477` User is not allowed to update the shipping address
- `14_0478` Sum of selected deliveryMethod amount and original offer amount does not match the total amount.
- `14_0479` Only 1 deliveryMethod should be selected.
- `14_0480` Order can only be updated if payment is still pending.
- `14_0481` The value 'amount.currency' provided in the request does not match the `amount.currency` of the original request.
- `14_0482` Invalid parameter 'returnUrl'
- `14_0483` Invalid field 'sessionResult'
- `14_0484` Missing required field 'UPI ID / VPA'
- `14_0485` Missing exchange rate token
- `14_0486` Invalid exchange rate token
- `14_0487` Exchange rate has expired
- `14_0488` Missing required field 'shopperTaxInfo.taxId'
- `14_0489` Invalid field 'shopperTaxInfo.taxId'
- `14_0490` Transaction amount exceeds the maximum allowed amount
- `14_0491` Conflicting data found on request fields and sdkData field
- `14_0492` Invalid tax-id type for the merchant
- `14_0493` Invalid field 'paymentMethod'. UPI Collect transactions using Android are no longer supported. Please use UPI Intent or UPI QR instead.
- `14_0494` The encrypted card details in 'paymentMethod' are invalid.

### 3DS2 error codes

- `15_002` Required field **FIELD_NAME** missing for device channel **DEVICE_CHANNEL_NAME**
- `15_003` Field **FIELD_NAME** should not be present for device channel **DEVICE_CHANNEL_NAME**
- `15_004` Invalid value in field **FIELD_NAME**
- `15_005` Unexpected field **FIELD_NAME**
- `15_006` messageCategory shall be present
- `15_007` Value of **FIELD_NAME** is too long (**VALUE** > **MAX_ALLOWED_VALUE**)
- `15_008` Invalid merchant configuration for 3DS 2.0
- `15_009` threeDS2Token is required
- `15_010` Invalid deviceChannel value. Must be 'app', 'browser' or 'TRI'.
- `15_011` pspReference is not a valid pspReference
- `15_012` Result not present
- `15_013` Cannot add 3DS2 authentication values because addRawThreeDSecureDetailsResult is not enabled
- `15_014` Invalid threeDS2Token
- `15_015` Invalid browserInfo provided
- `15_016` Language must be a valid tag according to IETF BCP47 definition
- `15_017` Cannot perform an authorisation on a 3DS2 transaction more than 12 hours after the transaction began
- `15_018` colorDepth must be 1, 4, 8, 15, 16, 24, 32 or 48 bit
- `15_019` stateOrProvince invalid
- `15_020` invalid transStatus
- `15_021` notificationURL cannot be localhost
- `15_022` invalid authenticationOnly field
- `15_023` threeDS2RequestData is not supported for threeDS2InMDFlow
- `15_024` Result not present - Authentication older than 60 days
- `15_026` Error during 3DS authentication process

### Amazon Pay error codes

- `5_601` Unknown error for Amazon Pay
- `5_602` Amazon Pay token doesn't exist for Amazon Pay
- `5_603` Amazon Pay signature corrupt
- `5_604` Invalid Amazon Pay value supplied: **VALUE**
- `5_605` Could not find Merchant Private Key
- `5_606` Amazon Pay token is not chargeable
- `5_607` Invalid account
- `5_608` Invalid refund status
- `5_609` Missing header fields
- `5_610` Amazon Pay is not available
- `5_611` Rejected by Amazon
- `5_612` Amazon Pay internal error
- `5_613` Amazon Pay timeout
- `5_614` Amazon Pay sent invalid response
- `5_615` Authentication Notification to Amazon Pay failed

### Apple Pay error codes

- `5_001` Apple Pay token amount-mismatch
- `5_002` Invalid Apple Pay token
- `5_003` Incorrect Apple Pay token version
- `5_004` Could not find Merchant Private Key
- `5_005` Could not find Merchant Public Key
- `5_006` Apple Pay signature corrupt
- `5_007` Missing certificate for Apple Pay signature
- `5_008` Apple Pay token currency-mismatch

### Google Pay error codes

- `5_202` Invalid Google Pay token
- `5_208` Google Pay token already expired
- `5_210` Google Pay token already used
- `OR_BIBED_11` Google Pay setup not approved

### Network token error codes

- `26_002` The network token is suspended/deactivated
- `26_003` The detokenization request has been declined
- `26_004` The network token PAN is missing
- `26_005` The network token is not active yet
- `26_006` The network token is not found
- `26_007` The detokenization request timed out
- `26_008` **PARAMETER_NAME**
- `26_010` Unsupported variant for network tokenization: **VALUE**
- `6_012` The tokenization request timed out
- `26_017` Operation **ACTION** is not allowed for network token
- `26_020` The detokenization request failed
