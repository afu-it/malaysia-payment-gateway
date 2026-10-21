# Omise Customers, Saved Cards and Schedules (recurring)

Checked 2026-10-21. Recurring charges on Omise are card-based: save a card on a Customer, then charge it yourself (merchant initiated) or let a Schedule create charges. Atome does not support recurring payments.

## Source URLs

- Customer API: `https://docs.omise.co/en/customers-api/malaysia`
- Card API: `https://docs.omise.co/en/cards-api/malaysia`
- Schedule API: `https://docs.omise.co/en/schedules-api/malaysia`
- Charge Schedule object: `https://docs.omise.co/en/charge-schedules-api/malaysia`
- Charge Schedule bulk API: `https://docs.omise.co/en/charge-schedule-api/malaysia`
- Occurrence API: `https://docs.omise.co/en/occurrences-api/malaysia`
- Transfer Schedule: `https://docs.omise.co/en/transfer-schedules-api/malaysia`
- Recurring payments (support): `https://docs.omise.co/en/how-to-do-recurring-payments/malaysia`
- Retries for scheduled transactions: `https://docs.omise.co/en/how-retries-work-for-scheduled-transactions/malaysia`
- 29th to 31st in monthly schedules: `https://docs.omise.co/en/why-isnt-the-29-31-supported-in-monthly-schedules/malaysia`
- Charging cards: `https://docs.omise.co/en/charging-cards/malaysia`

## Customers

| Endpoint | Parameters |
| --- | --- |
| `POST /customers` | `card` (unused token id), `email` (recommended), `description` (recommended), `metadata`, `linked_account` |
| `GET /customers` | list params |
| `GET /customers/{id}` | |
| `PATCH /customers/{id}` | `card` (adds a new card from a token), `default_card` (card id), `email`, `description`, `metadata`, `linked_account` |
| `DELETE /customers/{id}` | |

Customer object: `object=customer`, `id` (`/cust(_test)?_[0-9a-z]+/`), `livemode`, `location`, `email`, `description`, `metadata`, `default_card` (first card initially; if deleted, the latest added card becomes default; `null` with no cards), `cards` (list), `linked_accounts`, `deleted`, `created_at`.

A charge can also create the customer inline: `customer[card]=tokn_...`, `customer[email]`, `customer[description]`, `customer[metadata]`; or add a card to an existing customer with `customer[id]` plus `customer[card]`.

```bash
curl https://api.omise.co/customers -u "$OMISE_SECRET_KEY:" \
  -d "email=buyer@example.my" -d "description=User 42" -d "card=tokn_test_xxx"
```

Google Pay card tokens cannot be attached to customers.

## Cards (under a customer)

| Endpoint | Parameters |
| --- | --- |
| `GET /customers/{id}/cards` | list params |
| `GET /customers/{id}/cards/{card_id}` | |
| `PATCH /customers/{id}/cards/{card_id}` | `name`, `expiration_month`, `expiration_year`, `city`, `postal_code` |
| `DELETE /customers/{id}/cards/{card_id}` | |

Card object: `object=card`, `id` (`/card(_test)?_[0-9a-z]+/`), `livemode`, `location`, `brand`, `bank` (from IIN, may be wrong), `country`, `financing` (`Credit`, `Debit`, `Prepaid`, `Charge`, `Deferred Debit`, `Loan`, `Unknown`, empty), `fingerprint` (detect identical cards), `first_digits` (only for some merchants), `last_digits`, `name`, `expiration_month`, `expiration_year`, `street1`, `street2`, `city`, `state`, `postal_code`, `phone_number`, `security_code_check` (always `true` since 1 April 2020), `tokenization_method` (`googlepay` or `null`), `deleted`, `created_at`.

Webhooks: `card.update`, `card.destroy`, `customer.update.card`.

## Merchant-initiated charge on a saved card

```bash
curl https://api.omise.co/charges -u "$OMISE_SECRET_KEY:" \
  -d "amount=4900" -d "currency=MYR" -d "customer=cust_test_xxx" -d "card=card_test_xxx" \
  -d "description=Plan Pro October" -d "metadata[invoice_id]=INV-77" \
  -d "recurring_reason=subscription" -d "transaction_indicator=MIT" -d "first_charge=chrg_test_first" \
  -d "ref_id=inv-77"
```

`recurring_reason` values: blank (store credential for future use), `unscheduled`, `standing_order`, `subscription`, `installment`, `partial_shipment`, `delayed_charge`, `no_show`, `resubmission`. `transaction_indicator`: `MIT` or `CIT`. `first_charge`: the first charge id in the cycle, "to increase the likelihood of approval". If 3DS is enabled on the account, see the 3DS trade-off in `omise-charges.md`.

## Schedules

`POST https://api.omise.co/schedules` (secret key).

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `every` | integer | yes | Interval count, for example `3` with `period=week` is every 3 weeks |
| `period` | string | yes | `day`, `week`, `month` |
| `start_date` | string | yes | ISO 8601 date |
| `end_date` | string | yes | ISO 8601 date |
| `on` | object | no | `on[weekdays][]=monday` (weekly), `on[days_of_month][]=1` (monthly by date), `on[weekday_of_month]=2nd_monday` (monthly by weekday) |
| `charge` | object | conditional | Charge schedule: `charge[customer]` (required), `charge[amount]` (required), `charge[card]`, `charge[currency]`, `charge[description]` |
| `transfer` | object | conditional | Transfer schedule: `transfer[recipient]` (required), `transfer[amount]`, `transfer[currency]`, `transfer[percentage_of_balance]` (omit amount to send the whole balance) |

```bash
curl https://api.omise.co/schedules -u "$OMISE_SECRET_KEY:" \
  -d "every=1" -d "period=month" -d "on[days_of_month][]=1" \
  -d "start_date=2026-11-01" -d "end_date=2027-10-31" \
  -d "charge[customer]=cust_test_xxx" -d "charge[amount]=4900" -d "charge[description]=Membership"
```

- Test mode limit: 10 active scheduled charges (UI and API combined).
- Days 29 to 31: the newer support article (updated 2024-03-18) says the schedule runs on the last day of shorter months; an older article says those days are not supported. Prefer day 1 to 28 to be safe.
- Retries: a failed scheduled charge is retried the next day, at most 2 retries; after 3 consecutive failures the schedule is `suspended`. The Schedule API text says schedules are suspended three days after the charge fails, retried once per day.
- Failed scheduled transfers: unverified or inactive recipient marks it failed; insufficient transferable balance retries every business day until it succeeds.
- One transfer schedule per recipient.
- Dashboard: Charge Schedule menu (and bulk upload of many cards as a CSV).

### Schedule object

`object=schedule`, `id` (`/schd(_test)?_[0-9a-z]+/`), `livemode`, `location`, `status`, `active` (true when `running`/`active` or `expiring`), `state` (Active or Paused), `every`, `period`, `on`, `in_words` (for example "Every 2 months on the 15th"), `start_on`, `end_on`, `ended_at`, `next_occurrences_on` (up to 30 dates), `occurrences` (list), `charge` (scheduled charge) or `transfer` (scheduled transfer), `deleted`, `deleted_by`, `created_at`, plus `creator_display_name`, `execute_time`, `is_admin_managed`, `merchant_name` (TBD in docs).

`status` values on the Schedule API page: `active`, `expiring`, `expired`, `deleted`, `paused`, `suspended`. The 2019-05-29 upgrade guide and the example response use `running` for the active state. Accept both.

Scheduled charge object: `object=scheduled_charge`, `id` (`rchg_...`), `livemode`, `amount`, `currency`, `customer`, `card`, `default_card` (boolean), `description`, `metadata`, `created_at`.

Scheduled transfer object: `object=scheduled_transfer`, `id` (`rtrf_...`), `livemode`, `amount`, `currency`, `percentage_of_balance`, `recipient`, `is_realtime_payout`, `created_at`.

### Schedule endpoints

| Endpoint | Purpose |
| --- | --- |
| `GET /schedules` | List all schedules |
| `GET /schedules/{id}` | Retrieve |
| `DELETE /schedules/{id}` | Delete (stop) |
| `GET /schedules/{id}/occurrences` | Occurrences of a schedule |
| `GET /charges/schedules` | List charge schedules |
| `GET /customers/{id}/schedules` | Charge schedules for a customer |
| `GET /transfers/schedules` | List transfer schedules |
| `GET /recipients/{id}/schedules` | Transfer schedules for a recipient |

Webhooks: `schedule.create`, `schedule.destroy`, `schedule.expire`, `schedule.expiring`, `schedule.suspend`. Charges created by a schedule have `schedule` set and fire normal charge events.

### Occurrence object

`object=occurrence`, `id` (`/occu(_test)?_[0-9a-z]+/`), `livemode`, `location`, `schedule`, `scheduled_on` (date), `processed_at`, `status` (`skipped`, `failed`, `successful`), `result` (charge or transfer id), `message` (failure reason), `retry_on` (date), `created_at`. `GET /occurrences/{id}`.

## Charge schedule bulk APIs

Secret key. Documented on the Charge Schedule page.

| Endpoint | Body | Purpose |
| --- | --- | --- |
| `POST /schedules/upload` | multipart `file` (CSV) | Bulk create charge schedules |
| `GET /recurring_exports/{recurring_id}` | | Monitor a bulk job |
| `GET /recurring_exports/{recurring_id}/download` | | Download the job status report |
| `GET /charges/schedules` | list params | List charge schedules |
| `PATCH /schedules/bulk_pause` | `schedule_ids` (array) | Pause schedules |
| `PATCH /schedules/bulk_resume` | `schedule_ids` | Resume schedules |
| `DELETE /schedules/bulk_delete` | `schedule_ids` | Delete schedules |

Responses: `200` (pause and resume can return 200 with a failure list for unknown ids), `400` (bad file), `401` (bad key), `404` (unknown job or schedule id).

CSV format (documented sample; the sample puts token ids in the `card` column):

```text
customer_key,customer,card,amount,description,every,period,days_of_month,start_date,end_date
CUST0001001,cust_test_0000000000000001,tokn_test_0000000000000001,79000,REF0001001SAMPLE,1,month,5,2023-8-30,2028-4-5
```

```bash
curl -X POST https://api.omise.co/schedules/upload -u "$OMISE_SECRET_KEY:" \
  -H "Content-Type: multipart/form-data" -F "file=@schedules.csv;type=text/csv"
```
