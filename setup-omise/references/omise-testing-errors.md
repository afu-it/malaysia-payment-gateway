# Omise Testing, Errors and Failure Codes

Checked 2026-10-21. All test card numbers below are public sandbox data from Omise docs; they work only with test keys, with any future expiry date and any security code.

## Source URLs

- Testing: `https://docs.omise.co/en/api-testing/malaysia`
- Errors: `https://docs.omise.co/en/api-errors/malaysia`
- Error object: `https://docs.omise.co/en/errors-api/malaysia`
- Charge API (failure codes): `https://docs.omise.co/en/charges-api/malaysia`
- Merchant advice: `https://docs.omise.co/en/merchant-advice/malaysia`
- Collecting cards (token errors FAQ): `https://docs.omise.co/en/collecting-card-information/malaysia`

## Test mode

- Every account has test mode. Use `pkey_test_...` / `skey_test_...`; the same hosts serve both modes.
- Logic must be identical in test and live; only keys differ.
- Non-card methods: open the charge's `authorize_uri` and choose Successful or Failed, or in the dashboard go to Charges > charge > yellow Actions button > Mark as Successful / Mark as Failed. API equivalents: `POST /charges/{id}/mark_as_paid` and `/mark_as_failed` (not all methods).
- Atome QR (offline): dashboard Actions only.
- 3DS tests need 3DS enabled on the test account (email support@omise.co). Regular success cards then go through a mock authentication page.
- Transfers: dashboard Mark as sent / Mark as paid, or the API endpoints. Recipients: `PATCH /recipients/{id}/verify`.
- Disputes: open any captured test charge in the dashboard and create a dispute.
- Schedules: max 10 active scheduled charges in test mode.
- Test mode has different performance and lower priority; do not load test. Keep concurrency at 5 to 10.

## Successful test cards

| Brand | Number |
| --- | --- |
| Visa | 4242 4242 4242 4242 |
| Visa | 4111 1111 1111 1111 |
| Mastercard | 5555 5555 5555 4444 |
| Mastercard | 5454 5454 5454 5454 |
| JCB | 3530 1113 3330 0000 |
| JCB | 3566 1111 1111 1113 |
| Amex | 3782 8224 6310 005 |
| Discover | 6447 0324 0000 0000 001 |
| Diners | 3602 1042 0000 04 |
| UnionPay | 6250 9470 0000 0006 |

## Failing test cards

| `failure_code` | Visa | Mastercard | JCB | UnionPay | Other |
| --- | --- | --- | --- | --- | --- |
| `insufficient_fund` | 4111 1111 1114 0011 | 5555 5511 1111 0011 | 3530 1111 1119 0011 | 6250 9470 0000 0014 | |
| `stolen_or_lost_card` | 4111 1111 1113 0012 | 5555 5511 1110 0012 | 3530 1111 1118 0012 | 6250 9470 0000 0022 | |
| `failed_processing` | 4111 1111 1112 0013 | 5555 5511 1119 0013 | 3530 1111 1117 0013 | 6250 9470 0000 0030 | Amex 3771 3816 0000 009, Discover 6447 0324 0000 1111 005, Diners 3602 1042 1111 08 |
| `payment_rejected` | 4111 1111 1111 0014 | 5555 5511 1118 0014 | 3530 1111 1116 0014 | 6250 9470 0000 0048 | |
| `failed_fraud_check` | 4111 1111 1119 0016 | 5555 5511 1116 0016 | 3530 1111 1114 0016 | 6250 9470 0000 0055 | |
| `invalid_account_number` | 4111 1111 1118 0017 | 5555 5511 1115 0017 | 3530 1111 1113 0017 | 6250 9470 0000 0063 | |

## Failing 3DS test cards (result `payment_rejected`)

| Type | Visa | Mastercard | JCB | UnionPay | Other |
| --- | --- | --- | --- | --- | --- |
| Enrollment failure (setup problem, for example no `return_uri`; fails before authentication) | 4111 1111 1115 0002 | 5555 5511 1112 0002 | 3530 1111 1110 0002 | 6250 9470 0000 0071 | |
| Validation failure (wrong OTP or timeout) | 4111 1111 1114 0003 | 5555 5511 1111 0003 | 3530 1111 1119 0003 | 6250 9470 0000 0089 | Amex 3771 3816 1111 003, Discover 6447 0324 0000 2222 009, Diners 3602 1042 2222 02 |

Merchant advice test card: Visa 4111 1111 1117 0018; the amount picks the `merchant_advice_code` (for example `9132`).

Google Pay test environment: token card is always 4111 1111 1111 1111.

## Charge failure codes

| Code | Where documented | Meaning |
| --- | --- | --- |
| `confirmed_amount_mismatch` | Charge API | Final channel amount differs from the charge |
| `failed_fraud_check` | Charge API | Marked fraudulent |
| `failed_processing` | Charge API, all methods | General failure |
| `insufficient_balance`, `insufficient_fund` | Charge API, methods | Not enough funds or limit reached |
| `invalid_account_number`, `invalid_account` | Charge API, methods | No valid account |
| `payment_cancelled` | Charge API, TnG, ShopeePay | Payer cancelled |
| `payment_expired` | DuitNow, wallets, Atome | Payment window expired |
| `payment_rejected` | Charge API, methods, 3DS | Issuer rejected |
| `expired_card` | Charge API | Card expired |
| `stolen_or_lost_card` | Charge API | Stolen or lost |
| `timeout` | Charge API | Provider timeout |

## API errors (`object=error`, `code`, `message`, `location`)

| HTTP | `code` | Meaning |
| --- | --- | --- |
| 400 | `backend_error` | Provider error for this method; retry |
| 400 | `bad_request` | Invalid request; read `message` |
| 400 | `brand_not_supported` | Card brand not enabled |
| 400 | `documents_locked` | Dispute no longer open |
| 400 | `expired_charge` | Capture or reverse after expiry (cards uncaptured more than 7 days, 30 for Japan), or completing an expired charge |
| 400 | `failed_capture` | Not authorized yet or already captured |
| 400 | `failed_expire` | Could not expire |
| 400 | `failed_fraud_check` | Token creation on a card flagged as fraud |
| 400 | `failed_multi_currency` | Multi-currency not enabled |
| 400 | `failed_refund` | Refund failed |
| 400 | `failed_reverse` | Reverse failed |
| 400 | `failed_void` | Void failed |
| 400 | `feature_not_supported` | Feature not enabled for the account |
| 400 | `invalid_amount` | Non-integer amount |
| 400 | `invalid_bank_account` | Bad recipient bank account |
| 400 | `invalid_card` | Invalid token parameters |
| 400 | `invalid_card_token` | Non-string passed where a token id is expected |
| 400 | `invalid_charge` | Charge does not meet minimum requirements |
| 400 | `invalid_dispute` | Invalid dispute change |
| 400 | `invalid_domain` | Token request sent to api.omise.co instead of vault.omise.co |
| 400 | `invalid_file_type` | Wrong document type |
| 400 | `invalid_link` | Bad link currency or no description |
| 400 | `invalid_recipient` | Invalid recipient change or unknown recipient |
| 400 | `invalid_transfer` | Invalid transfer (amount over balance, inactive recipient, and so on) |
| 400 | `missing_card` | No card parameters |
| 400 | `missing_file` | No file uploaded |
| 400 | `used_token` | Token already used |
| 401 | `authentication_failure` | Invalid key or wrong key type |
| 403 | `key_expired_error` | Revoked key |
| 403 | `locked_account_error` | Account locked |
| 403 | `not_authorized` | Action not allowed |
| 404 | `not_found` | Unknown object id |
| 404 | `serializer_not_found` | Not supported by the request API version |
| 404 | `service_not_found` | Operation not supported by the API version |
| 422 | `failed_deletion` | Delete failed |
| 422 | `invalid_filter`, `invalid_page`, `invalid_per_page`, `invalid_scope` | Bad search parameters |
| 429 | `too_many_requests` | Rate limited |
| 500 | `internal_error` | Internal error |
| 503 | `search_unavailable` | Search temporarily unavailable |

Testing page extras: `missing_return_uri` (3DS charge without `return_uri`), `webhook_delivery_failed` (endpoint unreachable). Collecting-cards FAQ lists token errors `invalid_card`, `expired_card`, `invalid_security_code`, `authentication_failure`, `service_not_found`.

## Common troubleshooting

- Test charges failing: live keys used, wrong card number, missing `amount`/`currency`/`card`, amount not in sen.
- Webhooks missing: URL not public, not POST, invalid certificate, firewall blocking Omise IPs.
- 3DS not redirecting: 3DS not enabled on the test account, missing or unreachable `return_uri`.
- Token creation failing: wrong public key (must start `pkey_test_` in test), bad card format, browser console errors.
- TLS errors (`certificate verify failed`, `PKIX path building failed`, `x509: certificate signed by unknown authority`): update the Omise library to the minimum version and trust DigiCert Global Root G2.
- Intermittent `403 Forbidden` on charge creation from cloud servers: possible IP reputation block; use a static IP.
