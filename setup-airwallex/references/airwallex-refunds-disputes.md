# Airwallex Refunds, Cancellations and Disputes

Checked 2026-10-21.

## Source URLs

- Refunds guide: `https://www.airwallex.com/docs/payments/payment-operations/manage-payments/refunds`
- Cancellations: `https://www.airwallex.com/docs/payments/payment-operations/manage-payments/cancellations`
- Refunds API: `https://www.airwallex.com/docs/api/payments/refunds`
- Disputes overview: `https://www.airwallex.com/docs/payments/payment-operations/disputes`
- Handle disputes using APIs: `https://www.airwallex.com/docs/payments/payment-operations/disputes/handle-disputes-using-our-apis`
- Payment Disputes API: `https://www.airwallex.com/docs/api/payments/payment_disputes`

## Refund rules

- Refundable only when the payment succeeded (not Created, Pending, Failed), is not in an open dispute, the method supports refunds and is inside its refund window, and a refundable remainder exists.
- Refunds go back to the original instrument (same card, wallet, or account).
- Funds come from the same settlement currency: unsettled funds or available balance. The refund succeeds only when the total covers it and at least one of the two sources alone covers it; otherwise it fails and you retry later or top up.
- Multiple partial refunds are allowed up to the captured amount; over-refund fails.
- With Automatic Currency Conversion, base the refund on the shopper's transaction currency and amount (retrieve the intent first).
- Windows in Malaysia: FPX and DuitNow 180 days, Boost 180 days, TNG 1 year, GrabPay, ShopeePay and SPayLater 90 days, Atome 60 days, 7-Eleven not refundable.

## Refund lifecycle

| Status | Webhook | Meaning |
| --- | --- | --- |
| `RECEIVED` | `refund.received` | Created; amount valid; payment not in dispute |
| `ACCEPTED` | `refund.accepted` | Accepted by the method provider; in most cases effectively succeeded |
| `SETTLED` | `refund.settled` | In the provider's reconciliation; appears in settlement reports and wallet |
| `FAILED` | `refund.failed` | Rejected (insufficient refundable funds, window passed, under dispute, provider reason) |

A refund can stay accepted but not settled for a while. For cards, share the ARN (`acquirer_reference_number`) with the shopper so their bank can trace it.

```bash
curl -X POST https://api.sandbox.airwallex.com/api/v1/pa/refunds/create \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' -H 'Content-Type: application/json' \
  -d '{ "request_id": "<UUID>", "payment_intent_id": "int_...", "amount": 10.00, "reason": "customer requested" }'
```

Store the refund `id` and your own refund reference with a unique `request_id` so a retried refund job cannot double refund.

## Cancel vs refund

- Cancel (`POST /api/v1/pa/payment_intents/{id}/cancel`, body `request_id`, optional `cancellation_reason`): for intents that were not captured. The cancellations guide describes it for releasing an uncaptured authorization (`REQUIRES_CAPTURE`) on methods that need capture; the full amount is cancelled and a webhook follows. The issuer may take days to drop the hold.
- Refund: after capture (`SUCCEEDED`).

## Disputes

- Events: `payment_dispute.requires_response`, `.challenged`, `.accepted`, `.expired`, `.pending_closure`, `.pending_decision`, `.won`, `.lost`, `.reversed`.
- Stages: `RFI`, `PRE_CHARGEBACK`, `CHARGEBACK`, `PRE_ARBITRATION`, `ARBITRATION`.
- Statuses: `REQUIRES_RESPONSE`, `CHALLENGED`, `ACCEPTED`, `REVERSED`, `WON`, `LOST`, `PENDING_CLOSURE`, `EXPIRED`, `PENDING_DECISION`.
- Respond with accept or challenge (with evidence). In Malaysia, chargebacks apply to cards, TNG, ShopeePay and SPayLater; FPX, DuitNow, Boost, GrabPay, Atome have none.
- Simulate disputes in sandbox: see `airwallex-testing-errors.md`.

## Endpoint field tables
### Create a Refund

`POST /api/v1/pa/refunds/create`

Create a Refund against a Payment Intent/Payment Attempt.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. Maximum length is 64. |
| `amount` | number | no | Amount of the Refund. If not specified, it will be same as the remaining amount that has been captured but not yet refunded. |
| `metadata` | object | no | A set of key-value pairs that you can attach to the Refund. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `payment_attempt_id` | string | no | Payment Attempt unique identifier against which this Refund is created. Either this or the `payment_intent_id` must be provided. |
| `payment_intent_id` | string | no | Payment Intent unique identifier against which this Refund is created. Either this or the `payment_attempt_id` must be provided. |
| `reason` | string | no | Reason of the Refund. Maximum length is 128. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `acquirer_reference_number` | string | no | A unique number that tags a credit or debit card transaction when it goes from the merchant's bank through to the cardholder's bank. |
| `amount` | number | no | Amount to be refunded. |
| `connected_account_id` | string | no | Account identifier of the connected account. |
| `created_at` | string | no | Time at which the Refund was created. |
| `currency` | string | no | Currency of the Refund. |
| `failure_details` | object | no | Failure details. Only applicable when status is FAILED. |
| `id` | string | no | Unique identifier for the Refund. |
| `metadata` | object | no | A set of key-value pairs that you can attach to the Refund. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `payment_attempt_id` | string | no | Unique identifier of Payment Attempt on which the Refund is created. |
| `payment_intent_id` | string | no | Unique identifier of Payment Intent on which the Refund is created. |
| `reason` | string | no | Reason of the Refund. |
| `request_id` | string | no | Unique request identifier specified by the merchant in the last operation. |
| `status` | string | no | Status of the Refund. One of `RECEIVED`, `ACCEPTED`, `SETTLED` or `FAILED` - **RECEIVED**: The Refund is created, which means the requested refund amount is valid and the payment is not in dispute. - **ACCEPTED**: The Refund has been acc... Values: `RECEIVED`, `ACCEPTED`, `SETTLED`, `FAILED` |
| `updated_at` | string | no | Last time at which the Refund was updated. |

### Retrieve a Refund

`GET /api/v1/pa/refunds/{id}`

Retrieve a Refund by unique identifier.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Refund unique identifier. |

### List all Refunds

`GET /api/v1/pa/refunds`

List all Refunds.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `connected_account_id` | string | no | Optional Airwallex-assigned identifier of the connected account to filter by. Only applicable to platforms. |
| `currency` | string | no | Refund currency. |
| `from_created_at` | string | no | The start time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `page_num` | integer | no | Page number starting from 0. |
| `page_size` | integer | no | Number of Refunds to be listed per page. Default value is 10. |
| `payment_attempt_id` | string | no | Payment Attempt unique identifier of the Refunds. |
| `payment_intent_id` | string | no | Payment Intent unique identifier of the Refunds. |
| `status` | string | no | Refund status. |
| `to_created_at` | string | no | The end time of `created_at` in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |

### Retrieve a Payment Dispute

`GET /api/v1/pa/payment_disputes/{id}`

Retrieve Payment Dispute information using Payment Dispute unique identifier.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Dispute unique identifier. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `accept_details` | array[object] | no | Further details on why the client is accepting the Payment Dispute event. |
| `acquirer_reference_number` | string | no | The acquirer reference number of original payment. |
| `ai_dispute_automation` | object | no | Information about AI dispute automation, including recommendation and current status. Applicable only when the stage is `CHARGEBACK`. |
| `amount` | number | no | Payment Dispute amount. |
| `card_brand` | string | no | The card brand of original payment, applicable when payment_method_type is `CARD`. Possible values: `visa`, `mastercard`, `maestro`, `union pay`, `american express`, `jcb`, `diners club international` and `discover`. Values: `visa`, `mastercard`, `maestro`, `union pay`, `american express`, `jcb`, `diners club international`, `discover` |
| `challenge_details` | array[object] | no | The challenge data submitted at each stage. |
| `connected_account_id` | string | no | Account identifier of the connected account. |
| `created_at` | string | no | Time at which this Payment Dispute was created. |
| `currency` | string | no | Payment Dispute currency. |
| `customer_id` | string | no | The customer unique identifier of original payment. |
| `customer_name` | string | no | The customer name of original payment. |
| `due_at` | string | no | Payment Dispute due date. |
| `id` | string | no | Payment Dispute unique identifier. |
| `issuer_comment` | string | no | The issuer’s comment on Payment Dispute. |
| `issuer_documents` | array[string] | no | The issuer’s documents on Payment Dispute. |
| `merchant_order_id` | string | no | The order unique identifier of original payment. |
| `metadata` | object | no | A set of key-value pairs attached to the dispute by the merchant. |
| `mode` | string | no | Payment Dispute mode. Possible values: `ALLOCATION`, `COLLABORATION`, applicable when the stage is `CHARGEBACK`, `PRE_ARBITRATION` and `ARBITRATION`. Values: `ALLOCATION`, `COLLABORATION`, `CHARGEBACK`, `PRE_ARBITRATION`, `ARBITRATION` |
| `payment_attempt_id` | string | no | Payment Attempt unique identifier. |
| `payment_intent_id` | string | no | Payment Intent unique identifier. |
| `payment_method_type` | string | no | The payment method type of original payment. |
| `reason` | object | no | Payment Dispute reason. |
| `refunds` | array[object] | no | The Refunds of original payment. |
| `stage` | string | no | Payment Dispute stage. Possible values: `RFI`, `PRE_CHARGEBACK`, `CHARGEBACK`, `PRE_ARBITRATION`, `ARBITRATION`. Values: `RFI`, `PRE_CHARGEBACK`, `CHARGEBACK`, `PRE_ARBITRATION`, `ARBITRATION` |
| `status` | string | no | Payment Dispute status. Possible values: - **REQUIRES_RESPONSE**: In this status, you can decide whether to accept or challenge the Payment Dispute. - Applicable when receive notification from card schemes that Payment Dispute has entere... Values: `REQUIRES_RESPONSE`, `CHALLENGED`, `ACCEPTED`, `REVERSED`, `WON`, `LOST`, `PENDING_CLOSURE`, `EXPIRED`, `PENDING_DECISION` |
| `transaction_type` | string | no | The transaction type of the original transaction. Possible values: `PAYMENT`, `REFUND`. Values: `PAYMENT`, `REFUND` |
| `updated_at` | string | no | Last time at which this Payment Dispute was updated or operated on. |

### List all Payment Disputes

`GET /api/v1/pa/payment_disputes`

List all Payment Disputes using stage, status, reason code, due date, created time or updated time.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `connected_account_id` | string | no | Optional Airwallex-assigned identifier of the connected account to filter by. Only applicable to platforms. |
| `customer_id` | string | no | The customer unique identifier of original payment. |
| `customer_name` | string | no | The customer name of original payment. |
| `from_created_at` | string | no | The start time of created_at in ISO8601 format. |
| `from_due_at` | string | no | The start time of due_at in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `from_updated_at` | string | no | The start time of updated_at in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `merchant_order_id` | string | no | The order unique identifier of original payment. |
| `page` | string | no | `0` or the BASE 64 encoded resource unique identifier of Payment Dispute and navigation direction to indicate the page should be returned. `0` is used by default for initial page. |
| `payment_intent_id` | string | no | Payment Dispute status. |
| `payment_method_type` | string | no | The payment method type of original payment. |
| `reason_code` | string | no | Payment Dispute reason code. |
| `size` | integer | no | Number of Payment Disputes to be listed per page. Default value is 10. |
| `stage` | string | no | Payment Dispute stage. |
| `status` | string | no | Payment Dispute status. |
| `to_created_at` | string | no | The end time of created_at in ISO8601 format. |
| `to_due_at` | string | no | The end time of due_at in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `to_updated_at` | string | no | The end time of updated_at in ISO8601 format. The timestamp must include an explicit timezone (e.g. `Z` or `-04:00`). |
| `transaction_type` | string | no | The transaction type of the original transaction. Possible values: `PAYMENT`, `REFUND`. Values: `PAYMENT`, `REFUND` |

### Update a Payment Dispute

`POST /api/v1/pa/payment_disputes/{id}/update`

Update a Payment Dispute by ID. Currently only the `metadata` field can be updated.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Dispute ID. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `metadata` | object | yes | A set of key-value pairs attached to the dispute. Maximum 50 key-value pairs. |

### Accept a Payment Dispute

`POST /api/v1/pa/payment_disputes/{id}/accept`

Accept a Payment Dispute. When a Payment Dispute is under status `REQUIRES_RESPONSE`, you can accept it to avoid further escalation and fees. Accepting the dispute during `RFI` stage will trigger a full refund. The refund amount and reason can be customized by populating the `refund` field.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Dispute unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. |
| `accepted_by` | string | no | User unique identifier of person/system accepts Payment Dispute. |
| `description` | string | no | The accept description. |
| `reason` | string | no | The accept reason, `OTHERS` is used by default. One of - `AGREEMENT_REACHED_WITH_CUSTOMER` - `CUSTOMER_RELATIONSHIP_MAINTENANCE` - `LOW_VALUE_TRANSACTION` - `VALID_CUSTOMER_DISPUTE` - `COLLABORATION_ACCEPTED_MANUAL` - `OTHERS`. Values: `AGREEMENT_REACHED_WITH_CUSTOMER`, `CUSTOMER_RELATIONSHIP_MAINTENANCE`, `LOW_VALUE_TRANSACTION`, `VALID_CUSTOMER_DISPUTE`, `COLLABORATION_ACCEPTED_MANUAL`, `OTHERS` |
| `refund` | object | no | The Refund to be created. It is applicable when accepting a dispute at the `RFI` stage. |

### Challenge a Payment Dispute

`POST /api/v1/pa/payment_disputes/{id}/challenge`

Challenge a Payment Dispute. When you receive a Payment Dispute in Chargeback or RFI stage with `REQUIRES_RESPONSE` status then you can further challenge the Payment Dispute. Challenge response should include at least one of the recommended evidence document types attached, please use File Service to upload files and refer to it under `supporting_documents`. There are different suggested information sections in different stages, please check if the section is applicable before submission.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Dispute unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `request_id` | string | yes | Unique request identifier specified by the merchant. |
| `challenge_method` | string | no | The method used to challenge the dispute. Possible values: - **AI_DISPUTE_AUTOMATION**: Challenge using AI Dispute Automation - Applicable when Payment Dispute stage is `CHARGEBACK` and AI Dispute Automation status is `AVAILABLE` - **STA... Values: `AI_DISPUTE_AUTOMATION`, `STANDARD` |
| `challenged_by` | string | no | User unique identifier of person/system challenges Payment Dispute. |
| `customer_info` | object | no | Customer information. Applicable when Payment Dispute stage is `CHARGEBACK`. |
| `delivery_info` | object | no | The delivery information of goods or services. |
| `description` | string | no | The additional description of the Payment Dispute. Deprecated. |
| `duplicate_charge_info` | object | no | The defense information when a Payment Dispute is raised due to a duplicate charge. |
| `evidence` | object | no | Evidence required when challenge_method is set to `AI_DISPUTE_AUTOMATION`. |
| `order_info` | object | no | The order information. Applicable when Payment Dispute stage is `RFI`. |
| `product_description` | string | no | The description of the product. Applicable when the Payment Dispute stage is `CHARGEBACK`. |
| `product_type` | string | no | The type of the product. Possible values: `PHYSICAL_GOODS`, `DIGITAL_PRODUCT_OR_SERVICE`, `OFFLINE_SERVICE`, `TRAVEL`, `RESERVE_OR_BOOKING`, `OTHERS`. Values: `PHYSICAL_GOODS`, `DIGITAL_PRODUCT_OR_SERVICE`, `OFFLINE_SERVICE`, `TRAVEL`, `RESERVE_OR_BOOKING`, `OTHERS` |
| `reason` | string | no | The reason why the merchant challenges the dispute. Possible values: - **CUSTOMER_WITHDRAWN**: The customer withdrew the dispute - **CUSTOMER_REFUNDED**: The customer has already been refunded - **PRODUCT_RECEIVED**: The customer has alr... Values: `CUSTOMER_WITHDRAWN`, `CUSTOMER_REFUNDED`, `PRODUCT_RECEIVED`, `PURCHASE_HISTORY`, `NOT_ENTITLED`, `SEPARATE_PRODUCT`, `AUTHENTIC_PRODUCT`, `OTHER_REASONS` |
| `refund_refusal_reason` | string | no | Explanation of why refund is refused. Applicable when Payment Dispute stage is `CHARGEBACK`. |
| `seller_info` | object | no | Seller information. Applicable when Payment Dispute stage is `RFI`. |
| `supporting_documents` | object | no | The file unique identifiers of support documents. |

### List all Related Payment Intents

`GET /api/v1/pa/payment_disputes/{id}/related_payment_intents`

Get related payment intents for a Payment Dispute. Only applicable when the dispute reason type is `FRAUDULENT` or `DUPLICATE_CHARGE`.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Payment Dispute unique identifier. ### Query Parameters |
| `page` | string | no | A bookmark for use in pagination to retrieve either the next page or the previous page of results. You can fetch the value for this identifier from the response of the previous API call. |
| `page_size` | integer | no | Number of related payment intents to be listed per page. Maximum is 100. |

