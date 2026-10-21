# bayarlah Upay Web API Reference

Use this reference for factual bayarlah API guidance. It is distilled from the full public docs at `https://doc.bayarlah.my/` (checked 2026-10-21). Field names, lengths and examples are copied from the docs. Where the docs are silent, this file says "Not documented in official docs (checked 2026-10-21)".

## Source URLs

- `https://doc.bayarlah.my/llms.txt` and `https://doc.bayarlah.my/llms-full.txt` (index and full text)
- `https://doc.bayarlah.my/introduction-to-terms`
- `https://doc.bayarlah.my/upay-web-api/the-integration-process`
- `https://doc.bayarlah.my/upay-web-api/signature-mechanism`
- `https://doc.bayarlah.my/upay-web-api/callback-signature-verification`
- `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/*` (activation, check-in, CPM pay, pre-order, redirect, refund, cancellation, query, callback)
- `https://doc.bayarlah.my/more-payment-function-interfaces/mini-program-plugin-api-wechat-only`
- `https://doc.bayarlah.my/more-payment-function-interfaces/terminal-order-push`
- `https://doc.bayarlah.my/event-order-notification`

Markdown of any page: append `.md` to the page URL.

## What bayarlah is

bayarlah (Beez Fintech Sdn Bhd) is a merchant acquirer that "internally integrates PayNet". The Upay Web API supports B-to-C (cashier scans the customer's payment code) and C-to-B (customer scans a merchant QR). Channels listed in the overview: DuitNow, WeChat (cross-border), Alipay+ (coming soon). The query example also returns `payway_name: "SHOPEE"` (`payway "1021"`), showing that DuitNow QR orders report the wallet actually used.

Channel support per interface, as titled in the docs:

| Interface | Channels |
| --- | --- |
| Pre-order `precreate` | DuitNow (`payway "1000"`) and WeChat (`"3"`) |
| Payment code `pay` (CPM) | WeChat only |
| Redirect gateway (WAP) | WeChat only |
| Refund | "WeChat & Card" |
| Cancel / revoke | WeChat only |
| Mini Program plugin | WeChat only |
| Query, callback | all orders |

DuitNow refunds: Not documented in official docs (checked 2026-10-21).

## Domains

| Purpose | URL |
| --- | --- |
| API domain `{api_domain}` | `https://api.bayarlah.net` |
| WAP redirect gateway | `https://qr.bayarlah.net/gateway` |
| QR image host (returned in responses) | `https://pay.bayarlah.net/upay/qrcode?content=...` |
| Merchant platform (generate activation codes) | `https://s.bayarlah.net` |

Sandbox or test environment: Not documented in official docs (checked 2026-10-21). Ask bayarlah technical support for test parameters.

## Endpoint catalogue

| # | Method | Path | Signed with | Purpose |
| --- | --- | --- | --- | --- |
| 1 | POST | `{api_domain}/terminal/activate` | `vendor_sn` / `vendor_key` | Activate a terminal, get `terminal_sn` + `terminal_key` |
| 2 | POST | `{api_domain}/terminal/checkin` | `terminal_sn` / `terminal_key` | Rotate the terminal key (daily) |
| 3 | POST | `{api_domain}/upay/v2/pay` | terminal | B-to-C: charge a scanned WeChat payment code |
| 4 | POST | `{api_domain}/upay/v2/precreate` | terminal | C-to-B: pre-create order, get QR (DuitNow / WeChat), or WAP / Mini Program pay data |
| 5 | GET (302) | `https://qr.bayarlah.net/gateway?QUERY` | query `sign` (MD5 uppercase, terminal key) | WeChat in-browser WAP payment page |
| 6 | POST | `{api_domain}/upay/v2/refund` | terminal | Full or partial refund |
| 7 | POST | `{api_domain}/upay/v2/cancel` | terminal | Automatic cancellation (reversal when pay result unknown) |
| 8 | POST | `{api_domain}/upay/v2/revoke` | terminal | Manual cancellation (same-day full refund, no fee) |
| 9 | POST | `{api_domain}/upay/v2/query` | terminal | Order status |
| 10 | POST (to merchant) | merchant `notify_url` | bayarlah RSA key (SHA256WithRSA) | Payment success callback |
| 11 | POST | `{api_domain}/appPush/tradePush` | terminal | Push an order to a bayarlah hardware terminal (for example T9) |
| 12 | POST (to merchant) | merchant URL configured by bayarlah | `MD5(body + app_secret)` | Event Order Notification (real-time push of every transaction) |
| 13 | (WeChat plugin) | Mini Program plugin `wx55540b288c5ce319` | MD5 uppercase param signature | WeChat Mini Program payments |

HTTP method: the signature page example uses `curl --request POST` for `/upay/v2/pay`; the docs do not state the method per endpoint, so use POST with a JSON body for all `{api_domain}` calls.

The signature page lists only activate, checkin, pay, precreate and query under "signed with terminal". Refund, cancel, revoke and tradePush each say "refer to the signature mechanism documentation"; sign them with the terminal pair as well.

## Integration process

1. Terminal activation (once per terminal).
2. Terminal check-in (optional, recommended daily).
3. Transactions (pre-create, pay, query, refund, cancel).

Concepts (from Introduction to Terms):

- **vendor**: developer, brand merchant or ISV. Gets `vendor_sn`, `vendor_key` and `app_id` from bayarlah support. Vendor parameters "remain unchanged and are recommended to be hard-coded" (store them as secrets).
- **app_id**: number bayarlah assigns per application type (for example store cashier vs online shop).
- **store_sn**: store number. For online scenarios, define each scenario as a store (for example "ZARA Mini Program Payment").
- **code**: activation code, generated per store in `https://s.bayarlah.net`. Valid 7 days, limited number of uses.
- **terminal**: anything that transacts: cash register, app, or website. Multiple terminals per store.
- **device_id**: your identifier for the terminal, unique under the `app_id`. Suggested format "Brand + Store + POS no." (for example `ZARA001P01`). Do not reuse a `device_id` for multiple activations.

## Request signature (all `{api_domain}` calls)

Rules from the Signature Mechanism page:

- Body is JSON, UTF-8, header `Content-Type: application/json`.
- The UTF-8 byte stream of the HTTP body is the signed content; its format does not matter.
- `sign = MD5( CONCAT( body + key ) )`.
- Header: `Authorization: sn + " " + sign` (a single space).
- `sn` is `vendor_sn` for activation, otherwise `terminal_sn`. `key` is `vendor_key` or `terminal_key`.
- After a check-in, sign with the `terminal_sn` and `terminal_key` returned by check-in.
- All amounts are in cents. All returned data are strings.

Hex case of the MD5 for API requests: Not documented in official docs (checked 2026-10-21). The official Python sample for Event Order Notification uses lowercase `hexdigest()`; use lowercase and confirm with bayarlah if you get `ILLEGAL_SIGN`.

```js
// Node.js sketch
import crypto from "node:crypto";

async function bayarlahPost(path, payload, sn, key) {
  const body = JSON.stringify(payload);          // sign exactly these bytes
  const sign = crypto.createHash("md5")
    .update(Buffer.concat([Buffer.from(body, "utf8"), Buffer.from(key, "utf8")]))
    .digest("hex");
  const res = await fetch(`https://api.bayarlah.net${path}`, {
    method: "POST",
    headers: { "Content-Type": "application/json", Authorization: `${sn} ${sign}` },
    body,
  });
  return res.json();
}
```

## Response envelope

```json
{ "result_code": "200", "biz_response": { "result_code": "PRECREATE_SUCCESS", "data": { } } }
```

- Top-level `result_code`: `"200"` when the call was processed; `"400"` with top-level `error_code` / `error_message` for request errors (example: `INVALID_PARAMS`, "total_amount (The amount must be an integer, no more than 10 digits in length, and specified in cents)").
- `biz_response.result_code`: business result (see enums below). Business errors put `error_code` and `error_message` inside `biz_response`.
- Activation and check-in return fields directly in `biz_response` (no `data`).

## 1. Activation `/terminal/activate`

Signed with `vendor_sn` / `vendor_key`.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `app_id` | string | Y | App ID |
| `code` | string | Y | Activation code |
| `device_id` | string(128) | Y | Your ID, unique under the same `app_id` |
| `client_sn` | string(50) | N | Third-party terminal number, unique under `app_id` |
| `name` | string(128) | N | Terminal name |
| `os_info` | string(45) | N | For example `Android 5.0` |
| `sdk_version` | string(45) | N | SDK version |

Response (`biz_response`): `terminal_sn` (string, Y), `terminal_key` (string, Y).

Notes:

- Call once per terminal; persist the result. The developer must track whether a terminal is already activated.
- A shared activation code can activate several terminals; each call returns a different terminal number and key.
- Duplicate `device_id` under the same merchant fails; change it or ask bayarlah to unbind.
- Signature error during activation: check parameters and that the request is signed with vendor credentials.
- Activation is required for pre-order (C scans B) and WAP payments.

## 2. Check-in `/terminal/checkin`

Signed with the current valid `terminal_sn` / `terminal_key`.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `terminal_sn` | string | Y | Terminal number |
| `device_id` | string | Y | Device unique ID (same as activation) |
| `os_info` | string | N | System info |
| `sdk_version` | string | N | SDK version |

Response (`biz_response`): `terminal_sn`, `terminal_key`.

- Recommended: check in automatically on the first core payment call of each calendar day.
- After a successful check-in the old key is invalid. An expired key causes signature errors on payment calls.
- If no response arrives (network), retry check-in with the old key. After two failed attempts with no response, contact support.
- Check-in failure usually means a device change; apply for a new activation code.

## 3. Payment code pay (CPM, WeChat only) `/upay/v2/pay`

B-to-C: cashier scans the customer's WeChat payment code.

| Field | Type | Required | Notes / example |
| --- | --- | --- | --- |
| `terminal_sn` | String(32) | Y | Numeric string up to 32 digits |
| `client_sn` | String(32) | Y | Merchant order no., unique, max 32 bytes |
| `total_amount` | String(10) | Y | Cents, max 10 digits, `"1000"` |
| `payway` | String | N | `3` = WeChat. If set, payment-code channel detection is skipped |
| `dynamic_id` | String(32) | Y | Scanned barcode content |
| `subject` | String(64) | Y | Short description, shown in WeChat after payment |
| `operator` | String(32) | Y | Store operator |
| `description` | String(255) | N | Product description |
| `longitude` | String | N | |
| `latitude` | String | N | |
| `device_id` | String | N | Device fingerprint |
| `extended` | JSON map | N | Up to 24 fields, key max 64 bytes, value max 256 bytes |
| `reflect` | String(64) | N | Passed back unchanged |
| `notify_url` | String(128) | N | Callback URL; notify intervals 1s, 5s, 30s, 600s |

`goods_details` items (documented on this page): `goods_id` String(32) required, `goods_name` String(32) required, `quantity` Number(10) required, `price` Number(10) required (cents).

Response `biz_response.data`:

| Field | Type | Notes |
| --- | --- | --- |
| `result_code` | String | In `biz_response`: `PAY_SUCCESS`, `PAY_IN_PROGRESS`, `PAY_FAIL` |
| `error_code`, `error_message` | String | On failure |
| `terminal_sn` | String(32) | |
| `sn` | String(16) | bayarlah order number |
| `client_sn` | String(32) | |
| `status` | String(32) | Transaction status (`SUCCESS`, `IN_PROG`, `FAIL_CANCELED`) |
| `order_status` | String(32) | `PAID`, `CREATED`, `PAY_CANCELED` ... |
| `payway`, `payway_name` | String | |
| `sub_payway` | String(2) | |
| `payer_uid`, `payer_login` | String | WeChat payer |
| `trade_no` | String | Channel transaction number (in examples) |
| `total_amount` | String(10) | |
| `net_amount` | String(10) | `total_amount` minus refunds |
| `settlement_amount` | String(10) | Amount of this operation |
| `subject` | String(64) | |
| `finish_time` | String(13) | ms timestamp, only when final |
| `channel_finish_time` | String(13) | ms timestamp, only on success |
| `operator` | String(32) | |
| `reflect` | String(64) | |

Examples also include `client_tsn`, `ctime`, `store_id` and `payment_list` (array of `{type, amount_total}`, for example `BANKCARD_DEBIT`, `DISCOUNT_CHANNEL_MCH`).

Notes:

- Customer has about 40 seconds to enter a password; then it times out and you must re-initiate.
- If the final `order_status` is not returned, call query. If still unknown, call `cancel`.
- A signature error here can mean a single-terminal activation code was used on several terminals.
- Add a manual "query" button at the cashier for unclear results.

## 4. Pre-order `/upay/v2/precreate` (main online flow)

| Field | Type | Required | Notes / example |
| --- | --- | --- | --- |
| `terminal_sn` | String(32) | Y | |
| `client_sn` | String(32) | Y | Unique, max 32 bytes; new value for each retry after failure |
| `total_amount` | String(10) | Y | Cents, max 10 digits. Over RM100 million use bank transfer |
| `payway` | String | Y | `"1000"` DuitNow, `"3"` WeChat |
| `sub_payway` | String | N | For WAP: `"2"` dynamic QR, `"3"` static QR (not DuitNow), `"4"` Mini Program |
| `payer_uid` | String(64) | N | WeChat WAP: pass `open_id` (not DuitNow) |
| `subject` | String(64) | Y | Transaction overview |
| `operator` | String(32) | Y | Operator |
| `description` | String(255) | N | |
| `longitude`, `latitude` | String | N | Must be sent together |
| `extended` | String(256) | N | JSON, up to 24 fields. Mini Program scan-to-order requires `{"attach": "OrderSource=FoodOrder"}` |
| `goods_details` | JSON | N | Array of `{goods_id, goods_name, quantity, price}` (`price` Number(9), cents) |
| `reflect` | String(64) | N | Passed back unchanged |
| `notify_url` | String(128) | N | Required if you want success callbacks; intervals 1s, 5s, 30s, 600s |

Response `biz_response.data`:

| Field | Type | Notes |
| --- | --- | --- |
| `sn` | String(16) | bayarlah order no., for example `beez59246065254` |
| `client_sn` | String(64) | |
| `trade_no` | String(64) | Channel voucher number (optional) |
| `status` | String(32) | Transaction status (`IN_PROG` in example) |
| `order_status` | String(32) | `CREATED` |
| `payway` | String(2) | |
| `sub_payway` | String(2) | |
| `qr_code` | String(128) | EMV QR payload, render it yourself |
| `qr_code_image_url` | String(128) | Ready QR image URL on `pay.bayarlah.net` |
| `total_amount` | String(10) | |
| `net_amount` | String(10) | Equals `total_amount` |
| `subject` | String(64) | |
| `operator` | String(32) | |
| `reflect` | String(64) | |
| `wap_pay_request` | String(1024) | Returned for WAP and Mini Program payments |

The API returns only the QR value and URL; you render the image.

Getting the result (docs recommend polling plus callback together):

- Start polling after `PRECREATE_SUCCESS` or `order_status = CREATED`.
- Payment window is about 4 minutes. Documented schedule: 0 to 1 min at 10s, 5s, 5s, 10s, 10s, 10s, 10s; 1 to 2 min every 20s; 2 to 4 min every 30s; final poll at 6 minutes.
- The Integration Process page gives a simpler schedule: every 2s for the first 30s, then every 5s.
- Stop polling once polling or a callback has given the result.

## 5. Redirect payment gateway (WAP, WeChat only)

Prerequisite: bayarlah sales must file service-provider and test-merchant network applications; WeChat parameters must match yours. The terminal must already be activated.

Request: `GET https://qr.bayarlah.net/gateway?QUERY`, sent as a 302 redirect.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `terminal_sn` | String(32) | Y | |
| `client_sn` | String(32) | Y | Unique |
| `total_amount` | String(10) | Y | Cents |
| `subject` | String(64) | Y | |
| `payway` | String | N | `3` WeChat; default is the current client environment |
| `operator` | String(32) | Y | |
| `description` | String(255) | N | |
| `longitude`, `latitude` | String | N | Together |
| `extended` | String(256) | N | JSON string |
| `goods_details` | String | N | JSON array as a string |
| `reflect` | String(64) | N | |
| `notify_url` | String(128) | N | Server callback, intervals 1s, 5s, 30s, 600s |
| `return_url` | String(128) | Y | Browser return page |
| `sign` | String(32) | Y | See below |

Gateway signature rules:

1. Take all request parameters except byte-type ones (files, byte streams) and except `sign` and `sign_type`.
2. Sort by key in ASCII ascending order (character by character).
3. Join as `key=value` with `&` to form `stringA`.
4. Append `&key=<terminal_key>`.
5. `sign = MD5(stringSignTemp).toUpperCase()`.

```text
stringA        = "client_sn=123&terminal_sn=123&total_amount=1"
stringSignTemp = stringA + "&key=<TERMINAL_KEY>"
sign           = MD5(stringSignTemp).toUpperCase()
```

Official PHP sample:

```php
$paramsStr = "client_sn=test&operator=TEST&return_url=test&subject=TEST&terminal_sn=test&total_amount=3";
$sign = strtoupper(md5($paramsStr.'&key=test'));
header("Location:https://qr.bayarlah.net/gateway?".$paramsStr."&sign=".$sign);
```

The sample signs raw (not URL-encoded) values. URL-encoding of values before signing: Not documented in official docs (checked 2026-10-21). Empty-value handling: not stated for the gateway (the Mini Program rule says non-empty only).

`return_url` parameters (reference only; rely on query or callback):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `is_success` | String(1) | Y | `T` / `F`: API call success, not business result |
| `error_code`, `error_message` | String | N | Only when `is_success = F` |
| `terminal_sn` | String(32) | Y | |
| `sn` | String(16) | Y | |
| `trade_no` | String(64) | Y | |
| `client_sn` | String(32) | Y | |
| `status` | String | N | `SUCCESS` or `FAIL` |
| `result_code`, `result_message` | String | N | Deprecated, WeChat only: `get_brand_wcpay_request:ok` / `:fail` / `:cancel` |
| `total_amount` | String(10) | Y | |
| `subject` | String(64) | Y | |
| `operator` | String(32) | Y | |
| `reflect` | String(64) | N | |
| `sign` | String(32) | Y | Same signature rules |

After return, poll query for 100 to 120 seconds: every 2s for the first 30s, then every 5s.

## 6. Refund `/upay/v2/refund` (WeChat and card)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `terminal_sn` | String(32) | Y | |
| `sn` | String(16) | N | bayarlah order no. (or put `trade_no` here) |
| `client_sn` | String(32) | N | Merchant order no. |
| `refund_request_no` | String(20) | Y | Unique per refund; keep unchanged when retrying the same refund |
| `operator` | String(64) | Y | |
| `refund_amount` | String(10) | Y | Cents |
| `extended` | JSON map | N | Up to 24 fields |
| `goods_details` | JSON | N | Items `{goods_id, goods_name, refund_amount (optional, cents), quantity, price}` |
| `reflect` | String(64) | N | |

`sn` and `client_sn` cannot both be empty; lookup is by `sn` first, then `client_sn`.

Response `biz_response.data`: `result_code` (`REFUND_SUCCESS` / `FAIL`), `error_code`, `error_message`, `terminal_sn`, `sn`, `client_sn`, `status` (refund flow status), `order_status` (`REFUNDED`, `PARTIAL_REFUNDED`), `payway`, `payway_name`, `trade_no`, `total_amount`, `net_amount` (remaining), `settlement_amount` (this refund), `finish_time`, `channel_finish_time`, `subject`, `operator`.

Rules:

- Refund within three months of payment.
- Partial refunds allowed until the remaining amount is zero; fee is refunded pro rata.
- Reusing a `refund_request_no` for a different refund fails.
- WeChat designated merchants must authorize bayarlah for API refunds (see account-setup.md).
- Time to reach the consumer depends on the channel.

## 7. Cancel `/upay/v2/cancel` and 8. Revoke `/upay/v2/revoke` (WeChat only)

Same request for both:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `terminal_sn` | String(32) | Y | |
| `sn` | String(16) | N | Lookup first |
| `client_sn` | String(64) | N | Lookup second |

Response `data`: `result_code` (`CANCEL_SUCCESS` / `CANCEL_ERROR`), `error_code`, `error_message`, `terminal_sn`, `sn`, `client_sn`, `status`, `order_status` (`CANCELED`), `payway`, `payway_name`, `trade_no`, `total_amount`, `net_amount`, `settlement_amount` (`0` if cancelled before payment, full amount if after), `finish_time`, `channel_finish_time`, `subject`, `operator`.

- `cancel` (automatic): call when a `pay` result is unknown, to reverse and avoid disputes.
- `revoke` (manual): full refund of a paid order with no transaction fee, same business logic.
- One cancellation per order per day; not allowed after the payment day; not allowed on already refunded orders.

## 9. Query `/upay/v2/query`

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `terminal_sn` | String(32) | Y | |
| `sn` | String(16) | N | Lookup first |
| `client_sn` | String(32) | N | Lookup second |

Response `biz_response.data` (`biz_response.result_code = "SUCCESS"` on a found order):

| Field | Type | Notes |
| --- | --- | --- |
| `sn`, `client_sn` | String | Example also has `client_tsn` |
| `status` | String(32) | Transaction status |
| `order_status` | String(32) | Use this for settlement |
| `payway`, `payway_name` | String | For example `1021` / `SHOPEE` |
| `sub_payway` | String | |
| `payer_uid`, `payer_login` | String | |
| `trade_no` | String(64) | PayNet Biz Message ID for DuitNow |
| `total_amount`, `net_amount` | String(10) | |
| `settlement_amount` | String(10) | |
| `finish_time`, `channel_finish_time` | String(13) | ms timestamps |
| `subject`, `operator` | String | |
| `payment_list` | array | |

Call it after `precreate` (polling) and after `pay` when no final status came back.

## 10. Payment callback to `notify_url`

bayarlah POSTs JSON (UTF-8) to your `notify_url` with header `Authorization: sign`.

Verification (Callback Signature Verification page):

1. Read `sign` from the `Authorization` header.
2. Read the request body.
3. Get the bayarlah public key from your bayarlah technical liaison.
4. Verify with `SHA256WithRSA` over the body.

Signature encoding (base64 vs hex) and key format: Not documented in official docs (checked 2026-10-21). Verify over the raw body bytes.

```js
const ok = crypto.verify("RSA-SHA256", rawBodyBuffer, BAYARLAH_PUBLIC_KEY_PEM,
                         Buffer.from(req.headers.authorization.trim(), "base64")); // confirm encoding
```

Body fields: `sn` String(16) Y, `client_sn` String(32) Y, `trade_no` String(64) N (only on success), `status` String(32) Y, `order_status` String(32) Y, `payway` Y, `sub_payway` Y, `payer_uid` N, `payer_login` N, `total_amount` String(10) Y, `net_amount` Y, `settlement_amount` Y, `subject` Y, `finish_time` String(13) Y, `channel_finish_time` N, `operator` Y, `reflect` N.

Reply with the string `success`. Retries: 1s, 5s, 30s, 600s.

## 11. Terminal Order Push `/appPush/tradePush`

Sends an order to a bayarlah payment terminal (for example T9) so the terminal collects it.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `terminal_sn` | String(32) | Y | Terminal bound to the POS that pushes |
| `out_trade_no` | String(32) | Y | Same role as `client_sn`; use later for query and refund |
| `total_amount` | String(10) | Y | Cents (`"100"` = RM1.00) |
| `trade_time` | String(13) | Y | 13-digit ms timestamp |
| `target_device_id` | String(20) | Y | Device number, for T9 the TUSN on the back |
| `notify_url` | String(128) | Y | Result callback; retries 1s, 5s, 30s, 600s until valid reply |
| `cancel_notify_url` | String(128) | N | Called when payment is cancelled |
| `pay_type` | String(32) | N | `1` bank card, `2` QR, `4` QR pre-auth, `5` card pre-auth; comma-separated or empty |
| `remark` | String(16) | N | |

Response fields: Not documented in official docs (checked 2026-10-21).

## 12. Event Order Notification (merchant POS push)

bayarlah pushes every transaction at the merchant to a URL you give bayarlah. HTTPS is compulsory.

- Credentials: bayarlah provides `app_id` and `app_secret` for this push.
- Header: `Authorization: <app_id> <sign>`.
- `sign = MD5(body_bytes + app_secret_bytes)` (official sample: lowercase hex).
- Verify: split header on the space, check `app_id` equals yours, MD5 the raw body plus secret, compare to `sign`.
- Reply immediately with the string `success`.
- Retry intervals (seconds): 1, 5, 30, 600, 60, 300, 1800, 3600, 3600, 3600, 3600, 86400; then stops.

Body fields: `sn`, `tsn`, `type` (transaction type; list not published), `client_sn`, `client_tsn`, `ctime`, `status`, `payway` (example `1024`), `sub_payway`, `order_status`, `payer_uid`, `trade_no`, `total_amount`, `net_amount`, `finish_time`, `channel_finish_time`, `subject`, `store_id`, `terminal_id`, `operator`, `reflect`, `merchant_sn`, `store_sn`, `terminal_sn`, `device_fingerprint`, `device_sn`. All strings.

Match pushed orders using order time, terminal, store and your business data.

```python
# adapted from the official Flask sample (raw bytes, constant-time compare)
received_app_id, received_sign = request.headers["Authorization"].split(" ")
body = request.get_data()                      # raw bytes
ok = received_app_id == APP_ID and hmac.compare_digest(
    hashlib.md5(body + APP_SECRET.encode()).hexdigest(), received_sign)
return ("success", 200) if ok else ("Invalid signature", 403)
```

## 13. Mini Program plugin (WeChat only)

For platform Mini Programs with several payment entities. Apply for plugin permission and read calling parameters in the WeChat plugin docs (`https://mp.weixin.qq.com/wxopen/plugindevdoc?appid=wx55540b288c5ce319&token=930559648&lang=zh_CN`). Signature: take all non-empty parameters except `sign`, sort keys ASCII ascending, join `key=value` with `&`, append `&key=<key>`, MD5, uppercase. Object values are `JSON.stringify`-ed first. Use `precreate` with `sub_payway "4"` for Mini Program orders.

## Enums

`payway` (full appendix not published; values seen in docs): `1000` DuitNow, `3` WeChat, `1021` SHOPEE (query example), `1024` (event example, unnamed), `1` (pre-order and callback examples, unnamed).

`sub_payway`: `2` dynamic QR, `3` static QR (not DuitNow), `4` Mini Program; `1` appears in CPM examples (unnamed).

`biz_response.result_code`: `PAY_SUCCESS`, `PAY_IN_PROGRESS`, `PAY_FAIL`, `PRECREATE_SUCCESS`, `SUCCESS` (query), `REFUND_SUCCESS`, `FAIL` (refund), `CANCEL_SUCCESS`, `CANCEL_ERROR`.

Transaction `status` seen: `SUCCESS`, `IN_PROG`, `FAIL_CANCELED`, `CREATED`, `FAIL` (return_url).

`order_status`: `CREATED` (pending), final `PAID`, `PAY_CANCELED`, `REFUNDED`, `PARTIAL_REFUNDED`, `CANCELED`.

Error codes (published on the redirect page appendix):

| error_code | meaning |
| --- | --- |
| `INVALID_PARAMS` | Parameter error |
| `INVALID_TERMINAL` | Terminal error |
| `ILLEGAL_SIGN` | Signature error |
| `UNKNOWN_SYSTEM_ERROR` | System error |
| `INVALID_BARCODE` | Barcode error |
| `INSUFFICIENT_FUND` | Insufficient balance |
| `EXPIRED_BARCODE` | Expired payment barcode |
| `BUYER_OVER_DAILY_LIMIT` | Payer daily limit exceeded |
| `BUYER_OVER_TRANSACTION_LIMIT` | Payer single-payment limit exceeded |
| `SELLER_OVER_DAILY_LIMIT` | Receiver daily limit exceeded |
| `TRADE_NOT_EXIST` | Transaction does not exist |
| `TRADE_HAS_SUCCESS` | Already paid |
| `SELLER_BALANCE_NOT_ENOUGH` | Seller balance insufficient |
| `REFUND_AMT_NOT_EQUAL_TOTAL` | Invalid refund amount |
| `TRADE_FAILED` | Transaction failed |
| `UNEXPECTED_PROVIDER_ERROR` | Unrecognized payment channel |
| `TRADE_TIMEOUT` | Timeout auto-cancellation |
| `ACCOUNT_BALANCE_NOT_ENOUGH` | Merchant balance insufficient |
| `CLIENT_SN_CONFLICT` | `client_sn` already exists |
| `UPAY_ORDER_NOT_EXISTS` | Order does not exist |
| `REFUNDABLE_AMOUNT_NOT_ENOUGH` | Refundable amount insufficient |
| `UPAY_TERMINAL_NOT_EXISTS` | Terminal does not exist |
| `UPAY_TERMINAL_STATUS_ABNORMAL` | Terminal not activated |
| `UPAY_CANCEL_ORDER_NOOP` | Already cancelled |
| `UPAY_CANCEL_INVALID_ORDER_STATE` | Status cannot be cancelled |
| `UPAY_REFUND_ORDER_NOOP` | Refund already completed |
| `UPAY_REFUND_INVALID_ORDER_STATE` | Status cannot be refunded |
| `UPAY_STORE_OVER_DAILY_LIMIT` | Daily receipt limit exceeded |
| `UPAY_TCP_ORDER_NOT_REFUNDABLE` | Order in a promotion, cannot be cancelled |

## Not documented (checked 2026-10-21)

Sandbox domain and test credentials, test wallets or cards, the full payway / sub_payway / transaction type appendix, API MD5 hex case, RSA signature encoding and public key format, rate limits, IP allowlists, idempotency headers, DuitNow refunds, recurring or tokenisation, payouts, settlement report API, and Terminal Order Push response fields.
