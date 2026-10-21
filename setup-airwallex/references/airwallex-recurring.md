# Airwallex Customers, Payment Consents and Saved Methods

Checked 2026-10-21.

## Source URLs

- Recurring payments: `https://www.airwallex.com/docs/payments/about-airwallex-payments/recurring-payments`
- Credential-on-file payments: `https://www.airwallex.com/docs/payments/about-airwallex-payments/cof-payments`
- Save and reuse payment details: `https://www.airwallex.com/docs/payments/integration-options/web-checkout/save-and-reuse-payment-details`
- Native API registered user checkout: `https://www.airwallex.com/docs/payments/integration-options/web-checkout/native-api/registered-user-checkout`
- Touch 'n Go recurring: `https://www.airwallex.com/docs/payments/payment-methods/apac/touch-'n-go/accept-recurring-payments-touch-'n-go`
- Customers API: `https://www.airwallex.com/docs/api/payments/customers`
- Payment Consents API: `https://www.airwallex.com/docs/api/payments/payment_consents`
- Payment Methods API: `https://www.airwallex.com/docs/api/payments/payment_methods`

## Concepts

- Credential-on-file: the shopper authorizes you to store a method. Later charges are CIT (shopper in session) or MIT (merchant charges off-session).
- Customer (`cus_`): shopper profile with saved Payment Methods and Consents. Pass `customer_id` on intents to show saved methods.
- Payment Consent (`cst_`): the agreement. `next_triggered_by`: `merchant` or `customer`. `merchant_trigger_reason` (when merchant): `scheduled`, `unscheduled`, `installments`. Optional `terms_of_use` (billing cycles, schedule).
- Two routes: "Recurring with Payments" (you own billing logic: Customer, PaymentMethod, PaymentConsent, PaymentIntent) or "Recurring with Billing" (Airwallex Billing subscriptions; out of scope here).
- Malaysian methods supporting recurring: cards, Apple Pay, Google Pay, Touch 'n Go. Not FPX, DuitNow, Boost, GrabPay, ShopeePay, Atome, 7-Eleven.
- With HPP or Drop-in use `mode: 'recurring'` with a Customer `client_secret` (`GET /api/v1/pa/customers/{id}/generate_client_secret`) and `payment_consent` options.

## Touch 'n Go recurring flow (from the official guide)

1. Optional: `POST /api/v1/pa/customers/create` with `request_id`, `merchant_customer_id`, names, email, `phone_number`.
2. `POST /api/v1/pa/payment_consents/create` with `request_id`, `customer_id`, `next_triggered_by: "merchant"`, `merchant_trigger_reason: "unscheduled"`. Status `REQUIRES_PAYMENT_METHOD`.
3. `POST /api/v1/pa/payment_consents/{id}/verify` with `request_id`, `payment_method: { type: "tng", tng: { flow, os_type } }`, `return_url`. Status `REQUIRES_CUSTOMER_ACTION` with `next_action.url`; redirect the shopper to bind their wallet. Wait for `payment_consent.verified`.
4. Each charge: create a Payment Intent (MYR), then `POST /api/v1/pa/payment_intents/{id}/confirm` with `request_id` and `payment_consent_id`. No shopper re-authentication.
5. Stop with `POST /api/v1/pa/payment_consents/{id}/disable`.

## Card MIT notes

- First charge is CIT with consent setup (3DS as required); subsequent MIT uses `payment_consent_id` and `triggered_by: "merchant"`; CVC is not required for MIT.
- If you keep your own recurring records instead of consents, send `external_recurring_data` on create/confirm (card, Apple Pay, Google Pay only).

## Endpoint field tables
### Create a Customer

`POST /api/v1/pa/customers/create`

Create a **Customer** object. We require the Customer unique identifier in the merchant's system as a mandatory field.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `merchant_customer_id` | string | yes | Unique identifier of this customer in merchant's system. Maximum length is 64. |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `address` | object | no | Address of the customer. |
| `business_name` | string | no | Business name of the customer. Maximum length is 128. |
| `email` | string | no | Email address of the customer. Maximum length is 256. |
| `first_name` | string | no | First name of the customer. Maximum length is 128. |
| `last_name` | string | no | Last name of the customer. Maximum length is 128. |
| `metadata` | object | no | A set of key-value pairs that you can attach to this customer. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `phone_number` | string | no | Phone number of the customer. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `address` | object | no | Address of the customer. |
| `business_name` | string | no | Business name of the customer. |
| `client_secret` | string | no | Customer's client secret for browser or app. Only returned by Customer create API. The provided client_secret is valid for 60 minutes. |
| `created_at` | string | no | Time at which the Customer was created. |
| `email` | string | no | Email address of the Customer. |
| `first_name` | string | no | First name of the Customer. |
| `id` | string | no | Unique identifier for the Customer. |
| `last_name` | string | no | Last name of the Customer. |
| `merchant_customer_id` | string | no | Unique identifier of the Customer in merchant's system. Maximum length is 64. |
| `metadata` | object | no | A set of key-value pairs that you can attach to the Customer. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `phone_number` | string | no | Phone number of the Customer. |
| `request_id` | string | no | Unique request identifier specified by the merchant in the last operation. |
| `updated_at` | string | no | Last time at which the customer was updated or operated on. |

### Retrieve a Customer

`GET /api/v1/pa/customers/{id}`

Retrieve a Customer by unique identifier. This unique identifier is first returned when creating a new Customer.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Customer unique identifier. |

### List all Customers

`GET /api/v1/pa/customers`

List all Customers.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `from_created_at` | string | no | The start time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `merchant_customer_id` | string | no | The customer unique identifier on merchant side. |
| `page_num` | integer | no | Page number starting from 0. |
| `page_size` | integer | no | Number of Customers to be listed per page. Default value is 10. |
| `to_created_at` | string | no | The end time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |

### Update a Customer

`POST /api/v1/pa/customers/{id}/update`

Update customer's personal information. Use the Payment Methods API to update details of the attached Payment Methods.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Customer unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `address` | object | no | Address of the customer. |
| `business_name` | string | no | Business name of the customer. Maximum length is 128. |
| `email` | string | no | Email address of the customer. Maximum length is 256. |
| `first_name` | string | no | First name of the customer. Maximum length is 128. |
| `last_name` | string | no | Last name of the customer. Maximum length is 128. |
| `merchant_customer_id` | string | no | Unique identifier of this customer in merchant's system. Maximum length is 64. |
| `metadata` | object | no | A set of key-value pairs that you can attach to this customer. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `phone_number` | string | no | Phone number of the customer. |

### Generate a Client Secret for a Customer

`GET /api/v1/pa/customers/{id}/generate_client_secret`

Generate a client secret for a Customer so that the client secret can be used as HTTP header `client-secret` from client-side to create a Payment Method or list Payment Method.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Customer unique identifier. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `client_secret` | string | no | Customer's client secret for browser or app. Only returned by Customer create API. The provided client_secret is valid for 60 minutes. |
| `expired_time` | string | no | Time at when the client secret expires. |

### Create a Payment Consent

`POST /api/v1/pa/payment_consents/create`

Create a Payment Consent.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `customer_id` | string | yes | Unique identifier from Airwallex of the customer who will make the payment with this Payment Consent. |
| `next_triggered_by` | string | yes | The party to trigger subsequent payments. Possible values: - `customer` - `merchant` Values: `customer`, `merchant` |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `connected_account_id` | string | no | Airwallex assigned account identifier of the connected entity(another account which is linked with platform). It is to be used by the platform to indicate the connected entity in the transaction where platform is the owner of transaction. |
| `merchant_trigger_reason` | string | no | Indicate whether the subsequent payments are scheduled. Only applicable when `next_triggered_by` is `merchant`. Values: `installments`, `scheduled`, `unscheduled` |
| `metadata` | object | no | A set of key-value pairs that can be attached to this Payment Consent. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `terms_of_use` | object | no | Terms to specify how this Payment Consent will be used. |
| `terms_of_use.payment_amount_type` | string | yes | The agreed type of amounts for subsequent payment. Should be one of `FIXED`, `VARIABLE`. - **FIXED**: payment amount is fixed. Values: `FIXED`, `VARIABLE` |
| `terms_of_use.billing_cycle_charge_day` | integer | no | The granularity per billing cycle. Required when payment_schedule.period_unit is WEEK, MONTH, or YEAR. (e.g. charge_day_per_billing_cycle= 5, payment_schedule.period_unit  = MONTH, payment_schedule.period = 1 if collect payment on the 5t... |
| `terms_of_use.end_date` | string | no | End date to expect payment request. |
| `terms_of_use.first_payment_amount` | number | no | The first payment. It could include the costs associated with the first debited amount.Optional if payment agreement type is `VARIABLE`. |
| `terms_of_use.fixed_payment_amount` | number | no | The fixed payment amount that can be charged for a single payment.Required if payment agreement type is `FIXED`. |
| `terms_of_use.max_payment_amount` | number | no | The maximum payment amount that can be charged for a single payment.Optional if payment agreement type is `VARIABLE`. |
| `terms_of_use.min_payment_amount` | number | no | The minimum payment amount that can be charged for a single payment.Optional if payment agreement type is `VARIABLE`. |
| `terms_of_use.payment_currency` | string | no | The currency of this payment. Please refer to supported currencies. |
| `terms_of_use.payment_schedule` | object | no |  |
| `terms_of_use.start_date` | string | no | Start date to expect payment request. |
| `terms_of_use.total_billing_cycles` | integer | no | The total number of billing cycles. For example, the mandate will last for 1 year if `total_billing_cycles`=12, `payment_schedule.period`=1 and `payment_schedule.period_unit`=MONTH if the `merchant_trigger_reason` is `scheduled`. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `created_at` | string | yes | Time at which this Payment Consent was created. |
| `customer_id` | string | yes | Unique identifier from Airwallex of the customer who will make the payment with this Payment Consent. |
| `id` | string | yes | Unique identifier of this Payment Consent. |
| `next_triggered_by` | string | yes | The party to trigger subsequent payments. One of `merchant`, `customer`. Values: `merchant`, `customer` |
| `request_id` | string | yes | Unique request identifier specified by the merchant. |
| `status` | string | yes | - **REQUIRES_PAYMENT_METHOD**: Populate `payment_method` when calling **verify**. - This value is returned if `payment_method` is either null, or the `payment_method` has failed during verify, and a different `payment_method` should be p... Values: `REQUIRES_PAYMENT_METHOD`, `verify`, `REQUIRES_CUSTOMER_ACTION`, `PENDING`, `PAUSED`, `VERIFIED`, `DISABLED` |
| `updated_at` | string | yes | Time at which this Payment Consent was last updated. |
| `client_secret` | string | no | Payment Consent's client secret for browser or app. Only returned by Payment Consent create API. The provided client_secret is valid for 60 minutes. |
| `connected_account_id` | string | no | Account identifier of the connected account. |
| `disable_reason` | string | no | Disable reason. Only applicable when status is `DISABLED`. |
| `failure_reason` | object | no | Verification failure reason. Only applicable when status is `REQUIRES_PAYMENT_METHOD` and `REQUIRES_CUSTOMER_ACTION`. |
| `initial_payment_intent_id` | string | no | Unique identifier of the initial Payment Intent confirmed with this Payment Consent. |
| `mandate` | object | no | An authorization from shopper to merchant to debit payments from their bank account. |
| `merchant_trigger_reason` | string | no | Only applicable when `next_triggered_by` is `merchant`. One of `scheduled`, `unscheduled`, `installments`. Values: `scheduled`, `unscheduled`, `installments` |
| `metadata` | object | no | A set of key-value pairs that can be attached to this Payment Consent. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `next_action` | object | no | Next action for merchant. |
| `payment_method` | object | no | Payment Method information attached for subsequent payments. |
| `terms_of_use` | object | no | The terms of this consent. |

### Retrieve a Payment Consent

`GET /api/v1/pa/payment_consents/{id}`

Retrieve a Payment Consent by unique identifier.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Consent unique identifier. |

### List all Payment Consents

`GET /api/v1/pa/payment_consents`

List all Payment Consents.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `customer_id` | string | no | The unique identifier of a customer. |
| `from_created_at` | string | no | The start time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `merchant_trigger_reason` | string | no | One of `scheduled`, `unscheduled`. Values: `scheduled`, `unscheduled` |
| `next_triggered_by` | string | no | One of `merchant`, `customer`. Values: `merchant`, `customer` |
| `page_num` | integer | no | Page number starting from 0. |
| `page_size` | integer | no | Number of Payment Consents to be listed per page. Default value is 10. |
| `payment_method_id` | string | no | The unique identifier of a Payment Method. |
| `status` | string | no | Status of Payment Consent. |
| `to_created_at` | string | no | The end time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |

### Update a Payment Consent

`POST /api/v1/pa/payment_consents/{id}/update`

Update the metadata of a Payment Consent.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Consent unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `metadata` | object | no | A set of key-value pairs that will be updated for this Payment Consent. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |

### Verify a Payment Consent

`POST /api/v1/pa/payment_consents/{id}/verify`

Verify the Payment Consent with the `payment_method` details (or `payment_method.id` if reusing an existing one) when the customer is ready to store payment credentials into the Payment Consent for subsequent payments. Additional customer actions might be required to complete the verification for some payment methods. In that case, `next_action` will be returned to guide you through the process and the Payment Consent `status` will be updated to `REQUIRES_CUSTOMER_ACTION`. Once the `status` becomes `VERIFIED`, the Payment Consent is ready for making subsequent payments. When verifying a Pay...

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Consent unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `descriptor` | string | no | Descriptor that may be displayed to the customer during the verification. |
| `device_data` | object | no | Client device information. It is recommended to provide this data to increase the odds of having frictionless 3DS. |
| `payment_method` | object | no | Payment Method information that will be used for subsequent payments. When `payment_method.type` is set to `card`, `ach_direct_debit`, `bacs_direct_debit`, `becs_direct_debit`, `sepa_direct_debit`, `eft_direct_debit`, `googlepay`, `apple... |
| `payment_method.type` | string | yes | Type of the Payment Method. One of `card`, `googlepay`, `applepay`, `alipayhk`, `truemoney`, `gcash`, `dana`, `kakaopay`, `tng`, `alipaycn`, `rabbit_line_pay`, `wechatpay`, `ach_direct_debit`, `bacs_direct_debit`, `becs_direct_debit`, `s... Values: `card`, `googlepay`, `applepay`, `alipayhk`, `truemoney`, `gcash`, `dana`, `kakaopay`, `tng`, `alipaycn`, `rabbit_line_pay`, `wechatpay`, `ach_direct_debit`, `bacs_direct_debit` ... |
| `payment_method.ach_direct_debit` | object | no | ACH Direct Debit information. |
| `payment_method.alipaycn` | object | no | Alipaycn information. `flow` can be one of `qrcode`, `mobile_web`, `mobile_app`. Values: `qrcode`, `mobile_web`, `mobile_app` |
| `payment_method.alipayhk` | object | no | Alipayhk information. `flow` can be one of `qrcode`, `mobile_web`, `mobile_app`. Values: `qrcode`, `mobile_web`, `mobile_app` |
| `payment_method.applepay` | object | no | Apple pay information. |
| `payment_method.bacs_direct_debit` | object | no | BACS Direct Debit information. |
| `payment_method.becs_direct_debit` | object | no | BECS Direct Debit information. |
| `payment_method.card` | object | no | Card information. |
| `payment_method.dana` | object | no | Dana information. `flow` can be one of `qrcode`, `mobile_web`. Values: `qrcode`, `mobile_web` |
| `payment_method.eft_direct_debit` | object | no | EFT Direct Debit information. |
| `payment_method.gcash` | object | no | Gcash information. `flow` can be one of `qrcode`, `mobile_web`, `mobile_app`. Values: `qrcode`, `mobile_web`, `mobile_app` |
| `payment_method.googlepay` | object | no | Google pay information. |
| `payment_method.id` | string | no | Unique identifier of the Payment Method attached for subsequent payments. |
| `payment_method.kakaopay` | object | no | Kakaopay information. `flow` can be one of `qrcode`, `mobile_web`, `mobile_app`. Values: `qrcode`, `mobile_web`, `mobile_app` |
| `payment_method.payto` | object | no | Payto information. |
| `payment_method.rabbit_line_pay` | object | no | Rabbit LINE Pay information. `flow` can be one of `qrcode`, `mobile_web`, `mobile_app`. Values: `qrcode`, `mobile_web`, `mobile_app` |
| `payment_method.sepa_direct_debit` | object | no | SEPA Direct Debit information. |
| `payment_method.tng` | object | no | Tng information. `flow` can be one of `qrcode`, `mobile_web`, `mobile_app`. Values: `qrcode`, `mobile_web`, `mobile_app` |
| `payment_method.truemoney` | object | no | Truemoney information. `flow` can be one of `qrcode`, `mobile_web`, `mobile_app`. Values: `qrcode`, `mobile_web`, `mobile_app` |
| `payment_method.wechatpay` | object | no | Wechat information. |
| `return_url` | string | no | The web page URL or application scheme URI to redirect the customer after Payment Consent verification. |
| `risk_control_options` | object | no | Risk Control Options. |
| `verification_options` | object | no | Additional information needed to verify a Payment Consent. |
| `verification_options.applepay` | object | no | Apple pay information for verifying Payment Consent. |
| `verification_options.card` | object | no | Card information for verifying Payment Consent. |
| `verification_options.googlepay` | object | no | Google pay information for verifying Payment Consent. |
| `verification_options.wechatpay` | object | no | WeChat Pay information for verifying Payment Consent. |

### Continue To Verify a Payment Consent

`POST /api/v1/pa/payment_consents/{id}/verify_continue`

Continue verify a payment Consent, required when verification method is micro-deposit. Merchants can use this api to verify the amount of micro-deposit if the payment consent wait for the verification result of micro-deposit. Micro-debit is triggered when micro-deposit is triggered, micro-debit is not part of micro-deposit verification. If micro-deposit is success and micro-debit failed, merchants can use this api to retry micro-debit.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Consent unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. |
| `micro_deposit` | object | no | The verification request of micro-deposit, required only if type is `verify_micro_deposits`. |
| `type` | string | no | Type of the continue type. Possible values: - `retry_micro_debit` - `verify_micro_deposits` Values: `retry_micro_debit`, `verify_micro_deposits` |

### Disable a Payment Consent

`POST /api/v1/pa/payment_consents/{id}/disable`

Disable a Payment Consent by unique identifier. After a Payment Consent is disabled, the status will become `DISABLED` and the Payment Consent cannot be used or updated.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Consent unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |

### Create a Payment Method

`POST /api/v1/pa/payment_methods/create`

Create a Payment Method which can be referenced by Payment Consent. Alternatively you can directly provide entire payload of new Payment Method when verifying the Payment Consent, and then a Payment Method will be generated for you automatically during the process, which can be re-used for other Payment Consent later.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `customer_id` | string | yes | The Customer this Payment Method belongs to. This Payment Method is automatically added to the Customer as one of the available Payment Methods. |
| `request_id` | string | yes | Unique request identifier specified by the merchant. |
| `type` | string | yes | Type of the Payment Method. Possible values: - `card` - `applepay` - `googlepay` Values: `card`, `applepay`, `googlepay` |
| `applepay` | object | no | Apple pay information. This must be provided if `type` is set to `applepay`. |
| `card` | object | no | Card information. This must be provided if `type` is set to `card`. |
| `googlepay` | object | no | Google pay information. This must be provided if `type` is set to `googlepay`. |
| `metadata` | object | no | A set of key-value pairs that you can attach to this Payment Method. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `ach_direct_debit` | object | no | ACH Direct Debit information. Only provided when `type` is `ach_direct_debit`. |
| `applepay` | object | no | Apple pay information. Only provided when `type` is `applepay`. |
| `bacs_direct_debit` | object | no | BACS Direct Debit information. Only provided when `type` is `bacs_direct_debit`. |
| `becs_direct_debit` | object | no | BECS Direct Debit information. Only provided when `type` is `becs_direct_debit`. |
| `card` | object | no | Card information. Only provided when `type` is `card`. |
| `created_at` | string | no | Time when the Payment Method was created. |
| `customer_id` | string | no | The Customer this Payment Method belongs to. |
| `eft_direct_debit` | object | no | EFT Direct Debit information. Only provided when `type` is `eft_direct_debit`. |
| `googlepay` | object | no | Google pay information. Only provided when `type` is `googlepay`. |
| `id` | string | no | Unique identifier for the Payment Method. |
| `metadata` | object | no | A set of key-value pairs that you can attach to this Payment Method. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `request_id` | string | no | Unique request identifier specified by the merchant. |
| `sepa_direct_debit` | object | no | SEPA Direct Debit information. Only provided when `type` is `sepa_direct_debit`. |
| `status` | string | no | Status of the Payment Method. One of `CREATED`, `DISABLED`. Values: `CREATED`, `DISABLED` |
| `type` | string | no | Type of the Payment Method. One of `card`, `applepay`, `googlepay`, `ach_direct_debit`, `bacs_direct_debit`, `becs_direct_debit`, `sepa_direct_debit`, `eft_direct_debit`. Values: `card`, `applepay`, `googlepay`, `ach_direct_debit`, `bacs_direct_debit`, `becs_direct_debit`, `sepa_direct_debit`, `eft_direct_debit` |
| `updated_at` | string | no | Time when the Payment Method was last updated. |

### Retrieve a Payment Method

`GET /api/v1/pa/payment_methods/{id}`

Retrieve Payment Method by unique identifier.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Method unique identifier. |

### List all Payment Methods

`GET /api/v1/pa/payment_methods`

List all Payment Methods.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `customer_id` | string | no | Customer unique identifier of the Payment Methods. |
| `from_created_at` | string | no | The start time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `page_num` | integer | no | Page number starting from 0. |
| `page_size` | integer | no | Number of Payment Methods to be listed per page. Default value is 10. |
| `status` | string | no | Payment Method status. One of `CREATED`, `DISABLED`. Values: `CREATED`, `DISABLED` |
| `to_created_at` | string | no | The end time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `type` | string | no | Payment Method type, can be `card`, `ach_direct_debit`. |

### Update a Payment Method

`POST /api/v1/pa/payment_methods/{id}/update`

Depending on the `payment_method_type`, you may update some of the details contained within.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Method to update. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `card` | object | yes | Card information. This must be provided. |
| `request_id` | string | yes | Unique request identifier specified by the merchant. |

### Disable a Payment Method

`POST /api/v1/pa/payment_methods/{id}/disable`

Disable Payment Method by unique identifier. The Payment Method status would become `DISABLED`.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Method unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. |

