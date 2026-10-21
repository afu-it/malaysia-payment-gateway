# Omise Balance, Payouts and Reporting

Checked 2026-10-21. Malaysia: funds settle in MYR to a Malaysian bank account only.

## Source URLs

- Balance API: `https://docs.omise.co/en/balance-api/malaysia`
- Transaction API: `https://docs.omise.co/en/transactions-api/malaysia`
- Transfer API: `https://docs.omise.co/en/transfers-api/malaysia`
- Transfers guide: `https://docs.omise.co/en/transferring-funds/malaysia`
- Recipient API: `https://docs.omise.co/en/recipients-api/malaysia`
- Bank Account object: `https://docs.omise.co/en/bank-account-api/malaysia`
- Supported banks: `https://docs.omise.co/en/supported-banks/malaysia`
- Transfer (withdraw) balance, Malaysia: `https://docs.omise.co/en/how-do-i-transfer-withdraw-my-balance/malaysia`
- Holding period: `https://docs.omise.co/en/how-long-does-omise-hold-my-money/malaysia`
- Balance types: `https://docs.omise.co/en/what-are-the-total-on-hold-and-transferable-balance/malaysia`
- Automatic payouts: `https://docs.omise.co/en/how-to-set-up-automatic-payouts/malaysia`
- Delete a transfer request: `https://docs.omise.co/en/how-to-delete-a-transfer-request/malaysia`
- Receipt API: `https://docs.omise.co/en/receipts-api/malaysia`
- Search API: `https://docs.omise.co/en/search-api/malaysia`
- Search query and filters: `https://docs.omise.co/en/search-query-and-filters/malaysia`
- Account API: `https://docs.omise.co/en/account-api/malaysia`
- Capability API: `https://docs.omise.co/en/capability-api/malaysia`
- Forex API: `https://docs.omise.co/en/forex-api/malaysia`
- Currencies: `https://docs.omise.co/en/currency-and-amount/malaysia`

## Settlement timeline (Malaysia)

- Charges are held 3 days in the On hold balance, then move to Transferable. Daily batch covers midnight to midnight; for example charges on December 1 move on December 4 at 23:59.
- Transfers are batched and processed once a day on business days (Monday to Friday, not bank holidays). Oldest first.
- Funds reach the bank in 3 business days.
- Need at least RM1.00 transferable and no other incomplete transfer.
- Transfer fee: the Malaysia withdraw article (updated 2026-03-19) charges a fee on small transfers and waives it above a threshold; read the current numbers there and on the pricing page.
- Reserve: a percentage of the 6-month average balance may be held for higher-risk merchants.
- Test mode: received amounts become transferable immediately; transfers are simulated with "Mark as sent" and "Mark as paid".
- The Step by Step Guide and the Transfers guide quote Thai rules (7 day hold, "over 30 THB"); use the Malaysia articles above.

## Balance

`GET /balance`: `object=balance`, `livemode`, `location`, `currency`, `total` (after fees), `transferable`, `on_hold`, `reserve`, `at`, `created_at`.

## Transactions (balance ledger)

`GET /transactions` (list params), `GET /transactions/{id}`.

Transaction object: `object=transaction`, `id` (`trxn_...`), `livemode`, `location`, `amount`, `currency`, `direction` (`credit` or `debit`), `key` (for example `charge.captured.net`), `origin` (charge, refund, transfer, dispute or receipt id), `transferable_at`, `created_at`, `merchant_name`, `merchant_uid`. The Charges API shows the gross amount; transactions show net after fee and tax.

## Transfers

`POST /transfers` (secret key):

| Parameter | Type | Notes |
| --- | --- | --- |
| `amount` | integer | Optional; default is the full transferable balance. Must not exceed transferable |
| `recipient` | string | Optional; default is the account default recipient |
| `fail_fast` | boolean | `true` means do not retry on failure. Default `false` |
| `metadata` | object | |
| `idemp_key` | string | "Idemp key" (idempotency) |
| `split_transfer` | boolean | Split transfer |

Other endpoints: `GET /transfers`, `GET /transfers/{id}`, `DELETE /transfers/{id}` (not after it is marked sent), `PATCH /transfers/{id}` (callback: `success`, `failure_code`, `failure_message`), `POST /transfers/{id}/mark_as_sent` and `/mark_as_paid` (test only).

Transfer object: `object=transfer`, `id` (`trsf_...`), `livemode`, `location`, `amount`, `currency`, `fee`, `fee_vat`, `total_fee`, `net`, `recipient`, `bank_account`, `sendable`, `sent`, `sent_at`, `paid`, `paid_at`, `fail_fast`, `failure_code` (`insufficient_balance`, `invalid_recipient`, `transfers_suspended`, `transfer_deleted`, `transfer_sent`, `transfer_failed`), `failure_message`, `schedule`, `transactions`, `metadata`, `deleted`, `created_at`, plus `balance_hold_at`, `split_transfer_group_id`, `creator_display_name`, `is_admin_managed`, `merchant_info`, `merchant_name`, `merchant_uid`.

Webhooks: `transfer.create`, `transfer.update`, `transfer.send`, `transfer.pay`, `transfer.fail`, `transfer.destroy`.

Automatic payouts: Transfer schedules (dashboard or `POST /schedules` with `transfer[...]`, see `omise-recurring.md`): whole balance, fixed amount, or percentage of balance.

## Recipients (third-party payouts)

`POST /recipients`: `name`, `email`, `description`, `type` (`individual` or `corporation`), `tax_id`, `metadata`, `bank_account` (object), `document_ids`, `relationship_type`. The Transfers guide shows `bank_account[brand]`, `bank_account[number]`, `bank_account[name]`, with `brand=test` in test mode. The Bank Account object exposes `bank_code`, `brand` (bank name), `account_number`, `last_digits`, `name`; check which key your API version expects for the Malaysian bank code (codes below).

Lifecycle: created `inactive` and unverified; Omise verifies (usually within two business days; failures `name_mismatch`, `account_not_found`, `bank_not_found`); you activate it in the dashboard. Both are needed before transfers. `auto_activate_recipients` exists on the Account object.

Endpoints: `GET /recipients`, `GET /recipients/{id}`, `PATCH /recipients/{id}`, `DELETE /recipients/{id}`, `PATCH /recipients/{id}/verify` (manual verify, for testing).

Recipient object: `object=recipient`, `id` (`recp_...`), `livemode`, `location`, `name`, `email`, `description`, `type`, `tax_id`, `bank_account`, `active`, `activated_at`, `verified`, `verified_at`, `default`, `failure_code`, `schedule`, `metadata`, `deleted`, `created_at`, `recipient_relationships`, `relationship_type`.

Webhooks: `recipient.create`, `recipient.update`, `recipient.verify`, `recipient.activate`, `recipient.deactivate`, `recipient.destroy`.

### Malaysian bank codes for recipients (Supported banks page)

`AFBQMYKL` MBSB Bank; `AGOB9999`, `AGOBMY21`, `AGOBMYK1`, `AGOBMYKL`, `BPMBMYKL` Bank Pertanian (Agrobank) variants; `AIBBMYKL` Affin Islamic; `AISLMYKL` AmIslamic; `ALSRMYK1`, `ALSRMYKL` Alliance Islamic; `ARBKMYKL` AmBank; `BIMBMYKL` Bank Islam; `BKCHMYKL` Bank of China (Malaysia); `BKKBMYKL` Bangkok Bank; `BKRMMYK1`, `BKRMMYKL` Bank Rakyat; `BMMBMYKL` Bank Muamalat; `BNPA9999`, `BNPAMY21`, `BNPAMYKL` BNP Paribas Malaysia; `BOFAMY2X` Bank of America; `BOTKMYKX` MUFG Bank (Malaysia); `BSNA9999`, `BSNAMY21`, `BSNAMYK1` BSN; `CHASMYKX` J.P. Morgan Chase; `CIBBMYKL` CIMB Bank; `CTBBMYKL` CIMB Islamic; `CITI9999`, `CITIMYKL`, `CITIMYM1` Citibank; `DEUTMYKL` Deutsche Bank; `HBMBMYKL` HSBC Bank; `HMABMYKL` HSBC Amanah; `HLBBMYKL` Hong Leong Bank; `HLIBMYKL` Hong Leong Islamic; `ICBKMYKL` ICBC; `KFHOMYKL` Kuwait Finance House; `MBBEMYKL` Maybank; `MBISMYKL` Maybank Islamic; `MFBBMYKL` Alliance Bank; `MHCBMYKA` Mizuho; `OCBCMYKL` OCBC; `OABBMYKL` OCBC Al-Amin; `PBBEMYKL` Public Bank; `PIBEMYK1` Public Islamic; `PCBCMYKL` China Construction Bank; `PHBMMYKL` Affin Bank; `RHBBMYKL` RHB Bank; `RHBAMYKL` RHB Islamic; `RJHIMYKL` Al Rajhi; `SCBLMYKX` Standard Chartered; `SCSRMYK1`, `SCSRMYKK` Standard Chartered Saadiq; `SMBCMYKL` Sumitomo Mitsui; `UOVBMYKL` UOB. `GET /capability` also returns `banks` for transfers.

## Receipts (Omise fee invoices)

`GET /receipts`, `GET /receipts/{id}`. Fields: `id` (`rcpt_...`), `number`, `issued_on`, `currency`, `charge_fee`, `voided_fee`, `transfer_fee`, `transaction_fee`, `subtotal`, `vat`, `wht`, `total`, `credit_note`, `company_name`, `company_address`, `company_tax_id`, `customer_name`, `customer_address`, `customer_email`, `customer_statement_name`, `customer_tax_id`, `adjustment_transaction`, `created_at`.

## Search

`GET /search` with `scope` (required: `charge`, `charge_schedule`, `customer`, `dispute`, `event`, `link`, `linked_account`, `receipt`, `recipient`, `refund`, `transfer`, `transfer_schedule`, `transaction`, plus `audit`, `deposit_summary`, `log_entry`, `sub_merchant`), `query`, `filters[...]`, `page` (default 1), `per_page` (default 30, max 100). Amounts in filters are main units (RM, not sen). Response: `object=search`, `data`, `page`, `per_page`, `total`, `total_pages`, `filters`, `query`, `scope`, `export`.

Charge scope filters: `amount`, `authorized`, `capture`, `captured`, `captured_at`, `credit_card_brand`, `card_last_digits`, `created`, `currency`, `disputed`, `failure_code`, `fraud`, `is_installment`, `refunded`, `refund_amount`, `scheduled`, `source_of_fund` (`card`, `offsite`, `offline`), `source_type`, `status`, `voided`, `installment_terms`. Charge query keys: `id`, `card_bank`, `card_name`, `description`, `failure_message`, `metadata`.

```bash
curl https://api.omise.co/search -X GET -u "$OMISE_SECRET_KEY:" \
  -d "scope=charge" -d "filters[status]=pending" -d "filters[source_type]=fpx"
```

Errors: `invalid_scope`, `invalid_filter`, `invalid_page`, `invalid_per_page`, `search_unavailable` (503).

## Account and capability

- `GET /account` (Account API reference heading shows `GET /account/api_versions`): `id`, `team`, `email`, `country`, `currency`, `supported_currencies`, `api_version`, `webhook_uri`, `zero_interest_installments`, `auto_activate_recipients`, `chain_enabled`, `chain_return_uri`, `chaining_allowed`, `metadata_export_keys`, `transfer_config`, `livemode`, `created_at`.
- `PATCH /account`: `webhook_uri`, `zero_interest_installments`, `chain_enabled`, `chain_return_uri`, `metadata_export_keys`. `PATCH /account/api_version` updates the version.
- `GET /capability` (public key works): `country`, `banks` (transfer banks), `payment_methods[]` (`name`, `currencies`, `card_brands`, `installment_terms`, `banks`), `limits` (`charge_amount.min/max`, `transfer_amount.min/max`, `installment_amount.min`), `tokenization_methods` (`googlepay`, `applepay`, `null`), `zero_interest_installments`.
- `GET /forex/{currency}`: `base`, `quote` (funding currency), `rate`. Not useful for Malaysian accounts since multi-currency is unavailable.
