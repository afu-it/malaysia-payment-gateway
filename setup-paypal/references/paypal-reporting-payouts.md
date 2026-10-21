# PayPal Reporting, Payouts, Disputes, And Other APIs

Sources: Transaction Search v1, Payouts v1, Disputes v1, Invoicing v2 and Shipment Tracking v1 schemas; `/payouts/overview`, `/payouts/supported-features`, `/payouts/fees`, `/payouts/faqs`, `/payouts/currency-conversion`, `/api/payouts`, `/disputes/overview`, `/disputes/disputes-lifecycle`, `/reports/overview`. Checked 2026-10-21.

## Transaction Search (reconciliation)

| Method | Path | Notes |
| --- | --- | --- |
| `GET` | `/v1/reporting/transactions` | `start_date` and `end_date` required (RFC 3339, seconds required), max 31-day range, history up to 3 years; new transactions can take up to 3 hours to appear. Filters: `transaction_id` (17 chars; order IDs 19), `transaction_type` (event code), `transaction_status` (`D` denied, `P` pending, `S` success, `V` reversed/refunded), `transaction_amount` (`<lo> TO <hi>`), `transaction_currency`, `payment_instrument_type` (`CREDITCARD`, `DEBITCARD`), `store_id`, `terminal_id`, `fields` (`transaction_info` or `all` and others), `balance_affecting_records_only` (`Y` default, `N`), `page_size`, `page`. Header `PayPal-Enforce-ISO8601-Format: true` for ISO dates. |
| `GET` | `/v1/reporting/balances` | All balances. |
| `GET` | `/v1/reporting/get-balance-net-summary` | Balance and net activity summary. |
| `GET` | `/v1/reporting/get-daily-summary` | Daily summary. |

Useful `transaction_info` fields: `transaction_id`, `paypal_reference_id`, `paypal_reference_id_type` (`ODR`, `TXN`, `SUB`, `PAP`), `transaction_event_code` (T-codes, see `/reports/reference/t-codes`), `transaction_amount`, `fee_amount`, `transaction_status`, `invoice_id`, `custom_field`, `protection_eligibility` (`01` eligible, `02` not, `03` partial), `ending_balance`, `available_balance`.

Dashboard and SFTP reports (Settlement, Transaction Detail, Balance, Fees, Disbursement reconciliation and others) are described at `/reports/overview`; access is configured in the PayPal business account.

## Settlement And Withdrawal

The developer docs do not describe a scheduled settlement to a Malaysian bank: funds land in the PayPal balance and the merchant withdraws. Withdrawal fees and conversion spreads are on the official fee page `https://www.paypal.com/my/business/paypal-business-fees` (link only; numbers change). Use `seller_receivable_breakdown` on each capture (`gross_amount`, `paypal_fee`, `net_amount`, `receivable_amount`, `exchange_rate`) for per-payment accounting.

## Payouts (sending money)

Endpoints: `POST /v1/payments/payouts`, `GET /v1/payments/payouts/{id}`, `GET /v1/payments/payouts-item/{payout_item_id}`, `POST /v1/payments/payouts-item/{payout_item_id}/cancel` (unclaimed only). `PayPal-Request-Id` stored 30 days. A `sender_batch_id` reused within 30 days is rejected as duplicate. Up to 15000 items per batch. Item `transaction_status`: `SUCCESS`, `FAILED`, `PENDING`, `UNCLAIMED`, `RETURNED`, `ONHOLD`, `BLOCKED`, `REFUNDED`, `REVERSED`. Payouts must be enabled on the account (see `/payouts/use-payouts/set-up`).

Malaysia rules (documented):

- Malaysia is marked "Send, receive, and withdraw in local currency" with the footnote "This country has currency restrictions".
- "Malaysian users can send payouts in MYR to other Malaysian users and send payouts in other currencies to all users. Malaysian users cannot send payouts that require exchange to or from MYR, or send MYR balances to non-Malaysian users."
- FAQ: currency restrictions do not allow conversion of Malaysian Ringgit.
- Per-item limit for MYR: 300,000 to registered recipients, 100,000 to unregistered recipients.
- Fees are charged to the sender (typically 2% variable with caps; see the merchant fee page).

### Payouts Field Tables

#### create_payout_request (POST /v1/payments/payouts)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `sender_batch_header` | object (sender_batch_header) | yes | The sender-provided payout header for a payout request. |
| `sender_batch_header.sender_batch_id` | string | no | A sender-specified ID number. Tracks the payout in an accounting system.Note: PayPal does not process duplicate payouts. If you specify a sender_batch_id that was used ... [maxLength=256] |
| `sender_batch_header.recipient_type` | string | no | The ID type that identifies the recipient of the payment. For example, EMAIL. [maxLength=13] |
| `sender_batch_header.email_subject` | string | no | The subject line for the email that PayPal sends when payment for a payout item completes. The subject line is the same for all recipients. Value is an alphanumeric ... [maxLength=255] |
| `sender_batch_header.email_message` | string | no | The email message that PayPal sends when the payout item completes. The message is the same for all recipients. [maxLength=1000] |
| `sender_batch_header.note` | string | no | The payouts and item-level notes are concatenated in the email. The maximum combined length of the notes is 1000 characters. [maxLength=1000] |
| `items` | array<payout_item_request> (payout_item_request_list) | yes | An array of individual payout items. [maxItems=15000] |
| `items.recipient_type` | string | no | The recipient type. Value is:EMAIL. The unencrypted email. Value is a string of up to 127 single-byte characters.PHONE. The unencrypted phone number.Note: The PayPal ... [maxLength=13] |
| `items.amount` | object (currency) | yes | The currency and amount to pay the receiver. |
| `items.note` | string | no | The sender-specified note for notifications. Supports up to 4000 ASCII characters and 1000 non-ASCII characters. [maxLength=4000] |
| `items.receiver` | string | yes | The receiver of the payment. Corresponds to the `recipient_type` value in the request. Max value of up to 127 single-byte characters. [maxLength=127] |
| `items.sender_item_id` | string | no | The sender-specified ID number. Tracks the payout in an accounting system. [maxLength=63] |
| `items.recipient_wallet` | string | no | The recipient wallet. [maxLength=36; default=PAYPAL] |
| `items.alternate_notification_method` | object (alternate_notification_method) | no | Captures additional notification modes to reach out to the receiver regarding this payment. |
| `items.notification_language` | string (language) | no | The language in which to show the payout recipient's email message. Used only when the recipient does not have a PayPal account. If you omit the language or provide ... [minLength=2; maxLength=10] |
| `items.application_context` | object (application_context) | no | Metadata for accepting additional information from merchants to Venmo. |
| `items.purpose` | string (purpose_enum) | no | The purpose of the transaction. [enum: AWARDS, PRIZES, DONATIONS, GOODS, SERVICES, REBATES, CASHBACK, DISCOUNTS, NON_GOODS_OR_SERVICES; maxLength=40; pattern=^[A-Z0-9_]+$] |

#### payout_item (GET /v1/payments/payouts-item/{id})

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `payout_item_id` | string | yes | The ID for the payout item. Visible when you show details for a payout. [maxLength=30] |
| `transaction_id` | string | no | The PayPal-generated ID for the transaction. [maxLength=30] |
| `activity_id` | string | no | The unique PayPal-generated common ID that links the sender- and receiver-side transactions. Used for tracking. [maxLength=30] |
| `transaction_status` | string (transaction_enum) | no | The item transaction status.Note: For POST/v1/payments/payouts-item/{payout_item_id}/cancel, the only possible transaction_status value is RETURNED. [enum: SUCCESS, FAILED, PENDING, UNCLAIMED, RETURNED, ONHOLD, BLOCKED, REFUNDED, REVERSED; maxLength=36; pattern=^[0-9A-Z_]+$] |
| `payout_item_fee` | object (currency) | no | The estimate for the payout fee. Initially, the fee is `0`. The fee is populated after the item moves to the `PENDING` state |
| `payout_batch_id` | string | yes | The PayPal-generated ID for the payout batch. [maxLength=30] |
| `sender_batch_id` | string | no | A sender-specified ID. Tracks the payout in an accounting system. Should be unique within 30 days. [maxLength=256] |
| `payout_item` | object (payout_item_detail) | yes | The sender-provided information for the payout item. |
| `currency_conversion` | object (payout_currency_conversion) | no | The currency conversion applicable for this payout item. |
| `time_processed` | string | no | The date and time when this item was last processed, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [maxLength=100] |
| `errors` | object (error) | no | The error details. |
| `links` | array<link_description> (payout_item_definitions-link_description_list) | no | An array of request-related [HATEOAS links](/api/rest/responses/#hateoas-links). [maxItems=15000; readOnly] |

## Disputes

Endpoints are in the catalogue (`/v1/customer/disputes...`): list, get, patch, provide-evidence, appeal, accept-claim, adjudicate (sandbox), require-evidence (sandbox), escalate, send-message, make-offer, accept-offer, deny-offer, acknowledge-return-item, provide-supporting-info. Webhooks: `CUSTOMER.DISPUTE.CREATED`, `CUSTOMER.DISPUTE.UPDATED`, `CUSTOMER.DISPUTE.RESOLVED`. A `PAYMENT.CAPTURE.REVERSED` or capture `status_details.reason` of `CHARGEBACK` / `BUYER_COMPLAINT` should freeze fulfilment or entitlements in your app. Dispute reasons, evidence types and file limits: `/disputes/reasons-evidence`, `/disputes/supported-file-sizes-types`; dispute test values: `/api/disputes/test-values`.

## Other Current APIs (catalogue only)

- Invoicing v2 (`/v2/invoicing/...`): draft, send, remind, cancel, record payment/refund, QR code, templates, recurring invoices, auto reminders. Webhooks `INVOICING.INVOICE.*`.
- Shipment Tracking v1 (`/v1/shipping/trackers...`) and Orders `.../track`.
- Payment Links and Buttons v1 (`/v1/checkout/payment-resources`).
- Payment Method Tokens v3 (`/v3/vault/payment-tokens`, `/v3/vault/setup-tokens`): saving cards needs Expanded Checkout, not listed for Malaysia; PayPal-wallet vault eligibility for Malaysia is not documented.
- Payment Experience web profiles, Identity userinfo, Pricing exchange-rate quotes, Partner Referrals, Usage Based Billing: listed in the catalogue in `paypal-api.md`; not part of a standard Malaysian merchant checkout.
