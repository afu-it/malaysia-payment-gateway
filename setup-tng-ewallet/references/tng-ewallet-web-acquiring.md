# Touch 'n Go eWallet Web Acquiring API (from the TNGD WooCommerce plugin)

Use this reference when the merchant wants TNG eWallet on a website or app checkout without building a mini program.

## Source And Trust Level

- Plugin page: `https://wordpress.org/plugins/touch-n-go-ewallet/` ("Touch 'n Go eWallet for WooCommerce", v1.0.1, updated 2026-07-27, contributor account `tngd01`).
- Source zip: `https://downloads.wordpress.org/plugin/touch-n-go-ewallet.1.0.1.zip`, file `includes/class-wc-gateway-tngd.php`.
- The listing links to TNGD's onboarding form and TNGD policies and calls the API "external API endpoints provided by Touch 'n Go eWallet (TNGD)". The plugin header still says "Author: Custom Integration" and its comments say it was ported from a Magento module ("Sweetmag TngPayment"). TNGD has not published a separate web acquiring spec (checked 2026-10-21).

So treat everything here as the behaviour of a working client, not as a TNGD contract. Confirm field names, version strings, and the notification format with TNGD during sandbox setup.

## Environments

- Sandbox base (plugin default): `https://api-sd.tngdigital.com.my`.
- Production base: not published. TNGD provides it at onboarding.
- Paths appended to the base:

| Function | Path | Purpose |
| --- | --- | --- |
| `alipayplus.acquiring.order.create` | `/alipayplus/acquiring/order/create.htm` | Create order, get `checkoutUrl` |
| `alipayplus.acquiring.order.query` | `/alipayplus/acquiring/order/query.htm` | Query order status |
| `alipayplus.acquiring.order.refund` | `/alipayplus/acquiring/order/refund.htm` | Refund |
| `alipayplus.acquiring.refund.query` | `/alipayplus/acquiring/refund/query.htm` | Query refund |

All are `POST` with `Content-Type: application/json`.

## Credentials (plugin settings, per environment)

- Client ID, Client Secret, Merchant ID, Product Code, Merchant Private Key (RSA), MCC.
- A TNGD Public Key field exists in code but is commented out in the settings form.

## Envelope

Request body:

```json
{
  "request": {
    "head": {
      "version": "1.0",
      "function": "alipayplus.acquiring.order.create",
      "clientId": "<CLIENT_ID>",
      "clientSecret": "<CLIENT_SECRET>",
      "reqTime": "2026-10-21T10:15:30+08:00",
      "reqMsgId": "<uuid4>"
    },
    "body": { }
  },
  "signature": "<base64 RSA-SHA256 over the JSON of request>"
}
```

- `reqTime`: `Y-m-d\TH:i:sP` in `Asia/Kuala_Lumpur` (seconds, no milliseconds).
- `reqMsgId`: UUID v4 with dashes.
- `version`: the plugin sends `1.0` for create, query, and refund, and `2.8.2` for its connection test and refund query. `clientSecret` is sent in create, query, and refund heads but not in the other two. Confirm the expected values with TNGD.

Response body (fields the plugin reads):

```json
{
  "response": {
    "body": {
      "resultInfo": { "resultStatus": "S", "resultCodeId": "00000000", "resultCode": "SUCCESS", "resultMsg": "success" },
      "acquirementId": "<TNGD_ID>",
      "checkoutUrl": "https://..."
    }
  },
  "signature": "<base64>"
}
```

A `response.head` object is likely present (Alipay+ convention) but the plugin does not read it.

## Signature

Signing (plugin `generate_signature`):

```text
payload   = json_encode(request)            # PHP json_encode, no flags
signature = base64(openssl_sign(payload, merchantPrivateKey, SHA256))   # RSA SHA256withRSA, PKCS#1 v1.5
```

- Only the `request` object is signed, not the outer wrapper.
- PHP `json_encode` without flags escapes `/` as `\/` and non-ASCII as `\uXXXX`. The bytes signed must match the bytes TNGD re-reads. In other languages, serialize `request` once to a string, sign that string, and build the outer body by string concatenation: `{"request":` + requestJson + `,"signature":"` + sig + `"}`.
- Keys: PEM. The plugin wraps bare Base64 private keys in `-----BEGIN PRIVATE KEY-----` (PKCS#8) and public keys in `-----BEGIN PUBLIC KEY-----`.

Verification (plugin `verify_signature`, defined but never called):

```text
ok = openssl_verify(json_encode(data.response), base64_decode(data.signature), tngdPublicKey, SHA256) == 1
```

Do verify responses and notifications in your own code. Verify over the raw `response` substring of the received body when possible, since re-encoding parsed JSON may change bytes.

## order.create

Body sent by the plugin:

| Field | Value / Notes |
| --- | --- |
| `order.orderTitle` | String, plugin uses `TNG ORDER` |
| `order.orderAmount.currency` | `MYR` |
| `order.orderAmount.value` | Integer in sen (`round(total * 100)`). The plugin sends a JSON number, not a string. |
| `order.merchantTransId` | Your unique order ID (plugin: WooCommerce order ID) |
| `order.expiryTime` | Now + 10 minutes, `Y-m-d\TH:i:sP` |
| `merchantId` | Merchant ID from TNGD |
| `productCode` | Product code from TNGD |
| `envInfo.terminalType` | `SYSTEM` |
| `envInfo.orderTerminalType` | `WEB` (desktop), `WAP` (mobile browser), `APP` (in-app webview or REST call) |
| `notificationUrls[]` | `[{ "url": "<return URL>", "type": "PAY_RETURN" }]` |

Response: on `resultInfo.resultStatus == "S"` read `checkoutUrl` (redirect the browser there) and `acquirementId` (store it; query and refund use it). Anything other than S is treated as rejection.

The plugin does not send a server notification URL in the create body. Its `notify_url` setting is never sent. How TNGD learns the server notification URL for web acquiring is not documented (checked 2026-10-21); ask TNGD whether a `NOTIFICATION` type entry in `notificationUrls` is supported or whether the URL is configured on their side.

## order.query

Body: `merchantId`, `acquirementId`. (The connection test queries by `merchantTransId` instead and treats `ORDER_NOT_EXIST` as proof the credentials work.)

Response fields read: `resultInfo`, `acquirementId`, `statusDetail.acquirementStatus`, `statusDetail.frozen`.

`acquirementStatus` values handled:

| Value | Plugin action |
| --- | --- |
| `SUCCESS` | Mark paid |
| `INIT`, `PAYING`, `MERCHANT_ACCEPT` | Still in progress; query again later |
| `CLOSED`, `CANCELLED`, `PAYMENT_EXPIRED` | Cancel the pending order |

Query result code `TARGET_NOT_FOUND`: the plugin cancels the order if it is older than 1 hour.

## order.refund

Body: `requestId` (unique per refund; plugin uses `REFUND_<orderId>_<unixtime>`), `merchantId`, `acquirementId`, `refundAmount` `{currency, value}` (value in sen, integer).

Response: S means success; read `refundId`. Store every `requestId` for later refund query.

## refund.query

Body: `merchantId`, `acquirementId`. Response format is not parsed by the plugin. Note: the plugin reads `merchant_id` and `client_id` options without the environment suffix here, which looks like a bug; use the same credentials as the other calls.

## Notification (TNGD to merchant)

Fields the plugin reads from the raw JSON body:

- `response.body.resultInfo.resultCodeId` (required; `00000000` means success)
- `response.body.resultInfo.resultStatus` (`S` means success)
- `response.body.resultInfo.resultCode`
- `response.body.merchantTransId`
- `response.body.acquirementId`

The plugin replies with plain text `SUCCESS` and HTTP 200. Missing fields get HTTP 400.

Gaps to fix in your own handler:

1. The plugin does not verify the notification signature. Verify `signature` against `response` with the TNGD public key before acting.
2. It does not check amount or currency. Query `order.query` and compare with the stored order before marking paid.
3. It marks the order failed on any non-success notification. Prefer to keep the order pending and query, so a late success is not lost.

## Return Flow

The return URL carries `ref=base64("TNG" + orderId)`. On return the plugin queries `order.query` and settles from the query result, not from the redirect. Do the same.

## Fallback Polling

The plugin runs an hourly cron over orders pending longer than a configurable timeout (default 30 minutes) and recommends a real cron every 15 minutes. It also queries TNGD before allowing a pending order to be cancelled and blocks the cancel if `acquirementStatus` is `SUCCESS`.

## Error Codes Seen In Code

`ORDER_NOT_EXIST`, `ORDER_STATUS_INVALID`, `TARGET_NOT_FOUND`, `INVALID_SIGNATURE`, `CLIENT_AUTHENTICATION_FAILED`, `ACCESS_DENIED`, `PARAM_ILLEGAL`. A full list is not published.
