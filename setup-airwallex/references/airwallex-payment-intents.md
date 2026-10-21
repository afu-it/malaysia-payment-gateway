# Airwallex Payment Intents

Field tables below are generated from the official API reference pages (checked 2026-10-21). "Required" is as documented; "no" means optional. Nested objects with many rarely used fields are listed at the top level only; open the source page for the full schema. All calls need `Authorization: Bearer <token>`.

## Source URLs

- Resource overview: `https://www.airwallex.com/docs/api/payments/payment_intents`
- Create: `https://www.airwallex.com/docs/api/payments/payment_intents/create`
- Retrieve: `https://www.airwallex.com/docs/api/payments/payment_intents/retrieve`
- List: `https://www.airwallex.com/docs/api/payments/payment_intents/list`
- Update: `https://www.airwallex.com/docs/api/payments/payment_intents/update`
- Confirm: `https://www.airwallex.com/docs/api/payments/payment_intents/confirm`
- Confirm continue: `https://www.airwallex.com/docs/api/payments/payment_intents/confirm_continue`
- Capture: `https://www.airwallex.com/docs/api/payments/payment_intents/capture`
- Cancel: `https://www.airwallex.com/docs/api/payments/payment_intents/cancel`
- Increment authorization: `https://www.airwallex.com/docs/api/payments/payment_intents/increment_authorization`
- Payment Attempts: `https://www.airwallex.com/docs/api/payments/payment_attempts`
- Guide: `https://www.airwallex.com/docs/payments/get-started/using-payments-intent-api`
- Statuses: `https://www.airwallex.com/docs/payments/reference/payment-statuses`
- Native API guest checkout: `https://www.airwallex.com/docs/payments/integration-options/web-checkout/native-api/guest-user-checkout`
- 3DS (API versions on or after 2024-06-14): `https://www.airwallex.com/docs/payments/integration-options/web-checkout/native-api/3d-secure-authentication/native-api-(for-api-versions-on-or-after-2024-06-14)`
- Place a hold: `https://www.airwallex.com/docs/payments/payment-operations/manage-payments/place-a-hold-on-a-payment-method`

## Model

- A Payment Intent is the order ("customer wants to pay RM100"). It holds one or more Payment Attempts (each try). The intent status normally follows the latest attempt.
- Create with `amount`, `currency`, `merchant_order_id`, `request_id`. Passing `payment_method` on create confirms the intent immediately.
- With Airwallex.js (HPP, Drop-in, Elements) the SDK confirms for you; you do not call confirm from the server.
- With Native API you call confirm yourself with `payment_method` and handle `next_action`.

## Handling `next_action`

| `next_action.type` | What to do |
| --- | --- |
| `redirect` | Send the shopper to `next_action.url` with `next_action.method` (`GET` or `POST`; for `POST` use `next_action.data` and `content_type`, default `application/json`). For `mobile_app` flows `fallback_url` and `package_name` (Android) may be returned. |
| `render_qrcode` | Render `next_action.qrcode` text as a QR (for `tng` with `flow: "qrcode"`, also `wechatpay`, `alipaycn`, `kakaopay`, and others). A `url` may also be returned. |
| `redirect_iframe` | Card 3DS: render an iframe with `src = next_action.url` (`method` `GET`). Recommended sizes 600x400, 500x600, 390x400, 250x400. |
| `call_sdk` | Hand off to the method's SDK (method specific). |
| `request_pin` | Card-present PIN entry (POS). |

After the redirect or QR, the intent stays `REQUIRES_CUSTOMER_ACTION` until the provider reports back. Wait for the webhook; poll retrieve after the shopper returns if needed.

## 3D Secure (Native API, API version 2024-06-14 or later)

- Override per payment with `payment_method_options.card.three_ds_action`: `FORCE_3DS`, `EXTERNAL_3DS`, `SKIP_3DS`, or `null` (account default). `EXTERNAL_3DS` and `SKIP_3DS` only work if Airwallex enabled them on your account, else the request is rejected.
- Provide `device_data` on confirm and a Customer with name, email, billing to raise frictionless rates.
- On `REQUIRES_CUSTOMER_ACTION` with `next_action.type = redirect_iframe`, render the iframe. When authentication ends the iframe goes to your return URL with query params `payment_intent_id`, `client_secret`, `succeeded`, `error_code`, `error_message`; re-check status server-side.
- API versions before 2022-11-11 returned `redirect_form` (hidden iframe POST of `next_action.data` as `application/x-www-form-urlencoded`). A separate legacy guide covers versions before 2024-06-14.
- HPP, Drop-in, and Elements run 3DS automatically.

## Manual capture (holds)

- Set `payment_method_options.card.auto_capture: false` on confirm (or `autoCapture: false` in Airwallex.js). The intent goes to `REQUIRES_CAPTURE`.
- Capture with `POST /api/v1/pa/payment_intents/{id}/capture` (`amount` may be less than authorized; never more).
- Hold windows: cards usually 7 days; Visa up to 30 days and Mastercard up to 30 days with `authorization_type: "pre_auth"` (Visa 30-day only for cruise, lodging, vehicle rental; POS 5 days); Klarna 28 days. Holds are supported only for cards (including Apple Pay and Google Pay) and Klarna, not for FPX, DuitNow, or Malaysian e-wallets, and not for Payment Links.
- Release an uncaptured authorization with cancel.

## Cancel

`POST /api/v1/pa/payment_intents/{id}/cancel` with `request_id` and optional `cancellation_reason`. Only for intents not yet captured; the full amount is cancelled; a webhook (`payment_intent.cancelled`) follows.

## Endpoint field tables
### Create a Payment Intent

`POST /api/v1/pa/payment_intents/create`

Start collecting payment from your customer by creating a Payment Intent. The payment `amount` and `currency`, as well as the merchant order unique identifier must be provided.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `amount` | number | yes | Payment amount. This is the order amount you would like to charge your customer. |
| `currency` | string | yes | Payment currency in 3-letter ISO 4217 currency code. Please refer to supported currencies. |
| `merchant_order_id` | string | yes | The order unique identifier created in merchant's order system that corresponds to this Payment Intent. Maximum length is 64. |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `additional_info` | object | no | Additional info for the transaction. |
| `connected_account_id` | string | no | Airwallex assigned account identifier of the connected entity(another account which is linked with platform). It is to be used by the platform to indicate the connected entity in the transaction where platform is the owner of transaction. |
| `customer` | object | no | Details of the customer making a payment for this Payment Intent. When `customer_id` is provided in the request you must leave this object empty unless you want to provide `additional_info` related to the customer for this payment. |
| `customer.address` | object | no | Address of the customer. |
| `customer.business_name` | string | no | Business name of the customer. |
| `customer.email` | string | no | Email address of the customer. |
| `customer.first_name` | string | no | First name of the customer. |
| `customer.last_name` | string | no | Last name of the customer. |
| `customer.merchant_customer_id` | string | no | Unique identifier of the customer in merchant's system. Maximum length is 64. |
| `customer.phone_number` | string | no | Phone number of the customer. |
| `customer_id` | string | no | The unique identifier of the customer making a payment for this Payment Intent. This field is required if the Payment Intent is created for recurring payments. |
| `descriptor` | string | no | Descriptor that will display to the customer. Maximum length is 32. |
| `device_data` | object | no | Client device information applicable when `payment_method` is set. It is recommended to provide this data for risk analysis. |
| `external_recurring_data` | object | no | Some merchants prefer to maintain the recurring payments on their own instead of using **Payment Consent**. Providing this data indicates the request is an external recurring payment. Only effective when the `payment_method.type` is `car... |
| `funds_split_data` | array[object] | no | When provided, it creates specified funds splits that are automatically released after the Payment Intent succeeds. |
| `merchant_category_code` | string | no | Four-digit code to categorize the business or service. This field is only applicable for approved merchants to specify an alternative category code. |
| `merchant_website_url` | string | no | URL of the merchant's website where the customer is completing the transaction. |
| `metadata` | object | no | A set of key-value pairs that you can attach to the Payment Intent. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `order` | object | no | Purchase order related to this Payment Intent. |
| `order.type` | string | no | Industry category of the order. Maximum of 128 characters. |
| `payment_consent` | object | no | The associated **Payment Consent** to set up along with the Payment Intent. Only effective when the `payment_method.type` is `card`. |
| `payment_method` | object | no | The payment method details to confirm the Payment Intent. The Payment Intent will be confirmed automatically when `payment_method` is set. |
| `payment_method_options` | object | no | Options for Payment Method. |
| `payment_method_options.card` | object | no | The Payment Method options for card. |
| `return_url` | string | no | The web page URL or application scheme URI to redirect the customer after payment authentication. |
| `risk_control_options` | object | no | Risk Control Options. |
| `risk_control_options.skip_risk_processing` | boolean | no | Set it to `true` if you want to skip fraud processing. Please contact your account manager to enable the feature to take effect. |
| `risk_control_options.tra_applicable` | boolean | no | Set it to 'true' if you want to use TRA exemption when create payment intent, only applicable when you skip risk processing. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `additional_info` | object | no | Additional info for the transaction. |
| `amount` | number | no | Payment amount. This is the order amount you would like to charge your customer. |
| `cancellation_reason` | string | no | Reason for cancelling the Payment Intent. Only present when the Payment Intent was successfully cancelled, i.e. status is `CANCELLED`. |
| `cancelled_at` | string | no | Last time at which this Payment Intent was cancelled. Only present when the Payment Intent was successfully cancelled, i.e. status is `CANCELLED`. |
| `captured_amount` | number | no | Amount that captured from this Payment Intent. |
| `client_secret` | string | no | Payment Intent's client secret for browser or app. Returned by Payment Intent create API or Payment Intent retrieve API The provided client_secret is valid for 60 minutes. |
| `connected_account_id` | string | no | Account identifier of the connected account. |
| `conversion_quote_id` | string | no | The unique identifier of the conversion quote applied to the intent. |
| `created_at` | string | no | Time at which this Payment Intent was created. |
| `currency` | string | no | Amount currency. |
| `customer` | object | no | The details of the customer who is making a business payment for this Payment Intent, in which case it's not desired to create a Customer API resource. |
| `customer_id` | string | no | The unique identifier of the Customer who is paying for this Payment Intent. |
| `descriptor` | string | no | Descriptor that will be displayed to the customer. |
| `funds_split_data` | array[object] | no | The data that will create funds splits automatically after the Payment Intent succeeds. |
| `id` | string | no | Unique identifier for the Payment Intent. |
| `invoice_id` | string | no | Invoice unique identifier. |
| `latest_payment_attempt` | object | no | Latest Payment Attempt that was created under the Payment Intent. |
| `merchant_category_code` | string | no | Four-digit code to categorize the business or service. This field is only applicable for approved merchants to specify an alternative category code. |
| `merchant_order_id` | string | no | The order unique identifier created in merchant's order system that corresponds to this Payment Intent. |
| `merchant_website_url` | string | no | URL of the merchant's website where the customer is completing the transaction. |
| `metadata` | object | no | A set of key-value pairs that you can attach to this Payment Intent. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `next_action` | object | no | Next action for merchant. |
| `order` | object | no | Purchase order related to this Payment Intent. |
| `payment_consent` | object | no | The associated **Payment Consent** to set up along with the Payment Intent. |
| `payment_consent_id` | string | no | Unique identifier of the Payment Consent related to this Payment Intent. |
| `payment_link_id` | string | no | Payment Link unique identifier (if this intent was created by Payment Link). |
| `payment_method_options` | object | no | Options for Payment Method. |
| `request_id` | string | no | Unique request identifier specified by the merchant in the last operation. |
| `return_url` | string | no | The web page URL or application scheme URI to redirect the customer after payment authentication. |
| `risk_control_options` | object | no | Risk Control Options. |
| `status` | string | no | Please note that this list of statuses is not exhaustive, you should be prepared to handle intent statuses beyond those explicitly mentioned. - **REQUIRES_PAYMENT_METHOD**: The Payment Intent is waiting for the confirm request. - This st... Values: `REQUIRES_PAYMENT_METHOD`, `REQUIRES_CUSTOMER_ACTION`, `PENDING_REVIEW`, `REQUIRES_CAPTURE`, `PENDING`, `SUCCEEDED`, `CANCELLED` |
| `triggered_by` | string | no | Indicates who initiated the payment: the `merchant` or the `customer`. - `merchant`: The payment is initiated by the merchant to collect a subsequent payment where the customer is not in session. - `customer`: The payment is initiated b... |
| `updated_at` | string | no | Last time at which this Payment Intent was updated or operated on. |

### Retrieve a Payment Intent

`GET /api/v1/pa/payment_intents/{id}`

Retrieve a Payment Intent by unique identifier.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Intent unique identifier. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `additional_info` | object | no | Additional info for the transaction. |
| `amount` | number | no | Payment amount. This is the order amount you would like to charge your customer. |
| `cancellation_reason` | string | no | Reason for cancelling the Payment Intent. Only present when the Payment Intent was successfully cancelled, i.e. status is `CANCELLED`. |
| `cancelled_at` | string | no | Last time at which this Payment Intent was cancelled. Only present when the Payment Intent was successfully cancelled, i.e. status is `CANCELLED`. |
| `captured_amount` | number | no | Amount that captured from this Payment Intent. |
| `client_secret` | string | no | Payment Intent's client secret for browser or app. Returned by Payment Intent create API or Payment Intent retrieve API The provided client_secret is valid for 60 minutes. |
| `connected_account_id` | string | no | Account identifier of the connected account. |
| `conversion_quote_id` | string | no | The unique identifier of the conversion quote applied to the intent. |
| `created_at` | string | no | Time at which this Payment Intent was created. |
| `currency` | string | no | Amount currency. |
| `customer` | object | no | The details of the customer who is making a business payment for this Payment Intent, in which case it's not desired to create a Customer API resource. |
| `customer_id` | string | no | The unique identifier of the Customer who is paying for this Payment Intent. |
| `descriptor` | string | no | Descriptor that will be displayed to the customer. |
| `funds_split_data` | array[object] | no | The data that will create funds splits automatically after the Payment Intent succeeds. |
| `id` | string | no | Unique identifier for the Payment Intent. |
| `invoice_id` | string | no | Invoice unique identifier. |
| `latest_payment_attempt` | object | no | Latest Payment Attempt that was created under the Payment Intent. |
| `latest_payment_attempt.acquirer_reference_number` | string | no | Acquirer reference number of the card payment. Will be populated after the payment attempt has transitioned to the `CAPTURE_REQUESTED` or `SETTLED` status. |
| `latest_payment_attempt.amount` | number | no | Amount of the Payment Attempt. |
| `latest_payment_attempt.authentication_data` | object | no | Authentication data which contains AVS result, CVC result, three domain secure result and fraud evaluation result. |
| `latest_payment_attempt.authorization_code` | string | no | Authorization response code from issuer when the payment is authorized successfully. |
| `latest_payment_attempt.captured_amount` | number | no | Captured amount. |
| `latest_payment_attempt.created_at` | string | no | Time at which this Payment Attempt was created. |
| `latest_payment_attempt.currency` | string | no | Currency of the Payment Attempt. |
| `latest_payment_attempt.dcc_data` | object | no | Present when DCC amount is available and DCC currency is different from Payment Intent currency. DCC could be conducted either by third party or Airwallex when using Airwallex DCC product. |
| `latest_payment_attempt.failure_code` | string | no | Payment Attempt failure code. One of `authentication_failed`, `capture_failed`, `authorization_failed`, `provider_unavailable`, `system_unavailable`, `fraud_rejected`. Values: `authentication_failed`, `capture_failed`, `authorization_failed`, `provider_unavailable`, `system_unavailable`, `fraud_rejected` |
| `latest_payment_attempt.failure_details` | object | no | Failure details. Only applicable when status is FAILED. |
| `latest_payment_attempt.id` | string | no | Unique identifier for the Payment Attempt. |
| `latest_payment_attempt.merchant_advice_code` | string | no | An advice code from issuer to provide more details about the payment. It can be useful when implementing a retry logic for declined payments. |
| `latest_payment_attempt.merchant_order_id` | string | no | Merchant order unique identifier of the Payment Attempt. |
| `latest_payment_attempt.payment_consent_id` | string | no | Payment Consent unique identifier of the Payment Attempt. |
| `latest_payment_attempt.payment_intent_id` | string | no | Payment Intent unique identifier of the Payment Attempt. |
| `latest_payment_attempt.payment_method` | object | no | Payment Method used by the Payment Attempt. |
| `latest_payment_attempt.payment_method_options` | object | no | Options for Payment Method. |
| `latest_payment_attempt.payment_method_transaction_id` | string | no | The unique transaction identifier for a Payment Attempt, assigned by the payment provider (card issuers or local payment method providers such as Google Pay, i DEAL, etc.). |
| `latest_payment_attempt.provider_original_response_code` | string | no | The original response code from the payment method provider (or issuer for card). |
| `latest_payment_attempt.provider_original_response_description` | string | no | Human-readable description of `provider_original_response_code`, describing the reason returned by the issuer. |
| `latest_payment_attempt.provider_transaction_id` | string | no | Deprecated. Use `payment_method_transaction_id` instead. |
| `latest_payment_attempt.refunded_amount` | number | no | Refunded amount. |
| `latest_payment_attempt.settle_via` | string | no | Through which channel the merchant will get this payment settled. Possible values are `airwallex`, `paypal`, `amex`. Values: `airwallex`, `paypal`, `amex` |
| `latest_payment_attempt.status` | string | no | - **RECEIVED**: The Payment Attempt has been created upon request. - **AUTHENTICATION_REDIRECTED**: The Payment Attempt is waiting for further customer action of authentication, e.g. 3DS verification and QR code scan. - **PENDING_AUTHORI... Values: `RECEIVED`, `AUTHENTICATION_REDIRECTED`, `PENDING_AUTHORIZATION`, `AUTHORIZED`, `PENDING_REVIEW`, `CAPTURE_REQUESTED`, `EXPIRED`, `CANCELLED`, `FAILED`, `SETTLED`, `PAID` |
| `latest_payment_attempt.terminal_id` | string | no | Airwallex terminal unique identifier used to process the payment attempt. |
| `latest_payment_attempt.updated_at` | string | no | Last time at which this Payment Attempt was updated or operated on. |
| `merchant_category_code` | string | no | Four-digit code to categorize the business or service. This field is only applicable for approved merchants to specify an alternative category code. |
| `merchant_order_id` | string | no | The order unique identifier created in merchant's order system that corresponds to this Payment Intent. |
| `merchant_website_url` | string | no | URL of the merchant's website where the customer is completing the transaction. |
| `metadata` | object | no | A set of key-value pairs that you can attach to this Payment Intent. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `next_action` | object | no | Next action for merchant. |
| `next_action.content_type` | string | no | The content type of the request when `method` is `POST`. If not provided, the content type should be `application/json`. |
| `next_action.data` | object | no | The additional data that can be used to complete this action. |
| `next_action.dcc_data` | object | no | The DCC data that can be used to complete this action. |
| `next_action.fallback_url` | string | no | Fallback url to redirect the shopper to the web page if the redirection to the app fails. Returned only if the flow is `mobile_app`. |
| `next_action.method` | string | no | The redirect method if the action type is `redirect`. One of `GET`, `POST`. Values: `GET`, `POST` |
| `next_action.package_name` | string | no | Android package name. Returned only if the flow is `mobile_app` and the os_type is `android`. |
| `next_action.qrcode` | string | no | QR Code text representation if the action type is `render_qrcode`, only applicable to `wechatpay`, `alipaycn`, `alipayhk`, `kakaopay`, `gcash`, `tng`, `truemoney`, `rabbit_line_pay`, `fps`. |
| `next_action.type` | string | no | Type of next action. One of `render_qrcode`, `call_sdk`, `redirect`, `redirect_iframe`, `request_pin`. Values: `render_qrcode`, `call_sdk`, `redirect`, `redirect_iframe`, `request_pin` |
| `next_action.url` | string | no | The redirect url. |
| `order` | object | no | Purchase order related to this Payment Intent. |
| `payment_consent` | object | no | The associated **Payment Consent** to set up along with the Payment Intent. |
| `payment_consent_id` | string | no | Unique identifier of the Payment Consent related to this Payment Intent. |
| `payment_link_id` | string | no | Payment Link unique identifier (if this intent was created by Payment Link). |
| `payment_method_options` | object | no | Options for Payment Method. |
| `request_id` | string | no | Unique request identifier specified by the merchant in the last operation. |
| `return_url` | string | no | The web page URL or application scheme URI to redirect the customer after payment authentication. |
| `risk_control_options` | object | no | Risk Control Options. |
| `status` | string | no | Please note that this list of statuses is not exhaustive, you should be prepared to handle intent statuses beyond those explicitly mentioned. - **REQUIRES_PAYMENT_METHOD**: The Payment Intent is waiting for the confirm request. - This st... Values: `REQUIRES_PAYMENT_METHOD`, `REQUIRES_CUSTOMER_ACTION`, `PENDING_REVIEW`, `REQUIRES_CAPTURE`, `PENDING`, `SUCCEEDED`, `CANCELLED` |
| `triggered_by` | string | no | Indicates who initiated the payment: the `merchant` or the `customer`. - `merchant`: The payment is initiated by the merchant to collect a subsequent payment where the customer is not in session. - `customer`: The payment is initiated b... |
| `updated_at` | string | no | Last time at which this Payment Intent was updated or operated on. |

### List all Payment Intents

`GET /api/v1/pa/payment_intents`

List all Payment Intents.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `connected_account_id` | string | no | Airwallex assigned account identifier of the connected entity. Only applicable for platforms. |
| `currency` | string | no | Payment Intent currency. |
| `from_created_at` | string | no | The start time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `merchant_order_id` | string | no | Merchant Order unique identifier. |
| `page_num` | integer | no | Page number starting from 0. |
| `page_size` | integer | no | Number of Payment Intents to be listed per page. Default value is 10. |
| `payment_consent_id` | string | no | Payment Consent unique identifier. |
| `status` | string | no | Payment Intent status. |
| `to_created_at` | string | no | The end time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |

### Update a Payment Intent

`POST /api/v1/pa/payment_intents/{id}/update`

Update properties on a Payment Intent. Some properties are only allowed to be updated in `REQUIRES_PAYMENT_METHOD` or `REQUIRES_CUSTOMER_ACTION` status. Depending on which properties you update and the status of Payment Intent, you might need to confirm the Payment Intent again. For example, updating the `currency` and `amount` in `REQUIRES_CUSTOMER_ACTION` status always requires you to confirm the Payment Intent again. The update behavior follows these principles: 1\. Omitted fields in the request payload will be ignored and have no impact on the existing value. 2\. Provided fields in the ...

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Intent unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `additional_info` | object | no | Additional info for the transaction. |
| `amount` | number | no | Payment amount. This is the order amount you would like to charge your customer. |
| `conversion_quote_id` | string | no | The unique identifier of the conversion quote to be applied to the intent. |
| `currency` | string | no | Payment currency. Please refer to supported currencies. |
| `customer` | object | no | Details of the customer making a payment for this Payment Intent. When `customer_id` is provided in the request you must leave this object empty unless you want to provide `additional_info` related to the customer for this payment. |
| `customer_id` | string | no | The unique identifier of the customer making a payment for this Payment Intent. This field is required if the Payment Intent is created for recurring payments. |
| `descriptor` | string | no | Descriptor that will display to the customer. Maximum length is 32. |
| `merchant_order_id` | string | no | The order unique identifier created in merchant's order system that corresponds to this Payment Intent. Maximum length is 64. |
| `merchant_website_url` | string | no | URL of the merchant's website where the customer is completing the transaction. |
| `metadata` | object | no | A set of key-value pairs that you can attach to the Payment Intent. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `order` | object | no | Purchase order related to the Payment Intent. |
| `return_url` | string | no | The web page URL or application scheme URI to redirect the customer after payment authentication. |

### Confirm a Payment Intent

`POST /api/v1/pa/payment_intents/{id}/confirm`

Call this endpoint when your customer is ready to make a payment as per the details in the Payment Intent. At a minimum, a Payment Method must be set as the funding source of the payment. This is done by providing a `payment_method` object when confirming the Payment Intent.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Intent unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `customer_id` | string | no | Customer who intends to pay for the Payment Intent. |
| `device_data` | object | no | Client device information. It is recommended to provide this data to increase the odds of having frictionless 3DS. |
| `device_data.accept_header` | string | no | Accept header value. |
| `device_data.browser` | object | no | Browser specific data. |
| `device_data.browser.java_enabled` | boolean | no | Is Java enabled? |
| `device_data.browser.javascript_enabled` | boolean | no | Is JavaScript enabled? |
| `device_data.browser.user_agent` | string | no | User Agent header value. |
| `device_data.device_id` | string | no | Device unique identifier or Advertisement unique identifier or IMEI. |
| `device_data.ip_address` | string | no | Public IP address, both IPv4 and IPv6 are supported. |
| `device_data.language` | string | no | Language or locale. |
| `device_data.location` | object | no | Location data. |
| `device_data.location.lat` | string | no | Latitude. |
| `device_data.location.lon` | string | no | Longitude. |
| `device_data.mobile` | object | no | Mobile specific data. |
| `device_data.mobile.device_model` | string | no | Device model. |
| `device_data.mobile.os_type` | string | no | OS type. |
| `device_data.mobile.os_version` | string | no | OS version. |
| `device_data.screen_color_depth` | integer | no | Screen color depth in bits. |
| `device_data.screen_height` | integer | no | Screen height in pixels. |
| `device_data.screen_width` | integer | no | Screen width in pixels. |
| `device_data.timezone` | string | no | Timezone offset to UTC in hours, with added minutes if necessary. |
| `external_recurring_data` | object | no | Some merchants prefer to maintain the recurring payments on their own instead of using **Payment Consent**. Providing this data indicates the request is an external recurring payment. Only effective when `payment_consent_id` is not prese... Values: `card`, `applepay`, `googlepay` |
| `funds_split_data` | array[object] | no | When provided, it creates specified funds splits that are automatically released after the Payment Intent succeeds. |
| `payment_consent` | object | no | The associated **Payment Consent** to set up along with the Payment Intent. |
| `payment_consent_id` | string | no | Unique identifier of the Payment Consent referenced for this recurring payment. |
| `payment_method` | object | no | The **Payment Method** to use for the Payment Intent. |
| `payment_method.applepay` | object | no | Applepay information. This must be provided if `type` is set to `applepay`. |
| `payment_method.applepay.billing` | object | no | Billing information. |
| `payment_method.applepay.encrypted_payment_token` | string | no | Payment data from Apple Pay Payment Token. |
| `payment_method.applepay.payment_data_type` | string | no | Type of the payment data details. One of `tokenized_card` or `encrypted_payment_token`. Values: `tokenized_card`, `encrypted_payment_token` |
| `payment_method.applepay.tokenized_card` | object | no | Payment data with the tokenized card details. |
| `payment_method.atome` | object | no | Atome information. This must be provided if `type` is set to `atome`. |
| `payment_method.atome.shopper_phone` | string | no | Customer's mobile number, formatted in E.164 standard. |
| `payment_method.boost` | object | no | Boost information. It takes effect when `type` is set to `boost`. |
| `payment_method.boost.shopper_email` | string | no | Email address of the shopper. |
| `payment_method.boost.shopper_name` | string | no | The name of the shopper. |
| `payment_method.boost.shopper_phone` | string | no | Phone number of the shopper. |
| `payment_method.card` | object | no | Card information. This must be provided if `type` is set to `card`. |
| `payment_method.card.additional_info` | object | no | Additional information of external network token requestor. Should be provided when number_type is `EXTERNAL_NETWORK_TOKEN`. |
| `payment_method.card.billing` | object | no | Billing information. |
| `payment_method.card.cryptogram` | string | no | The cryptogram value. Must be provided if card.number_type is `EXTERNAL_NETWORK_TOKEN`. |
| `payment_method.card.cvc` | string | no | CVC code of this card. Required for all transactions except merchant initiated transactions and network tokenized transactions. |
| `payment_method.card.expiry_month` | string | no | Two digit number representing the card’s expiration month. |
| `payment_method.card.expiry_year` | string | no | Four digit number representing the card’s expiration year. |
| `payment_method.card.external_three_ds` | object | no | Authentication data from an external 3D-Secure provider (3RI for MIT subsequent transaction). |
| `payment_method.card.korean_card` | object | no | Information for Korean Card. |
| `payment_method.card.name` | string | no | Card holder name. Maximum length is 128. |
| `payment_method.card.number` | string | no | Card number. |
| `payment_method.card.number_type` | string | no | Type of the number. One of `PAN`, `EXTERNAL_NETWORK_TOKEN`. Values: `PAN`, `EXTERNAL_NETWORK_TOKEN` |
| `payment_method.duit_now` | object | no | Duit Now information. It takes effect when `type` is set to `duit_now`. |
| `payment_method.duit_now.shopper_name` | string | no | The name of the shopper. |
| `payment_method.fpx` | object | no | FPX Transfer information. This must be provided if `type` is set to `fpx`. |
| `payment_method.fpx.bank_name` | string | yes | The name of the bank. One of `affin`, `agrobank`, `alliance`, `ambank`, `bank_islam`, `muamalat`, `rakyat`, `bsn`, `cimb_clicks`, `hong_leong`, `hsbc`, `kfh`, `maybank2u`, `ocbc`, `public_bank`, `rhb`, `standard_chartered`, `uob`. Values: `affin`, `agrobank`, `alliance`, `ambank`, `bank_islam`, `muamalat`, `rakyat`, `bsn`, `cimb_clicks`, `hong_leong`, `hsbc`, `kfh`, `maybank2u`, `ocbc` ... |
| `payment_method.fpx.shopper_email` | string | no | Email address of the shopper. |
| `payment_method.fpx.shopper_name` | string | no | The name of the shopper. |
| `payment_method.fpx.shopper_phone` | string | no | Phone number of the shopper. |
| `payment_method.googlepay` | object | no | Googlepay information. This must be provided if `type` is set to `googlepay`. |
| `payment_method.googlepay.billing` | object | no | Billing information. |
| `payment_method.googlepay.encrypted_payment_token` | string | no | A UTF-8 encoded, serialized JSON object with the keys specified. |
| `payment_method.googlepay.external_three_ds` | object | no | Authentication data from an external 3D-Secure provider. Only applicable when Google Pay `PAN_ONLY` authentication method is used. |
| `payment_method.googlepay.payment_data_type` | string | no | Type of the payment data details.  One of `tokenized_card` or `encrypted_payment_token`. Values: `tokenized_card`, `encrypted_payment_token` |
| `payment_method.googlepay.tokenized_card` | object | no | Payment data with the tokenized card details. |
| `payment_method.grabpay` | object | no | Grab Pay information. It takes effect when `type` is set to `grabpay`. |
| `payment_method.grabpay.shopper_name` | string | no | The name of the shopper. |
| `payment_method.id` | string | no | The unique identifier of the payment method. If this field is provided, you must also include either `payment_consent` or `triggered_by`. |
| `payment_method.seven_eleven` | object | no | 7-Eleven information. It takes effect when if `type` is set to `seven_eleven`. |
| `payment_method.seven_eleven.shopper_email` | string | no | Email address of the shopper. |
| `payment_method.seven_eleven.shopper_name` | string | no | The name of the shopper. |
| `payment_method.seven_eleven.shopper_phone` | string | no | Phone number of the shopper. |
| `payment_method.shopee_pay` | object | no | Shopee Pay information. It takes effect when `type` is set to `shopee_pay`. |
| `payment_method.shopee_pay.shopper_email` | string | no | Email address of the shopper. |
| `payment_method.shopee_pay.shopper_name` | string | no | The name of the shopper. |
| `payment_method.shopee_pay.shopper_phone` | string | no | Phone number of the shopper. |
| `payment_method.tng` | object | no | Tng information. This must be provided if `type` is set to `tng`. |
| `payment_method.tng.flow` | string | yes | One of `qrcode`, `mobile_web`, `mobile_app`. Values: `qrcode`, `mobile_web`, `mobile_app` |
| `payment_method.tng.os_type` | string | no | Required when `flow` is `mobile_web` or `mobile_app`. One of `ios`, `android`. Values: `ios`, `android` |
| `payment_method.type` | string | no | Type of the Payment Method. One of `card`, `applepay`, `wechatpay`, `alipaycn`, `alipayhk`, `googlepay`, `gcash`, `dana`, `kakaopay`, `tng`, `truemoney`, `rabbit_line_pay`, `family_mart`, `hi_life`, `esun`, `boost`, `shopee_pay`, `fpx`, ... Values: `card`, `applepay`, `wechatpay`, `alipaycn`, `alipayhk`, `googlepay`, `gcash`, `dana`, `kakaopay`, `tng`, `truemoney`, `rabbit_line_pay`, `family_mart`, `hi_life` ... |
| `payment_method_options` | object | no | Options for Payment Method. |
| `payment_method_options.card` | object | no | The Payment Method options for card. |
| `payment_method_options.card.authorization_type` | string | no | The authorization type of the card payment. One of `final_auth`, `pre_auth`. Values: `final_auth`, `pre_auth` |
| `payment_method_options.card.auto_capture` | boolean | no | Specifies whether the funds should be requested automatically after the payment is authorized. Default to `true`. |
| `payment_method_options.card.merchant_trigger_reason` | string | no | Types of transactions where the merchant is allowed to initiate payments because it is a customary, legitimate need of the merchant’s industry. Only applicable for merchant-initiated transactions. Values: `delayed_charges`, `no_show`, `partial_shipment`, `reauthorization`, `resubmission` |
| `return_url` | string | no | The web page URL or application scheme URI to redirect the customer after payment authentication or cancellation. |
| `triggered_by` | string | no | Indicates who initiated the payment: the `merchant` or the `customer`. - `merchant`: The payment is initiated by the merchant to collect a subsequent payment where the customer is not in session. - `customer`: The payment is initiated b... |

### Continue To Confirm a Payment Intent

`POST /api/v1/pa/payment_intents/{id}/confirm_continue`

In certain scenarios (i.e. 3DS and DCC), customer is required to provide additional information multiple times after the initial confirmation request.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Intent unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `card_pin_data` | object | no | PIN data to continue a card-present transaction, required only if type = `submit_card_pin_data`. |
| `card_pin_data.encrypted_pin` | string | yes | Encrypted personal identification number. |
| `card_pin_data.key_serial_number` | string | no | Key serial number while using DUKPT for PIN encryption. |
| `micro_deposit` | object | no | The verification request of micro-deposit. |
| `micro_deposit.amounts` | array[number] | no | The amounts of the micro-deposit in the shopper’s bank statement. Please refer to supported currencies for supported minor units. |
| `type` | string | no | Type of the continue type. Possible values: - `submit_card_pin_data` - `verify_micro_deposits` Values: `submit_card_pin_data`, `verify_micro_deposits` |

### Capture a Payment Intent

`POST /api/v1/pa/payment_intents/{id}/capture`

Capture funds that have been confirmed but have not yet been captured or have only been captured partially. A Payment Intent is capturable if `auto_capture=false` and its `next_action` indicates that capture is required.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Intent unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `amount` | number | no | Amount to capture. Please refer to supported currencies for supported minor units. |
| `funds_split_data` | array[object] | no | When provided, it creates specified funds splits that are automatically released after the Payment Intent succeeds. |

### Cancel a Payment Intent

`POST /api/v1/pa/payment_intents/{id}/cancel`

A Payment Intent can be cancelled when it is in one of these statuses: `REQUIRES_PAYMENT_METHOD`, `REQUIRES_CUSTOMER_ACTION`, `REQUIRES_CAPTURE`. Any outstanding, un-captured funds will be refunded once cancelled.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Intent unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `cancellation_reason` | string | no | The reason for canceling the Payment Intent. |

### Increment Authorization for a Payment Intent

`POST /api/v1/pa/payment_intents/{id}/increment_authorization`

Allows you to authorize additional funds on an existing Payment Intent before capture. This is useful when the final payment amount is not known upfront, such as in hotel or car rental scenarios. Only a single capture and clearing occurs for the final authorized amount.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Intent unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `amount` | number | yes | The new total amount to authorize, which must be greater than the previously authorized total amount on this Payment Intent. Please refer to supported currencies for supported minor units. |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |

### Retrieve a Payment Attempt

`GET /api/v1/pa/payment_attempts/{id}`



Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Attempt unique identifier. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `acquirer_reference_number` | string | no | Acquirer reference number of the card payment. Will be populated after the payment attempt has transitioned to the `CAPTURE_REQUESTED` or `SETTLED` status. |
| `amount` | number | no | Amount of the Payment Attempt. |
| `authentication_data` | object | no | Authentication data which contains AVS result, CVC result, three domain secure result and fraud evaluation result. |
| `authorization_code` | string | no | Authorization response code from issuer when the payment is authorized successfully. |
| `captured_amount` | number | no | Captured amount. |
| `created_at` | string | no | Time at which this Payment Attempt was created. |
| `currency` | string | no | Currency of the Payment Attempt. |
| `dcc_data` | object | no | Present when DCC amount is available and DCC currency is different from Payment Intent currency. DCC could be conducted either by third party or Airwallex when using Airwallex DCC product. |
| `failure_code` | string | no | Payment Attempt failure code. One of `authentication_failed`, `capture_failed`, `authorization_failed`, `provider_unavailable`, `system_unavailable`, `fraud_rejected`. Values: `authentication_failed`, `capture_failed`, `authorization_failed`, `provider_unavailable`, `system_unavailable`, `fraud_rejected` |
| `failure_details` | object | no | Failure details. Only applicable when status is FAILED. |
| `id` | string | no | Unique identifier for the Payment Attempt. |
| `merchant_advice_code` | string | no | An advice code from issuer to provide more details about the payment. It can be useful when implementing a retry logic for declined payments. |
| `merchant_order_id` | string | no | Merchant order unique identifier of the Payment Attempt. |
| `payment_consent_id` | string | no | Payment Consent unique identifier of the Payment Attempt. |
| `payment_intent_id` | string | no | Payment Intent unique identifier of the Payment Attempt. |
| `payment_method` | object | no | Payment Method used by the Payment Attempt. |
| `payment_method_options` | object | no | Options for Payment Method. |
| `payment_method_transaction_id` | string | no | The unique transaction identifier for a Payment Attempt, assigned by the payment provider (card issuers or local payment method providers such as Google Pay, i DEAL, etc.). |
| `provider_original_response_code` | string | no | The original response code from the payment method provider (or issuer for card). |
| `provider_original_response_description` | string | no | Human-readable description of `provider_original_response_code`, describing the reason returned by the issuer. |
| `provider_transaction_id` | string | no | Deprecated. Use `payment_method_transaction_id` instead. |
| `refunded_amount` | number | no | Refunded amount. |
| `settle_via` | string | no | Through which channel the merchant will get this payment settled. Possible values are `airwallex`, `paypal`, `amex`. Values: `airwallex`, `paypal`, `amex` |
| `status` | string | no | - **RECEIVED**: The Payment Attempt has been created upon request. - **AUTHENTICATION_REDIRECTED**: The Payment Attempt is waiting for further customer action of authentication, e.g. 3DS verification and QR code scan. - **PENDING_AUTHORI... Values: `RECEIVED`, `AUTHENTICATION_REDIRECTED`, `PENDING_AUTHORIZATION`, `AUTHORIZED`, `PENDING_REVIEW`, `CAPTURE_REQUESTED`, `EXPIRED`, `CANCELLED`, `FAILED`, `SETTLED`, `PAID` |
| `terminal_id` | string | no | Airwallex terminal unique identifier used to process the payment attempt. |
| `updated_at` | string | no | Last time at which this Payment Attempt was updated or operated on. |

### List all Payment Attempts

`GET /api/v1/pa/payment_attempts`

List all Payment Attempts. Specify a value for `payment_intent_id` to list the Payment Attempts made on the Payment Intent. You may filter out the results further by `method` and/or `status`.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `currency` | string | no | Payment Attempt currency. |
| `from_created_at` | string | no | The start time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `page_num` | integer | no | Page number starting from 0. |
| `page_size` | integer | no | Number of Payment Attempts to be listed per page. Default value is 10. |
| `payment_intent_id` | string | no | Payment Intent unique identifier of the attempt. |
| `status` | string | no | Payment Attempt status. |
| `to_created_at` | string | no | The end time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |

