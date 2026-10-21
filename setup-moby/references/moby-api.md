# MOBY (MobyPay) API Reference

Entry point for factual MOBY / MobyPay online payment guidance. Recurring tokens and subscriptions are in `moby-recurring.md`. POS, kiosk TCP and installment plans are in `moby-pos.md`. All facts below were checked on 2026-10-21 against the sources in `source-map.md`.

MOBY publishes three overlapping official sources that do not fully agree. Order of trust used in this skill:

1. Current developer docs at `https://docs.mobypay.my/` (pages and its API Reference explorer).
2. The OpenAPI 3.0 spec served by MOBY itself at `https://dev.pay.mobycheckout.com/docs?api-docs.json` (Swagger UI: `https://dev.pay.mobycheckout.com/api/documentation`; the production host serves an identical file).
3. MOBY's own plugin source code (WooCommerce v2.0, Magento 2, OpenCart 4) downloadable from the docs site, used for signature and callback behaviour.
4. GitHub `MobyPayTech/moby-plugins` READMEs. The repo marks itself "[Deprecated, use the below url for active doc]", so use it only where it adds detail and the current docs are silent.

Where they conflict, this file says so. Do not silently pick one.

## Source URLs

- `https://docs.mobypay.my/docs/introduction`
- `https://docs.mobypay.my/docs/sandbox-testing`
- `https://docs.mobypay.my/docs/authentication`
- `https://docs.mobypay.my/docs/hosted-checkout`
- `https://docs.mobypay.my/docs/direct-cards`, `/docs/direct-fpx`, `/docs/direct-ewallets`, `/docs/direct-duitnow`, `/docs/direct-bnpl`
- `https://docs.mobypay.my/docs/notifications`
- `https://docs.mobypay.my/docs/signature-verification`
- `https://docs.mobypay.my/docs/api-reference`
- `https://dev.pay.mobycheckout.com/docs?api-docs.json`
- `https://docs.mobypay.my/plugins/downloads/woocommerce/moby-checkout/v2.0/moby-checkout.zip`
- `https://docs.mobypay.my/plugins/downloads/magento/moby-checkout/Moby_Checkout.zip`
- `https://github.com/MobyPayTech/moby-plugins/tree/main/direct-integration`

## Environments

| Environment | Base URL |
|---|---|
| Sandbox | `https://dev.pay.mobycheckout.com` |
| Production | `https://pay.mobycheckout.com` |

- Sandbox and production use different Client ID / Secret Key pairs. Both are issued by MOBY Merchant Support (`merchantsupport@moby.my`); there is no self-serve key page documented.
- The deprecated GitHub recurring README lists a different sandbox host, `https://dev-pay-refactor.mobycheckout.com/`. Current docs use `https://dev.pay.mobycheckout.com` for everything, including recurring. Use the current one.
- `https://api.mobypay.my/api/v1` is the base for the Financing, Loyalty and Merchant Management APIs, which are all marked Coming Soon / "Not yet in production".
- Hosted payment pages are served from the same host (example `payLink`: `https://pay.mobycheckout.com/checkout/sess_abc789`).

## Authentication

`POST /api/auth/token` (no auth header). Body fields:

| Field | Type | Required | Notes |
|---|---|---|---|
| `clientId` | string | yes | Merchant Client ID (format like `MOBY00000123`). Also called "Merchant ID" in plugins. |
| `secretKey` | string | yes | Secret Key. Also called "API Key" in plugins. The same value signs callbacks. |
| `scope` | string | no | Default `pay`. |

- Current docs send JSON. The OpenAPI spec declares `multipart/form-data`. MOBY's plugins send `Content-Type: application/json`, so JSON works.
- The deprecated GitHub README spells the field `clientID`. Use `clientId` (current docs, OpenAPI, all plugins).

Success response (200):

```json
{ "status": "success", "success": true, "msg": "Authentication success",
  "token": "<TOKEN>", "expired_at": "2026-03-17T06:32:30.775097Z", "scope": "pay" }
```

Error (400): `{ "success": false, "error": "BAD_REQUEST", "msg": "Incorrect credentials!" }`.

Use `Authorization: Bearer <token>` on every other call.

- Token lifetime: 30 minutes per current docs and the recurring README notes. The deprecated hosted README says 60 minutes. Treat `expired_at` as the truth, refresh early, and re-authenticate on any `401`.
- Token string format varies (`2419|...` in docs, a JWT in the OpenAPI example). Treat it as opaque.

Scopes (request extra scopes from Merchant Support):

| Scope | Product |
|---|---|
| `pay` | Payments, Hosted Checkout, Recurring |
| `loyalty:read`, `loyalty:credit`, `loyalty:debit` | MobyCash wallet (Coming Soon) |
| `financing` | Credit assessment and loan APIs (Coming Soon) |

A second security scheme, `XToken` (header `x-token`), appears in the OpenAPI spec for `POST /api/bnpl/merchant/status` and the IPP plan endpoints. How a merchant obtains an `x-token` is not documented in official docs (checked 2026-10-21).

## Amount And Currency Rules

- Live v1 endpoints take amounts in ringgit as decimal numbers, for example `100.00` or `75.50`. Not sen.
- MOBY's Magento plugin sends `amount` as a 2-decimal string (`"100.00"`); the docs examples send a JSON number. Format to exactly 2 decimals.
- Currency defaults to `MYR`. The plugins refuse non-MYR stores ("Store currency must be set to MYR").
- The Coming Soon v2 API (`/v2/...`) uses integer cents. Do not mix the two; v2 is not live.
- Callback `amount` arrives as a string such as `"100.00"`. Compare as a 2-decimal string or as integer sen, never as a float.

## Endpoint Catalogue

Live (v1, `pay.mobycheckout.com` / `dev.pay.mobycheckout.com`):

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | `/api/auth/token` | none | Get Bearer token |
| POST | `/api/merchant/payment/checkout/hosted` | Bearer | Create hosted checkout session, returns `payLink` |
| POST | `/api/merchant/payment/method/list` | Bearer | List enabled payment methods (codes for `payMethod`) |
| POST | `/api/merchant/payment/direct/pay` | Bearer | Direct payment for one method, returns `payUrl` |
| GET | `/api/merchant/payment/verify/dn` | Bearer | Verify a DuitNow QR payment |
| POST | `/api/merchant/payment/status` | Bearer | Payment status by `referenceNo` + `transactionId` |
| POST | `/api/merchant/payment/detail` | Bearer | Payment detail with transaction history by `referenceNo` |
| POST | `/api/merchant/payment/capture` | Bearer | Capture a pre-authorised payment |
| POST | `/api/merchant/payment/void` | Bearer | Void before settlement, same day only |
| POST | `/api/merchant/payment/refund` | Bearer | Refund a settled payment |
| POST | `/api/omnipanel/payments/{transaction}/refund` | Bearer | Refund by MOBY transaction ID with amount (OpenAPI only) |
| POST | `/api/v2/tokens` | Bearer | Start card tokenisation (see `moby-recurring.md`) |
| GET | `/api/v2/tokens` | Bearer | List saved card tokens |
| DELETE | `/api/v2/tokens/{token}` | Bearer | Delete a token |
| POST | `/api/v2/tokens/{token}/charges` | Bearer | Charge a token (optional 3DS) |
| GET | `/api/v2/charges/{order_reference}` | Bearer | Token charge status |
| POST | `/api/v2/charges/{order_reference}/refund` | Bearer | Refund a token charge (async) |
| POST | `/api/v2/customers/mobile` | Bearer | Move customer mobile and consolidate tokens (OpenAPI only) |
| POST | `/api/merchant/subscription/list` | Bearer | List subscriptions (OpenAPI only) |
| POST | `/api/merchant/subscription/create` | Bearer | Create subscription (OpenAPI only) |
| POST | `/api/merchant/subscription/{subscriptionId}/update` | Bearer | Update subscription (OpenAPI only) |
| POST | `/api/merchant/subscription/subscriber/create` | Bearer | Create subscriber (OpenAPI only) |
| POST | `/api/merchant/subscription/subscriber/add` | Bearer | Add subscriber to subscription (OpenAPI only) |
| POST | `/api/merchant/subscription/subscriber/get` | Bearer | Get subscribers (OpenAPI only) |
| POST | `/api/merchant/subscription/subscriber/create-bulk` | Bearer | Bulk create subscribers (OpenAPI only) |
| GET | `/api/omnipanel/terminal/installment-plans` | x-token per OpenAPI | Eligible Visa IPP plans (see `moby-pos.md`) |
| POST | `/api/omnipanel/terminal/installment-plans` | x-token per OpenAPI | Submit chosen IPP plan |
| DELETE | `/api/visa/cancel-plan` | Bearer | Cancel Visa installment plan and refund |
| POST | `/api/bnpl/merchant/status` | x-token | Enable/disable BNPL merchant platforms `moby` / `moby-i` (platform use, OpenAPI only) |

Coming Soon / not in production (do not build against):

| Method | Path | Purpose |
|---|---|---|
| POST | `/v2/auth/token` | `client_id` + `client_secret`, 60 min token |
| POST | `/v2/payments/hosted` | Hosted checkout, amount in cents, returns `checkout_url` |
| POST | `/v2/payments/direct` | Direct payment, cents |
| GET | `/v2/payment-methods` | Method list, `type` filter `FPX`/`EWALLET`/`CARD`/`QR` |
| POST | `/v2/payments/{payment_id}/refund` | Refund, optional amount in cents, `reason` |
| GET | `/v2/tokens` | Token list |
| POST | `/v2/financing/assess` | Credit assessment |
| GET | `/v2/loyalty/wallet` | MobyCash wallet balance |
| GET | `/v2/merchants/{merchant_id}/reports` | Financial reports |
| various | `https://api.mobypay.my/api/v1/financing/...`, `/loyalty/...`, `/merchants/...` | Financing, loyalty, merchant profile, outlets, MDR settings, settlements |

The planned settlements endpoint (`GET /merchants/{id}/settlements`, scope `merchant:read`, statuses `pending`/`completed`/`failed`) is "Not yet in production". Use the dashboard settlement reports today.

## Hosted Checkout (recommended path)

Flow (docs, Hosted Checkout page):

1. Server gets a token: `POST /api/auth/token`.
2. Server creates a session: `POST /api/merchant/payment/checkout/hosted`.
3. Redirect the customer to `payLink`.
4. Customer pays on the MOBY hosted page (card, FPX, e-wallet, DuitNow QR, BNPL as enabled).
5. MOBY POSTs the result to `callbackUrl`.
6. MOBY redirects the customer to `returnUrl` with signed query parameters.

Request fields (docs API Reference, cross-checked with OpenAPI):

| Field | Type | Required | Notes |
|---|---|---|---|
| `clientId` | string | yes | Client ID |
| `customerName` | string | yes (docs) / optional (OpenAPI) | Send it |
| `customerEmail` | string | no (docs, OpenAPI) | Deprecated GitHub README says yes. Send it |
| `customerMobile` | string | no | OpenAPI: required when merchant has insurance enabled |
| `nric` | string | conditional | Mandatory for insurance merchants, ignored otherwise. OpenAPI names it `customerNric`. Confirm the field name with MOBY if you are an insurance merchant |
| `referenceNo` | string | yes | Your unique order reference. Echoed in callback |
| `details` | string | yes | Order description |
| `amount` | number | yes | MYR, 2 decimals |
| `callbackUrl` | string | no (docs) | Server-to-server result URL. Always send it; settlement depends on it |
| `returnUrl` | string | no (docs) | Browser redirect after payment |
| `currency` | string | no | Default `MYR` |
| `cart` | array | no | Cart items |
| `cart.products[].id` | numeric | no | Product ID |
| `cart.products[].name` | string | no | Product name |
| `cart.products[].price` | string | no | Unit price, e.g. `"100.00"` |
| `cart.products[].quantity` | string | no | Quantity |
| `metadata` | array | no | Extra metadata |
| `tags` | string | no | Comma-separated tags |
| `templateId` | string | no | Payment page template ID |
| `additionalInfo` | any | no | "passed through to callbacks". Plugins send an object with order id and cart |

Response 200: `{ "success": true, "payLink": "https://pay.mobycheckout.com/checkout/sess_abc789" }`. 401: `{ "success": false, "error": "Unauthorized" }`.

The response returns no session ID, so `referenceNo` is your only correlation key. Make it unique per payment attempt. MOBY's Magento plugin uses `MAGE-<increment_id>-<entity_id>`.

## Direct API

Build your own method picker, then send the customer to MOBY for the chosen method.

### List payment methods

`POST /api/merchant/payment/method/list`, body `{ "clientId": "<CLIENT_ID>" }` (docs send JSON; OpenAPI says multipart/form-data).

Response `data[]` fields: `id`, `payment_gateway_id`, `display_name` (OpenAPI: `displayName`), `code` (use as `payMethod`), `payment_gateway_code`, `currencies` (e.g. `"RM,MYR"`), `logo_url` (OpenAPI: `logo`), `redirect_url`, `status` (1 = enabled), `type`, `created_at`, `updated_at`, `deleted_at`, `pivot`.

Type values are inconsistent in the docs: the method pages say filter on `CARD`, `FPX`, `EWALLET`; the sample response shows `EW` for DuitNow QR and OpenAPI gives `CC` and `EW` as examples. Inspect the live list for your account and map types from it.

### Direct pay

`POST /api/merchant/payment/direct/pay`

| Field | Type | Required | Notes |
|---|---|---|---|
| `clientId` | string | yes | |
| `customerName` | string | yes | |
| `customerEmail` | string | yes | |
| `customerMobile` | numeric per docs | yes | Send as a digit string (e.g. `"0111222333"`) to keep the leading zero |
| `referenceNo` | string | yes | Unique order reference |
| `details` | string | yes | Description |
| `amount` | number | yes | MYR |
| `payMethod` | string | yes | `code` from the method list |
| `returnUrl` | string | no | Browser redirect |
| `currency` | string | no | Default `MYR` |
| `metadata`, `cart`, `cart.products[]...` | array | no | Same shape as hosted |
| `tags` | string | no | |
| `invoiceId` | string | no | External invoice ID |
| `linkId` | string | no | Payment link ID if from a payment link |
| `promoCodes` | string | no | Promo codes |
| `cc_no`, `cc_month` (MM), `cc_year` (YYYY), `cc_cvc` | numeric | no | Raw card fields "for direct card payment without redirect". Sending raw PAN puts you in full PCI DSS scope; use the redirect flow unless MOBY has approved you |
| `skip_receipt` | boolean | no | Skip receipt email |

Response 200: `{ "success": true, "payUrl": "https://pay.mobycheckout.com/pay/redirect/abc123" }`. 422: `{ "success": false, "error": "Validation error", "message": "The payMethod field is required." }`.

`callbackUrl` is not listed for direct pay in the docs or OpenAPI. How direct-pay results are notified server-to-server is not documented (checked 2026-10-21). Confirm with MOBY; until then poll `POST /api/merchant/payment/status` or `/detail` after the return redirect.

`payMethod` examples in the docs (accounts differ, always use the list): `MPGS` (card), `MB2U` (Maybank2u FPX), `TNG` (Touch 'n Go), `DUITNOW_QR`, `BNPL`. Codes also seen in responses: `RMS_RPP_DUITNOWQR`, `DN_RPP_ACFBMYK1`.

Per method:

- Cards: filter type `CARD`, redirect to `payUrl`. 3DS handled on MOBY's page.
- FPX: filter type `FPX`; `code` is the `payMethod`. Sandbox has only ACF Bank B2B.
- E-wallets (TNG, GrabPay, Boost, ShopeePay): filter `EWALLET`. No sandbox; tests are live charges.
- DuitNow QR: `payMethod` `DUITNOW_QR`, display the QR from `payUrl`.
- BNPL: `payMethod` `BNPL`, `payUrl` runs the BNPL approval flow. MOBY says its BNPL is shariah-compliant.

### Verify DuitNow payment

`GET /api/merchant/payment/verify/dn` with Bearer and query: `transaction_id` (req), `signature` (req, "HMAC-SHA256 signature"), `mobyMerchantId` (req), `clientId` (req), `reference_no` (req), `productId` (req), `merchantId` (req, "from the DuitNow payload"), `gps_coordinate` (opt), `ip_address` (opt).

Response: `{ "status": "success", "message": "Payment verified", "time": "2026-03-17 14:00:00" }`.

What the `signature` here is computed over, and where `productId` comes from, is not documented (checked 2026-10-21). Ask MOBY before using this endpoint; the status/detail endpoints cover normal reconciliation.

## Callback (Webhook) And Return Redirect

- MOBY sends an HTTP POST with a JSON body to `callbackUrl` when a payment event occurs. There is no dashboard webhook registry; the URL is set per request.
- MOBY's plugins also accept form-encoded and query-string input on the callback route, so read the JSON body first and fall back to form fields.
- The signature is a field in the body (`signature`). No signature header is documented.
- No source IP list is published.
- Acknowledge with HTTP 200 and body `{"status":"success"}`. The deprecated README says MOBY "may retry" without a 200. Retry schedule is not documented.
- After payment, the browser is redirected to `returnUrl` carrying the same signed fields as query parameters (deprecated README; plugins verify the HMAC on the return request too). Treat the return as UX; settle from the callback or a status query.

Callback payload:

| Field | Type | Notes |
|---|---|---|
| `referenceNo` | string | Your reference from the create call |
| `transactionId` | string | MOBY transaction ID (may be partly masked, e.g. `******11196916084427898`) |
| `amount` | string | e.g. `"100.00"` |
| `payMethod` | string | e.g. `MB2U` |
| `status` | string | see status values |
| `time` | string | `YYYY-MM-DD HH:mm:ss` |
| `merchantOrderRef` | string or null | Often `null` |
| `description` | string | e.g. `Approved` |
| `signature` | string | 64-char lowercase hex HMAC-SHA256 |

Status values: docs list `success`, `failed`, `pending`. MOBY's WooCommerce and Magento plugins also handle `cancelled` and `processing`. Only `success` is paid. Treat unknown values as not paid.

Return URL query-string gotcha: MOBY's plugins read both `status` and `?status`, `transactionId` and `?transactionId`, and strip `?referenceNo`. This suggests MOBY appends `?field=` even when `returnUrl` already contains a query string, producing keys that start with `?` (inferred from plugin code, not stated in the docs). Use a `returnUrl` and `callbackUrl` without an existing query string, or normalise keys by stripping a leading `?` before verifying.

## Signature Verification (HMAC-SHA256)

Key: your Secret Key (the same `secretKey` used for `/api/auth/token`; plugins call it "API Key").

Documented algorithm (Signature Verification page):

1. Take these 8 fields in this exact order (alphabetical by key): `amount`, `description`, `merchantOrderRef`, `payMethod`, `referenceNo`, `status`, `time`, `transactionId`.
2. Convert each value to a string. Missing or `null` becomes the empty string `""`.
3. Concatenate with no separator.
4. `hex(HMAC_SHA256(key = secretKey, message = concatenated))`, lowercase hex.
5. Compare with `signature` in constant time.

```text
fields = [amount, description, merchantOrderRef, payMethod, referenceNo, status, time, transactionId]
msg    = join("", [str(payload[f]) if payload[f] is not null else "" for f in fields])
expect = lower_hex(hmac_sha256(SECRET_KEY, msg))
valid  = constant_time_equal(expect, payload.signature)
```

Example string-to-sign for the documented callback (merchantOrderRef null):

```text
100.00ApprovedMB2UORDER-001success2026-03-17 14:52:52******11196916084427898
```

How MOBY's own plugins do it (WooCommerce v2.0 `validate_hmac`, Magento `SignatureValidator`, OpenCart 4):

- Merge all received params (query + form + JSON), remove `signature` plus their own URL params (`key`, `mobypay_order_id`, `type`, `route`, `?referenceNo`), require non-empty `referenceNo`, `transactionId`, `amount`, `payMethod`, `status`.
- `ksort` the remaining keys, concatenate every scalar value (Magento skips arrays), HMAC-SHA256 with the API Key, `hash_equals`.

When the body contains exactly the 8 documented fields plus `signature`, both methods give the same string. If MOBY ever adds a field (for example `additionalInfo` echoed back), the two methods diverge. Use the documented 8-field list, log the raw payload on mismatch, and ask MOBY rather than loosening the check.

Pitfalls:

- JavaScript: `String(null)` is `"null"`. Map `null`/`undefined` to `""` first. The deprecated GitHub JS sample that sorts all keys would hash `"null"` for a null `merchantOrderRef`.
- Do not reformat `amount` (`"100.00"` must stay `"100.00"`) or `time` before hashing. Hash the values exactly as received.
- Do not add your own query params to `callbackUrl` unless you strip them before hashing (that is why the plugins remove `mobypay_order_id` and `type`).
- The docs mention "test vectors" but publish none (checked 2026-10-21). Build your own vector in tests with a dummy key.

## Status And Detail Queries (requery)

`POST /api/merchant/payment/status`

| Field | Type | Required |
|---|---|---|
| `clientId` | string | yes |
| `referenceNo` | string | yes |
| `transactionId` | string | yes |

Response (docs): `{ "success": true, "data": { "referenceNo", "merchantId", "status", "amount", "payAt", "createdAt", "customer": { "name", "email", "phone" } } }`. OpenAPI also lists `transactionId` ("Only present when status is 'success'") and `returnUrl`.

`POST /api/merchant/payment/detail` (needs only `clientId`, `referenceNo`; good when you have no `transactionId` yet)

Response fields: `referenceNo`, `merchantId`, `status`, `amount`, `payAt`, `createdAt`, `customer{name,email,phone}`, `transactions[]{transactionId, status, amount, payMethod, createdAt, payAt}`. One reference can have several transaction attempts; settle only when an attempt has `status` `success` and the amount matches.

## Capture, Void, Refund

Capture: `POST /api/merchant/payment/capture`, fields `clientId`, `referenceNo`. 200 `{ "success": true, "status": "captured", "msg": "Payment captured successfully" }` (OpenAPI example `CAPTURED`, `"Payment is captured"`); 400 `{ "success": false, "error": "Payment already captured" }`. How to create a pre-authorised (auth-only) payment is not documented (checked 2026-10-21).

Void: `POST /api/merchant/payment/void`, must be called the same day as the original transaction.

| Field | Required | Notes |
|---|---|---|
| `clientId` | yes | |
| `referenceNo` | yes | |
| `transactionId` | docs: yes; OpenAPI: optional | "If omitted, the first paid transaction is used" |
| `callback_url` | no (OpenAPI) | Notified when void is processed |

200 `{ "success": true, "status": "voided", "msg": "Payment voided successfully" }`.

Refund: `POST /api/merchant/payment/refund`, "Refunds a settled payment. Can be full or partial."

| Field | Required | Notes |
|---|---|---|
| `clientId` | yes | |
| `referenceNo` | yes | |
| `transactionId` | docs: yes; OpenAPI: optional | First paid transaction used if omitted |
| `callback_url` | no (OpenAPI) | Notified when refund is processed |
| `amount` | not in docs or OpenAPI | MOBY's WooCommerce plugin sends `amount` (refund amount) and `callback_url`. Confirm partial-refund field with MOBY |

200 `{ "success": true, "status": "refunded", "msg": "Refund processed successfully" }`. WooCommerce treats the refund as "being processed" and marks it refunded only when a later callback with `status` `success` arrives on the refund `callback_url`. Do the same: a refund is final only after the signed refund callback or a status check.

Alternative refund (OpenAPI only): `POST /api/omnipanel/payments/{transaction}/refund?clientId=<CLIENT_ID>`, body `{ "amount": <float> }` (required). 200 `{ "message": "Refund Successful" }`; 400 not allowed, 401, 404 transaction not found, 500.

Dashboard refunds: Merchant Dashboard > Refund & Disputes (make refund request, track status).

## Sandbox Test Data (sandbox only)

Sandbox credentials come from `merchantsupport@moby.my`.

Cards (docs; the deprecated sandbox README adds two Visa numbers):

| Card | 3DS |
|---|---|
| `5123450000000008` | yes |
| `2223000000000007` | yes |
| `5111111111111118` | no |
| `2223000000000023` | no |
| `4508750015741019` | yes (deprecated README only) |
| `4012000033330026` | no (deprecated README only) |

Expiry drives the result: `01/39` APPROVED, `05/39` DECLINED, `04/27` EXPIRED_CARD; deprecated README adds `08/28` TIMED_OUT, `01/37` ACQUIRER_SYSTEM_ERROR, `02/37` UNSPECIFIED_FAILURE, `05/37` UNKNOWN.

CVV: `100` MATCH, `102` NO_MATCH; deprecated README adds `101` NOT_PROCESSED.

FPX: only ACF Bank B2B, username `user1`, password `password1`.

E-wallets: no sandbox. Any e-wallet test is a real live charge. The deprecated README advises testing with less than RM 5 and contacting MOBY for a refund.

## Rate Limits

| Type | Limit |
|---|---|
| Authentication requests | 10 per minute per client |
| General API requests | 60 per minute per token |

Over the limit: HTTP 429 with a `Retry-After` header, body like `{ "success": false, "error": "TOO_MANY_REQUESTS", "message": "Rate limit exceeded. Please retry after 30 seconds." }`. Cache the token instead of authenticating per request.

## Errors

| HTTP | Meaning |
|---|---|
| 400 | Bad request, missing/invalid params, bad credentials on token call |
| 401 | Invalid or expired token: re-authenticate once and retry |
| 403 | Access denied (ownership or permissions, unauthorized merchant) |
| 404 | Transaction or resource not found |
| 422 | Validation failed (e.g. missing `payMethod`) |
| 429 | Rate limited, honour `Retry-After` |
| 500 | Server error |

Common body: `{ "success": false, "error": "<message>" }`. Card declines on token charges look like `{ "success": false, "error": "DECLINED - Do not honor", "message": "05 - Do not honor" }`.

## Plugins (no-code)

| Platform | Credentials it asks for | Notes |
|---|---|---|
| WooCommerce Moby Checkout v2.0 | Environment, Merchant ID, API Key, Company Name | WP 5.0+, WC 5.0+, PHP 7.4+, MYR store. Server must be public for callbacks |
| WooCommerce Price Divider v1.3 | none | Shows 2/3/6 instalment pricing, Moby or Moby Islamic branding |
| Shopify Moby Checkout (`apps.shopify.com/moby-checkout`) | API Key, Secret Key, Test/Live | |
| Shopify Moby Islamic BNPL (`apps.shopify.com/moby-islamic`) | Merchant ID, Merchant API Key, Billing API URL (example `https://dev.external-api.moby.my/`), API Version `2` | |
| Shopify / EasyStore Price Divider | none | Display widget |
| Magento 2 `Moby_Checkout` | Mode test/live, Merchant ID, API Key, Company Name | Log at `var/log/moby_checkout.log` |
| OpenCart 1.0 to 4.0 `mobypay.ocmod.zip` | API Key, Secret Key | |
| EasyStore Moby Checkout | Client ID, Secret Key | |

In plugins "Merchant ID" = Client ID and "API Key" = Secret Key.
