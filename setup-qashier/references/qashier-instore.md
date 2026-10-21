# Qashier In-Store Payment Open API (Partner Terminal API)

Use this only when a partner POS or app drives a QashierPay payment terminal. It is distilled from the official OpenAPI file `payment-instore-api.yaml` ("Payment Open API", version 1.1.0), checked 2026-10-21. It is not used for website checkout; see `qashier-api.md` for that.

## Source URLs

- `https://docs.qashier.com/api/payment-instore/`
- `https://docs.qashier.com/redocusaurus/payment-instore-api.yaml` (raw OpenAPI 3.1.1 spec)
- `https://qashier.com/my/api-integration`

## Environments

| Environment | API base |
|---|---|
| Staging | `https://api.staging.qashier.com/v2/partner/qpay` |
| Production | `https://api.qashier.com/v2/partner/qpay` |

Access is partner-only: Qashier issues the partner ID and HMAC key, enables the payment API feature, and whitelists your server IPs. Contact Qashier sales or partner manager.

## Request Rules

- Headers: `x-qashier-partner: <QASHIER_PARTNER_ID>`, `x-qashier-sign: <signature>`, and `Content-Type: application/json` for POST/PUT.
- IP whitelist: requests from IPs not on the partner whitelist get 403 "Invalid IP address".
- Stale-request check: a `timestamp` (Unix seconds) must be within 10 minutes of server time. POST/PUT read it from the body; other methods read it from the query string.

## Signature

`signature = Base64( HMAC_SHA256(key = <QASHIER_HMAC_KEY>, message = rawString) )`

The docs use crypto-js: `CryptoJS.enc.Base64.stringify(CryptoJS.HmacSHA256(rawString, hmacKey))`.

- POST/PUT: `rawString` is the exact raw JSON body string you send. Sign the string, then send that same string as the body.
- Other methods (GET): `rawString` is the query parameters sorted alphabetically by name, joined as `key=value` with `&`, no leading `?`. Example: `clientId=client123&storeId=store456&timestamp=1702500000`.

```js
import crypto from "node:crypto";
const sign = (raw, key) => crypto.createHmac("sha256", key).update(raw).digest("base64");

// POST
const body = JSON.stringify({ clientId: "<CLIENT_ID>", storeId: "<STORE_ID>", timestamp: Math.floor(Date.now() / 1000) /* ...fields */ });
const sigPost = sign(body, process.env.QASHIER_HMAC_KEY);

// GET
const params = { clientId: "<CLIENT_ID>", storeId: "<STORE_ID>", timestamp: Math.floor(Date.now() / 1000) };
const raw = Object.keys(params).sort().map((k) => `${k}=${params[k]}`).join("&");
const sigGet = sign(raw, process.env.QASHIER_HMAC_KEY);
```

Per-endpoint notes in the spec that conflict with the general rule (confirm with Qashier):

- Cancel: "Generate the signature using this string format `clientId={clientId}&paymentRecordId={paymentRecordId}`", plus `&remark={remark}` if remark is sent.
- Void: "`client={clientId}&paymentRecordId={paymentRecordId}`" (note `client`, not `clientId`; likely a typo).
- Cancel and void list `timestamp` both as a query parameter and in the body.
- Status (GET with only `timestamp` in the query): whether the path `paymentRecordId` is part of the signed string is not documented.

## Endpoint Catalogue

| Method | Path | Purpose |
|---|---|---|
| GET | `/payment-method/list` | List approved payment methods for a store |
| POST | `/initiate` | Start a payment on the terminal |
| POST | `/{paymentRecordId}/cancel` | Cancel an initiated payment |
| POST | `/{paymentRecordId}/void` | Void a payment (card only, normally) |
| GET | `/{paymentRecordId}/status` | Query payment status |
| POST (to you) | `notificationUrl` | Payment status callback |

## GET /payment-method/list

Query (all required): `clientId` string, `storeId` string (store of the terminal), `timestamp` integer.

Response `data.paymentMethods[]`:

| Field | Type | Notes |
|---|---|---|
| `name` | string | Display name. |
| `scheme` | string | Use in `/initiate`, e.g. `visa-master`. |
| `paymentMethod` | string | Use in `/initiate`, e.g. `card-payment`. |
| `country` | string | Use in `/initiate`, e.g. `SG`. |
| `voidable` | boolean | Normally only `card-payment` is voidable. |
| `enabled` | boolean | If false (deactivated or maintenance), do not use. |
| `brandImages.sm` / `.md` / `.lg` | string | Logo URLs. |

Only methods the merchant submitted and that are approved are returned.

## POST /initiate

| Field | Type | Required | Notes |
|---|---|---|---|
| `clientId` | string | yes | |
| `storeId` | string | yes | |
| `amount` | string | yes | Amount to pay. Unit and decimal format not documented (string type). |
| `currency` | string | yes | e.g. `SGD`; use `MYR` in Malaysia. |
| `country` | string | yes | e.g. `SG`. |
| `paymentMethod` | string | yes | Enum: `card-payment` (only one supported). |
| `scheme` | string | no | `visa-master` (default) or `amex`. |
| `clientReference` | string | yes | Your reference. |
| `notificationUrl` | string (uri) | no | Status callback URL; must be callable, or omit. |
| `timestamp` | integer | yes | Unix seconds, within 10 minutes. |

Response `data`: `paymentRecordId` (required), `qrString` (QR payments only), `timeoutSec` (required, e.g. 60).

## POST /{paymentRecordId}/cancel

Body: `clientId` string, `remark` string (optional), `timestamp` integer. Response CommonResponseBody.

## POST /{paymentRecordId}/void

Body: `clientId` string (required), `timestamp` integer (required). Response CommonResponseBody; void schema adds optional `data.reason`.

## GET /{paymentRecordId}/status

Query: `timestamp`. Response `data.status`: `approved`, `cancelled`, `failed`, `voided`, `refunded`; `data.cardPaymentInfo` is CardInfo.

CardInfo fields (strings): `remark` (optional), `scheme`, `merchantId`, `terminalId`, `maskedCardNumber`, `panSeq`, `cardEntryMode`, `aid`, `authCode`, `retrievalReferenceNumber`, `transactionCertificate`.

## Callback to notificationUrl

Body:

| Field | Type | Notes |
|---|---|---|
| `eventCode` | string | `AUTHORIZATION`, `TERMINAL_CANCEL`, `VOID_RESULT`. |
| `eventTimestamp` | string (date-time) | |
| `notificationBody` | object | One of the bodies below. |

- AUTHORIZATION: CardInfo fields plus `result` `approved` or `failed`.
- TERMINAL_CANCEL: `remark`.
- VOID_RESULT: `result` `voided` or `failed`, `remark`.

Respond 200. No callback signature is documented, so confirm results with `GET /{paymentRecordId}/status` before recording a sale.

## Common Response and Errors

CommonResponseBody: `success` boolean, `message` string (error message when failed), `data` object.

| Status | Message | Meaning |
|---|---|---|
| 400 | Content-Type must be application/json for POST and PUT requests | Missing or wrong Content-Type. |
| 400 | Missing signature | No `x-qashier-sign`. |
| 400 | Missing timestamp | No `timestamp` in body (POST/PUT) or query (GET). |
| 400 | Timestamp is stale or invalid (must be within the last 10 minutes) | Old or future timestamp. |
| 400 | Invalid signature | Signature mismatch. |
| 403 | Invalid IP address | IP not whitelisted. |
| 403 | Partner is expired | Partner config issue; contact Qashier. |
| 403 | Partner does not have payment API feature enabled | Partner config issue; contact Qashier. |
| 404 | Partner not found | Partner config issue; contact Qashier. |
