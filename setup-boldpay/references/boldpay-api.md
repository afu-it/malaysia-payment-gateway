# BOLD.Pay API Reference

Use this reference for factual BOLD.Pay (by MACROKIOSK) integration guidance.

BOLD.Pay does not publish a public developer API reference. The merchant portal (`https://www.boldpass.com/`) is login-only, and MACROKIOSK routes developer questions through an enquiry form. Every protocol fact below comes from the official MACROKIOSK WordPress.org plugin "BOLD.Pay for WooCommerce" (slug `bold-pay`, author MACROKIOSK), version 1.7.1, checked 2026-10-21. Older tags (1.0.0, 1.5.9) were read to spot changes. Anything the plugin does not show is marked "Not documented in official docs (checked 2026-10-21)". Ask MACROKIOSK for the merchant API guide before going live with a custom (non-plugin) integration.

## Source URLs

- Plugin listing: `https://wordpress.org/plugins/bold-pay/`
- Plugin source (current): `https://plugins.svn.wordpress.org/bold-pay/trunk/`
- Plugin tags: `https://plugins.svn.wordpress.org/bold-pay/tags/` (1.0.0 to 1.7.1)
- Plugin zip: `https://downloads.wordpress.org/plugin/bold-pay.zip`
- Files used: `boldpay.php`, `includes/boldpaycore.php`, `includes/boldpay.php`, `includes/boldpayblocks.php`, `readme.txt`
- Product page: `https://macrokiosk.com/boldpay-payment-gateway`
- Merchant portal: `https://www.boldpass.com/login`, `https://www.boldpass.com/register`

## Integration Model

BOLD.Pay is a hosted payment page reached by a browser redirect (HTTP GET with a query string). There is no JSON create-payment API in the plugin. The flow is:

1. Merchant server builds the payment request fields and an `AccessToken` (MD5 hash, see below).
2. Browser is redirected to `PaymentRequest?<fields>`.
3. Customer picks a channel (card, online banking, e-wallet, BNPL, OTC) on the BOLD.Pay page and pays.
4. BOLD.Pay redirects the browser back to `CallbackURL` (GET) with the result.
5. BOLD.Pay sends a server-to-server POST to `NotificationURL` with the same result fields. The merchant replies with body `OK`.
6. The merchant server calls `QueryRequest` to confirm the status before settling.

## Environments

| Environment | Payment page (redirect target) | Status query |
|---|---|---|
| Live | `https://pay.etracker.cc/BoldPayApi/PaymentRequest` | `https://pay.etracker.cc/BoldPayApi/QueryRequest` |
| Sandbox (UAT) | `https://uat.pay.etracker.cc/BoldPayApi/PaymentRequest` | `http://uat.pay.etracker.cc/BoldPayApi/QueryRequest` (plugin constant) |

Notes:

- The plugin's UAT query constant uses plain `http://`. The UAT host also answers over HTTPS (both hosts return HTTP 400 to a bare request, checked 2026-10-21). Use `https://` for both environments.
- Plugin default environment is `live`. Unknown values fall back to live. Keep the environment explicit in config.
- Whether sandbox needs a separate username and API key: Not documented in official docs (checked 2026-10-21).

## Endpoint Catalogue

Everything the official plugin uses. No other operation is published.

| # | Method | Path | Purpose |
|---|---|---|---|
| 1 | GET (browser redirect) | `/BoldPayApi/PaymentRequest` | Open the hosted payment page for one order |
| 2 | GET (server-to-server) | `/BoldPayApi/QueryRequest` | Requery payment status; result comes back as a redirect `Location` query string |
| 3 | GET to merchant (browser) | `CallbackURL` (merchant) | Return of the customer with the result fields |
| 4 | POST to merchant (server) | `NotificationURL` (merchant) | Asynchronous result notification; reply body `OK` |

Not documented in official docs (checked 2026-10-21): refund API, cancel/void, capture, recurring or auto-debit token API, instalment parameters, payout, settlement report, payment link API, carrier billing API, channel list API, rate limits, IP ranges of BOLD.Pay servers.

## 1. PaymentRequest (create payment)

Build a query string and redirect the browser to `<base>/BoldPayApi/PaymentRequest?<query>`. The plugin joins `key=value` pairs with `&` in this order (it encodes only some values itself, shown in Notes):

| Field | Required | Type / format | Notes |
|---|---|---|---|
| `AccessToken` | Yes | 32 hex chars, uppercase | MD5 token, see "AccessToken algorithm" |
| `Username` | Yes | string | BOLD.Pass sign-up username |
| `CustIP` | Yes | IPv4 string | Plugin sends the merchant server's own IP (`gethostbyname(gethostname())`), not the shopper IP. Must be the same value used in the token. Error 402 if invalid |
| `PymtMethod` | Yes | string | Plugin always sends `4`, and `4` is hard-coded in the token. Other values: Not documented |
| `Amount` | Yes | decimal, 2 places, ringgit | e.g. `10.50`. Not sen. Error 404 invalid, 417 over limit, 418 under minimum |
| `Description` | Yes | URL-encoded string | Plugin sends `order <id>`. Error 405 |
| `CustName` | Yes | URL-encoded string | First + last name. Error 421 |
| `CustEmail` | Yes | email | Plugin sends raw (not encoded); encode `+` yourself. Error 406 |
| `CustPhone` | Yes | digits only | Plugin strips `+` and validates 6 to 20 digits at checkout; the plugin forces billing phone to be required. Error 407 |
| `OrderNumber` | Yes | string | Merchant order ID, echoed back. Error 408 |
| `PayID` | Yes | string, unique per attempt | Plugin format: first 4 chars of username (upper) + first 4 chars of API key + `ymdHis`. Error 416 invalid or duplicate |
| `CallbackURL` | Yes | URL-encoded URL | Browser return URL. Error 409 |
| `NotificationURL` | Yes | URL-encoded URL | Server notification URL. Plugin falls back to `CallbackURL` if unset. Error 410 |
| `ApiKey` | Yes | string | API key from BOLD.Pass Business Setting. Error 413 |
| `Origin` | Plugin only | string | Plugin sends `PLUGIN_WC`. Value for custom integrations: Not documented |
| `CurrencyCode` | Yes | numeric code | `1` = MYR, `3` = THB (plugin options). Other codes: Not documented. Error 412 |

Gotchas:

- The API key travels in the browser URL. The plugin has no server-side create call, so the protection is the Domain/IP Restrictions list in BOLD.Pass (error 415 when the domain or IP is not listed). Add only your production domain and server IP.
- `PayID` from `ymdHis` collides if two orders start in the same second. Add a random or sequence suffix if BOLD.Pay accepts it; maximum length and allowed characters: Not documented.
- Plugin 1.6.0 added the separate `NotificationURL`. Versions up to 1.5.9 sent `CallbackURL` in both fields and in the token.

## AccessToken algorithm

From `BoldPayCore::prepareAccessToken()` (plugin 1.7.1):

```text
raw   = UPPER( Username
             + "4"                         # PymtMethod constant
             + format(Amount, 2 decimals, "." separator, no thousands separator)
             + CustIP
             + PayID
             + CallbackURL                 # raw URL, NOT url-encoded
             + NotificationURL             # raw URL, NOT url-encoded
             + ApiKey )
token = UPPER( hex( MD5(raw) ) )
```

- Concatenation has no separators.
- The whole string, including URLs and the API key, is uppercased before hashing.
- URLs in the hash are the plain URLs (plugin uses `esc_url_raw`), while the query string carries them URL-encoded.

Pseudo-code (Node):

```js
import crypto from "node:crypto";
function boldpayToken({ username, amount, custIp, payId, callbackUrl, notificationUrl, apiKey }) {
  const raw = (username + "4" + Number(amount).toFixed(2) + custIp + payId +
               callbackUrl + notificationUrl + apiKey).toUpperCase();
  return crypto.createHash("md5").update(raw, "utf8").digest("hex").toUpperCase();
}
```

Note: `toFixed(2)` matches PHP `number_format(x, 2, '.', '')` for normal money values; compute from integer sen to avoid float drift.

## 3 and 4. Callback (browser return) and Notification (server POST)

Both arrive at the merchant handler with these fields (read from `$_REQUEST`; the notification is a form POST, the return is a GET):

| Field | Notes |
|---|---|
| `AccessToken` | Same token as the request (recompute and compare) |
| `TransactionID` | BOLD.Pay transaction ID |
| `OrderNumber` | Your order ID |
| `PayID` | Your PayID |
| `Status` | `SUCCESS`, `PENDING`, `FAIL`, `REFUNDED`, `REVERSED`; or a numeric error code (see below) |
| `Reason` | Failure reason text (used on `FAIL`) |
| `PymtMethod` | Channel used: `CC` credit card, `DD` online banking, `WA` e-wallet, `BNPL` buy now pay later, `OTC` over the counter |
| `IssuingBank` | Bank or wallet sub-type (values not listed) |

How the plugin tells them apart: if `OrderNumber` is in the POST body it is the notification, otherwise the browser return.

Error variant: when the request is rejected, BOLD.Pay returns `Status` (numeric code), `PayID` and `OrderNumber` without `AccessToken`/`TransactionID`. Treat as not paid and log the code.

Notification reply: the plugin always prints `OK` and exits for notifications, including on validation failure. Retry policy if `OK` is missing: Not documented.

Verification (what the token proves): the response `AccessToken` is the same deterministic value as the request token. It does not cover `Status`, `TransactionID` or `Reason`, and it was visible in the payment URL. So a matching token only proves the response belongs to that `PayID` and amount. It is not a signature over the result. Always confirm with `QueryRequest` before changing order state. The plugin queries for `SUCCESS`, `PENDING`, `REFUNDED`, `REVERSED` and trusts `FAIL` directly; querying on `FAIL` too is safer.

Recompute with your stored values: stored username, the order's expected amount, the same `CustIP` you sent, the received `PayID`, and the same callback and notification URLs. Use a constant-time compare.

## 2. QueryRequest (status requery)

`GET <base>/BoldPayApi/QueryRequest?<query>` from the server. Field names and casing differ from PaymentRequest:

| Field | Value |
|---|---|
| `AccessToken` | The token (from the callback, or your recomputed one) |
| `APIKey` | API key (note casing) |
| `Username` | Username |
| `PayID` | PayID |
| `CallBackUrl` | URL-encoded CallbackURL (note casing) |

Response: the plugin does not follow redirects. It reads the HTTP redirect target (`Location`) and parses its query string. Fields used: `AccessToken`, `TransactionID`, `OrderNumber`, `PayID`, `Status`.

- If the parsed result has no `AccessToken`, it is an error; `Status` holds the error code.
- The plugin checks `TransactionID`, `OrderNumber` and `PayID` equal the callback values, and treats a mismatch as tampering.
- Settle from the query `Status`, not the callback `Status`.
- On notifications the plugin waits 2 seconds before querying.
- The plugin disables TLS peer verification on this call. Do not copy that; keep TLS verification on.
- Whether the query result includes the amount: Not documented. Amount is bound only through the token, so recompute the token from your stored expected amount.

Pseudo-code:

```js
const qs = new URLSearchParams({ AccessToken: token, APIKey: apiKey, Username: username,
  PayID: payId, CallBackUrl: callbackUrl });
const res = await fetch(`${base}/BoldPayApi/QueryRequest?${qs}`, { redirect: "manual" });
const loc = res.headers.get("location");
const q = loc ? Object.fromEntries(new URL(loc, base).searchParams) : {};
if (!q.AccessToken) throw new Error(`BOLD.Pay query error ${q.Status ?? res.status}`);
if (q.PayID !== payId || q.OrderNumber !== orderNumber || q.TransactionID !== txId) throw new Error("mismatch");
```

## Status Values And Plugin Order Mapping

| Query `Status` | Meaning | Plugin action (WooCommerce) |
|---|---|---|
| `SUCCESS` | Paid | `payment_complete()` if order is pending, failed or on-hold; skip if already processing |
| `PENDING` | Awaiting confirmation | Set on-hold if pending or failed |
| `FAIL` | Declined (callback value, not queried) | Set failed if pending, failed or on-hold; note `Reason` |
| `REFUNDED` | Refunded | Set refunded if processing or on-hold |
| `REVERSED` | Reversed | Same as `REFUNDED` |
| other | Error | Order note, show connection error |

## Error Codes

From `getStatusCodeDescription()`:

| Code | Meaning |
|---|---|
| 400 | Missing parameter(s) |
| 401 | Invalid Access Token |
| 402 | Invalid Customer IP |
| 403 | Invalid Payment Method |
| 404 | Invalid Amount |
| 405 | Invalid Description |
| 406 | Invalid Customer Email |
| 407 | Invalid Customer Phone |
| 408 | Invalid Order Number |
| 409 | Invalid Call Back URL |
| 410 | Invalid Notification URL |
| 411 | Invalid Username |
| 412 | Invalid Currency Code |
| 413 | Invalid API Key |
| 414 | Invalid Transaction ID |
| 415 | Invalid Domain or IP Address |
| 416 | Invalid Pay ID / Duplicate Pay ID |
| 417 | Exceeded Transaction Limit (plugin keeps the order pending) |
| 418 | Does Not Meet Minimum Payment Amount |
| 421 | Invalid Customer Name |
| 422 | Invalid Instalment Parameter(001) |
| 423 | Invalid Instalment Parameter(002) |
| 500 | Server Error |

Codes 419 and 420 are not listed. 422 and 423 show instalment parameters exist, but their names are Not documented.

## Amounts And Currency

- Amount is a decimal in major units with exactly 2 decimals (`wc_format_decimal(total, 2)`, `number_format(..., 2, '.', '')`).
- Currency codes: `1` MYR, `3` THB. The plugin's unused `is_valid_for_use()` checks only MYR.
- Minimum and maximum amounts are account limits (errors 417, 418); values Not documented.

## Refunds, Recurring, Instalments

- Refund API: Not documented. The plugin declares `supports = ['products']` only (no WooCommerce refunds). Refunds show up as `REFUNDED`/`REVERSED` notifications, so they are done outside the plugin (portal or MACROKIOSK).
- Auto-debit recurring and card instalments are listed as features in `readme.txt`; API fields: Not documented.

## Sandbox Test Data

Not documented in official docs (checked 2026-10-21): test cards, test banks, sandbox credentials. Ask MACROKIOSK for UAT access.

## Common Errors

- 401 Invalid Access Token: wrong field order, missing uppercase step, URL-encoded URLs inside the hash, amount not 2 decimals, `NotificationURL` left out of the hash, or a different `CustIP` than the one sent.
- 415 Invalid Domain or IP: add the shop domain and server IP in BOLD.Pass Admin Management > Business Setting > API Key > Domain/IP Restrictions.
- 416 Duplicate Pay ID: generate a new `PayID` for every attempt, including retries of the same order.
- 407 Invalid Customer Phone: send digits only, no `+`, spaces or dashes.
- Token mismatch on callback after config change: the callback is checked against the current settings, so changing callback/notification URLs breaks verification for in-flight payments. Store the URLs and IP used per payment.
