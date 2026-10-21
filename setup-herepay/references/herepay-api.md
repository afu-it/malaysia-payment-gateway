# Herepay API Reference

Use this reference for factual Herepay API guidance. All facts were checked against official sources on 2026-10-21. Where the official docs and official sample code disagree, both are shown; confirm with Herepay support before going live.

## Source URLs

- `https://herepay.readme.io/llms.txt` (docs index; append `.md` to any page URL for markdown)
- `https://herepay.readme.io/reference/integrating-with-herepay-api-payment`
- `https://herepay.readme.io/reference/p` (Transaction Flow)
- `https://herepay.readme.io/reference/introduction` (Environment)
- `https://herepay.readme.io/reference/headers` and `https://herepay.readme.io/reference/common-headers-copy`
- `https://herepay.readme.io/reference/get_api-v1-herepay-payment-channels`
- `https://herepay.readme.io/reference/post_api-v1-herepay-initiate`
- `https://herepay.readme.io/reference/post_callback`
- `https://herepay.readme.io/reference/get_api-v1-herepay-transactions`
- `https://herepay.readme.io/reference/get_api-v1-herepay-transactions-reference-code`
- `https://herepay.readme.io/reference/what-is-checksum` and `https://herepay.readme.io/reference/generating-checksum`
- `https://herepay.readme.io/reference/banks` (Mapping Table, test banks)
- `https://herepay.readme.io/reference/error-codes` (Status Codes)
- `https://github.com/Herepay/herepay-integration-samples` (official PHP and Node.js samples)
- `https://github.com/Herepay/herepay-for-woocommerce` (official plugin)
- `https://github.com/Herepay/herepay-php-sdk` (official PHP SDK, last pushed 2024-12)

## Endpoint Catalogue

Every operation in the official docs (readme.io OpenAPI, 26 pages in `llms.txt`) plus the official sample repo. Base URL per environment below. Checked 2026-10-21.

| # | Method | Path | Auth | Purpose | Source |
| --- | --- | --- | --- | --- | --- |
| 1 | `GET` | `/api/v1/herepay/payment/channels` | `SecretKey` + `XApiKey` | List payment methods, charges, and banks (`prefix`, `active`). | readme.io V1 |
| 2 | `POST` | `/api/v1/herepay/initiate` | `SecretKey` + `XApiKey` + body `checksum` | Create payment; returns HTML that form-POSTs the customer to the bank. | readme.io V1 |
| 3 | `GET` | `/api/v1/herepay/transactions` | `SecretKey` + `XApiKey` | List recent transactions (no documented paging or filters). | readme.io V1 |
| 4 | `GET` | `/api/v1/herepay/transactions/{reference_code}` | `SecretKey` + `XApiKey` | Get one transaction (requery). | readme.io V1 |
| 5 | `POST` | `<your callback URL>` | body `checksum` | Server-to-server payment result sent by Herepay to you. | readme.io Callback |
| 6 | `POST` or `GET` | `<your redirect URL>` | body `checksum` | Browser return after payment (UX only). | samples, plugin |
| 7 | `POST` | `/api/integration/create-payment-link` | `SecretKey` only | Create hosted payment link; redirect to `data.pay_url`. | samples only (`php/payment-link/`) |

Not documented anywhere official (checked 2026-10-21): refund, cancel, void, capture, recurring or token charge, payout or disbursement, settlement report, transaction lookup by `payment_code` (see Requery note), webhook management API.

## Integration Surfaces

- **Direct API (V1)**: merchant builds the bank picker. Fetch channels, initiate, receive callback. Documented on readme.io.
- **Payment link**: Herepay hosts the pay page. `POST /api/integration/create-payment-link`. Only in the official sample repo (`php/payment-link/`), not on readme.io.
- **WooCommerce plugin**: `herepay-for-woocommerce`.

The readme.io docs list FPX Online Banking and Corporate Banking only. The marketing site also mentions FPX credit card (Visa/Mastercard) and the callback `fpx_type` example includes `CCA`, but no API `payment_method` value for cards or e-wallets is documented (checked 2026-10-21).

## Environments

| Environment | Base URL |
| --- | --- |
| Testing (UAT) | `https://uat.herepay.org` |
| Production | `https://app.herepay.org` |

UAT and production are fully separated. Keys and data are not shared, and keys are not interchangeable.

## Authentication

Headers on every V1 endpoint:

| Header | Value | Notes |
| --- | --- | --- |
| `SecretKey` | string | Provided during onboarding. |
| `XApiKey` | string | Provided during onboarding. |

A third credential, the **Private Key**, is the HMAC key for checksums. It is not sent as a header. The checksum pages call it "your private key"; the SDK, samples, and plugin name it `private_key` / `HEREPAY_PRIVATE_KEY`.

Docs say API keys are for backend use only and must not be committed to version control.

Common header page lists `Content-Type: application/json`. The initiate OpenAPI schema says `multipart/form-data`, and the official PHP sample and WooCommerce plugin send `application/x-www-form-urlencoded`. The Node.js sample posts a JS object with axios (JSON). Safest choice: follow the official PHP sample (form-urlencoded).

## Transaction Flow (direct API)

From the Transaction Flow page:

1. Customer chooses Herepay; merchant calls Herepay to get available payment methods.
2. Merchant shows payment methods and the bank list.
3. Customer picks a bank; merchant sends initiate request with that `bank_prefix`.
4. Herepay sends the AR message to the bank; the customer authorises at the bank.
5. Herepay sends the callback (webhook) to the merchant and redirects the customer to the merchant redirect URL.
6. Bank crediting and RC message complete the transaction for interbank settlement.

## Get Payment Channels

`GET /api/v1/herepay/payment/channels` with `SecretKey` and `XApiKey`.

Response example shape:

```json
{
  "status": "success",
  "data": [
    {
      "payment_method": "Online Banking",
      "charges": { "currency": "MYR", "fixed": "0.70", "percentage": 0 },
      "channels": [
        { "prefix": "ABB0233", "active": "Active", "name": "Affin Bank", "img_url": "..." }
      ]
    },
    { "payment_method": "Corporate Banking", "charges": { ... }, "channels": [ ... ] }
  ],
  "message": "Payment channels and methods retrieved successfully."
}
```

Response fields (from the OpenAPI example; no schema is declared, only `200` is documented):

| Field | Type | Notes |
| --- | --- | --- |
| `status` | string | `"success"` (string here, unlike the integer `status` on transaction endpoints). |
| `message` | string | `Payment channels and methods retrieved successfully.` |
| `data[]` | array | One entry per payment method. |
| `data[].payment_method` | string | `Online Banking` (B2C) or `Corporate Banking` (B2B). Send this exact string as `payment_method`. |
| `data[].charges.currency` | string | `MYR`. |
| `data[].charges.fixed` | string | Fixed fee as decimal string. Informational; do not hardcode (Mapping Table page and this example show different numbers). |
| `data[].charges.percentage` | number | Percentage fee. |
| `data[].channels[]` | array | Banks for that method. |
| `data[].channels[].prefix` | string | Bank code; send as `bank_prefix`. |
| `data[].channels[].active` | string | `Active` or `Inactive`. |
| `data[].channels[].name` | string | Display name. Can be empty (example has prefix `MBBM2U2` with empty name); skip or label such rows. |
| `data[].channels[].img_url` | string | Bank logo URL. Can be empty. |

The full UAT example bank list is in `references/herepay-codes.md`.

- Use `channels[].prefix` as `bank_prefix` and the parent `payment_method` string as `payment_method`.
- `active` can be `Active` or `Inactive`; hide inactive banks.
- Call this live; do not hardcode bank lists. The static Mapping Table page is reference only.

## Initiate Invoice (create payment)

`POST /api/v1/herepay/initiate` with `SecretKey` and `XApiKey`.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `payment_code` | string | yes | Unique merchant reference, e.g. `INV20250609-XYZ123`. |
| `created_at` | string | yes | `YYYY-MM-DD hh:mm:ss`, e.g. `2024-06-12 12:23:32`. Timezone not documented. |
| `amount` | number/string | yes | Ringgit (MYR), not sen. `1` = RM1. Decimals allowed per OpenAPI ("MYR, Decimal"). |
| `name` | string | yes | Payer full name, one line. |
| `email` | string | yes | Valid email, max 50 chars. |
| `phone` | string | yes | e.g. `0123121989`, max 15 chars. SDK enforces digits only. |
| `description` | string | yes | Max 150 chars, e.g. `Registration Fee 2025`. |
| `bank_prefix` | string | yes | `prefix` from Payment Channels, e.g. `TEST0021`. |
| `payment_method` | string | yes | From Payment Channels, e.g. `Online Banking`, `Corporate Banking`. |
| `redirect_url` | string | optional | Not in the readme.io table. Official PHP/Node samples show it as optional and the WooCommerce plugin sends it. If sent, it must be included in the checksum. |
| `checksum` | string | yes | HMAC-SHA256 over the full request body excluding `checksum`. |

Response: on success Herepay returns an HTML page that form-POSTs the customer to the bank simulator (UAT) or real bank (production). The official samples and plugin echo that body back to the browser. There is no JSON checkout URL for this endpoint.

Errors: the docs warn that a bad request (missing field, wrong format, wrong checksum) only shows "Checksum verification failed", with no detail in the browser or network console. Test with static known-good values first.

## Checksum Algorithm

Source: Generating Checksum page (PHP, Java, Python, JavaScript, C# samples) and the official sample callback handlers.

String to sign:

1. Take every field you send (initiate) or receive (callback), **excluding** `checksum`.
2. Sort by key name ascending.
3. Replace any array/object value with its JSON encoding.
4. Join the **values only** (no keys) with `,`.
5. HMAC-SHA256 with the Private Key; output lowercase hex.

Official PHP:

```php
ksort($data);
foreach ($data as $k => $v) { if (is_array($v)) $data[$k] = json_encode($v); }
return hash_hmac('sha256', implode(',', $data), $privateKey);
```

Node.js:

```js
const keys = Object.keys(data).filter(k => k !== 'checksum').sort();
const str = keys.map(k => (typeof data[k] === 'object' && data[k] !== null) ? JSON.stringify(data[k]) : data[k]).join(',');
const checksum = crypto.createHmac('sha256', PRIVATE_KEY).update(str).digest('hex');
```

Worked order for an initiate payload: keys sort to `amount, bank_prefix, created_at, description, email, name, payment_code, payment_method, phone` (`redirect_url`, if sent, sorts last). String: `10,TEST0021,2024-06-12 12:23:32,Pembayaran Buku,johndoe@example.com,johndoe,INV20250609-XYZ123,Online Banking,0135162634`.

Gotchas:

- Sign the exact string form you send. `10`, `10.0`, and `10.00` produce different hashes.
- Do not URL-encode values before signing; sign raw values, then form-encode for transport.
- Empty-string fields are still values and still add a comma.
- The published PHP SDK passes the key through `openssl_pkey_get_private()` before `hash_hmac`; every other official sample uses the key string directly. Use the direct form.

## Callback (server-to-server)

Herepay POSTs to your callback URL after the customer completes payment. The callback URL is set in the Herepay dashboard (Managing URLs page; plugin README "Configure Webhook in Herepay Dashboard"). It must be publicly reachable; the samples suggest ngrok for local testing.

Body (`multipart/form-data` per OpenAPI; read as form fields):

| Field | Notes |
| --- | --- |
| `reference_code` | Herepay reference, e.g. `HP-INVAPI-674323C391A08`. Needed for requery. |
| `payment_code` | Your `payment_code`. |
| `transaction_id` | FPX transaction id. |
| `status` | Callback page: `Failed`, `Completed`, `Pending`. |
| `status_code` | Callback page: `0` failed, `1` completed, `2` pending. Samples: `00`, `29`, `30`. |
| `message` | `Approved`, `Failed`, `Pending`. |
| `amount` | Decimal string. |
| `currency` | e.g. `MYR`. |
| `payment_method` | String. |
| `fpx_type` | e.g. `FPX`, `CCA`. |
| `bank_name` | String. |
| `checksum` | HMAC as above over all other posted fields. |

Verification (official sample `php/direct-api/callback.php`):

```php
$received = $data['checksum'] ?? '';
unset($data['checksum']);
ksort($data);
$values = array_map(fn($v) => is_array($v) ? json_encode($v) : $v, $data);
$expected = hash_hmac('sha256', implode(',', $values), $privateKey);
if (!hash_equals($expected, $received)) { http_response_code(400); exit; }
```

All callback fields are typed `string` in the OpenAPI schema. The OpenAPI lists only a `200` response (`application/json`, no body schema); the official samples reply `200` with `OK` and `400` on bad checksum.

Field set to sign: the official samples sign every posted field except `checksum`. The WooCommerce plugin instead rebuilds the array from a fixed whitelist of the 12 fields above (missing ones become empty strings). Both agree today; if Herepay ever adds a field, the "all posted fields" form keeps matching, so prefer it.

Respond `200` with a short body after processing. Retry policy, retry count, and expected response body are not documented (checked 2026-10-21); design for duplicates anyway.

## Redirect (browser)

After payment Herepay sends the customer to your redirect URL. The samples read `$_POST` (payment-link sample accepts POST or GET) with the same fields as the callback, and the WooCommerce plugin verifies the same checksum there. Treat redirect as UX only: show the result, but settle only from the verified callback or a requery.

## Status Codes

Status Codes page:

| Code | Name | Meaning |
| --- | --- | --- |
| `00` | Success | Completed successfully. |
| `29` | Pending | In progress, awaiting confirmation. |
| `30` | Failed | Could not be completed. |
| `41` | Unauthorized | Not authorised to perform the action. |

Inconsistency: the Callback reference page and the Get Transaction By Reference Code example use `status_code` `1` with `status` `Completed`, while Get Transactions list examples use `00` with `status` `Success`. Map defensively, and only after checksum passes:

- Success: `status_code` in (`00`, `1`) or `status` in (`Success`, `Completed`).
- Pending: `29`, `2`, `Pending`. B2B example message: `Pending Authorization (Applicable for B2B model)`.
- Failed: `30`, `0`, `Failed`.

Full enum tables (status codes, callback values, HTTP codes, bank codes) are in `references/herepay-codes.md`.

HTTP codes: `200`, `201`, `204` success; `400` invalid input; `401` bad credentials; `403` invalid API key, bad signature, or no permission; `404` not found; `500`, `502` server side.

## Requery

`GET /api/v1/herepay/transactions/{reference_code}` with `SecretKey` and `XApiKey`.

```json
{
  "status": 200,
  "message": "Successfully fetched transaction for reference code: HP-INVAPI-...",
  "data": {
    "status": "Completed", "status_code": "1", "amount": "50.00",
    "reference_code": "HP-INVAPI-...", "payment_code": "PGW...",
    "bank_name": "SBI Bank C", "fpx_transaction_id": "...", "fpx_type": "FPX",
    "callback_url": "https://...", "direct_url": "https://..."
  }
}
```

Path parameter: `reference_code` (string, required), e.g. `HP-INVAPI-674323C391A08`.

Response fields (`200`):

| Field | Type | Notes |
| --- | --- | --- |
| `status` | integer | HTTP-style code, `200`. |
| `message` | string | `Successfully fetched transaction for reference code: <ref>`. |
| `data.status` | string | Example `Completed`. |
| `data.status_code` | string | Example `"1"` (see Status Codes inconsistency). |
| `data.amount` | string | Decimal string, e.g. `"50.00"`. |
| `data.reference_code` | string | Herepay reference. |
| `data.payment_code` | string | Your `payment_code`. |
| `data.bank_name` | string | e.g. `SBI Bank C`. |
| `data.fpx_transaction_id` | string | Same value the callback calls `transaction_id`. |
| `data.fpx_type` | string | e.g. `FPX`. |
| `data.callback_url` | string | Callback URL Herepay used. |
| `data.direct_url` | string | Redirect URL Herepay used. |

No `currency` field in this response example; check currency from the verified callback.

Not found: `404` with `{"status": 404, "message": "No transactions found for reference code: ..."}`.

### Get Transactions (list)

`GET /api/v1/herepay/transactions`, headers `SecretKey` and `XApiKey`, no query parameters documented.

| Field | Type | Notes |
| --- | --- | --- |
| `status` | integer | `200`. |
| `message` | string | `Successfully fetched all transactions`. |
| `data[].reference_code` | string | e.g. `HP-INVAPI-6751B20E7E3C4`. |
| `data[].payment_code` | string or null | `null` in every example. |
| `data[].transaction_id` | string | FPX transaction id. |
| `data[].status` | string | `Success`, `Pending`. |
| `data[].status_code` | string | `00`, `29`. |
| `data[].message` | string | `Approved`, `Transaction Pending`, `Pending Authorization (Applicable for B2B model)`. |
| `data[].amount` | string | e.g. `"100.00"`. |
| `data[].currency` | string | `MYR`. |
| `data[].payment_method` | string | `FPX` in examples (not `Online Banking`). |

Empty: `404` with `{"status": 404, "message": "No transactions found."}`. Pagination, ordering, and page size are not documented (example shows 10 rows).

Requery key: readme.io, the PHP SDK (`getTransactionDetails($referenceCode)`), and the samples all use `reference_code`. The WooCommerce plugin's `checkTransactionStatus()` calls `/api/v1/herepay/transactions/{payment_code}` and its `INTEGRATION_COMPLETE.md` lists that path. Lookup by `payment_code` is not on readme.io; treat it as unconfirmed. Persist `reference_code` from the first callback or redirect and requery by that.

## Payment Link (official samples only)

`POST /api/integration/create-payment-link`, headers `SecretKey`, `Content-Type: application/json`, `Accept: application/json` (no `XApiKey`, no checksum in the sample).

| Field | Required | Notes |
| --- | --- | --- |
| `title` | yes | e.g. `Consultation Fee`. |
| `amount` | yes | Ringgit, e.g. `10.50`. |
| `description` | no | HTML allowed in sample. |
| `usage_type` | no | `single` (one payment) or `multiple` (reusable). |
| `expires_at` | no | `YYYY-MM-DD hh:mm:ss`. |
| `collect_phone`, `note_enabled` | no | Booleans. |
| `note_label` | no | String. |
| `redirect_url`, `callback_url` | no | Per-link URLs. |
| `payer_name`, `payer_email`, `payer_phone` | no | Prefill. |

Response: redirect the customer to `data.pay_url`. No other response fields are shown in the official sample; the sample echoes the raw body when `data.pay_url` is missing (treat that as an error). The callback is verified the same way as the direct API (sample `php/payment-link/callback.php`, success when `status_code === '00'`). The payment-link redirect sample reads POST or GET and checks `status_code === '00'` without verifying checksum; verify it anyway if you show order state.

## Amount Rules

- Amounts are ringgit with decimals, never sen.
- Callback and requery return amounts as strings like `"100.00"`. Compare as decimal or integer sen, not as raw strings.

## Sandbox Test Data (UAT only)

Test banks: `ABB0234` (Affin B2C - Test ID), `LOAD001`, `TEST0021` (SBI Bank A), `TEST0022` (SBI Bank B), `TEST0023` (SBI Bank C).

Bank simulator credentials, published for everyone, UAT only:

- B2C: User Id `1234`, Password `1234`.
- B2B Maker: User Id `1234`, Password `1234`.
- B2B Authorizer: open `https://simulator.fpx.uat.inet.paynet.my/UatBuyerBankSim1.7/B2BAuth.jsp`, enter the Transaction ID from the callback or Maker submission, and authorise.

No test cards are published.

## Refunds, Recurring, Tokenisation

No refund, recurring, or tokenisation API is documented (checked 2026-10-21). The WooCommerce plugin readme says refunds are processed through the Herepay merchant dashboard.

## Rate Limits and IPs

Response examples show `X-RateLimit-Limit: 60` and `X-RateLimit-Remaining: 59`. No explicit limit or window is documented. No callback source IP list is documented (checked 2026-10-21); rely on checksum verification.

## Common Errors

- "Checksum verification failed" on initiate: field missing, wrong format, wrong key, amount formatting drift, or an extra field (such as `redirect_url`) left out of the signed string.
- `401`/`403`: UAT key used on production or the reverse, or wrong `SecretKey`/`XApiKey`.
- Callback never arrives: callback URL not set in dashboard or not public.
- B2B payment stuck pending: authorizer step not done.
