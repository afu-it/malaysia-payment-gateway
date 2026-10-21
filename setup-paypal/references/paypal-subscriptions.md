# PayPal Subscriptions (Recurring)

Sources: `/subscriptions/about`, `/subscriptions/integrate`, `/subscriptions/webhooks`, `/subscriptions/payment-failure-retry`, `/subscriptions/trial-period`, `/subscriptions/billing-cycle`, `/subscriptions/change-price`, `/subscriptions/change-quantity`, `/subscriptions/pause-or-resume`, Subscriptions v1 schema (spec 1.11) and Catalog Products v1 schema. Checked 2026-10-21. No Malaysia-specific restriction for Subscriptions was found in the docs; MYR plans follow the general MYR in-country rule in `paypal-api.md`.

## Flow

1. `POST /v1/catalogs/products` (name, type `PHYSICAL|DIGITAL|SERVICE`) and store `id` (`PROD-...`).
2. `POST /v1/billing/plans` with `product_id`, `name`, `billing_cycles`, `payment_preferences`. Only one `currency_code` per plan (create another plan for another currency). Store `id` (`P-...`, 26 chars). Default status `ACTIVE`.
3. Buyer approval:
   - JS SDK v5: `?client-id=<ID>&vault=true&intent=subscription` and `createSubscription: (d, actions) => actions.subscription.create({ plan_id })`; `onApprove` gives `data.subscriptionID`.
   - JS SDK v6: `createPayPalSubscriptionSession({ onApprove({ subscriptionId, payerId }) })`, `start(options, Promise<{ subscriptionId }>)` where your server called `POST /v1/billing/subscriptions`.
   - API redirect: `POST /v1/billing/subscriptions` with `plan_id`, `custom_id` (your user/contract ID), `application_context.return_url` and `cancel_url` (both required, 10 to 4000 chars), then send the buyer to the approval URL in the response `links` array (the docs read do not show the subscription link `rel` value; read it from your sandbox response, status starts as `APPROVAL_PENDING`).
4. Server: `GET /v1/billing/subscriptions/{id}`; grant access only when `status == ACTIVE`, `plan_id` and `custom_id` match.
5. Each charge: webhook `PAYMENT.SALE.COMPLETED` ("A payment is made on a subscription"). The docs read do not document the sale resource fields; match the event to your subscription from the payload you receive in sandbox, then re-fetch the subscription (`billing_info.last_payment`, `next_billing_time`) before extending access. Failures: `BILLING.SUBSCRIPTION.PAYMENT.FAILED`; lifecycle: `BILLING.SUBSCRIPTION.ACTIVATED|UPDATED|SUSPENDED|CANCELLED|EXPIRED`.
6. Reconcile with `GET /v1/billing/subscriptions/{id}/transactions?start_time=...&end_time=...` (both required).

## Operations

| Method | Path | Body | Success |
| --- | --- | --- | --- |
| `POST` | `/v1/billing/plans` | `plan_request_POST` | `200`/`201` |
| `GET` | `/v1/billing/plans` | query filters | `200` |
| `GET` / `PATCH` | `/v1/billing/plans/{id}` | JSON Patch | `200` / `204` |
| `POST` | `/v1/billing/plans/{id}/activate` / `deactivate` | none | `204` |
| `POST` | `/v1/billing/plans/{id}/update-pricing-schemes` | `pricing_schemes` (required) | `204` |
| `POST` | `/v1/billing/subscriptions` | `subscription_request_post` | `200`/`201` |
| `GET` | `/v1/billing/subscriptions` | query filters | `200` |
| `GET` / `PATCH` | `/v1/billing/subscriptions/{id}` | JSON Patch | `200` / `204` |
| `POST` | `/v1/billing/subscriptions/{id}/revise` | `plan_id`, `quantity`, `shipping_amount`, `shipping_address`, `application_context`, `plan` (needs buyer consent) | `200` |
| `POST` | `/v1/billing/subscriptions/{id}/suspend` | `reason` (required) | `204` |
| `POST` | `/v1/billing/subscriptions/{id}/cancel` | `reason` (required) | `204` |
| `POST` | `/v1/billing/subscriptions/{id}/activate` | `reason` | `204` |
| `POST` | `/v1/billing/subscriptions/{id}/capture` | `note`, `capture_type: OUTSTANDING_BALANCE`, `amount` (all required) | `200`/`202` |
| `GET` | `/v1/billing/subscriptions/{id}/transactions` | `start_time`, `end_time` | `200` |

`PayPal-Request-Id` keys are stored 72 hours for Subscriptions and Catalog Products.

## Payment Failure Rules

- `payment_preferences.payment_failure_threshold`: consecutive failures before the subscription is `SUSPENDED` (default 0).
- On a failed payment PayPal retries every 5 days, up to twice per billing cycle; if the second retry fails it counts one failure and the amount is added to the outstanding balance for the next cycle (`auto_bill_outstanding`, default true).
- Recover an outstanding balance with `/capture` (`capture_type: OUTSTANDING_BALANCE`).
- `setup_fee_failure_action`: `CONTINUE` or `CANCEL` (default).

## Billing Cycles

- At most two `TRIAL` cycles and one `REGULAR` cycle per plan; `sequence` orders them (trial 1, regular 2). `total_cycles` 1 to 999 for trial; `0` on a regular cycle means infinite (see schema description).
- `frequency.interval_unit`: `DAY`, `WEEK`, `MONTH`, `YEAR`; `interval_count` default 1; maximum `DAY` 365, `WEEK` 52, `MONTH` 12, `YEAR` 1.
- `pricing_scheme.fixed_price` is a `money` object; tiered plans use `pricing_model` `VOLUME|TIERED` with `tiers`.

## Field Tables

#### product_request_POST (POST /v1/catalogs/products)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `id` | string | no | The ID of the product. You can specify the SKU for the product. If you omit the ID, the system generates it. System-generated IDs have the `PROD-` prefix. [minLength=6; maxLength=50] |
| `name` | string | yes | The product name. [maxLength=127] |
| `description` | string | no | The product description. [maxLength=256] |
| `type` | string | yes | The product type. Indicates whether the product is physical or digital goods, or a service. [enum: PHYSICAL, DIGITAL, SERVICE; maxLength=24; pattern=^[A-Z_]+$; default=PHYSICAL] |
| `category` | string (product_category) | no | The product category. [enum: AC_REFRIGERATION_REPAIR, ACADEMIC_SOFTWARE, ACCESSORIES, ACCOUNTING, ADULT, ADVERTISING, AFFILIATED_AUTO_RENTAL, AGENCIES, ... (446 values, see schema); minLength=4; maxLength=256; pattern=^[A-Z_]+$] |
| `image_url` | string | no | The image URL for the product. [maxLength=2000] |
| `home_url` | string | no | The home page URL for the product. [maxLength=2000] |

#### plan_request_POST (POST /v1/billing/plans)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `product_id` | string | yes | The ID of the product created through Catalog Products API. [minLength=22; maxLength=22; pattern=^PROD-[A-Z0-9]*$] |
| `name` | string | yes | The plan name. [maxLength=127] |
| `status` | string | no | The initial state of the plan. Allowed input values are CREATED and ACTIVE. [enum: CREATED, INACTIVE, ACTIVE; maxLength=24; pattern=^[A-Z_]+$; default=ACTIVE] |
| `description` | string | no | The detailed description of the plan. [maxLength=127] |
| `billing_cycles` | array<billing_cycle> (billing_cycle_list) | yes | An array of billing cycles for trial billing and regular billing. A plan can have at most two trial cycles and only one regular cycle. [maxItems=12] |
| `payment_preferences` | object (payment_preferences) | yes | The payment preferences for a subscription. |
| `merchant_preferences` | object (merchant_preferences) | no | The merchant preferences for a subscription. |
| `taxes` | object (taxes) | no | The tax details. |
| `quantity_supported` | boolean | no | Indicates whether you can subscribe to this plan by providing a quantity for the goods or service. [default=False] |

#### billing_cycle

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `pricing_scheme` | object (pricing_scheme) | no | The active pricing scheme for this billing cycle. A free trial billing cycle does not require a pricing scheme. |
| `frequency` | object (frequency) | yes | The frequency details for this billing cycle. |
| `tenure_type` | string | yes | The tenure type of the billing cycle. In case of a plan having trial cycle, only 2 trial cycles are allowed per plan. [enum: REGULAR, TRIAL; maxLength=24; pattern=^[A-Z_]+$] |
| `sequence` | integer | yes | The order in which this cycle is to run among other billing cycles. For example, a trial billing cycle has a `sequence` of `1` while a regular billing cycle has a `sequence` of `2`, so that trial cycle runs before the ... [minimum=1; maximum=99] |
| `total_cycles` | integer | no | The number of times this billing cycle gets executed. Trial billing cycles can only be executed a finite number of times (value between 1 and 999 for total_cycles). Regular billing cycles can be executed infinite times ... [minimum=0; maximum=999; default=1] |

#### pricing_scheme

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `version` | integer | no | The version of the pricing scheme. [minimum=0; maximum=999; readOnly] |
| `fixed_price` | object (money) | no | The fixed amount to charge for the subscription. The changes to fixed amount are applicable to both existing and future subscriptions. For existing ... |
| `pricing_model` | string | no | The pricing model for tiered plan. The `tiers` parameter is required. [enum: VOLUME, TIERED; maxLength=24; pattern=^[A-Z_]+$] |
| `tiers` | array<pricing_tier> (pricing_tier_list) | no | An array of pricing tiers which are used for billing volume/tiered plans. pricing_model field has to be specified. [maxItems=32] |
| `create_time` | string (date_time) | no | The date and time when this pricing scheme was created, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64] |
| `update_time` | string (date_time) | no | The date and time when this pricing scheme was last updated, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64] |

#### frequency

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `interval_unit` | string | yes | The interval at which the subscription is charged or billed. [enum: DAY, WEEK, MONTH, YEAR; maxLength=24; pattern=^[A-Z_]+$] |
| `interval_count` | integer | no | The number of intervals after which a subscriber is billed. For example, if the `interval_unit` is `DAY` with an `interval_count` of `2`, the subscription is billed once every two days. The following table lists the maximum allowed values for the ... [minimum=1; maximum=365; default=1] |

#### payment_preferences

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `auto_bill_outstanding` | boolean | no | Indicates whether to automatically bill the outstanding amount in the next billing cycle. [default=True] |
| `setup_fee` | object (money) | no | The initial set-up fee for the service. |
| `setup_fee_failure_action` | string | no | The action to take on the subscription if the initial payment for the setup fails. [enum: CONTINUE, CANCEL; maxLength=24; pattern=^[A-Z_]+$; default=CANCEL] |
| `payment_failure_threshold` | integer | no | The maximum number of payment failures before a subscription is suspended. For example, if `payment_failure_threshold` is `2`, the subscription automatically updates to the `SUSPEND` state if two consecutive payments ... [minimum=0; maximum=999; default=0] |

#### subscription_request_post (POST /v1/billing/subscriptions)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `plan_id` | string | yes | The ID of the plan. [minLength=26; maxLength=26; pattern=^P-[A-Z0-9]*$] |
| `start_time` | string (date_time) | no | The date and time when the subscription started, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64] |
| `quantity` | string | no | The quantity of the product in the subscription. [maxLength=32; pattern=^([0-9]+|([0-9]+)?[.][0-9]+)$] |
| `shipping_amount` | object (money) | no | The shipping charges. |
| `subscriber` | object (subscriber_request) | no | |
| `auto_renewal` | boolean | no | DEPRECATED. Indicates whether the subscription auto-renews after the billing cycles complete. [default=False] |
| `application_context` | object (application_context) | no | DEPRECATED. The application context, which customizes the payer experience during the subscription approval process with PayPal. |
| `custom_id` | string | no | The custom id for the subscription. Can be invoice id. [maxLength=127; pattern=^[\x20-\x7E]+] |
| `plan` | object (plan_override) | no | An inline plan object to customise the subscription. You can override plan level default attributes by providing customised values for the subscription in this object. |
| `merchant_inventory` |  (merchant_inventory) | no | Merchant's inventory identifier details for the WPS hosted button. |

#### application_context (subscription)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `brand_name` | string | no | The label that overrides the business name in the PayPal account on the PayPal site. [maxLength=127] |
| `locale` | string (language) | no | The BCP 47-formatted locale of pages that the PayPal payment experience shows. PayPal supports a five-character code. For example, `da-DK`, `he-IL`, `id-ID`, `ja-JP`, ... [minLength=2; maxLength=10] |
| `shipping_preference` | string | no | The location from which the shipping address is derived. [enum: GET_FROM_FILE, NO_SHIPPING, SET_PROVIDED_ADDRESS; maxLength=24; pattern=^[A-Z_]+$; default=GET_FROM_FILE] |
| `user_action` | string | no | Configures the label name to `Continue` or `Subscribe Now` for subscription consent experience. [enum: CONTINUE, SUBSCRIBE_NOW; maxLength=24; pattern=^[A-Z_]+$; default=SUBSCRIBE_NOW] |
| `payment_method` | object | no | The customer and merchant payment preferences. |
| `return_url` | string | yes | The URL where the customer is redirected after the customer approves the payment. [minLength=10; maxLength=4000] |
| `cancel_url` | string | yes | The URL where the customer is redirected after the customer cancels the payment. [minLength=10; maxLength=4000] |

#### subscription (response)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `status` | string | no | The status of the subscription. [enum: APPROVAL_PENDING, APPROVED, ACTIVE, SUSPENDED, CANCELLED, EXPIRED; maxLength=24; pattern=^[A-Z_]+$] |
| `status_change_note` | string | no | The reason or notes for the status of the subscription. [maxLength=128] |
| `status_update_time` | string (date_time) | no | The date and time, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). Seconds are required while fractional seconds ... [minLength=20; maxLength=64] |
| `id` | string | no | The PayPal-generated ID for the subscription. [minLength=3; maxLength=50; readOnly] |
| `plan_id` | string | no | The ID of the plan. [minLength=3; maxLength=50] |
| `start_time` | string (date_time) | no | The date and time, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). Seconds are required while fractional seconds ... [minLength=20; maxLength=64] |
| `quantity` | string | no | The quantity of the product in the subscription. [maxLength=32; pattern=^([0-9]+|([0-9]+)?[.][0-9]+)$] |
| `shipping_amount` | object (money) | no | The currency and amount for a financial transaction, such as a balance or payment due. |
| `subscriber` | object (subscriber) | no | |
| `billing_info` | object (subscription_billing_info) | no | The billing details for the subscription. If the subscription was or is active, these fields are populated. |
| `create_time` | string (date_time) | no | The date and time, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). Seconds are required while fractional seconds ... [minLength=20; maxLength=64] |
| `update_time` | string (date_time) | no | The date and time, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). Seconds are required while fractional seconds ... [minLength=20; maxLength=64] |
| `payer_holding_preference` |  (payer_holding_preference) | no | The payer's holding preference for subscriptions payments. |
| `custom_id` | string | no | The custom id for the subscription. Can be invoice id. [maxLength=127; pattern=^[\x20-\x7E]+] |
| `plan_overridden` | boolean | no | Indicates whether the subscription has overridden any plan attributes. [readOnly] |
| `plan` | object | no | The plan details. |
| `links` | array<link_description> (link_description_list) | no | An array of request-related [HATEOAS links](/docs/api/reference/api-responses/#hateoas-links). [maxItems=10; readOnly] |

#### subscription_billing_info

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `outstanding_balance` | object (money) | yes | The total pending bill amount, to be paid by the subscriber. |
| `cycle_executions` | array<cycle_execution> (cycle_execution_list) | no | The trial and regular billing executions. [maxItems=3; readOnly] |
| `last_payment` | object (last_payment_details) | no | The details for the last payment of the subscription. |
| `next_billing_time` | string (date_time) | no | The next date and time for billing this subscription, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64] |
| `final_payment_time` | string (date_time) | no | The date and time when the final billing cycle occurs, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [minLength=20; maxLength=64] |
| `failed_payments_count` | integer | yes | The number of consecutive payment failures. Resets to `0` after a successful payment. If this reaches the `payment_failure_threshold` value, the subscription updates to ... [minimum=0; maximum=999; readOnly] |
| `last_failed_payment` | object (failed_payment_details) | no | The details for the last failed payment of the subscription. |
