# Betterpay Recurring Reference

Checked: 2026-10-21.

Sources:

- Docs `https://www.betterpay.me/docs/#recurring1` (PRO direct debit)
- Docs `https://www.betterpay.me/docs/#batchrecurring1` (OCBC collection)
- Docs `https://www.betterpay.me/docs/#bplite_recurring1` (LITE card)
- Postman `Recurring (BP PRO)` and `Recurring (BP LITE)` at `https://github.com/betterpayme/POSTMAN-APIs`

All calls are POST with JSON bodies (docs samples use `Content-Type: application/json`). The sandbox host is `https://www.demo.betterpay.me`. Production paths are not published, so confirm them in the logged-in docs.

## 1. PRO Recurring: FPX Direct Debit (e-Mandate)

- Periodic debits (daily, weekly, monthly, yearly) authorised once by the payer through an FPX e-Mandate.
- The first (enrolment) payment is RM1.00.
- Create a subscription package first, then add subscribers to it.

Package types:

- `fixed`: same amount every period, no expiry.
- `flexi`: the payer sets the amount, no expiry.
- `calculated`: tiered amounts with an expiry, e.g. hire purchase.

| Operation | Sandbox path (Postman) | MD5 hash string |
|---|---|---|
| Add subscription | `/merchant/api/recurring/v1/add_subscription` | `api\|merchant_id\|subscription_id\|subscription_type\|name\|price\|terms\|purpose` (`price` empty if not fixed) |
| Update subscription | Not published | `api\|merchant_id\|subscription_id` |
| Add subscriber | `/merchant/api/recurring/v1/add_subscribers/{subscription_id}` | `api\|merchant_id\|amount\|subscriber_details.id_number\|subscriber_details.email\|subscriber_details.reference_no\|callbacks.back_end\|bank_id` |
| Get subscribers (of a package) | `/merchant/api/recurring/v1/get_subscribers/{subscription_id}` | `api\|merchant_id\|subscription_id` |
| Get subscriber details | Not published (URL param `{Subscriber Reference No}`) | `api\|merchant_id\|reference_no` |
| Get subscriptions (all) | Not published | `api\|merchant_id` |
| Get subscription details | Not published (URL param `{Subscription ID}`) | `api\|merchant_id\|subscription_id` |
| Modify active subscriber | Not published (URL param `{Subscriber Reference No}`) | `api\|merchant_id\|reference_no` |
| Terminate active subscriber | Not published (URL param `{Subscriber Reference No}`) | `api\|merchant_id\|reference_no` |

The PRO Postman collection also has "Get Single Subscriptions", "Get All Subscribers" and "Get Single Subscriber Data" items that point at LITE (`/bplite/...`) paths with HMAC hashing. Treat those as Postman mistakes for PRO and follow the docs.

### Add subscription (PRO)

| Field | Required | Notes |
|---|---|---|
| `merchant_id` | yes | |
| `subscription_type` | yes | `fixed`, `flexi` or `calculated` |
| `terms` | yes | `Daily`, `Weekly`, `Monthly` or `Yearly`. May be empty for `flexi` so the payer chooses |
| `subscription_id` | yes | Unique, alphanumeric, max 18, no spaces |
| `name` | yes | Max 50 |
| `price` | fixed only | 2dp |
| `purpose` | yes | Max 27 |
| `repayment_periods` | calculated only | List of `{total, terms (Week\|Month\|Year), amount}`. `amount` is per term |
| `max_debit_frequency` | no | Max debits allowed per frequency period under the mandate |
| `hash` | yes | |

Response:

- `response` (`00`)
- `system_subscription_id`, `subscription_id`, `subscription_type`, `name`, `terms`, `purpose`
- `repayment_periods`
- `created`, `modified`

Update subscription: `merchant_id`, `subscription_id`, `hash`. Optional: `terms`, `price`, `purpose`, `max_debit_frequency`, `repayment_periods` (calculated).

### Add subscriber (PRO)

| Field | Required | Notes |
|---|---|---|
| URL `{subscription_id}` | yes | |
| `merchant_id` | yes | |
| `subscriber_details.id_number` | yes | IC number etc., digits |
| `subscriber_details.id_type` | yes | `1` new IC, `2` old IC, `3` passport, `4` business registration, `5` others |
| `subscriber_details.name` | yes | |
| `subscriber_details.reference_no` | yes | Your reference, digits only, max 10 (the URL-param sections say max 12) |
| `subscriber_details.email`, `.phone` | no | |
| `mandate_frequency` | flexi | `Daily`, `Weekly`, `Monthly` or `Yearly` |
| `bank_id` | yes | From Get Bank List (e.g. `MB2U0227`) |
| `amount` | flexi | 2dp |
| `repayment_periods` | calculated | `{total, terms, amount}` |
| `payment_method` | no | `b2c` (default) or `b2b` |
| `callbacks.back_end`, `.fe_success`, `.fe_fail` | no | Callback and return URLs |
| `hash` | yes | |

Response:

- `response`, `system_subscriber_id`, `amount`, `frequency`
- `subscriber_details`, `callbacks`, `bank_id`, `hash`
- `paymentURL`: send the payer here to sign the e-Mandate
- `created`, `modified`

### Other PRO recurring calls

- **Get subscribers**: optional `page`, `no_of_record_per_page` (default 100). Response: `response`, `total_page_available`, rows.
- **Get subscriber details** response:
  - `response`, `subscriber_ref_no`, `name`, `id_number`, `phone`, `email`
  - `subscription_id`, `active_date`, `subscription_name`, `amount`, `subscription_purpose`
  - `fpx_reference_no`: keep it; OCBC collection needs it
  - `subscription_status`, `created`, `modified`
- **Get subscriptions**: optional `page`, `offset` (under 100).
- **Modify active subscriber**: optional `terms`, `price`, `last_date_payment` (yyyy-mm-dd), `subscriber_details.phone`, `.email`. The response is a URL string (`.../merchant/mandate/update?...`). The subscriber must confirm the change at their bank.
- **Terminate active subscriber**: the response is a URL string (`.../merchant/mandate/terminate?...`). The subscriber confirms at their bank.

### Subscriber callback (PRO)

POSTed to your back-end callback. Two callbacks arrive for the same reference:

1. `msg` = `Transaction Approved`: approved by the subscriber.
2. `msg` = `Direct Debit Approved`: approved by the bank.

Only debit after both.

Fields:

- `others_trx_id`, `reference_no`, `invoice_no`, `txn_status`, `msg`
- `pay_method` (`others`), `trx_txt` (`emandate`)
- `subscription_id`, `subscription_status` (`new`, `maintenance` or `terminated`)
- `bank_type`, `bank_id`, `bank_branch`, `hash`

```
expected = md5(api + others_trx_id + reference_no + invoice_no + txn_status)   // no separators
```

Reply with plain `OK`. Betterpay retries at most 3 times every 10 minutes.

## 2. PRO OCBC Recurring Collection (batch debit)

- Supported bank: OCBC.
- You submit batches that debit subscribers who already hold an approved mandate. Each subscriber is identified by `fpx_reference_no`, which you get from the subscriber callback / details.
- Limits: max 99 successful Add Collection calls per day, max 200 subscribers per batch.

| Operation | MD5 hash string |
|---|---|
| Add Collection | `api\|merchant_id\|batch_id\|collection_date\|fpx_reference_no` |
| Cancel Collection by Batch | `api\|merchant_id\|batch_id` |
| Cancel Single Collection | `api\|merchant_id\|batch_id\|fpx_reference_no` |
| Get All Submitted Batch ID | `api\|merchant_id` |
| Get Collection By Batch ID | `api\|merchant_id\|batch_id\|status` |
| Get Collection Details | `api\|merchant_id\|batch_id\|fpx_reference_no` |
| Callback (verify) | `api\|merchant_id\|batch_id\|{ref1},{msg1},{ref2},{msg2}...` |

Paths: Not published (the DuitNow QR (OCBC) Postman collection does not include them). The hashing tables in this section list Merchant ID before API, but the PHP code and the example strings put the API key first. Use API key first.

Add Collection fields:

| Field | Required | Notes |
|---|---|---|
| `merchant_id` | yes | |
| `batch_id` | yes | Unique, max 14, no `&`, `'` or `-`. Duplicate: `99` "Batch ID is already exist" |
| `collection_date` | yes | `dd-mm-yyyy`. Rules below |
| `fpx_reference_no` | yes | With `max_amount` empty or `1`: comma list `ref1,ref2,...`, each debited at the mandate maximum. With `max_amount=0`: `ref,amount;ref,amount;`, each over RM1.50 and under RM100,000.00. Max 200 subscribers, each ref max 30 |
| `max_amount` | no | `1`/empty = charge mandate max. `0` = custom amounts, up to the mandate max |
| `response_format` | no | `1`/empty = single string. `2` = `data` array |
| `hash` | yes | |

Collection date rules:

| When you submit | Allowed collection date |
|---|---|
| Mon to Fri before 3:31 PM | Any date not before today |
| Mon to Thu after 3:30 PM | Next day or later |
| Fri after 3:30 PM, or Sat/Sun | Next Monday or later |
| Any time | Never a Federal Territory public holiday |

Add Collection response: `response`, `status`, `batch_id`, `collection_date`, `fpx_reference_no` (or `data[]` of `{fpx_reference_no, amount}`), `created`.

- **Get All Submitted Batch ID**: optional `date_from`, `date_to`, `page`.
- **Get Collection By Batch ID**:
  - `status` filter: `0` failed, `1` successful, `2` all.
  - Rows: `batch_id`, `fpx_reference_no`, `collection_date`, `date_created`, `status`, `callback_url_be`, `permission_to_cancel` (`ALLOWED` or `NOT ALLOWED`).
  - Error: `99` "Authentication error. Check your hash and try again."
- Collection status values are listed in `betterpay-codes.md`.

Collection callback body:

```
{"batch_id":"BATCH0000001","data":[{"status":"00","msg":"SUCCESSFUL","fpx_reference_no":"C5R..."},...],"hash":"..."}
```

- `data[].status` is `00` (success) or `99` (fail).
- Data string: `{FPX Reference No #1},{Status Message #1},{FPX Reference No #2},{Status Message #2}...`.
- Hash: `md5(API|Merchant ID|Batch ID|DataString)`. Docs example: `APIKEY123456|10122|BATCH0000001|C5R213131231231C1,SUCCESSFUL,C5R123122131311B1,CANCELLED BY CUSTOMER`.
- Reply `OK`.

## 3. LITE Recurring: Credit/Debit Card

- Monthly card recurring. Enrolment is a card verification charged at RM1.00.
- Create a package, then add a subscriber. Alternatively, give the payer the package `paymentURL` and skip Add Subscriber.
- All LITE recurring calls use the LITE HMAC-SHA256 hash. Nested objects are concatenated in the order sent (see `betterpay-api.md`).

| Operation | Sandbox path (Postman) |
|---|---|
| Add subscription | `/merchant/api/recurring/v1/bplite/add_subscription` |
| Add subscriber | `/merchant/api/recurring/v1/bplite/add_subscriber` |
| Get all subscriptions | `/merchant/api/recurring/v1/bplite/get_subscription` |
| Get single subscription | `/merchant/api/recurring/v1/bplite/get_subscription_details` |
| Get all subscribers | `/merchant/api/recurring/v1/bplite/get_subscribers` |
| Get single subscriber | `/merchant/api/recurring/v1/bplite/get_subscriber_details` |
| Update subscriber | `/merchant/api/recurring/v1/bplite/update_subscriber` (Postman only, not in docs) |
| Terminate subscriber | `/merchant/api/recurring/v1/bplite/terminate` (Postman only, not in docs) |

Add subscription (LITE):

- Fields: `merchant_id`, `subscription_type` (`fixed`, `flexi` or `calculated`), `subscription_id` (max 18), `name` (max 50), `purpose` (max 27), `repayment_periods` (list of `{totalMonth, amount}`), `hash`.
- Response: `system_subscription_id`, `subscription_id`, `subscription_type`, `name`, `terms`, `purpose`, `repayment_periods` (`total`, `terms`, `amount`), `created`, `modified`, `paymentURL`.

Add subscriber (LITE):

- Fields: `merchant_id`, `subscriber_details` (`id_number`, `id_type` 1 to 5, `name`, `reference_no` max 12, `email`, `phone`), `subscription_id`, `repayment_periods` (`totalMonth`, `amount`, which must match the package), `hash`.
- Response: `response`, `system_subscriber_id`, `subscription_id`, `subscriber_details`, `hash`, `created`, `modified`, `paymentUrl` (e.g. `https://www.demo.betterpay.me/merchant/api/recurring/v1/bplite/enroll?...`).

New subscriber callback (LITE):

- How to set the URL: Dashboard > Recurring > Settings > Callback URL tab.
- Body: `bp_lite_trx_id`, `merchant_id`, `invoice_no`, `txn_status`, `msg`, `txn_amount` (`1.00` for verification), `pay_method` (`card`), `hash`.
- Verify with the LITE HMAC method over the received fields except `hash`.

Get calls (LITE):

| Call | Request | Response |
|---|---|---|
| Get all subscriptions | `merchant_id`, `hash`, optional `page`, `offset` (default 100 rows) | `total_page_available`, `response`, `data` |
| Get single subscription | `subscription_id` | package fields |
| Get all subscribers | `subscription_id`, optional `page`, `offset` | `total_page_available`, `response`, `data` |
| Get single subscriber | `system_subscriber_id` | `system_subscriber_id`, `subscription_id`, `subscriber_details`, `created`, `modified` |

Postman-only bodies (not in docs):

- Update subscriber: `system_subscriber_id`, `price`, `last_date_payment`, `subscriber_details.email`, `.phone`.
- Terminate: `system_subscriber_id`.

Confirm both with support before relying on them.
