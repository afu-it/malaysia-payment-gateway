# Tulus Partner and Customisation APIs

These are not needed for a normal merchant checkout. Use them only when a partner pushes transactions into Tulus, or when a merchant customises receipts or receipt emails. Sources: `https://developers.tulus.my/adb/`, `/grab/`, `/receipt/`, `/customemail/`, `/schema/adb.json`, `/schema/grab.json` (checked 2026-10-21). Credentials for all of these are issued by Tulus (support@tulus.my).

## ADB Webhook (partner pushes zakat transactions)

- `POST https://pay.tulus.my/api/adb/webhook`, `Content-Type: application/json`.
- Auth header: `X-API-Password: <API_PASSWORD>`.
- Agency codes: `ADBLZS` (to LZS), `ADBPPZ` (to PPZ-MAIWP), `ADB` (live test entries).
- Schema: `https://developers.tulus.my/schema/adb.json`.

Body: `{"$schema"?: string, "transactions": [ ... ]}`.

| Transaction field | Type | Required | Notes |
|---|---|---|---|
| `agency` | string | yes | `ADBLZS`, `ADBPPZ`, `ADB` |
| `orderno` | string | yes | Unique per transaction (duplicates prevented) |
| `amount` | number | yes | RM, `>= 0` |
| `name` | string | yes | Customer name |
| `email` | string | yes | Valid email |
| `time` | string | yes | ISO 8601 with offset, for example `2025-01-01T10:10:10+08:00` |
| `status` | string | yes | `success`, `pending`, `failed` |
| `data` | object | yes | See below |

`data` (all optional): `email`, `zakat_name`, `zakat_ic_no` (12 digits), `zakat_state` (enum in `tulus-codes.md`), `zakat_type` (enum in `tulus-codes.md`), `zakat_year` (4 digits), `zakat_phone` (10 to 12 digits, no spaces), `zakat_address`, `zakat_postcode` (5 digits), `zakat_city`, `zakat_amount` (numeric string, up to 2 decimals), `zakat_rate` and `zakat_dependant` (fitrah only; amount should equal rate times dependants).

Responses: 200 processed, 400 invalid data or schema failure, 401 invalid or missing password, 500 server error.

## Grab Webhook (draft spec, several items marked TBD by Tulus)

- `POST https://pay.tulus.my/api/grab/webhook` and `POST https://pay.tulus.my/api/grab/sync` (connectivity check, body `{}`), JSON.
- Auth: `X-API-Password` (or `X-Notification-Secret` if the first is absent). Webhook and sync use different secrets. A bad secret returns 400 `{"status":"invalid secret"}`.
- Schema: `https://developers.tulus.my/schema/grab.json`.

| Transaction field | Type | Required | Notes |
|---|---|---|---|
| `agency` | string | yes | `GRAB` (auto-classified), proposed `GRABLZS`, `GRABPPZ`, `GRABFLZS`, `GRABFPPZ` |
| `orderno` | string | yes | Max 255, unique across all Grab collections, identical on retry (dedup key) |
| `amount` | number or string | yes | MYR, min `1.00`, max 2 decimals, no exponent |
| `name` | string | yes | Max 255 |
| `email` | string | yes | Max 255, receipt address |
| `time` | string | yes | ISO 8601 with offset (minute or second precision) |
| `status` | string | yes | `success`/`SUCCESS`, `failed`/`failure`/`FAILURE` |
| `category` | string | no | `zakat`, `donation`, `bill_payment` |
| `data` | object | yes | `collection_type`, `collection_state`, `ic_no` (required, max 20), `grab_order_id`, `grab_merchant_id`, `phone`, `address`, `postcode`, `city` (keys case-insensitive, underscores optional) |

Test signal: payer email containing `test` on `@grab.com` gets an `X` agency prefix (for example `XGRABLZS`).

| HTTP | Body | Meaning / retry |
|---|---|---|
| 200 | `{"status":"ok"}` | All recorded |
| 400 | `{"status":"invalid secret"}` | Final |
| 400 | `{"status":"invalid request"}` | Final; whole batch rejected (for example missing `data.ic_no`) |
| 405 | `{"status":"method not allowed"}` | Final |
| 409 | `{"status":"conflict"}` | Tried to change a recorded success; others saved; final |
| 500 | `{"status":"error"}` | Retry |
| 502 | `{"status":"error"}` | Retry whole batch |

Retry only 500, 502 and timeouts, exponential back-off from 5 s, up to 3 retries. A recorded success can never change.

## Custom Receipts (`/receipt/`)

Tulus can call a merchant receipt endpoint with POST parameters describing the payment (the developer page lets you test rendering of success and failure receipts). The sample payload is the `fpx` object shape plus an `extra` object with nested `extra`, agency-specific (`pij` in the example), `zakat`, and `zakatstat` sections. Exact request encoding beyond the sample is not documented (checked 2026-10-21); confirm with Tulus.

## Receipt Email Customisation (`/customemail/`)

- Tulus POSTs form-encoded to your endpoint with two fields: `data` (EmailTemplateData JSON) and `fpx` (snake_case transaction JSON).
- EmailTemplateData fields: `Template`, `Dear`, `ServiceName`, `StatusMessage`, `StatusText`, `FPXTxnID`, `DateTime`, `SellerName`, `MerchantOrderNo`, `SellerOrderNo`, `BuyerBank`, `DebitStatus`, `CreditStatus`, `TransactionAmount`, `ExtraMessage`, `Subject`, `FPXDebitAuthCode`, `Attachments`, `LogoImage`, `Sender`, `Agency`, `AgencyName`.
- Respond with JSON containing only fields to change. `Attachments` replaces the list; each entry `{"Filename": "...", "Data": "<base64>"}`.
- Do not set `Template` unless Tulus gave you the name (unknown template renders empty).
- Failure is safe: unreachable, over 30 s, non-2xx, or invalid JSON means the default email is sent.
- Endpoint: `https://` on a public hostname (no IPs or internal hosts), max 512 chars, set per agency by Tulus support. Sample: `https://developers.tulus.my/customemail/endpoint.php` (`?attach=1` adds a PDF).
- Requests contain payer personal data; do not log or keep longer than needed.
