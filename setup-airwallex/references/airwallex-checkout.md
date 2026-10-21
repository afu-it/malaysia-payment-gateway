# Airwallex Checkout Surfaces

Checked 2026-10-21. Hosted Payment Page (Airwallex.js), Drop-in Element, Checkout Sessions API, and Payment Links API.

## Source URLs

- Airwallex.js reference: `https://www.airwallex.com/docs/js/getting-started`
- `redirectToCheckout()` reference: `https://www.airwallex.com/docs/js/payments/hosted-payment-page`
- Drop-in reference: `https://www.airwallex.com/docs/js/payments/dropin`
- Hosted Payment Page guide: `https://www.airwallex.com/docs/payments/integration-options/web-checkout/hosted-payment-page`
- Drop-in quickstart: `https://www.airwallex.com/docs/payments/integration-options/web-checkout/drop-in-element/guest-user-checkout`
- Checkout Session quickstart: `https://www.airwallex.com/docs/payments/integration-options/web-checkout/checkout-session/quickstart`
- Checkout Sessions API: `https://www.airwallex.com/docs/api/checkout/checkout_sessions`
- Payment Links API: `https://www.airwallex.com/docs/api/payments/payment_links`
- Web checkout overview: `https://www.airwallex.com/docs/payments/integration-options/web-checkout`
- Customize appearance: `https://www.airwallex.com/docs/payments/integration-options/web-checkout/customize-appearance`
- Airwallex.js error codes: `https://www.airwallex.com/docs/developer-tools/sdks/airwallex.js/error-codes`

## Install Airwallex.js

- npm: `npm install @airwallex/components-sdk` then `import { init, createElement } from '@airwallex/components-sdk'`.
- CDN: `<script src="https://static.airwallex.com/components/sdk/v1/index.js"></script>` exposes `window.AirwallexComponentsSDK`.
- Call `init()` once: `await init({ env, enabledElements: ['payments'], locale })`. `env` is `'sandbox' | 'prod'` in the reference (default `'prod'`); the guides use `'demo'` for sandbox. `locale` includes `'ms'` (Bahasa Melayu).
- `init()` resolves to `{ payments }`; `payments.redirectToCheckout(props)` opens the hosted page.

## Hosted Payment Page: `redirectToCheckout(props)`

```js
const { payments } = await window.AirwallexComponentsSDK.init({ env: 'prod', enabledElements: ['payments'] });
payments.redirectToCheckout({
  intent_id, client_secret, currency: 'MYR',   // from your server
  country_code: 'MY',
  successUrl: 'https://shop.example/orders/<ORDER_REF>/return', // HTTPS
  methods: ['fpx', 'duit_now', 'tng', 'card', 'googlepay', 'applepay'],
  locale: 'ms',
});
```

| Prop | Type | Required | Notes |
| --- | --- | --- | --- |
| `client_secret` | string | yes | Intent `client_secret` (or Customer `client_secret` in recurring setup) |
| `currency` | string | yes | Only methods supporting it are shown |
| `intent_id` | string | when `mode` is `'payment'` | Payment Intent id |
| `mode` | `'payment'` or `'recurring'` | no | Default `'payment'` |
| `env` | `'sandbox'` or `'prod'` | no | Default `prod` |
| `successUrl` | string | no | HTTPS redirect after success; else the intent `return_url` |
| `cancelUrl` | string | no | HTTPS redirect when shopper goes back or cancels |
| `country_code` | string | no | Shopper country; filters methods |
| `methods` | array | no | Restrict and order methods. MY-relevant values: `card`, `applepay`, `googlepay`, `fpx`, `duit_now`, `tng`, `boost`, `grabpay`, `shopee_pay`, `atome`, `spaylater`, `seven_eleven` |
| `autoCapture` | boolean | no | Default `true` |
| `authorizationType` | `'pre_auth'` or `'final_auth'` | no | Default `'final_auth'` |
| `customer_id` | string | no | Registered-user checkout |
| `payment_consent` | object | no | Consent details for recurring |
| `requiredBillingContactFields` | array of `name`, `email`, `country_code`, `address`, `phone` | no | Billing data for card AVS/3DS |
| `shopper_name`, `shopper_email`, `shopper_phone` | string | no | Prefill |
| `appearance`, `layout`, `logoUrl`, `submitType` | various | no | Look and feel; `submitType` is `pay`, `book`, `subscribe`, `save` |
| `applePayRequestOptions`, `googlePayRequestOptions` | object | no | Wallet options |
| `allowedCardNetworks` | array | no | `visa`, `mastercard`, `maestro`, `unionpay`, `amex`, `jcb`, `diners`, `discover` |
| `disableAutoRedirect` | boolean | no | Default `false` |
| `locale` | string | no | Defaults to browser locale, else `en` |
| `platformConnectedAccount` | string | no | Payments for Platforms only |
| `autoSaveCardForFuturePayments`, `savedPaymentMethod`, `cvcRequired` | various | no | Saved card behaviour |
| `failUrl`, `theme`, `withBilling`, `components` | | deprecated | Do not use in new code |

By default HPP shows every activated method, filtered by `currency` and `country_code`, ordered as in `GET /api/v1/pa/config/payment_method_types` with `applepay`, `googlepay`, `paypal` first. Apple Pay on HPP needs no domain registration. HPP supports 25 languages including Malay.

## Drop-in Element

```js
await init({ env: 'prod', enabledElements: ['payments'] });
const el = await createElement('dropIn', { intent_id, client_secret, currency: 'MYR', country_code: 'MY' });
el.mount('dropIn');                     // id of a container div
el.on('ready', () => {});
el.on('success', (e) => { const { id } = e.detail.intent; /* tell backend; backend verifies */ });
el.on('error', (e) => { const { code, message } = e.detail.error; });
```

- Options mirror HPP: `intent_id`, `client_secret` (required), `currency` (required), `mode`, `methods`, `country_code` (required for `online_banking`, `bank_transfer`, `skrill`, `seven_eleven`), `customer_id`, `autoCapture`, `authorizationType`, `appearance`, `layout`, `submitType`, `payment_consent`.
- Events: `ready`, `success`, `error`, `cancel`, `clickConfirmButton`, `switchMethod`, `pendingVerifyAccount`, `surchargeFeeChange`, `change`. Methods: `mount`, `unmount`, `destroy`, `update`, `confirm`, `trigger('intentRefetch')`. Attach listeners after `createElement`.
- `success` is a client signal only; fulfil on the webhook.

## Checkout Sessions (no JS SDK)

- `POST /api/v1/checkout/checkout_sessions/create` with `mode: "PAYMENT"`, `request_id`, `currency`, `line_items` (each `price_id` or inline `price`, plus `quantity`), `success_url`, and `invoice_creation.enabled: false` for a plain payment. The API reference marks `account_id` required (find it in **Settings > Account details**).
- Response has `id`, `status: "ACTIVE"`, `url` (present only while `ACTIVE`), `expires_at` (1 hour). Redirect the shopper to `url` (client-side or HTTP 302).
- Session statuses include `ACTIVE`, `COMPLETED`, `CANCELLED`, `EXPIRED`. On completion the session creates the Payment Intent; `GET .../checkout_sessions/{id}` returns `payment_intent_id`.
- The API reference flags Checkout Sessions as early access (`Status: early_access`). Confirm availability on your account.
- Fulfil on `payment_intent.succeeded`, not on `success_url`.

## Payment Links

No-code or API (`POST /api/v1/pa/payment_links/create`). `payment_link.paid` webhook fires per payment; each payment still creates a Payment Intent (`payment_link_id` on the intent). Holds (manual capture) are not supported on Payment Links.

## Endpoint field tables
### Create a Checkout Session

`POST /api/v1/checkout/checkout_sessions/create`

> Status: early_access Creates a Checkout Session object. A Checkout Session is created for one of the following use cases: - **One-off payment.** Set `mode` to `PAYMENT`. Omit `invoice_creation`, or set `invoice_creation.enabled` to `false`. See Checkout Session quickstart. - **One-off payment with an invoice.** Set `mode` to `PAYMENT` and set `invoice_creation.enabled` to `true`. See Invoicing. - **Subscription payments.** Set `mode` to `SUBSCRIPTION`. See Subscription management. - **Save a payment method for future use.** - **Save as Payment Consent.** Set `mode` to `SETUP` and pass `cu...

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `account_id` | string | yes | ID of the Airwallex payment account to collect payment. You can find this ID in the Airwallex web app > Settings > Account details, within the Account Information section. |
| `mode` | string | yes | The mode of the Checkout Session object, can be one of: - `PAYMENT`: Accept one-off payments. - `SUBSCRIPTION`: Set up subscriptions. - `SETUP`: Collect payment details to charge your customers later. Values: `PAYMENT`, `SUBSCRIPTION`, `SETUP` |
| `request_id` | string | yes | Unique request ID specified by the merchant. |
| `back_url` | string | no | If set, the hosted checkout page displays a back button which allows customers to cancel the flow and be directed to this URL. |
| `billing_customer_id` | string | no | The ID of the Billing Customer associated with this Checkout Session. Use this field when: - `mode` is `PAYMENT` and `invoice_creation.enabled` is `true`. - `mode` is `SUBSCRIPTION`. - `mode` is `SETUP` and the collected payment method i... |
| `currency` | string | no | The checkout currency in 3-letter ISO-4217 format. Required when `mode` is `PAYMENT` or `SETUP`. |
| `customer_data` | object | no | The prefilled information of the customer. Only applicable when `billing_customer_id` and `customer_id` are omitted and `mode` is not `SETUP`. |
| `customer_data_collection` | object | no | Provides configuration for the Checkout Session to collect information from customers. |
| `customer_data_collection.billing_address` | string | no | Controls whether the billing address will be collected during checkout. - `AUTO`: Billing address is collected only when necessary, including when automatic tax is enabled or a tax ID is collected. - `REQUIRED`: Billing address is always... |
| `customer_data_collection.email` | string | no | Controls whether the email address will be collected during checkout. - `AUTO`: Email address is collected only when necessary, including when an invoice is created or when required by the selected payment method. - `REQUIRED`: Email add... Values: `AUTO`, `REQUIRED` |
| `customer_data_collection.name` | string | no | Controls whether the name will be collected during checkout. - `AUTO`: Name is collected only when necessary, including when an invoice is created or when required by the selected payment method. - `REQUIRED`: Name is always collected. P... Values: `AUTO`, `REQUIRED` |
| `customer_data_collection.phone_number` | string | no | Controls whether the phone number will be collected during checkout. - `AUTO`: Phone number is collected only when necessary. - `REQUIRED`: Phone number is always collected. Possible values: - `AUTO` - `REQUIRED` Values: `AUTO`, `REQUIRED` |
| `customer_data_collection.tax_id` | string | no | Controls whether the tax ID will be collected for a business customer during checkout. - `DISABLED`: Tax ID will not be collected. - `OPTIONAL_IF_SUPPORTED`: Tax ID is optionally collected in supported countries by displaying the "I am p... |
| `customer_id` | string | no | The ID of the Customer associated with this Checkout Session. Use this field when: - `mode` is `PAYMENT` and `invoice_creation.enabled` is `false` or omitted. - `mode` is `SETUP` and the collected payment method is saved as a Payment Con... |
| `discounts` | array[object] | no | List of discounts to be applied to the subscription or invoice. Discounts of line items are applied before subscription or invoice discounts. |
| `enable_automatic_tax` | boolean | no | Whether automatic tax is enabled for this checkout. When enabled, the system attempts to calculate taxes for this checkout's invoice automatically. |
| `hosted_completion_page` | object | no | Configuration of the customer experience after the Checkout Session completes. Only applicable when `ui_mode` is `HOSTED`. |
| `invoice_creation` | object | no | Invoice creation settings. Only applicable when `mode` is `PAYMENT`. |
| `invoice_creation.enabled` | boolean | yes | Whether to create an invoice for this Checkout Session. Only applicable when `mode` is `PAYMENT`. |
| `invoice_creation.invoice_data` | object | no | The settings of the invoice to be created when `enabled` is `true`. |
| `legal_entity_id` | string | no | ID of the legal entity to be used as billing entity. You can find this ID in the Airwallex web app > Settings. |
| `line_items` | array[object] | no | A list of items the customer is purchasing. Required when `mode` is `SUBSCRIPTION` or `PAYMENT`. |
| `line_items.discounts` | array[object] | no | List of discounts to be applied to the line item. Discounts of line items are applied before subscription or invoice discounts. |
| `line_items.price` | object | no | One-time price definition following the same schema as the Price resource. Exactly one of `price_id` or `price` is required. Values: `price_id`, `price` |
| `line_items.price_id` | string | no | ID of the Price object. Exactly one of `price_id` or `price` is required. Values: `price_id`, `price` |
| `line_items.quantity` | integer | no | The quantity of the line item being purchased. Defaults to `1`. |
| `line_items.tax_percent` | number | no | The effective tax percent of this line item. Only applicable when an invoice is created. |
| `locale` | string | no | The locale for your website. Defaults to the browser's locale; however, if the browser's locale is not supported, it will default to `en`. |
| `metadata` | object | no | A set of string key-value pairs that you can attach to this object for storing additional information. |
| `payment_intent_data` | object | no | A subset of parameters used to create the Payment Intent for this Checkout Session. |
| `payment_options` | object | no | Payment settings of this Checkout Session. |
| `payment_options.allowed_card_networks` | array[string] | no | One or more card networks that you support. Accepted values are `visa`, `mastercard`, `maestro`, `unionpay`, `amex`, `jcb`, `diners`, and `discover`. |
| `payment_options.authorization_type` | string | no | The authorization type of the card payment. One of `final_auth`, `pre_auth`. Values: `final_auth`, `pre_auth` |
| `payment_options.auto_capture` | boolean | no | Whether the amount should be captured automatically upon successful payment authorization. Set to `false` to place a hold on the payment method and capture the funds later. |
| `payment_options.pay_by_invoice_enabled` | boolean | no | Enables the option to "Request an invoice" for payment. After the customer confirms to pay with this option, the status of Checkout Session will transit to COMPLETED, and an invoice will be generated. |
| `payment_options.payment_method_save` | object | no | Settings that control whether and how the payment method can be reused for future payments. |
| `payment_options.payment_method_types` | array[string] | no | Specify which payment methods to display on the Checkout Page and set their display order. Only methods supported for the specified currency and the customer's country code will appear on the Checkout Page. Values: `mode`, `SUBSCRIPTION`, `card`, `applepay`, `googlepay`, `alipaycn`, `alipayhk`, `dana`, `gcash`, `kakaopay`, `klarna`, `rabbit_line_pay`, `tng`, `truemoney` |
| `return_url` | string | no | The URL to redirect your customer back to after they authenticate or cancel their payment on the payment method's app or site, with payment status not confirmed. Only applicable when `ui_mode` is `EMBEDDED` or `ELEMENTS`. |
| `subscription_data` | object | no | The settings of subscription to be passed to the subscription creation upon completion of the Checkout Session in `SUBSCRIPTION` mode. |
| `success_url` | string | no | The URL the customer will be directed to after the checkout completion. Only applicable when `ui_mode` is `HOSTED`. |
| `ui_mode` | string | no | The UI mode of the Checkout Session: - `HOSTED`: Customers complete checkout on an Airwallex-hosted page. - `EMBEDDED`: Embed the checkout UI directly in your website or app. - `ELEMENTS`: Build a custom checkout UI using Airwallex Eleme... |
| `ui_options` | object | no | Customization options for the Checkout Page. |
| `ui_options.layout` | object | no | The layout of the Checkout Page. |
| `ui_options.submit_type` | string | no | Describes the type of transaction being performed in order to customize the submit button text. Accepted values are `pay`, `book`, `subscribe`, and `save`. |
| `ui_options.theme_id` | string | no | ID of the theme to apply to this Checkout Session, overriding the theme applied to its legal entity. You can find this ID in the Airwallex web app > Settings > Themes. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `account_id` | string | yes | ID of the Airwallex payment account used to collect payment. You can find this ID in the Airwallex web app > Settings > Account details, within the Account Information section. |
| `automatic_tax_status` | string | yes | The automatic tax status of this checkout. One of: - `APPLIED`: The automatic tax is applied. - `DISABLED`: The automatic tax is disabled. - `LOCATION_NOT_SUPPORTED`: Automatic tax calculation isn't supported in the customer's location. ... Values: `APPLIED`, `DISABLED`, `LOCATION_NOT_SUPPORTED`, `LOCATION_NOT_REGISTERED_BY_MERCHANT`, `ADDRESS_INCOMPLETE` |
| `created_at` | string | yes | Time when the Checkout Session was created. |
| `discounts` | array[object] | yes | List of discounts applied to the subscription or invoice. Discounts of line items are applied before subscription or invoice discounts. |
| `enable_automatic_tax` | boolean | yes | Whether automatic tax is enabled for this checkout. When enabled, the system attempts to calculate taxes for this checkout's invoice automatically. |
| `expires_at` | string | yes | Time when the Checkout Session expires. |
| `id` | string | yes | ID of the Checkout Session object. |
| `line_items` | array[object] | yes | A list of items the customer is purchasing. |
| `mode` | string | yes | The mode of the Checkout Session object, can be one of: - `PAYMENT`: Accept one-off payments. - `SUBSCRIPTION`: Set up subscriptions. - `SETUP`: Collect payment details to charge your customers later. Values: `PAYMENT`, `SUBSCRIPTION`, `SETUP` |
| `status` | string | yes | The status of the Checkout Session object, can be one of: - `ACTIVE`: The Checkout Session is available for the customer to complete a purchase. - `COMPLETED`: The Checkout Session has been completed. - `CANCELLED`: The Checkout Session ... Values: `ACTIVE`, `COMPLETED`, `CANCELLED`, `EXPIRED` |
| `ui_mode` | string | yes | The UI mode of the Checkout Session, can be one of: `HOSTED`: Customers complete the checkout on an Airwallex-hosted page. `EMBEDDED`: Embed the checkout UI directly in your website or app. Values: `HOSTED`, `EMBEDDED`, `ELEMENTS` |
| `updated_at` | string | yes | Time when the Checkout Session was last updated. |
| `back_url` | string | no | If set, the hosted checkout page displays a back button which allows customers to cancel the flow and be directed to this URL. |
| `billing_customer_id` | string | no | ID of the Billing Customer associated with this Checkout Session, if applicable. |
| `cancelled_at` | string | no | Time when the Checkout Session was cancelled. |
| `client_secret` | string | no | The client secret used to initialize the Checkout Session in `EMBEDDED` or `ELEMENTS` `ui_mode`. Only present when `ui_mode` is `EMBEDDED` or `ELEMENTS`. |
| `completed_at` | string | no | Time when the Checkout Session was completed. |
| `currency` | string | no | The checkout currency in 3-letter ISO-4217 format. |
| `customer_data` | object | no | The prefilled information of the customer. |
| `customer_data_collection` | object | no | Provides configuration for the Checkout Session to collect information from customers. |
| `customer_id` | string | no | ID of the Customer from the Payments API associated with this Checkout Session, if applicable. |
| `hosted_completion_page` | object | no | Configuration of the customer experience after the Checkout Session completes. Only applicable when `ui_mode` is `HOSTED`. |
| `invoice_creation` | object | no | Invoice creation settings for this Checkout Session. |
| `invoice_id` | string | no | ID of the invoice for the Checkout Session when `mode` is `PAYMENT`. Only applicable when `status` is `COMPLETED`. |
| `legal_entity_id` | string | no | ID of the legal entity to be used as billing entity, which can be retrieved from the `legal_entity_id` field in the Retrieve account details API response. |
| `locale` | string | no | The locale for the Checkout Session. |
| `metadata` | object | no | A set of string key-value pairs that you can attach to this object for storing additional information. |
| `payment_consent_id` | string | no | ID of the Payment Consent created when the Checkout Session is completed. |
| `payment_intent_data` | object | no | The Payment Intent parameters configured for this Checkout Session. |
| `payment_intent_id` | string | no | ID of the Payment Intent associated with this Checkout Session. |
| `payment_options` | object | no | Payment settings of this Checkout Session. |
| `payment_source_id` | string | no | ID of the Payment Source created when the Checkout Session is completed. |
| `return_url` | string | no | The URL to redirect your customer back to after they authenticate or cancel their payment on the payment method's app or site, with payment status not confirmed. |
| `subscription_data` | object | no | The settings of subscription to be passed to the subscription creation upon completion of the Checkout Session in `SUBSCRIPTION` mode. |
| `subscription_id` | string | no | ID of the subscription for the Checkout Session when `mode` is `SUBSCRIPTION`. Only applicable when `status` is `COMPLETED`. |
| `success_url` | string | no | The URL the customer will be directed to after the checkout completion. |
| `ui_options` | object | no | Customization options for the Checkout Page. |
| `url` | string | no | The URL to the Checkout Session. Only present when `status` is `ACTIVE`. |

### Retrieve a Checkout Session

`GET /api/v1/checkout/checkout_sessions/{id}`

> Status: early_access Retrieves a Checkout Session.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | ID of the Checkout Session object. |

### Get list of Checkout Sessions

`GET /api/v1/checkout/checkout_sessions`

> Status: early_access Retrieves a list of Checkout Sessions based on the query parameters.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `account_id` | string | no | ID of the Airwallex payment account associated with the Checkout Session. You can find this ID in the Airwallex web app > Settings > Account details, within the Account Information section. |
| `from_created_at` | string | no | The start date of `created_at` in ISO8601 format (inclusive). |
| `page` | string | no | A bookmark for use in pagination to retrieve either the next page or the previous page of results. You can fetch the value for this identifier from the response of the previous API call. |
| `page_size` | integer | no | Number of Checkout Sessions per page. Defaults to `20`. |
| `status` | string | no | Status of the Checkout Session. |
| `subscription_id` | string | no | ID of the subscription for the Checkout Session upon completion. |
| `to_created_at` | string | no | The end date of `created_at` in ISO8601 format (exclusive). |

### Update a Checkout Session

`POST /api/v1/checkout/checkout_sessions/{id}/update`

> Status: early_access Updates a Checkout Session object.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | ID of the Checkout Session |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `metadata` | object | no | A set of string key-value pairs that you can attach to this object for storing additional information. |

### Cancel a Checkout Session

`POST /api/v1/checkout/checkout_sessions/{id}/cancel`

> Status: early_access Cancels an `ACTIVE` Checkout Session. Once cancelled, the customer will no longer be able to complete the checkout and will see an error message when attempting to load it.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | ID of the Checkout Session |

### Create a Payment Link

`POST /api/v1/pa/payment_links/create`

Create a new payment link.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `reusable` | boolean | yes | Specifies whether the payment link can be used once or multiple times. |
| `title` | string | yes | The title of the payment link that is displayed in the payment checkout page. |
| `amount` | number | no | The payment amount. For Fixed pricing only. |
| `collectable_shopper_info` | object | no | Every payment link requires the shopper to enter their name and email address. Additionally, you can request information for the following fields on each payment link. |
| `connected_account_id` | string | no | Airwallex assigned account identifier of the connected entity(another account which is linked with platform). It is to be used by the platform to indicate the connected entity in the transaction where platform is the owner of transaction. |
| `currency` | string | no | The payment currency. For Fixed pricing only. |
| `customer_id` | string | no | The Customer who is paying for this Payment Link. Only applicable for links with `reusable` equals to `false`. |
| `default_currency` | string | no | The default payment currency from the list of `supported_currencies`. The shopper can change the payment currency during checkout. |
| `description` | string | no | The additional description text that appears under the title in the payment checkout page. |
| `expires_at` | string | no | Payment link expiry timestamp in ISO 8601 format. |
| `metadata` | object | no | A set of key-value pairs that you can attach to the Payment Link for storing additional information. |
| `reference` | string | no | A reference only visible to the merchant. |
| `supported_currencies` | array[string] | no | List of supported currencies the shopper may choose from. For Flexible pricing only. |
| `ui_options` | object | no | UI configuration for the payment checkout page. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `active` | boolean | no | The payment link’s active status, either `true` or `false`. |
| `amount` | number | no | The payment amount for Fixed pricing. |
| `collectable_shopper_info` | object | no | Every payment link requires the shopper to enter their name and email address. Additionally, you can request information for the following fields on each payment link. |
| `created_at` | string | no | Timestamp in ISO 8601 format when the payment link was created. |
| `currency` | string | no | The payment currency for Fixed pricing. |
| `customer_id` | string | no | The Customer who is paying for this Payment Link. Only applicable for links with `reusable` equals to `false`. |
| `default_currency` | string | no | The default currency for Flexible pricing. |
| `description` | string | no | The additional description text that appears under the title in the payment checkout page. |
| `expires_at` | string | no | Payment link expiry timestamp in ISO 8601 format. |
| `id` | string | no | Payment link unique identifier. |
| `latest_successful_payment_intent_id` | string | no | Payment Intent unique identifier of the latest payment (if any) successfully paid by your shopper via the payment link. |
| `metadata` | object | no | A set of key-value pairs that you can attach to the Payment Link for storing additional information. |
| `reference` | string | no | A reference only visible to the merchant. |
| `reusable` | boolean | no | Specifies whether the payment link can be used once or multiple times. |
| `status` | string | no | Status of the payment link, one of `UNPAID` or `PAID`. The payment link transitions from `UNPAID` to `PAID` when a payment is received. Values: `UNPAID`, `PAID` |
| `successful_payment_intent_count` | integer | no | The number of Payment Intents successfully paid by your shopper via the payment link. |
| `supported_currencies` | array[string] | no | The supported currencies for Flexible pricing. |
| `title` | string | no | The title of the payment link that is displayed in the payment checkout page. |
| `ui_options` | object | no | UI configuration for the payment checkout page. |
| `updated_at` | string | no | Timestamp in ISO 8601 format when the payment link was last updated. |
| `url` | string | no | The payment URL to the checkout page. |

### Retrieve a Payment Link

`GET /api/v1/pa/payment_links/{id}`

Retrieve information about an existing payment link.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment link unique identifier. |

### List all Payment Links

`GET /api/v1/pa/payment_links`

Retrieves a list of payment links that a merchant has created previously.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `active` | boolean | no | The payment link's active status, either `true` or `false`. |
| `from_created_at` | string | no | Specifies start time (inclusive) in ISO 8601 for a range query by created timestamp. |
| `page_num` | integer | no | Page number starting from 0. |
| `page_size` | integer | no | Number of payment links to be listed per page. |
| `reusable` | boolean | no | Specifies whether the payment link can be used once or multiple times. |
| `status` | string | no | Status of the payment link, one of `UNPAID` or `PAID`. Values: `UNPAID`, `PAID` |
| `to_created_at` | string | no | Specifies end time (exclusive) in ISO 8601 for a range query by created timestamp. |

### Update a Payment Link

`POST /api/v1/pa/payment_links/{id}/update`

Update attributes of an existing payment link.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment link unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `amount` | number | no | The payment amount. For Fixed pricing only. |
| `collectable_shopper_info` | object | no | Every payment link requires the shopper to enter their name and email address. Additionally, you can request information for the following fields on each payment link. |
| `currency` | string | no | The payment currency. For Fixed pricing only. |
| `default_currency` | string | no | The default payment currency from the list of `supported_currencies`. The shopper can change the payment currency during checkout. |
| `description` | string | no | The additional description text that appears under the title in the payment checkout page. |
| `metadata` | object | no | A set of key-value pairs that you can attach to the Payment Link for storing additional information. |
| `reference` | string | no | A reference only visible to the merchant. |
| `supported_currencies` | array[string] | no | List of supported currencies the shopper may choose from. For Flexible pricing only. |
| `title` | string | no | The title of the payment link that is displayed in the payment checkout page. |
| `ui_options` | object | no | UI configuration for the payment checkout page. |

### Send a Payment Link

`POST /api/v1/pa/payment_links/{id}/notify_shopper`

Send an email notification to shopper with a payment link.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment link unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `shopper_email` | string | yes | Shopper email address. |

### Activate a Payment Link

`POST /api/v1/pa/payment_links/{id}/activate`

Activates an inactive payment link.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment link unique identifier. |

### Deactivate a Payment Link

`POST /api/v1/pa/payment_links/{id}/deactivate`

Deactivates an active payment link.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment link unique identifier. |

