# Airwallex Settlement and Reporting

Checked 2026-10-21.

## Source URLs

- Settlements: `https://www.airwallex.com/docs/payments/about-airwallex-payments/settle-into-your-airwallex-balance`
- Reserve plan: `https://www.airwallex.com/docs/payments/about-airwallex-payments/reserve-plan`
- Reporting: `https://www.airwallex.com/docs/payments/payment-operations/reporting`
- Settlement report: `https://www.airwallex.com/docs/payments/payment-operations/reporting/settlement-report`
- Transaction and fee types: `https://www.airwallex.com/docs/payments/payment-operations/reporting/transaction-and-fee-types`
- Settlements API: `https://www.airwallex.com/docs/api/finance/settlements`
- Settlement records API: `https://www.airwallex.com/docs/api/payments/settlement_records`
- Financial transactions API: `https://www.airwallex.com/docs/api/finance/financial_transactions`
- Financial reports API: `https://www.airwallex.com/docs/api/finance/financial_reports`

## How funds settle

- Captured payments settle into your multi-currency Airwallex Wallet (not straight to a bank). Payout to a bank is a separate transfer (Payouts product, out of scope).
- Set principal settlement currencies (for example MYR) and a fallback currency at onboarding; enable them in **Wallet > Balances**. Currencies without like-for-like settlement are converted (FX fee) to the fallback.
- T is the capture date. Each method has its own T+N (see `airwallex-malaysia-methods.md`). A reserve plan can hold funds longer; the later of method schedule and reserve applies. Check **Payments > Settings > Reserve plan**.
- Daily batch at the configured settlement time (commonly 23:00 in the Airwallex entity's time zone); payments at or after that time count as the next day. Fast settlement configs run intraday batches about every 20 minutes.
- Net settlement (default) deducts fees before crediting; gross settlement charges fees separately.
- Webhooks `payment_attempt.settled` (Airwallex received funds from acquirer) and `payment_attempt.paid` (funds paid to your wallet) track this.
- Reports: settlement report, balance activity report, fee report, payment activity export. Use the settlement report for batch totals and the balance activity report to trace to the wallet.
- Sandbox default settlement: `BACKTOBACK`, 0 day delay, calendar days.

## Endpoint field tables
### List all Settlements

`GET /api/v1/pa/financial/settlements`

Get list of settlements that contributed to the Airwallex account balance from a specified settled date, with optional filters on end settled date, currency and status.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `currency` | string | yes | Currency of the settlement. |
| `from_settled_at` | string | yes | The start date of `settled_at` in ISO8601 format (inclusive). |
| `page_num` | integer | no | Page number, starts from 0. |
| `page_size` | integer | no | Number of results per page, default is 100, max is 1000. |
| `status` | string | yes | Status of the settlement, one of: `PENDING`, `SETTLED`. Values: `PENDING`, `SETTLED` |
| `to_settled_at` | string | yes | The end date of `settled_at` in ISO8601 format (inclusive). |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `has_more` | boolean | no | A flag which identifies whether there are more results. |
| `items` | array[object] | no | List items. |
| `items.amount` | number | no | Settlement amount Airwallex pays to the merchant. |
| `items.created_at` | string | no | The time this settlement was created at. |
| `items.currency` | string | no | Currency of the settlement. |
| `items.estimated_settled_at` | string | no | The time when the settlement will be settled at. |
| `items.fee` | number | no | Fee of the settlement. |
| `items.settled_at` | string | no | The time this settlement was settled at. |
| `items.settlement_id` | string | no | ID of the settlement. |
| `items.status` | string | no | One of `PENDING`, `SETTLED`. Values: `PENDING`, `SETTLED` |

### Retrieve a Settlement

`GET /api/v1/pa/financial/settlements/{id}`

Get a specific settlement detail by specifying the Airwallex settlement unique identifier.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Batch unique identifier of transaction. |

### Retrieve a Settlement Report

`GET /api/v1/pa/financial/settlements/{id}/report`

Get a single batch settlement report by specifying the Airwallex settlement batch unique identifier.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Batch unique identifier of transaction. ### Query Parameters |
| `file_format` | string | no | File format of the requested report. Supported formats are `csv` and `excel`. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `created_at` | string | no | The date and time this settlement report was created. |
| `report_status` | string | no | Status of the report. Possible values: - `READY`: `READY` once available for download - `IN_PROGRESS`: `IN_PROGRESS` while still being generated Values: `READY`, `IN_PROGRESS` |
| `report_url` | string or null | no | The url of the report. Available when `report_status` is `READY`, otherwise `null`. |
| `settlement_id` | string | no | ID of the settlement. |

### List all Settlement Records

`GET /api/v1/pa/settlement_records`

Get Settlement Records by specifying the settlement (batch) unique identifier or payment intent unique identifier.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `page` | string | no | Value from page_after. |
| `page_size` | integer | no | The number of items requested by page (default and max is 50). |
| `payment_intent_id` | string | no | The payment intent unique identifier of the settlement records. |
| `settlement_id` | string | no | The settlement(batch) unique identifier of the settlement records. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `has_more` | boolean | no | A flag which identifies whether there are more results. |
| `items` | array[object] | no | List items. |
| `items.account_id` | string | no | Unique identifier of your account in Airwallex. |
| `items.acquirer_reference_number` | string | no | A unique number assigned to a card transaction as it moves through the payment flow. It can be used by banks to help trace a transaction if it appears to be missing. |
| `items.card_category` | string | no | Card category, such as consumer, commercial, if applicable. |
| `items.card_funding` | string | no | Card funding type, such as credit, debit, prepaid, if applicable. |
| `items.card_issuing_or_shopper_country` | string | no | Country the shopper's payment method was issued in, if applicable. |
| `items.card_transaction_region` | string | no | Identifies what region the card transaction has been classified as, such as domestic, regional or international. |
| `items.connected_account_id` | string | no | Unique identifier of the connected account related to this transaction under the Payments for Platforms model, if applicable. |
| `items.created_at` | string | no | Timestamp of when the financial transaction was created. |
| `items.customer_email` | string | no | Email of the customer, if applicable. |
| `items.customer_id` | string | no | Unique identifier of the customer in Airwallex, if applicable. |
| `items.customer_name` | string | no | Name of the customer, if applicable. |
| `items.customer_phone` | string | no | Phone number of the customer, if applicable. |
| `items.descriptor` | string | no | Transaction descriptor that is displayed to the shopper. |
| `items.dispute_reason` | string | no | Dispute reason from the payment method provider. For more information, refer to https://www.airwallex.com/docs/payments/disputes/dispute-reason-codes. |
| `items.dispute_reason_code` | string | no | Dispute reason code from the payment method provider. For more information, refer to https://www.airwallex.com/docs/payments/disputes/dispute-reason-codes. |
| `items.dispute_status` | string | no | The latest status of the dispute. For more information on dispute statuses, refer to https://www.airwallex.com/docs/payments/manage-payments/disputes/handle-disputes-on-the-airwallex-web-app. |
| `items.exchange_rate` | number | no | Exchange rate used for converting the transaction amount into the settlement amount. |
| `items.fee_details_list` | array[object] | no | List of related fee records. |
| `items.fees` | number | no | Amount of the total fees paid for the transaction in the settlement currency, exclusive of taxes. |
| `items.gross_amount` | number | no | Amount of the transaction, converted into the settlement currency. |
| `items.issuing_or_originating_bank` | string | no | Name of financial institution that the shopper's payment method was issued from, if applicable. |
| `items.merchant_customer_id` | string | no | Unique identifier of the customer provided by you, if applicable. Maximum length is 64. |
| `items.metadata` | object | no | A set of key-value pairs that you can attach to the Settlement Record. You can specify up to 50 keys with key names up to 50 characters long and values up to 500 characters long. |
| `items.net_amount` | number | no | Net amount of the transaction in the settlement currency, after subtracting the fees. |
| `items.order_id` | string | no | Unique identifier from your order system associated with a payment intent. |
| `items.payment_attempt_ids` | array[string] | no | Unique identifiers for payment attempt associated with this payment. |
| `items.payment_created_time` | string | no | Time at which the payment intent was created. |
| `items.payment_intent_id` | string | no | Unique identifier for the payment intent. |
| `items.payment_link_reference` | string | no | Reference for the payment link input by your customer, if applicable. |
| `items.payment_method` | string | no | The payment method used by the shopper for this transaction. |
| `items.payment_method_type` | string | no | Helps you identify the payment method, if applicable, for example: Card brand when Apple Pay or Google Pay are used: Visa, Mastercard, etc. Klarna: Pay Now, Pay Later, etc. |
| `items.request_id` | string | no | Unique identifier from your system associated to your request to Airwallex. |
| `items.settled_at` | string | no | Timestamp of when the financial transaction was settled into your wallet. |
| `items.settlement_batch_id` | string | no | Unique identifier associated with the batch settlement. |
| `items.settlement_currency` | string | no | Three-letter ISO code for the settlement currency of the transaction. |
| `items.shipping_address` | string | no | First line of the shipping address associated with this transaction, if applicable. |
| `items.shipping_city` | string | no | City of the shipping address associated with this transaction, if applicable. |
| `items.shipping_country` | string | no | Country of the shipping address associated with this transaction, if applicable. |
| `items.shipping_name` | string | no | Name on the shipping address associated with this transaction, if applicable. |
| `items.shipping_postal_code` | string | no | Postal code of the shipping address associated with this transaction, if applicable. |
| `items.shipping_state` | string | no | State of the shipping address associated with this transaction, if applicable. |
| `items.source_entity` | string | no | Source object associated with the transaction. |
| `items.source_id` | string | no | Unique identifier for the payment attempt, payment consent, refund, or dispute. |
| `items.subscription_id` | string | no | Unique identifier of the subscription related to this transaction, if applicable. |
| `items.taxes_on_fees` | number | no | Amount of the total taxes on fees paid for the transaction in the settlement currency. |
| `items.transaction_amount` | number | no | Amount of the transaction. |
| `items.transaction_currency` | string | no | Three-letter ISO code for the original currency sent to Airwallex for the transaction. |
| `items.transaction_id` | string | no | Unique identifier associated with the financial transaction. |
| `items.transaction_type` | string | no | Categorisation of financial transactions. Possible values: `Payment`, `Payment Reserve Hold`, `Payment Reserve Release`, `Split`, `Split Reversal`, `Dispute`, `Dispute Reversal`, `Dispute Lost`, `Dispute Won`, `Pre Chargeback Accepted`, ... Values: `Payment`, `Payment Reserve Hold`, `Payment Reserve Release`, `Split`, `Split Reversal`, `Dispute`, `Dispute Reversal`, `Dispute Lost`, `Dispute Won`, `Pre Chargeback Accepted`, `Pre Chargeback Withdrawn`, `Refund Pre Chargeback Accepted`, `Refund Chargeback Received`, `Refund Chargeback Withdrawn` ... |
| `page_after` | string | no | Page after. |

### List all Financial Transactions

`GET /api/v1/financial_transactions`

Get a list of financial transactions that contributed to the Airwallex account balance from a specified creation date/time, with optional filters on end creation date/time, batch_id and status.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `batch_id` | string | no | Batch unique identifier of the financial transaction. |
| `currency` | string | no | The currency (3-letter ISO-4217 code) of the financial transaction. |
| `from_created_at` | string | no | The start time of `created_at` in ISO8601 format (inclusive). |
| `page_num` | integer | no | Page number, default is 0, max is 2000. |
| `page_size` | integer | no | Number of results per page, default is 100, max is 1000. |
| `source_id` | string | no | The source unique identifier of the transaction. |
| `status` | string | no | Status of the financial transaction, one of: `PENDING`, `SETTLED`. Values: `PENDING`, `SETTLED` |
| `to_created_at` | string | no | The end time of `created_at` in ISO8601 format (inclusive). |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `has_more` | boolean | no | A flag which identifies whether there are more results. |
| `items` | array[object] | no | List items. |
| `items.amount` | number | no | Gross amount the transaction. |
| `items.batch_id` | string | no | The batch unique identifier of the settlement that the transaction belongs to. |
| `items.client_rate` | number | no | The client rate for the transaction. |
| `items.created_at` | string | no | The time this transaction was created at. |
| `items.currency` | string | no | Currency the transaction should be made in (3-letter ISO-4217 code). |
| `items.currency_pair` | string | no | The currency pair that the `client_rate` is quoted in. |
| `items.description` | string | no | The description of the transaction. |
| `items.estimated_settled_at` | string | no | The time when the transaction will be settled at. |
| `items.fee` | number | no | Fee paid for the transaction. |
| `items.funding_source_id` | string | no | The unique identifier of the funding source from which the transaction originated. |
| `items.id` | string | no | Unique identifier  of the transaction. |
| `items.net` | number | no | Net amount of the transaction. |
| `items.settled_at` | string | no | The time when the transaction was settled at. |
| `items.source_id` | string | no | The source unique identifier of the transaction. |
| `items.source_type` | string | no | The type of the source transaction, One of `PAYOUT`, `CONVERSION`, `DEPOSIT`, `ADJUSTMENT`, `FEE`, `PAYMENT_ATTEMPT`, `REFUND`, `DISPUTE`, `CHARGE`, `TRANSFER`, `YIELD`, `BATCH_PAYOUT`, `CARD_PURCHASE`, `CARD_REFUND`, `PURCHASE`, `REFUND... Values: `PAYOUT`, `CONVERSION`, `DEPOSIT`, `ADJUSTMENT`, `FEE`, `PAYMENT_ATTEMPT`, `REFUND`, `DISPUTE`, `CHARGE`, `TRANSFER`, `YIELD`, `BATCH_PAYOUT`, `CARD_PURCHASE`, `CARD_REFUND` ... |
| `items.status` | string | no | One of `PENDING`, `SETTLED`, `CANCELLED` . Values: `PENDING`, `SETTLED`, `CANCELLED` |
| `items.transaction_type` | string | no | The type of the transaction, One of `DISPUTE`, `DISPUTE_REVERSAL`, `DISPUTE_LOST`, `REFUND`, `REFUND_REVERSAL`,`REFUND_FAILURE`, `PAYMENT_RESERVE_HOLD`,`PAYMENT_RESERVE_RELEASE`,`PAYOUT`,`PAYOUT_FAILURE`, `PAYOUT_REVERSAL`, `CONVERSION_S... Values: `DISPUTE`, `DISPUTE_REVERSAL`, `DISPUTE_LOST`, `REFUND`, `REFUND_REVERSAL`, `REFUND_FAILURE`, `PAYMENT_RESERVE_HOLD`, `PAYMENT_RESERVE_RELEASE`, `PAYOUT`, `PAYOUT_FAILURE`, `PAYOUT_REVERSAL`, `CONVERSION_SELL`, `CONVERSION_BUY`, `CONVERSION_REVERSAL` ... |

### Retrieve a Financial Transaction

`GET /api/v1/financial_transactions/{id}`

Get a specific financial transaction by an Airwallex financial transaction unique identifier.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Unique identifier of financial transaction. |

### Create a Financial Report

`POST /api/v1/finance/financial_reports/create`

Create a balance activity, transaction reconciliation, or settlement report.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `file_format` | string | yes | File format of financial report to create. Possible values: - `CSV`: Supported formats for Balance Activity Reports, Settlement and Transaction Reconciliation Reports - `EXCEL`: Supported formats for Balance Activity Reports, Settlemen... Values: `CSV`, `EXCEL`, `PDF` |
| `from_date` | string | yes | Start date of the report (inclusive). |
| `to_date` | string | yes | End date of the report (inclusive). |
| `type` | string | yes | Type of financial report to create. Possible values: - `ACCOUNT_STATEMENT_REPORT` - `BALANCE_ACTIVITY_REPORT` - `ONLINE_PAYMENTS_TRANSACTION_REPORT` - `SETTLEMENT_REPORT` - `TRANSACTION_RECON_REPORT` Values: `ACCOUNT_STATEMENT_REPORT`, `BALANCE_ACTIVITY_REPORT`, `ONLINE_PAYMENTS_TRANSACTION_REPORT`, `SETTLEMENT_REPORT`, `TRANSACTION_RECON_REPORT` |
| `additional_fields` | array[string] | no | List of additional fields to include as extra columns in the report, identified by field code. Additional fields add business-context columns to each report row to support reconciliation, as in the web app. |
| `currencies` | array[string] | no | Currencies to be included in the report. Required for `ACCOUNT_STATEMENT_REPORT` and supported as an optional filter for `BALANCE_ACTIVITY_REPORT` and `SETTLEMENT_REPORT`. |
| `file_name` | string | no | File name of the report. If left empty, a file name will be automatic generated following `{REPORT_TYPE}_{DATE}`. |
| `report_options` | object | no | Additional options for configuring report generation behavior. |
| `report_version` | string | no | Version of the report (Not applicable to `SETTLEMENT_REPORT`). If left empty, defaults to 1.1.0 for the `BALANCE_ACTIVITY_REPORT` and `TRANSACTION_RECON_REPORT`, and to 1.0.0 for other report types. |
| `settlement_currencies` | array[string] | no | Settlement currencies to be included in the report. The report will include all settlement currencies if left empty. |
| `statuses` | array[string] | no | Transaction statuses to be included in the report. The report will include all statuses if left empty. |
| `time_zone` | string | no | Timezone of the user requesting the report (Not applicable to `SETTLEMENT_REPORT`). Values: `SETTLEMENT_REPORT` |
| `transaction_currencies` | array[string] | no | Transaction currencies to be included in the report. The report will include all transaction currencies if left empty. |
| `transaction_types` | array[string] | no | Transaction types to be included in the report. Will include all transaction types if left empty. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `file_format` | string | no | File format of the report. One of `CSV`, `PDF`, `EXCEL`. Values: `CSV`, `PDF`, `EXCEL` |
| `file_name` | string | no | File name. |
| `id` | string | no | Unique identifier of the financial report. |
| `report_expires_at` | string | no | Contents expiration date. Will be empty string if report is in `PENDING` state or the report does not expire. |
| `report_parameters` | object | no | Parameters used to generate report. |
| `report_version` | string | no | Version of the report. |
| `status` | string | no | Status of the report. Possible values: - `COMPLETED` - `FAILED` - `PENDING` Values: `COMPLETED`, `FAILED`, `PENDING` |
| `type` | string | no | Possible values: - `ACCOUNT_STATEMENT_REPORT` - `AGGREGATED_SETTLEMENT_REPORT` - `BALANCE_ACTIVITY_REPORT` - `ONLINE_PAYMENTS_TRANSACTION_REPORT` - `SETTLEMENT_REPORT` - `TRANSACTION_RECON_REPORT` Values: `ACCOUNT_STATEMENT_REPORT`, `AGGREGATED_SETTLEMENT_REPORT`, `BALANCE_ACTIVITY_REPORT`, `ONLINE_PAYMENTS_TRANSACTION_REPORT`, `SETTLEMENT_REPORT`, `TRANSACTION_RECON_REPORT` |

### Retrieve a Financial Report

`GET /api/v1/finance/financial_reports/{id}`

Get a single generated balance activity, transaction reconciliation, settlement or aggregated settlement report.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Unique identifier of financial report. |

### List all Financial Reports

`GET /api/v1/finance/financial_reports`

Get list of previously generated balance activity, transaction reconciliation, settlement, account statement, and aggregated settlement reports.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `page_num` | integer | no | Page number, starts from 0. |
| `page_size` | integer | no | Number of results per page, default is 100, max is 1000. |

### Retrieve a Contents of a Financial Report

`GET /api/v1/finance/financial_reports/{id}/content`

Download a previously generated balance activity, transaction reconciliation, settlement or aggregated settlement reports.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Unique identifier of financial report. |

