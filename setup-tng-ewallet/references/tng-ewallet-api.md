# Touch 'n Go eWallet Open API Reference (Mini Program platform)

Use this reference for factual TNGD Open API guidance. It is distilled from the public TNGD Mini Program developer docs (checked 2026-10-21). Base doc path: `https://miniprogram.tngdigital.com.my/docs/miniprogram_tngd/mpdev/`. Page slugs below are relative to that path.

For a website checkout without a mini program, read `tng-ewallet-web-acquiring.md` instead; it is a different API.

## Source URLs

- API fundamentals: `rekqh-qm`
- Data dictionary: `vs3pkf`
- Message transmission security: `msg_transmission_security`, `encode`, `call_api`
- Payments: `v1_pay`, `dl7y4p` (inquiryPayment), `edgctc` (notifyPayment), `v1_refund`, `tp3ntm` (inquiryRefund)
- Authorization: `v1_auth_prepare`, `qfd7r1` (applyToken), `uh7agv` (cancelToken), `lab2r2` (inquiryUserInfoByAccessToken)
- Messaging: `v2_miniprogram_send`
- Error codes: `error_codes`
- JSAPI: `api_openapi_tradepay`, `api_openapi_getauthcode`, `api_openapi_getopenuserinfo`, `component_open_web-view`
- Settlement: `partner_settlement_process`
- Onboarding plan: `project_planning`

## Integration Model

TNGD Open APIs serve mini programs that run inside the TNG eWallet app. Your server calls TNGD; your mini program front end calls JSAPIs such as `my.tradePay` and `my.getAuthCode`.

Two payment scenarios are documented for `/v1/payments/pay`:

- Cashier Payment (online). Server creates the order, TNGD returns a cashier URL, the mini program opens it with `my.tradePay`. TNGD notifies `paymentNotifyUrl` on final status.
- Agreement Payment (auto debit). After the user binds via `/v1/authorizations/prepare` and the merchant gets an access token from `/v1/authorizations/applyToken`, the server charges directly with the token. Synchronous; notification is not required.

Inside a `web-view`, TNGD says not to process payment in H5 and always call `my.tradePay` (`component_open_web-view`).

## Environments

- Host: "the standard domain name assigned by the wallet backend" (`rekqh-qm`). Not published. TNGD supplies sandbox server access plus a Postman script at sandbox setup, and "Production API end points" by email after UAT (`project_planning`).
- Sample cashier and auth URLs in the docs use `https://m-sd.tngdigital.com.my/...` (sandbox front end). Do not hardcode; use the URL TNGD returns.
- Sandbox mini programs run in the "TNGD Partner Test App"; sandbox releases need no approval (`project_planning`).
- Sandbox test users, wallets, and balances: not documented in official docs (checked 2026-10-21). TNGD provides them during sandbox setup.

## Request Structure

URL: `https://{host}/acl/api/{restful_path}` where `restful_path` is `/{version}/...`, for example `/v1/payments/pay`. `/v1/...` and `/v2/...` are different interfaces.

Method: always `POST`. Body: JSON.

Request headers (names case-insensitive):

| Header | Required | Notes |
| --- | --- | --- |
| `Signature` | Yes | `algorithm=RSA256, keyVersion=1, signature=<value>` |
| `Encrypt` | No | Only for sensitive data: `algorithm=RSA_AES, keyVersion=1, symmetricKey=<encrypted key>`. Only `RSA_AES` is supported. Encryption procedure itself is not documented. |
| `Content-Type` | No | `application/json; charset=UTF-8`. The charset is used for signing and encryption. |
| `Client-Id` | Yes | Identifies the client and its keys. In mini program OAuth, it is also the OAuth client id. |
| `Request-Time` | Yes | RFC3339, accurate to milliseconds, for example `2019-04-04T12:08:56.253+05:30`. |

`Signature` keys: `algorithm` (`RSA256` default, or `ECC224`; case-insensitive), `keyVersion` (defaults to the latest key for the Client-Id), `signature`.

Response headers: `signature`, optional `Encrypt`, `Content-Type`, optional `traceId` (use for troubleshooting), `client-id`, `response-time` (RFC3339, ms).

Every response body contains `result`:

| Field | Type | Notes |
| --- | --- | --- |
| `resultStatus` | String | `S` success, `F` failed, `U` unknown, `A` accepted (continue per `actionForm`) |
| `resultCode` | String(64) | Error code when status is F, U, or A |
| `resultMessage` | String(256) | Detail |

Encoding rules (`rekqh-qm`, `encode`): byte data (signature, encrypted content) is Base64 encoded; URL values are URL-encoded before transmission (for example `https%3A%2F%2Fwww.merchant.com%2FauthorizationResult`).

## Signature

Source: `call_api`. Mandatory for all requests and responses (`msg_transmission_security`).

### Sign a request

String to sign (`Content_To_Be_Signed`):

```text
<HTTP_METHOD> <HTTP_URI>
<Client-Id>.<Request-Time>.<HTTP_BODY>
```

- `HTTP_METHOD`: `POST`.
- `HTTP_URI`: the path part of the URL. The docs example signs `/api/v1/payments/pay` for `https://example.com/api/v1/payments/pay`. The fundamentals page gives the real path as `/acl/api/v1/...`. Sign the exact path of the URL you call; confirm with TNGD's Postman pack.
- `Client-Id`, `Request-Time`: exactly the header values.
- `HTTP_BODY`: the exact body string sent.
- One newline between the first and second line; dots between the three second-line parts.

Signature: `base64UrlEncode(sha256withrsa(Content_To_Be_Signed, privateKey))`. The doc examples show standard Base64 that is then URL-encoded (`/` as `%2F`, `+` as `%2B`, `=` as `%3D`).

Header: `Signature: algorithm=RSA256, keyVersion=<n>, signature=<generatedSignature>`.

### Verify a response

1. Get the TNGD public key (sent at onboarding; the key is tied to `Client-Id` and `keyVersion`).
2. Build `Content_To_Be_Validated`:

```text
<HTTP_METHOD> <HTTP_URI>
<Client-Id>.<Response-Time>.<HTTP_BODY>
```

`HTTP_METHOD` and `HTTP_URI` are those of your request; `Client-Id` and `Response-Time` come from the response headers; `HTTP_BODY` is the raw response body.

3. Extract `target_signature` from the response `signature` header.
4. `sha256withrsa_verify(base64UrlDecode(target_signature), Content_To_Be_Validated, serverPublicKey)`.

### Pseudo-code

```text
body      = json_serialize(payload)            # serialize once, send these bytes
uri       = "/acl/api/v1/payments/pay"          # path of the URL actually called
reqTime   = now_rfc3339_millis()                # e.g. 2026-10-21T10:15:30.123+08:00
toSign    = "POST " + uri + "\n" + CLIENT_ID + "." + reqTime + "." + body
sig       = url_encode(base64(rsa_sha256_sign(MERCHANT_PRIVATE_KEY, utf8(toSign))))
headers   = { "Content-Type": "application/json; charset=UTF-8",
              "Client-Id": CLIENT_ID, "Request-Time": reqTime,
              "Signature": "algorithm=RSA256, keyVersion=" + KEY_VERSION + ", signature=" + sig }
resp      = http_post(HOST + uri, body, headers)
check     = "POST " + uri + "\n" + resp.headers["client-id"] + "." + resp.headers["response-time"] + "." + resp.raw_body
ok        = rsa_sha256_verify(TNGD_PUBLIC_KEY, utf8(check), base64_decode(url_decode(parse_sig(resp.headers["signature"]))))
```

### Signing the notification acknowledgement

TNGD uses "your public key ... to decrypt your response to the Payment notify callback" (`project_planning`). So your `notifyPayment` response must carry `client-id`, `response-time`, and `signature` headers built like a response signature, signed with your private key. The exact headers TNGD sends on the notification request, and the exact URI to use when verifying it, are not documented in official docs (checked 2026-10-21). Apply the same scheme (TNGD public key, request path of your notify URL, `Client-Id.Request-Time.body`) and confirm with TNGD in sandbox.

## Endpoint Catalogue

All `POST`, path prefix `/acl/api`.

| Path | Direction | Purpose | Doc |
| --- | --- | --- | --- |
| `/v1/payments/pay` | Merchant to TNGD | Create cashier payment or charge agreement token | `v1_pay` |
| `/v1/payments/inquiryPayment` | Merchant to TNGD | Query payment status | `dl7y4p` |
| `/v1/payments/notifyPayment` | TNGD to merchant `paymentNotifyUrl` | Final payment result notification | `edgctc` |
| `/v1/payments/refund` | Merchant to TNGD | Full or partial refund | `v1_refund` |
| `/v1/payments/inquiryRefund` | Merchant to TNGD | Query refund status | `tp3ntm` |
| `/v1/authorizations/prepare` | Merchant to TNGD | Start auto debit binding, returns `authURL` | `v1_auth_prepare` |
| `/v1/authorizations/applyToken` | Merchant to TNGD | Exchange authCode or refresh token for access token | `qfd7r1` |
| `/v1/authorizations/cancelToken` | Merchant to TNGD | Revoke access token (unbind) | `uh7agv` |
| `/v1/customers/user/inquiryUserInfoByAccessToken` | Merchant to TNGD | Get user profile per granted scopes | `lab2r2` |
| `/v2/customers/message/sendByAccessToken` | Merchant to TNGD | Send templated message to user | `v2_miniprogram_send` |

Not documented: capture, void/cancel of a payment, close order, payouts, settlement API (settlement is SFTP only), webhooks registration API, rate limits per endpoint, IP allowlists for API calls.

## /v1/payments/pay

Request:

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `partnerId` | String(32) | Yes | Partner ID allocated by TNGD |
| `appId` | String(32) | No | Mini program ID |
| `productCode` | String(32) | Yes | Cashier `51051000101000100001`; Agreement `51051000101000100031`. Determines fee and limit contract. |
| `paymentOrderTitle` | String(256) | Yes | Order title |
| `paymentRequestId` | String(64) | Yes | Your unique ID; idempotency key with `partnerId`. Same ID after a final S or F returns the same result. |
| `paymentAmount` | Amount | Yes | Amount charged |
| `paymentMethods` | PaymentMethod | No | Funding method |
| `paymentAuthCode` | String(128) | No | Access token when `isAgreementPay`; user auth code when `isPaymentCode` |
| `paymentFactor` | PaymentFactor | Yes | `{"isCashierPayment": true}` or `{"isAgreementPay": true}` |
| `paymentExpiryTime` | String/Datetime | No | ISO 8601 order close time |
| `paymentReturnUrl` | String(1024) | No | Where the cashier returns the user |
| `paymentNotifyUrl` | String(1024) | No | Where TNGD posts `notifyPayment`. Send it for cashier payments. |
| `mcc` | String(32) | No | Merchant category code |
| `extraParams` | map | No | Keys `ORDER` (Order model) and `shopInfo` (`shopId`, `shopName`). Display and reporting only; not used for fund movement. |
| `extendInfo` | String(4096) | No | JSON string with `customerBelongsTo` = `siteName` from `my.getSiteInfo`; "mandatory" in the mini program scenario. Samples use `TNG`. |
| `envInfo` | EnvInfo | Yes | Use `terminalType` `MINI_APP` when initiated from a mini program |

Response:

| Field | Type | Notes |
| --- | --- | --- |
| `result` | Result | Status |
| `paymentId` | String(64) | TNGD payment ID |
| `paymentTime` | Datetime | Success time |
| `actionForm` | ActionForm | For `A`: `actionFormType` `REDIRECTION` with `redirectionUrl` (cashier) or `ORDER_CODE` with `orderCode` |
| `authExpiryTime` | Datetime | Only when `isAuthorizationPayment` |
| `extendInfo` | String(4096) | |

Result logic:

- `S`: success (agreement pay). For payment evaluation, S means only "evaluation OK"; for authorization payment, S means authorized, capture pending.
- `A` / `ACCEPT`: continue with `actionForm` (open cashier).
- `F`: failed; the same request will not succeed again.
- `U`: unknown. Query or wait for notification. Do not mark failed or success, and do not refund offline.

Result codes: `S SUCCESS`; `U UNKNOWN_EXCEPTION`, `U REQUEST_TRAFFIC_EXCEED_LIMIT`, `U PAYMENT_IN_PROCESS`; `A ACCEPT`; `F REPEAT_REQ_INCONSISTENT`, `PROCESS_FAIL`, `INVALID_API`, `PARAM_ILLEGAL`, `ACCESS_DENIED`, `PAYMENT_AMOUNT_EXCEED_LIMIT`, `USER_NOT_EXIST`, `USER_STATUS_ABNORMAL`, `USER_BALANCE_NOT_ENOUGH`, `PARTNER_NOT_EXIST`, `PARTNER_STATUS_ABNORMAL`, `RISK_REJECT`, `CURRENCY_NOT_SUPPORT`, `ORDER_STATUS_INVALID`, `INVALID_ACCESS_TOKEN`, `USER_AMOUNT_EXCEED_LIMIT`, `EXPIRED_ACCESS_TOKEN`, `AUTH_CODE_ALREADY_USED`, `INVALID_CODE`, `EXPIRED_AGENT_TOKEN`, `INVALID_AGENT_TOKEN`.

Cashier request sample (from docs, IDs are doc samples):

```json
{
  "partnerId": "<PARTNER_ID>",
  "paymentRequestId": "ord_60ff9d6bc39f8a001711510a",
  "paymentOrderTitle": "MYR",
  "productCode": "51051000101000100001",
  "mcc": "4399",
  "paymentAmount": { "currency": "MYR", "value": "10000" },
  "paymentFactor": { "isCashierPayment": true },
  "paymentNotifyUrl": "https://merchant.example/tng/notify",
  "extraParams": {
    "ORDER": { "referenceOrderId": "<ORDER_ID>", "orderAmount": { "currency": "MYR", "value": "10000" } },
    "shopInfo": { "shopId": "<SHOP_ID>", "shopName": "<SHOP_NAME>" }
  },
  "envInfo": { "terminalType": "MINI_APP" }
}
```

Response sample:

```json
{
  "result": { "resultCode": "ACCEPT", "resultMessage": "accept", "resultStatus": "A" },
  "paymentId": "20210727211212800110171824907593592",
  "actionForm": { "actionFormType": "REDIRECTION", "redirectionUrl": "https://m-sd.tngdigital.com.my/s/cashier/index.html?bizNo=...&sign=..." }
}
```

Front end: `my.tradePay({ paymentUrl: actionForm.redirectionUrl, success, fail })`.

## my.tradePay (JSAPI)

Params: `tradeNO` (String, optional), `orderStr` (String, optional), `paymentUrl` (String, optional; the `redirectionUrl` from `/v1/payments/pay`). Success callback returns `resultCode`:

| Code | Meaning |
| --- | --- |
| `9000` | Pay success |
| `8000` | Trade processing |
| `4000` | Pay failed |
| `6001` | User cancelled |
| `6002` | Network exception |
| `6004` | Unknown result, may be success |
| `99` | User tapped forgot password and quit (iOS only) |

These are client hints. Confirm on the server via notification or `inquiryPayment`.

## /v1/payments/inquiryPayment

Use when the result has not arrived. Poll every 5 s, up to 1 minute (doc recommendation).

Request: `partnerId` String(32) required; `paymentId` String(64) optional; `paymentRequestId` String(64) optional; `extendInfo` String(4096) optional (`customerBelongsTo`). If both IDs are sent, `paymentId` wins.

Response: `result`; `paymentId`; `paymentRequestId`; `paymentAmount` (Amount); `paymentTime`; `paymentStatus` (`SUCCESS`, `FAIL`, `PROCESSING`, `CANCELLED`); `paymentFailReason` String(256); `authExpiryTime`; `extendInfo`.

Logic: `result.resultStatus` S means the query worked, then read `paymentStatus`. F with `ORDER_NOT_EXIST` means not accepted, treat as failed; other F needs human review. U means retry.

Result codes: `S SUCCESS`; `U UNKNOWN_EXCEPTION`, `U REQUEST_TRAFFIC_EXCEED_LIMIT`; `F ORDER_NOT_EXIST`, `INVALID_API`, `PARAM_ILLEGAL`, `PROCESS_FAIL`, `ACCESS_DENIED`, `EXPIRED_AGENT_TOKEN`, `INVALID_AGENT_TOKEN`.

## /v1/payments/notifyPayment (TNGD to you)

Sent to `paymentNotifyUrl` when payment reaches final status. Not sent for synchronous flows (agreement pay, B scan C).

Request fields:

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `paymentId` | String(64) | Yes | TNGD payment ID |
| `paymentRequestId` | String(64) | Yes | Your original ID |
| `paymentAmount` | Amount | Yes | Amount collected |
| `paymentTime` | Datetime | No | Success time |
| `paymentResult` | Result JSON | Yes | `resultStatus` `S` success or `F` failed, plus `resultCode`, `resultMessage` |
| `paymentFailReason` | String(256) | No | When failed |
| `extendInfo` | String(4096) | No | |

Samples also include `customerId`. A failed sample uses `resultCode` `ORDER_IS_CLOSED` (usually expired).

Required response (signed, see Signature):

```json
{ "result": { "resultCode": "SUCCESS", "resultStatus": "S", "resultMessage": "success" } }
```

Rules from the doc (updated 2026-04-16): return `S` once the message is received; the ack must not depend on internal processing; process asynchronously; internal failures must not change the ack. If you do not return `S`, TNGD retries, which can cause duplicates. `U` or any other response is treated as retry.

Handler outline:

```text
raw = read_raw_body()
verify signature (TNGD public key) -> reject 401 if bad, do not ack S
store event (paymentId unique) -> if duplicate, still ack S
respond signed {"result":{"resultCode":"SUCCESS","resultStatus":"S","resultMessage":"success"}}
async: load local payment by paymentRequestId; check paymentId, amount.value, currency MYR;
       if paymentResult.resultStatus == "S" mark paid once; if "F" mark failed if still pending
```

## /v1/payments/refund

Full or partial; multiple refunds allowed; total cannot exceed the payment. Returns SUCCESS when funds are deducted from the merchant.

Request: `partnerId` String(32) Yes; `refundRequestId` String(64) Yes (idempotency with `partnerId`); `paymentId` String(64) No; `paymentRequestId` String(64) No (one of the two IDs is required); `refundAmount` Amount Yes (same currency as payment); `refundReason` String(256) No; `extendInfo` String(4096) No (`customerBelongsTo`).

Response: `result`; `refundId` String(64) (mandatory when S); `refundTime` (mandatory when S); `extendInfo`.

Logic: S success. F failed (for example `REFUND_WINDOW_EXCEED`, `REFUND_AMOUNT_EXCEED`). U unknown: call `inquiryRefund` or retry the same refund; do not mark failed or success, do not refund offline.

Result codes: `S SUCCESS`; `U UNKNOWN_EXCEPTION`, `U REQUEST_TRAFFIC_EXCEED_LIMIT`, `U REFUND_IN_PROCESS`; `F REPEAT_REQ_INCONSISTENT`, `PROCESS_FAIL`, `PARAM_ILLEGAL`, `INVALID_API`, `ACCESS_DENIED`, `PARTNER_STATUS_ABNORMAL`, `ORDER_NOT_EXIST`, `ORDER_STATUS_INVALID`, `REFUND_WINDOW_EXCEED`, `REFUND_AMOUNT_EXCEED`, `PARTNER_BALANCE_NOT_ENOUGH`, `CURRENCY_NOT_SUPPORT`, `EXPIRED_AGENT_TOKEN`, `INVALID_AGENT_TOKEN`.

The refund window length is not documented (checked 2026-10-21).

## /v1/payments/inquiryRefund

Poll every 5 s, up to 1 minute.

Request: `partnerId` Yes; `refundId` String(64) No; `refundRequestId` String(64) No; `extendInfo` No. If both IDs are sent, `refundId` wins.

Response: `result`; `refundId`; `refundRequestId`; `refundAmount`; `refundReason`; `refundTime`; `refundStatus` (`PROCESSING`, `SUCCESS`, `FAIL`); `refundFailReason`; `extendInfo`.

Logic: F with `REFUND_NOT_EXIST` means not accepted, treat as failed; other F needs human review; U retry.

Result codes: `S SUCCESS`; `U UNKNOWN_EXCEPTION`, `U REQUEST_TRAFFIC_EXCEED_LIMIT`; `F REFUND_NOT_EXIST`, `INVALID_API`, `PARAM_ILLEGAL`, `PROCESS_FAIL`, `ACCESS_DENIED`, `EXPIRED_AGENT_TOKEN`, `INVALID_AGENT_TOKEN`.

## Auto Debit (Agreement Pay) and User Authorization

Flow (`v1_pay`, `v1_auth_prepare`): server calls `prepare`, passes `authURL` to the mini program, mini program calls the signing JSAPI (docs name it `my.signContract` and `my.tngdSignContract`; neither JSAPI page is public), user approves, mini program gets an auth code, server calls `applyToken`, then charges with `/v1/payments/pay` (`isAgreementPay`, `paymentAuthCode` = access token, agreement `productCode`). Unbind with `cancelToken`.

### /v1/authorizations/prepare

Request: `authClientId` String(128) Yes (client ID from TNGD); `partnerReqId` String(128) Yes (unique); `partnerId` String(32) Yes; `authClientDisplayName` String(128) Yes; `authRedirectUrl` String(1024) Yes; `authLogoUrl` String(1024) Yes; `scope` Array Yes (`AGREEMENT_PAY`); `authState` String(128) Yes (unguessable, for CSRF); `extendInfo` String(1024) No; `envInfo` EnvInfo No (sample: `terminalType` `SYSTEM`, `orderTerminalType` `MINI_APP`).

Response: `authId`, `authURL`, `authClientId`, `result`. Codes: `S SUCCESS`; `U UNKNOWN_EXCEPTION`, `REQUEST_TRAFFIC_EXCEED_LIMIT`; `F PROCESS_FAIL`, `PARAM_ILLEGAL`, `ACCESS_DENIED`, `INVALID_API`, `INVALID_SCOPE`, `INVALID_CLIENT`, `INVALID_REDIRECT_URL`.

### /v1/authorizations/applyToken

Request: `referenceClientId` String(128) No; `grantType` String(16) Yes (`AUTHORIZATION_CODE` or `REFRESH_TOKEN`); `authCode` String(32) (when AUTHORIZATION_CODE); `refreshToken` String(128) (when REFRESH_TOKEN); `extendInfo` No.

Response: `result`; `accessToken` String(128); `accessTokenExpiryTime`; `refreshToken` String(128); `refershTokenExpiryTime` (spelled this way in the table; the sample uses `refreshTokenExpiryTime`, so accept both); `customerId` String(64); `extendInfo`. Sample also has `accessTokenExpiresIn`, `refreshTokenExpiresIn`.

Codes: `S SUCCESS`; `U UNKNOWN_EXCEPTION`, `REQUEST_TRAFFIC_EXCEED_LIMIT`; `F PROCESS_FAIL`, `PARAM_ILLEGAL`, `ACCESS_DENIED`, `INVALID_API`, `AUTH_CLIENT_UNSUPPORTED_GRANT_TYPE`, `INVALID_AUTH_CLIENT`, `INVALID_AUTH_CLIENT_STATUS`, `INVALID_REFRESH_TOKEN`, `EXPIRED_REFRESH_TOKEN`, `USED_REFRESH_TOKEN`, `INVALID_CODE`, `USED_CODE`, `EXPIRED_CODE`, `REFERENCE_CLIENT_ID_NOT_MATCH`, `EXPIRED_AGENT_TOKEN`, `INVALID_AGENT_TOKEN`. F or U: guide the user to retry.

### /v1/authorizations/cancelToken

Request: `accessToken` String(128) Yes; `extendInfo` No. Response: `result`, `extendInfo`. After cancel, the refresh token is also unusable. Codes add `INVALID_ACCESS_TOKEN`, `INVALID_AUTH_CLIENT`, `INVALID_AUTH_CLIENT_STATUS`, `EXPIRED_ACCESS_TOKEN`.

### /v1/customers/user/inquiryUserInfoByAccessToken

Request: `accessToken` Yes; `extendInfo` No. Response: `result`, `userInfo` (OpenUserInfo). Codes add `INVALID_ACCESS_TOKEN`, `INVALID_AUTH_CLIENT`, `EXPIRED_ACCESS_TOKEN`.

### my.getAuthCode scopes

`scopes` default `auth_base`. Callback: `authCode`, `authErrorScopes`, `authSuccessScopes`. Scopes are case-sensitive.

| Scope | Silent | Preassigned | Data |
| --- | --- | --- | --- |
| `auth_base` / `BASE_USER_INFO` | Yes | Yes | Wallet account ID |
| `auth_user` / `USER_INFO` | No | Yes | Registered name, mobile, email |
| `USER_NAME`, `USER_AVATAR`, `USER_GENDER`, `USER_BIRTHDAY`, `USER_NATIONALITY`, `USER_CONTACTINFO`, `USER_ADDRESS` | No | No | As named |
| `USER_INFO_EKYC` | No | No | Verified name, ID type and number, nationality, address, DOB, occupation, nature of business |
| `ASSET_PROFILE` | No | No | Total balance, quick payment status, mobile |

### /v2/customers/message/sendByAccessToken

Request: `appId` String(32) Yes; `accessToken` String(128) Yes; `requestId` String(64) Yes; `templateCode` String(64) Yes (from the Mini Program portal); `templates` Array of ContentTemplate (max 5) Yes; `extendInfo` No. Response: `result` (S, A `ACCEPT`, U, F), `extendInfo`.

## Data Dictionary

- Amount: `currency` String(3) ISO 4217; `value` String(16), positive integer in the smallest unit. MYR is in sen.
- ActionForm: `actionFormType` (`ORDER_CODE`, `REDIRECTION`), `orderCode` String(2048), `redirectionUrl` String(4096).
- EnvInfo: `terminalType` required (`MINI_APP`, `APP`, `WEB`, `WAP`, `SYSTEM`); `osType` (`IOS`, `ANDROID`); `userAgent` (1024); `deviceTokenId` (128); `clientIp` (64); `cookieId` (128); `extendInfo` (4096). The prepare sample also uses `orderTerminalType`.
- PaymentFactor (booleans, default false): `isPaymentEvaluation`, `isOrderCode`, `isPaymentCode`, `isAgreementPay`, `isCashierPayment`, `isAuthorizationAndPay`, `isAuthorizationPayment`.
- PaymentMethod: `paymentMethodType` required (`BALANCE`, `COUPON`, `CREDIT_CARD`, `DEBIT_CARD`), `paymentMethodId` (128), `extendMetaData` (2048).
- Order: `referenceOrderId` (64) required, `orderDescription` (256), `orderAmount` required, `orderCreateTime`, `referenceMerchant` (Merchant), `goods` (Goods), `shipping` (Shipping), `buyer` (Buyer), `extendInfo` (2048).
- Merchant: `referenceMerchantId` (32), `merchantMCC` (32), `merchantName` (256) required; `merchantDisplayName`, `merchantAddress`, `merchantRegisterDate`, `store`.
- Store: `referenceStoreId` (64) required; `storeName`, `storeMCC`, `storeDisplayName`, `storeTerminalId`, `storeOperatorId`, `storeAddress`, `storePhoneNo`.
- Address: `region` (ISO 3166 alpha-2) required; `state`, `city`, `address1`, `address2`, `zipCode`.
- Goods: `referenceGoodsId`, `goodsName` required; `goodsCategory`, `goodsBrand`, `goodsUnitAmount`, `goodsQuantity`, `goodsUrl`, `extendInfo`.
- Shipping: `shippingName` (UserName), `shippingAddress` required; `shippingCarrier`, `shippingPhoneNo`, `shippingFee`.
- Buyer: `referenceBuyerId`, `buyerName`, `buyerPhoneNo`.
- OpenUserInfo: `userId`, `userName`, `avatar`, `gender` (`F`/`M`), `birthday`, `nationality` (ISO alpha-3), `contactInfos`, `extendInfo` (stringified `ekycData` for `USER_INFO_EKYC`).
- EKYCInfo `kycLevel`: `SGPROFILE`, `MYPROFILE`, `BINDCARD`, `EKYC`, `PREMIUM`, `PREMIUM+`. `ekycIdType`: `MYKAD`, `PASSPORT`, `POLICE`, `ARMY`, `OTHERS`.

## Common Error Codes

| resultCode | Status | Action |
| --- | --- | --- |
| `PROCESS_FAIL` | F | Do not retry; contact TNGD support |
| `PARAM_ILLEGAL` | F | Fix header or body fields |
| `INVALID_API` | F | Wrong or inactive API |
| `ACCESS_DENIED` | F | Check permissions and resultMessage |
| `REQUEST_TRAFFIC_EXCEED_LIMIT` | F on the error codes page, U in API tables | Back off and reduce call rate |
| `EXPIRED_ACCESS_TOKEN` | F | Refresh via `applyToken` |
| `UNKNOWN_EXCEPTION` | U | Retry |

Rate limits: numeric values not documented (checked 2026-10-21).

## Settlement File (SFTP)

From `partner_settlement_process`:

- T+1 settlement to the bank account captured at onboarding.
- CSV on TNGD SFTP, generated 9 AM on T+1. Folder `settlement/<date>/`. Your IP must be whitelisted; host, port, user (Partner ID) and password come at onboarding.
- File name: `MERCHANT_SETTLEMENT_<PartnerID>_<yyyymmdd>` (transaction date).
- Summary fields: `Title` (`SETTLEMENT_SUMMARY`), `INVOICE_NO`, `INVOICE_DATE` (DD/MM/YYYY), `SETTLEMENT_BATCH_NO`, `MERCHANT_ID`, `MERCHANT_NAME`, `NET_TRANSACTION_AMOUNT`, `NET_COMMISSION_AMOUNT`, `NET_SETTLEMENT_AMOUNT`, `TOTAL_COUNT`.
- Detail fields: `MERCHANT_ID`, `MERCHANT_NAME`, `SHOP_ID`, `SHOP_NAME`, `MERCHANT_TRANS_ID` (your ID), `ACQUIREMENT_ID` (TNGD ref), `TRANSACTION_TYPE` (Payment/Refund), `TRANSACTION_REQUEST_ID` (your refund request ID), `TRANSACTION_ID` (= ACQUIREMENT_ID for payment, refund ID for refund), `PRODUCT_CODE`, `TRANSACTION_AMOUNT` (cent, positive), `TRANSACTION_CURRENCY` (MYR), `TRANSACTION_DATETIME` (`yyyy/MM/dd HH:MM:SS`), `COMMISSION_AMOUNT` (cent; fee charged, or returned on refund), `COMMISSION_TAX` (reserved), `SETTLE_AMOUNT` (= transaction - commission, cent), `PAY_METHOD` (default `balance`, multiple separated by `|`), `MERCHANT_CUST_ID`.

## Deeplinks

`tngdwallet://client/dl/mp?mpid=<MINI_PROGRAM_ID>&path=<url-encoded page path>&<extra params>`. `mpid` and `path` are reserved. Extra params arrive in `onLaunch`/`onShow` `options.query`. Requires TNGD app 1.7.39+ (1.7.50+ for AC mini programs). Base64 encode values with characters such as `+`.
