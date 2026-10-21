# Omise API Reference (entry point)

Distilled from the official Omise docs (Malaysia view, `https://docs.omise.co/en/<page>/malaysia`) and the official OpenAPI schema at `https://api.omise.co/schema`, checked 2026-10-21. Scope: online payment acceptance for a merchant whose Omise account is registered in Malaysia (funding currency MYR). Omise traded as "Opn Payments" for a period; some pages still say Opn. Every endpoint is listed in the Endpoint catalogue at the end of this file.

Topic files:

- `omise-charges.md`: Charge object and every charge operation, card tokens, Omise.js, 3-D Secure, capture.
- `omise-malaysia-methods.md`: each Malaysian payment method, Source API, bank codes, limits, refund windows.
- `omise-webhooks.md`: signature verification, events, Events API, dynamic webhooks.
- `omise-refunds-disputes.md`: Refund API, voids, disputes, documents.
- `omise-recurring.md`: Customers, Cards, Schedules, Occurrences, charge schedule bulk APIs.
- `omise-payouts-reporting.md`: Balance, Transactions, Transfers, Recipients, Receipts, Search, Forex, Account, Capability.
- `omise-payment-links.md`: Payment Links+ API.
- `omise-testing-errors.md`: test mode, test cards, simulation, error codes.

## Source URLs

- Getting started: `https://docs.omise.co/en/api-reference/malaysia`
- Authentication: `https://docs.omise.co/en/api-authentication/malaysia`
- Versioning: `https://docs.omise.co/en/api-versioning/malaysia`
- Currencies and amounts: `https://docs.omise.co/en/currency-and-amount/malaysia`
- Pagination: `https://docs.omise.co/en/api-pagination/malaysia`
- Rate limiting: `https://docs.omise.co/en/api-rate-limiting/malaysia`
- IPs: `https://docs.omise.co/en/api-ips/malaysia`
- Errors: `https://docs.omise.co/en/api-errors/malaysia`
- Charge API: `https://docs.omise.co/en/charges-api/malaysia`
- Source API: `https://docs.omise.co/en/sources-api/malaysia`
- Webhooks: `https://docs.omise.co/en/api-webhooks/malaysia`
- OpenAPI schema: `https://api.omise.co/schema` (version `2019-05-29`)

## Environments

There is no separate sandbox host. Test mode and live mode use the same hosts; the key decides the mode.

| Host | Used for | Key |
| --- | --- | --- |
| `https://api.omise.co` | Everything except token creation | Secret key (sources also accept the public key) |
| `https://vault.omise.co` | `POST /tokens`, `GET /tokens/{id}` | Public key only |
| `https://linksplus-api.omise.co` | Payment Links+ API | Payment Links+ API key |
| `https://cdn.omise.co/omise.js` | Browser library | Public key |
| `https://dashboard.omise.co` | Dashboard (test and live modes) | Login |

- Test keys contain `_test_` (`pkey_test_...`, `skey_test_...`). Live keys are `pkey_...` / `skey_...` (the MCP page shows live secret keys as `skey_live_...`; check the dashboard).
- Object ids carry `_test_` in test mode (`chrg_test_...`) and every object has `livemode` (boolean).
- Creating a token on `api.omise.co` returns `invalid_domain`.

## Authentication

HTTP Basic auth. The key is the username; the password is blank.

```bash
curl https://api.omise.co/charges -u "$OMISE_SECRET_KEY:" -d "amount=10000" -d "currency=MYR" -d "card=$TOKEN_ID"
```

- Public key: create and view tokens and sources only. Safe in the browser.
- Secret key: all other calls. Never ship to clients. Roll it in the dashboard if exposed.
- Chain key: account chaining (sub-merchant) calls; treat as a secret.
- A wrong key type returns `401 authentication_failure`. A revoked key returns `403 key_expired_error`.

## Request format and versioning

- Request bodies are `application/x-www-form-urlencoded`. Nested fields use brackets: `source[type]=fpx`, `metadata[order_id]=123`, `on[weekdays][]=monday`.
- Responses are JSON. Errors return an `error` object (`object`, `location`, `code`, `message`) with a 4xx or 5xx status.
- API versions: `2019-05-29` (latest), `2017-11-02`, `2015-11-17`, `2014-07-27`. Test and live modes have independent account versions (dashboard > API Versions).
- Per request override: header `Omise-Version: 2019-05-29`.
- Webhook and Events API payloads are serialized with the account default version at the time of the event, not the request header.
- Most Malaysian payment methods need at least `2017-11-02`.
- Omise adds fields and event types without a version change. Ignore unknown fields and unknown event keys.
- Unsupported feature for the version: `404 serializer_not_found` or `service_not_found`.

## Amounts and currency

- `amount` is an integer in the smallest currency unit: `100` = RM1.00, `400000` = RM4,000.00. Non-integers return `invalid_amount`.
- Malaysia charge limits: min `100` (RM1.00), max `10000000` (RM100,000.00). Each method has its own tighter limits (see `omise-malaysia-methods.md`).
- Charges settle in the account funding currency, MYR for Malaysia. Multi-currency is not supported for Malaysian accounts.
- Malaysia transfer limits: min `100` (RM1.00), max `1000000000` (RM10,000,000.00).
- Search API filters use the main unit (`filters[amount]=20` means RM20), not sen.
- Currency is sent as ISO 4217 (`MYR`); API version 2019-05-29 returns upper case.

## Payment flow (all methods)

1. Create a local pending order (amount in sen, currency MYR).
2. Get a payment instrument:
   - Card: browser posts card data to `https://vault.omise.co/tokens` via Omise.js or a mobile SDK and gets `tokn_...` (single use).
   - Other methods: create a source `POST https://api.omise.co/sources` (public key from the browser, or secret key from the server) and get `src_...`. Or skip this and pass `source[type]=...` inline on the charge.
3. Server: `POST https://api.omise.co/charges` with `amount`, `currency`, `card` or `source` (or `customer`), `return_uri`, `description`, `ip`, `metadata`, and `ref_id`.
4. Read the charge:
   - `status=successful`: paid (non-3DS card with auto capture).
   - `status=pending` with `authorize_uri`: redirect the customer there (3DS, FPX, DuitNow, wallets, Atome, installments).
   - `status=pending` with `source.flow=offline` (Atome QR): show `source.scannable_code.image.download_uri`.
   - `status=failed`: show `failure_message`; see `failure_code`.
5. Customer returns to `return_uri`. Do not trust this. Show "processing" and check server-side.
6. Webhook `charge.complete` arrives. Verify signature, then `GET /charges/{id}` and settle.
7. Reconcile anything still pending (re-fetch or `GET /events`).

## Charge statuses

| `status` | Meaning |
| --- | --- |
| `pending` | Not yet authorized by the payer, or authorized and waiting for manual capture (`authorized=true`, `capture=false`) |
| `successful` | Paid (`authorized=true`, `paid=true`) |
| `failed` | Failed; see `failure_code`, `failure_message`, `merchant_advice` |
| `expired` | Payer did not complete in time, or uncaptured authorization lapsed |
| `reversed` | Authorization released by `POST /charges/{id}/reverse` or automatic release |

Source and token objects carry `charge_status` (one of the charge statuses, or `unknown` before a charge exists).

## Webhook signature (summary)

Full detail and code in `omise-webhooks.md`.

```text
headers:  Omise-Signature: <hex>[,<hex>]     Omise-Signature-Timestamp: <unix seconds>
signed  = Omise-Signature-Timestamp + "." + raw_request_body_utf8
key     = base64_decode(WEBHOOK_SECRET)
expect  = hex(HMAC_SHA256(key, signed))
valid   = any(constant_time_equal(expect, s) for s in Omise-Signature.split(","))
optional: reject if |now - timestamp| > 300 seconds
```

The header is present only when a webhook secret is configured for the account. Event verification (re-fetching the resource) is Omise's documented alternative, and you should do it anyway before fulfilment.

## Status query (requery)

- `GET /charges/{id}`: current charge. Use after every webhook and for reconciliation.
- `GET /charges/{id}/events`: events for one charge.
- `GET /events?from=...&order=reverse_chronological`: catch missed webhooks.
- `GET /search?scope=charge&filters[status]=pending`: find stuck charges.
- `GET /sources/{id}`: source with `charge_status`.

Omise discourages polling as a primary mechanism (rate limiting page) and does not guarantee webhook retries (webhooks FAQ), so poll only for reconciliation.

## Idempotency

- Charge create accepts `ref_id`: your idempotency reference. Each `ref_id` can be used once per account, which stops a retried request from creating a second charge. Omise does not generate it. Returned on the charge only if sent.
- Refund create and transfer create also list `ref_id` / `idemp_key` parameters (descriptions are "TBD" in the docs).
- Webhook handling must be idempotent on your side (unique index on charge id, event id log).

## Lists and pagination

List endpoints take `from`, `to` (ISO 8601 UTC), `offset` (default 0), `limit` (default 20, max 100), `order` (`chronological` default, or `reverse_chronological`). Response: `object=list`, `from`, `to`, `offset`, `limit`, `total`, `data[]`, `location`. Search uses `page` and `per_page` (default 30, max 100) instead.

## Rate limits

- Limits are by IP and by account, with short bursts allowed. No numeric limit is published.
- Token creation on the vault has a much lower limit than the main API.
- Priority: live POST/PUT, then live GET, then all test mode calls. Test mode performance differs from live; do not load test the API.
- Exceeding the limit returns `429 too_many_requests`.
- Advice: use webhooks instead of polling, spread requests, use no more than 10 parallel workers, and contact Omise before big events.

## IP allowlists

- `api.omise.co` may resolve to `76.223.19.23` or `13.248.150.88`.
- Webhooks come from `54.169.118.227`, `52.74.199.175`, `18.139.13.19`.
- Machine readable: `https://cdn.omise.co/ips.json`.
- Requests from an IP on a third-party reputation blocklist can get `403 Forbidden` on charge creation; Omise suggests a static IP for servers with dynamic IPs (fraud protection page).

## TLS and libraries

- HTTPS with TLS 1.2 or later is required for checkout pages and webhook endpoints; self-signed certificates are rejected for webhooks.
- `api.omise.co` chains to DigiCert Global Root G2. Old libraries pinned certificates; minimum versions: Android 3.0.0, Go 1.0.5, iOS 3.2.0, Java 3.1.1, PHP 3.0.0, Python 0.9.0, Ruby 0.8.0, EC-CUBE 2.2, Magento 2.18.6, OpenCart 2.5, PrestaShop 1.7.10, WooCommerce 4.8.
- Official server libraries: Ruby, Node.js, PHP, Python, Go, Java, .NET, Elixir, Dart (all on `github.com/omise`). Client: Omise.js, iOS SDK, Android SDK, Flutter, Dart.

## Endpoint catalogue

Every endpoint documented on docs.omise.co (Malaysia view), plus Payment Links+ and charge-schedule bulk endpoints, plus operations that appear only in the OpenAPI schema. Base `https://api.omise.co` unless stated.

### Charges (`omise-charges.md`)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/charges` | Create a charge (card token, customer card, or source) |
| GET | `/charges` | List charges |
| GET | `/charges/{id}` | Retrieve a charge |
| PATCH | `/charges/{id}` | Update `description` or `metadata` |
| POST | `/charges/{id}/capture` | Capture an authorized charge (`capture=false`) |
| POST | `/charges/{id}/reverse` | Release an uncaptured authorization |
| POST | `/charges/{id}/expire` | Expire a pending charge (supported sources only, includes `touch_n_go`; ShopeePay and Atome pages also use it) |
| POST | `/charges/{id}/mark_as_paid` | Test mode only: mark paid |
| POST | `/charges/{id}/mark_as_failed` | Test mode only: mark failed |
| GET | `/charges/{id}/events` | List events for a charge |
| GET | `/links/{id}/charges` | List charges for a (classic) link |

### Tokens and sources

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `https://vault.omise.co/tokens` | Create a card token (public key only) |
| GET | `https://vault.omise.co/tokens/{id}` | Retrieve a token (public key only) |
| POST | `/sources` | Create a source (public or secret key) |
| GET | `/sources/{id}` | Retrieve a source |

### Refunds and disputes (`omise-refunds-disputes.md`)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/charges/{id}/refunds` | Create a refund (or void) |
| GET | `/charges/{id}/refunds` | List refunds for a charge |
| GET | `/charges/{id}/refunds/{refund_id}` | Retrieve a refund |
| GET | `/refunds` | List all refunds |
| GET | `/disputes` | List disputes |
| GET | `/disputes/open` | List open disputes |
| GET | `/disputes/pending` | List pending disputes |
| GET | `/disputes/closed` | List closed disputes (`won`, `lost`) |
| GET | `/disputes/{id}` | Retrieve a dispute |
| PATCH | `/disputes/{id}` | Update dispute `message`, `metadata` |
| PATCH | `/disputes/{id}/accept` | Accept a dispute |
| PATCH | `/disputes/{id}/close` | Close an `open` or `pending` dispute as `won` or `lost` |
| POST | `/charges/{id}/disputes` | Create a dispute for a charge (the support article describes opening disputes in test mode from the dashboard) |
| POST | `/disputes/{id}/documents` | Upload evidence (PNG, JPG, PDF) |
| GET | `/disputes/{id}/documents` | List documents |
| GET | `/disputes/{id}/documents/{document_id}` | Retrieve a document |
| DELETE | `/disputes/{id}/documents/{document_id}` | Delete a document |

### Events (`omise-webhooks.md`)

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/events` | List events |
| GET | `/events/{id}` | Retrieve an event |

### Customers, cards, schedules (`omise-recurring.md`)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/customers` | Create a customer (optionally with a card token) |
| GET | `/customers` | List customers |
| GET | `/customers/{id}` | Retrieve a customer |
| PATCH | `/customers/{id}` | Update a customer, add a card, set default card |
| DELETE | `/customers/{id}` | Delete a customer |
| GET | `/customers/{id}/cards` | List a customer's cards |
| GET | `/customers/{id}/cards/{card_id}` | Retrieve a card |
| PATCH | `/customers/{id}/cards/{card_id}` | Update a card |
| DELETE | `/customers/{id}/cards/{card_id}` | Delete a card |
| POST | `/schedules` | Create a charge or transfer schedule |
| GET | `/schedules` | List schedules |
| GET | `/schedules/{id}` | Retrieve a schedule |
| DELETE | `/schedules/{id}` | Delete a schedule |
| GET | `/schedules/{id}/occurrences` | List occurrences |
| GET | `/occurrences/{id}` | Retrieve an occurrence |
| GET | `/charges/schedules` | List charge schedules |
| GET | `/customers/{id}/schedules` | List charge schedules for a customer |
| GET | `/transfers/schedules` | List transfer schedules |
| GET | `/recipients/{id}/schedules` | List transfer schedules for a recipient |
| POST | `/schedules/upload` | Bulk create charge schedules from CSV (multipart) |
| GET | `/recurring_exports/{recurring_id}` | Monitor a bulk creation job |
| GET | `/recurring_exports/{recurring_id}/download` | Download bulk creation status |
| PATCH | `/schedules/bulk_pause` | Pause charge schedules (`schedule_ids`) |
| PATCH | `/schedules/bulk_resume` | Resume charge schedules |
| DELETE | `/schedules/bulk_delete` | Delete charge schedules |

### Balance, transfers, recipients, reporting (`omise-payouts-reporting.md`)

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/balance` | Retrieve balance (`total`, `transferable`, `reserve`, `on_hold`) |
| GET | `/transactions` | List balance transactions |
| GET | `/transactions/{id}` | Retrieve a transaction |
| POST | `/transfers` | Create a payout transfer |
| GET | `/transfers` | List transfers |
| GET | `/transfers/{id}` | Retrieve a transfer |
| PATCH | `/transfers/{id}` | Transfer callback (`success`, `failure_code`, `failure_message`) |
| DELETE | `/transfers/{id}` | Delete a transfer not yet sent |
| POST | `/transfers/{id}/mark_as_sent` | Test mode only: mark sent |
| POST | `/transfers/{id}/mark_as_paid` | Test mode only: mark paid |
| POST | `/recipients` | Create a recipient |
| GET | `/recipients` | List recipients |
| GET | `/recipients/{id}` | Retrieve a recipient |
| PATCH | `/recipients/{id}` | Update a recipient |
| DELETE | `/recipients/{id}` | Delete a recipient |
| PATCH | `/recipients/{id}/verify` | Test mode: mark a recipient verified |
| GET | `/receipts` | List Omise fee receipts |
| GET | `/receipts/{id}` | Retrieve a receipt |
| GET | `/search` | Search (`scope` required) |
| GET | `/forex/{currency}` | Exchange rate to funding currency |
| GET | `/capability` | Account capabilities: methods, FPX banks, limits (public key works) |
| GET | `/account` | Retrieve account (docs reference lists `GET /account/api_versions`) |
| PATCH | `/account` | Update `webhook_uri`, chaining, `zero_interest_installments`, export keys |
| PATCH | `/account/api_version` | Update account API version |

### Payment Links+ (`omise-payment-links.md`), base `https://linksplus-api.omise.co`

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/external/templates/{template_id}` | Retrieve a template |
| GET | `/external/{team_id}/templates` | List templates for a team |
| GET | `/external/links/{link_id}` | Retrieve a link |
| GET | `/external/{team_id}/links` | List links for a team |
| POST | `/external/links` | Create a payment link |
| PUT | `/external/links/{link_id}` | Update a link |
| POST | `/external/links/{link_id}/disable` | Disable a link |

### Thailand-only direct debit (out of scope for Malaysia)

`POST /linked_accounts`, `GET /linked_accounts/{id}`, `DELETE /linked_accounts/{id}` (Online Direct Debit, supported country Thailand).

### In the OpenAPI schema but not documented on docs.omise.co

Do not build on these without confirming with Omise support. Listed for completeness: `GET /` (system info), `GET /schema`, `GET /audits/search`, `GET /chains`, `GET /chains/{id}`, `GET /chains/{id}/keys`, `GET /chains/{id}/account`, `POST /chains/{id}/authorize`, `POST /chains/{id}/revoke`, `GET /charges/search`, `GET /charges/schedules/search`, `GET /customers/search`, `GET /disputes/search`, `GET /refunds/search`, `GET /recipients/search`, `GET /transfers/search`, `GET /transfers/schedules/search`, `GET /receipts/receipts/search`, `GET /links`, `POST /links`, `GET /links/{id}`, `DELETE /links/{id}`, `GET /links/search`, `GET /linked_accounts`, `GET /linked_accounts/search`, `PATCH /consent`, `POST /mfa_confirmations`, `GET /routing_group_rules`, `GET /sub_merchants`, `POST /sub_merchants`, `GET /sub_merchants/search`, `GET /webhooks/secrets`, `POST /webhooks/secrets`, `GET /webhooks/secrets/{id}`, `DELETE /webhooks/secrets/{id}`. Webhook secrets are documented only as a dashboard feature (Webhooks Settings).
