# Betterpay Card Tokenization And Pre-authorization Reference

Checked: 2026-10-21.

Sources:

- Docs `https://www.betterpay.me/docs/#mpgs_tokenization1` (PRO tokenization)
- Docs `#batch_mpgs_tokenization1` (PRO pre-auth by token and batch charge)
- Docs `#mpgs_preauth1` (Card Pre-authorization)
- Docs `#tokenization1` and `#batch_tokenization1` (LITE)
- Postman `Tokenization (BP PRO)`, `Tokenization (BP LITE)`, `BP PRO APIs - Card Pre-Authentication` at `https://github.com/betterpayme/POSTMAN-APIs`

Card data is entered on the MPGS (Mastercard Payment Gateway Services) hosted page. Your server never sees the card number, which reduces your PCI scope. Only VISA and Mastercard can be charged by token.

**Tokenization is not enabled by default.** Email `support@qlicknpay.com` to enable it (the docs address). The newer support address in the WP plugin readme is `support@betterpay.me`.

## 1. PRO Tokenization (MPGS)

Flow:

1. Create a Verification URL.
2. Redirect the payer to it.
3. Betterpay emails you and POSTs the token to your callback.
4. Charge the card with the token.
5. Wait 5 minutes between successful charges on the same token.
6. Delete the token, then reuse the same Verification URL to change the card.

| Operation | Sandbox path (Postman) | MD5 hash string |
|---|---|---|
| Create Verification URL | `/merchant/api/cards/token/v1/create` | `api\|merchant_id\|reference_id`. With `amount`: `api\|merchant_id\|reference_id\|amount\|invoice` |
| Verification callback (verify) | (your URL) | `api + reference_id + txn_status + msg + token` (no separators) |
| Charge Card by Token | `/merchant/api/cards/token/v1/pay/{token_id}` | `api\|merchant_id\|token\|invoice\|amount` |
| Delete Card | `/merchant/api/cards/token/v1/delete_token/{reference_id or token_id}` | `api\|merchant_id\|reference_id` (Postman hashes the token ID; hash the ID you put in the URL) |
| Get Available Token | `/merchant/api/cards/token/v1/get_token` | `api\|merchant_id` |
| Get Token Details | `/merchant/api/cards/token/v1/get_token_details/{reference_id}` | `api\|merchant_id\|reference_id` |
| Get Transactions | `/merchant/api/cards/token/v1/get_transaction` | `api\|merchant_id` |
| Get Transaction Details | `/merchant/api/cards/token/v1/get_transaction_details/{invoice}` | `api\|merchant_id\|invoice` |
| Authorize Card (token pre-auth) | Not published, URL param `{Token ID}` | `api\|merchant_id\|token\|invoice\|max_amount` (the PHP sample names the 5th value `$amount`, the table says Max Amount) |
| Capture Card | Not published, URL param `{Token ID}` | `api\|merchant_id\|token\|invoice\|amount` |
| Void Authorized Card | Not published, URL param `{Token ID}` | `api\|merchant_id\|token\|invoice` |
| Batch Charge Card | Not published | `api\|merchant_id\|batch_id\|token_list` (PHP sample names the 3rd value `$reference_id`) |
| Batch callback (verify) | (your URL) | Docs code: `md5(api.merchant_id.batch_id.token_list)`. Docs example string uses pipes: `API\|MID\|BATCH\|token_list`. Test both in sandbox |
| Cancel Charge Card by Batch | Not published | `api\|merchant_id\|batch_id` |
| Get Batch | Not published | `api\|merchant_id` |
| Get Batch Details | Not published | `api\|merchant_id\|batch_id` |

### Create Verification URL

| Field | Required | Notes |
|---|---|---|
| `merchant_id` | yes | |
| `reference_id` | yes | Unique, alphanumeric, max 20, no `&`, `'` or `-` |
| `hash` | yes | |
| `amount` | no | First charge. Omit for verify-only. With 3DS, a RM1.00 verification amount is used and not charged |
| `invoice` | if `amount` | Max 20 |
| `callback_url_be` | no | Host must match API Management |
| `callback_url_fe_succ`, `callback_url_fe_fail` | no | Browser return URLs |
| `skip_receipt` | no | `0`/`1` |
| `baggage_variable` + fields | no | Echoed back in the callback, each max 100 chars |
| `email_notification` | no | Payer gets the URL again if verification fails |

Response: `response`, `reference_id`, `callback_url_be`, `success_url`, `fail_url`, `skip_receipt`, `credit_card_verification_url`, `created`, `modified`.

Verification callback fields:

- `reference_id`, `txn_status`, `msg` (e.g. `Verification Approved`)
- `token`: store it securely
- `hash`
- Card details: `nameOnCard`, `numberCard` (masked), `schemeCard`, `issuerCard`, `expiryMonthCard`, `expiryYearCard`, `fundingMethodCard`
- `amount`, `invoice` (first payment)

### Charge Card by Token

- Request: URL `{token_id}`, `merchant_id`, `invoice` (unique, max 20), `amount` (2dp, at least 1.50), `hash`.
- Sandbox only: `sandbox_charge_status` (`0` succeeds, anything else fails).

Response:

| Field | Meaning |
|---|---|
| `response` | API result, not payment status. `00` means the call worked |
| `txn_status` | `00` success, `12` failed |
| `msg` | Status message |
| `reason` | MPGS reason text, empty on success |
| `error_code` | MPGS error code, empty on success |
| `invoice`, `amount`, `token` | Echo of the charge |
| `transaction_authorizationCode` | Issuer authorization code |

### Lists and details

- **Delete Card** response: `response`, `msg`, `reference_id`.
- **Get Available Token**:
  - Optional: `date_from`, `date_to`, `status` (0 failed, 1 success, 2 both), `page`.
  - Rows: `reference_id`, `verification_status`, `token`, `credit_card_verification_url`, `created`, `modified`.
  - Plus `total_page_available` (100 per page).
- **Get Token Details**: `response`, `reference_id`, `verification_status`, `token`, `callback_url_be`, `callback_url_fe_succ`, `callback_url_fe_fail`, `skip_receipt`, `credit_card_verification_url`, `credit_card_details`, `created`, `modified`.
- **Get Transactions**:
  - Optional: `date_from`, `date_to`, `status`, `page`.
  - Rows: `reference_id`, `invoice`, `amount`, `txn_status`, `status`, `time`.
- **Get Transaction Details**: `response`, `reference_id`, `invoice`, `amount`, `txn_status` (`00` success, `12` failed, `09` pending), `status`, `reason`, `error_code`, `credit_card_details`, `authorization_code`, `time`.
- **MPGS Error Code** table: published as a JSON file linked from the docs (`./error_code_mpgs/error.json` relative to the docs page).

### Pre-auth by token (Authorize, Capture, Void)

- An authorization expires 7 days after it is created and is voided automatically if not captured.
- **Authorize**: URL `{token_id}`, `merchant_id`, `invoice` (max 20), `max_amount` (2dp, at least 1.50), `hash`. Optional: `sandbox_charge_status`.
  - Response: `response`, `txn_status` (`00` authorized, `12` failed), `msg`, `reason`, `error_code`, `invoice`, `max_amount`, `token`.
- **Capture**: URL `{token_id}`, `merchant_id`, `invoice`, `amount`, `hash`. Optional: `sandbox_charge_status`.
  - Response: as for Charge Card.
- **Void**: URL `{token_id}`, `merchant_id`, `invoice`, `hash`.
  - Response: `response`, `txn_status` (`00` voided, `12` failed), `msg`, `invoice`, `amount`, `token`.
- If the pre-auth is under USD 2.00 (about RM10), do not void within 48 hours. A USD 0.50 penalty per transaction applies.

### Batch Charge Card

- Request: `merchant_id`, `token_list`, `batch_id` (max 20), `callback_url_be`, `hash`.
- `token_list` is `token,invoice,amount;token,invoice,amount;`, with fewer than 500 tokens.
- Callback: `batch_id`, `token_list` as `token,invoice,amount,status,error_code;` (e.g. `...,INV01,41.00,12,51;...,INV02,11.00,00,;`), `hash`.
- Sandbox: trigger the callback by hand in Merchant account > MPGS Tokenization > Batch Token Approval.
- **Cancel by batch**: `merchant_id`, `batch_id`, `hash`. Only within 10 minutes of submission.
- **Get Batch**: optional `date_from`, `date_to`, `status` (0 failed, 1 success, 2 both, 3 failed/success/pending), `page`.
- **Get Batch Details**: `merchant_id`, `batch_id`, `hash`.

## 2. Card Pre-authorization (PRO, MPGS hosted)

Holds funds on a card with no stored token, then captures later.

Flow:

1. Create a Pre-authorization URL.
2. The payer verifies the card on MPGS.
3. Betterpay sends a callback and an email.
4. Capture (Pay) by invoice, or Void.

Paths come from Postman, prefixed with the collection variable `{{API_URL}}`. The variable is empty in the published collection: use the environment host (sandbox `https://www.demo.betterpay.me`) and confirm.

| Operation | Path | MD5 hash string |
|---|---|---|
| Create Pre-authorization URL | `/merchant/api/cards/auth/v1/create` | `api\|merchant_id\|invoice\|max_amount\|payment_desc\|currency\|expiry_date` |
| Callback (verify) | (your URL) | `api + mastercard_trx_id + invoice_no + txn_status + msg` (no separators) |
| Capture / Pay | `/merchant/api/cards/auth/v1/pay` | `api\|merchant_id\|invoice\|amount\|currency` |
| Void | `/merchant/api/cards/auth/v1/void` | `api\|merchant_id\|invoice` |
| Get list of transactions | `/merchant/api/cards/auth/v1/get_transactions` | `api\|merchant_id` |

Create request:

- Required: `merchant_id`, `invoice` (max 14), `max_amount` (2dp, at least 1.50; the USD 2.00 / 48h void rule applies), `payment_desc` (max 1000), `expiry_date` (YYYY-MM-DD), `currency` (`MYR`, `SGD`, `IDR`, `THB` or `PHP`), `hash`.
- Optional: `buyer_name`, `buyer_email`, `phone`, `add_line_1`, `add_line_2`, `postcode`, `city`, `state`, `comment`, `callback_url_be`, `callback_url_fe_succ`, `callback_url_fe_fail`, `baggage_variable` + fields, `header_title`, `header_email`, `header_phone`. The Postman body names these `checkout_header_title` / `_email` / `_phone`: confirm which one is accepted.

Create response: the request echo plus `response` (`00`) and `url` (send the payer here, e.g. `https://www.qlicknpay.com/s?cc-auth=...`).

Callback: `mastercard_trx_id`, `invoice_no`, `txn_status` (`00` authorized, `76` cancel, `1C` void), `msg`, `txn_amount` (the max chargeable amount), `hash`. Reply `OK`.

Capture / Pay:

- Request: `merchant_id`, `invoice` (from create), `amount` (at least 1.50 and at most `max_amount`), `currency`, `hash`.
- Sandbox only: `simulation_status` (`SUCCESS` or `FAIL`, default `FAIL`).
- Response: `response`, `txn_status`, `msg`, `reason`, `error_code`, `invoice`, `amount`, `authorization_code`.

Void:

- Request: `merchant_id`, `invoice`, `hash`. Sandbox: `simulation_status`.
- Response: `response`, `txn_status`, `msg` (e.g. `Payment Voided Successfully`), `reason`, `error_code`, `invoice`, `authorization_code`.

Get list:

- Optional: `date_from`, `date_to`, `invoice`, `page` (100 per page).
- Rows: `invoice`, `reference_no`, `max_amount`, `currency`, `paid_amount`, `status` (e.g. `PENDING`, `PAID`, `VOID`), `expiry_date`, `url`, `updated_at`, `created_at`.

The docs list two endpoints: "Production (3DS)" and "Sandbox". Both are behind login.

## 3. LITE Tokenization

The LITE docs describe the same flow and fields as PRO. Differences:

- Charge and Delete send the token in the body (`token`), not in the URL.
- The LITE docs give no per-endpoint hash table. The LITE Hashing Guide (HMAC-SHA256, sorted values) applies.
- The official LITE Postman collection uses HMAC for create, pay, delete and get_token, but MD5 `api|mid|id` for get_token_details, get_transaction and get_transaction_details. Confirm with support.

| Operation | Sandbox path (Postman LITE) | Body |
|---|---|---|
| Create Verification URL | `/merchant/api/cards/token/v1/create` | `merchant_id`, `reference_id`, optional `amount`, `invoice`, callbacks, `skip_receipt`, `hash` |
| Charge Card | `/merchant/api/cards/token/v1/pay/sandbox` | `merchant_id`, `token`, `invoice` (max 20), `amount` (at least 1.50), `hash`, sandbox `sandbox_charge_status` |
| Delete Card | `/merchant/api/cards/token/v1/delete_token/sandbox` | `merchant_id`, `token` (reference ID max 20 or token ID max 64), `hash` |
| Get Available Token | `/merchant/api/cards/token/v1/get_token` | `merchant_id`, `hash` |
| Get Token Details | `/merchant/api/cards/token/v1/get_token_details/{refID}` | `merchant_id`, `hash` |
| Get Transactions | `/merchant/api/cards/token/v1/get_transaction` | `merchant_id`, `hash` |
| Get Transaction Details | `/merchant/api/cards/token/v1/get_transaction_details/{invoice}` | `merchant_id`, `hash` |
| Batch Charge, Cancel Batch, Get Batch, Get Batch Details | Not published | Same fields as PRO |

The `/sandbox` suffix on the LITE pay and delete paths looks sandbox-specific. The production paths are not published (checked 2026-10-21).

The LITE create response and callback fields match PRO: `credit_card_verification_url`; callback `reference_id`, `txn_status`, `msg`, `token`, `hash`, `schemeCard`, `fundingMethodCard`, `amount`.
