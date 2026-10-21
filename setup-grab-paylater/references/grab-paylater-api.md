# GrabPay & PayLater API Reference (One Time Charge, Partner API v2)

Use this reference for factual GrabPay and PayLater by Grab online checkout guidance. Checked against official Grab docs on 2026-10-21. Status, reason, and error enums are in `grab-paylater-codes.md`. Recurring (tokenised) charges are in `grab-paylater-tokenisation.md`.

## Source URLs

- `https://developer.grab.com/docs/payment-otc/api/v2` (current OTC reference, "GrabPay Integration (1.0.0)", Redoc)
- `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/` (previous OTC guide, still online; more detail on nested fields)
- `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/api-hmac`
- `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/generate-hmac`
- `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/init-payments`
- `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/generate-web-url`
- `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/handle-payment-noti`
- `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/otc-endpoint`
- `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/service-discovery-endpoint`
- `https://developer.grab.com/assets/docs/grab-pay/GrabPay_One_time_Payments_Integration_Guide.pdf` (v2.1 PDF)
- `https://github.com/grab/grabpay-merchant-sdk` (official SDK: Node, Python, Go, PHP, Java, .NET)

## Scope

- One Time Charge (OTC): the shopper logs in to Grab (mobile number + SMS OTP) on a Grab-hosted page, picks GrabPay wallet or a PayLater option, approves, and is redirected back. The merchant backend then completes (captures) the charge.
- PayLater by Grab is selected by the shopper inside this same checkout. There is no separate PayLater API for web stores. Methods returned: `GPWALLET`, `POSTPAID` (pay next month), `INSTALMENT_4`, `INSTALMENT_8`, `INSTALMENT_12`. `CARD` is Thailand only.
- Supported currencies: `SGD`, `MYR`, `PHP`, `IDR`, `THB`. For Malaysia use `MYR` and country `MY`.
- In-store QR (POS API v3, QR payments, QR interop) is out of scope.

## Endpoint Catalogue

OTC (one-time checkout):

| # | Method | Path | Auth | Purpose |
|---|---|---|---|---|
| 1 | GET | `/grabid/v1/oauth2/.well-known/openid-configuration` | none | Service discovery: returns `authorization_endpoint`, `token_endpoint`, etc. Cache; re-run if `/token` or `/authorize` returns resource not found. |
| 2 | POST | `/grabpay/partner/v2/charge/init` | Partner HMAC | Initiate payment, returns `request` JWT |
| 3 | GET | `/grabid/v1/oauth2/authorize` | browser redirect (PKCE) | Grab-hosted login and payment approval page (302) |
| 4 | POST | `/grabid/v1/oauth2/token` | client_secret in body | Exchange OAuth `code` for `access_token` |
| 5 | POST | `/grabpay/partner/v2/charge/complete` | Bearer + `X-GID-AUX-POP` | Capture the authorised charge |
| 6 | GET | `/grabpay/partner/v2/one-time-charge/{partnerTxID}/status?currency=` | Partner HMAC | OTC status, any time after init; returns `oAuthCode` when authorised |
| 7 | GET | `/grabpay/partner/v2/charge/{partnerTxID}/status?currency=` | Bearer + POP | Partner charge status, only after token redeemed |
| 8 | POST | `/grabpay/partner/v2/refund` | Bearer + POP | Full or partial refund |
| 9 | GET | `/grabpay/partner/v2/refund/{partnerTxID}/status?currency=` | Bearer + POP | Refund status (refund `partnerTxID`) |
| 10 | POST | merchant webhook URL (registered with Grab) | Grab sends Partner HMAC | Payment notification |

Other GrabID endpoints listed by discovery (staging sample): `/grabid/v1/oauth2/userinfo`, `/grabid/v1/oauth2/revoke`, `/grabid/v1/oauth2/public_keys` (JWKS), `/grabid/v1/oauth2/id_tokens/token_info`. A `/token-info` lookup for a unique customer ID from `id_token` is "enabled for your account (upon request)" (OTC FAQ).

Tokenised Payments v4 endpoints are catalogued in `grab-paylater-tokenisation.md`. Legacy Tokenised Payments v2 (`/grabpay/partner/v2/bind`, `/auth`, `/capture`, `/cancel`, `/redirect/init`, `/wallet/info`, ...) is superseded by v4; do not build new work on it.

## Environments

| Environment | Base URL |
|---|---|
| Production | `https://partner-api.grab.com` |
| Staging | `https://partner-api.stg-myteksi.com` |

The official SDK uses the same two hosts for MY (Vietnam uses `moca.vn` hosts). Current OTC docs: "an Integration Manager will contact you to collect the test URLs (Redirect URL, Webhook URL) and then you will receive the Production test API credentials and a test Grab customer account ... use our Production environment to test it." Older pages and every endpoint still list staging URLs. Ask the Integration Manager which environment your test credentials belong to and never mix them.

Partner requirements: OAuth 2.0 + HMAC support, TLS 1.2 or 1.3. Internet Explorer is not supported for the checkout page.

## Credentials

| Credential | Use |
|---|---|
| `partner_id` | Partner identifier; prefix of the HMAC `Authorization` header. One partner can have several OAuth clients and several merchants. |
| `partner_secret` | Key for the partner HMAC signature. |
| `client_id` | OAuth client (registered app) ID. |
| `client_secret` | Used in the token exchange and as the key for `X-GID-AUX-POP`. |
| `merchant_id` | Merchant account credited. One merchant account is one currency. |

Scopes: `openid` (base, required) and `payment.one_time_charge` (OTC). Recurring uses `payment.recurring_charge`.

## Common Headers

| Header | Value |
|---|---|
| `Content-Type` | `application/json` (only supported type) |
| `Date` | RFC 7231 GMT, for example `Mon, 21 Dec 2020 08:29:58 GMT`. Must equal the value used in the signature. |
| `Authorization` | Either `{partner_id}:{hmac_signature}` (partner HMAC endpoints) or `Bearer {access_token}` |
| `X-GID-AUX-POP` | Proof-of-possession signature, required with Bearer on complete, charge status, refund, refund status |

## Signature 1: Partner HMAC (Authorization header)

Used by: charge init, OTC status, and to verify Grab's webhook.

Steps (OTC v2 docs, Authentication section):

1. `body_digest = base64(sha256(request_body_bytes))` (standard base64, with padding).
2. `string_to_sign = METHOD + "\n" + Content-Type + "\n" + Date + "\n" + request_path + "\n" + body_digest + "\n"` (note the trailing newline).
3. `signature = base64(hmac_sha256(key = partner_secret, msg = string_to_sign))`.
4. Send `Authorization: {partner_id}:{signature}`, plus the same `Date` and `Content-Type`.

`request_path` is the part between the host and the query string per the older guide, for example `/grabpay/partner/v2/charge/init`.

GET requests (status endpoints): the official SDKs (Node `Utils.generateHmac` and `Requester`, Python `RestClient.generateHmac`) use an empty string as `body_digest` for GET and sign the path including the query string, for example `/grabpay/partner/v2/one-time-charge/ORDER1/status?currency=MYR`. The tokenisation v4 JavaScript sample instead hashes an empty body (`base64(sha256(""))`) and signs the path without the query. Follow the SDK for OTC status calls; if you get 401, confirm the convention with your Integration Manager.

Pseudo-code:

```js
import { createHash, createHmac } from 'crypto';
function partnerHmac({ method, path, contentType, date, body, partnerSecret }) {
  const digest = method !== 'GET' && body ? createHash('sha256').update(body, 'utf8').digest('base64') : '';
  const toSign = [method, contentType, date, path, digest, ''].join('\n');
  return createHmac('sha256', partnerSecret).update(toSign, 'utf8').digest('base64');
}
const body = JSON.stringify(payload);            // sign and send these exact bytes
const date = new Date().toUTCString();           // "Tue, 21 Oct 2026 08:00:00 GMT"
headers = { 'Content-Type': 'application/json', Date: date,
  Authorization: `${PARTNER_ID}:${partnerHmac({ method: 'POST', path: '/grabpay/partner/v2/charge/init', contentType: 'application/json', date, body, partnerSecret: PARTNER_SECRET })}` };
```

Docs test vector inputs (JS sample): partner ID `569c95f1-f5e4-4bd9-8dff-541d8f2af81e`, secret `NvPwMZqiUkYyfDwG`, date `Mon, 12 Nov 2018 08:20:55 GMT`, path `/grabpay/partner/v2/charge/init`, body `{"amount":100,"currency":"SGD","description":"paysi-agent-partner-v2-otc","merchantID":"44e46f25-7787-44ce-b785-cf9f9b95601e","partnerGroupTxID":"ORDER123","partnerTxID":"ORDER123"}`. Expected signature: `pSK8CLf03MJzmSPmeDyqsrSIuwyy1QvExmxmf3sme4w=` (the value in the docs header example; recomputed 2026-10-21 with the algorithm above and this exact body, so it is a usable unit-test vector). The same fields in a different key order give a different signature, which is why exact bytes matter. The docs page also offers a "Verifying HMAC signature" form.

401 checklist (OTC FAQ): server clock within 5 minutes, Date in GMT, Content-Type `application/json`, newline-joined string to sign.

## Signature 2: X-GID-AUX-POP (after token exchange)

Used with `Authorization: Bearer {access_token}` on charge complete, partner charge status, refund, refund status. Generate a fresh one per request.

```js
function popSignature(accessToken, clientSecret) {
  const ts = Math.round(Date.now() / 1000);                                   // unix seconds
  const sig = createHmac('sha256', clientSecret).update(String(ts) + accessToken, 'utf8').digest('base64');
  const b64url = s => s.replace(/=/g, '').replace(/\+/g, '-').replace(/\//g, '_');
  const payload = JSON.stringify({ time_since_epoch: ts, sig: b64url(sig) });
  return b64url(Buffer.from(payload, 'utf8').toString('base64'));
}
```

Sample output shape: `eyJ0aW1lX3NpbmNlX2Vwb2NoIjoxNTUxNzc5OTk1LCJzaWciOiJfajh3dm1IT0NXRFdJSGdIVE8yS0tSSkpqVHZRdW9WS2RnWGo4TE1kWm13In0`. The JSON key order differs between docs (`time_since_epoch` first) and Node SDK (`sig` first); both decode to the same object.

## Checkout Flow (OTC)

1. (Optional) Service discovery, cache `authorization_endpoint` and `token_endpoint`.
2. `POST /grabpay/partner/v2/charge/init` with unique `partnerTxID`. Response gives `request`.
3. Build the Web URL (authorize) with PKCE and redirect the shopper.
4. Shopper logs in and taps Pay; funds are earmarked; Grab redirects to `redirect_uri?code=...&state=...` (or `?error=...&state=...`). Grab also sends an `Auth` webhook.
5. If the redirect was lost: on the `Auth` webhook, call OTC status to get `oAuthCode`. If `oAuthCode` is empty while authorised, retry after 5 seconds; if still empty, mark failed (Grab refunds the hold).
6. `POST /grabid/v1/oauth2/token` with the code. Store `access_token` securely; it is the transaction identifier for later status and refund calls.
7. `POST /grabpay/partner/v2/charge/complete`. Grab moves money and returns `txID`, `paymentMethod`, `txStatus`, `reason`.
8. If `txStatus` is `processing`, wait for webhook or poll status no more than once every 2 minutes.

### 2. Initiate Payments: `POST /grabpay/partner/v2/charge/init`

Headers: `Authorization: {partner_id}:{hmac}`, `Date`, `Content-Type: application/json`.

| Field | Type | Req | Notes |
|---|---|---|---|
| `partnerGroupTxID` | string <=32, `^[a-zA-Z0-9\-_]+$` | Yes | Order or receipt ID; one order may have several charges; can be shown to user |
| `partnerTxID` | string <=32, same pattern | Yes | Unique per transaction; idempotency key |
| `amount` | int64 | Yes | Smallest unit, positive (MYR 5.34 = 534) |
| `currency` | string enum | Yes | `SGD` `MYR` `PHP` `IDR` `THB`; must match the merchant account currency |
| `description` | string <=255 | No | Charge description |
| `merchantID` | string | Yes | From Grab |
| `hidePaymentMethods` | string[] | No | `INSTALMENT`, `POSTPAID`, `CARD` (TH only). Wallet cannot be hidden |
| `metaInfo` | object | Cond. | Required for PayLater instalment transactions, otherwise optional |
| `items` | object[] | No (recommended) | Helps PayLater approval; max 255 chars per field |
| `shippingDetails` | object | No (recommended) | Helps PayLater approval; max 255 chars per field |

`metaInfo` fields: `brandName` (merchant name shown in checkout), `location.ipAddress`, `location.latitude`, `location.longitude`, `location.accuracy` (metres), `device.deviceID`, `device.deviceModel`, `device.deviceBrand`, `device.iosUDID`, `device.imei`, `subMerchant.name`, `subMerchant.category`, `partnerUserInfo.id`, `partnerUserInfo.phone`, `partnerUserInfo.email`, `echo` (merchant reference returned in the webhook).

`items[]` fields: `supplierName`, `itemName`, `category`, `quantity` (int), `price` (int, per item, smallest unit; should reconcile to total), `itemCategory` (enum string, see codes file), `url`, `imageURL`, `supplierDetails` {`supplierID`, `supplierName`, `supplierURL`, `legalID`, `supplierPhone`, `supplierEmail`, `supplierAddress`, `supplierRating` (double)}.

`shippingDetails` fields: `firstName`, `lastName`, `address`, `city`, `postalCode`, `phone`, `email`, `countryCode`.

Response 200: `{ "partnerTxID": "...", "request": "<unsigned JWT>" }`. Errors 400, 401, 403, 409 (reasons in codes file), 429, 5xx. Error body sample: `{"code": 4090, "reason": "invalid_merchant"}`.

### 3. Generate Web URL: `GET {authorization_endpoint}` = `/grabid/v1/oauth2/authorize`

| Query param | Req | Value |
|---|---|---|
| `acr_values` | Yes | `consent_ctx:countryCode=MY,currency=MYR` |
| `client_id` | Yes | Your client ID |
| `code_challenge` | Yes | `BASE64URL(SHA256(ASCII(code_verifier)))`, no padding; `code_verifier` is 43 to 128 chars from `[A-Za-z0-9-._~]` |
| `code_challenge_method` | Yes | `S256` only (`plain` not allowed) |
| `nonce` | Yes | Random string; compare with `nonce` claim in `id_token` |
| `redirect_uri` | Yes | Must be registered with Grab for this client |
| `request` | Yes | `request` from charge init |
| `response_type` | Yes | `code` |
| `scope` | Yes | Docs table says `payment.one_time_charge`; docs sample URL and official SDK send `openid payment.one_time_charge`. Use the SDK value. |
| `state` | Yes | Random CSRF token; must match on return |

Return: `{redirect_uri}?code=...&state=...` on approval; `?error=<code>&state=...` on failure (codes file). Always check `state` first.

Sample: `https://partner-api.stg-myteksi.com/grabid/v1/oauth2/authorize?acr_values=consent_ctx%3AcountryCode%3DSG,currency%3DSGD&client_id=...&code_challenge=...&code_challenge_method=S256&nonce=...&redirect_uri=...&request=...&response_type=code&scope=openid%20payment.one_time_charge&state=...`

### 4. Get OAuth Token: `POST /grabid/v1/oauth2/token`

Header `Content-Type: application/json`. Body: `code`, `client_id`, `grant_type` (`authorization_code`), `redirect_uri` (same as step 3), `code_verifier`, `client_secret`. All required.

Response: `access_token`, `token_type` (`Bearer`), `expires_in` (sample 31535999 seconds), `id_token` (RS256 JWT; check `nonce`). Server-side only.

### 5. Complete Payments: `POST /grabpay/partner/v2/charge/complete`

Headers: `Authorization: Bearer {access_token}`, `X-GID-AUX-POP`, `Date`, `Content-Type`. Body: `{ "partnerTxID": "<same as init>" }`.

Response fields: `txID` (32-char hex GrabPay ID, needed for refunds), `status` (deprecated), `paymentMethod`, `description` (error text on failure, empty on success), `txStatus`, `reason`. HTTP 404 "could not find any client". Calling complete again with a used `partnerTxID` returns the latest status (newer merchants).

### 6. OTC Status: `GET /grabpay/partner/v2/one-time-charge/{partnerTxID}/status?currency=MYR`

Partner HMAC auth. Callable any time after init. Response: `txID`, `oAuthCode`, `paymentMethod`, `status` (deprecated), `txStatus`, `reason`. `oAuthCode` is returned only when the user consented, at least one redirect to `redirect_uri` was attempted, and the charge awaits complete (`txStatus` authorised). Wrong `partnerTxID` or currency returns 404.

### 7. Partner Charge Status: `GET /grabpay/partner/v2/charge/{partnerTxID}/status?currency=MYR`

Bearer + POP, only after token exchange. Response: `txID`, `status` (deprecated), `paymentMethod`, `description`, `txStatus`, `reason`. 401 if the token is not tied to that `partnerTxID`.

## Webhook: Payment Notification

Grab POSTs to the webhook URL you gave the Integration Manager (registered via the Developer Portal). Grab sends a notification for every transaction status change; several per transaction with different `txType`.

Headers: `Authorization: {partner_id}:{hmac_signature}`, `Date`, `Content-Type: application/json`. Grab: "when you compute HMAC, use the value `application/json; charset=utf-8` as Content-Type."

Verification as documented: recompute the partner HMAC the same way as for charge init (method `POST`, Content-Type `application/json; charset=utf-8`, the received `Date`, path, SHA-256 of the raw body) with your `partner_secret` and compare in constant time. The docs do not state which path to sign (your webhook path is the natural reading). Not documented in official docs (checked 2026-10-21): webhook source IPs, retry schedule for OTC. Because of these gaps, after a valid signature also confirm via OTC or charge status before fulfilment.

| Field | Type | Notes |
|---|---|---|
| `txType` | string | `Init`, `Auth`, `Charge`, `Capture`, `Refund` (table lists Capture and Refund meanings; may grow) |
| `txStatus` | string | See codes file |
| `partnerID` | string <=32 | Your partner ID |
| `partnerTxID` | string <=32 | Your transaction ID (refund events carry the refund `partnerTxID`) |
| `txID` | string 32 hex | GrabPay transaction ID |
| `origTxID` | string | Original charge `txID` for refunds, else empty |
| `amount` | int64 | Smallest unit |
| `currency` | string | e.g. `MYR` |
| `status` | string | Deprecated, use `txStatus` |
| `createdAt`, `completedAt` | int64 | Unix seconds |
| `payload.partnerGroupTxID` | string | From init |
| `payload.reason` | string | Reason code, may be empty on success |
| `payload.paymentMethod` | string | `GPWALLET`, `POSTPAID`, `INSTALMENT_4`, `INSTALMENT_8`, `INSTALMENT_12`, `CARD` (TH) |
| `payload.rewardsMeta` | object | `offerCode`, `offerName`, `offerDiscount` (int64); only for merchant-funded promos |
| `payload.echo` | string | Your `echo` |
| `payload.newStatus` | string | Deprecated |

Respond HTTP 200 quickly and process idempotently keyed on `partnerTxID` + `txType` + `txStatus`.

## Refunds

### `POST /grabpay/partner/v2/refund`

Headers: `Authorization: Bearer {access_token of the original charge}`, `X-GID-AUX-POP`, `Date`, `Content-Type`. Prerequisite: the charge was completed.

| Field | Type | Req | Notes |
|---|---|---|---|
| `partnerGroupTxID` | string <=32 | Yes | Order ID |
| `partnerTxID` | string <=32 | Yes | New unique ID for this refund (SDK names it `refundPartnerTxID`) |
| `amount` | int64 | Yes | Refund amount; partial allowed unless `partial_refund_not_allowed` |
| `currency` | string | Yes | Same as charge |
| `description` | string <=255 | No | |
| `merchantID` | string | Yes | |
| `originTxID` | string 32 hex | Yes in older guide, optional in v2 schema; always send | `txID` from complete |
| `echo` | string <=64 | No | Returned in webhook |

Response: `txID`, `status` (deprecated), `paymentMethod`, `description`, `txStatus`, `reason`. 409 reasons in codes file. Refund window Malaysia, Singapore, Philippines: 365 days after payment; Thailand 180. Refunds go back to the original funding source. Refunds are asynchronous; before processing, status shows `txStatus: processing`, `reason: refunding`.

### `GET /grabpay/partner/v2/refund/{refundPartnerTxID}/status?currency=MYR`

Bearer + POP. Response: `txID`, `status` (deprecated), `paymentMethod`, `description`, `txStatus`, `reason`, `echo`.

## Amount Rules

Positive integer in the smallest currency unit (ISO 4217). MYR has 2 decimals: MYR 10 = `1000`. VND has 0 decimals.

## Timeouts and Lifetimes (OTC FAQ)

- Request code (init to user confirmation): 20 minutes.
- OAuth code (user confirmation to complete): 15 minutes.
- OTC access token: 1 year.
- Charge not completed within 15 minutes of init: Grab cancels and auto-refunds; you get a cancellation notification.
- `auth_expired` guidance: call complete within 10 minutes of the auth webhook or redirect.

## Rate Limits and Retries

- Max 50 API calls per second per Partner ID; on 429 use exponential backoff with jitter.
- 5xx: retry later with backoff (docs say retry init with a new `partnerTxID`).
- Status polling: not more than once every 2 minutes.

## Mobile and Deep Links

Web redirect flow is the default. Deep link / desktop QR flows apply only to approved merchant partners. In-app webviews must intercept `grab://` and `grabtaxi://` URLs and open them with the OS (Android `shouldOverrideUrlLoading`, iOS `decidePolicyFor navigationAction`); native browsers handle deep links already.

## SDK

Official SDK `https://github.com/grab/grabpay-merchant-sdk` (last push 2023-09-07) wraps: `chargeInit`, `generateWebUrl`, `oauth2Token`, `chargeComplete`, `getChargeStatus`, `refund`, `getRefundStatus`, `getOtcStatus`. Config: `env` (`STG`/`PRD`), `country` (`MY`), `partnerId`, `partnerSecret`, `merchantId`, `clientId`, `clientSecret`, `redirectUrl`. Server-side only. The SDK adds headers `X-Sdk-Country`, `X-Sdk-Language`, `X-Sdk-Signature`, `X-Sdk-Version`. Grab's older prerequisites page said "integrate directly with GrabPay endpoints without the use of an SDK"; the current v2 docs offer the SDK, so either is acceptable.

## Testing

- No public sandbox signup and no published test wallets or cards. Grab issues test credentials and a test Grab customer account (phone number; top up manually) during onboarding.
- Promotion mechanics available for testing (FAQ): click to claim/return vouchers, % discount, % discount with minimum spend, absolute discount, absolute discount with minimum spend, accelerated points, point booster, point booster with minimum spend.
- To test another country's wallet, the FAQ suggests a GPS spoofing app.
- Postman collection: `https://www.getpostman.com/collections/fd99c391287fb18868e3`.

## Not Documented (checked 2026-10-21)

- Webhook source IP allowlist and OTC webhook retry schedule (tokenisation v4 says 2 retries if no HTTP 200).
- Exact path used in the webhook HMAC.
- Cancel/void endpoint for OTC (expiry auto-cancels; use refund after capture).
- Settlement or payout API (reports are SFTP or portal only).
