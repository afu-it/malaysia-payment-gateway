# PayPal Orders v2 And Payments v2

Field tables below are generated from the official schemas `https://developer.paypal.com/api/orders/v2/schema.json` (spec 2.36) and `https://developer.paypal.com/api/payments/v2/schema.json` (spec 2.12), checked 2026-10-21. "Req" is the schema `required` flag. Long descriptions are shortened with `...`; open the schema for the full text. Nested objects not expanded here (shipping, payer, card, supplementary_data, platform fees) are in the schema.

## Operations

| Operation | Method and path | Key headers | Body | Success |
| --- | --- | --- | --- | --- |
| Create order | `POST /v2/checkout/orders` | `PayPal-Request-Id`, `Prefer` | `order_request` | `200`/`201`/`202` `order` |
| Show order | `GET /v2/checkout/orders/{id}` | | query `fields=payment_source` (optional) | `200` `order` |
| Update order | `PATCH /v2/checkout/orders/{id}` | | JSON Patch array | `204` |
| Confirm payment source | `POST /v2/checkout/orders/{id}/confirm-payment-source` | `Prefer` | `payment_source` (required), `processing_instruction`, `application_context` | `200` `order` |
| Authorize order | `POST /v2/checkout/orders/{id}/authorize` | `PayPal-Request-Id`, `Prefer` | optional `payment_source` | `200`/`201`/`202`/`207` |
| Capture order | `POST /v2/checkout/orders/{id}/capture` | `PayPal-Request-Id`, `Prefer` | optional `payment_source` (send `{}` or no body for PayPal wallet) | `200`/`201`/`202`/`207` `order` |
| Add tracking | `POST /v2/checkout/orders/{id}/track` | | tracker | |
| Update tracker | `PATCH /v2/checkout/orders/{id}/trackers/{tracker_id}` | | JSON Patch | |
| Show authorization | `GET /v2/payments/authorizations/{authorization_id}` | | | `200` |
| Capture authorization | `POST /v2/payments/authorizations/{authorization_id}/capture` | `PayPal-Request-Id`, `Prefer` | `capture_request` | `200`/`201` |
| Reauthorize | `POST /v2/payments/authorizations/{authorization_id}/reauthorize` | `PayPal-Request-Id`, `Prefer` | `reauthorize_request` | `200`/`201` |
| Void authorization | `POST /v2/payments/authorizations/{authorization_id}/void` | `PayPal-Request-Id`, `Prefer` | none | `200` or `204` |
| Show capture | `GET /v2/payments/captures/{capture_id}` | | | `200` |
| Refund capture | `POST /v2/payments/captures/{capture_id}/refund` | `PayPal-Request-Id`, `Prefer` | `refund_request` (`{}` = full) | `200`/`201` `refund` |
| Show refund | `GET /v2/payments/refunds/{refund_id}` | | | `200` |
| Find eligible methods | `POST /v2/payments/find-eligible-methods` | | see schema | |

Errors on all: `400`, `401`, `403`, `404`, `409` (capture/authorize), `422`, `500`, body `error` (see bottom table).

## Minimal MYR Create Order (redirect flow, no JS SDK)

```bash
curl -X POST https://api-m.sandbox.paypal.com/v2/checkout/orders \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "PayPal-Request-Id: <UUID_PER_LOCAL_ORDER>" \
  -d '{
    "intent": "CAPTURE",
    "purchase_units": [{
      "reference_id": "default",
      "custom_id": "<LOCAL_ORDER_ID>",
      "invoice_id": "<UNIQUE_INVOICE_NO>",
      "amount": { "currency_code": "MYR", "value": "49.90" }
    }],
    "payment_source": { "paypal": { "experience_context": {
      "brand_name": "<SHOP_NAME>",
      "shipping_preference": "NO_SHIPPING",
      "user_action": "PAY_NOW",
      "payment_method_preference": "IMMEDIATE_PAYMENT_REQUIRED",
      "return_url": "https://<YOUR_SITE>/paypal/return?ref=<LOCAL_ORDER_ID>",
      "cancel_url": "https://<YOUR_SITE>/paypal/cancel?ref=<LOCAL_ORDER_ID>"
    } } }
  }'
```

With `payment_source.paypal` the response status is `PAYER_ACTION_REQUIRED` and the approval URL is the `links[]` entry with `rel: "payer-action"`; without `payment_source` the order is `CREATED` and the link is `rel: "approve"`. Then capture with `POST /v2/checkout/orders/{id}/capture`. `user_action: PAY_NOW` is for when the final amount is known at creation; `CONTINUE` (default) sends the buyer back to your site to review. Sources: `/whats-an-order`, `/api/rest/integration/orders-api/api-use-cases/standard`, Orders v2 schema.

`payment_method_preference: IMMEDIATE_PAYMENT_REQUIRED` accepts only immediate funding (card, PayPal balance, instant ACH) so the capture is not `pending`. `invoice_id` must be unique per merchant account by default (duplicates fail with `DUPLICATE_INVOICE_ID`); keep it unique. `soft_descriptor` is truncated to 22 characters on the card statement, shown as `PAYPAL *<merchant descriptor> <soft descriptor>`.

## PATCH Order

Allowed only while status is `CREATED` or `APPROVED`. Body is an array of `{ "op": "add|remove|replace|move|copy|test", "path": "...", "value": ..., "from": ... }`. With one purchase unit and no `reference_id`, PayPal sets it to `default`, path form `/purchase_units/@reference_id=='default'/amount`.

Patchable (from the schema): `intent` (replace), `payer` (replace, add), `purchase_units` (replace, add), `purchase_units[].custom_id` (replace, add, remove), `.description` (replace, add, remove), `.payee.email` (replace), `.shipping.name|email_address|phone_number|options|address|type` (replace, add), `.soft_descriptor` (replace, remove), `.amount` (replace), `.items` (replace, add, remove), `.invoice_id` (replace, add, remove), `.payment_instruction` (replace), `.payment_instruction.disbursement_mode` (replace), `.payment_instruction.payee_receivable_fx_rate_id` and `.platform_fees` (replace, add, remove), `.supplementary_data.airline|card` (replace, add, remove), `application_context.client_configuration` (replace, add).

## Settlement Checks On A Capture Response

1. HTTP 2xx and `purchase_units[0].payments.captures[0]` present.
2. `captures[0].status == "COMPLETED"`; `PENDING` means wait for webhook; `DECLINED`/`FAILED` means failed.
3. `captures[0].amount.currency_code == "MYR"` and `value` equals expected.
4. `custom_id` / `invoice_id` equals your local order.
5. Record `captures[0].id` (for refunds), `seller_receivable_breakdown.paypal_fee` and `net_amount`.
6. On `422 INSTRUMENT_DECLINED` from capture, restart the buyer flow so they pick another funding source (`/v5/checkout/handle-funding-failure`). On `ORDER_ALREADY_CAPTURED` (for example a retry after timeout), `GET` the order and settle from it.

## Orders v2 Field Tables

#### order_request (body of POST /v2/checkout/orders)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `intent` | string (checkout_payment_intent) | yes | The intent to either capture payment immediately or authorize a payment for an order after order creation. [enum: CAPTURE, AUTHORIZE] |
| `processing_instruction` | string (processing_instruction) | no | The instruction to process an order. [enum: ORDER_COMPLETE_ON_PAYMENT_APPROVAL; default=NO_INSTRUCTION] |
| `payer` | object | no | DEPRECATED. The customer is also known as the payer. The Payer object was intended to only be used with the `payment_source.paypal` object. In order to make this design ... |
| `purchase_units` | array<> | yes | An array of purchase units. Each purchase unit establishes a contract between a payer and the payee. Each purchase unit represents either a full or partial order that ... [maxItems=10] |
| `payment_source` | object (payment_source) | no | The payment source definition. |
| `application_context` | object | no | Customize the payer experience during the approval process for the payment with PayPal. |

#### purchase_unit_request

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `reference_id` | string | no | The API caller-provided external ID for the purchase unit. Required for multiple purchase units when you must update the order through `PATCH`. If you omit this value ... [maxLength=256] |
| `amount` | object | yes | The total order amount with an optional breakdown that provides details, such as the total item amount, total tax amount, shipping, handling, insurance, and discounts, ... |
| `payee` | object | no | The merchant who receives payment for this transaction. |
| `payment_instruction` | object (payment_instruction) | no | Any additional payment instructions to be consider during payment processing. This processing instruction is applicable for Capturing an order or Authorizing an Order. |
| `description` | string | no | This field supports up to 3,000 characters, but any content beyond 127 characters (including spaces) will be truncated. The 127 character limit is reflected in the ... [maxLength=3000] |
| `custom_id` | string | no | The API caller-provided external ID. Used to reconcile client transactions with PayPal transactions. Appears in transaction and settlement reports but is not visible to ... [maxLength=255] |
| `invoice_id` | string | no | The API caller-provided external invoice number for this order. Appears in both the payer's transaction history and the emails that the payer receives. invoice_id values ... [maxLength=127] |
| `soft_descriptor` | string | no | This field supports up to 127 characters, but any content beyond 22 characters (including spaces) will be truncated. The 22 character limit is reflected in the response ... [maxLength=1000] |
| `items` | array<> | no | An array of items that the customer purchases from the merchant. |
| `shipping` | object | no | The name and address of the person to whom to ship the items. |
| `supplementary_data` | object | no | Contains Supplementary Data. |

#### amount_with_breakdown (purchase_units[].amount)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `currency_code` | string (currency_code) | yes | The [three-character ISO-4217 currency code](/api/rest/reference/currency-codes/) that identifies the currency. [minLength=3; maxLength=3] |
| `value` | string | yes | The value, which might be:An integer for currencies like `JPY` that are not typically fractional.A decimal fraction for currencies like `TND` that are subdivided into ... [maxLength=32; pattern=^((-?[0-9]+)|(-?([0-9]+)?[.][0-9]+))$] |
| `breakdown` | object (amount_breakdown) | no | The breakdown of the amount. Breakdown provides details such as total item amount, total tax amount, shipping, handling, insurance, and discounts, if any. |
| `breakdown.item_total` | object | no | The subtotal for all items. Required if the request includes `purchase_units[].items[].unit_amount`. Must equal the sum of `(items[].unit_amount * items[].quantity)` for ... |
| `breakdown.shipping` | object | no | The shipping fee for all items within a given `purchase_unit`. shipping.value can not be a negative number. |
| `breakdown.handling` | object | no | The handling fee for all items within a given `purchase_unit`. handling.value can not be a negative number. |
| `breakdown.tax_total` | object | no | The total tax for all items. Required if the request includes `purchase_units.items.tax`. Must equal the sum of `(items[].tax * items[].quantity)` for all items. ... |
| `breakdown.insurance` | object | no | The insurance fee for all items within a given `purchase_unit`. insurance.value can not be a negative number. |
| `breakdown.shipping_discount` | object | no | The shipping discount for all items within a given `purchase_unit`. shipping_discount.value can not be a negative number. |
| `breakdown.discount` | object | no | The discount for all items within a given `purchase_unit`. discount.value can not be a negative number. |

#### money (every amount object)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `currency_code` | string (currency_code) | yes | The [three-character ISO-4217 currency code](/api/rest/reference/currency-codes/) that identifies the currency. [minLength=3; maxLength=3] |
| `value` | string | yes | The value, which might be:An integer for currencies like `JPY` that are not typically fractional.A decimal fraction for currencies like `TND` that are subdivided into thousandths.For the required number of decimal places for a currency code, see [Currency Codes](/api/rest/reference/currency-codes/). [maxLength=32; pattern=^((-?[0-9]+)|(-?([0-9]+)?[.][0-9]+))$] |

#### item (purchase_units[].items[])

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `name` | string | yes | The item name or title. [maxLength=127] |
| `unit_amount` | object | yes | The item price or rate per unit. If you specify unit_amount, purchase_units[].amount.breakdown.item_total is required. Must equal unit_amount * ... |
| `tax` | object | no | The item tax for each unit. If tax is specified, purchase_units[].amount.breakdown.tax_total is required. Must equal tax * quantity for all items. ... |
| `quantity` | string | yes | The item quantity. Must be a whole number. [maxLength=10; pattern=^[1-9][0-9]{0,9}$] |
| `description` | string | no | The detailed item description. [maxLength=2048] |
| `sku` | string | no | The stock keeping unit (SKU) for the item. [maxLength=127] |
| `url` | string | no | The URL to the item being purchased. Visible to buyer and used in buyer experiences. [maxLength=2048] |
| `category` | string | no | The item category type. [enum: DIGITAL_GOODS, PHYSICAL_GOODS, DONATION] |
| `image_url` | string | no | The URL of the item's image. Supported image formats: JPEG (.jpg, .jpeg), PNG (.png), GIF (.gif). [maxLength=2048] |
| `upc` | object | no | The Universal Product Code of the item. |
| `billing_plan` | object (order_billing_plan) | no | Metadata for merchant-managed recurring billing plans. Valid only during the saved payment method token or billing agreement creation. |

#### payee

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `email_address` | string | no | The email address of merchant. [minLength=3; maxLength=254] |
| `merchant_id` | string | no | The encrypted PayPal account ID of the merchant. [minLength=13; maxLength=13; pattern=^[2-9A-HJ-NP-Z]{13}$] |

#### payment_source (top level keys)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `card` | object (card_request) | no | The payment card to use to fund a payment. Can be a credit or debit card.Note: Passing card number, cvv and expiry ... |
| `token` | object (token) | no | The tokenized payment source to fund a payment. |
| `paypal` | object | no | Indicates that PayPal Wallet is the payment source. Main use of this selection is to provide additional instructions ... |
| `bancontact` | object | no | Bancontact is the most popular online payment in Belgium. [More Details](https://www.bancontact.com/). |
| `blik` | object | no | BLIK is a mobile payment system, created by Polish Payment Standard in order to allow millions of users to pay in ... |
| `eps` | object | no | The eps transfer is an online payment method developed by many Austrian banks. [More ... |
| `giropay` | object | no | Giropay is an Internet payment System in Germany, based on online banking. [More Details](https://giropay.de/). |
| `ideal` | object | no | The Dutch payment method iDEAL is an online payment method that enables consumers to pay online through their own bank. ... |
| `mybank` | object | no | MyBank is an e-authorisation solution which enables safe digital payments and identity authentication through a ... |
| `p24` | object | no | P24 (Przelewy24) is a secure and fast online bank transfer service linked to all the major banks in Poland. [More ... |
| `sofort` | object | no | SOFORT Banking is a real-time bank transfer payment method that buyers use to transfer funds directly to merchants from ... |
| `trustly` | object | no | Trustly is a payment method that allows customers to shop and pay from their bank account. [More ... |
| `apple_pay` | object | no | ApplePay payment source, allows buyer to pay using ApplePay, both on Web as well as on Native. |
| `google_pay` | object | no | Google Pay payment source, allows buyer to pay using Google Pay. |
| `venmo` | object | no | Information needed to indicate that Venmo is being used to fund the payment. |
| `crypto` | object | no | Indicates that Crypto Wallet is the payment source. |

#### payment_source.paypal

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `vault_id` | string | no | The PayPal-generated ID for the payment_source stored within the Vault. [maxLength=255; pattern=^[0-9a-zA-Z_-]+$] |
| `email_address` | string | no | The email address of the PayPal account holder. [minLength=3; maxLength=254] |
| `name` | object | no | The name of the PayPal account holder. Supports only the `given_name` and `surname` properties. |
| `phone` | object | no | The phone number of the customer. Available only when you enable the **Contact Telephone Number** option in the **Profile & Settings** for the merchant's PayPal account. ... |
| `birth_date` | string | no | The birth date of the PayPal account holder in `YYYY-MM-DD` format. [minLength=10; maxLength=10] |
| `tax_info` | object | no | The tax information of the PayPal account holder. Required only for Brazilian PayPal account holder's. Both `tax_id` and `tax_id_type` are required. |
| `address` | object | no | The address of the PayPal account holder. Supports only the `address_line_1`, `address_line_2`, `admin_area_1`, `admin_area_2`, `postal_code`, and `country_code` ... |
| `attributes` | object | no | Additional attributes associated with the use of this wallet. |
| `experience_context` | object (paypal_wallet_experience_context) | no | Customizes the payer experience during the approval process for payment with PayPal.Note: Partners and Marketplaces might configure brand_name and shipping_preference ... |
| `billing_agreement_id` | string (billing_agreement_id) | no | The PayPal billing agreement ID. References an approved recurring payment for goods or services. [minLength=2; maxLength=128; pattern=^[a-zA-Z0-9-]+$] |
| `stored_credential` | object (paypal_wallet_stored_credential) | no | Provides additional details to process a payment using the PayPal wallet billing agreement or a vaulted payment method that has been stored or is intended to be stored. |

#### payment_source.paypal.experience_context

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `brand_name` | string | no | The label that overrides the business name in the PayPal account on the PayPal site. The pattern is defined by an external party and supports Unicode. [maxLength=127] |
| `locale` | string | no | The BCP 47-formatted locale of pages that the PayPal payment experience shows. PayPal supports a five-character code. For example, `da-DK`, `he-IL`, `id-ID`, `ja-JP`, `no-NO`, `pt-BR`, `ru-RU`, ... [minLength=2; maxLength=10] |
| `shipping_preference` | string | no | The location from which the shipping address is derived. [enum: GET_FROM_FILE, NO_SHIPPING, SET_PROVIDED_ADDRESS; default=GET_FROM_FILE] |
| `contact_preference` | string | no | The preference to display the contact information (buyer’s shipping email & phone number) on PayPal's checkout for easy merchant-buyer communication. [enum: NO_CONTACT_INFO, UPDATE_CONTACT_INFO, RETAIN_CONTACT_INFO; default=NO_CONTACT_INFO] |
| `return_url` | string | no | The URL where the customer will be redirected upon approving a payment. |
| `cancel_url` | string | no | The URL where the customer will be redirected upon cancelling the payment approval. |
| `app_switch_context` | object | no | Merchants can use this to switch buyers from their website/application to the PayPal consumer app to review and approve the transaction. |
| `landing_page` | string | no | The type of landing page to show on the PayPal site for customer checkout. [enum: LOGIN, GUEST_CHECKOUT, NO_PREFERENCE, BILLING; default=NO_PREFERENCE] |
| `user_action` | string | no | Configures a Continue or Pay Now checkout flow. [enum: CONTINUE, PAY_NOW; default=CONTINUE] |
| `payment_method_preference` | string | no | The merchant-preferred payment methods. [enum: UNRESTRICTED, IMMEDIATE_PAYMENT_REQUIRED; default=UNRESTRICTED] |
| `order_update_callback_config` | object | no | Merchant provided Order Update callback configuration for PayPal Wallet.PayPal will call back merchant when the specified event occurs.we recommend merchants to pass both the shipping_options and ... |

#### order (response of create, get, capture, authorize)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `create_time` | string | no | The date and time when the transaction occurred, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64; readOnly] |
| `update_time` | string | no | The date and time when the transaction was last updated, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64; readOnly] |
| `id` | string | no | The ID of the order. [readOnly] |
| `payment_source` | object | no | The payment source used to fund the payment. |
| `intent` | string | no | The intent to either capture payment immediately or authorize a payment for an order after order creation. [enum: CAPTURE, AUTHORIZE] |
| `processing_instruction` | string (processing_instruction) | no | The instruction to process an order. [enum: ORDER_COMPLETE_ON_PAYMENT_APPROVAL; default=NO_INSTRUCTION] |
| `payer` | object | no | DEPRECATED. The customer is also known as the payer. The Payer object was intended to only be used with the `payment_source.paypal` object. In order to make this design more clear, the details in the ... |
| `purchase_units` | array<> | no | An array of purchase units. Each purchase unit establishes a contract between a customer and merchant. Each purchase unit represents either a full or partial order that the customer intends to ... [maxItems=10] |
| `status` | string | no | The order status. [enum: CREATED, SAVED, APPROVED, VOIDED, COMPLETED, PAYER_ACTION_REQUIRED; readOnly] |
| `links` | array<> | no | An array of request-related HATEOAS links. To complete payer approval, use the `approve` link to redirect the payer. The API caller has 6 hours (default setting, this which can be changed by your ... [readOnly] |

#### purchase_unit (response)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `reference_id` | string | no | The API caller-provided external ID for the purchase unit. Required for multiple purchase units when you must update the order through `PATCH`. If ... [maxLength=256] |
| `amount` | object (amount_with_breakdown) | no | The total order amount with an optional breakdown that provides details, such as the total item amount, total tax amount, shipping, handling, ... |
| `payee` | object | no | The merchant who receives payment for this transaction. |
| `payment_instruction` | object (payment_instruction) | no | Any additional payment instructions to be consider during payment processing. This processing instruction is applicable for Capturing an order or ... |
| `description` | string | no | The purchase description. [maxLength=127] |
| `custom_id` | string | no | The API caller-provided external ID. Used to reconcile API caller-initiated transactions with PayPal transactions. Appears in transaction and ... [maxLength=255] |
| `invoice_id` | string | no | The API caller-provided external invoice ID for this order. [maxLength=127] |
| `id` | string | no | The PayPal-generated ID for the purchase unit. This ID appears in both the payer's transaction history and the emails that the payer receives. In ... [maxLength=19] |
| `soft_descriptor` | string | no | The payment descriptor on account transactions on the customer's credit card statement, that PayPal sends to processors. The maximum length of the ... [maxLength=22] |
| `items` | array<> | no | An array of items that the customer purchases from the merchant. |
| `shipping` | object | no | The shipping address and method. |
| `supplementary_data` | object | no | Supplementary data about this payment. Merchants and partners can add Level 2 and 3 data to payments to reduce risk and payment processing costs. For ... |
| `payments` | object | no | The comprehensive history of payments for the purchase unit. [readOnly] |

#### payment_collection (purchase_units[].payments)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `authorizations` | array<> | no | An array of authorized payments for a purchase unit. A purchase unit can have zero or more authorized payments. |
| `captures` | array<> | no | An array of captured payments for a purchase unit. A purchase unit can have zero or more captured payments. |
| `refunds` | array<> | no | An array of refunds for a purchase unit. A purchase unit can have zero or more refunds. |

#### capture (purchase_units[].payments.captures[])

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `status` | string | no | The status of the captured payment. [enum: COMPLETED, DECLINED, PARTIALLY_REFUNDED, PENDING, REFUNDED, FAILED; readOnly] |
| `status_details` | object | no | The details of the captured payment status. [readOnly] |
| `id` | string | no | The PayPal-generated ID for the captured payment. [readOnly] |
| `amount` | object | no | The amount for this captured payment. [readOnly] |
| `invoice_id` | string | no | The API caller-provided external invoice number for this order. Appears in both the payer's transaction history and the emails that the payer receives. [readOnly] |
| `custom_id` | string | no | The API caller-provided external ID. Used to reconcile API caller-initiated transactions with PayPal transactions. Appears in transaction and settlement reports. [maxLength=255] |
| `network_transaction_reference` | object | no | Reference values used by the card network to identify a transaction. |
| `seller_protection` | object | no | The level of protection offered as defined by [PayPal Seller Protection for Merchants](https://www.paypal.com/us/webapps/mpp/security/seller-protection). [readOnly] |
| `final_capture` | boolean | no | Indicates whether you can make additional captures against the authorized payment. Set to `true` if you do not intend to capture additional payments against the ... [default=False; readOnly] |
| `seller_receivable_breakdown` | object | no | The detailed breakdown of the capture activity. This is not available for transactions that are in pending state. [readOnly] |
| `disbursement_mode` | string | no | The funds that are held on behalf of the merchant. [enum: INSTANT, DELAYED; default=INSTANT] |
| `links` | array<> | no | An array of related [HATEOAS links](/docs/api/reference/api-responses/#hateoas-links). [readOnly] |
| `processor_response` | object | no | An object that provides additional processor information for a direct credit card transaction. |
| `create_time` | string | no | The date and time when the transaction occurred, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64; readOnly] |
| `update_time` | string | no | The date and time when the transaction was last updated, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64; readOnly] |

#### seller_receivable_breakdown

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `gross_amount` | object | yes | The amount for this captured payment in the currency of the transaction. |
| `paypal_fee` | object | no | The applicable fee for this captured payment in the currency of the transaction. |
| `paypal_fee_in_receivable_currency` | object | no | The applicable fee for this captured payment in the receivable currency. Returned only in cases the fee is charged in the receivable currency. ... |
| `net_amount` | object | no | The net amount that the payee receives for this captured payment in their PayPal account. The net amount is computed as gross_amount minus the ... |
| `receivable_amount` | object | no | The net amount that is credited to the payee's PayPal account. Returned only when the currency of the captured payment is different from the currency ... |
| `exchange_rate` | object | no | The exchange rate that determines the amount that is credited to the payee's PayPal account. Returned when the currency of the captured payment is ... [readOnly] |
| `platform_fees` | array<> | no | An array of platform or partner fees, commissions, or brokerage fees that associated with the captured payment. [maxItems=1] |

#### error (all 4xx/5xx bodies)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `name` | string | yes | The human-readable, unique name of the error. |
| `message` | string | yes | The message that describes the error. |
| `debug_id` | string | yes | The PayPal internal ID. Used for correlation purposes. |
| `details` | array<> | no | An array of additional details about the error. |
| `links` | array<> | no | An array of request-related [HATEOAS links](/api/rest/responses/#hateoas-links). [readOnly] |

#### error_details (error.details[])

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `field` | string | no | The field that caused the error. If this field is in the body, set this value to the field's JSON pointer value. Required for client-side errors. |
| `value` | string | no | The value of the field that caused the error. |
| `location` | string | no | The location of the field that caused the error. Value is `body`, `path`, or `query`. [default=body] |
| `issue` | string | yes | The unique, fine-grained application-level error code. |
| `links` | array<> | no | An array of request-related [HATEOAS links](/api/rest/responses/#hateoas-links) that are either relevant to the issue by providing additional information or offering ... [maxItems=4; readOnly] |
| `description` | string | no | The human-readable description for an issue. The description can change over the lifetime of an API, so clients must not depend on this value. |

## Payments v2 Field Tables

#### capture_request (POST /v2/payments/authorizations/{authorization_id}/capture)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `invoice_id` | string | no | The API caller-provided external invoice number for this order. Appears in both the payer's transaction history and the emails that the payer receives. [maxLength=127] |
| `note_to_payer` | string | no | An informational note about this settlement. Appears in both the payer's transaction history and the emails that the payer receives. [maxLength=255] |
| `amount` | object | no | The amount to capture. To capture a portion of the full authorized amount, specify an amount. If amount is not specified, the full authorized amount is captured. The ... |
| `final_capture` | boolean | no | Indicates whether you can make additional captures against the authorized payment. Set to `true` if you do not intend to capture additional payments against the ... [default=False] |
| `payment_instruction` | object | no | Any additional payment instructions to be consider during payment processing. This processing instruction is applicable for Capturing an order or Authorizing an Order. |
| `soft_descriptor` | string | no | The payment descriptor on the payer's account statement. [maxLength=22] |

#### reauthorize_request

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `amount` | object | no | The amount to reauthorize for an authorized payment. |

#### authorization (response of get/void/reauthorize)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `status` | string | no | The status for the authorized payment. [enum: CREATED, CAPTURED, DENIED, PARTIALLY_CAPTURED, VOIDED, PENDING; readOnly] |
| `status_details` | object | no | The details of the authorized order pending status. [readOnly] |
| `id` | string | no | The PayPal-generated ID for the authorized payment. [readOnly] |
| `amount` | object | no | The amount for this authorized payment. [readOnly] |
| `invoice_id` | string | no | The API caller-provided external invoice number for this order. Appears in both the payer's transaction history and the emails that the payer ... [readOnly] |
| `custom_id` | string | no | The API caller-provided external ID. Used to reconcile API caller-initiated transactions with PayPal transactions. Appears in transaction and ... [maxLength=255] |
| `network_transaction_reference` | object | no | Reference values used by the card network to identify a transaction. |
| `seller_protection` | object | no | The level of protection offered as defined by [PayPal Seller Protection for ... [readOnly] |
| `expiration_time` | string | no | The date and time when the authorized payment expires, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64; readOnly] |
| `links` | array<> | no | An array of related [HATEOAS links](/docs/api/reference/api-responses/#hateoas-links). [readOnly] |
| `create_time` | string | no | The date and time when the transaction occurred, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64; readOnly] |
| `update_time` | string | no | The date and time when the transaction was last updated, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64; readOnly] |

#### refund_request (POST /v2/payments/captures/{capture_id}/refund)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `amount` | object | no | The amount to refund. To refund a portion of the captured amount, specify an amount. If amount is not specified, an amount equal to captured amount - previous refunds is ... |
| `custom_id` | string | no | The API caller-provided external ID. Used to reconcile API caller-initiated transactions with PayPal transactions. Appears in transaction and settlement reports. The ... [maxLength=127] |
| `invoice_id` | string | no | The API caller-provided external invoice ID for this order. The pattern is defined by an external party and supports Unicode. [maxLength=127] |
| `note_to_payer` | string | no | The reason for the refund. Appears in both the payer's transaction history and the emails that the payer receives. The pattern is defined by an external party and ... [maxLength=255] |
| `payment_instruction` | object | no | Any additional refund instructions to be set during refund payment processing. This object is only applicable to merchants that have been enabled for PayPal Commerce ... |

#### refund (response)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `status` | string | no | The status of the refund. [enum: CANCELLED, FAILED, PENDING, COMPLETED; readOnly] |
| `status_details` | object | no | The details of the refund status. [readOnly] |
| `id` | string | no | The PayPal-generated ID for the refund. [readOnly] |
| `amount` | object | no | The amount that the payee refunded to the payer. [readOnly] |
| `invoice_id` | string | no | The API caller-provided external invoice number for this order. Appears in both the payer's transaction history and the emails that the payer ... [readOnly] |
| `custom_id` | string | no | The API caller-provided external ID. Used to reconcile API caller-initiated transactions with PayPal transactions. Appears in transaction and ... [maxLength=255; pattern=^[A-Za-z0-9-_.,]*$] |
| `acquirer_reference_number` | string | no | Reference ID issued for the card transaction. This ID can be used to track the transaction across processors, card brands and issuing banks. [maxLength=36; pattern=^[a-zA-Z0-9]+$] |
| `note_to_payer` | string | no | The reason for the refund. Appears in both the payer's transaction history and the emails that the payer receives. [readOnly] |
| `seller_payable_breakdown` | object | no | The breakdown of the refund. [readOnly] |
| `payer` | object | no | The details associated with the merchant for this transaction. [readOnly] |
| `links` | array<> | no | An array of related [HATEOAS links](/docs/api/reference/api-responses/#hateoas-links). [readOnly] |
| `create_time` | string | no | The date and time when the transaction occurred, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64; readOnly] |
| `update_time` | string | no | The date and time when the transaction was last updated, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64; readOnly] |
