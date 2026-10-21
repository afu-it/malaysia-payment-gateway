# Revenue Monster API Reference

Use this reference for factual Revenue Monster (RM) Open API guidance. Checked against official docs and official GitHub SDKs on 2026-10-21. Verify current Merchant Portal settings, method activation, pricing, and settlement when those details matter.

## Source URLs

- `https://doc.revenuemonster.my/docs/introduction/overview`
- `https://doc.revenuemonster.my/docs/quickstart/accesstoken/client-credentials`
- `https://doc.revenuemonster.my/docs/quickstart/accesstoken/refresh-token`
- `https://doc.revenuemonster.my/docs/quickstart/signature-algorithm`
- `https://doc.revenuemonster.my/docs/quickstart/verify-signature`
- `https://doc.revenuemonster.my/docs/v2/payment/online-payment`
- `https://doc.revenuemonster.my/docs/v2/payment/query-transaction`
- `https://doc.revenuemonster.my/docs/v2/payment/cancel-transaction`
- `https://doc.revenuemonster.my/docs/v2/payment/reconciliation`
- `https://doc.revenuemonster.my/docs/v2/payment/tokenization-payment`
- `https://doc.revenuemonster.my/docs/v2/payment/online-payment-backup`
- `https://doc.revenuemonster.my/docs/digital-wallet/apple-pay`
- `https://doc.revenuemonster.my/docs/digital-wallet/google-pay`
- `https://doc.revenuemonster.my/docs/downloads/testing-wallets`
- `https://doc.revenuemonster.my/docs/quickstart/accesstoken/authorization-code`
- `https://doc.revenuemonster.my/docs/v2/plugin/integration`
- `https://doc.revenuemonster.my/sitemap.xml` (endpoint catalogue; every listed page, see `source-map.md`)
- `https://doc.revenuemonster.my/docs/payment-method`
- `https://doc.revenuemonster.my/docs/bank-code`
- `https://doc.revenuemonster.my/docs/error-codes`
- `https://github.com/RevenueMonster/rm-go-sdk/blob/master/signature.go`
- `https://github.com/RevenueMonster/rm-js-sdk/blob/main/src/signature.ts`
- `https://github.com/RevenueMonster/rm-php-sdk/blob/main/src/Modules/Module.php`
- `https://github.com/RevenueMonster/woocommerce-revenuemonster-payment-gateway`

## Integration Surfaces

- Hosted Web Payment / Mobile Payment checkout (`POST /v3/payment/online`). Default for a custom web app.
- Direct Payment Checkout (`POST /v3/payment/online/checkout`): your own UI on top of a checkout session (URL mode, QR mode, DuitNow QR, FPX bank pick, card web, card native, Apple Pay / Google Pay, Alipay / WeChat mini program).
- Quick Pay (merchant scans customer wallet code), terminal integration, deeplink.
- Tokenization: recurring payment and tokenized customer.
- Low-code checkout button, WooCommerce, OpenCart, EasyStore plugins.
- Invoice (payment link) API.

## Reference Files

- `revenue-monster-objects.md`: Transaction Object and nested objects, transaction type/status enums, method and region matrix, system method names, full FPX B2C/B2B bank codes.
- `revenue-monster-tokenization.md`: recurring payment, tokenized payment, customer APIs, card-on-file tokens (full field tables).
- `revenue-monster-instore.md`: Quick Pay, terminal events, POS, Android deeplink.
- `revenue-monster-invoice.md`: invoice / payment link API.
- `revenue-monster-errors.md`: every documented error code.

## Environments

| Service | Sandbox | Production |
| --- | --- | --- |
| OAuth server | `https://sb-oauth.revenuemonster.my` | `https://oauth.revenuemonster.my` |
| Open API server | `https://sb-open.revenuemonster.my` | `https://open.revenuemonster.my` |
| Hosted checkout page (returned in `item.url`) | `https://sb-pg.revenuemonster.my/...` | returned by API |
| Card native API | `https://sb-card.revenuemonster.my/checkout` | `https://card.revenuemonster.my/checkout` |
| Merchant Portal | `https://sb-merchant.revenuemonster.my/` | `https://merchant.revenuemonster.my/` |

OAuth paths use `/v1`; payment paths use `/v3` (the Go SDK defaults are `v1` and `v3`). `API_INACTIVE` means you called an old version; migrate to v3. Other documented hosts: eKYC liveness `https://ekyc.revenuemonster.my/` (sandbox `https://sb-ekyc.revenuemonster.my/`); low-code button assets on `rm-component.oss-ap-southeast-3.aliyuncs.com` (sandbox `rm-component-sandbox...`).

## Endpoint Catalogue

Every operation on the live docs site (sitemap of `doc.revenuemonster.my`, checked 2026-10-21). Paths are relative to the Open API host unless a host is shown. "Fields" says where full field tables live. Payment operations have full tables in this skill; non-payment products (loyalty, voucher, a la carte, eKYC, Visa VOP, SMS, partner onboarding, account settings) are listed for completeness with purpose only. See `source-map.md` for why.

### Auth (OAuth host)

| Method | Path | Purpose | Fields |
| --- | --- | --- | --- |
| POST | `/v1/token` (`grantType: client_credentials`) | Access token for your own merchant | this file |
| POST | `/v1/token` (`grantType: refresh_token`) | Refresh access token | this file |
| GET | `/authorize?responseType=code&clientId=&redirectUri=&scope=&state=` | Partner/plugin authorization code (browser) | this file |
| POST | `/v1/token` (authorization code exchange) | Exchange the one-time code for a token (plugin developers). The exchange body (grant type string, code field name) is not documented in official docs (checked 2026-10-21) | this file |

### Online Payment

| Method | Path | Purpose | Fields |
| --- | --- | --- | --- |
| POST | `/v3/payment/online` | Create hosted checkout (WEB_PAYMENT / MOBILE_PAYMENT) | this file |
| GET | `/v3/payment/online?checkoutId={id}` | Query checkout session | this file |
| POST | `/v3/payment/online/checkout` | Direct Payment Checkout (URL, FPX, card web, QRCODE, DUITNOW_QRCODE, Apple/Google Pay, mini program) | this file |
| GET | `/v3/payment/fpx-bank` | FPX bank list with `isOnline` | this file, objects |
| POST | `https://card.revenuemonster.my/checkout` | Card native (raw card to 3DS URL) | this file |
| (callback) | your `redirectUrl` (GET) | Browser return | this file |
| (callback) | your `notifyUrl` (POST, `PAYMENT_WEB_ONLINE`) | Server notify on success | this file |

### Transaction, Refund, Settlement

| Method | Path | Purpose | Fields |
| --- | --- | --- | --- |
| GET | `/v3/payment/transaction/order/{orderId}` | Query by order ID | this file, objects |
| GET | `/v3/payment/transaction/{transactionId}` | Query by transaction ID | this file, objects |
| GET | `/v3/payment/transactions?limit=N` | List transactions | this file |
| POST | `/v3/payment/refund` | Refund | this file |
| POST | `/v3/payment/reverse` | Reverse (timeouts) | this file |
| POST | `/v3/payment/reconciliation` | Daily reconciliation / settlement amounts | this file |

### Tokenization And Recurring (`revenue-monster-tokenization.md`)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/v3/recurring-payment` | Create recurring customer (RM schedules charges) |
| POST | `/v3/tokenized-payment` | Create tokenized customer (you trigger charges) |
| GET | `/v3/customer/{customer_id}` | Customer info |
| GET | `/v3/customer/{customer_id}/orders` | Customer orders |
| PUT | `/v3/customer/{customer_id}/status` | Toggle customer active |
| POST | `/v3/customer/{customer_id}/order` | Charge the bound card |
| GET | `/v3/payment/tokens/{customer_id}` | Card-on-file tokens |
| DELETE | `/v3/payment/tokens/{customer_id}` | Delete a token |
| (callback) | your `notifyUrl` (`RECURRING_PAYMENT`) | Recurring charge success |

### Invoice (`revenue-monster-invoice.md`)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/api/v3/invoice` | Create invoice (payment link) |
| GET | `/api/v3/invoices` | List invoices |
| GET | `/api/v3/invoice/{id}` | Get invoice |
| PATCH | `/api/v3/invoice/{id}` | Edit pending invoice |
| POST | `/api/v3/invoice/{id}/cancel` | Cancel pending invoice |

### In-Store (`revenue-monster-instore.md`)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/v3/payment/quickpay` | Quick Pay (scan customer code; voucher, loyalty, membership variants) |
| POST | `/v3/membership/verify` | Verify Alipay Gourmet membership card |
| POST | `/v3/payment/terminal/quickpay` | Terminal payment event (`E-WALLET`, `CARD`, `RETAIL-QR`) |
| POST | `/v3/event/terminal` | Terminal `REFUND`, `SETTLEMENT`, `CANCEL` events |
| (Android intent) | `REVENUE_MONSTER_PAYMENT` | Deeplink on RM terminal (types 1 to 5) |

### Merchant, Store, User Settings (purpose only)

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/v3/merchant` | Merchant profile |
| GET | `/v3/merchant/subscriptions` | Product subscriptions (`1000` Loyalty and Voucher, `1001` Payment, `1002` Market Place, `1003` Social Media) |
| GET | `/v3/user` | User profile |
| GET | `/v3/stores` | List stores (find Store ID) |
| GET | `/v3/store/{storeId}` | Get store |
| POST | `/v3/store` | Create store (max 10 stores, `STORE_MAXIMUM`) |
| PATCH | `/v3/store/{storeId}` | Update store |
| DELETE | `/v3/store/{storeId}` | Delete store (irreversible; default store cannot be deleted) |
| GET | `/v3/accounts` | List business accounts |
| GET | `/v3/account/{id}` | Get account |
| POST | `/v3/account` | Create account |
| PUT | `/v3/accounts/{id}` | Update account |
| POST | `/v3/account/{id}/review` | Submit account for review |
| PUT | `/v3/account/{id}/agent-rate` | Update account agent rate |

### Merchant Wallet (purpose only; prepaid credit, not customer payments)

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/v3/wallet/credit` | Wallet balance |
| GET | `/v3/wallet/history` | Credit history (`TOPUP`, `DEDUCT`) |
| GET | `/v3/wallet/transaction` | Top-up history |
| POST | `/v3/wallet/topup` | Top-up URL (`redirect`, `amount`) |

### Loyalty, Member, Voucher (purpose only)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/v3/loyalty/reward` | Give loyalty points |
| DELETE | `/v3/loyalty/reward` | Deduct loyalty points |
| POST | `/v3/loyalty/spending-reward` | Spending loyalty point |
| POST | `/v3/loyalty/spending-reward/calculate` | Calculate spending reward |
| POST | `/v3/loyalty/spending-reward/cancel` | Cancel spending loyalty point |
| GET | `/v3/loyalty/balances` | Loyalty balances |
| POST | `/v3/loyalty/balances` | Spend loyalty balance |
| GET | `/v3/loyalty/members` | List members |
| POST | `/v3/loyalty/members` | Bulk create members (limit 1,000) |
| GET | `/v3/loyalty/member/{memberId}` | Member by ID |
| GET | `/v3/loyalty/member/{memberId}/history` | Member history |
| POST | `/v3/loyalty/member/authorize` | Member authorize |
| POST | `/v3/loyalty/member/{memberId}/topup-offline` | Top up member balance via Quick Pay |
| POST | `/v3/loyalty/member/{memberId}/topup-online` | Top up member balance via payment gateway (`topUpAmount`, `redirectUrl`; returns `item.paymentUrl`) |
| POST | `/v3/loyalty/member` | Register loyalty member |
| POST | `/v3/loyalty/member/check` | Check loyalty member |
| GET | `/v3/loyalty/me?countryCode=&phoneNumber=` | Member profile |
| GET | `/v3/loyalty/me/rewards` | Member rewards |
| GET | `/v3/loyalty/me/reward/{rewardId}` | Reward by ID |
| POST | `/v3/loyalty/me/reward/{rewardId}/redeem` | Redeem reward |
| GET | `/v3/loyalty/me/vouchers` | Member vouchers |
| GET | `/v3/loyalty/me/voucher/{code}` | Member voucher by code |
| POST | `/v3/loyalty/me/voucher/{code}/redeem` | Redeem member voucher |
| POST | `/v3/loyalty/chop-stamp/card/scan` | Give chop stamp |
| GET | `/v3/voucher-batches` | Voucher batches |
| GET | `/v3/voucher-batch/{key}/vouchers` | Vouchers in batch |
| POST | `/v3/voucher-batch/{key}/issue` | Issue voucher |
| GET | `/v3/voucher/{code}` | Voucher by code |
| POST | `/v3/voucher/{code}/void` | Void voucher |
| PATCH | `/v3/voucher/{code}/reinstate` | Reinstate voucher |
| POST | `/v3/vouchers/redeem` | Bulk redeem vouchers |

Voucher callbacks (voucher overview page): retried 5 times at 1-minute intervals until your server returns HTTP 200; if unreachable, call Get Voucher By Code. This retry rule is documented for vouchers only, not for payment notify.

### A La Carte (food ordering) via `POST /v3/service` (purpose only)

Body `{"service":"alacarte","version":"v1","function":"<name>","request":{...}}`, signed like other calls. Functions: `create.category`, `create.item`, `get.store.items`, `get.categories`, `get.category.items`, `update.category`, `update.item`, `update.item.quantity`, `get.store.order`, `get.store.orders`, `refund.store.order`, `update.order.status`, `set.notification.webhook` (max 3 `notificationUrls`, optional `storeIds`), `get.store.settings`, `update.store.avaiablility` (official spelling), `update.store.delivery`, `update.store.types`.

### eKYC via `POST /v3/service` (purpose only)

`service: "ekyc"`, functions `id-mykad` (MyKad recognition), `get-mykad-result`, `face-compare`, `get-ekyc-result`. Liveness check with face verification is a hosted page at `https://ekyc.revenuemonster.my/` (sandbox `sb-ekyc`) taking `mykad_request_id` and `query_image_content`.

### Other (purpose only)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/v3/sms` | Send SMS / TAC (`countryCode`, `phoneNumber`, `message`, `type`) |
| POST | `/v3/push-notification/merchant` | Push notification to merchant app |
| POST | `/v3/vop/enroll-user` | Visa Offers Platform enroll user |
| POST | `/v3/vop/enroll-card` | VOP enroll card |
| DELETE | `/v3/vop/unenroll-card` | VOP unenroll card |
| DELETE | `/v3/vop/unenroll-user` | VOP unenroll user |
| (callback) | your webhook | VOP transaction webhook (`userId`, `cardId`, `transactionId`, `transactionAt`, `amount`, `currency` numeric ISO, `merchantCategoryCode`) |

### Partner Merchant Onboarding (partner accounts only; purpose only)

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/v3/partner/merchant` | Create sub-merchant |
| GET | `/v3/partner/merchants` | List merchants |
| GET | `/v3/partner/merchant/{merchantId}` | Get merchant |
| PATCH | `/v3/partner/merchant/{merchantId}` | Update merchant |
| POST | `/v3/partner/merchant/{merchantId}` | Submit merchant for review |
| POST | `/v3/partner/file/upload` | Upload merchant file |
| POST | `/v3/partner/merchant/{merchantId}/application` | Create application client for merchant |
| GET | `/v3/partner/merchant/{merchantId}/applications` | List application clients |
| PUT | `/v3/partner/merchant/{merchantId}/application/{clientId}` | Update application client |

### Plugin Platform (no REST endpoint)

Plugins built on the RM plugin platform receive a signed request `<sig>.<payload>`: verify by HMAC-SHA256 of the second segment with the plugin secret key, compare with the Base64URL-decoded first segment; payload is Base64URL JSON (`merchantId`, `nonceStr`, `platform`, `referenceId`, `referenceRole`, `storeIds`, ...). Webview plugins talk to the app with `postMessage` actions such as `FINISH_HANDSHAKE`. Source: `https://doc.revenuemonster.my/docs/v2/plugin/integration`.

### Legacy Pages (GitHub doc source only, not on the live site)

The official doc source repo `github.com/RevenueMonster/doc-v2` still holds an older `docs/payment/` tree that the live site no longer serves (those URLs return the site's fallback page). Most entries duplicate the v2 pages above. Endpoints that appear only there: `GET /v3/payment/online/qrcode?checkoutId=&method=` (individual QR), `POST /v3/payment/transaction/qrcode` and `GET /v3/payment/transaction/qrcodes` (Transaction QR), `GET /v3/payment/transaction/qrcode/{code}`, `GET /v3/payment/transaction/qrcode/{code}/transactions`. Treat these as unsupported unless RM confirms them.

## Authentication (OAuth 2.0 Client Credentials)

```text
POST https://sb-oauth.revenuemonster.my/v1/token
Content-Type: application/json
Authorization: Basic base64("<CLIENT_ID>:<CLIENT_SECRET>")

{"grantType":"client_credentials"}
```

Response fields: `accessToken`, `tokenType` (`Bearer`), `expiresIn` (seconds; client-credentials example `2591999`, about 30 days; the refresh-token page example shows `72591999` while also saying 30 days, so read the value instead of assuming), `refreshToken`, `refreshTokenExpiresIn` (example `1576799999`). The docs type these as String but the examples are JSON numbers; parse both.

Scopes (authorization code grant): `manage_payment`, `manage_store`, `get_merchant_profile`, `get_user_profile`, comma-separated without spaces. The code is single-use. OAuth errors (`InvalidClient`, `InActiveClient`, `InvalidGrant`, ...) are listed in `revenue-monster-errors.md`.

Refresh: same URL and Basic header, body `{"grantType":"refresh_token","refreshToken":"<REFRESH_TOKEN>"}`. RM recommends refresh tokens over re-sending client credentials. The authorization code grant (`/authorize?responseType=code&clientId=...&redirectUri=...&scope=manage_payment`) is for plugin/partner developers only.

Every Open API call then needs:

```text
Authorization: Bearer <ACCESS_TOKEN>
Content-Type: application/json
X-Signature: sha256 <BASE64_RSA_SIGNATURE>
X-Nonce-Str: <RANDOM_32_CHARS>
X-Timestamp: <UTC_UNIX_SECONDS>
```

## Signature Algorithm (Request Signing)

Algorithm: RSA PKCS#1 v1.5 with SHA-256 (`signType` `sha256`), signed with YOUR private key. Your matching public key must be uploaded to the Merchant Portal application. Suggested key size 2048 bit.

Steps (docs, confirmed by Go, JS and PHP SDK source):

1. Take the JSON request body. Sort keys alphabetically, recursively for nested objects. Compact it (no whitespace).
2. Escape `<` as `\u003c`, `>` as `\u003e`, `&` as `\u0026` in the JSON string.
3. Base64 (standard, with padding) the UTF-8 bytes.
4. Build the plain text in this exact order (alphabetical keys):

```text
data=<BASE64_BODY>&method=<lowercase http method>&nonceStr=<NONCE>&requestUrl=<FULL URL CALLED>&signType=sha256&timestamp=<UNIX_SECONDS>
```

5. Sign the plain text with RSA-SHA256 using the private key; Base64 the signature.
6. Send `X-Signature: sha256 <signature>`, plus `X-Nonce-Str` and `X-Timestamp` with the same values used in the string.

Rules:

- No body (GET, or body `{}` / `null`): omit the `data=` pair entirely; the string starts with `method=`.
- `requestUrl` must be exactly the URL called, including host, path and any query string.
- `method` is lowercase (`post`, `get`, `delete`).
- `X-Nonce-Str`: no spaces, unique for at least 120 seconds, else duplicate request error.
- `X-Timestamp`: UTC Unix seconds; max 120 seconds difference from RM server, else `INVALID_TIMESTAMP`.
- Arrays keep their order. The JS SDK sorts nested objects but not objects inside arrays; the Go SDK sorts everything. Keep arrays of plain strings (for example `method: []`) to avoid ambiguity.
- On `INVALID_REQUEST_SIGNATURE`, sandbox returns `debug.preVerifyContent` showing RM's step1 to step4 strings and per-header validity. Diff your string against it.

Pseudo-code (Node):

```js
const sortDeep = o => Array.isArray(o) ? o
  : (o && typeof o === 'object')
    ? Object.keys(o).sort().reduce((a, k) => (a[k] = sortDeep(o[k]), a), {})
    : o;
const esc = s => s.replace(/</g,'\\u003c').replace(/>/g,'\\u003e').replace(/&/g,'\\u0026');

function signRM({ body, method, url, privateKeyPem }) {
  const nonceStr = crypto.randomBytes(16).toString('hex');           // 32 chars
  const timestamp = Math.floor(Date.now() / 1000).toString();
  const parts = [];
  if (body && Object.keys(body).length) {
    parts.push('data=' + Buffer.from(esc(JSON.stringify(sortDeep(body)))).toString('base64'));
  }
  parts.push('method=' + method.toLowerCase(), 'nonceStr=' + nonceStr,
             'requestUrl=' + url, 'signType=sha256', 'timestamp=' + timestamp);
  const sig = crypto.createSign('SHA256').update(parts.join('&')).sign(privateKeyPem, 'base64');
  return { 'X-Signature': 'sha256 ' + sig, 'X-Nonce-Str': nonceStr, 'X-Timestamp': timestamp };
}
// Send the same JSON you signed (sorted, compact) as the HTTP body.
```

The Go SDK parses the private key with `x509.ParsePKCS1PrivateKey`, so it expects a PKCS#1 PEM (`-----BEGIN RSA PRIVATE KEY-----`). Node and PHP OpenSSL accept PKCS#1 or PKCS#8.

## Verify Signature (Responses and Callbacks)

RM documents verifying content it sends you:

1. Sort the received JSON keys alphabetically (nested too) and compact it.
2. Base64 it.
3. Build `data=<b64>&method=<method>&nonceStr=<nonce>&requestUrl=<url>&signType=sha256&timestamp=<ts>`, taking nonce and timestamp from the response headers.
4. "If it's verifying our callback then the requestUrl can be skip."
5. Verify with RSA-SHA256 using the RM server public key (Merchant Portal > Developer > Applications > Server public key).

The Verify Signature page's worked example prints `requestUrl=` as the last pair (after `timestamp`), unlike the alphabetical order in its own step table and in the signing page. The Go SDK `formData()` sorts the pairs alphabetically for both `Generate` and `Validate`, and the PHP SDK also builds them in alphabetical order, so use alphabetical order.

Go SDK `Validate()` builds the same sorted string and drops pairs whose value is empty (so a skipped `requestUrl` disappears). PHP SDK `verifySignature()` always includes `requestUrl=` even when empty. Try the Go/doc form (pair omitted) first.

Not documented in official docs (checked 2026-10-21): the exact header names RM sends on notify callbacks. Assume `X-Signature` / `X-Nonce-Str` / `X-Timestamp` only after you see them in a sandbox notify. The official WooCommerce plugin does not verify the notify signature; it settles by calling `GET /v3/payment/transaction/order/{orderId}`. Do the same: signed requery is the settlement source of truth.

## Hosted Payment Checkout

```text
POST https://sb-open.revenuemonster.my/v3/payment/online
POST https://open.revenuemonster.my/v3/payment/online
```

Request fields:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `storeId` | String | Yes | Store ID from portal or `GET /v3/stores`. |
| `redirectUrl` | String URL | Yes | Browser return. Any URL the browser can open (web, deeplink). |
| `notifyUrl` | String URL | Yes | Server callback. |
| `layoutVersion` | String | Yes | Always `v4`. |
| `type` | String | Yes | `WEB_PAYMENT` or `MOBILE_PAYMENT`. |
| `method` | []String | No | `[]` shows all enabled methods; otherwise system methods such as `FPX_MY`, `TNG_MY`. |
| `order.id` | String | Yes | Your order ID, max length 24. Must be unique. |
| `order.title` | String | Yes | Max 32. |
| `order.currencyType` | String | Yes | `MYR` only. |
| `order.amount` | Uint64 | Yes | Sen. `100` = RM 1.00. Minimum `10` (RM 0.10). |
| `order.detail` | String | No | Max 600. |
| `order.additionalData` | String | No | Max 128. |
| `customer.userId` | String | Conditional | Required when tokenization is enabled. |
| `customer.email`, `customer.countryCode`, `customer.phoneNumber` | String | No | Buyer info. |

Response: `code: "SUCCESS"`, `item.checkoutId`, `item.url` (hosted page, for example `https://sb-pg.revenuemonster.my/v4/checkout?checkoutId=...`). Errors: `error.code`, `error.message`, `error.debug` (sandbox only).

The official WooCommerce plugin uses the same URL for `redirectUrl` and `notifyUrl` and URL-encodes the query part before sending.

### Query Payment Checkout

```text
GET /v3/payment/online?checkoutId={checkoutId}
```

Query parameter `checkoutId` (required). Response:

| Field | Type | Notes |
| --- | --- | --- |
| `code` | String | `SUCCESS` |
| `error.code`, `error.message`, `error.debug` | String | `debug` sandbox only |
| `item.id` | String | Checkout ID |
| `item.type` | String | `WEB_PAYMENT`, `WEB_MOBILE_PAYMENT`, `MOBILE_PAYMENT` |
| `item.transactionId` | String | Use with Query Transaction |
| `item.order.id` | String | Max 24 |
| `item.order.title` | String | Max 32 |
| `item.order.currencyType` | String | `MYR` |
| `item.order.amount` | Uint64 | Sen |
| `item.order.detail` | String | Max 600 |
| `item.order.additionalData` | String | Max 128 |
| `item.platform` | String | `OPEN_API` |
| `item.method` | String | Available methods |
| `item.redirectUrl`, `item.notifyUrl` | String URL | |
| `item.startAt` | RFC3339 | Checkout start |
| `item.status` | String | `SUCCESS`, `FAILED`, `CANCELLED`, `EXPIRED` |
| `item.createdAt`, `item.updatedAt` | RFC3339 | |

Checkout is not the transaction; use `item.transactionId` with Query Transaction for full details.

### Direct Payment Checkout (own UI)

```text
POST /v3/payment/online/checkout
```

Body uses the `checkoutId` from hosted checkout creation plus a mode:

| Mode | Body | Response |
| --- | --- | --- |
| URL | `{"checkoutId","type":"URL","method":"TNG_MY"}` | `item.url` to redirect |
| FPX | add `"method":"FPX_MY","fpx":{"bankCode":"MB2U0227:B2C"}` | `item.url` (FPX) |
| Card (web) | `"method":"MASTERCARD_MY","type":"URL"` | `item.url`, `item.checkoutId` |
| QR | `"type":"QRCODE","method":"MAYBANK_MY"` | `item.qrcode.base64Image`, `item.qrcode.data` |
| DuitNow QR | `"type":"DUITNOW_QRCODE","method":""` | `item.qrcode.*` |
| Apple/Google Pay | `"method":"APPLE_PAY"` or `"GOOGLE_PAY"`, `"wallet":{"token":"<raw SDK token>"}` | `item.action: 3DS_CHALLENGE`, `item.url` |
| Mini program | `"type":"MINI_PROGRAM","method":"ALIPAY_CN"` or `WECHATPAY_CN` (+`userId`) | `item.data` (Base64) |

FPX bank list: `GET /v3/payment/fpx-bank` (`item.<code>.isOnline`). Card native: `POST https://card.revenuemonster.my/checkout` with `code` (checkoutId), `card.no`, `card.name`, `card.expiryMonth`, `card.expiryYear`, `card.securityCode`, `type: URL`; returns a 3DS challenge URL. RM does not state PCI requirements on this page; sending raw card numbers through your own server generally brings it into PCI DSS scope, so prefer hosted checkout or Card (web) unless the merchant has that compliance.

Direct Payment Checkout field tables:

| Mode | Request fields (all required unless noted) | Response fields |
| --- | --- | --- |
| URL | `checkoutId`, `type` = `URL`, `method` (system method) | `item.type` (`URL`), `item.url` |
| FPX | `checkoutId`, `type` = `URL`, `method` = `FPX_MY`, `fpx.bankCode` (see objects file) | `item.type`, `item.url` (FPX page) |
| Card (web) | `checkoutId`, `type` = `URL`, `method` = `MASTERCARD_MY` | `item.checkoutId`, `item.type`, `item.url` |
| Card (native) on `card` host | `code` (the `item.checkoutId` from Card (web)), `card.no` (String), `card.name` (String), `card.expiryMonth` (Integer), `card.expiryYear` (Integer, 4 digits), `card.securityCode` (String), `type` = `URL` | `item.action` (`3DS_CHALLENGE`), `item.type`, `item.url` (3DS URL) |
| Apple Pay / Google Pay | `checkoutId`, `type` = `URL`, `method` = `APPLE_PAY` or `GOOGLE_PAY`, `wallet.token` (raw token string from the wallet SDK) | `item.action` (`3DS_CHALLENGE`), `item.type`, `item.url` |
| QRCODE | `checkoutId`, `type` = `QRCODE`, `method` | `item.type` (`QRCODE`), `item.qrcode.base64Image`, `item.qrcode.data` |
| DuitNow QR | `checkoutId`, `type` = `DUITNOW_QRCODE`, `method` = `""` | `item.type`, `item.qrcode.base64Image`, `item.qrcode.data` |
| Alipay mini program | `checkoutId`, `type` = `MINI_PROGRAM`, `method` = `ALIPAY_CN` | `item.type`, `item.data` (Base64; decode and pass as `orderStr` to `my.tradePay`) |
| WeChat Pay mini program | `checkoutId`, `type` = `MINI_PROGRAM`, `method` = `WECHATPAY_CN`, `userId` (WeChat Open ID) | `item.data` (Base64 JSON; spread into `wx.requestPayment`) |

All responses also carry `code` or `error.code`, `error.message`, `error.debug`. The FPX response table labels `item.type` as `QRCODE`; treat `item.url` as the field that matters. Which methods support URL vs QRCODE mode is in the method matrix in `revenue-monster-objects.md`. Payment method icons: `https://github.com/RevenueMonster/Payment-Icon` (linked from the docs).

### Apple Pay And Google Pay (web)

Sources: `https://doc.revenuemonster.my/docs/digital-wallet/apple-pay`, `https://doc.revenuemonster.my/docs/digital-wallet/google-pay`.

Apple Pay prerequisites: RM merchant with Apple Pay enabled; HTTPS; Apple Developer membership with a Merchant ID, Payment Processing Certificate (from a CSR that RM's integration team gives you) and Merchant Identity Certificate; verified domains. Use the browser Payment Request API with `supportedMethods: "https://apple.com/apple-pay"` and `data`: `version`, `merchantIdentifier`, `merchantCapabilities` (include `supports3DS`; confirm others with RM), `supportedNetworks` (`visa`, `masterCard`), `countryCode` `MY`. `total.amount.currency` `MYR`, `total.amount.value` a decimal string such as `"10.00"` (not sen, not a number). Handle `onmerchantvalidation` on your server, then pass `paymentResponse.details.token` as `wallet.token`.

Google Pay: `allowedAuthMethods` `PAN_ONLY`, `CRYPTOGRAM_3DS`; `allowedCardNetworks` `MASTERCARD`, `VISA`; `tokenizationSpecification.type` `PAYMENT_GATEWAY` with `parameters.gateway` `revenuemonster` and `parameters.gatewayMerchantId` (your RM merchant ID; confirm both with RM); `transactionInfo.currencyCode` `MYR`, `countryCode` `MY`, `totalPriceStatus` `FINAL`, `totalPrice` decimal string. No billing address needed (RM does not run AVS). Pass `paymentData.paymentMethodData.tokenizationData.token` as `wallet.token`. RM always returns `3DS_CHALLENGE`; redirect to `item.url` and RM captures after 3DS. Use Google `environment: 'TEST'` until production access is approved.

Direct checkout caveat: you must keep calling Query Payment Checkout (docs suggest every 3 to 5 seconds or longer) or the status will not update until RM's pre-timeout check.

## Redirect URL

`GET <redirectUrl>?status=...&orderId=...&reason=...`

| Param | Values |
| --- | --- |
| `status` | `SUCCESS`, `FAILED`, `CANCELLED`, `EXPIRED` |
| `orderId` | Your order ID |
| `reason` | Optional failure reason |

Browser UX only. Anyone can craft it. Use it to show a result page and trigger a requery.

## Notify URL (Callback)

RM `POST`s JSON to `notifyUrl`. Notify is called only when the transaction succeeds; failures and refunds do not send notify. The payload is partial; RM suggests querying the transaction for full info.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `eventType` | String | Yes | `PAYMENT_WEB_ONLINE` |
| `data.store` | Store object | No | See objects file |
| `data.referenceId` | String | No | Wallet/provider reference |
| `data.transactionId` | String | Yes | RM transaction ID |
| `data.terminalId` | String | No | |
| `data.currencyType` | String | Yes | `MYR` |
| `data.balanceAmount` | Uint64 | Yes | Remaining refundable amount |
| `data.finalAmount` | Uint64 | Yes | Amount after vouchers/membership |
| `data.platform` | String | Yes | `TERMINAL`, `MOBILE_APP`, `OPEN_API`, `WEB_LOYALTY` |
| `data.type` | String | Yes | Transaction Type, for example `WEB_PAYMENT` |
| `data.method` | String | Yes | For example `FPX` |
| `data.region` | String | Yes | For example `MALAYSIA` |
| `data.status` | String | Yes | Transaction Status |
| `data.transactionAt` | RFC3339 | No | Only when `SUCCESS` |
| `data.createdAt`, `data.updatedAt` | RFC3339 | Yes | |
| `data.payee.userId`, `data.payee.subUserId` | String | No | Provider user IDs |
| `data.order.id` | String | Yes | Your order ID |
| `data.order.title` | String | Yes | Max 32 |
| `data.order.currencyType` | String | Yes | `MYR` |
| `data.order.amount` | Uint64 | Yes | Sen |
| `data.order.detail` | String | No | Max 600 |
| `data.order.additionalData` | String | No | Max 128 |
| `data.extendInfo.duitnowInfo` | DuitNow Info | No | |
| `data.extraInfo.onlineBanking` | Online Banking | No | FPX only |

Handler pattern: store raw body, optionally verify signature (see above), requery by order ID, settle, return HTTP 200 quickly. Retry behaviour and retry count are not documented in official docs (checked 2026-10-21).

## Query Transaction

```text
GET /v3/payment/transaction/order/{orderId}
GET /v3/payment/transaction/{transactionId}
GET /v3/payment/transactions?limit=N
```

All signed GET requests (no `data=` pair). Path param `id` is the order ID or transaction ID; list takes optional query `limit`. Response: `item` (single) or `items[]` (list) of Transaction Object, plus `code` or `error.code`, `error.message`, `error.debug`. The full Transaction Object field table and nested objects (Store, Card, Online Banking, DuitNow, BNPL, vouchers, loyalty) are in `revenue-monster-objects.md`. Key fields: `transactionId`, `referenceId`, `order.*`, `currencyType`, `finalAmount`, `balanceAmount`, `status`, `method`, `type`, `platform`, `region`, `store`, `transactionAt` (only when `SUCCESS`), `error.message` (only when failed).

Transaction statuses: `SUCCESS`, `FAILED`, `IN_PROCESS`, `FULL_REFUNDED`, `PARTIAL_REFUNDED`, `REVERSED` (auto reverse on timeout). Keep polling `IN_PROCESS`.

Settle only when `status == SUCCESS`, `order.id` matches, `order.amount` (or `finalAmount` if you allow RM vouchers) equals expected sen, `currencyType == MYR`, and `store.id` is yours.

## Refund

```text
POST /v3/payment/refund
{"transactionId":"<TXN_ID>","refund":{"type":"FULL","currencyType":"MYR","amount":100},"reason":"customer request"}
```

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `transactionId` | String | Yes | RM transaction ID |
| `refund.type` | String | Yes | Documented enum `FULL` only |
| `refund.currencyType` | String | Yes | `MYR` |
| `refund.amount` | Uint64 | Yes | Sen |
| `reason` | String | Yes | |

Partial refunds: the only documented `refund.type` is `FULL`, yet the Transaction Object has `PARTIAL_REFUNDED` and `balanceAmount`. How to request a partial refund is not documented (checked 2026-10-21); confirm with RM. Response `item` is the Transaction Object.

Manual refund past the provider window: email support@revenuemonster.my, subject `[Refund Request] <Merchant name> - <Merchant ID>`, body with Transaction ID, Reference ID (if any), total transaction amount, refund amount, and reason (format recommended by the docs). Refund windows depend on the payment provider; past the window, email support@revenuemonster.my. Errors include `NOT_ALLOW_REFUND`, `PAYMENT_FULLY_REFUNDED`, `PAYMENT_REFUNDING`, `INSUFFICIENT_REFUND_AMOUNT`, `REFUND_AMT_NOT_SAME_AS_TRANSACTION`, `UNSUPPORTED_METHOD_PAYMENT_REFUND`, `PAYMENT_REFUND_AMOUNT_EXCEED_PER_DAY`. No notify is sent for refunds; requery to confirm.

## Reverse

```text
POST /v3/payment/reverse
{"orderId":"<ORDER_ID>"}
```

Request: `orderId` (String, required). Response: `item` Transaction Object, `code` or `error.*`.

For timeouts or bad connections, within a short window (docs example: 15 minutes). If a transaction timed out, reverse before creating a new one. Errors: `EXCEED_TIME_FOR_REVERSE_TRANSACTION`, `FAILED_TO_REVERSE_ORDER`, `UNSUPPORTED_METHOD_REVERSE`. Terminal card transactions cannot be reversed this way; use the terminal `REFUND` event (`revenue-monster-instore.md`).

## Reconciliation

```text
POST /v3/payment/reconciliation
{"transactionType":"PAYMENT","date":"2026-10-20","method":["FPX"],"region":["MALAYSIA"],"cursor":""}
```

Request:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `transactionType` | String | No | `PAYMENT` or `REFUND` (the docs description cell says "Terminal ID", a copy error) |
| `date` | String `YYYY-MM-DD` | Yes | Transaction date |
| `method` | []String | No | Method without suffix, for example `["BOOST"]`, `["FPX"]` |
| `region` | []String | No | For example `["MALAYSIA"]` |
| `cursor` | String | No | Pagination cursor |

Response `items[]`:

| Field | Type | Notes |
| --- | --- | --- |
| `transactionAt` | RFC3339 | |
| `merchantId`, `merchantName`, `storeId`, `storeName` | String | |
| `region`, `method` | String | |
| `transactionType` | String | `PAYMENT`, `REFUND` |
| `type` | String | Transaction Type |
| `transactionId`, `orderId` | String | |
| `currencyType` | String | `MYR` |
| `grossAmount` | String | Amounts are typed String here, unlike Uint64 elsewhere; parse before summing. |
| `mdr` | String | MDR amount |
| `serviceFee` | String | |
| `settlementAmount` | String | Net to merchant |

Plus `meta.cursor` (next page), `code` or `error.*`. Loop until `meta.cursor` is empty. Error `RECONCILIATION_NOT_FOUND` when no record. Payout schedule and bank transfer timing are not documented in the developer docs (checked 2026-10-21).

## Tokenization And Recurring

Full request/response tables are in `revenue-monster-tokenization.md`: recurring customer (`POST /v3/recurring-payment`, `recurringInterval` `MONTHLY`/`WEEKLY`/`DAILY`, target rules), tokenized customer (`POST /v3/tokenized-payment`), customer APIs including charging a bound card (`POST /v3/customer/{customer_id}/order`), `RECURRING_PAYMENT` notify, and card-on-file tokens (`GET`/`DELETE /v3/payment/tokens/{customer_id}`; call after success/notify, the token may not exist before that).

## Payment Methods

For online checkout append the region: `<METHOD>_MY` (exceptions: `APPLE_PAY`, `GOOGLE_PAY` as-is). Documented system methods: `ALIPAY_CN`, `ALIPAYPLUS_MY`, `APPLE_PAY`, `ATOME_MY`, `BOOST_MY`, `MASTERCARD_MY` (card online), `FPX_MY`, `GOOGLE_PAY`, `GRABPAY_MY`, `MAYBANK_MY`, `MCASH_MY`, `SETEL_MY`, `SHOPEEPAY_MY`, `SARAWAKPAY_MY`, `TNG_MY`, `WECHATPAY_CN`. Transaction objects report method without suffix (for example `FPX`, `GOBIZ`). Full availability tables: `revenue-monster-objects.md`.

FPX bank codes use `CODE:B2C` or `CODE:B2B`, for example `MB2U0227:B2C`, `BCBB0235:B2C`, `PBB0233:B2C`. Full B2C/B2B list in `revenue-monster-objects.md`. Use `GET /v3/payment/fpx-bank` for live availability.

## Sandbox Test Data

- Testing wallets page table: sandbox wallets exist for Alipay, ShopeePay, Maybank and RazerPay (most are beta); WeChat Pay, GrabPay and MCash are production only; GOBIZ is "Credit Card" in production and No in sandbox. The same page's note contradicts its table: it says WeChat Pay is the only wallet whose real production wallet works in BOTH sandbox and production, and recommends one round of WeChat Pay testing in sandbox. Try WeChat Pay in sandbox; if it fails, test it in production.
- Doc examples use FPX bank code `TEST0021` ("SBI Bank A") and card `4111111111111111` in sandbox requests. RM does not publish them as an official test data list; confirm in sandbox.
- Hosted demo: `https://sb-api.revenuemonster.my/demo/payment/online`.

## Common Errors

Full list (about 390 codes, plus OAuth errors): `revenue-monster-errors.md`.

| Code | Cause / fix |
| --- | --- |
| `INVALID_REQUEST_SIGNATURE` | String-to-sign mismatch: unsorted keys, whitespace, missing escapes, wrong `requestUrl`, uppercase method, wrong key pair. Read sandbox `debug`. |
| `INVALID_TIMESTAMP` | Clock skew over 120 s or non-UTC timestamp. |
| `CLIENT_NOT_SET_PUBLIC_KEY` | Upload your RSA public key to the portal application. |
| `TOKEN_EXPIRED` | Refresh access token. |
| `ORDER_ID_DUPLICATE` | Reuse of `order.id`; generate a new one per attempt. |
| `*_NOT_ACTIVE` (for example `BOOST_MALAYSIA_NOT_ACTIVE`) | Method not activated on merchant; contact RM. |
| `EXCEEDED_LIMIT` | Rate limit hit. Numeric limits are not documented in official docs (checked 2026-10-21). |

IP allowlists for notify are not documented in official docs (checked 2026-10-21).
