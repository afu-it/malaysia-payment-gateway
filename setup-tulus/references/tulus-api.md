# Tulus (SnapNPay) API Reference

Use this reference for factual Tulus API guidance. Tulus and SnapNPay are the same gateway run by Tulus Digital Sdn Bhd. Everything here comes from the official Tulus Developer Portal, its downloadable PHP SDK (`sdk.zip`), its agent skill file, and its Laravel handoff (checked 2026-10-21). Where official pages disagree, this file says so and names the source that wins.

## Source URLs

- `https://developers.tulus.my/` (also served at `https://developers.snapnpay.my/`)
- `https://developers.tulus.my/how-it-works/`
- `https://developers.tulus.my/SKILL.md`
- `https://developers.tulus.my/laravel-handoff.md` (rendered at `/laravel`)
- `https://developers.tulus.my/sdk/` and `https://developers.tulus.my/sdk/callback.md` (rendered at `/sdk/callback`)
- `https://developers.tulus.my/mpgs_callbackresponse.md` (rendered at `/mpgs`)
- `https://developers.tulus.my/download/sdk.zip` (`Tulus.php`, `SnapNPay.php`, examples)
- `https://developers.tulus.my/sdk/qrcode/`

## Endpoint Catalogue

| Method | URL | Auth | Purpose |
|---|---|---|---|
| POST | `https://tulus.my/v2/checkout` (documented; see Hosts) | signature (API key) | Create a payment order, hosted checkout |
| GET | `https://tulus.my/pay/{agency}/{paymentCategory}/{urlencode(refNo)}/{amount}` (documented; see Hosts) | none | Hosted pay page by URL (no form) |
| GET | `https://reports.tulus.my/api/report/v1/payment` | Bearer AuthToken or signed URL | Paginated payment report |
| GET | `https://reports.tulus.my/api/report/v1/fpxrequest?refno=` or `?fpx_seller_order_no=` | Bearer or signed URL | Single order lookup (status requery) |
| GET | `https://reports.tulus.my/api/user/v1/transactions?email=` | Bearer or signed URL | A payer's transaction history |
| POST | `https://reports.tulus.my/api/qrcode/v1/generator` | signed POST | Signed (tamper resistant) payment QR, returns URL of a PNG |
| POST | `https://reports.tulus.my/api/duitnowqr/v1/generator` | signed POST | DuitNow QR variant |
| GET | `https://tulus.my/qr2/?q=<urlencoded pay URL>` (SDK uses `https://snapnpay.my/qr2/`) | none | Unsigned QR image for a pay URL |
| GET | `https://reports.tulus.my/api/version` | none | Health / version |
| GET | `https://reports.tulus.my/api/echo` | none | Echo test |
| POST | your `returnUrl` | signature (direct only) | Direct callback and indirect browser return |

Partner inbound webhooks (ADB, Grab), receipt templates and receipt email customisation: see `tulus-partner-apis.md`.

Not documented in official docs (checked 2026-10-21): refund, cancel, void, capture, recurring, card tokenisation, payout/disbursement and settlement report APIs. Do not invent them; ask Tulus.

## Hosts

| Use | Documented | Observed 2026-10-21 |
|---|---|---|
| Checkout POST | `https://tulus.my/v2/checkout` (SKILL.md, how-it-works, Laravel handoff) | `tulus.my/v2/checkout` returned 404 (GET and empty POST). `https://pay.tulus.my/v2/checkout` returned 200 with "Bad request ... Invalid Agency Code V2" for an empty POST. |
| SDK playground order form | Production `https://pay.tulus.my/`, Staging `https://paydev.tulus.my/` (FPX test banks, no real money) | both answer 200 |
| Legacy | `https://snapnpay.my/v2/checkout` (WooCommerce plugin, `payment.php`) | answers 200 |
| Hosted pay URL | `https://tulus.my/pay/...` | `tulus.my/pay/...` 404; `pay.tulus.my/pay/...` and `snapnpay.my/pay/...` 200 |
| Unsigned QR | `https://tulus.my/qr2/` (SKILL.md), `https://snapnpay.my/qr2/` (SDK) | `tulus.my/qr2/` 404; `snapnpay.my/qr2/` and `pay.tulus.my/qr2/` 200 |
| Reporting | `https://reports.tulus.my` | `/api/version` returns `{"Version":"..."}` |

`tulus.my` now serves the marketing site, which is likely why the documented checkout path 404s. Make the checkout URL a config value (`TULUS_CHECKOUT_URL`) and confirm the production and staging checkout URLs with Tulus (support@tulus.my) before go-live. The signed message path for checkout is `/v2/checkout` in all official signing examples.

## Authentication

Two supported methods; the third (`token`, username+password, `GetToken`) is deprecated and the SDK throws `DeprecatedException`.

1. AuthToken (Bearer JWT). Log in to `https://reports.tulus.my/`, open the top-right menu (below Logout), choose "Copy AuthToken to Clipboard". Send `Authorization: Bearer <AUTHTOKEN>`. Can span several related agencies. Preferred for reporting. Cannot sign orders.
2. API key (HMAC). A per-agency 128-bit UUID, format like `XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX`, from the merchant dashboard API keys page. Required for signed orders, signed QR, and callback verification. Fixed to one agency: always send the `agency` field. The key never travels; only the 16-hex signature does.

## Signature Algorithm

Same HMAC for everything; only the message differs.

1. `key = hex_decode(api_key.replace("-", ""))` (16 raw bytes; equals Go `uuid.MarshalBinary`, PHP `pack("H*", ...)`).
2. `signature = hex(HMAC_SHA256(key, message))[0:16]` (lowercase hex, first 16 characters).

| Use | Message | `exp` |
|---|---|---|
| Checkout POST | `/v2/checkout?` + ksorted fields, RFC 3986 encoded | add `exp = unix_now + 60` as a field, signed and posted |
| Report / lookup GET | `<path>?` + ksorted query, RFC 3986 encoded | add `exp`, append `&signature=` |
| QR generator POST | `<path>?` + ksorted fields, RFC 3986 encoded (SDK `SignMessage`) | add `exp`; body is the query string plus `&signature=` |
| Verify direct callback | ksorted POST fields except `signature`, `k=v&k=v`, Go `url.QueryEscape` encoding, no path | none, verify exactly what arrived |

Doc inconsistency: SKILL.md and how-it-works say outbound POST messages have "no path". The SDK `SignMessage`, the WooCommerce plugin `SignURLParams`, and the Laravel handoff (which states it wins over older examples) all include the path, and the official checkout test vector below only reproduces with the path. Include the path.

Encoding details:

- Outbound: RFC 3986 (PHP `http_build_query(..., PHP_QUERY_RFC3986)`, Python `urlencode(..., quote_via=quote)` with `safe=''`). Space is `%20`, `~` stays literal, `@ : /` are encoded.
- Callback: Go `url.QueryEscape` semantics. Space is `+`; `A-Z a-z 0-9 - _ . ~` stay literal; everything else is `%XX` uppercase. Python `urllib.parse.urlencode` (default `quote_plus`) matches. PHP `urlencode` matches except it encodes `~` as `%7E` while Go leaves it (the Laravel handoff notes this). `extraData` in callbacks can contain `~` (official example), so in PHP use `str_replace('%7E', '~', urlencode($v))`.

### Official test vectors

Key (test value published in the docs): `DACA31D4-869A-4323-9455-E4F533EB08DC`. Both vectors were reproduced locally with Python on 2026-10-21.

Direct callback. Message:

```text
addCharge=1.00&agency=SNAPNPAY&amount=2.50&extraData=881225-08-1234&fpx=%7B%22fpx_seller_order_no%22%3A%22SNAPNPAYd8d7674c50%22%2C%22fpx_debit_auth_code%22%3A%2200%22%7D&fpxTxnId=2503061436060451&fpxTxnTime=20250306143605&orderNo=SNAPNPAYd8d7674c50&refNo=ORDER123&status=success
```

Expected signature: `1fed398f127653fd`.

Checkout. Fields `agency=snapnpay`, `refNo=ORDER123~881225-08-1234`, `amount=1.00`, `email=payer@example.com`, `returnUrl=https://shop.example.com/tulus/return`, `exp=1800000000`. Message:

```text
/v2/checkout?agency=snapnpay&amount=1.00&email=payer%40example.com&exp=1800000000&refNo=ORDER123~881225-08-1234&returnUrl=https%3A%2F%2Fshop.example.com%2Ftulus%2Freturn
```

Expected signature: `8ba7befa130c872b`.

### Pseudo-code

```python
import hmac, hashlib, time
from urllib.parse import urlencode, quote

def key_bytes(api_key):            # UUID -> 16 raw bytes
    return bytes.fromhex(api_key.replace("-", ""))

def sign_checkout(fields, api_key, path="/v2/checkout"):
    f = dict(fields, exp=str(int(time.time()) + 60))
    msg = path + "?" + urlencode(sorted(f.items()), quote_via=quote, safe="")
    f["signature"] = hmac.new(key_bytes(api_key), msg.encode(), hashlib.sha256).hexdigest()[:16]
    return f                        # POST all of these, nothing else

def sign_get(path, params, api_key):  # reports.tulus.my
    p = dict(params, exp=str(int(time.time()) + 60))
    msg = path + "?" + urlencode(sorted(p.items()), quote_via=quote, safe="")
    sig = hmac.new(key_bytes(api_key), msg.encode(), hashlib.sha256).hexdigest()[:16]
    return "https://reports.tulus.my" + msg + "&signature=" + sig

def verify_callback(form, api_key):
    got = form.get("signature", "")
    if not got:
        return False
    fields = {k: v for k, v in form.items() if k != "signature"}
    msg = urlencode(sorted(fields.items()))        # space -> '+', '~' literal (Go QueryEscape)
    want = hmac.new(key_bytes(api_key), msg.encode(), hashlib.sha256).hexdigest()[:16]
    return hmac.compare_digest(want, got.lower())
```

Verify over the fields exactly as received (parse the raw form body; do not drop unknown fields such as `_tracing` seen in the MPGS example, and do not trim or reformat values).

## Create a Payment Order (Checkout)

`POST` (browser auto-submit form) to the checkout URL, `Content-Type: application/x-www-form-urlencoded`. All values strings, ASCII, at most 255 bytes; UTF-8 is undefined behaviour.

| Field | Required | Notes |
|---|---|---|
| `agency` | yes | Agency code from onboarding (for example `snapnpay`). Case-insensitive on input; callbacks return it uppercase. |
| `refNo` | yes | Your unique order reference. No `~` in the reference itself. Optional extraData is appended as `refNo~extra1~extra2`. |
| `amount` | yes | Ringgit, string, 2 decimals, for example `"1.00"`. Min `1.00` FPX B2C, `2.00` FPX B2B (after charges, per SDK page). |
| `email` | yes | Payer email; receipt is sent here. |
| `returnUrl` | yes | Public HTTPS URL receiving both direct and indirect POSTs. Private and loopback addresses are rejected in production. |
| `cancelUrl` | no | Where the buyer goes if they cancel on the Tulus checkout page before the request reaches the bank/FPX. After that, cancels and failures come back via `returnUrl`. |
| `exp` | yes when signing | `unix_now + 60`, included in the signed message and posted. |
| `signature` | yes in production | 16-hex HMAC. Unsigned or wrongly signed checkouts get HTTP 403. The SDK page still says the key is "optional but recommended to sign and lock in the payment amount"; the Laravel handoff (newer) says production rejects unsigned orders. Always sign. |
| `utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content` | no | Tracking fields shown in the SDK playground. |
| `pref_method` | no | [ALPHA] preferred method, for example `fpx/b2b1`. |
| `pref_bank` | no | [ALPHA] preferred bank code, for example `ABB0235` (Affin Bank B2B). |

Rules: render the form at redirect time (60-second expiry; cached pages fail 403). Post only signed fields (no CSRF token). Amount must come from your own records.

Hosted pay URL alternative (no form, no signature in the documented form): `/pay/{agency}/{paymentCategory}/{urlencode(refNo)}/{amount}`, for example `/pay/snapnpay/general/ORDER123/1.00`. The SDK builds it on `https://snapnpay.my/pay/` (`GetPaymentV2Url`). Because it is unsigned, the amount is not locked; prefer the signed POST.

Response: the buyer's browser lands on the Tulus hosted checkout (bank, DuitNow, card selection). There is no JSON response to parse. An empty or invalid POST returns an HTML "Bad request, some required fields are missing or invalid." page.

## Callbacks to returnUrl

Tulus delivers the result twice to the same `returnUrl`, both `POST application/x-www-form-urlencoded`. Tell them apart by the presence of `signature`.

### Direct callback (server-to-server, signed, authoritative)

Sent when the transaction reaches a terminal or pending state. Respond HTTP 2xx quickly (no redirect); otherwise retried on the next monitor cycle (about 30 seconds). Staff can also re-send it from the dashboard (Transactions, "CB Logs"), so the same success can arrive more than once.

| Field | Type | Notes |
|---|---|---|
| `status` | string | `success`, `failed`, `pending` |
| `orderNo` | string | Seller order number, agency prefix + hash, for example `SNAPNPAYd8d7674c50`. Seen empty in the MPGS example. |
| `refNo` | string | Your `refNo` as submitted (callback example shows it without the `~extra` part, which is in `extraData`) |
| `amount` | string | Net amount in MYR, excludes platform fee, 2 decimals |
| `addCharge` | string | Platform fee on top, 2 decimals, `0.00` if none. Buyer paid `amount + addCharge`. |
| `fpxTxnId` | string | PayNet FPX transaction id (16 digits). Empty for non-FPX (card example shows a session id). |
| `fpxTxnTime` | string | `YYYYMMDDHHmmss`, UTC+8. Empty for non-FPX. |
| `agency` | string | Agency code, uppercase |
| `extraData` | string | Your extra data, returned as-is (often `~`-delimited) |
| `fpx` | string | JSON string with full detail (below) |
| `signature` | string | 16 hex chars |

`fpx` JSON fields: `fpx_seller_order_no`, `fpx_seller_ex_order_no`, `fpx_seller_txn_time` (`YYYY-MM-DD HH:MM:SS` Malaysia time; before 2026-01-30 it could carry a trailing `+0800 UTC+8`), `ref_no`, `amount` (net), `fpx_txn_amount` (gross), `add_charge` (number), `agency`, `fpx_fpx_txn_id`, `fpx_fpx_txn_time` (compact `YYYYMMDDHHmmss`), `fpx_debit_auth_code`, `fpx_debit_auth_code_desc`, `fpx_buyer_bank_id`, `fpx_buyer_bank_name`, `fpx_buyer_email`, `fpx_buyer_name`, `extra_data`, `payment_method` (for example `fpxpay`, `mpgs`; Laravel handoff also cites `fpx.b2c`, `fpx.b2b1`), `payment_status`, `order_chan` (for example `web`, `app`, `my.snapnpay.tulus.ios`). The MPGS card example also shows `akaun_no` and `payment_category`.

For card (MPGS) callbacks, Tulus says the fields that matter are `agency`, `refNo`, `amount`, `status`, `fpx.payment_method`; the rest mirror legacy apps.

Status mapping (from `fpx_debit_auth_code`): `00` is `success`; empty, `09`, `76`, `99` are `pending`; everything else is `failed`. Full table in `tulus-codes.md`. A `pending` may later become `success` or `failed`.

### Indirect return (buyer browser, unsigned, display only)

An auto-submitting hidden form in the buyer's browser posts to `returnUrl`. Always POST; a GET means a bug (report it to Tulus).

| Field | Notes |
|---|---|
| `status` | `success` or `failed` only |
| `orderNo`, `refNo`, `agency` | as direct |
| `amount` | Gross (includes fee). Differs from direct. |
| `addCharge` | Plain decimal (`0`, `1`, `0.5`) |
| `fpxTxnId` | as direct |
| `extraData` | Omitted entirely when empty |
| `fpx` | Only for agencies with JSON response injection enabled (ask Tulus) |

Render the receipt from your database, not from these values; poll your own status endpoint until the direct callback lands (it can arrive before or after the indirect return).

### Settlement checklist (direct callback)

1. `signature` present and valid, else 403.
2. `status` in `success|pending|failed`, else 400.
3. Look up local payment by `refNo` under a row lock; unknown `refNo`: log, do not fulfil.
4. Signed `amount` (net) equals stored amount; `agency` matches.
5. Never move `success` back to another state.
6. On `success` with no `fulfilled_at`, fulfil once. Answer 200.

## Report API (reconciliation and status requery)

Host `https://reports.tulus.my`. Auth: `Authorization: Bearer <AUTHTOKEN>`, or API key signed URL with `agency` in the query.

### GET /api/report/v1/payment

| Param | Notes |
|---|---|
| `output` | `json` (report schema also supports CSV or Excel serialisation) |
| `items` | Page size; `-1` for all; SDK default 20; SDK example uses 5000 "maximum" |
| `page` | 0-based |
| `seller_order_no` | Filter by `fpx_seller_order_no` |
| `start`, `end` | ISO date, UTC+8, filter on `created_date` |
| `is_approved` | `0` or `1` |
| `is_download` | `0` or `1`; SDK says pick `0` |
| `view` | `default` |
| `agency` | Required with API key auth |

Response: `{"status":"OK","data":[...],"stats":{"total_pages":N}}`. Iterate `page` from 0 until a page returns fewer rows than `items`. Row fields used in SDK examples: `ref_no`, `fpx_seller_order_no`, `fpx_txn_amount`, `fpx_buyer_email`, `fpx_debit_auth_code` (`00` approved), `fpx_seller_txn_time`. 401/403 means bad credentials (SDK throws `LoginException`).

### GET /api/report/v1/fpxrequest

Query `refno=<refNo>` or `fpx_seller_order_no=<orderNo>` (plus `agency` and signature with API key). Response `{"status":"OK","fpxrequest":{... "fpx_debit_auth_code":"00","payment_status":"success" ...}}`. Map `fpxrequest.payment_status` the same as callback `status`. Treat `NOTFOUND` as not paid.

Example signed lookup (Laravel handoff): `signUrl(report_url + "/api/report/v1/fpxrequest?refno=" + rawurlencode(refNo) + "&agency=" + agency)`. Schedule it (for example every 10 minutes) for payments still `created` or `pending`.

### GET /api/user/v1/transactions

Query `email=<payer email>` (plus `agency` and signature with API key). Returns the payer's transaction history as JSON.

## QR Codes

Fields (SDK arrays use these keys): `agency`, `paymentCategory` (alphanumeric, no spaces, for reporting, for example `compound`), `refno` (unique per agency, no `~`; SDK also accepts `refnoFull` which replaces `refno` when sent), `amount` (ringgit, numeric, no commas, `>= 1.00`), optional `extras` / `extra` map. The QR page also lists `extradata` (no `~`), `extrafragment`, and an "order is unique" toggle. Note the SDK uses lowercase `refno` for QR while checkout uses `refNo`.

- Unsigned: `https://snapnpay.my/qr2/?q=<urlencode(https://snapnpay.my/pay/{agency}/{paymentCategory}/{refno}/{amount})>` (SDK `GetQRCodeUrl`).
- Signed: POST signed form to `/api/qrcode/v1/generator` (`GetSecureQRCodeUrl`) or `/api/duitnowqr/v1/generator` (`GetDuitNowQRCodeUrl`); returns a URL serving the QR PNG. Non-200 means failure.
- The legacy QR page mentions a development "security token" `90069AAB1234`; that belongs to the deprecated token method. Use the API key.

## Amount Rules

- Ringgit (not sen), string with 2 decimals, `.` separator, no commas.
- Minimum `1.00`; FPX B2B `2.00`.
- Direct callback `amount` net, indirect `amount` gross, `fpx.fpx_txn_amount` gross.

## Sandbox / Staging

- Staging checkout: `https://paydev.tulus.my/` with FPX test banks (MEPS), no real money (SDK playground).
- No public staging credentials, test agency, or test bank logins are published (checked 2026-10-21). Ask Tulus for staging access.
- Live test payments must end in `12.34` (for example RM 12.34) so reconcilers can identify them.
- Use a tunnel (cloudflared or ngrok) for a public HTTPS `returnUrl` during development.

## Rate Limits, IP Allowlist

Not documented in official docs (checked 2026-10-21). The legacy WooCommerce plugin resolved hosts `prod.snapnpay.co`, `snapnpay.my`, `dev.snapnpay.co`, `reports.snapnpay.my` for an IP check; treat that as legacy and rely on the signature.

## Common Errors

| Symptom | Cause |
|---|---|
| Checkout 403 | Wrong key bytes, wrong encoding, path missing from message, expired `exp`, or an unsigned extra field (for example `_token`) |
| Checkout "Invalid Agency Code V2" | Missing or unknown `agency` |
| Callback signature mismatch | Added `exp` or path, used RFC 3986 instead of QueryEscape, `~` encoded as `%7E`, verified the parsed-and-reformatted values, dropped a field |
| Callback retried every ~30 s | Your handler did not return 2xx (CSRF 419, redirect, slow work) |
| Double fulfil | Non-idempotent handler; retries and dashboard re-sends repeat success |
| Report 401/403 | Expired or wrong AuthToken, or bad signature |

## SDK Notes

- `Tulus.php` (namespace `Tulus`, current) methods: `SetAuthToken`, `SetApiKey($agency, $apikey)`, `GetPaymentReport`, `GetFpxRequestReport($refno)`, `GetFpxRequest($orderNo)`, `GetTransactionHistory($email)`, `GetSecureQRCodeUrl`, `GetDuitNowQRCodeUrl`, `GetQRCodeUrl`, `GetPaymentV2Url`, `SignURL`, `SignMessage`, `TestVersion`, `TestEcho`. `SnapNPay.php` is the older class.
- The SKILL.md and how-it-works pages show `GetCheckoutUrl` and `VerifyCallback`; neither exists in the `sdk.zip` downloaded 2026-10-21. Implement checkout signing and callback verification yourself (pseudo-code above).
- `Tulus.php` sets `strict_ssl = false` (disables TLS peer verification) by default; set it to `true` in production code.
