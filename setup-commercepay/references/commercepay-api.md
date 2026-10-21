# CommercePay API Reference

Use this reference for factual CommercePay API guidance. Distilled from the full official docs at `https://docs.commercepay.asia` (all 32 pages plus `openapi.json`, checked 2026-10-21). Channel and bank code tables live in `commercepay-codes.md`.

## Source URLs

- `https://docs.commercepay.asia/llms.txt` (index), `https://docs.commercepay.asia/llms-full.txt`, `https://docs.commercepay.asia/openapi.json`
- `https://docs.commercepay.asia/service-environments`
- `https://docs.commercepay.asia/api-credentials`
- `https://docs.commercepay.asia/request-headers`
- `https://docs.commercepay.asia/generate-signature`
- `https://docs.commercepay.asia/callback-notifications`
- `https://docs.commercepay.asia/direct-integration`
- `https://docs.commercepay.asia/hosted-session-checkout`
- `https://docs.commercepay.asia/recurring-payments`
- `https://docs.commercepay.asia/refund-integration`
- `https://docs.commercepay.asia/payout-integration`
- `https://docs.commercepay.asia/api-reference/*` (one page per endpoint, see source-map.md)
- Official WooCommerce plugin v1.4 source: `https://capstoragedata.blob.core.windows.net/landingpage/plugins/woocommerce/cap-gateway.zip` (linked from `https://www.commercepay.asia/developer`)

## Integration Surfaces

- **Direct Integration**: merchant fetches channels (`GetTenantChannelsByCountry`), optionally bank sub-channels (`GetProviderChannels`), renders them, then calls `RequestPayment` with `channelId` (+ `providerChannelId`). Customer goes straight to the provider (bank, wallet, card page).
- **Hosted Session Checkout**: merchant calls `InitialSession`; customer picks a channel on a CommercePay page. The session stays open until success or expiry, allowing retries and channel switches; the same session URL can be reopened.
- **Hosted Payment Web via RequestPayment**: omitting `channelId` on `RequestPayment` also routes the customer to the CommercePay hosted page.
- **Recurring Payments**: Direct Integration only, Credit Card and TNG eWallet only, needs approval.
- **Saved payments (tokenisation)**: `AddSavePayment`, `CheckSavedPayment` (TNG only), `RevokeTokenize`, `savePayment` flag.
- **Pre-auth / capture**: `ProcessPayment` with `type` PreAuth, FullPayment, PartialPayment.
- **Payouts**: `RequestPayment` with the payout channel, bank account or DuitNow Proxy, funded from a payout wallet.
- **No-code**: payment links, payment forms, recurring payment forms in the back office; plugins for WooCommerce, OpenCart, NopCommerce, WHMCS, SiteGiant (see account-setup.md).

## Environments

| Environment | API base URL |
| --- | --- |
| Staging | `https://staging-payments.commerce.asia/` |
| Production | `https://payments.commerce.asia/` |

The OpenAPI spec lists only the staging server. Credentials, Secret Key and channel IDs are environment-specific.

## Credentials

Issued by CommercePay Business Development at onboarding:

| Credential | Used for |
| --- | --- |
| Merchant Id | `Abp-TenantId` header (and `tenantId` body fields where present) |
| Username | `userNameOrEmailAddress` in Authenticate |
| Password | `password` in Authenticate |
| Secret Key | HMAC key for `cap-signature` on requests and callbacks |

Never put Secret Key or Password in client-side code.

## Request Headers

Every API call except Authenticate:

| Header | Value |
| --- | --- |
| `Content-Type` | `application/json` |
| `Authorization` | `Bearer <accessToken>` |
| `Abp-TenantId` | `<MERCHANT_ID>` |
| `cap-signature` | HMAC-SHA256 hex, see below |

## Response Envelope

Responses use the ABP framework envelope. Read data from `result`.

```json
{ "result": { ... }, "targetUrl": null, "success": true, "error": null, "unAuthorizedRequest": false, "__abp": true }
```

Error form: `"success": false, "error": { "code": 1, "message": "Invalid Signature." }`. Some documented payout/wallet samples show the invalid-signature object inside `result` with `error: null`; check both `success`/`error` and `result.code` defensively.

## Authentication

```text
POST {base}/api/TokenAuth/Authenticate
Header: Abp-TenantId: <MERCHANT_ID>
No cap-signature.
```

| Request field | Type | Required | Notes |
| --- | --- | --- | --- |
| `userNameOrEmailAddress` | string, max 50 | yes | API username |
| `password` | string, max 50 | yes | API password |

| Response field | Type | Notes |
| --- | --- | --- |
| `accessToken` | string | Use as `Authorization: Bearer <accessToken>` |
| `expireInSeconds` | int32 | `86400` (24 hours) |

`401` = invalid username or password. Cache and reuse the token; call again when it expires or calls start failing auth.

## Signature (`cap-signature`)

HMAC-SHA256, hex output, key = Secret Key, over a lowercased string.

### POST requests

1. Build the payload as JSON with camelCase keys. Remove every field that is `null` or not provided, at every level (nested objects too). Keep empty strings `""`.
2. Sort keys ascending A-Z, recursively (nested objects such as `customer` too).
3. Serialize minified (no spaces/newlines). String to sign = `FULL_URL + SORTED_JSON` with no separator. FULL_URL is the absolute URL including `https://`, exactly as sent.
4. Lowercase the whole string (URL, keys and values) with a Unicode-aware lowercase.
5. HMAC-SHA256 with the Secret Key; both string and key as UTF-8 bytes.
6. Hex-encode into the `cap-signature` header.

Encoding rules:

- Do not escape non-ASCII: sign `陈大文 Café`, not `陈...`. Python: `ensure_ascii=False`. .NET `System.Text.Json`: use an unescaped encoder.
- Escape only what JSON requires (`"`, `\`, control chars). `+`, `<`, `>`, `&`, `'` stay literal (System.Text.Json escapes `+` as `+` by default; disable that).
- Body key casing is matched case-insensitively by the server, but always sign the camelCase form. Send camelCase too.

### GET requests

1. Turn the query params into a JSON object with camelCase keys (`TransactionNumber` becomes `transactionNumber`), omit nulls, numbers unquoted.
2. Sort keys recursively.
3. String to sign = `FULL_URL_WITHOUT_QUERY_STRING + SORTED_JSON` (no `?`, no params in the URL part).
4. Lowercase, HMAC-SHA256, hex, as above.

### Test vectors (docs sample Secret Key, staging; verified locally 2026-10-21)

Secret Key (published documentation example only): `bd90787d8e2340b49757438d66eed86d209e6a36a32d865a8287f90b033b815d`

| Case | String to sign (before lowercase) | Expected `cap-signature` |
| --- | --- | --- |
| POST | `https://staging-payments.commerce.asia/api/services/app/PaymentGateway/RequestPayment{"amount":100,"currencyCode":"MYR","customer":{"email":"test.user@example.com","name":"Test User"},"referenceCode":"TEST-111"}` | `477d7efe6d3c319a722bd202d3434dbfde3fe19877febb0e5c62ef85e7d30efd` |
| POST non-ASCII (`name` = `陈大文 Café`) | same URL + `{"amount":100,"currencyCode":"MYR","customer":{"email":"test.user@example.com","name":"陈大文 Café"},"referenceCode":"TEST-111"}` | `d6bb6d6b0e502ff08161655323df67deefddddb92088d2e144194ebfd18ead28` |
| GET Query | `https://staging-payments.commerce.asia/api/services/app/PaymentGateway/Query{"timestamp":1621851652617,"transactionNumber":"20003D2625A16279BAB260115"}` | `a4f50b21c6e29ea7d93357906637a90a5ccf01016a61c52c218686b7ab0d2798` |

If the non-ASCII name is escaped, the result is `3ae10f25...` and the server rejects it. The callback example on the Callback Notifications page (`7e52da8c...`) does not reproduce with this sample key; the docs do not state which key produced it, so do not use it as a test vector.

### Pseudo-code

```js
import crypto from "node:crypto";

function canon(v) {
  if (Array.isArray(v)) return v.map(canon);
  if (v && typeof v === "object") {
    const out = {};
    for (const k of Object.keys(v).sort()) {
      if (v[k] === null || v[k] === undefined) continue;
      out[k] = canon(v[k]);
    }
    return out;
  }
  return v;
}

function capSignature(urlNoQuery, payload, secretKey) {
  const s = (urlNoQuery + JSON.stringify(canon(payload))).toLowerCase();
  return crypto.createHmac("sha256", Buffer.from(secretKey, "utf8"))
    .update(Buffer.from(s, "utf8")).digest("hex");
}
// JSON.stringify keeps non-ASCII and + < > & ' literal, which is what CommercePay wants.
// Send the same canon(payload) as the POST body so what you send equals what you sign.
```

```python
import hmac, hashlib, json
def canon(v):
    if isinstance(v, dict):
        return {k: canon(v[k]) for k in sorted(v) if v[k] is not None}
    if isinstance(v, list):
        return [canon(x) for x in v]
    return v
def cap_signature(url_no_query, payload, secret_key):
    s = (url_no_query + json.dumps(canon(payload), separators=(",", ":"), ensure_ascii=False)).lower()
    return hmac.new(secret_key.encode("utf-8"), s.encode("utf-8"), hashlib.sha256).hexdigest()
```

Sort order: docs say "ascending (A-Z)". For keys that differ in case only at the first letter the docs do not define ordinal vs case-insensitive order; always use camelCase keys and the issue does not arise.

Debug tool: Signature Validator `https://staging-backoffice-payments.commerce.asia/app/paymentgateway/signaturevalidator`. Mismatch causes listed by docs: non-minified JSON, relative URL, unsorted nested keys, missing lowercase step, wrong environment key.

## Endpoint Catalogue

All paths are relative to the environment base URL. All except Authenticate need Bearer + `Abp-TenantId` + `cap-signature`.

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/api/TokenAuth/Authenticate` | Get access token (no signature) |
| GET | `/api/services/app/Channel/GetTenantChannelsByCountry` | List channels available to the merchant for a country |
| GET | `/api/services/app/Channel/GetProviderChannels` | List sub-channels (banks) for a channel with `isProviderHostChannel: true` |
| POST | `/api/services/app/PaymentGateway/RequestPayment` | Create payment (Direct Integration, hosted web without `channelId`, recurring charges, payouts) |
| POST | `/api/services/app/PaymentGateway/InitialSession` | Create Hosted Session Checkout |
| GET | `/api/services/app/PaymentGateway/Query` | Query payment or session status |
| POST | `/api/services/app/PaymentGateway/ProcessPayment` | Pre-auth, full or partial capture on an existing transaction |
| POST | `/api/services/app/PaymentGateway/AddSavePayment` | Pre-save a customer payment method (MYR 1.00 verification charge, auto refunded) |
| POST | `/api/services/app/PaymentGateway/CheckSavedPayment` | Check whether a customer has a saved method (TNG only) |
| POST | `/api/services/app/PaymentGateway/RevokeTokenize` | Revoke a saved payment method |
| POST | `/api/services/app/PaymentGateway/RefundPayment` | Full or partial refund |
| GET | `/api/services/app/PaymentGateway/QueryRefund` | Refund status |
| POST | `/api/services/app/PaymentGateway/QueryWallets` | Wallet balances (payout wallet) |
| POST | `/api/services/app/PaymentGateway/QueryWalletHistory` | Wallet transaction history (documented on its page, not in openapi.json) |
| POST (to you) | your `callbackUrl` | Signed host-to-host status callback |

## Get Channel List

`GET /api/services/app/Channel/GetTenantChannelsByCountry?countryCode=MY`

| Query param | Type | Required | Notes |
| --- | --- | --- | --- |
| `countryCode` | string, 2 | yes | ISO 3166 alpha-2, e.g. `MY` |

Signature JSON: `{"countryCode":"MY"}` (camelCase).

Response: array of `ChannelDto`.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | int32 | The `channelId` |
| `name` | string | Channel name |
| `displayName` | string | Merchant-facing name |
| `type` | int | ChannelType: `1` OnlinePayment, `2` OnlinePayout, `3` OnlineRecurringPayment |
| `status` | int | ChannelStatus: `0` Inactive (maintenance or onboarding pending), `1` Active |
| `refundType` | int | `0` UnSupportedRefund, `1` FullRefund, `2` FullRefundOrPartialRefund |
| `refundChargesType` | int | `0` PostRefundCharges (charges applied after/on top of refund), `1` AlwaysRefundCharges (charges refunded to customer) |
| `isProviderHostChannel` | bool | `true`: call GetProviderChannels and send `providerChannelId`. `false`: omit `providerChannelId` |
| `currencies[]` | array | `acceptedCurrencyCode`, `minRequestAmount`, `maxRequestAmount`, `dailyTransactionCount`, `dailyTransactionAmount`, `minSurchargeAmount`, `maxSurchargeAmount`, `isSurchargePercentage`, `surcharge` (all nullable) |
| `imageUrl` | string | Channel logo |
| `groupId`, `groupName`, `groupImageUrl` | | Channel grouping (used by hosted page filters) |
| `savePayment` | bool | Channel supports saving payment info |

## Get Provider Channels

`GET /api/services/app/Channel/GetProviderChannels?ChannelId=2&CurrencyCode=MYR&Amount=10000`

| Query param | Type | Required | Notes |
| --- | --- | --- | --- |
| `ChannelId` | int32 | yes | From Get Channel List |
| `CurrencyCode` | string, 3 | no | Filter by currency |
| `Amount` | int64 | no | Minor units; filter channels supporting the amount |

Response: array of `{ id, name, displayName, imageUrl, status, whitelist, options }`. `id` is the `providerChannelId`. `status`: `0` Inactive, `1` Active, `2` Offline (provider down), `-1` Unknown. `whitelist`/`options` reserved, always null.

Errors (HTTP 500): `2001` InvalidChannel, `2017` ProviderDoesNotHaveChannels, `2018` FailedToGetProviderChannels.

## Request Payment (Direct Integration)

`POST /api/services/app/PaymentGateway/RequestPayment`

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `channelId` | int32 | conditional | From Get Channel List. Omit to send the customer to the CommercePay hosted payment page |
| `providerChannelId` | string | conditional | Mandatory when channel `isProviderHostChannel` is true (bank code, e.g. `TEST0021`); omit otherwise |
| `currencyCode` | string, 3 | yes | `MYR` |
| `amount` | int64 | yes | Minor units, `1000` = 10.00 |
| `referenceCode` | string, 50 | yes | Your unique order/invoice reference |
| `description` | string, 100 | yes | English letters, digits, spaces, `. , - _ ( ) @ /`. Providers may strip or truncate (FPX keeps first 30 chars); payment is not rejected |
| `ipAddress` | string, 50 | yes | Customer IP captured by your server |
| `userAgent` | string, 200 | conditional | Recommended; affects channels such as ShopeePay (desktop vs mobile vs app) |
| `returnUrl` | string, 500 | yes | Browser return after completion |
| `callbackUrl` | string, 500 | no | Host-to-host callback target. Omit = no callback |
| `timestamp` | int64 | yes | Epoch milliseconds |
| `savePayment` | bool | conditional | Tokenise for subsequent payments |
| `customer` | object | no | `email` (150), `mobileNo` (16), `name` (36), `username` (36, your unique customer id for recurring/saved payments), `identificationNumber` (FPX recurring), `identificationType` (`1` NewIdentityNumber, `2` OldIdentityNumber, `3` PassportNumber, `4` BusinessRegistration, `5` Others; FPX recurring) |
| `subMerchant` | object | no | `uniqueId`, `name` |
| `localCitizen` | bool | conditional | Certain channels: `true` if customer is Malaysian |
| `recurringPaymentInfo` | object | conditional | See Recurring |
| `requestPaymentExtraInfo` | object | no | Hosted page filters: `channelGroupId` (int), `whitelistChannelGroup` (int[]), `whitelistChannel` (int[]), `hiddenChannel` (int[], deprecated) |
| `platformCharge` | object | no | Authorized partners only: `partnerId`, `amount` (int64) |
| `channelDetails` | object | no | Card data for direct card/pre-auth: `cardNumber`, `expiryMonth`, `expiryYear`, `holderName`, `securityCode` (3), `authType` (`1` PreAuth) |
| `browserInfo` | object | no | `javaEnabled`, `colorDepth`, `screenHeight`, `screenWidth`, `timeZoneOffset` (e.g. `+08:00`) |
| `virtualAccountCode` | string, 30 | no | Virtual account code |
| `loyaltyPaymentInfoDto` | object | conditional | Loyalty payment: `cardNumber`, `cardPassword`, `redemptionPoint` (int64) |

The OpenAPI schema spells some property names in PascalCase (`ChannelDetails`, `BrowserInfo`, `VirtualAccountCode`, `ChannelGroupId`). The server matches names case-insensitively; sign and send the camelCase form. Sending raw card data (`channelDetails`) brings PCI DSS scope; not documented further.

Sample:

```json
{
  "channelId": 2,
  "providerChannelId": "TEST0021",
  "currencyCode": "MYR",
  "amount": 1000,
  "referenceCode": "INV-2026-00941",
  "ipAddress": "172.56.21.89",
  "returnUrl": "https://merchant.example/checkout/result",
  "callbackUrl": "https://merchant.example/api/commercepay/callback",
  "description": "Order checkout payment",
  "timestamp": 1774845600000
}
```

Response `result`:

| Field | Type | Notes |
| --- | --- | --- |
| `currencyCode` | string, 3 | |
| `amount` | int64 | Minor units |
| `transactionNumber` | string, 24 | CommercePay transaction number; store it |
| `channelId` | int32 | |
| `redirectionType` | int | `1` UrlRedirection (send browser to `redirectUrl`), `2` RenderScriptRedirection (render `clientScript`) |
| `redirectUrl` | string | Provider/hosted page URL |
| `clientScript` | string | Script for type 2 |
| `transactionExpired` | date-time | ISO 8601 expiry |

Errors: `1` Invalid Signature (HTTP 400, `unAuthorizedRequest: true`), "Invalid Timestamp." (code not documented).

## Hosted Session Checkout

`POST /api/services/app/PaymentGateway/InitialSession` (no `channelId`)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `currencyCode` | string, 3 | yes | |
| `amount` | int64 | yes | Minor units |
| `referenceCode` | string, 50 | yes | |
| `description` | string, 100 | yes | Same character rules as RequestPayment |
| `ipAddress` | string, 50 | yes | |
| `userAgent` | string, 200 | conditional | |
| `returnUrl` | string, 500 | yes | |
| `callbackUrl` | string, 500 | no | |
| `customer` | object | no | Same fields as RequestPayment `customer` |
| `subMerchant` | object | no | `uniqueId`, `name` |
| `platformCharge` | object | no | `partnerId`, `amount` |
| `virtualAccountCode` | string, 30 | no | |
| `requestPaymentExtraInfo` | object | no | Channel group/whitelist filters for the hosted page |
| `expiredInMinutes` | int | no | Session lifetime in minutes |
| `tenantId` | int | no | Merchant Id |
| `timestamp` | int64 | yes | Epoch ms |

Response `result`: `redirectionType`, `redirectUrl` (session URL), `sessionNumber`. Store `sessionNumber`. The session can hold several attempts; the callback includes `paymentSessionNumber`. If the session page does not return success, call Query Payment.

## Callback Notifications

CommercePay POSTs JSON to the `callbackUrl` given on RequestPayment, InitialSession, AddSavePayment, or payout requests, after it confirms the provider result. Applies to payments, the AddSavePayment MYR 1.00 verification and its refund, and payouts.

Payload (fields may vary slightly by channel/provider):

| Field | Type | Notes |
| --- | --- | --- |
| `amount` | int64 | Minor units |
| `channelId` | int | |
| `currencyCode` | string | |
| `referenceCode` | string | Your reference |
| `status` | int | Payment status code |
| `transactionNumber` | string | CommercePay transaction number |
| `providerTransactionNumber` | string | Upstream reference, when available |
| `paymentSessionNumber` | string | Session flows only |

### Verify the callback

1. Take the received payload with camelCase keys.
2. Sort keys A-Z recursively. Include `paymentSessionNumber` only if present.
3. String = YOUR original `callbackUrl` (exactly as you sent it, including any query string, not the path the request arrived on) + minified sorted JSON.
4. Lowercase the whole string.
5. HMAC-SHA256 with the Secret Key of the environment that sent it; hex.
6. Constant-time compare with the `cap-signature` request header.

Official WooCommerce plugin v1.4 computes `hash_hmac('sha256', strtolower(callbackUrl . rawBody), secretKey)` over the raw body, which implies the body arrives already sorted and minified. Safest: verify with the docs canonical method; if your framework already parsed the body, rebuild the canonical JSON from it (amounts are integers so re-serialization is stable).

Acknowledge with HTTP 200 (sequence diagram). Retry schedule, timeout and source IPs: not documented in official docs (checked 2026-10-21). Always requery `GET Query` before fulfilment and make the handler idempotent on `transactionNumber`.

### Payment Status Codes

| Value | Name | Meaning |
| --- | --- | --- |
| 0 | Pending | Pending |
| 1 | Success | Completed successfully |
| 2 | Failed | Failed |
| 3 | Cancelled | Cancelled |
| 4 | TransactionLimitExceeded | Over allowed limit |
| 5 | MinTransactionAmountNotMeet | Below minimum amount |
| 6 | InsufficientFunds | Insufficient funds |
| 7 | InvalidTransaction | Invalid |
| 8 | UserCancelled | Customer abandoned/cancelled at provider |
| 9 | Authorized | Pre-auth held, not captured |
| 10 | TransactionExpired | Expired before completion |
| 11 | Refunded | Refunded |
| 12 | ProcessingRefund | Refund processing |
| 13 | FailedRefund | Refund failed |
| 14 | ProcessingTimeOut | Provider did not respond in time |
| 15 | Verified | Verified |
| 16 | RefundWithCharges | Refunded with charges deducted |
| 17 | RefundReverse | Completed refund reversed by provider |
| 18 | Voided | Voided |
| 19 | Cancelling | Cancellation in progress |
| 20 | InstallmentInProgress | Being processed as instalment plan |

Treat only `1` as paid. Treat `0`, `9`, `14`, `19`, `20` as non-final; requery later.

## Return URL

The customer browser is redirected to `returnUrl` after the attempt. Query parameters appended to it are not documented in the docs. The official WooCommerce plugin reads `ReferenceCode` and `TransactionNumber` from the return query string. Use them only to look up the order and show a page; never to settle.

## Query Payment

`GET /api/services/app/PaymentGateway/Query`

| Query param | Type | Required | Notes |
| --- | --- | --- | --- |
| `TransactionNumber` | string, 24 | conditional | For a specific transaction (normal or within a session) |
| `SessionNumber` | string, 24 | conditional | For a session |
| `Timestamp` | int64 | yes | Epoch ms |

Signature JSON uses camelCase keys, e.g. `{"timestamp":1621851652617,"transactionNumber":"..."}`.

Behaviour: the provider is queried live only when the stored status is Pending, Authorized or InstallmentInProgress; other statuses return the stored value. Querying by `SessionNumber` alone checks up to 3 pending transactions in that session, returns the first Success, else the session status with `transactionNumber: null`.

Response `result`:

| Field | Type | Notes |
| --- | --- | --- |
| `transactionNumber` | string, 24 | |
| `paymentSessionNumber` | string, 24 | |
| `referenceCode` | string, 50 | |
| `status` | int | Payment status code |
| `settlementStatus` | int | `0` Unsettled, `1` Settled, `2` InProgress, `3` Locked, `4` CompleteProcessing |
| `currencyCode` | string, 3 | |
| `amount` | int64 | Request amount, minor units |
| `channelId` | int32 | |
| `providerTransactionNumber` | string | |
| `creationTime` | date-time | |
| `remark` | string | |
| `providerChannelId` | string | |
| `providerPaymentMethod` | string | |
| `providerErrorMessage` | string | |
| `loyaltyType` | int | `1` BCard (loyalty only) |
| `loyaltyId` | int32 | Loyalty only |
| `redemptionPoint`, `redemptionAmount`, `rewardPoint`, `paidAmount` | int64 | Loyalty only; `paidAmount` is the amount actually paid |
| `loyaltyRedemptionStatus`, `loyaltyRewardStatus` | int | `0` Pending, `1` Success, `2` Failed, `3` Void |

Errors: `2010` TransactionNotFound, `2014` ChannelNotAllowedQuery, `2116` QueryProviderError.

## Process Payment (pre-auth and capture)

`POST /api/services/app/PaymentGateway/ProcessPayment`

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `type` | int | yes | `1` PreAuth, `2` FullPayment, `3` PartialPayment (description also mentions `4` Verify, not in the enum) |
| `transactionNumber` | string, 24 | yes | Existing CommercePay transaction |
| `amount` | int | yes | Minor units |
| `channelDetails` | object | no | Card fields as in RequestPayment |
| `savePayment` | bool | no | Tokenise |
| `lastPayment` | bool | no | Undocumented meaning (schema field `LastPayment`) |

Response: `status`, `type`, `transactionNumber`, `partialTransactionNumber`, `amount`. Errors and timestamp requirement: not documented beyond the schema; include `timestamp` if the server returns "Invalid Timestamp."

## Saved Payments (tokenisation)

### Add Saved Payment

`POST /api/services/app/PaymentGateway/AddSavePayment`. Places a real MYR 1.00 verification charge, refunded automatically; callbacks fire for it. One saved method per `customer.username` per channel.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `channelId` | int32 | yes | Must support saved payments |
| `referenceCode` | string, 50 | yes | |
| `ipAddress` | string, 50 | yes | |
| `returnUrl` | string, 500 | yes | |
| `currencyCode` | string | no | |
| `userAgent` | string, 200 | conditional | |
| `callbackUrl` | string, 500 | no | |
| `customer` | object | yes in practice | `username` (your unique customer id) required (error 2102); `email`, `mobileNo`, `name`, `identificationNumber`, `identificationType` |
| `tenantId` | int32 | no | |
| `timestamp` | int64 | yes | |

Response: `transactionNumber` (verification transaction), `redirectUrl`. Errors: `2001` InvalidChannel, `2102` UsernameIsRequired, `2103` UserIsSavedPayment (revoke first), `2104` ChannelNotSupportPreAddPayment.

### Check Saved Payment (TNG only)

`POST /api/services/app/PaymentGateway/CheckSavedPayment` body `uniqueAccountId` (required, the username), `channelId`, `tenantId`, `timestamp`. Response `isSavedPayment` (bool), `maskedAccountNumber` (e.g. `60XXXXX123`, `43******1234`).

### Revoke Saved Payment

`POST /api/services/app/PaymentGateway/RevokeTokenize` body `username` (required), `channelId` (required), `tenantId`, `timestamp` (required). HTTP 200 with empty body on success. Fails if the channel does not support saved payments or nothing is saved.

To charge a saved method, send `RequestPayment` with the same `channelId`, `customer.username`, and `savePayment` as required by the channel. Exact token-charge semantics beyond this are not documented (checked 2026-10-21).

## Recurring Payments

Credit Card and TNG eWallet only. Direct Integration only (not hosted session or plugins). Requires approval from Business Development, who also tell you the recurring-enabled channel(s) (production list shows "Debit/Credit Cards Recurring" `channelId` 26).

First payment: `RequestPayment` with normal fields plus `recurringPaymentInfo`; customer authorizes on the provider page. On Success callback, activate the subscription keyed by `recurringPaymentInfo.id`.

Subsequent charges: CommercePay does not auto-collect. Your scheduler calls `RequestPayment` with the same channel and same `recurringPaymentInfo.id`, a new `referenceCode` and `amount`; no customer redirect. Handle declines with your own retry logic within `numberOfPayments`/`expiryDate`.

`recurringPaymentInfo`:

| Field | Type | Credit Card | Notes |
| --- | --- | --- | --- |
| `id` | string | required | Merchant subscription/customer id |
| `mode` | int | required | `1` Daily, `2` Weekly, `3` Monthly, `4` Yearly |
| `numberOfPayments` | int32 | required | Max 999 |
| `expiryDate` | string | required | `ddMMyyyy`, e.g. `20122022` |
| `isFixedAmount` | bool | required | Fixed amount per charge |
| `maximumPaymentAmount` | int64 | no | Max per charge, minor units |
| `effectiveDate` | string | optional | Start date `ddMMyyyy` |

All channels require `id`, `mode`, `numberOfPayments`. No-code alternative: Recurring Payment Forms in the back office.

## Refunds

### Refund Payment

`POST /api/services/app/PaymentGateway/RefundPayment`

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `transactionNumber` | string, 24 | yes | Original Success transaction |
| `reason` | string | yes | Missing = 2044 |
| `amount` | int32 | partial only | Minor units. Omit or null for full refund. `0`/negative rejected (2042) |
| `timestamp` | int64 | yes | |

Response (`RequestRefundOutputDto`): `tenantId`, `transactionNumber` (original), `refundTransactionNumber`, `referenceCode`, `amount`, `currencyCode`, `refundStatus`, `responseMessage`, `paymentSessionNumber`, `refundPaymentList` { `transactionNumber`, `amount`, `currencyCode`, `refundReason`, `refundStatus` }.

`refundStatus`: `11` Refunded, `12` ProcessingRefund (poll QueryRefund), `13` FailedRefund, `16` RefundWithCharges, `17` RefundReverse, `18` Voided (original voided instead, e.g. same-day).

Errors: `2010` TransactionNotFound (or not Success), `2042` FailedToRequestRefund (in progress, over remaining amount, partial unsupported, provider unavailable, zero amount), `2044` MissingRequireField, `2094` InvalidTenantProfile, `2124` RequestRefundTimeOut, `2130` ChannelNotSupportRefund, `2160` InvalidPaymentGatewaySetting.

### Query Refund Payment

`GET /api/services/app/PaymentGateway/QueryRefund?TransactionNumber=...&Timestamp=...` (both required). Same response shape. Errors: `2010`, `2043` FailedToRefundQuery, `2130`, `2160`.

Rules: support varies by bank and channel (check `refundType` from Get Channel List). Published bank notes: BSN, Public Bank, Co-opbank Pertama full only; Ryt Bank (YTL) none. Cross-border transactions cannot be refunded. Banks may change capability without notice; test in staging.

## Payouts

Payout = `POST /api/services/app/PaymentGateway/RequestPayment` with the payout channel. Requires a payout wallet created and topped up via Business Development.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `channelId` | int | yes | Staging `15`, production `9` |
| `providerChannelId` | string | yes | Bank SWIFT-style code (bank payout) or proxy type `MBNO`/`NRIC`/`BREG` (DuitNow Proxy). See codes file |
| `currencyCode` | string, 3 | yes | `MYR` |
| `amount` | int64 | yes | Minor units, `1050` = 10.50 |
| `referenceCode` | string, 50 | yes | |
| `description` | string, 100 | yes | |
| `ipAddress` | string, 50 | yes | |
| `userAgent` | string, 200 | conditional | |
| `returnUrl` | string, 500 | yes | |
| `callbackUrl` | string, 500 | no | Status callbacks |
| `customer` | object | yes | `email` (150); `mobileNo` (16, required for proxy by mobile); `name` (36, required for bank transfer: account holder name); `username` (36: bank account number, NRIC, BRN, or DuitNow ID for TNG) |
| `localCitizen` | bool | yes | `true` if recipient is Malaysian |
| `timestamp` | int64 | yes | |

Response: `currencyCode`, `amount`, `transactionNumber`, `channelId`, `redirectionType` (`1`), `redirectUrl`. Errors: `2056` Failed Payment Request, `1` Invalid Signature. Payout status arrives by callback and Query Payment. Proxy must be registered with DuitNow and linked to a valid account.

### Query Wallets

`POST /api/services/app/PaymentGateway/QueryWallets` body `tenantId` (required), `timestamp` (required). Response `tenantId`, `tenantName`, `wallets[]` { `id`, `currencyCode`, `balance` (decimal, ringgit), `walletType` (`0` Basic, `1` Premium, `2` Payout), `walletStatus` (`0` Inactive, `1` Active, `2` Suspend) }. Errors: `1005` Tenant Wallet Not Found, tenant not found.

### Query Wallet History

`POST /api/services/app/PaymentGateway/QueryWalletHistory`

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `nextRecord` | int | for page 2+ | First 100 records without it; then pass previous `nextRecords` |
| `dateTimeFrom` | string | yes | UTC ISO 8601 |
| `dateTimeTo` | string | yes | UTC ISO 8601 |
| `walletId` | int | yes | From Query Wallets |

Response `walletId`, `walletBalance` (decimal), `walletHistories[]` { `id`, `createdDate`, `transactionNumber`, `refenceCode` (spelled this way), `amount` (decimal), `currencyCode`, `type` (`1` Topup, `2` Payout, `3` Reverse, `4` Refund, `5` Rebate, `6` Unknown), `description`, `postTransactionWalletBalance` }, `nextRecords`. Error `1005`. The sample body has no `timestamp`; add one if the server rejects with "Invalid Timestamp."

## Amount Rules

- Payment, session, payout, refund, callback and query `amount`: integer minor units (sen). `1000` = MYR 10.00. Never send `10.00` or `1,000.00`.
- Wallet `balance`, `walletBalance`, history `amount`: decimal ringgit.
- Refund response `amount` is typed double but documented as integer minor units.

## Error Codes

| Code | Name / message | Where |
| --- | --- | --- |
| 1 | Invalid Signature. | All signed endpoints |
| (none given) | Invalid Timestamp. | RequestPayment, InitialSession |
| HTTP 401 | Invalid user name or password | Authenticate |
| 1005 | Tenant Wallet Not Found | Wallet APIs |
| 2001 | InvalidChannel | GetProviderChannels, AddSavePayment |
| 2010 | TransactionNotFound | Query, Refund, QueryRefund |
| 2014 | ChannelNotAllowedQuery | Query |
| 2017 | ProviderDoesNotHaveChannels | GetProviderChannels |
| 2018 | FailedToGetProviderChannels | GetProviderChannels |
| 2042 | FailedToRequestRefund | Refund |
| 2043 | FailedToRefundQuery | QueryRefund |
| 2044 | MissingRequireField | Refund (`reason`) |
| 2056 | Failed Payment Request | Payout |
| 2094 | InvalidTenantProfile | Refund |
| 2102 | UsernameIsRequired | AddSavePayment |
| 2103 | UserIsSavedPayment | AddSavePayment |
| 2104 | ChannelNotSupportPreAddPayment | AddSavePayment |
| 2116 | QueryProviderError | Query |
| 2124 | RequestRefundTimeOut | Refund |
| 2130 | ChannelNotSupportRefund | Refund, QueryRefund |
| 2160 | InvalidPaymentGatewaySetting | Refund, QueryRefund |

## Not Documented (checked 2026-10-21)

Rate limits, callback retry policy and source IPs, timestamp tolerance window, return URL query parameters, idempotency key header, sandbox test cards for payins, settlement report API, chargeback/dispute API, void/cancel endpoint (only status `18` Voided exists).

## Known Discrepancies

- Official WooCommerce plugin v1.4 sends `timestamp` in seconds, puts `tenantId` in the body, and omits `Authorization`; the current docs require milliseconds and Bearer + `Abp-TenantId`. Follow the docs.
- Two Query Wallets pages: payout page includes `wallets.id` and status `0/1`; OpenAPI adds `2` Suspend and omits `id`.
- The legacy Stoplight docs (`commerceasiapayments.stoplight.io`) state they are no longer maintained; use `docs.commercepay.asia`.
