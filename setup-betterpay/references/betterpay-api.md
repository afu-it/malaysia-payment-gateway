# Betterpay API Reference

Checked: 2026-10-21. This is the entry point. Detail lives in:

- `betterpay-recurring.md`: FPX e-Mandate direct debit (PRO), OCBC collection batches (PRO), card recurring (LITE).
- `betterpay-tokenization.md`: card tokenization, token pre-auth, batch charge, Card Pre-authorization (MPGS), LITE tokenization.
- `betterpay-codes.md`: every status, response, error and bank/channel code.

Official sources (see `source-map.md`):

- Docs (one long page, anchors per section): `https://www.betterpay.me/docs/`
- Official Postman collections: `https://github.com/betterpayme/POSTMAN-APIs`
- Official WooCommerce plugin source: `https://wordpress.org/plugins/betterpay/` (v1.2.4)

## What Betterpay Is

- Betterpay is run by Pay Direct Technology Sdn Bhd. It was formerly QlicknPay and runs on the same API stack. The docs banner says: "All API endpoints using the TLD Betterpay.me or qnp.com is valid". The docs still use the QlicknPay name and `qlicknpay.com` links in many places.
- There are two account products with **different hashing**:
  - **Betterpay PRO** (the original QlicknPay v1 API, numeric merchant ID such as `10001`): MD5 over pipe-joined values. The field order is different per endpoint.
  - **Betterpay LITE** (merchant ID starting with `R`, such as `R1001`): HMAC-SHA256 over the sorted, concatenated values with spaces removed.
- The official WooCommerce plugin tells the products apart with `str_contains($merchantId, 'R')`: an `R` in the merchant ID means LITE, otherwise PRO.
- Treat PRO and LITE as two different integrations. Never mix their endpoints or hash methods.

## Environments And Base URLs

The docs hide every endpoint behind "Please login to view the endpoint". The paths below come from the official Postman collections (sandbox) and the official WooCommerce plugin (production hosts).

| Item | Value | Source |
|---|---|---|
| Sandbox host (PRO and LITE APIs) | `https://www.demo.betterpay.me` | Postman collections |
| Older sandbox host in docs | `https://www.demo.qlicknpay.com` | docs |
| PRO production host | `https://www.betterpay.me` | WP plugin posts to `https://www.betterpay.me/merchant/api/v2/woocommerce_receiver` |
| PRO sandbox (plugin) | `https://www.demo.betterpay.me/merchant/api/v2/woocommerce_receiver` | WP plugin |
| LITE production checkout (plugin) | `https://lite.betterpay.me/api/merchant/v1/checkout` | WP plugin |
| PRO API Management (keys) | `https://www.betterpay.me/merchant/v-api-management` | docs |
| LITE API Management (keys) | `https://lite.betterpay.me/merchant/api/management` | docs |

- Production API paths for the non-plugin endpoints are not published. The docs say Betterpay.me hosts are valid, so the PRO production path is expected to be `https://www.betterpay.me` + the same path as sandbox. Confirm this from the logged-in docs before go-live.
- LITE production API hosts and paths (other than the plugin checkout URL) are not documented publicly (checked 2026-10-21). Read them from the logged-in docs or ask support.
- Keep host and credentials in config per environment. Never hardcode the sandbox host.

## Auth

- Credentials: **Merchant ID** and **API Key**, from the API Management page. There is no bearer token, OAuth or header auth.
- Every request carries `merchant_id` and `hash` in the body. The API key is only used to build the hash. Never send the API key itself.
- "All API requests must be made over HTTPS. Calls made over plain HTTP will fail. All request must be hashed for authentication." (docs, Introduction)
- PRO error text says "Your API key must be less than 14 characters". Do not trim or transform the key.
- Bodies: Standard Payment is a browser form POST (`application/x-www-form-urlencoded`). Most other APIs are server-to-server POSTs. The docs samples use both form fields and JSON (`Content-Type: application/json`), and the Postman collections send JSON for most calls. Use what the endpoint sample shows and test it in sandbox.

## Signature Algorithms

### PRO: MD5 over pipe-joined values

```
hash = md5( join("|", [values in the endpoint's exact order]) )   // lowercase hex
```

- The PHP samples `urldecode()` each value first. So you hash the plain, unencoded value, exactly as it will be sent.
- The order differs per endpoint, and some endpoints put `merchant_id` BEFORE the API key. Use the table below. Never guess.
- Callbacks use plain concatenation with no separators.

| Operation | String to MD5 |
|---|---|
| Standard Payment, Create Bill | `api\|merchant_id\|invoice\|amount\|payment_desc\|currency` |
| Remove Created Bill | `api\|merchant_id\|invoice` |
| Direct Payment (incl. DuitNow QR create) | `api\|merchant_id\|invoice\|amount\|payment_desc\|bank_code\|payment_method\|currency` |
| Payment Link create | `api\|merchant_id\|purpose\|amount_option\|amount\|delivery_option\|currency` (`amount` empty for `Unfixed`) |
| Payment Link delete | `api\|merchant_id\|id` |
| Payment Link list | `api\|merchant_id` |
| Payment Link report | `api\|merchant_id\|purpose\|date_to\|date_from` (note: date_to before date_from) |
| Get Bank List | `api\|merchant_id\|type` |
| Get Payment Status (requery) | `merchant_id\|api\|invoice\|method` |
| FPX Refund | `merchant_id\|api\|merchant_bank_id\|merchant_acc_number\|merchant_maker_name\|organization_id\|refund_list\|enable_callback\|enable_email\|refund_callback_url` |
| DuitNow QR (OCBC) Refund | `merchant_id\|api\|refund_list` |
| Get Refund Status | `merchant_id\|api\|ref_no` |
| Get Transactions, Get Settlement, Get Callbacks, Get Balance, Get Disburse Records | `api\|merchant_id` |
| Get Transaction (by invoice) | `api\|merchant_id\|invoice` |
| Recurring, OCBC, tokenization, pre-auth | see `betterpay-recurring.md` and `betterpay-tokenization.md` |

Standard Payment example from the docs: `APIKEY123456|100055|INV10154632|1289.00|Parking Fee|MYR`, then MD5. The older sample PHP form in the same section omits `currency`. Follow the hashing guide (with currency), which also matches the official Postman pre-request script.

**PRO payment callback verification** (Standard and Direct Payment):

```
trx_id = fpx_fpxTxnId | paypal_trx_id | mastercard_trx_id | others_trx_id   // whichever pay_method gives you
expected = md5(api + trx_id + invoice_no + txn_status + msg)                  // no separators
```

Docs example string: `APIKEY1234561808241535340347INV01234500Transaction Approved`.

- `pay_method` = `fpx` uses `fpx_fpxTxnId`, `paypal` uses `paypal_trx_id`, `mastercard` uses `mastercard_trx_id`, `others` uses `others_trx_id` (and `trx_txt` names the channel, for example `eWallet (DuitNow QR)`).
- The WooCommerce plugin uses its own endpoint (`/merchant/api/v2/woocommerce_receiver`). It signs the request with `md5(api . merchant_id . order_id . amount . detail)` (no separators) and verifies the callback with `md5(api . trx_id . invoice . msg)` (no `txn_status`). That is plugin-specific. For the documented `/merchant/api/v2/receiver` flow, use the docs formula above and test it with a real sandbox callback.

### LITE (and PRO Invoice API): HMAC-SHA256 over sorted values

From the LITE "Hashing Guide" and "Generate Hash Invoice Guide":

1. Take every parameter you send except `hash`.
2. Sort by parameter name in ascending order (PHP `ksort`).
3. Concatenate the values only (no names, no separators).
4. Remove all spaces.
5. `hash = hash_hmac('sha256', string, API_KEY)` (lowercase hex).

Docs example: `{amount:1.50, currency:MYR, invoice:INV1001, merchant_id:R10005, payment_desc:"Parking fee"}` gives `1.50MYRINV1001R10005Parkingfee`, then `hash_hmac('sha256', "1.50MYRINV1001R10005Parkingfee", "MYAPIKEY")`.

```
function liteHash(params, apiKey):
    keys = sort(params.keys() without "hash")            // byte order, like PHP ksort
    s = ""
    for k in keys:
        v = params[k]
        if v is object/array:                            // nested (recurring, invoice items)
            for child in v (in sent order):              // NOT re-sorted
                s += child is object ? concat(child values in sent order) : child
        else:
            s += string(v)
    s = s.replace(" ", "")
    return hmac_sha256_hex(key=apiKey, msg=s)
```

- The nested-object rule comes from the official Postman pre-request scripts and the WP plugin `generateHash()`. Nested values are concatenated in the order sent, not sorted.
- Every optional field you send is part of the string. The docs PHP sample sorts the whole `$_POST`.
- PHP `str_replace(' ', '', ...)` removes spaces only. Postman uses `/\s+/g`. Do not send tabs or newlines in values.
- Send amounts as strings exactly as you hash them (`"1.50"`, not `1.5`).

**LITE callback verification**: the docs say to use the Hashing Guide method over the received fields. The official WP plugin does this: take all received fields except `hash`, `ksort` them, concatenate, remove spaces, HMAC-SHA256 with the API key, then compare. The plugin also drops its own query-string parameters (`wc-api`, `cart_id`) that it added to the callback URL. Hash only the fields Betterpay POSTs. Do not include query parameters you appended yourself. Baggage fields sent back by Betterpay are included in the plugin's check.

### Verification rules (both products)

- Compare hashes with a constant-time function, after lowercasing both sides.
- Reject the callback when the hash does not match. Still log it.
- After a good hash, check that `invoice_no` exists, that `txn_amount` equals the stored amount, and that the order is not already settled.

## Endpoint Catalogue

All methods are POST. "Sandbox path" is the path on `https://www.demo.betterpay.me` (from Postman). "Not published" means the docs list the operation but no public path exists: read it from the logged-in docs.

### PRO

| Operation | Sandbox path | Purpose |
|---|---|---|
| Standard Payment | `/merchant/api/v2/receiver` | Browser form POST to the hosted checkout |
| Create Bill | `/merchant/api/v2/create_bill` | Create a bill (pay later via portal) |
| Remove Created Bill | `/merchant/api/v2/create_bill` (Postman uses the same path, body has only invoice) | Remove a bill |
| Direct Payment | `/merchant/api/v2/direct/receiver` | Skip checkout, go straight to the bank/e-wallet/card/DuitNow QR |
| Get Bank List | `/merchant/api/v2/direct/get_bank_list` | FPX/e-wallet/indo bank codes |
| Get Payment Status | `/merchant/api/v2/requery` | Requery one transaction |
| DuitNow QR (OCBC) Refund | `/merchant/api/v2/refund_dnqr` | Real-time DuitNow QR refund |
| Payment Link create / delete / list / report | `/merchant/api/v2/link/create`, `/link/delete`, `/link/list`, `/link/report` | Reusable collection links |
| Get Settlement | `/merchant/api/v2/get_settlement` | Settled transactions |
| FPX Refund API | Not published | Refund FPX (own seller ID only) |
| Get Refund Status | Not published | Status of an FPX refund batch |
| Get Transactions | Not published | Transaction list |
| Get Transaction (by invoice) | Not published | All attempts for one invoice |
| Get Callbacks | Not published | Callback delivery log |
| Get Balance | Not published | Current balance |
| Get Disburse Records | Not published | Payout records |
| Create / Update / Get All / Get Invoice API | Not published | Invoices with payment link, reminders |
| Recurring (e-Mandate), OCBC collection | see `betterpay-recurring.md` | |
| Tokenization, pre-auth, batch charge, Card Pre-authorization | see `betterpay-tokenization.md` | |

### LITE

| Operation | Sandbox path | Purpose |
|---|---|---|
| Standard Payment | `/merchant/api/v2/receiver` | Hosted checkout (form POST) |
| Direct Payment | `/merchant/api/v2/lite/direct/receiver` | Skip checkout |
| Payment Link create | `/merchant/api/v2/lite/link/create` | Collection link |
| Payment Link delete / list / report | Not published | |
| Get Payment Channels | `/merchant/api/v2/lite/channels` | Channel codes for Direct Payment |
| Get Payment Status | `/merchant/api/v2/lite/requery` | Requery |
| Get Settlement | Not published | Settled transactions |
| Recurring (card), tokenization, batch charge | see `betterpay-recurring.md`, `betterpay-tokenization.md` | |

## PRO Standard Payment (hosted checkout)

Flow: build the form server-side, compute the hash server-side, then the browser auto-submits a POST to the endpoint. The API returns no data. It shows the checkout page, or an error page (400 "Invalid format data entered", 401 "Invalid Merchant ID or API Key").

Required:

| Field | Type | Notes |
|---|---|---|
| `merchant_id` | integer | Numeric, from API Management |
| `invoice` | string | Unique per transaction, max 14 chars, no `&`, `'` or `-` |
| `amount` | string | 2 decimal places, e.g. `192.00` |
| `payment_desc` | string | Alphanumeric, no `!`, max 1000 |
| `currency` | string | `MYR`, `SGD`, `IDR`, `THB`, `PHP` |
| `hash` | string | MD5, see table |

Optional:

| Field | Notes |
|---|---|
| `buyer_name`, `buyer_email`, `phone` | phone: digits and `+` |
| `add_line_1`, `add_line_2`, `postcode`, `city`, `comment` | Alphanumeric |
| `callback_url_be` | Server callback. Host must match the URL or a Whitelist Domain in API Management. Defaults to the dashboard value |
| `callback_url_fe_succ`, `callback_url_fe_fail` | Browser return URLs. Default to the dashboard values |
| `baggage_variable` | Pipe list of your own field names, max 5000, e.g. `VARIABLE1\|VARIABLE2`. Send each as its own field. Values come back in the callback. Names must not clash with API fields. Add `installmentPeriodFilter` to set IPP periods (value such as `3,6`, default 12 months) |
| `header_title` (80), `header_email` (50), `header_phone` (50) | Checkout header text |
| `payment_method` | Comma list, no spaces: `b2c`, `b2b`, `cc`, `ewallet`, `indo_bank`, `bnpl`, `ipp`, `otc`, `dnqr` |

Amount rule (error guide): at least RM1.50 and less than RM30,000 for B2C. The amount is ringgit with 2 decimals, not sen.

## PRO Direct Payment

Same fields as Standard Payment, plus:

| Field | Required | Notes |
|---|---|---|
| `payment_method` | yes | `b2c`, `b2b`, `cc`, `ewallet` (also DuitNow QR), `indo_bank`, `bnpl`, `ipp` |
| `bank_code` | yes | FPX/e-wallet/indo bank: from Get Bank List. Card: `CC00001`. BNPL: `BNPL00001` (PaySlowSlow/Riipay), `IOUPay-PW`, `Atome`. IPP: `IPP00001`. DuitNow QR: `RPP_DUITNOWQR` with `payment_method=ewallet` |
| `agent_channel_code` | if `indo_bank` | From Get Bank List (IDR) |
| `respond` | no | `1` returns JSON with `payment_url` instead of redirecting. For DuitNow QR it also returns `duitnow_qr_image` (image URL, removed once paid or expired) and `duitnow_qr_raw_string` |
| `skip_receipt` | no | `1` skips the Betterpay receipt page |
| `qr_expiry_date_time` | no | DuitNow QR, `YYYY-MM-DD HH:MM:SS`, at least 15 min ahead |
| `terminal_label`, `store_label` | no | Max 25, shown on bank statement |
| `qr_image_size` | no | 1 to 17 |

Flow (docs): call Get Bank List, call Direct Payment, payer goes to the bank, then the Betterpay receipt page (unless `skip_receipt`), and the callback goes to your backend.

## LITE Standard And Direct Payment

LITE Standard required: `merchant_id` (e.g. `R1005`), `invoice`, `amount`, `payment_desc`, `currency` (adds `USD`), `buyer_name`, `buyer_email`, `phone`, `hash`. Optional: `add_line_1`, `add_line_2`, `postcode`, `city`, `comment`, `callback_url_be`, `callback_url_fe_succ`, `callback_url_fe_fail`, `baggage_variable` plus its fields.

LITE Direct adds:

- `bank_code` (required): from Get Payment Channels.
- `hostedCard='true'`: compulsory for card and BNPL channels.
- `respond=1`: returns JSON with `payment_url`.
- `skip_receipt`.

Response sample: `{"response":"00","merchant_id":"R1017","invoice":"INV01","amount":"2.00", ...}`.

Get Payment Channels (LITE): request `merchant_id`, `hash`. The response lists:

| Field | Meaning |
|---|---|
| `status` | `0` offline, `1` online |
| `type` | `IB`, `CC`, `EW` or `BNPL` |
| `value` | The bank code to send |
| `display_name` | Channel name |
| `logo_url` | Channel logo |
| `currencies` | Supported currencies |

The LITE error guide is the same as PRO (amount RM1.50 to under RM30,000 B2C, unique invoice of max 14 chars, callback host must match API Management).

## Return URL vs Callback

- `callback_url_fe_succ` / `callback_url_fe_fail` are browser redirects. They are UX only. Never fulfil from them. The WP plugin only redirects on them.
- `callback_url_be` receives a server-to-server **form POST** after the payer completes or cancels.
- Your endpoint must reply with plain text `OK`. Without `OK`, "Betterpay will send the callback data at most 3 times every 10 minutes".
- Make the handler idempotent. Retries and duplicate callbacks happen. Get Callbacks (PRO) shows the delivery log: `callback_url`, `data`, `status` (true/false), `attempt`, `created`.

PRO callback fields:

| Field | Notes |
|---|---|
| `fpx_fpxTxnId` / `paypal_trx_id` / `mastercard_trx_id` / `others_trx_id` | Transaction ID, depends on `pay_method` |
| `trx_txt` | Channel label when `others_trx_id` is set (e.g. `eWallet 2C2P (TNG)`, `eWallet (DuitNow QR)`) |
| `fpx_sellerId` | FPX seller ID |
| `invoice_no` | Your invoice |
| `txn_status` | See `betterpay-codes.md` (`00` approved, `09` pending, `99` pending authorization B2B) |
| `msg` | e.g. `Transaction Approved` |
| `txn_amount` | Paid amount |
| `pay_method` | `fpx`, `paypal`, `mastercard` or `others` |
| `hash` | MD5, see above |
| your baggage fields | Returned as sent |

LITE callback fields:

| Field | Notes |
|---|---|
| `bp_lite_trx_id` | Transaction ID |
| `merchant_id` | Your merchant ID |
| `invoice_no` | Your invoice |
| `txn_status` | `00` success, `11` failed, `22` pending, `failed` customer cancelled |
| `msg` | e.g. `success` |
| `txn_amount` | Paid amount |
| `pay_method` | `online_banking`, `card`, `ewallet` or `bnpl` |
| `hash` | HMAC, see above |

Settle only when the hash is valid, `txn_status` is `00`, the amount matches, and the order is not already paid. The WP plugin also checks `msg` against `Transaction Approved` / `Success` / `settled` (PRO) and `transaction approved` / `success` / `captured` / `settled` (LITE). Prefer `txn_status`.

## Requery (Get Payment Status)

Use it when no callback arrives. Do not use it as a replacement for callbacks.

PRO request: `merchant_id`, `invoice`, `method`, `hash` (`md5(merchant_id|api|invoice|method)`).

- `invoice`: the docs say to use the system Betterpay/QlicknPay reference number for an accurate result.
- `method`: `1` = payment via the gateway, `2` = payment via the WhatsApp/Email invoice.

PRO response: always `response`, `invoice`, `status_code`, `status`, `amount`. Plus extra fields per channel:

| Channel | Extra fields |
|---|---|
| FPX | `fpx_fpxTxnTime`, `fpx_fpxTxnId`, `fpx_buyerName`, `fpx_txnCurrency`, `fpx_sellerOrderNo`, `fpx_txnAmount`, `fpx_buyerBankBranch` |
| Card | `time`, `payer_first_name`, `payer_last_name`, `mastercardID`, `category_code`, `transaction_result` |
| PayPal | `transactionID`, `state`, `cartID`, `payer_email`, `payer_id`, `payer_phone` |
| Boost | `transactionType`, `transactionTime`, `boostRefNum`, `customerLast4DigitMSISDN`, `onlineRefNum` |
| Other providers | `transactionType`, `transactionTime`, `providerRefNum` |

Response codes: `00` ok, `97` pending (the docs also list `97` for invalid hash), `98` failed transaction ("Sorry, we can't requery a failed transaction"), `99` invalid invoice / invalid data.

LITE request: `merchant_id`, `invoice`, `hash` (HMAC). Response `data`: `bp_lite_trx_id`, `merchant_id`, `invoice_no`, `txn_status`, `msg`, `txn_amount`, `pay_method`.

## Refunds

**FPX Refund API (PRO)**: only for merchants with their own FPX seller ID. Max 10 transactions per request, within 5 days of the transaction. In production the merchant then approves the refund in their own bank portal (maker/checker). In sandbox, approve it on the Refund List (Bank Simulation) page.

Request fields:

| Field | Required | Notes |
|---|---|---|
| `merchant_id` | yes | |
| `merchant_bank_id` | yes | e.g. `BIMB0340` (from Get Bank List) |
| `merchant_acc_number` | yes | |
| `merchant_maker_name` | yes | Maker user in your bank portal |
| `organization_id` | yes | From your bank portal |
| `refund_list` | yes | `ref,amount;ref,amount;`. `ref` = Betterpay reference no / seller order no / invoice |
| `hash` | yes | |
| `enable_callback` | no | `1` or `0` |
| `enable_email` | no | `1` or `0` |
| `refund_callback_url` | no | Needs `enable_callback=1`. Defaults to the dashboard callback |

Response: `response` (`00`), `msg`, `ref_no` (keep it), `merchant_id`, `merchant_bank_id`, `merchant_maker_name`, `merchant_acc_number`, `refund_list`. Errors: `99` "This account is not valid to proceed with refund API", `99` "Invalid data".

Refund callback: `type=refund_api`, `ref_no`, `refund_list` (`ref,amount,status;`), `hash = md5(api + 'refund_api' + ref_no + refund_list)` with no separators.

**Get Refund Status (PRO)**: request `merchant_id`, `ref_no`, `hash`. Response: `response`, `status`, `msg`, `ref_no`, `refund_list` (`ref,amount,status`), `ref_transaction_id`. Refund status codes (B0 to C5, 1S) are in `betterpay-codes.md`.

**DuitNow QR (OCBC) Refund (PRO)**: real time. Same limits as FPX refund (max 10 per request, within 5 days). Request `merchant_id`, `refund_list` (`ref,amount;`), `hash`. Response: `response`, `merchant_id`, `refund_list` as `ref,code;` (e.g. `...AVs,C2;...AKk,00;`).

LITE refunds: Not documented in official docs (checked 2026-10-21).

## Reports (PRO)

| API | Optional request fields | Response |
|---|---|---|
| Get Transactions | `date_from`, `date_to` (yyyy-mm-dd), `status` (0 failed, 1 success, 2 both), `page` | `total_page_available`, then rows: `invoice`, `prod_id`, `total_amount`, `buyer_name`, `buyer_email`, `phone`, `address`, `city`, `state`, `postcode`, `comment`, `status`, `pay_method`, `qlicknpay_ref_no`, `time` |
| Get Transaction (by invoice, `invoice` required) | none | `response`, `merchant_id`, `invoice`, `buyer` list (one entry per attempt, same fields) |
| Get Settlement | `date_from`, `date_to`, `page`, `formated` (1 = include max page), `invoice` | rows: `method`, `date`, `invoice_no`, `amount`, `fee`, `due` |
| Get Callbacks | `max` (default 100) | `response`, `callbacks[]` |
| Get Balance | none | `response`, `merchant_id`, `fee`, `total_balance` |
| Get Disburse Records | `date_from`, `date_to`, `page`, `formated` | rows: `date_paid`, `reference_no`, `invoice_no`, `amount_paid`, `payment_to` |

- Errors: `97` "Invalid date_from format.", `99` "Verification fail".
- Lists default to 100 rows per page.
- Get Disburse Records is disabled in the sandbox (demo) account, and disburse data is available T+3 days after the transaction date (docs).

LITE Get Settlement: `merchant_id`, `hash`. Optional `date_from`, `date_to`, `page`. Same row fields as PRO.

## Payment Links, Bills, Invoices

**Create Bill (PRO)**: Standard Payment fields. Error codes `00` ok, `99` "Fail to created a bill". **Remove Created Bill**: `merchant_id`, `invoice`, `hash`.

**Payment Link create** (PRO and LITE):

| Field | Required | Notes |
|---|---|---|
| `merchant_id` | yes | |
| `purpose` | yes | Unique, max 200 |
| `amount_option` | yes | `Fixed` or `Unfixed` |
| `amount` | if `Fixed` | 2dp, over RM1.50 |
| `delivery_option` | yes | `0` or `1` (delivery form) |
| `currency` | yes (PRO) | `MYR`, `IDR` or `SGD` |
| `hash` | yes | |
| `due_date` | no | YYYY-MM-DD |
| `usage` | no | Max successful payments |
| `max_collection` | no | Max total collected |

Response: `response` (`00`/`99`), `payment_link_url`, `purpose`, `amount_option`, `amount`, `delivery_option`, `merchant_id`, `due_date`, `usage`, `max_collection`, `hash`.

- **Delete**: request `id`, `merchant_id`, `hash`. Response `id`, `merchant_id`, `response`, `msg`.
- **List**: request `merchant_id`, `hash`. Rows: `Id`, `inv_number`, `purpose`, `amount_option`, `amount`, `delivery_option`, `logo_url`, `qr_url`, `usage`, `max_collection`, `due_date`.
- **Report**: request `merchant_id`, `hash`. Optional `date_from`, `date_to`, `status` (PRO docs: 0 successful, 1 failed), `purpose`.
  - Rows: `transaction_id`, `amount_option`, `amount`, `delivery_option`, `buyer_name`, `phone`, `address`, `city`, `state`, `postcode`, `comment`, `status`, `callback_be`, `currency`, `created_at`, `logo_url`, `qr_url`, `updated_at`, `deleted_at`.

LITE Payment Link hash: the LITE docs point to the HMAC Hashing Guide. The official LITE Postman collection instead uses `md5(merchant_id|api|purpose|amount_option|amount|delivery_option)`. These conflict. Test in sandbox and confirm with support.

**Invoice API (PRO, HMAC hash like LITE)**:

Create Invoice required fields:

| Field | Notes |
|---|---|
| `merchant_id` | |
| `invoice_id` | Max 20, no spaces |
| `currencies` | Array, first entry is the default |
| `items` | Object of `{id: {description, quantity, amount}}`, max 10 items, each amount over 1.50 |
| `partial_payment_status` | boolean |
| `hash` | HMAC |

Create Invoice optional fields:

| Field | Notes |
|---|---|
| `invoice_to_name` | |
| `tax` | List of `{type: percentage\|fixed, amount}` |
| `email` | |
| `phone` | With `+` country code, used for WhatsApp |
| `reminders` | `{due_in: {days: {message}}, overdue: {days: {message}}}` |
| `description` | Max 500 |
| `note` | Max 100 |
| `paid_at` | |
| `send_at` | `YYYY-MM-DD HH:II:SS` |
| `expired_at` | `YYYY-MM-DD HH:II:SS` |
| `attachments` | List of URLs |

- **Update Invoice**: `merchant_id`, `invoice_id`, `hash`, plus any optional field.
- **Get All Invoices**: optional `date_from` / `date_to` (default the last 30 days).
- **Get Invoice**: `merchant_id`, `invoice_id`, `hash`.

## Rate Limits

- All API GET endpoints share a limit of 100 requests per 5-minute window. It is counted per IP address or per account, across all endpoints together.
- Going over returns HTTP 429 with body `{"response":"99","msg":"Rate limit exceeded ..."}`.
- Betterpay may set stricter limits for abusive use without notice.
- Rate limits for POST endpoints: Not documented in official docs (checked 2026-10-21).
- OCBC collection has its own 99-requests-per-day limit (see recurring file).

## IP Allowlist

- Betterpay checks your callback host against the URL / Whitelist Domain in API Management. A mismatch gives "Invalid URL variable specified".
- Betterpay's outbound callback IPs: Not documented in official docs (checked 2026-10-21). Rely on the hash, not on IPs.

## Sandbox Test Data

- Sandbox host: `https://www.demo.betterpay.me` (docs also show `https://www.demo.qlicknpay.com`). Sandbox and live have separate merchant IDs and API keys.
- Simulation fields (sandbox only): `sandbox_charge_status=0` (token charge, authorize, capture succeed; anything else fails), `simulation_status=SUCCESS|FAIL` (Card Pre-authorization pay/void).
- Batch token charge callbacks in sandbox must be triggered by hand: Merchant account > MPGS Tokenization > Batch Token Approval.
- EasyStore plugin sandbox: add `?testing=true` to the checkout URL.
- Published test cards or test bank logins: Not documented in official docs (checked 2026-10-21).

## Common Errors

| Message | Cause |
|---|---|
| Unable to process payment due to invalid API or invalid merchant ID or no data entered | Wrong merchant ID/API key, or empty post |
| Maximum total of amount exceeded or invalid data entered | Amount under RM1.50 or RM30,000 and above (B2C) |
| Unable to process payment due to invalid Product ID entered | `payment_desc` empty or 1000 chars or more |
| Invalid data entered | Validation failed or **hash mismatch** |
| Invalid URL variable specified | `callback_url_be` host differs from API Management. Fix: send null or use the same host |
| Invalid name of baggages | Baggage name clashes with an API field |
| Your Invoice number is too long | Over 14 chars |
| Invalid invoices | Forbidden characters (e.g. `-`) |
| Invalid invoice or duplicated invoice entered | Invoice reused |
| Indirect error | No API / plugin URL set in API Management |
