# Billplz API Reference

Source: https://www.billplz.com/api
Downloaded: 2026-05-24

This is a curated offline reference for common Billplz integration work. If exact live availability, account-specific settings, or newly added endpoint fields matter, verify the current official docs and account dashboard.

---

## Environment URLs

| Environment | Base URL |
|-------------|----------|
| Production | `https://www.billplz.com/api/` |
| Sandbox | `https://www.billplz-sandbox.com/api/` |

Sandbox and production are **separate accounts** with separate API keys. Never mix keys between environments.

---

## Authentication

HTTP Basic Auth - API Secret Key as username, no password.

```bash
# Option A: -u flag
curl https://www.billplz.com/api/v4/webhook_rank \
  -u YOUR_API_SECRET_KEY:

# Option B: Authorization header (base64 of "KEY:")
curl https://www.billplz.com/api/v4/webhook_rank \
  -H "Authorization: Basic BASE64_OF_KEY_COLON"
```

- All requests must use **HTTPS**. Plain HTTP fails.
- API Secret Key and XSignature Key are obtained from account settings page.
- Two keys required: `API Secret Key` (auth) and `XSignature Key` (signature verification).

---

## API Versions

| Version | Status | Scope |
|---------|--------|-------|
| V3 | Stable, no new features | Collections, Bills, Transactions, Payment Methods, FPX Banks |
| V4 | Active development | All V3 + 2-recipient split rules, Customer Receipt Delivery, Webhook Rank, Payment Gateways list, Card Tokenization (Senangpay) |
| V5 | Active development | Payment Orders (bank transfers), requires `epoch` + HMAC_SHA512 `checksum` on every request |

---

## Currency and Amounts

- Billplz accepts **Malaysian Ringgit (MYR) only**. No currency conversion.
- All amounts are in the **smallest currency unit** (sen).
- Example: `amount=200` = RM 2.00.

---

## API Flow

Normal payment flow:

1. Customer visits your site and chooses to pay.
2. Your server creates a Bill via API.
3. Billplz returns `bill.url`.
4. Your server redirects customer to `bill.url`.
5. Customer pays via chosen payment method.
6. Billplz POSTs to `callback_url` (server-side, compulsory).
7. Billplz redirects customer to `redirect_url` if set (optional, UX only).

**`callback_url` is compulsory.** `redirect_url` is optional but recommended for instant UX feedback.

The order of `redirect_url` and `callback_url` execution is **not guaranteed**. Implement idempotency.

---

## Collections (V3)

A Collection groups Bills. Bills must belong to a Collection. Lifetime limit: **20,000 collections per account** (standard + open combined).

### Create a Collection

```
POST https://www.billplz.com/api/v3/collections
```

**Required:**

| Field | Description |
|-------|-------------|
| `title` | Collection title (string) |

**Optional:**

| Field | Description |
|-------|-------------|
| `logo` | Image file (jpg, jpeg, gif, png); resized to 40x40 and 180x180 |
| `split_payment[email]` | Split recipient email (verified account) |
| `split_payment[fixed_cut]` | Fixed cut in smallest unit (required if `variable_cut` absent) |
| `split_payment[variable_cut]` | Percentage as positive integer (required if `fixed_cut` absent) |
| `split_payment[split_header]` | Boolean - show split infographic on bill/receipt |

**Response:**

```json
{
  "id": "inbmmepb",
  "title": "My Collection",
  "logo": { "thumb_url": null, "avatar_url": null },
  "split_payment": { "email": null, "fixed_cut": null, "variable_cut": null, "split_header": false }
}
```

### Get a Collection

```
GET https://www.billplz.com/api/v3/collections/{COLLECTION_ID}
```

Response adds `"status": "active"` or `"inactive"`.

### Get Collection Index

```
GET https://www.billplz.com/api/v3/collections?page=1&status=active
```

Up to 15 per page. Optional: `page`, `status` (`active`|`inactive`).

### Create an Open Collection (Payment Form)

```
POST https://www.billplz.com/api/v3/open_collections
```

**Required:** `title` (max 50), `description` (max 200), `amount` (required if `fixed_amount=true`)

**Optional:** `fixed_amount` (default `true`), `fixed_quantity` (default `true`), `payment_button` (`buy`|`pay`), `reference_1_label` (max 20), `reference_2_label` (max 20), `email_link`, `tax`, `photo`, `split_payment[*]`

Response includes `url` (payment form URL).

### Activate / Deactivate a Collection

```
POST https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/activate
POST https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/deactivate
```

Returns `{}` on success.

---

## Collections (V4)

V4 supports **2 split rule recipients** using `split_payments` array (vs V3's single `split_payment` object).

```
POST https://www.billplz.com/api/v4/collections
```

**Optional split fields:** `split_payments[][email]`, `split_payments[][fixed_cut]`, `split_payments[][variable_cut]`, `split_payments[][stack_order]` (starts at 0), `split_header`

Custom logo upload is removed starting in V4. Do not send V3 `logo` uploads to V4 collection endpoints.

V4 collection responses include `split_payments` and a top-level `split_header` boolean.

### V4 Open Collections

```
POST https://www.billplz.com/api/v4/open_collections
GET  https://www.billplz.com/api/v4/open_collections/{COLLECTION_ID}
GET  https://www.billplz.com/api/v4/open_collections
```

V4 Open Collections follow the V3 Open Collection shape with V4 split payment support. `redirect_uri` is an optional V4 field; Billplz redirects the customer there after payment completion or failure and appends the bill ID to the URL.

### Customer Receipt Delivery (V4 only)

```
POST /v4/collections/{COLLECTION_ID}/customer_receipt_delivery/activate
POST /v4/collections/{COLLECTION_ID}/customer_receipt_delivery/deactivate
POST /v4/collections/{COLLECTION_ID}/customer_receipt_delivery/global
GET  /v4/collections/{COLLECTION_ID}/customer_receipt_delivery
```

GET response: `{ "id": "...", "customer_receipt_delivery": "active"|"inactive"|"global" }`

---

## Bills (V3)

Bills can only be created inside a **standard Collection** (not Open Collection). Collection is auto-activated on bill creation. Default bill expiry: **30 days**.

### Create a Bill

```
POST https://www.billplz.com/api/v3/bills
```

**Required:**

| Field | Description |
|-------|-------------|
| `collection_id` | Collection ID |
| `email` | Recipient email (required if `mobile` absent) |
| `mobile` | Recipient mobile with country code e.g. `+60122345678` (required if `email` absent) |
| `name` | Recipient name (max 255 chars) |
| `amount` | Positive integer in smallest currency unit (e.g. `200` = RM 2.00) |
| `callback_url` | Webhook URL for server-side payment notification (POST) |
| `description` | Bill description (max 200 chars) |

**Optional:**

| Field | Description |
|-------|-------------|
| `due_at` | Due date `YYYY-MM-DD`, default today. Year range 19xx-2xxx. Informational only - does not affect payability. |
| `redirect_url` | URL to redirect customer after payment (GET with bill status) |
| `deliver` | Boolean - send email/SMS on creation. Default `false`. SMS costs apply. |
| `reference_1_label` | Label for reference 1 (max 20 chars, default "Reference 1") |
| `reference_1` | Value for reference 1 (max 120 chars) |
| `reference_2_label` | Label for reference 2 (max 20 chars, default "Reference 2") |
| `reference_2` | Value for reference 2 (max 120 chars) |

**Response:**

```json
{
  "id": "8X0Iyzaw",
  "collection_id": "inbmmepb",
  "paid": false,
  "state": "due",
  "amount": 200,
  "paid_amount": 0,
  "due_at": "2020-12-31",
  "email": "api@billplz.com",
  "mobile": "+60112223333",
  "name": "Sara",
  "url": "https://www.billplz.com/bills/8X0Iyzaw",
  "reference_1_label": "Reference 1",
  "reference_1": null,
  "reference_2_label": "Reference 2",
  "reference_2": null,
  "redirect_url": "http://example.com/redirect/",
  "callback_url": "http://example.com/webhook/",
  "description": "Maecenas eu placerat ante.",
  "paid_at": null
}
```

### Get a Bill

```
GET https://www.billplz.com/api/v3/bills/{BILL_ID}
```

Not recommended - subject to rate limits. Use X Signature Callback/Redirect instead.

`state` values: `due`, `paid`, `deleted`.

### Delete a Bill

```
DELETE https://www.billplz.com/api/v3/bills/{BILL_ID}
```

Only `due` bills can be deleted. Paid bills return `422`. Deleted bills reappear if customer later pays.

### Get Transaction Index

```
GET https://www.billplz.com/api/v3/bills/{BILL_ID}/transactions?page=1&status=completed
```

Optional: `page` (default 1, max 15/page), `status` (`pending`|`completed`|`failed`)

**Response:**

```json
{
  "bill_id": "inbmmepb",
  "transactions": [
    {
      "id": "60793D4707CD",
      "status": "completed",
      "completed_at": "2017-02-23T12:49:23.612+08:00",
      "payment_channel": "FPX"
    }
  ],
  "page": 1
}
```

**Payment channel values:** `AMEXMBB`, `BANKISLAM`, `BILLPLZ`, `BOOST`, `TOUCHNGO`, `EBPGMBB`, `FPX`, `FPXB2B1`, `ISUPAYPAL`, `MPGS`, `OCBC`, `PAYDEE`, `RAZERPAYWALLET`, `SECUREACCEPTANCE`, `SENANGPAY`, `TWOCTWOP`, `TWOCTWOPIPP`, `TWOCTWOPWALLET`

---

## Direct Payment Gateway (Bypass Bill Page)

Skip Billplz payment selection page and redirect customer directly to a specific payment gateway.

1. Create bill with `reference_1_label="Bank Code"` and `reference_1="{BANK_CODE}"`.
2. Append `?auto_submit=true` to the returned `bill.url`.
3. Redirect customer to that URL.

Falls back to Billplz payment selection page if bank code is invalid or inactive.

---

## Payment Methods (V3)

### Get Payment Method Index

```
GET https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/payment_methods
```

Response: `{ "payment_methods": [{ "code": "fpx", "name": "Online Banking", "active": true }] }`

### Update Payment Methods

```
PUT https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/payment_methods
```

Body: `payment_methods[][code]=fpx&payment_methods[][code]=paypal`

Pass only codes to **enable**. Omitted codes are disabled.

**Valid codes:** `amexmbb`, `bankislam`, `billplz`, `boost`, `touchngo`, `ebpgmbb`, `fpx`, `fpxb2b1`, `isupaypal`, `mpgs`, `ocbc`, `paydee`, `razerpaywallet`, `secureacceptance`, `senangpay`, `twoctwop`, `twoctwopipp`, `twoctwopwallet`

---

## Get FPX Banks (V3)

```
GET https://www.billplz.com/api/v3/fpx_banks
```

Pull on **hourly** basis. Returns `name` (bank code for `reference_1`) and `active` (boolean).

---

## Get Payment Gateways (V4)

```
GET https://www.billplz.com/api/v4/payment_gateways
```

Returns current payment gateway codes including FPX, e-wallets, and cards. Pull this dynamically when exact availability matters.

Response shape:

```json
{
  "payment_gateways": [
    { "code": "MBU0227", "active": true, "category": "fpx" },
    { "code": "BP-FKR01", "active": true, "category": "billplz" }
  ]
}
```

---

## Webhook Rank (V4)

```
GET https://www.billplz.com/api/v4/webhook_rank
```

Response: `{ "rank": 0.0 }` - 0.0 = highest priority, 10.0 = lowest.

Resets daily at 17:00. Degrades by 1 per failed callback attempt. Maintain a healthy rank by returning HTTP 200 promptly from callback endpoints.

---

## Payment Completion

### Basic Callback URL (POST)

Billplz POSTs to `callback_url` on payment completion (success or failure).

**POST fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Bill ID |
| `collection_id` | string | Collection ID |
| `paid` | boolean | `true` = paid |
| `state` | string | `due`, `paid`, `deleted` |
| `amount` | integer | Bill amount in smallest unit |
| `paid_amount` | integer | Amount actually paid |
| `due_at` | string | `YYYY-MM-DD` |
| `email` | string | Recipient email |
| `mobile` | string | Recipient mobile |
| `name` | string | Recipient name |
| `url` | string | Bill page URL |
| `paid_at` | string | `YYYY-MM-DD HH:MM:SS +TIMEZONE` |

Retry schedule on failure: +15s+N (0-300s random), +15min+N, +15min+N, +24h+N. Max 5 attempts. 20s timeout per attempt.

### Basic Redirect URL (GET)

Billplz redirects customer to `redirect_url?billplz[id]={BILL_ID}`. Only `billplz[id]` is sent. Not guaranteed (browser may close).

### Payment Order Callback (V5)

Billplz sends Payment Order callback notifications when a payment order status changes to `completed` or `refunded`. Callback payloads can also include `cancelled`; handle it as a terminal non-success state.

**Callback fields:** `id`, `payment_order_collection_id`, `bank_code`, `bank_account_number`, `name`, `description`, `email`, `status`, `notification`, `recipient_notification`, `total`, `reference_id`, `display_name`, `epoch`, `checksum`

**Callback status values:** `completed`, `refunded`, `cancelled`

**Checksum args:** `[id, bank_account_number, status, total, reference_id, epoch]`

Timeout is 20 seconds. Billplz makes at most 2 attempts; the second attempt is after 1 hour. Account webhook rank is degraded by 1 for every unsuccessful callback attempt.

---

## X Signature

Verifies that callbacks and redirects genuinely originate from Billplz and data has not been tampered with. Uses **HMAC_SHA256** with your **XSignature Key**.

### X Signature Calculation Rules

1. Extract every key-value element from the payload except `x_signature`.
2. For nested objects or arrays, prepend the parent key to child keys before concatenating. Example: `collections[{id,title}]` becomes elements such as `collectionsid...` and `collectionstitle...`.
3. Include empty values in the source string. Example: an empty `mobile` field contributes `mobile` with no value suffix.
4. Sort all elements case-insensitively in ascending order.
5. Join sorted elements with `|`.
6. Compute `HMAC_SHA256(source_string, XSignature_Key)`.

Do not JSON stringify or reformat values before signature calculation. Preserve the exact values received from Billplz.

### X Signature Callback (POST)

Extra fields vs Basic Callback: `x_signature`, and optionally `transaction_id` + `transaction_status` (if "Extra Payment Completion Information" is enabled in dashboard).

**Verification steps:**

1. Extract all POST body fields **except `x_signature`**.
2. For each field: concatenate `key` + `value` (no separator between key and value).
3. Sort all entries **case-insensitively ascending**.
4. Join with `|` pipe character.
5. Compute `HMAC_SHA256(source_string, XSignature_Key)`.
6. Compare with received `x_signature`. Match = authentic. No need to call Get Bill API.

**Example source string (sorted, piped):**
```
amount100|collection_idyhx5t1pp|due_at2018-9-27|emailapi@billplz.com|idzq0tm2wc|mobile|nameTESTER|paid_amount100|paid_at2018-09-27 15:15:09 +0800|paidtrue|statepaid|urlhttp://www.billplz-sandbox.com/bills/zq0tm2wc
```

`metadata` may be present in legacy/basic callback payloads but is deprecated. Include it in signature calculation if Billplz sends it and `x_signature` mode is enabled.

### X Signature Redirect (GET)

Query params: `billplz[id]`, `billplz[paid]`, `billplz[paid_at]`, `billplz[x_signature]`, and optionally `billplz[transaction_id]`, `billplz[transaction_status]`. `billplz[paid_at]` is included in X Signature redirects.

**Verification steps:**

1. Extract all `billplz[...]` query params **except `billplz[x_signature]`**.
2. For each: concatenate `billplz` + key + value (e.g. `billplzidzq0tm2wc`).
3. Sort case-insensitively ascending, join with `|`.
4. Compute `HMAC_SHA256(source_string, XSignature_Key)`.
5. Compare with `billplz[x_signature]`. Match = authentic.

**Example source string:**
```
billplzidzq0tm2wc|billplzpaid_at2018-09-27 15:15:09 +0800|billplzpaidtrue
```

---

## V5 Payment Orders

Make payment transfers to any Malaysian bank account.

**Prerequisite:** Sufficient Payment Order Limit. Sandbox: use `bank_code=DUMMYBANKVERIFIED` for guaranteed success.

Every V5 request requires `epoch` (UNIX timestamp) and `checksum` (HMAC_SHA512).

### V5 Checksum Formula

1. Concatenate **values only** of required args in the **strict order** per endpoint (no delimiters).
2. Compute `HMAC_SHA512(raw_string, XSignature_Key)`.

For optional checksum arguments, include the value only when the optional argument is present. If Billplz sends nullable callback values such as `reference_id`, preserve the received value exactly when rebuilding the checksum string.

**Endpoint checksum arg orders:**

| Endpoint | Args (strict order) |
|----------|---------------------|
| Create Payment Order Collection | `[title, callback_url*, epoch]` |
| Get Payment Order Collection | `[payment_order_collection_id, epoch]` |
| Create Payment Order | `[payment_order_collection_id, bank_account_number, total, epoch]` |
| Get Payment Order | `[payment_order_id, epoch]` |
| Get Payment Order Limit | `[epoch]` |
| Payment Order Callback | `[id, bank_account_number, status, total, reference_id, epoch]` |

`*` = optional; omit from checksum string if not used.

### Create Payment Order Collection

```
POST https://www.billplz.com/api/v5/payment_order_collections
```

**Required:** `title`, `epoch`, `checksum`
**Optional:** `callback_url`

**Response:** `{ id, title, payment_orders_count, paid_amount, status, callback_url }`

### Get Payment Order Collection

```
GET https://www.billplz.com/api/v5/payment_order_collections/{PAYMENT_ORDER_COLLECTION_ID}
```

**Required query/body values:** `epoch`, `checksum`

**Checksum args:** `[payment_order_collection_id, epoch]`

**Response:** `{ id, title, payment_orders_count, paid_amount, status, callback_url }`

### Create Payment Order

```
POST https://www.billplz.com/api/v5/payment_orders
```

**Required:** `payment_order_collection_id`, `bank_code`, `bank_account_number`, `name`, `description` (max 200, no special chars), `total` (smallest unit), `epoch`, `checksum`

**Optional:** `email`, `notification` (sender email opt-in on completion, default `false`), `recipient_notification` (recipient email, default `true`), `reference_id` (max 255, unique per collection)

Use `reference_id` as an idempotency/correlation key. Billplz enforces uniqueness within a payment order collection to prevent accidental duplicate payment orders.

`bank_code` is case-sensitive. `description` has max 200 characters and should not contain special characters.

**Response fields:** `id`, `payment_order_collection_id`, `bank_code`, `bank_account_number`, `name`, `description`, `email`, `status`, `notification`, `recipient_notification`, `total`, `reference_id`, `display_name`

**Payment Order statuses:** `processing`, `enquiring`, `executing`, `reviewing`, `completed`, `refunded`

### Get Payment Order

```
GET https://www.billplz.com/api/v5/payment_orders/{PAYMENT_ORDER_ID}
```

**Required query/body values:** `epoch`, `checksum`

**Checksum args:** `[payment_order_id, epoch]`

**Response fields:** `id`, `payment_order_collection_id`, `bank_code`, `bank_account_number`, `name`, `description`, `email`, `status`, `notification`, `recipient_notification`, `total`, `reference_id`, `display_name`

### Get Payment Order Limit

```
GET https://www.billplz.com/api/v5/payment_order_limit
```

Required: `epoch`, `checksum` (args: `[epoch]`)

Response: `{ "total": 100000 }` in smallest currency unit.

Rate limits: Sandbox 1 req/10s, Production 3 req/10min.

---

## Card Tokenization (V4 - Senangpay)

Available to paid plan members only. Requires enabling via email to partner@billplz.com.

### Create Card Token

```
POST https://www.billplz.com/api/v4/cards
```

**Required:** `name`, `email`, `phone`, `callback_url`

**Response:** `{ id, status: "pending", authentication_redirect_url, ... }`

Redirect cardholder to `authentication_redirect_url` for 3DS verification. Billplz POSTs result to `callback_url` within 1 hour regardless of whether 3DS was completed.

### 3D Secure Update Callback

Billplz POSTs to the card `callback_url` with `id`, `card_number` (last 4 digits), `provider`, `token`, `status`, and `checksum`.

Card status values include `pending`, `active`, `failed`, and `deleted`.

Verify `checksum` using the same HMAC_SHA256 construction pattern as X Signature. Respond with HTTP status `200`-`308` within 20 seconds; other responses are treated as failures.

### Charge Card

```
POST https://www.billplz.com/api/v4/bills/{BILL_ID}/charge
```

**Required:** `card_id`, `token`

Requirements: Collection without split payment; Bill with email and mobile.

**Response fields:** `amount`, `status` (`success` or `failed`), `reference_id`, `hash_value`, `message`

### Pre-Auth

```
POST https://www.billplz.com/api/v4/bills/{BILL_ID}/preauth
```

**Required:** `card_id`, `token`

Requirements: Collection without split payment; Bill with email and mobile.

**Response fields:** `amount`, `preauth_status` (`success` or `failed`), `preauth_id`, `hash_value`, `message`

### Pre-Auth Capture

```
POST https://www.billplz.com/api/v4/bills/{BILL_ID}/preauth_capture
```

**Required:** `card_id`, `token`, `preauth_id`

Requirements: Collection without split payment; Bill with email and mobile.

**Response fields:** `amount`, `status` (`success` or `failed`), `preauth_id`, `reference_id`, `hash_value`, `message`

### Delete Card

```
DELETE https://www.billplz.com/api/v4/cards/{CARD_ID}
```

**Required body parameter:** `token`

**Response fields:** `id`, `card_number` (last 4 digits), `provider`, `token`, `status` (`deleted`)

---

## Rate Limits

Applies to all **GET** endpoints. Cumulative per IP and per account.

| Level | Allowance |
|-------|-----------|
| Default | 100 requests per 5-minute window |
| Abusive | 10 requests per 5-minute window |

**Response headers:** `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset` (seconds until reset, max 300)

**On exceeding:** HTTP `429` - `{ "error": { "type": "RateLimit", "message": ["Too many requests"] } }`

---

## Error Codes

| Code | Meaning |
|------|---------|
| `401` | Unauthorized - invalid API key |
| `404` | Not Found |
| `422` | Unprocessable Entity - invalid parameter |
| `429` | Too Many Requests - rate limit exceeded |
| `500` | Internal Server Error - retry later |
| `503` | Service Unavailable - maintenance |

---

## FPX Bank Codes (B2C)

Pull live list from `GET /v3/fpx_banks` hourly. Selected static reference below (may change):

| Code | Bank |
|------|------|
| ABMB0212 | allianceonline |
| ABB0233 | affinOnline |
| ABB0234* | Affin Bank |
| AMBB0209 | AmOnline |
| AGRO01 | AGRONet |
| BCBB0235 | CIMB Clicks |
| BIMB0340 | Bank Islam Internet Banking |
| BKRM0602 | i-Rakyat |
| BMMB0341 | i-Muamalat |
| BOCM01* | Bank of China |
| BSN0601 | myBSN |
| CIT0219 | Citibank Online |
| HLB0224 | HLB Connect |
| HSBC0223 | HSBC Online Banking |
| KFH0346 | KFH Online |
| MB2U0227 | Maybank2u |
| MBB0228 | Maybank2E |
| OCBC0229 | OCBC Online Banking |
| PBB0233 | PBe |
| RHB0218 | RHB Now |
| SCB0216 | SC Online Banking |
| UOB0226 | UOB Internet Banking |
| UOB0229* | UOB Bank |
| TEST0001* | Test 0001 (sandbox only) |
| TEST0002* | Test 0002 (sandbox only) |
| TEST0003* | Test 0003 (sandbox only) |
| TEST0004* | Test 0004 (sandbox only) |
| TEST0021* | Test 0021 (sandbox only) |
| TEST0022* | Test 0022 (sandbox only) |
| TEST0023* | Test 0023 (sandbox only) |
| BP-FKR01* | Billplz Simulator (sandbox only) |

`*` = sandbox/staging only.

## FPX Bank Codes (B2B)

| Code | Bank |
|------|------|
| B2B1-ABB0235 | AFFINMAX |
| B2B1-ABMB0213 | Alliance BizSmart |
| B2B1-AGRO02 | AGRONetBIZ |
| B2B1-AMBB0208 | AmAccess Biz |
| B2B1-BCBB0235 | BizChannel@CIMB |
| B2B1-BIMB0340 | Bank Islam eBanker |
| B2B1-BKRM0602 | i-bizRAKYAT |
| B2B1-BMMB0342 | iBiz Muamalat |
| B2B1-HLB0224 | HLB ConnectFirst |
| B2B1-HSBC0223 | HSBCnet |
| B2B1-MBB0228 | Maybank2E |
| B2B1-OCBC0229 | Velocity@ocbc |
| B2B1-PBB0233 | PBe |
| B2B1-RHB0218 | RHB Reflex |
| B2B1-SCB0215 | SC Straight2Bank |
| B2B1-UOB0228 | UOB BIBPlus |

## Payment Gateway Codes (E-Wallet / Card)

| Code | Gateway |
|------|---------|
| BP-BILLPLZ1 | Visa/Mastercard (Billplz) |
| BP-PPL01 | PayPal |
| BP-OCBC1 | Visa/Mastercard (OCBC) |
| BP-BST01 | Boost |
| BP-TNG01 | TouchNGo E-Wallet |
| BP-SGP01 | Senangpay |
| BP-2C2P1 | e-pay |
| BP-2C2PC | Visa/Mastercard (2C2P) |
| BP-2C2PU | UnionPay |
| BP-2C2PGRB | Grab |
| BP-2C2PGRBPL | GrabPayLater |
| BP-2C2PATOME | Atome |
| BP-2C2PBST | Boost (2C2P) |
| BP-2C2PTNG | TnG (2C2P) |
| BP-2C2PSHPE | Shopee Pay |
| BP-2C2PSHPQR | Shopee Pay QR |
| BP-2C2PIPP | IPP |
| BP-RZRGRB | Grab (Razer) |
| BP-RZRBST | Boost (Razer) |
| BP-RZRTNG | TnG (Razer) |
| BP-RZRPAY | RazerPay |
| BP-RZRMB2QR | Maybank QR |
| BP-RZRWCTP | WeChat Pay |
| BP-RZRSHPE | Shopee Pay (Razer) |
| BP-MPGS1 | MPGS |
| BP-CYBS1 | Secure Acceptance |
| BP-EBPG1 | Visa/Mastercard (EBPG) |
| BP-EBPG2 | AMEX |
| BP-PAYDE | Paydee |
| BP-MGATE1 | Visa/Mastercard/AMEX |

## SWIFT Bank Codes (V5 Payment Orders)

| Bank | SWIFT Code |
|------|-----------|
| Affin Bank Berhad | PHBMMYKL |
| AGROBANK | AGOBMYKL |
| Alliance Bank Malaysia Berhad | MFBBMYKL |
| AmBank (M) Berhad | ARBKMYKL |
| Bank Islam Malaysia Berhad | BIMBMYKL |
| Bank Kerjasama Rakyat Malaysia Berhad | BKRMMYKL |
| Bank Muamalat (Malaysia) Berhad | BMMBMYKL |
| Bank Simpanan Nasional Berhad | BSNAMYK1 |
| CIMB Bank Berhad | CIBBMYKL |
| Citibank Berhad | CITIMYKL |
| Hong Leong Bank Berhad | HLBBMYKL |
| HSBC Bank Malaysia Berhad | HBMBMYKL |
| Kuwait Finance House | KFHOMYKL |
| Maybank / Malayan Banking Berhad | MBBEMYKL |
| OCBC Bank (Malaysia) Berhad | OCBCMYKL |
| Public Bank Berhad | PBBEMYKL |
| RHB Bank Berhad | RHBBMYKL |
| Standard Chartered Bank (Malaysia) Berhad | SCBLMYKX |
| United Overseas Bank (Malaysia) Berhad | UOVBMYKL |
| DUMMYBANKVERIFIED | Sandbox test - always succeeds |

---

## Sandbox Notes

- Sandbox endpoint: `https://www.billplz-sandbox.com/api/`
- Sandbox account is separate from production - use sandbox keys in sandbox, production keys in production.
- For V5 Payment Orders sandbox testing: use `bank_code=DUMMYBANKVERIFIED` for guaranteed success. Any other bank code fails.
- FPX sandbox-only bank codes: `TEST0001`-`TEST0004`, `TEST0021`-`TEST0023`, `BP-FKR01` (Billplz Simulator).
- Some production features may not be available in sandbox.

---

## Required Environment Variables

```
BILLPLZ_API_KEY=           # API Secret Key from account settings
BILLPLZ_X_SIGNATURE_KEY=   # XSignature Key from account settings
BILLPLZ_COLLECTION_ID=     # Pre-created collection ID (or create via API)
BILLPLZ_SANDBOX=true       # Set to false for production
```
