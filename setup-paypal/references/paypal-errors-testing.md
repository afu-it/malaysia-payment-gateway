# PayPal Errors, Sandbox, And Testing

Sources: `/api/rest/responses`, `/api/rest/integration/orders-api/errors`, `/api/orders/v2/error-messages`, `/api/payments/v2/errors/*` (107 pages), `/api/errors/*`, `/payment-methods/error-codes`, `/sdk/js/reference`, `/sandbox-testing/*`, `/negative-testing/*`, `/checkout/integrate`, `/checkout/refund-payment`, `/api/rest/webhooks/simulator`. Checked 2026-10-21.

## Error Shape

```json
{ "name": "UNPROCESSABLE_ENTITY", "message": "...", "debug_id": "<id>",
  "details": [ { "issue": "INSTRUMENT_DECLINED", "description": "...", "field": "...", "value": "...", "location": "body" } ],
  "links": [ ... ] }
```

Branch on `details[].issue`, not on `description` (PayPal says descriptions can change). Log `debug_id` for PayPal support.

## HTTP Status

| Status | Name | Notes |
| --- | --- | --- |
| `200`/`201`/`202`/`204` | Success | `201` created; `200` if the same request already created it (idempotent replay); `202` accepted for later; `207` multi-status on some capture/authorize responses. |
| `400` | `INVALID_REQUEST` | Malformed or schema violation. |
| `401` | `AUTHENTICATION_FAILURE` | Missing/invalid token or wrong environment credentials. |
| `403` | `NOT_AUTHORIZED` / `PERMISSION_DENIED` | Insufficient permission or feature not enabled. |
| `404` | `RESOURCE_NOT_FOUND` | Wrong ID or environment. |
| `405` / `406` | `METHOD_NOT_SUPPORTED` / `MEDIA_TYPE_NOT_ACCEPTABLE` | |
| `409` | `RESOURCE_CONFLICT` | Conflicts with an in-progress request; retry later. |
| `415` | `UNSUPPORTED_MEDIA_TYPE` | Send JSON. |
| `422` | `UNPROCESSABLE_ENTITY` | Business rule failure; see `issue`. |
| `429` | `RATE_LIMIT_REACHED` | Back off. |
| `500` / `503` | `INTERNAL_SERVER_ERROR` / `SERVICE_UNAVAILABLE` | Retry with the same `PayPal-Request-Id`. |

## Issues To Handle Explicitly (Malaysia checkout)

| Issue | Handling |
| --- | --- |
| `INSTRUMENT_DECLINED` | Funding source failed; restart the PayPal flow so the buyer picks another source (`/v5/checkout/handle-funding-failure`). |
| `PAYER_ACTION_REQUIRED` | Send buyer to `payer-action` link. |
| `ORDER_NOT_APPROVED` | Buyer has not approved; do not capture yet. |
| `ORDER_ALREADY_CAPTURED` | Only one capture per order with `intent=CAPTURE`; `GET` the order and settle from it. |
| `DUPLICATE_INVOICE_ID` | `invoice_id` already used on this account; use unique invoice IDs. |
| `CURRENCY_NOT_SUPPORTED` / `UNSUPPORTED_PAYEE_CURRENCY` / `CURRENCY_MISMATCH` | Check the account can receive MYR and that SDK `currency` matches the order. |
| `PAYEE_ACCOUNT_RESTRICTED`, `PAYEE_ACCOUNT_LOCKED_OR_CLOSED`, `PAYEE_ACCOUNT_INVALID` | Merchant account problem; stop and alert ops. |
| `TRANSACTION_REFUSED`, `PAYER_CANNOT_PAY`, `PAYMENT_DENIED`, `REDIRECT_PAYER_FOR_ALTERNATE_FUNDING` | Show failure, let buyer retry another way. |
| `PREVIOUS_REQUEST_IN_PROGRESS` | Wait and retry with the same `PayPal-Request-Id`. |
| `CANNOT_PAY_SELF` | Buyer and merchant are the same account (common in testing). |
| `REFUND_AMOUNT_EXCEEDED`, `CAPTURE_FULLY_REFUNDED`, `REFUND_NOT_ALLOWED`, `REFUND_TIME_LIMIT_EXCEEDED`, `REFUND_FAILED_INSUFFICIENT_FUNDS` | Refund rules; see Payments list. |

## Cancel URL Error Codes (alternative payment methods)

Returned as `<cancel_url>?errorcode=<code>&token=<id>` (case-insensitive): `order_not_confirmed`, `system_config_error`, `invalid_payment_method`, `payee_not_enabled_for_payment_method`, `payment_method_change_not_allowed`, `processing_error`, `min_amount_required_by_payment_method`, `payment_method_error`, `declined_by_payment_method`, `currency_not_supported_by_payment_method`, `country_not_supported_by_payment_method`, `invalid_expiry_date`, `unsupported_processing_instruction`, `order_complete_on_payment_approval`, `order_completion_in_progress`, `internal_server_error`, `not_enabled_for_payment_source`, `payment_error`. Source: `/payment-methods/error-codes`.

## JS SDK v6 Error Codes

`ERR_INVALID_CLIENT_TOKEN`, `ERR_DOMAIN_MISMATCH`, `ERR_DEV_UNABLE_TO_OPEN_POPUP`, `ERR_FLOW_PAYMENT_HANDLER_BROWSER_INCOMPATIBLE`, `INSTRUMENT_DECLINED`, `NETWORK_ERROR`. Guest card form warning: `WARN_FLOW_GUEST_CHECKOUT_SUBMIT_ERROR`.

## Sandbox Setup

- Developer Dashboard creates one sandbox Business (seller) and one Personal (buyer) account automatically (`sb-...@business.example.com`, `sb-...@personal.example.com`). Testing Tools > Sandbox Accounts > Create account lets you pick a **Country**: create a Malaysia business account to test MYR receiving behaviour, and a buyer from the market you sell to.
- View credentials: account menu > View/Edit Account. Log in at `https://www.sandbox.paypal.com`.
- PayPal: you need a PayPal Business account to test integrations outside the US and to go live (`/api/rest`).
- Webhooks simulator (`/dashboard/webhooksSimulator/`) posts mock events to any HTTPS 443 URL; mock events verify with webhook ID `WEBHOOK_ID`, cannot be resent, and cannot use the postback verify endpoint.

## Sandbox Test Cards (sandbox only, published by PayPal)

Use with a future expiry and any 3-digit CVV (4 for Amex). These target Expanded Checkout card processing and saved-card tests; for a Malaysia merchant use them through the PayPal-hosted card option or by adding them to a sandbox buyer account. The Credit card generator in the docs creates more.

| Number | Brand |
| --- | --- |
| `4012888888881881`, `4005519200000004`, `4012000033330026`, `4012000077777777`, `4217651111111119`, `4500600000000061`, `4772129056533503`, `4915805038587737` | Visa |
| `2223000048400011` | Mastercard |
| `371449635398431`, `376680816376961` | American Express |
| `36461510000039`, `36461510000013` | Diners Club |
| `6304000000000000`, `5063516945005047` | Maestro |
| `6200680000000004`, `6200680000000038` | UnionPay |
| `3636500000000260`, `3636500000000989` | JCB |

Rejection triggers (put in First Name / Name on Card, case-sensitive, Expanded Checkout): `CCREJECT-REFUSED` (`0500` DO_NOT_HONOR), `CCREJECT-SF` (`9500` SUSPECTED_FRAUD), `CCREJECT-EC` (`5400` EXPIRED_CARD), `CCREJECT-IRC` (`5180` INVALID_OR_RESTRICTED_CARD), `CCREJECT-IF` (`5120` INSUFFICIENT_FUNDS), `CCREJECT-LS` (`9520` LOST_OR_STOLEN), `CCREJECT-IA` (`1330` INVALID_ACCOUNT), `CCREJECT-BANK_ERROR` (`5100` GENERIC_DECLINE), `CCREJECT-CVV_F` (`00N7` CVV2_FAILURE_POSSIBLE_RETRY_WITH_CVV).

## Negative Testing (sandbox only)

1. Enable Negative Testing on the sandbox business account (Dashboard > Sandbox > Accounts > View/Edit).
2. Send `PayPal-Mock-Response: {"mock_application_codes":"<ISSUE>"}` on Orders v2 create/patch/get/authorize/capture and Payments v2 authorization get/capture/void, capture get/refund, refund get.
3. Useful codes: `INSUFFICIENT_FUNDS`, `INSTRUMENT_DECLINED`, `TRANSACTION_REFUSED`, `INTERNAL_SERVER_ERROR`, `DUPLICATE_INVOICE_ID`; refunds: `REFUND_AMOUNT_EXCEEDED`, `CAPTURE_FULLY_REFUNDED`, `REFUND_NOT_ALLOWED_AFTER_180_DAYS`, `PERMISSION_DENIED`.
4. Disputes, Invoicing, Payouts and Subscriptions use test values in request fields instead (for example payout `items[0].note = "ERRPYO002"` returns `SENDER_EMAIL_UNCONFIRMED`).

## Required Test Matrix

- Successful MYR capture; buyer cancel; server error during create; `INSTRUMENT_DECLINED` at capture; duplicate `invoice_id`; capture retry with same `PayPal-Request-Id` after a timeout; `ORDER_ALREADY_CAPTURED` handled as success via GET.
- Capture `PENDING` then webhook `PAYMENT.CAPTURE.COMPLETED`; `PAYMENT.CAPTURE.DENIED`; `CHECKOUT.PAYMENT-APPROVAL.REVERSED`.
- Forged webhook (bad signature), replayed webhook (same event `id`), webhook for another environment's Webhook ID, body re-serialized before verification (must fail).
- Full refund, partial refund, over-refund.

## Orders v2 Issue Codes (complete list from `/api/orders/v2/error-messages`)

| Issue | Description |
| --- | --- |
| `PERMISSION_DENIED` | You do not have permission to access or perform operations on this resource. |
| `INVALID_RESOURCE_ID` | Specified resource ID does not exist. Please check the resource ID and try again. |
| `INVALID_ACCOUNT_STATUS` | Account validations failed for the user. |
| `PREVIOUS_REQUEST_IN_PROGRESS` | A previous request on this resource is currently in progress. Please wait for some time and try again. It is best to space out the initial and the subsequent request(s) to avoid receiving this error. |
| `INVALID_ARRAY_MAX_ITEMS` | The number of items in an array parameter is too large. |
| `INVALID_ARRAY_MIN_ITEMS` | The number of items in an array parameter is too small. |
| `INVALID_COUNTRY_CODE` | Country code is invalid. Please refer to https://developer.paypal.com/api/rest/reference/country-codes/ for a list of supported country codes. |
| `INVALID_PARAMETER_SYNTAX` | The value of a field does not conform to the expected format. |
| `INVALID_STRING_LENGTH` | The value of a field is either too short or too long Possible error locations: application_context/brand_name application_context/locale payer/address/admin_area1 payer/address/admin_area2 payer/address/address_line1 payer/address/address_line2 payer/address/c |
| `INVALID_PARAMETER_VALUE` | A parameter value is not valid. |
| `MISSING_REQUIRED_PARAMETER` | A required parameter is missing. |
| `NOT_SUPPORTED` | This field is not currently supported. |
| `PAYPAL_REQUEST_ID_REQUIRED` | A PayPal-Request-Id is required if you are trying to process payment for an Order. Please specify a PayPal-Request-Id or Create the Order without a 'payment_source' specified. |
| `MALFORMED_REQUEST_JSON` | The request JSON is not well formed. |
| `PERMISSION_DENIED_FOR_DONATION_ITEMS` | The API Caller or Payee have not been granted appropriate permissions to send 'items.category' as 'DONATION'. Please speak to your account manager if you want to process these type of items. |
| `MALFORMED_REQUEST` | MALFORMED_REQUEST |
| `PAYEE_NOT_CONSENTED` | Payee does not have appropriate consent to allow the API caller to process this type of transaction on their behalf. Your current setup requires the payee to provide a consent before this transaction can be processed successfully. |
| `AMOUNT_MISMATCH` | Should equal item_total + tax_total + shipping + handling + insurance - shipping_discount - discount. |
| `INVALID_POSTAL_CODE` | For vaulting with purchase, postal code (billing_address.postal_code) can only contain letters, numbers, spaces, and hyphens |
| `INVALID_POSTAL_CODE_LENGTH` | For vaulting with purchase, postal code (billing_address.postal_code) may contain no more than 9 letter or number characters |
| `BILLING_AGREEMENTS_CANNOT_BE_USED_FOR_SUBSCRIPTIONS` | Billing Agreements cannot be used for Subscription orders. |
| `CANNOT_BE_NEGATIVE` | Must be greater than or equal to 0. If the currency supports decimals, only two decimal place precision is supported. |
| `CANNOT_BE_ZERO_OR_NEGATIVE` | Must be greater than zero. If the currency supports decimals, only two decimal place precision is supported. |
| `CARD_EXPIRED` | The card is expired |
| `MISSING_PREVIOUS_REFERENCE` | For Merchant initiated network token transactions, either the payment_source.card.stored_credential.previous_network_transaction_reference or payment_source.card.stored_credential.previous_transaction_reference must be included in the request. |
| `MISSING_CRYPTOGRAM` | Cryptogram is mandatory for any customer initiated network token transactions. |
| `CITY_REQUIRED` | The specified country requires a city (address.admin_area_2). |
| `DECIMAL_PRECISION` | If the currency supports decimals, only two decimal place precision is supported. |
| `DONATION_ITEMS_NOT_SUPPORTED` | If 'purchase_unit' has "DONATION" as the 'items.category' then the Order can at most have one purchase_unit. Multiple purchase_units are not supported if either of them have at least one items with category as "DONATION". |
| `DUPLICATE_REFERENCE_ID` | reference_id must be unique if multiple purchase_unit are provided. |
| `INVALID_CURRENCY_CODE` | Currency code is invalid or is not currently supported. Please refer https://developer.paypal.com/api/rest/reference/currency-codes/ for list of supported currency codes. |
| `INVALID_PAYER_ID` | The payer ID is not valid. |
| `ITEM_TOTAL_MISMATCH` | Should equal sum of (unit_amount * quantity) across all items for a given purchase_unit. |
| `ITEM_TOTAL_REQUIRED` | If item details are specified (items.unit_amount and items.quantity) corresponding amount.breakdown.item_total is required. |
| `MAX_VALUE_EXCEEDED` | Should be less than or equal to 999999999999999.99. |
| `PAYMENT_DENIED` | PayPal has declined to process this transaction. |
| `MISSING_PICKUP_ADDRESS` | A pickup address( shipping.address ) is required for the provided shipping.type . |
| `MULTI_CURRENCY_ORDER` | Multiple differing values of currency_code are not supported. Entire Order request must have the same currency_code. |
| `MULTIPLE_ITEM_CATEGORIES` | For a given 'purchase_unit' the 'items.category' could be either "PHYSICAL_GOODS" and/or "DIGITAL_GOODS" or just "DONATION".  'items.category' as "DONATION" cannot be combined with items with either "PHYSICAL_GOODS" or "DIGITAL_GOODS". |
| `MULTIPLE_SHIPPING_ADDRESS_NOT_SUPPORTED` | Multiple shipping addresses are not supported. |
| `MULTIPLE_SHIPPING_TYPE_NOT_SUPPORTED` | Different shipping.type are not supported across purchase units nor within an individual item's shipping options. |
| `PAYEE_ACCOUNT_INVALID` | Payee account specified is invalid. Please check the payee.email_address or payee.merchant_id specified and try again. Ensure that either payee.merchant_id or payee.email_address is specified. |
| `PAYEE_ACCOUNT_LOCKED_OR_CLOSED` | The merchant account is locked or closed. |
| `PAYEE_ACCOUNT_RESTRICTED` | The merchant account is restricted. |
| `PAYMENT_ORIGIN_NOT_ENABLED` | The API caller account or payee is not enabled for the given 'payment_origin' option. Please contact customer support for assistance. |
| `PAYEE_PRICING_TIER_ID_NOT_ENABLED` | The API Caller is not enabled to process transactions by specifying a 'payee_pricing_tier_id'. Please work with your Account Manager to enable this option for your account. |
| `INVALID_PAYEE_PRICING_TIER_ID` | Please check the value specified or confirm with your Account Manager that the 'payee_pricing_tier_id' specified has been setup for the account. |
| `UNSUPPORTED_PAYMENT_SOURCE_FOR_SUBSCRIPTIONS` | The specified payment source is not supported for subscriptions. Only a vault_id or vault_setup_token with payment_source.card and payment_source.paypal are supported. |
| `PAYEE_FX_RATE_ID_EXPIRED` | The specified FX Rate ID has expired. Please specify a different FX Rate Id and try the request again. Alternately, remove the FX Rate ID to process the request using the default exchange rate. |
| `PAYEE_FX_RATE_ID_CURRENCY_MISMATCH` | The specified FX Rate ID is for a currency that does not match with the currency of this request. Please specify a different FX Rate ID and try the request again. Alternately, remove the FX Rate ID to process the request using the default exchange rate. |
| `INVALID_FX_RATE_ID` | The specific FX Rate ID is not valid. This could be either because we are not able to look up the FX Rate based on this ID or it could be because the ID belongs to another API Caller. |
| `PLATFORM_FEES_NOT_SUPPORTED` | The API Caller is not enabled to process transactions by specifying 'platform_fees'. Please work with your PayPal Account Manager to enable this option for your account. |
| `INVALID_PLATFORM_FEES_ACCOUNT` | The specified platform_fees payee account is either invalid or account setup is incomplete.Please work with your PayPal Account Manager to enable this option for your account. |
| `INVALID_PLATFORM_FEES_AMOUNT` | The platform_fees amount cannot be greater than order amount. |
| `POSTAL_CODE_REQUIRED` | The specified country requires a postal code. |
| `REFERENCE_ID_REQUIRED` | 'reference_id' is required for each 'purchase_unit' if multiple 'purchase_unit' are provided. |
| `SHIPPING_OPTIONS_NOT_SUPPORTED` | Shipping options are not supported when shipping.type is specified or when 'application_context.shipping_preference' is set as 'NO_SHIPPING' or 'SET_PROVIDED_ADDRESS'. |
| `TAX_TOTAL_MISMATCH` | Should equal sum of (tax * quantity) across all items for a given purchase_unit. |
| `TAX_TOTAL_REQUIRED` | If item details are specified (items.tax_total and items.quantity) corresponding amount.breakdown.tax_total is required. |
| `UNSUPPORTED_INTENT` | intent=AUTHORIZE is not supported for this operation. Only intent=CAPTURE is supported. |
| `UNSUPPORTED_INTENT_FOR_KYC_INCOMPLETE_PAYEE` | intent=AUTHORIZE is not supported due to payee's incomplete KYC, while their partner is enabled for Progressive Onboarding. Only intent=CAPTURE is supported. |
| `UNSUPPORTED_PAYMENT_INSTRUCTION` | You must provide the payment instruction when you capture an authorized payment for intent=AUTHORIZE . For details, see Capture authorization . For intent=CAPTURE , send the payment instruction when you create the order. |
| `SHIPPING_TYPE_NOT_SUPPORTED_FOR_CLIENT` | The API Caller account is not setup to be able to support a shipping.type = PICKUP_IN_PERSON . This feature is only supported for PayPal Complete Payments Platform for Platforms and Marketplaces . |
| `UNSUPPORTED_SHIPPING_TYPE` | The provided shipping.type is only supported for application_context.shipping_preference = SET_PROVIDED_ADDRESS or NO_SHIPPING . |
| `SHIPPING_OPTION_NOT_SELECTED` | At least one of the shipping.option should be set to 'selected = true'. |
| `MULTIPLE_SHIPPING_OPTION_SELECTED` | Only one shipping.option can be set to 'selected = true'. |
| `INVALID_SHIPPING_OPTION_ID` | The items.shipping.options_ids[].id does not exist in purchase_units.shipping.options[]. Please ensure that all ids used in options_ids are first defined in purchase_units.shipping.options[]. |
| `INVALID_DELIVERY_ESTIMATE` | The provided delivery_time_max must be greater than the delivery_time_min , and vice versa. |
| `INELIGIBLE_SHIPPING_OPTION` | One or more shipping options cannot be used in conjunction with this Order. |
| `MISSING_SHIPPING_CALL_BACK_CONFIGURATION` | payment_source.paypal.experience_context.order_update_callback_config is mandatory for one or more shipping.options provided in the request. |
| `MISSING_ITEM_SHIPPING_OPTIONS` | All items in the purchase_unit must have options_ids defined if any item in the Order has item[].shipping.options_ids defined. |
| `PREFERRED_SHIPPING_OPTION_AMOUNT_MISMATCH` | The amount provided in the preferred shipping option should match the amount provided in amount breakdown Possible error locations: /purchase_units/0/shipping/options/0/amount/value |
| `AGREEMENT_ALREADY_CANCELLED` | The requested agreement is already canceled. |
| `BILLING_AGREEMENT_NOT_FOUND` | The requested Billing Agreement token was not found. |
| `COMPLIANCE_VIOLATION` | Transaction is declined due to compliance violation. |
| `DOMESTIC_TRANSACTION_REQUIRED` | This transaction requires the payee and payer to be resident in the same country, a domestic transaction is required to create this payment. |
| `DUPLICATE_INVOICE_ID` | Duplicate Invoice ID detected. To avoid a potential duplicate transaction your account setting requires that Invoice Id be unique for each transaction. |
| `INSTRUMENT_DECLINED` | The instrument presented  was either declined by the processor or bank, or it can't be used for this payment. |
| `MAX_NUMBER_OF_PAYMENT_ATTEMPTS_EXCEEDED` | You have exceeded the maximum number of payment attempts. |
| `NOT_ENABLED_FOR_CARD_PROCESSING` | The API Caller account is not setup to be able to process card payments. Please contact PayPal customer support. |
| `PAYEE_BLOCKED_TRANSACTION` | Declined by fraud tool (Fraud Protection/Chargeback Protection) due to potential fraud risk. Please review your fraud settings. |
| `PAYER_ACCOUNT_LOCKED_OR_CLOSED` | The payer account cannot be used for this transaction. |
| `PAYER_ACCOUNT_RESTRICTED` | PAYER_ACCOUNT_RESTRICTED |
| `PAYER_CANNOT_PAY` | Combination of payer and payee settings mean that this buyer cannot pay this seller. |
| `TRANSACTION_BLOCKED_BY_PAYEE` | Transaction blocked by Payee’s Fraud Protection settings. |
| `TRANSACTION_LIMIT_EXCEEDED` | Total payment amount exceeded transaction limit. |
| `TRANSACTION_RECEIVING_LIMIT_EXCEEDED` | The transaction exceeds the receiver's receiving limit. |
| `TRANSACTION_REFUSED` | The request was refused. |
| `AUTH_CAPTURE_NOT_ENABLED` | Authorization and Capture feature is not enabled for the merchant. Make sure that the recipient of the funds is a verified business account. |
| `UNSUPPORTED_PROCESSING_INSTRUCTION` | The specified processing_instruction is not supported for the given payment_source. Please refer to https://developer.paypal.com/api/orders/v2/#definition-processing_instruction for the list of payment_source that can be specified with this value. |
| `ORDER_COMPLETE_ON_PAYMENT_APPROVAL` | A processing_instruction of ORDER_COMPLETE_ON_PAYMENT_APPROVAL is required for the specified payment_source. Please refer to the integration guide https://developer.paypal.com/docs/limited-release/alternative-payment-methods-with-orders/ for more details |
| `INVALID_EXPIRY_DATE` | Expiry date is invalid. Expiry date should be a date in future and within the threshold for the payment source. |
| `INCOMPATIBLE_PARAMETER_VALUE` | The value of the field is incompatible/redundant with other fields in the order. |
| `INVALID_PREVIOUS_TRANSACTION_REFERENCE` | The authorization or capture referenced by previous_transaction_reference is not valid. This could be either because the previous_transaction_reference is not found or doesn't belong to the payee. Please use a valid previous_transaction_reference . |
| `PREVIOUS_TRANSACTION_REFERENCE_HAS_CHARGEBACK` | The capture referenced by previous_transaction_reference has a chargeback and hence cannot be used for this order. Please use a previous_transaction_reference which does not have a chargeback. |
| `PREVIOUS_TRANSACTION_REFERENCE_VOIDED` | The status of authorization referenced by previous_transaction_reference is VOIDED and hence cannot be used for this order. Please use a previous_transaction_reference whose status is not VOIDED . |
| `PAYMENT_SOURCE_MISMATCH` | The payment_source in the request must match the payment_source used for the authorization or capture referenced by previous_transaction_reference . Please use previous_transaction_reference whose payment_source matches with the payment_source specified in the |
| `MERCHANT_INITIATED_WITH_SECURITY_CODE` | stored_payment_source.payment_initiator = MERCHANT is not supported if payment_source.card.security_code is present in the order. |
| `MERCHANT_INITIATED_WITH_AUTHENTICATION_RESULTS` | stored_payment_source.payment_initiator = MERCHANT is not supported if 3D-Secure authentication results are present in the order. 3D-Secure authentication results can be present in the order only when customer is the payment initiator. It is semantically incor |
| `MERCHANT_INITIATED_WITH_MULTIPLE_PURCHASE_UNITS` | stored_payment_source.payment_initiator = MERCHANT is not supported if more than one purchase_unit is present in the Order. Merchant initiated payments are not supported from orders with more than one purchase_unit. Please retry the request with multiple Order |
| `PAYMENT_SOURCE_INFO_CANNOT_BE_VERIFIED` | The combination of the payment_source name, billing address, shipping name and shipping address could not be verified. Please correct this information and try again by creating a new order. |
| `PAYMENT_SOURCE_DECLINED_BY_PROCESSOR` | The provided payment source is declined by the processor. Please try again with a different payment source by creating a new order. |
| `PAYMENT_SOURCE_CANNOT_BE_USED` | The provided payment source cannot be used to pay for the order. Please try again with a different payment source by creating a new order. |
| `NOT_ENABLED_FOR_APPLE_PAY` | The 'API caller' and/or 'payee' is not setup to be able to process apple pay. Please contact your Account Manager. |
| `NOT_ENABLED_FOR_GOOGLE_PAY` | The 'API caller' and/or 'payee' is not setup to be able to process google pay. Please contact your Account Manager. |
| `APPLE_PAY_AMOUNT_MISMATCH` | The 'amount' specified in the Order should match the amount that was viewed and authorized by the payer/buyer on Apple Pay. If the amount has changed, please redirect the buyer to authorize the order again via Apple Pay. |
| `BILLING_ADDRESS_INVALID` | Provided billing address is invalid. |
| `SHIPPING_ADDRESS_INVALID` | Provided shipping address is invalid. |
| `VAULT_INSTRUCTION_DUPLICATED` | Only one vault instruction is allowed. Please use vault.store_in_vault to provide vault instruction. |
| `VAULT_INSTRUCTION_REQUIRED` | Vault instruction is required. Please use vault.store_in_vault to provide vault instruction. |
| `MISMATCHED_VAULT_OWNER_ID` | The owner_id does not match with the API caller. Please verify that the owner_id is same as the API caller's payer id. |
| `VAULT_OWNER_ID_NOT_SUPPORTED` | API caller is not enabled to specify the vault owner_id explicitly. Please contact your account manager to enable the functionality. |
| `MISMATCHED_VAULT_ID_TO_PAYMENT_SOURCE` | The vault_id does not match the payment_source provided. Please verify that the vault_id token used refers to the matching payment_source and try again. For example, a PayPal token cannot be passed in the vault_id field in the payment_source.card object. |
| `MISMATCHED_VAULT_SETUP_TOKEN_TO_PAYMENT_SOURCE` | The vault_setup_token does not match the payment_source provided. Please verify that the vault_setup_token token used refers to the matching payment_source and try again. For example, a PayPal token cannot be passed in the vault_setup_token field in the paymen |
| `CRYPTOGRAM_REQUIRED` | Cryptogram is required if authentication method is CRYPTOGRAM 3DS. |
| `EMV_DATA_REQUIRED` | EMV Data is required if authentication method is EMV. |
| `NOT_ELIGIBLE_FOR_PNREF_PROCESSING` | API caller is not enabled to process payments with the pnref . Please contact customer support to request permissions to process transactions with PNREF. |
| `NOT_ELIGIBLE_FOR_PAYPAL_TRANSACTION_ID_PROCESSING` | API caller is not enable to process payments using paypal_transaction_id . Please contact customer support to request permissions to process transactions with PayPal transaction ID. |
| `PAYPAL_TRANSACTION_ID_NOT_FOUND` | Specified paypal_transaction_id was not found. Verify the value and try the request again. |
| `PNREF_NOT_FOUND` | Specified pnref was not found. Verify the value and try the request again. |
| `INVALID_SECURITY_CODE_LENGTH` | The security_code length is invalid for the specified card brand. |
| `NOT_ENABLED_TO_VAULT_PAYMENT_SOURCE` | The API caller or the merchant on whose behalf the API call is initiated is not allowed to vault the given source. Please contact PayPal customer support for assistance. |
| `REQUIRED_PARAMETER_FOR_CUSTOMER_INITIATED_PAYMENT` | This parameter is required when the customer is present. If the customer is not present, indicate so by sending payment_initiator= MERCHANT . For details, see Stored Credential . |
| `TOKEN_EXPIRED` | The token is expired and cannot be used for payment. |
| `INVALID_GOOGLE_PAY_TOKEN` | The google pay token is invalid. PayPal was not able to decrypt the googlepay token or PayPal was not able to find the necessary data in the token after decryption. |
| `GOOGLE_PAY_GATEWAY_MERCHANT_ID_MISMATCH` | The gateway merchant ID in Google Pay token is not valid. This could be because the gateway merchant Id that was authorized by payer/buyer on Google Pay does not match with the API caller of the order. |
| `ONE_OF_PARAMETERS_REQUIRED` | One or more field is required to continue with this request. |
| `ALIAS_DECLINED_BY_PROCESSOR` | The provided alias was declined by the processor. Please create a new order with a different alias_key and/or alias_label and try again. |
| `PREFERRED_BRAND_NOT_SUPPORTED` | Processing of this preferred brand is not supported. Please use another brand preference to continue with this transaction. |
| `BLIK_ONE_CLICK_MISSING_REQUIRED_PARAMETER` | Blik's one_click flow requires one_click.auth_code and one_click.alias_label parameters for the buyer's first transaction. For all subsequent transactions,only the one_click.alias_key parameter is required. |
| `INCOMPATIBLE_ACCOUNT_OWNERSHIP_TYPE` | Account ownership type is not compatible with standard entry class code. |
| `DELAYED_DISBURSEMENT_NOT_SUPPORTED` | The API Caller is not enabled to process transactions by specifying disbursement mode as delayed. |
| `DELAYED_DISBURSEMENT_NOT_SUPPORTED_FOR_KYC_INCOMPLETE_PAYEE` | The Delayed disbursement is not supported due to payee's incomplete KYC, while their partner is enabled for Progressive Onboarding. |
| `PARTNER_SETTLEMENT_NOT_SUPPORTED_FOR_KYC_INCOMPLETE_PAYEE` | Partner settlement is not supported due to payee's incomplete KYC, while their partner is enabled for Progressive Onboarding. |
| `NOT_ENABLED_FOR_PAYMENT_SOURCE` | The 'API caller' and/or 'payee' is not setup to be able to process the selected payment source. If you have already completed the required steps, please allow 2 business days for PayPal to complete the setup. If you continue to receive this error, please conta |
| `INVALID_VAULT_ID` | The specified Vault ID is invalid or could not be found. |
| `MULTIPLE_PURCHASE_UNITS_NOT_SUPPORTED_FOR_PAYMENT_SOURCE` | The provided payment source cannot be used to pay for the multiple purchase units order. |
| `COUNTRY_NOT_SUPPORTED_BY_PAYMENT_SOURCE` | Country code provided is not supported by the provided payment source. |
| `LOCALE_NOT_SUPPORTED_BY_PAYMENT_SOURCE` | Locale provided is not supported by the provided payment source. |
| `BANK_VERIFICATION_REQUIRED` | Only verified bank accounts can be processed. |
| `SHIPPING_CALLBACK_CONFIG_NOT_SUPPORTED` | The shipping callback events SHIPPING_ADDRESS and/or SHIPPING_OPTIONS in 'order_update_callback_config.callback_events' field are not supported when shipping.type is specified or when 'shipping_preference' is set as 'NO_SHIPPING' or 'SET_PROVIDED_ADDRESS'. |
| `MISSING_REQUIRED_PARAMETER_FOR_BILLING_PLAN` | PayPal Vault billing plan requires usage_pattern to be passed in the request. Supported Values: SUBSCRIPTION, UNSCHEDULED, RECURRING, INSTALLMENT (prepaid/postpaid) |
| `UNSUPPORTED_INTEGRATION_FOR_BILLING_PLAN` | For Recurring Purchases, passing billing plan and vaulting instruction (in payment_source.paypal.attributes.vault or payment_source.paypal.billing) is mandatory. Also, only one billing plan, product and purchase unit are supported. |
| `INCOMPATIBLE_PARAMETER_FOR_BILLING_PLAN` | For Recurring Purchase, amount fields like handling, insurance, discount and shipping discount are unsupported. |
| `BILLING_ITEM_AMOUNT_MISMATCH` | For Recurring Purchases, the billing plan’s unit amount should include both the setup fee and the plan price (only if the plan starts immediately, i.e. no start_date). |
| `PARTNER_OR_MERCHANT_RESTRICTED_FOR_COUNTRY` | The Partner or Merchant is not allowed to process payments using this payment source in the specified country. Please contact PayPal technical support. |
| `MAX_EXPIRY_TIME_EXCEEDED` | The provided authorization_expiry is longer than allowed for the selected payment_source . |
| `DISCOUNT_TOTAL_MISMATCH` | The amount specified in amount.breakdown.discount should equal the sum of all individual discount values specified in amount.breakdown.discount.breakdown . |
| `CARD_PROVISIONING_FORBIDDEN` | Card provisioning is not available. This may be due to: missing provision_card indicator in order creation, incomplete TRID onboarding, invalid encryption keys, or incomplete merchant onboarding. |
| `PAYER_NOT_SUPPORTED_BY_PAYMENT_SOURCE` | The Payer was declined by the payment source. |
| `UNSUPPORTED_AUTHENTICATION_METHODS_FOR_PAYMENT_SOURCE` | The provided list of authentication methods is not supported for this payment source. |
| `FIELD_NOT_PATCHABLE` | Field cannot be patched. |
| `AMOUNT_NOT_PATCHABLE` | The amount cannot be updated as the 'payer' has chosen and approved a specific financing offer for a given amount. Please Create a new Order with the updated Order amount and have the 'payer' approve the new payment terms. |
| `INVALID_PATCH_OPERATION` | The operation cannot be honored. Cannot add a property that's already present, use replace. Cannot remove a property thats not present, use add. Cannot replace a property thats not present, use add. |
| `PAYEE_ACCOUNT_NOT_SUPPORTED` | Payee does not have an account. Your current setup requires the 'payee' to have a verified account with PayPal before you can process transactions on their behalf. |
| `PAYEE_ACCOUNT_NOT_VERIFIED` | Payee has not verified their account with PayPal. The selected payment method requires the recipient to have a verified PayPal account before transactions can be processed on their behalf. |
| `INVALID_JSON_POINTER_FORMAT` | Path should be a valid JSON Pointer https://tools.ietf.org/html/rfc6901 that references a location within the request where the operation is performed. |
| `INVALID_PARAMETER` | Cannot be specified as part of the request. |
| `NOT_PATCHABLE` | Cannot be patched. |
| `UNSUPPORTED_PATCH_PARAMETER_VALUE` | The value specified for this field is not currently supported. |
| `PATCH_VALUE_REQUIRED` | Please specify a 'value' to for the field that is being patched. |
| `PATCH_PATH_REQUIRED` | Please specify a 'path' for the field for which the operation needs to be performed. |
| `REFERENCE_ID_NOT_FOUND` | Filter expression value is incorrect. Please check the value of the reference_id and try again. |
| `DUPLICATE_SHIPPING_OPTION_ID` | Please specify a unique shipping option id for each shipping option provided in the request and try again. |
| `ORDER_ALREADY_COMPLETED` | The order cannot be patched after it is completed. |
| `CANNOT_MODIFY_INTENT` | The value of 'intent' cannot be changed. Please create a new order. |
| `AMOUNT_CHANGE_NOT_ALLOWED` | The amount specified is different from the amount authorized by payer. |
| `INVOICE_ID_CHANGE_NOT_ALLOWED` | The invoice Id cannot be changed for the given payment source. |
| `NOT_ELIGIBLE_FOR_TOKEN_PROCESSING` | API caller is not enabled to process payments with the specified type of token. Please contact customer support to request permissions to process transactions with this type of token. |
| `ACTION_DOES_NOT_MATCH_INTENT` | Order was created with an intent to 'CAPTURE'. Please use v2/checkout/orders/order_id/capture to complete the transaction or alternately Create an order with an intent of 'AUTHORIZE'. |
| `CARD_BRAND_NOT_SUPPORTED` | Processing of this card brand is not supported. Please use another card to continue with this transaction. |
| `CARD_COUNTRY_NOT_SUPPORTED` | The country in which this card was issued is not currently supported. Please use another card to continue with this transaction. |
| `CARD_DATA_NOT_FOUND` | The card issuer data information could not be retrieved. Please use a different card to proceed with this transaction. |
| `DECLINED_DUE_TO_RELATED_TXN` | One or more transactions in this Order did not succeed. Since this Order is being processed as an All or None Order, if one or more transactions in this Order do not succeed, then all purchase units are marked declined and will not be processed. |
| `ORDER_NOT_APPROVED` | Payer has not yet approved the Order for payment. Please redirect the payer to the 'rel':'approve' url returned as part of the HATEOAS links within the Create Order call. |
| `CONTINGENCY_NOT_SUCCESSFUL` | 3D-Secure payer authentication is unsuccessful, so the payment cannot be completed. |
| `UNSUPPORTED_INTENT_FOR_PAYMENT_SOURCE` | intent=AUTHORIZE is not supported for the specified payment_source. Only intent=CAPTURE is supported. |
| `PAYPAL_TRANSACTION_ID_EXPIRED` | Specified paypal_transaction_id has expired. PayPal transaction ID expires 4 years after the date of the initial transaction. |
| `PNREF_EXPIRED` | Specified pnref has expired. PNREF expires 15 months after the date of the initial transaction. |
| `REFERENCED_CARD_EXPIRED` | The card underlying the token has expired and hence cannot be used to process a payment. |
| `TOKEN_ID_NOT_FOUND` | Specified token was not found. Verify the token and try the request again. |
| `ORDER_ALREADY_AUTHORIZED` | Order already authorized.If 'intent=AUTHORIZE' only one authorization per order is allowed. |
| `AMOUNT_CANNOT_BE_SPECIFIED` | An authorization amount can only be specified if an Order has been saved by calling /v2/checkout/orders/{order_id}/save.  Please save the order and try again. |
| `AUTHORIZATION_AMOUNT_EXCEEDED` | Authorization amount specified exceeded allowable limit. Specify a different amount and try the request again. Alternately, contact Customer Support to increase your limits. Local regulations (e.g. in PSD2 countries) prohibit overages above the amount authoriz |
| `AUTHORIZATION_CURRENCY_MISMATCH` | The currency of the authorization should be same as that in which the Order was created and approved by the Payer. Please check the 'currency_code' and try again. |
| `MAX_AUTHORIZATION_COUNT_EXCEEDED` | Maximum number of authorization allowed for the order is reached. Please contact Customer Support if you need to increase your limit. |
| `ORDER_COMPLETED_OR_VOIDED` | Order is voided or completed and hence cannot be authorized. |
| `ORDER_EXPIRED` | Order is expired and hence cannot be authorized. Please contact Customer Support if you need to increase your order validity period. |
| `INVALID_PICKUP_ADDRESS` | If the 'shipping_option.type' is set as 'PICKUP' then the 'shipping_detail.name.full_name' should start with 'S2S' meaning Ship To Store. Example: 'S2S My Store'. |
| `PAYMENT_TYPE_NOT_SUPPORTED_FOR_INTENT` | Provided payment type not supported for order intent. Payment authorizations are supported only for order with intent=AUTHORIZE and payment captures are supported only for order with intent=CAPTURE . |
| `BILLING_AGREEMENT_ID_MISMATCH` | Billing Agreement ID must exactly match the Billing Agreement ID that was provided during order creation. |
| `PREFERRED_PAYMENT_SOURCE_MISMATCH` | Payment Source must exactly match the Preferred Payment Source that was provided during order creation. |
| `RETURN_URL_REQUIRED` | The return url is required when attempting to vault this source. |
| `CANCEL_URL_REQUIRED` | The cancel url is required when attempting to vault this source. |
| `PAYER_ACTION_REQUIRED` | Transaction cannot complete successfully, instruct the buyer to return to PayPal. |
| `CARD_NUMBER_REQUIRED` | The card number is required when attempting to process payment with card. |
| `CARD_EXPIRY_REQUIRED` | The card expiry is required when attempting to process payment with card. |
| `ORDER_CANNOT_BE_SAVED` | The option to save an order is only available if the intent is AUTHORIZE and processing_instruction uses one of the ORDER_SAVED options. For example, intent =AUTHORIZE, processing_instruction =ORDER_SAVED_EXPLICITLY. Please change the intent and/or processing_ |
| `SAVE_ORDER_NOT_SUPPORTED` | The API caller account is setup in a way that does not allow it to be used for saving the order. This functionality is not available for PayPal Complete Payments Platform for Platforms & Marketplaces. |
| `ORDER_AUTHORIZATION_IN_PROGRESS` | The processing_instruction sent with the selected payment_source was ORDER_COMPLETE_ON_PAYMENT_APPROVAL , resulting in auto-authorization. Merchant initiated authorization is not allowed. |
| `PROVISION_CARD_UNSUPPORTED_OPERATION` | The requested operation is not available for purchase units that contain provision_card objects. |
| `MSP_MISSING_REQUIRED_PARAMETER` | A required field / parameter is missing. |
| `CONSENT_NEEDED` | CONSENT_NEEDED |
| `REDIRECT_PAYER_FOR_ALTERNATE_FUNDING` | Transaction failed. Redirect the payer to select another funding source. |
| `ORDER_ALREADY_CAPTURED` | Order already captured.If 'intent=CAPTURE' only one capture per order is allowed. |
| `NOT_ENABLED_FOR_BANK_PROCESSING` | The API Caller account is not setup to be able to process bank payments. Please contact your PayPal account manager. |
| `PAYEE_NOT_ENABLED_FOR_BANK_PROCESSING` | Payee account is not setup to be able to process bank payments. Please contact your PayPal account manager. |
| `PAYEE_NOT_ENABLED_FOR_CARD_PROCESSING` | Payee account is not setup to be able to process card payments. Please contact PayPal customer support. |
| `PAYMENT_SOURCE_NOT_SUPPORTED` | The payer selected method of payment is not supported when multiple purchase units are specified for an Order. |
| `ORDER_COMPLETION_IN_PROGRESS` | The order was created with processing_instruction of ORDER_COMPLETE_ON_PAYMENT_APPROVAL. The customer has approved the payment and PayPal is still in the process of capturing the order on your behalf as instructed. Please try your request again. |
| `SETUP_ERROR_FOR_BANK` | The API Caller account setup, for bank payments, is incomplete or incorrect. Please contact your PayPal account manager. |
| `BANK_NOT_SUPPORTED_FOR_VERIFICATION` | Verification for this bank account is not supported. |
| `CURRENCY_NOT_SUPPORTED_FOR_BANK` | The payment_source does not support the currency of the Order. For ACH debit, only USD is supported and for SEPA debit, only EUR is supported. |
| `ONLY_ONE_BANK_SOURCE_ALLOWED` | More than one payment method within the bank payment object is not supported. |
| `INVALID_IBAN` | IBAN provided is not a valid bank account number. |
| `IBAN_COUNTRY_NOT_SUPPORTED` | Country code of issuer bank for the provided IBAN is not supported for SEPA debit payments. |
| `INVALID_VAULT_SETUP_TOKEN` | The specified Vault Setup Token is invalid or could not be found. |
| `PLATFORM_FEE_PAYEE_CANNOT_BE_SAME_AS_PAYER` | The payer cannot pay themselves. The recipient account of the platform fees must be different from the payer account. |
| `IDENTIFIER_NOT_FOUND` | Specified identifier was not found. Please verify the correct identifier was used and try the request again. |
| `AUTO_CAPTURE_ENABLED` | This order is being auto-captured. Merchant initiated capture requests are not allowed. |
| `INVALID_STRING_MAX_LENGTH` | The value of a field is too long. |
| `ORDER_CANNOT_BE_CONFIRMED` | An order with status = 'COMPLETED' cannot be confirmed again. |
| `CURRENCY_NOT_SUPPORTED_FOR_COUNTRY` | For the payment_source specified, the currency of the Order is restricted by the country in which the payee account is based. Please refer https://developer.paypal.com/api/rest/reference/currency-codes/ for list of supported currency codes. |
| `CARD_TYPE_NOT_SUPPORTED` | Processing of this card type is not supported. Use another card type. |
| `CURRENCY_NOT_SUPPORTED_FOR_CARD_TYPE` | The issued currency code of this card is not supported for direct card payments. Please refer https://developer.paypal.com/api/rest/reference/currency-codes/ for list of supported currency codes. |
| `ONLY_ONE_PAYMENT_SOURCE_ALLOWED` | More than one payment method within the payment source is not supported. |
| `NO_PAYMENT_SOURCE_PROVIDED` | At least one payment method is required within the payment source. |
| `PAYMENT_ALREADY_APPROVED` | The payment has already been approved.  Please capture the order, or create and confirm a new order with this payment source. |
| `REQUIRED_PARAMETER_FOR_PAYMENT_SOURCE` | The parameter is required for provided payment source. |
| `ITEM_CATEGORY_NOT_SUPPORTED_BY_PAYMENT_SOURCE` | The provided payment source does not support provided item category. |
| `ONE_OF_THE_PARAMETERS_REQUIRED` | One or more field is required to continue with this request. |
| `ORDER_IS_PENDING_APPROVAL` | The order was confirmed and payer action completed but order approval processing from PayPal is pending. No action is needed from Payee or Payer. Please wait until order status changes to 'APPROVED'. |
| `DEVICE_DATA_NOT_AVAILABLE` | Device Data is not available for processing this order. The PayPal-Client-Metadata-Id header value sent during Create Order api call is either missing or incorrect or there was an error in collecting required data. Please verify if appropriate value for PayPal |
| `PAYEE_COUNTRY_NOT_SUPPORTED_FOR_PAYMENT_SOURCE` | Payee country code is not supported by the provided payment source. |
| `PUI_DUPLICATE_ORDER` | A Pay Upon Invoice (Rechnungskauf) order with the same payload has already been successfully processed in the last few seconds. To process a new order, please try again in a few seconds. |
| `INVALID_TRACKING_NUMBER` | Use a valid tracking number. It must not be empty, contain spaces or tab characters |
| `CAPTURE_STATUS_NOT_VALID` | Invalid capture status. Tracker information can only be added to captures in COMPLETED state. |
| `ITEM_SKU_MISMATCH` | Item sku must match one of the items sku that was provided during order creation. |
| `CAPTURE_ID_NOT_FOUND` | Specified capture ID does not exist. Check the capture ID and try again. |
| `MSP_NOT_SUPPORTED` | Multiple purchase units are not supported for this operation. |
| `TRACKER_ID_NOT_FOUND` | Specified tracker ID does not exist. Check the tracker ID and try again. |
| `MAX_TRACKER_UPDATE_COUNT_EXCEEDED` | Exceeding the permissible number of updates; tracker update operations do not support more than 15 updates. |
| `DOMAIN_NOT_REGISTERED` | The provided domain is not registered for this merchant. |
| `TESTING_WITH_REAL_WALLET_NOT_ENABLED` | Testing with real wallet is not supported for this domain. To enable testing using real wallet, please unregister the domain and register the domain again. |

## Payments v2 Issue Codes (complete list from `/api/payments/v2/errors/*`)

| Issue | Message and description |
| --- | --- |
| `AUTH_CAPTURE_CURRENCY_MISMATCH` | Currency of capture must be the same as currency of authorization. Verify the currency of the capture and try the request again. |
| `AUTH_CURRENCY_MISMATCH` | The currency specified during reauthorization should be the same as the currency specified in the original authorization. Please check the currency of the authorization for which you are trying to reauthorize and try again. |
| `AUTHENTICATION_FAILURE` | Authentication failed due to missing authorization header, or invalid authentication credentials. Account validations failed for the user. |
| `AUTHORIZATION_ALREADY_CAPTURED` | Authorization has already been captured. If `final_capture` is set to to `true`, additional captures are not possible against the authorization. |
| `AUTHORIZATION_AMOUNT_EXCEEDED` | Authorization amount specified exceeded allowable limit. Specify a different amount and try the request again. Alternately, contact Customer Support to increase your limits. Local regulations (e.g. in PSD2 countries) prohibit overages above the amount authorized by the payer. |
| `AUTHORIZATION_DENIED` | A denied authorization cannot be captured. You cannot capture a denied authorization. |
| `AUTHORIZATION_EXPIRED` | An expired authorization cannot be captured. You cannot capture an expired authorization. |
| `AUTHORIZATION_VOIDED` | A voided authorization cannot be captured or reauthorized. You cannot capture or reauthorize a voided authorization. |
| `CANNOT_BE_NEGATIVE` | Must be greater than or equal to 0. |
| `CANNOT_BE_VOIDED` | A reauthorization cannot be voided. Please void the original parent authorization. You cannot void a reauthorized payment. You must void the original parent authorized payment. |
| `CANNOT_BE_ZERO_OR_NEGATIVE` | Must be greater than zero. If the currency supports decimals, only two decimal place precision is supported. Specify a different value and try the request again. |
| `CANNOT_REFUND_SELF` | The payer and payee for the refund cannot be same. |
| `CAPTURE_FULLY_REFUNDED` | The capture has already been fully refunded. You cannot capture additional refunds against this capture. |
| `CAPTURED_AMOUNT_FULLY_REFUNDED` | The captured amount has already been fully refunded. |
| `CARD_BILLING_ADDRESS_COUNTRY_NOT_SUPPORTED` | Specified country is not currently supported for payment processing. |
| `CARD_BRAND_NOT_SUPPORTED` | Refund cannot be issued to this card. The card brand `card_brand` is not supported. Please try again with another card. |
| `CARD_EXPIRED` | The card is expired. |
| `CARD_ISSUER_COUNTRY_NOT_SUPPORTED` | Card is issued by a financial institution for a country (e.g. Cuba, Iran, North Korea, Syria) that is not current supported for payment processing. |
| `CARD_TYPE_NOT_SUPPORTED` | Processing of this card type is not supported. Use another type of card. |
| `COMPLIANCE_VIOLATION` | Transaction is declined due to compliance violation. |
| `CURRENCY_MISMATCH` | All amounts specified should be in the same currency. Please ensure that the currency for the 'amount' and that of 'platform_fees.amount' is the same. |
| `CURRENCY_NOT_SUPPORTED_FOR_CARD_BRAND` | Currency not supported for card specified. Card is `card_brand`. Only `currency_code` is supported for this brand of card. |
| `CURRENCY_NOT_SUPPORTED_FOR_CARD_TYPE` | Currency code not supported for direct card payments using this card type. See [Currency codes](/docs/api/reference/currency-codes/) for list of supported currency codes. |
| `CURRENCY_NOT_SUPPORTED_FOR_COUNTRY` | Currency code not supported for card payments in your country. |
| `DECIMAL_PRECISION` | The value of the field should not be more than two decimal places. If the currency supports decimals, only two decimal place precision is supported. |
| `DECIMALS_NOT_SUPPORTED` | Currency does not support decimals. Currency does not support decimals. Please refer to https://developer.paypal.com/docs/api/reference/currency-codes/ for more information. |
| `DELAYED_DISBURSEMENT_NOT_SUPPORTED` | The API Caller is not enabled to process transactions by specifying disbursement mode as delayed. Verify the API caller is enabled to process transaction with delayed disbursement. |
| `DUPLICATE_INVOICE_ID` | Requested invoice number has been previously captured. Possible duplicate transaction. Payment for this invoice was already captured. |
| `DUPLICATE_REFUND` | Requested invoice_id has been previously refunded. Possible duplicate transaction. |
| `DUPLICATE_TRANSACTION` | Duplicate invoice Id detected. |
| `FX_RATE_CHANGE_DUE_TO_MARKET_EVENT` | The FX rate associated with the specified FX rate ID has been changed due to market events. Please refer to the provided new_fx_id link to retrieve a new FX rate ID and try the request again. |
| `INSTRUMENT_DECLINED` | The instrument presented was either declined by the processor or bank, or it can't be used for this payment. |
| `INTERNAL_SERVER_ERROR` | An internal server error has occurred. Try your request again later. |
| `INVALID_ACCOUNT_STATUS` | Account validations failed for the user. The user account could not be validated. |
| `INVALID_ARRAY_LENGTH` | Request is not well-formed, syntactically incorrect, or violates schema. The number of items in an array parameter is too small or too large. |
| `INVALID_CARD_NUMBER` | The card number is invalid. |
| `INVALID_COUNTRY_CODE` | The requested action could not be performed, semantically incorrect, or failed business validation. Country code is invalid. Please refer to https://developer.paypal.com/docs/integration/direct/rest/country-codes/ for a list of supported country codes. |
| `INVALID_CURRENCY_CODE` | Currency code should be a three-character ISO-4217 currency code. Currency code is invalid or is not currently supported. Please refer https://developer.paypal.com/docs/api/reference/currency-codes/ for list of supported currency codes. |
| `INVALID_FX_RATE_ID` | The specified FX Rate ID is not valid. |
| `INVALID_INVOICE_ID` | Specified invoice_id does not exist. Please check the invoice_id and try again. |
| `INVALID_PARAMETER_SYNTAX` | The value of the field does not conform to the expected format. Verify the specification for supported `pattern` and try the request again. |
| `INVALID_PARAMETER_VALUE` | The value of a field is invalid. Verify the specification for the allowed values and try the request again. |
| `INVALID_PAYEE_ACCOUNT` | Payee account is invalid. Verify the payee account information and try the request again. |
| `INVALID_PLATFORM_FEES_ACCOUNT` | The specified platform_fees payee account is either invalid or account setup is incomplete. Please work with your PayPal Account Manager to enable this option for your account. Verify the platform fee account set up is completed or correct account. |
| `INVALID_PLATFORM_FEES_AMOUNT` | The `platform_fees` amount cannot be greater than the capture amount. Verify the `platform_fees` amount and try the request again. |
| `INVALID_RESOURCE_ID` | Specified resource ID does not exist. Please check the resource ID and try again. Verify the resource ID and try the request again. |
| `INVALID_SECURITY_CODE_LENGTH` | The security_code length is invalid for the specified card brand. |
| `INVALID_STRING_LENGTH` | The value of a field is either too short or too long. Verify the specification for the supported `min` and `max` values and try the request again. |
| `INVALID_STRING_MAX_LENGTH` | The value of a field is too long. The parameter string is too long. |
| `KYC_HOLD` | The merchant's KYC status is incomplete. The payment was pending due to the merchant's incomplete KYC status. |
| `MALFORMED_REQUEST_JSON` | Request is not well-formed, syntactically incorrect, or violates schema. The request JSON is not well formed. |
| `MAX_CAPTURE_AMOUNT_EXCEEDED` | Capture amount exceeds allowable limit. Please contact customer service or your account manager to request the change to your overage limit. The default overage limit is 115%, which allows the sum of all captures to be up to 115% of the order amount. Local regulations (e.g. in PSD2 countries) ... |
| `MAX_CAPTURE_COUNT_EXCEEDED` | Maximum number of allowable captures has been reached. No additional captures are possible for this authorization. Please contact customer service or your account manager to change the number of captures that be made for a given authorization. You cannot make additional captures. |
| `MAX_NUMBER_OF_REFUNDS_EXCEEDED` | You have exceeded the number of refunds that can be processed per capture. Please contact customer support or your account manager to review the number of refunds that can be processed per capture. |
| `MAX_PAYEE_AMOUNT_LIMIT_EXCEEDED` | Refund amount exceeds the allowed cumulative limit that the payee can receive. |
| `MAX_PAYER_AMOUNT_LIMIT_EXCEEDED` | Refund amount exceeds the allowed cumulative limit that the payer can refund. |
| `MAX_REFUND_LIMIT_EXCEEDED` | Refund amount exceeds allowable limit. Specify a different amount and try the request again. Alternately, contact Customer Support to increase your limits. |
| `MISSING_REQUIRED_PARAMETER` | A required field / parameter is missing. Verify the specification for `required` fields and try the request again. |
| `MULTI_CURRENCY_CODE` | The requested action could not be performed, semantically incorrect, or failed business validation. Multiple differing values of currency_code are not supported. Entire request must have the same currency_code. |
| `MULTIPLE_AUTHORIZATIONS_FOUND` | Cannot void multiple authorizations. Specified invoice_id is associated with more than one authorization. |
| `NOT_AUTHORIZED` | You do not have permission to access or perform operations on this resource. To make API calls on behalf of a merchant, ensure that you have sufficient permissions to proceed with this transaction. |
| `PARTIAL_REFUND_NOT_ALLOWED` | You cannot do a refund for an amount less than the original capture amount. Specify an amount equal to the capture amount or omit the `amount` object from the request. Then, try the request again. |
| `PAYEE_ACCOUNT_INVALID` | The requested action could not be performed, semantically incorrect, or failed business validation. Payee account specified is invalid. Please check the `payee.merchant_id` specified and try again. |
| `PAYEE_ACCOUNT_LOCKED_OR_CLOSED` | Transaction could not complete because payee account is locked or closed. To get more information about the status of the account, call Customer Support. |
| `PAYEE_ACCOUNT_RESTRICTED` | Payee account is restricted. To get more information about the status of the account, call Customer Support. |
| `PAYEE_FX_RATE_ID_CURRENCY_MISMATCH` | The specified FX Rate ID is for a currency that does not match with the currency of this request. |
| `PAYEE_FX_RATE_ID_EXPIRED` | The specified FX Rate ID has expired. |
| `PAYEE_NOT_CONSENTED` | Payee does not have appropriate consent to allow the API caller to process this type of transaction on their behalf. Your current setup requires the 'payee' to provide a consent before this transaction can be processed successfully. |
| `PAYEE_REFUND_AMOUNT_LIMIT_EXCEEDED` | Refund amount exceeds per transaction limit that the payee can receive. |
| `PAYER_ACCOUNT_LOCKED_OR_CLOSED` | The payer account cannot be used for this transaction. Contact the payer for an alternate payment method. |
| `PAYER_ACCOUNT_RESTRICTED` | Payer account is restricted. To get more information about the status of the account, call Customer Support. |
| `PAYER_CANNOT_PAY` | Payer cannot pay for this transaction. Please contact the payer to find other ways to pay for this transaction. |
| `PAYER_REFUND_AMOUNT_LIMIT_EXCEEDED` | Refund amount exceeds per transaction limit that the payer can refund. |
| `PENDING_CAPTURE` | Cannot initiate a refund as the capture is pending. Capture is typically pending when the payer has funded the transaction by using an e-check or bank account. |
| `PERMISSION_DENIED` | You do not have permission to access or perform operations on this resource. To make API calls on behalf of a merchant, ensure that you have sufficient permissions to proceed with this transaction. |
| `PERMISSION_NOT_GRANTED` | Payee of the authorization has not granted permission to perform capture on the authorization. To make API calls on behalf of a merchant, ensure that you have sufficient permissions to capture the authorization. |
| `PLATFORM_FEE_EXCEEDED` | Platform fee amount specified exceeds the amount that is available for refund. You can only refund up to the available platform fee amount. This error is also returned when no platform_fee was specified or was zero when the payment was captured. |
| `PLATFORM_FEE_NOT_ENABLED` | The API Caller account is not setup to be able to process refunds with 'platform_fees'. Please contact your Account Manager. This feature is useful when you want to contribute a portion of the 'platform_fees' you had capture as part of the refund being processed. |
| `PLATFORM_FEES_NOT_SUPPORTED` | The API Caller is not enabled to process transactions by specifying 'platform_fees'. Please work with your PayPal Account Manager to enable this option for your account. Verify the API caller is enabled to process the transaction with platform fees. |
| `PREVIOUS_REQUEST_IN_PROGRESS` | A previous request on this resource is currently in progress. Please wait for sometime and try again. It is best to space out the initial and the subsequent request(s) to avoid receiving this error. This scenario only occurs when making multiple API requests on the same resource within a very short ... |
| `PREVIOUSLY_CAPTURED` | Authorization has been previously captured and hence cannot be voided. This authorized payment was already captured. You cannot capture it again. |
| `PREVIOUSLY_VOIDED` | Authorization has been previously voided and hence cannot be voided again. This authorized payment was already voided. You cannot void it again. |
| `RATE_LIMIT_REACHED` | Too many requests. Blocked due to rate limiting. The rate limit was reached. |
| `REAUTHORIZATION_DECLINED_BY_PROCESSOR` | The reauthorization was declined by the processor. |
| `REAUTHORIZATION_NOT_SUPPORTED` | A reauthorize cannot be attempted on an authorization_id that is the result of a prior reauthorization or on an authorization made on an Order saved using the `v2/orders/id/save` API. |
| `REAUTHORIZATION_NOT_SUPPORTED_FOR_PAYMENT_SOURCE` | Reauthorization is not supported for the payment source. |
| `REAUTHORIZATION_TOO_SOON` | A reauthorization is only allowed once from Day 4 to Day 29 since the date of the original authorization. |
| `REFUND_AMOUNT_EXCEEDED` | The refund amount must be less than or equal to the capture amount that has not yet been refunded. Verify the refund amount and try the request again. |
| `REFUND_AMOUNT_TOO_LOW` | The amount after applying currency conversion is zero and hence the capture cannot be refunded. The currency conversion is required because the currency of the capture is different than the currency in which the amount was settled into the payee account. |
| `REFUND_CAPTURE_CURRENCY_MISMATCH` | Refund must be in the same currency as the capture. Verify the currency of the refund and try the request again. |
| `REFUND_CURRENCY_MISMATCH` | Refund must be in the same currency as the capture. |
| `REFUND_FAILED_BY_PAYMENT_SOURCE` | Refund was refused by the payment source. We're unable to process refunds for the payer's selected payment source. Contact the payer directly to arrange a refund via alternative means. |
| `REFUND_FAILED_INSUFFICIENT_FUNDS` | Refund cannot be processed due to insufficient funds. Capture could not be refunded due to insufficient funds. Please check to see if you have sufficient funds in your PayPal account or if the bank account linked to your PayPal account is verified and has sufficient funds. |
| `REFUND_FUNDS_NOT_AVAILABLE` | There is no valid funding instrument linked to the account from which refund can be processed. |
| `REFUND_IS_RESTRICTED` | This refund can only be processed by the API caller that had 'captured' the transaction. If you facilitate your transactions via a platform/partner, please initiate a refund through them. |
| `REFUND_NOT_ALLOWED` | Capture cannot be refunded. You cannot refund this capture. |
| `REFUND_NOT_PERMITTED_DUE_TO_CHARGEBACK` | The requested action could not be performed, semantically incorrect, or failed business validation. Refunds not allowed on this capture due to a chargeback on the card or bank. Please contact the payee to resolve the chargeback. |
| `REFUND_NOT_SUPPORTED_FOR_PAYMENT_SOURCE` | Refund was refused by the payment source. Refunds are not supported for the payer's selected payment source. Contact the payer directly to arrange a refund via alternative means. |
| `REFUND_REFUSED_BY_PROCESSOR` | The funding instrument linked to the account has been declined by either the processor or PayPal internal system. |
| `REFUND_TIME_EXCEEDED_FOR_PAYMENT_SOURCE` | Refund was refused by the payment source. The time limit set by the payer's selected payment source to refund this transaction has expired. Contact the payer directly to arrange a refund via alternative means. |
| `REFUND_TIME_LIMIT_EXCEEDED` | You are over the time limit to perform a refund on this capture. The refund cannot be issued at this time. |
| `REFUND_TRANSACTION_REFUSED` | PayPal's internal controls or user account settings prevent refund from being processed. |
| `SHIPPING_ADDRESS_INVALID` | Address field does not match the corresponding validation regex. |
| `SHIPPING_ADDRESS_NOT_ALLOWED_BY_RESIDENCE_COUNTRY` | The provided shipping address is not allowed by buyer's residency country. |
| `TRANSACTION_DISPUTED` | Partial refunds cannot be offered at this time because there is an open case on this transaction. Visit the PayPal Resolution Center to review this case. Refund for an amount less than the remaining transaction amount cannot be processed at this time because of an open dispute on the capture. ... |
| `TRANSACTION_REFUSED` | PayPal's internal controls prevent authorization from being captured. For more information about this transaction, contact customer support. |
| `UPDATE_AUTHORIZATION_NOT_SUPPORTED` | Update authorization is not allowed for this type of authorization. |
