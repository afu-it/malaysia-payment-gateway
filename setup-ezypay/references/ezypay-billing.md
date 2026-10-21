# Ezypay Billing Field Reference

Request and response fields for every payment-related operation, taken from the OpenAPI blocks on `https://developer.ezypay.com/reference/<slug>` and the guides under `/docs`. Checked 2026-10-21. Base `https://api-sandbox.ezypay.com/v2` (production `https://api-global.ezypay.com/v2`). All calls need `Authorization: Bearer <token>` and header `merchant: <MERCHANT_ID>` unless noted.

Amount object (`Amount`): `currency` (string, required, ISO 4217, use `MYR`), `value` (number, required, greater than zero, decimal major units), `type` (optional, `percentage` or `fixed_amount`).
Tax object (`Tax`): `rate` (number, 0 to 99.99 percent; omitted means merchant default).

## Customers

### POST /billing/customers (create)

No duplicate check is done.

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `firstName` | string | yes | max 50 |
| `lastName` | string | yes | max 50 |
| `email` | string | yes | max 255; Ezypay emails go here |
| `gender` | string | no | `male` or `female` |
| `address.address1`, `address.address2` | string | no | max 100 each |
| `address.postalCode` | string | no | max 20 |
| `address.city`, `address.state` | string | no | max 100 |
| `address.countryCode` | string | no | ISO 3166-1 alpha-2, for example `MY` |
| `homePhone` | string | no | max 50 |
| `mobilePhone` | string | no | max 50, numeric, E.164 (for example `+60123456789`) |
| `companyName` | string | no | max 100 |
| `dateOfBirth` | string | no | `YYYY-MM-DD` |
| `referenceCode` | string | no | your customer number, max 30; recommended for reconciliation |
| `metadata` | object | no | key/value |
| `customerNumber` | string | no | existing `EZYxxxxxx` number, used for customer transfer between merchants |

Response: `id` (UUID, store it), `number` (`EZYxxxxxx`), `referenceCode`, `firstName`, `lastName`, `email`, `companyName`, `mobilePhone`, `homePhone`, `gender`, `dateOfBirth`, `createdOn`, `address`, `metadata`.

### PUT /billing/customers/{customerId} (update)

Required: `email`, `firstName`, `lastName`. Optional: `gender`, `homePhone`, `mobilePhone`, `referenceCode`, `companyName`, `dateOfBirth`, `address.*`. Sync every change of email and mobile so Ezypay notifications reach the customer.

### GET /billing/customers (list)

Query: `name` (max 100), `firstName`, `lastName`, `companyName`, `referenceCode`, `customerNumber`, `createdDate`, `sortExpression` (for example `lastname,ASC`), `limit`, `cursor`.

### GET /billing/customers/{customerId} (retrieve)

`customerId` is case-sensitive.

### Customer transfer between merchants

Retrieve the old customer's `number`, create a customer under the new merchant with `customerNumber` set to it, list the old payment methods, link each token with Create Payment Method. Card tokens (Visa, Mastercard) can move across regions; bank, PayTo and GCash cannot. Tokens never move to or from South Korea. Do not transfer GCash or PayTo tokens. The customer must consent.

## Payment methods

Cards are captured only through the Payment Capture Page (PCP), Checkout or Pay Now. See `ezypay-payment-methods.md` for PCP.

### POST /billing/customers/{customerId}/paymentmethods (link token)

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `paymentMethodToken` | string | yes | token from the vault (PCP, HPP, vault APIs) |
| `primary` | boolean | no | first token becomes primary automatically; else only changes when `true`. PromptPay cannot be primary |

No verification is performed while linking. Response is a payment method payload: `paymentMethodToken`, `customerId`, `type`, `bank`, `card` (`first6`, `last4`, `accountHolderName`, `type`, `expiryMonth`, `expiryYear`), `invalidReason`, `lastUsedOn`, `valid`, `primary`.

### Other payment method calls

| Call | Body / query | Notes |
| --- | --- | --- |
| `GET /billing/customers/{customerId}/paymentmethods` | `limit`, `cursor` | returns valid and invalid methods |
| `GET /billing/customers/{customerId}/paymentmethods/{paymentMethodToken}` | none | check `valid` and `customerId` before billing |
| `GET /billing/customers/{customerId}/paymentmethods/primary` | none | current default |
| `PUT .../paymentmethods/{paymentMethodToken}/primary` | none | does not move existing invoices or subscriptions; error `INVALID_PRIMARY_METHOD` for QR |
| `PUT .../paymentmethods/{paymentMethodToken}/new` | `newPaymentMethodToken` (required), `primary` | replaces the old token on all invoices and subscriptions and sets the new one primary |
| `DELETE .../paymentmethods/{paymentMethodToken}` | none | deletes from vault if no other customer uses it; cannot delete the only (primary) method or one with an active subscription |

Ezypay auto rebill stops once a payment method is invalid. Creating an invoice or subscription with an invalid token returns an error and creates nothing. Inactive methods flagged invalid for 13 months are purged; methods unused for 24 months and not linked to active subscriptions or future invoices are removed from the vault.

### Vault token APIs (not Malaysia)

`POST https://vault-sandbox.ezypay.com/v2/vault/paymentmethodtokens/bank` (AU, NZ): `accountHolderName`, `accountNumber`, `bankNumber`, `branchNumber`, `suffixNumber`, `termAndConditionAgreed` (`true`/`false`), `countryCode`, all required. Returns `type`, `bank.*`, `paymentMethodToken`.

`POST https://vault-sandbox.ezypay.com/v2/vault/paymentmethodtokens/qrpayment` (PromptPay, TH): `accountHolderName`, `countryCode`, `qrType` (`PROMPTPAY`), `termAndConditionAgreed` required; `customerId` optional (links directly). Error `payment_method_not_accepted` if not enabled.

## Invoices (on-demand)

### POST /billing/invoices (create)

Charges immediately (status `PROCESSING`) unless `scheduledPaymentDate` is set (status `UNPAID`). If no token is given and the customer has no default, the invoice is created `past_due`.

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `customerId` | string | yes | Ezypay customer UUID |
| `paymentMethodToken` | string | no | defaults to the customer's primary token |
| `memo` | string | no | max 100 |
| `items[]` | array | yes | one entry per line item |
| `items[].description` | string | docs say yes | line description |
| `items[].amount` | Amount | yes | `currency` and `value` required |
| `items[].tax` | Tax | no | |
| `items[].accountingCode` | string | no | max 50, groups items in reports; no `&` |
| `items[].reference` | string | no | extra reference |
| `autoPayment` | string/bool | no | default `true`; `false` creates without charging |
| `scheduledPaymentDate` | string | no | `YYYY-MM-DD`, at least tomorrow, at most 365 days ahead; needs `autoPayment=true` |
| `externalInvoiceId` | string | recommended | your id, unique per merchant; Ezypay's guide calls it the idempotency key |
| `processingModel` | string | no | `cardonfile` for one-off purchases, `recurring` (default) for recurring agreements |

Invoice response (also returned by retrieve, list, retry, refund, cancel and in invoice webhooks):

| Field | Notes |
| --- | --- |
| `id` | invoice UUID |
| `creditNoteId` | set after refund or chargeback |
| `documentNumber` | for example `IN0000000000004192` |
| `date`, `dueDate` | `YYYY-MM-DD` |
| `scheduledPaymentDate` | next scheduled attempt or retry date |
| `status` | see statuses in `ezypay-api.md` |
| `memo` | |
| `items[]` | `id`, `description`, `amount`, `tax.rate`, `type`, `discounted` (Amount), `accountingCode`, `reference` |
| `amount` | total charged, including customer-paid fee items |
| `amountWithoutDiscount`, `totalDiscounted`, `totalRefunded`, `totalTax` | Amount objects |
| `customerId`, `customerFirstName`, `customerLastName` | |
| `subscriptionId`, `subscriptionName` | set for subscription invoices |
| `checkoutId`, `checkoutResult` | set for checkout invoices |
| `paymentMethodToken`, `paymentMethodData` | full payment method payload |
| `autoPayment`, `processingModel` (`CARD_ON_FILE`, `TAP_TO_PAY`, ...), `transactionSource` | |
| `channel` | for example `api`, `online_payment_recovery`, `MOBILE_POINT_OF_SALE` |
| `createdOn` | |
| `failedPaymentReason.code`, `.description` | generic reason (build logic on this) |
| `paymentProviderResponse.code`, `.description` | raw processor response |
| `payNowUrl` (`paynowUrl` in one schema) | hosted Pay Now link for past due invoices |
| `paymentMethodInvalid` | true means the token is now invalid, replace it before retry |
| `manualRetryPossible` | false means Retry API will be refused |
| `invoiceCategory`, `invoiceSubCategory` | |
| `qrData.qrString`, `qrData.expires` | PromptPay only |
| `externalInvoiceId`, `terminalId` | when set |

### GET /billing/invoices (list)

Query: `customerId`, `subscriptionId`, `externalInvoiceId`, `checkoutId` (session id, not the URL hash), `status`, `from`, `until`, `limit` (default 10, max 100), `cursor`. Sorted newest first. If a filter matches nothing the API returns the unfiltered list, so check each row.

### PUT /billing/invoices/{invoiceId} (update)

`UNPAID` on-demand invoices: all body fields. `PAST_DUE`: only payment method token, scheduled payment date, description, and discounts. Amounts cannot be reduced except through `discountAmount`.

| Field | Req | Notes |
| --- | --- | --- |
| `paymentMethodToken` | no | replaces the invoice token |
| `scheduledPaymentDate` | no | tomorrow to 365 days; `null` disables scheduled payment (`autoPayment=false`); omitted keeps value |
| `externalInvoiceId` | no | |
| `processingModel` | no | `cardonfile` or `recurring` |
| `items[].invoiceItemId` | yes | |
| `items[].discountAmount` | no | Amount; fee items discount in full or not at all |
| `items[].description`, `items[].amount`, `items[].accountingCode`, `items[].reference` | no | |

### POST /billing/invoices/{invoiceId}/retrypayment

Only `PAST_DUE`. Status goes to `PROCESSING`. Body: `oneOff` (`false` default uses the invoice token; `true` uses `paymentMethodToken` for this attempt only). Validates the token first. You must cap manual retries yourself.

### POST /billing/invoices/{invoiceId}/writeoff

`PAST_DUE` or `UNPAID` to `WRITTEN_OFF`. No body. Uncollected customer-paid fees are waived or moved to the merchant per fee configuration. Pay Now link is disabled.

### POST /billing/invoices/{invoiceId}/recordpayment

Only `PAST_DUE`. Body `paymentMethodType`: `cash`, `cheque`, `card`, `others`, `bank_transfer`. Sets `PAID` with no `invoice_paid` webhook, so update local state from your own call. Fees are reallocated to the merchant at next settlement.

### PUT /billing/invoices/{invoiceId}/refund

Only `PAID` or `PARTIALLY_REFUNDED`. Refunds to the original payment method. Fails if the merchant wallet lacks funds (retry later) or if the token was deleted or invalidated.

| Field | Type | Notes |
| --- | --- | --- |
| (empty body `{}`) | | full refund, includes customer-paid fees (fees then charged to merchant; a refund fee may apply) |
| `amount` | number | partial; must be less than invoice amount minus customer fees |
| `items[].id` | string | invoice item id to refund |
| `items[].amount` | number | partial item amount; not allowed on fee items |

Ezypay refunds its fees only on full refunds. Response is the invoice with `creditNoteId`; follow `credit_note_created`, then `credit_note_paid` (success) or `credit_note_failed`. Refunds to bank accounts can take up to 3 business days. NZ card refunds are limited to 60 days after payment (changelog 20/12/2024). PromptPay: `refund_not_allowed`. GCash: no refunds.

### PUT /billing/invoices/{invoiceId}/cancel

PromptPay only: cancels a `PENDING_QR_PAYMENT` invoice. Status goes to `PROCESSING`, then `UNPAID` on success; result arrives in `qr_response` (`cancelStatus` `SUCCESS` or `FAILED`).

## Checkout

### POST /billing/checkout

Header `Merchant`. Card transactions only.

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `amount` | Amount | yes | |
| `description` | string | yes | becomes the main invoice line |
| `accountingCode` | string | no | |
| `successUrl` | string | no (always set it) | redirect after an outcome, any status |
| `cancelUrl` | string | no (always set it) | redirect on cancel, no parameters |
| `customerId` | string | no | omit for anonymous checkout (customer registers on the page) |
| `paymentMethodToken` | string | no | reuse an existing token |

Response: `checkoutUrl` (for example `https://checkout-sandbox.ezypay.com/bf42feec`), `id`. Redirect to `successUrl` appends `documentNumber`, `invoiceId`, `customerId`, `paymentMethodToken`. New payment methods added on checkout raise `payment_method_linked`. Fees shown on the confirmation step vary by method. Customer completes a Captcha. Errors: `currency_not_accepted_by_merchant`, `new_checkout_creation_disabled`, `checkout_session_not_found`, `checkout_session_disabled`.

### DELETE /billing/checkout/{id}

Path value is the hash in the checkout URL (for example `bf42feec`). Irreversible. Default auto-disable after 3 failed attempts (configurable at onboarding). Disable unused sessions to stop card testing (BIN attacks).

## Plans

### POST /billing/plans

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `name` | string | yes | max 50 |
| `memo` | string | no | max 50 |
| `accountingCode` | string | no | max 50 |
| `amount` | Amount | no | gross, tax inclusive |
| `tax` | Tax | no | |
| `setupPayments[]` | array | no | `amount`, `tax`, `description` (max 255), `accountingCode` |
| `intervalUnit` | string | no | `day`, `week` (default), `month` |
| `interval` | integer | no | default 1 |
| `billingStart` | string | no | `immediate` (default), `day_of_week`, `day_of_month` (value in `billingStartValue`) |
| `billingEnd` | string | no | `ongoing` (default), `amount_collected`, `billing_cycles`, `end_date` (value in `billingEndValue`) |
| `firstBilling` | string | no | `full_amount` (default) or `prorate` |
| `metadata` | object | no | |

Response adds `id`, `status` (`ACTIVE`), `recurringBillingDay`, `failedPaymentHandling` (`initialAction`, `autoRetry`, `retryInDays`, `maximumFailedAttempts`), `createdOn`.

`PUT /billing/plans/{planId}`: `name`, `memo`, `amount`, `status` (`active`/`inactive`), `setupPayments`, `intervalUnit`, `tax`, `interval`, `billingStart` (`weekday`, `day_of_month`, `subscription_date` default), `billingEnd`, `firstBilling`, `accountingCode`. Only new subscriptions see the change. Plans cannot be deleted. `GET /billing/plans` filters: `name` (partial), `status`, `limit`, `cursor`.

## Subscriptions

### POST /billing/subscriptions (and POST /billing/subscriptions/preview with the same body)

Fields not given inherit from the plan. If `paymentMethodToken` is omitted the customer's primary token is used. A failed invoice moves the subscription to `PAST_DUE`.

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `customerId` | string | yes | |
| `planId` | string | yes | |
| `paymentMethodToken` | string | no | |
| `markAsPending` | boolean | no | `true` creates `PENDING` (no schedule) until activated |
| `startDate` | string | no | `YYYY-MM-DD`, not in the past; future date creates `FUTURE` |
| `name` | string | no | max 50 |
| `accountingCode` | string | no | max 50 |
| `amount` | Amount | no | |
| `tax` | Tax | no | |
| `setupPayments` | object | no | one-time first-invoice item: `amount`, `tax`, `description`, `accountingCode` |
| `interval` | string | no | default 1 |
| `intervalUnit` | string | no | `day`, `week` (default), `month` |
| `billingStart` | string | no | `immediate`, `day_of_week` (weekly only), `day_of_month` (monthly only) |
| `billingStartValue` | string/int | no | for example `tuesday` or `31` (shown in scenario guide) |
| `billingEnd` | string | no | `ongoing`, `amount_collected`, `billing_cycles`, `end_date` |
| `billingEndValue` | string/number | no | amount, cycle count, or date (shown in scenario guide) |
| `firstBilling` | string | no | `full_amount` or `prorate` |
| `firstBillingAmount` | Amount | no | custom first charge |
| `metadata` | object | no | key max 40, value max 255 |
| `autoPayment` | boolean | no | default `true` |
| `customerEmailNotification` | boolean | no | overrides merchant notification setting |

Response: `id`, `customerId`, `planId`, `name`, `status`, `startDate`, `endDate`, `paymentMethodToken`, `accountingCode`, `amount`, `tax`, `nextBillingDate`, `nextFutureInvoice` (`date`, `cycleStartDate`, `cycleEndDate`, `items`, `amount`, `totalTax`), `interval`, `intervalUnit`, `totalPaid`, `totalBillingCycles`, `remainingToPay`, `remainingBillingCycles`, `endTargetAmount`, `endTargetBillingCycles`, `cancelledDate`, `failedPaymentHandling`, `failedAttemptsCount`, `totalPastDue`, `totalDiscounted`, `metadata`, `createdOn`, `autoPayment`, `setupPayments`, `paymentMethodInvalid`, `paymentMethodInvalidReason`, `paymentMethodInvalidDescription`.

Prorate rule (`/docs/prorate-calculation`): covers signup date to the day before first billing; day rate per month is amount divided by days in that month, rounded to cents; setup payments are not prorated. Docs example: 50 monthly billed on the 5th, signup 11 May, first charge 40.55 on 5 June, then 50.

### Other subscription calls

| Call | Body | Notes |
| --- | --- | --- |
| `PUT /billing/subscriptions/{id}` | `accountingCode`, `name`, `amount`, `billingEnd` (`amount_collected`, `end_date`), `billingEndValue`, `startDate` (FUTURE only), `billingInterval` (`DAY_OF_WEEK`, `DAY_OF_MONTH`, ACTIVE only), `billingIntervalValue` (`MONDAY`..`FRIDAY` or 1 to 31), `autoPayment`, `metadata` | pending, future or active |
| `PUT /billing/subscriptions/{id}/activate` | `startDate` (not back-dated), `paymentMethodToken` | `PENDING` only; falls back to customer default token; invoice generated immediately if start is today |
| `PUT /billing/subscriptions/{id}/cancel` | `cancellationDate` `YYYY-MM-DD` (default today; set one day before the real end) | outstanding invoices remain |
| `POST /billing/subscriptions/{id}/undo-cancel` | none | only `PENDING_CANCELLATION` |
| `PUT /billing/subscriptions/{id}/paymentmethod/{token}` | `{}` | token must already be linked to the customer |
| `GET /billing/subscriptions` | `customerId`, `status`, `filterNullPaymentMethods`, `limit`, `cursor` | newest first |

Billing stops (`subscription_payment_stopped`) when the token becomes invalid, when maximum failed attempts are reached, or when set to stop on first failure (configured by Ezypay). Invoices keep being created but go straight to `past_due`. Reactivate by replacing the token if needed and resolving failed invoices (retry until paid, write off, or record external payment); `subscription_payment_reactivate` follows.

## Future invoices

| Call | Fields | Notes |
| --- | --- | --- |
| `GET /billing/futureinvoices` | `from`, `until` (required), `subscriptionId` or `customerId`, `limit` (1 to 100, default 10), `cursor` | virtual invoices, no id yet |
| `PUT /billing/futureinvoices` | `subscriptionId`, `cycleStartDate`, `date` (required); `items[]` with `type` (`subscription_payment` default, `addon_payment`, `setup_payment`), `amount`, `tax`, `description`, `accountingCode` | new date within the cycle, not today, max 365 days after cycle start; omitting `addon_payment` removes earlier add-ons |
| `DELETE /billing/futureinvoices` | query `subscriptionId`, `cycleStartDate` | skips one cycle; adds a cycle for cycle-count or amount targets; can pause a subscription |
| `POST /billing/futureinvoices/recordpayment` | `subscriptionId`, `cycleStartDate`, `paymentMethodType` (`cash`, `cheque`, `card`, `others`, `bank_transfer`) | issues a `PAID` invoice now and fires `invoice_created` |

## Transactions and credit notes

`GET /billing/transactions` query: `transactionNumber`, `senderId`, `receiverId`, `documentId` (invoice id), `status` (`success`, `processing`, `failed`, `settled`), `from`, `until`, `limit`, `cursor`.

Transaction fields: `id`, `number`, `status`, `createdOn`, `failedOn`, `amount`, `type`, `source`, `paymentMethodType`, `paymentMethodDescription`, `failedPaymentReason`, `paymentProviderResponse.code`/`.description`, `document.id`/`.number`/`.type`, `sender.id`/`.name`/`.type`, `receiver.id`/`.name`/`.type`, `channel`.

`GET /billing/creditnotes` needs one of `customerId`, `subscriptionId`, `invoiceId`; also `status` (`paid`, `processing`, `failed`), `reason` (`chargeback`, `refund`), `from`, `until`, `limit`, `cursor`.

Credit note fields: `id`, `invoiceId`, `documentNumber` (`CN...`), `date`, `status`, `items[]` (`description`, `amount`, `tax`, `id`, `type`, `accountingCode`), `amount`, `totalTax`, `reason` (`REFUND` or `CHARGEBACK`), `customerId`, `paymentMethodToken`.

## Merchant, fees, settlement, reports

- `GET /billing/merchant`: business and address details.
- `GET /billing/merchant/pricing/fees`: customer-paid fees only. `data[]`: `pricingCode` (`failed_payment_fee`, `transaction_fee`, `load_fee`), `description`, `appliesTo` (for example `VISA`, `MASTERCARD`, `BANK`; null or empty means all), `accountingCode`, `chargingOption` (`AMOUNT`, `PERCENTAGE`, `BOTH`), `amount`, `percentage`; plus `paging`. Show these before the customer adds a payment method.
- `GET /billing/merchant/nextSettlement/{merchantId}`: `id`, `number`, `legalName`, `name`, `settlementScheduleDTO` (`intervalUnit`, `interval`, `nextSettlementDate`, `lastSettlementDate`), `amount`. Use it to check wallet funds before refunds or partner invoices.
- `GET /billing/settlements`: query `from`, `until`, `limit`, `cursor`. `data[]`: `number`, `date`, `revenue` (all attempted collection), `deduction` (failed payments, refunds, fees), `amount` (net = revenue minus deduction), `settlementPayoutMethod` (`paymentMethodToken`, `bank.accountHolderName`, `bank.last4`, `bank.bankNumber`, ...), `status` (for example `success`), `failedSettlementReason.code`/`.description`, `createdOn`.
- `POST /billing/settlements/{settlementNumber}/file`: `documentType` (`tax_invoice` PDF or `detail_report` CSV), `sendEmail`, `locale`, `merchantId`; header `Accepted-Language` with `en` or `zh_TW`. Returns `fileId`, `createdOn`, `documentType`.
- `GET /files/{fileId}`: `id`, `url` (time-limited download link), `type`, `status`, `name`, `purpose`, `createdBy`, `createdOn`.
- `POST /billing/reports/merchanttransactions`: `from`, `to` (`yyyy-MM-dd`, required, max 90 days). Returns CSV with columns Transaction Date, Transaction Type, Currency, Amount, Customer Number, Customer Name, Merchant Number, Merchant Name, Transaction Source, Payment Method Type, Transaction Number, Document Number, Document Date, Item Type, Description, Accounting Code, Channel.
- `POST /billing/reports/outstandingcollectionsreport/generate`: `reportFilters` (required: `FAILED_WITHOUT_FURTHER_RETRY`, `FAILED_BUT_WILL_ATTEMPT_RETRY`, `DUE_BUT_NOT_ATTEMPTED_YET`, `NOT_YET_DUE`), `sortField` (required: `CUSTOMER_NAME`, `CUSTOMER_NUMBER`, `EZYPAY_NUMBER`, `TOTAL_OUTSTANDING_AMOUNT`), `reportFormat` (`PDF` default or `CSV`), `daysOutstanding` (`ALL`, `THIRTY` default, `SIXTY`, `NINETY`).

## Partner billing (multi-split payouts)

Moves funds between merchant wallets before settlement, with no holding period. Needs merchant consent, Ezypay enabling the wallet connection, enough wallet funds, and the `partner` token scope.

- `POST /partnerinvoices`: `issuerMerchantId`, `recipientMerchantId`, `items[]` (`description`, `amount`, `tax`, `accountingCode`) required; `memo` (max 100).
- `GET /partnerinvoices`: `issuerMerchantId`, `recipientMerchantId` (required), `status` (`paid`, `past_due`), `from`, `until`, `countryCode`, `partnerSubscriptionId`, `limit`, `cursor`.
- `POST /retrypartnerinvoices/{partnerInvoiceId}`: retry a past due partner invoice.
- `POST /partnersubscriptions`: `partnerPlanId`, `recipientId` required; `startDate`, `metadata`. Response includes `issuer`, `recipient`, `amount`, `intervalUnit`, `interval`, `intervalDayType`, `endType`, `merchantConnectionId`, cycle counters.
- `GET /partnersubscriptions`: `recipientId`, `status` (`active`, `inactive`), `name`, `limit`, `cursor`. `PUT /partnersubscriptions/{id}/cancel`.
- `POST /partnersubscriptions/{id}/addons/{code}/recordusage`: `value` (required, integer 0 or more), `displayMeasurementUnit` (max 100), `referenceCode`, `recordingTimestamp` (`yyyy-MM-ddTHH:mm:ss[.SSS]`); add-on `type` must be `amount_per_usage`. `GET` same path lists usage (filters `referenceCode`, `recordingTimestampFrom/Until`, `createdFrom/Until`).
- `GET /merchantconnections` (`status`: `pending`, `active`, `cancelled`), `GET /merchantconnections/{id}`.
- Track with `partner_invoice_created`, `partner_invoice_paid`, `partner_invoice_past_due`.

## Tap to Pay (AU and NZ only, listed for completeness)

`POST /billing/terminal/register`: `appDeviceId`, `deviceName` (unique per merchant) required. Returns `code` (6-digit activation code, 24 hours), `deviceName`, `appDeviceId`, `message`. Activation confirmed by `terminal_registration` webhook.
`POST /billing/terminal/invoices` (header `merchantId`): `terminalId`, `items[]` required; `memo`, `autoPayment`, `externalInvoiceId`, `customerId` (optional, guest if omitted). Max 1,500 per invoice. Status `PENDING_TERMINAL_PAYMENT` then `PAID`, `FAILED` or `VOIDED` via `terminal_response`. After that only refunds are possible.
`GET /billing/terminal/invoices`: `customerId`, `terminalId`, `externalInvoiceId`, `status`, `from`, `until`, `limit`, `cursor`, `sortBy`.
