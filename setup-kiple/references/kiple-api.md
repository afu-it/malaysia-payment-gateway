# Kiple (kiplePay) API Reference

Use this reference for factual Kiple payment gateway guidance. Checked 2026-10-21.

## Where This Comes From

Kiple does not publish a public developer portal or API specification (no developer subdomain, sitemap entry, OpenAPI file or PDF found; checked 2026-10-21). Merchants receive their Merchant ID and Secret Key from Kiple after approval. Everything below is distilled from the **official Kiple plugins and SDKs** published by the GitHub organisation `m2c` (display name "Kiplepay Sdn Bhd"), plus the live behaviour of Kiple's own gateway URLs. Where plugins disagree, both forms are shown. Anything not present in those sources is marked "Not documented in official sources (checked 2026-10-21)".

Kiple was formerly called Webcash (operated by Webonline Dot Com Sdn Bhd, now Kiplepay Sdn Bhd). Older plugins and field names still say `webcash` / `wc`.

Primary sources (full list in `source-map.md`):

- WooCommerce plugin v4.5.6: `https://github.com/m2c/woocommerce/blob/woocommerce_plugin_v4.5.6/wc-kiple.php`
- OpenCart 3 plugin v3.1.5 (includes recurring): `https://github.com/m2c/opencart/tree/kiplepay_opencart_plugin_version_v3.1.5`
- Kiple PHP SDK v1.0: `https://github.com/m2c/php-sdk/tree/Kiple_PHP_SDK_v1.0`
- Magento 2.3.5-p1 plugin (split orders): `https://github.com/m2c/magento/tree/Magento_v2.3.5-p1`
- PrestaShop 1.7.1 plugin (requery after return): `https://github.com/m2c/prestashop/tree/kiplepay_Prestashop_v1.7.1`

## Integration Surfaces

- **Hosted payment page (the only online gateway surface in official sources).** Your server builds an HTML form; the browser auto-submits it by POST to the Kiple gateway URL; the buyer picks a method (cards, FPX, e-wallets, DuitNow QR, depending on the merchant's activation) on Kiple's page; Kiple POSTs the result back to your `ord_returnURL` (browser) and, if you pass one, to your `dynamic_callback_url` (server-to-server).
- **Enquiry (requery).** Plain HTTP GET that returns a one-word status.
- **Recurring (card subscriptions).** The same hosted form with extra `ord_recurring_*` fields. Requires a recurring subscription from Kiple.
- **Split orders (marketplace).** The same form with pipe-separated merchant IDs and amounts. Shown in the Magento 2 plugin as sample code.
- **Legacy iOS app-to-app deeplink SDK (2018).** See "Legacy iOS Deeplink SDK" below; its hosts did not respond on 2026-10-21.
- **No-code:** plugins for WooCommerce, OpenCart, Magento 1.9 and 2.3, PrestaShop 1.4 to 1.7, WHMCS 7, CubeCart 6, osCommerce 2.3. Kiple's marketing pages also list payment links; no API for them is published.

There is no JSON REST API, no API key header, no refund API, and no published webhook signature beyond the `ord_key` hash in official sources.

## Environments

| Purpose | UAT / sandbox | Production |
|---|---|---|
| Payment entry (plugins, current) | `https://uat.kiplepay.com/wcgatewayinit.php` | `https://kiplepay.com/wcgatewayinit.php` |
| Payment entry (PHP SDK) | `https://uat.kiplepay.com/gateway` | `https://kiplepay.com/gateway` |
| Payment entry (legacy Webcash host) | `https://staging.webcash.com.my/wcgatewayinit.php` | `https://webcash.com.my/wcgatewayinit.php` |
| Enquiry (PrestaShop) | `https://uat.kiplepay.com/enquiry.php` | `https://kiplepay.com/enquiry.php` |
| Enquiry (PHP SDK) | `https://uat.kiplepay.com/enquiries` | `https://kiplepay.com/enquiries` |

Notes:

- The WooCommerce plugin v4.5.6 uses `uat.kiplepay.com` for test and `webcash.com.my` for live. OpenCart 3.1.5, Magento 2, WHMCS 7.2 and PrestaShop 1.7 use `kiplepay.com` for live. Prefer `kiplepay.com`; confirm with Kiple which host your merchant account is provisioned on.
- On 2026-10-21 all entry and enquiry URLs above answered HTTP 200. A GET to the UAT entry URL with no fields shows Kiple's validation page: "Invalid Order Date", "Invalid Amount or Amount cannot be less than RM1.0", "Invalid Merchant Reference", "Invalid Merchant ID", "Invalid Return URL", "Return URL must be fully qualified link". A GET to either enquiry URL with no fields returns `Invalid Parameters`.
- UAT and production use different Merchant IDs and secrets.
- Currency: every plugin restricts to MYR ("Kiplepay supports only Malaysian currency (MYR)", OpenCart and CubeCart readmes).

## Endpoint Catalogue

| # | Method | URL | Auth | Purpose |
|---|---|---|---|---|
| 1 | POST (browser form, `application/x-www-form-urlencoded`) | `/wcgatewayinit.php` (or `/gateway`) on `uat.kiplepay.com` / `kiplepay.com` | `merchant_hashvalue` field | Start a hosted payment (one-off, recurring, or split) |
| 2 | POST (Kiple to your site, via browser) | your `ord_returnURL` | `ord_key` field | Return the buyer with the result |
| 3 | POST (Kiple to your server) | your `dynamic_callback_url` | `ord_key` field; request carries `nbcb` | Server-to-server result notification |
| 4 | GET | `/enquiry.php` or `/enquiries` on `uat.kiplepay.com` / `kiplepay.com` | none (merchant ID + reference + amount in query) | Requery a transaction status |
| 5 | POST (Kiple to your server) | your recurring callback URL (OpenCart route `extension/payment/kiplepay/recurringCallback`) | Not documented in official sources | Result of each later recurring charge |
| 6 | (legacy) app URL open | `https://deeplink.kiplepay.com/ios?...` | `CheckSum` (SHA-1) | iOS app-to-app wallet payment (2018 SDK) |
| 7 | (legacy) GET | `https://portal.kiplepay.com:94/api/wallets/me/deeplink-payment/{merchantId}/{referenceId}` | none shown | iOS deeplink payment status |
| 8 | (legacy, sample app) POST JSON | `https://staging.kiplepay.com:94/api/deeplinks/generate` | none shown | Generate a deeplink (staging sample only) |

## 1. Create Payment (hosted form POST)

Build the form on your server, then render it with an auto-submit script. All values are form fields. Official plugins send the following.

### Request fields

| Field | Required | Notes |
|---|---|---|
| `ord_mercID` | Yes | Kiple Merchant ID (numeric, for example `8000xxxx`). For split orders, pipe-separated list of sub-merchant IDs. |
| `ord_mercref` | Yes | Your merchant reference. Must be unique per attempt. Plugins use `<orderId>-<uniqid>` (WooCommerce), `kp-<uniqid>-<orderId>` (OpenCart), `<merchantId>-<uniqid>-<orderId>` (Magento 2). Gateway rejects empty values ("Invalid Merchant Reference"). Maximum length: Not documented. |
| `ord_totalamt` | Yes | Amount in ringgit with decimals, for example `50.00`. Minimum RM1.00 (gateway validation page). Split orders: pipe-separated amounts, for example `1.77|0.22|0.33`. See "Amount rules". |
| `ord_date` | Yes | Order date-time `Y-m-d H:i:s`, for example `2026-10-21 14:05:00` (PHP SDK validates this format; gateway shows "Invalid Order Date" if missing). |
| `ord_returnURL` | Yes | Fully qualified HTTPS URL that receives the browser POST back. Gateway rejects relative URLs. |
| `merchant_hashvalue` | Yes | Request hash, see "Signatures". |
| `ord_shipname` | Yes (PHP SDK marks it mandatory) | Buyer full name. |
| `ord_email` | Recommended | Buyer email. |
| `ord_telephone` | Recommended | Buyer phone. |
| `version` | Yes for current plugins | `2.0`. WooCommerce changelog 14/03/2022: "Updated to kiple payment version 2.0". Older plugins omit it. |
| `currency` | Recommended | `MYR`. WHMCS and PrestaShop send `Currency`; CubeCart sends `RM`. |
| `dynamic_callback_url` | Recommended | Server-to-server callback URL. Kiple POSTs the result here with `nbcb` set. If omitted, only the browser return is guaranteed by the plugin sources. |
| `ord_gstamt` | Optional | GST amount, plugins send `0` or `0.00`. PHP SDK: "only works if you registered as GST merchant". |
| `ord_delcharges` | Optional | Delivery charges (PHP SDK, "reference" only). |
| `ord_svccharges` | Optional | Service charges (PHP SDK, "reference" only). |
| `ord_shipcountry` | Optional | Country code. Echoed back on return (osCommerce relies on this to carry its session ID). |
| `desc` | Optional | Item description (OpenCart, CubeCart). |
| `address`, `postcode` | Optional | Billing address (OpenCart, CubeCart). |
| `ord_customfield4` | Optional | Plugins tag their platform, for example `plg_woocommerce`, `plg_opencart_v3.1.5`. |
| `ord_customfield3` | Optional | OpenCart sends `opencart_recurring` on recurring orders. Other `ord_customfieldN` slots: Not documented. |
| `payment_code` | Optional | OpenCart sends `CC` for recurring (card). Other values (FPX, wallets) and preselecting a method: Not documented. |
| `promo_bin` | Optional | Sent empty by Magento 2. Purpose: Not documented. |
| `is_split_orders`, `all_split_totalamt`, `ord_mmercID` | Split orders only | See "Split orders". |
| `ord_recurring_*` | Recurring only | See "Recurring". |

Legacy fields seen only in older plugins (WHMCS, PrestaShop): `ProdDesc`, `Remark`, `Lang=UTF-8`, `Signature`, and PayPal-style cart fields. They are not needed for v2.0.

### Example form (sandbox)

```html
<form id="kiple" method="post" action="https://uat.kiplepay.com/wcgatewayinit.php">
  <input type="hidden" name="ord_mercID" value="<MERCHANT_ID>">
  <input type="hidden" name="ord_mercref" value="1001-6534a1b2c3d4e">
  <input type="hidden" name="ord_totalamt" value="50.00">
  <input type="hidden" name="ord_gstamt" value="0.00">
  <input type="hidden" name="currency" value="MYR">
  <input type="hidden" name="ord_date" value="2026-10-21 14:05:00">
  <input type="hidden" name="ord_shipname" value="Ali Abu">
  <input type="hidden" name="ord_email" value="buyer@example.com">
  <input type="hidden" name="ord_telephone" value="0123456789">
  <input type="hidden" name="ord_returnURL" value="https://shop.example.com/kiple/return">
  <input type="hidden" name="dynamic_callback_url" value="https://shop.example.com/kiple/callback">
  <input type="hidden" name="version" value="2.0">
  <input type="hidden" name="merchant_hashvalue" value="<SHA1_HEX>">
</form>
<script>document.getElementById('kiple').submit();</script>
```

HTML-escape every value. Official plugins do not, which breaks on names with quotes.

## Amount Rules

- Unit: ringgit with a decimal point (not sen). Minimum RM1.00.
- The hash uses the amount **as posted** with `.` and `,` removed (`|` also removed for split orders). `50.00` hashes as `5000`; `1,250.50` hashes as `125050`.
- Always format to exactly two decimals and post the same string you hashed. `50` and `50.00` produce different hash input (`50` vs `5000`); the PHP SDK sample posts `50`, plugins post two decimals. Use two decimals.
- Do not send thousands separators. Plugins strip `,` from the hash input only, so a posted `1,250.50` is risky.

## 2 and 3. Return and Callback

Kiple POSTs form fields to `ord_returnURL` (browser redirect) and to `dynamic_callback_url` (server). The WooCommerce plugin points both to the same URL and tells them apart by the `nbcb` field: present and truthy means server callback, absent means browser return. The value of `nbcb` is Not documented.

### Response fields

| Field | Meaning |
|---|---|
| `returncode` | Result code, see "Status codes". |
| `ord_key` | Response hash, see "Signatures". |
| `ord_mercID` | Merchant ID. |
| `ord_mmercID` | Master merchant ID (split orders only). |
| `ord_mercref` | Your reference, as sent. |
| `ord_totalamt` | Amount. |
| `wcID` | Kiple transaction ID. Store it; use it as the provider reference and for duplicate detection (WHMCS `checkCbTransID`, OpenCart recurring). |
| `entry_id` | Kiple entry ID (OpenCart logs it). |
| `payment_type` | Payment method used (OpenCart logs it). Value list: Not documented. |
| `channel` | Channel (WooCommerce reads it). Value list: Not documented. |
| `appcode` | Approval code (WooCommerce reads it). |
| `domain` | Read by WooCommerce, meaning Not documented. |
| `currency` | Currency. |
| `ord_date` | Date-time. |
| `error_description` | Failure text (OpenCart shows it for `E1` / `E2`). |
| `nbcb` | Present on the server-to-server callback. |
| `ord_shipname`, `ord_telephone`, `ord_email`, `ord_shipcountry`, `ord_gstamt`, `ord_delcharges`, `ord_svccharges` | Echoed order fields (PHP SDK maps them on response; osCommerce reads `ord_shipcountry`). |
| `ord_famt`, `ord_fcur` | Foreign amount and currency, read only by the legacy Magento 1.9 plugin. |

Response format expected from your callback handler (for example `OK` text or HTTP 200 only): Not documented. Retry policy for failed callbacks: Not documented. Return HTTP 200 quickly after processing; plugins simply render a page or redirect.

### Status codes

| `returncode` | Meaning (plugins and PHP SDK) |
|---|---|
| `100` | Successful transaction |
| `E1` | Failed transaction |
| `E2` | Aborted by buyer or timed out |
| `-1` | Invalid request from merchant (validation failed at the gateway) |
| anything else | Treat as error / not paid |

WooCommerce sets the order to `on-hold` (not failed) when the server callback reports `E1` or `E2`, and to `failed` on the browser return.

## Signatures

Algorithm: **SHA-1**, lowercase hex (PHP `sha1()`), over a plain string concatenation with no separators. The key is the merchant **Secret Key** (called "Verify Key" in WooCommerce, "Hash Key" in OpenCart, "Merchant Key" in WHMCS/PrestaShop, "merchantSecret" in the PHP SDK).

Request (`merchant_hashvalue`):

```
sha1( secret + ord_mercID + ord_mercref + amountDigits )
```

Response (`ord_key`), on both return and callback:

```
sha1( secret + merchantId + ord_mercref + amountDigits + returncode )
```

- `amountDigits` = `ord_totalamt` with `.` and `,` removed (and `|` for split orders).
- `merchantId` = `ord_mmercID` if present (split orders), otherwise `ord_mercID`.
- Split orders: request hash uses the master merchant ID and its secret, and the pipe-joined amount string with `.`, `,` and `|` removed (Magento 2).

Pseudo-code:

```python
import hashlib, hmac

def amount_digits(a: str) -> str:
    return a.replace(".", "").replace(",", "").replace("|", "")

def request_hash(secret, merchant_id, ref, amount):
    return hashlib.sha1((secret + merchant_id + ref + amount_digits(amount)).encode()).hexdigest()

def verify_response(secret, expected_merchant_id, expected_ref, expected_amount, post):
    code = post.get("returncode", "")
    calc = hashlib.sha1((secret + expected_merchant_id + expected_ref
                         + amount_digits(expected_amount) + code).encode()).hexdigest()
    return hmac.compare_digest(calc, post.get("ord_key", "").lower())
```

Worked example with fake values: secret `MySecret`, merchant `80001234`, reference `1001-6534a1b2c3d4e`, amount `50.00`.
- Request string `MySecret800012341001-6534a1b2c3d4e5000` gives `f6e2d596bc1c4435cd3bfb38237ab5a1bedee05c`.
- Response string with `returncode=100` gives `c0a289149f3435649ded152c6a4887b2a9ce85f8`.

Verification cautions:

- Recompute with your **stored** merchant ID, reference and expected amount, not with posted values (the WHMCS plugin hashes the posted `ord_mercID`; do not copy that). A valid hash over a different amount or reference must still be rejected.
- The `ord_key` hash does not cover `wcID`, `payment_type` or other fields. Do not trust them for anything security-relevant.
- SHA-1 with a secret prefix is weak by modern standards. Combine it with an enquiry call before fulfilling (PrestaShop does this).
- Use constant-time comparison.

## 4. Enquiry (Requery)

```
GET https://kiplepay.com/enquiry.php?ord_mercID=<MERCHANT_ID>&ord_mercref=<REF>&ord_totalamt=<AMOUNT>
GET https://kiplepay.com/enquiries?ord_mercID=<MERCHANT_ID>&ord_mercref=<REF>&ord_totalamt=<AMOUNT>
```

UAT: same paths on `uat.kiplepay.com`. Query parameters are URL-encoded; amount in the same format you posted (PHP SDK sample sends `50`; PrestaShop sends the posted `ord_totalamt`).

Response is a plain-text body:

| Body | Meaning |
|---|---|
| `S` | Successful |
| `F` | Failed |
| `A` | Aborted |
| `Invalid Parameters` | Missing or bad parameters |
| `Records not Found` | No transaction for that merchant, reference and amount |

The enquiry is unsigned and returns no transaction ID, timestamp or method. Use it as a second factor before settlement and as a fallback when the callback is missing. Enquiry rate limits: Not documented.

## Recurring (OpenCart 3.1.5)

Requires a recurring subscription from Kiple (`support@kiplepay.com`). Card only (`payment_code=CC`). Extra form fields on the first payment:

| Field | Value |
|---|---|
| `ord_recurring_enable` | `1` |
| `ord_recurring_frequency` | `monthly`, `quarterly` or `yearly` (readme says only monthly or quarterly; code also allows yearly) |
| `ord_recurring_start_date` | `d/m/Y`, for example `21/10/2026` |
| `ord_recurring_recurrence` | Number of charges, or `perpetually` |
| `ord_recurring_charging_date` | Day of month `01` to `28`, or `end of month` |
| `payment_code` | `CC` |
| `ord_customfield3` | `opencart_recurring` (plugin tag) |

Rules from the OpenCart readme: cycle must be 1; no trial periods; one recurring product per cart and no mixing with one-off items; Kiple charges only the recurring price (not product or shipping price); cancellation is done in Kiple's recurring portal (no API).

Later charges: Kiple POSTs `ord_mercref`, `returncode`, `ord_totalamt`, `wcID` to the plugin's `recurringCallback` route. The plugin does not check `ord_key` there and dedupes by `wcID`. How Kiple learns that URL, and whether `ord_key` is sent: Not documented. Verify `ord_key` if present and requery before crediting.

## Split Orders (Magento 2.3.5-p1 sample)

| Field | Value |
|---|---|
| `is_split_orders` | true |
| `ord_mercID` | `idA|idB|idC` |
| `ord_totalamt` | `1.77|0.22|0.33` |
| `all_split_totalamt` | Sum, two decimals (`2.32`) |
| `ord_mmercID` | Master merchant ID |
| `merchant_hashvalue` | `sha1(masterSecret + masterId + ref + "17722033")` |

The plugin ships this as commented sample code (split disabled by default). Enablement requires Kiple: Not documented publicly.

## Refunds, Voids, Captures, Payouts

No refund, void, capture, payout or settlement-report API exists in any official Kiple source. Kiple's Merchant Terms (clause 7.6) make the merchant liable for refunds and chargebacks. Settlement reports are downloaded from the kiplePay Portal (Merchant Settlement Process Schedule v2.0). Process refunds through Kiple support or the portal.

## Legacy iOS Deeplink SDK (KPPaymentSDK, 2018)

For app-to-app payment through the Kiple wallet app. Hosts did not resolve or redirect to kiple.com on 2026-10-21; treat as historical.

- Open `https://deeplink.kiplepay.com/ios?MerchantId=&StoreId=&Amount=&ReferenceId=&CheckSum=&Type=` (staging base `https://staging.webcash.com.my`). `Amount` two decimals; `Type` is the integer `0` Payment, `1` MobileReload, `2` PayBill.
- Outgoing `CheckSum = sha1(MerchantId + TypeName + StoreId + Amount + ReferenceId + secret)` where `TypeName` is `Payment`, `MobileReload` or `PayBill`.
- Return URL to the app carries `StoreId`, `Amount`, `ReferenceId`, `TransactionId`, `Status`, `CheckSum`; verify `sha1(secret + MerchantId + StoreId + Amount + ReferenceId + TransactionId + Status)`.
- `Status`: `Successful`, `Pending`, `Failed`, `Cancelled`.
- Status API: `GET https://portal.kiplepay.com:94/api/wallets/me/deeplink-payment/{merchantId}/{referenceId}` (staging `https://staging.kiplepay.com:94`), JSON with `Status`, `Amount`, `TransactionId`, `TradeDate`, or `Code: TRANSACTION_NOT_FOUND`.

## Sandbox Test Data

- Official WHMCS and Magento 1.9 readmes (Webcash era) publish a shared test Merchant ID `80000155` and a dummy Webcash wallet account (`20000030` / `tmp1234`), "keep value low". These pre-date the `version=2.0` hash flow and may no longer work; request your own UAT Merchant ID and secret from Kiple.
- Test cards and FPX test banks: Not documented in official sources (checked 2026-10-21).

## Common Errors

| Symptom | Cause |
|---|---|
| Kiple page "Invalid Merchant ID" / "Invalid Amount or Amount cannot be less than RM1.0" / "Invalid Return URL" | Missing or bad field; amount below RM1.00; relative return URL |
| `returncode=-1` | Gateway validation failed (bad hash, bad field) |
| Return says invalid hash | Wrong secret for environment, amount format mismatch (`50` vs `50.00`), reference changed, or `ord_mmercID` ignored on split orders |
| WooCommerce comment "Invalid Transaction maybe is because vkey not found / skey wrong generated" | Verify Key (secret) empty or wrong. The WooCommerce field help says "Just enter any value"; that is wrong for v2.0, the value is the hash key |
| Session lost on return (cookies) | Cross-site POST back; OpenCart and Magento set the session cookie `SameSite=None; Secure` |
| Enquiry `Records not Found` | Amount format or reference differs from the original request |

Rate limits, IP allowlists for callbacks, and error code lists beyond the above: Not documented in official sources (checked 2026-10-21).
