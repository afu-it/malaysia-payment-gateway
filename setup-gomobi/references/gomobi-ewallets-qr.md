# GoMobi E-wallets, DuitNow QR and Cash

Read `gomobi-api.md` first for hosts, credentials and the AES-CBC/3DES algorithms. Checked 2026-10-21. Docs: `https://gomobi.io/documents/payin/malaysia/...`.

## Shared E-wallet Rules

- Minimum MYR 5 for Boost, GrabPay, ShopeePay, TNG.
- Expiry: Boost 5m, ShopeePay 5m, TNG 10m, GrabPay 15m.
- Amount in requests is a 12-digit sen string (`000000001000` = RM10.00).
- `checkSum` = base64 AES-CBC (password mobiKey, salt `<MOBI_SALT>`) of the minified JSON of all non-null request params, excluding `checkSum`. Omit unused optional fields entirely (no empty strings) so the checksum matches.
- Keep `invoiceId` unique per attempt.
- Common error descriptions (`responseCode` `0001`): see `gomobi-codes.md`.

### Optional POST IPN (Boost, GrabPay, ShopeePay, TNG)

Activation on request. `Content-Type: application/x-www-form-urlencoded`. Fields: `transaction_amount` (2 dp, e.g. `30.00`), `transaction_id`, `transaction_order` (your order id), `transaction_description` (status, e.g. `completed`), `transaction_code` (`00` success), `transaction_type` (e.g. `BOOST`), `transaction_signature`.

`transaction_signature` is described as an "AES-CBC encrypted hash" of `transaction_id`, `transaction_description`, `transaction_amount`, `transaction_type`. Field order and separator: Not documented in official docs (checked 2026-10-21). Confirm with Mobi; until then also confirm via the status API. Reply `HTTP 200 OK`; non-200 may cause retries.

## Boost

Docs: `ewallet/boost`, `/genrate-qr`, `/status-check`, `/ipn`.

### Generate QR

`POST /payment/mobiezywayservice/jsonservice`, header `mobiApiKey: <MOBI_API_KEY>`.

| Field | Req | Notes |
|---|---|---|
| service | Yes | `BOOST_QRCODEECOM` |
| amount | Yes | 12-digit sen |
| userName | No | business name |
| invoiceId | Yes | unique |
| redirectUrl | Yes | customer return |
| callback | Yes | HTTPS IPN/callback URL |
| subMerchantMid | No | omit if not registered |
| checkSum | Yes | see shared rules |

Response `0000 SUCCESS`, `responseData`: `base64ImageQRCode`, `url`, `trxId`, `aid` (keep for status).

### Status

Same URL, body `{"service":"BOOST_STATUSECOM","aid":"<aid>"}`. Response data: `boostRefNumber`, `transactionStatus` (`completed`, `completed-void`, `completed-refund`, `Transaction Failed`), `orderId`, `amount` (12-digit), `aidResponse`, `txnType` (`BOOST`). Pending returns `0001` "Transaction not completed with QRCode".

### IPN (GET to callback)

Query params: `merchantId`, `onlineRefNum`, `transactionType`, `amount` (e.g. `25.50`), `transactionTime` (HHMMSS), `boostRefNum`, `customerLast4DigitMSISDN`, `transactionStatus` (`completed`, `failed`, `pending`), `checksum`, `orderId`, `boostRefNumber` (duplicate of `boostRefNum`), `responseCode` (`00` success). Sent even if the customer never returns to `redirectUrl`. Check `boostRefNum` equals `boostRefNumber`. GET `checksum` verification method: Not documented in official docs (checked 2026-10-21).

## GrabPay

Docs: `ewallet/grab`, `/create-one-time-link`, `/callBack-url`, `/status`, `/ipn`.

### Create one-time link

`POST /payment/grabpay/mobi_jsonservice`, no headers.

| Field | Req | Notes |
|---|---|---|
| tid | Yes | "GPay Transaction ID from the earlier handshake/init API" (handshake not documented) |
| amount | Yes | 12-digit sen |
| sessionId | Yes | session id provided for integration |
| service | Yes | `GPAY_ONE_TIME_INIT` |
| currency | Yes | `MYR` |
| mode | Yes | `WEB` |
| callback | Yes | result post URL |
| invoiceId | Yes | unique |
| subMid | No | |
| checkSum | Yes | see shared rules |

Response: `qrcode` (Grab redirect URL), `msgID`, `txID` (store invoiceId to txID mapping). Description "One Time Init Success".

### Callback (browser form POST)

`msgID`, `txID`, `description`, `paymentMethod` (`GRABWALLET`), `txStatus` (`success`, `failed`), `reason`, `mode` (`WEB`/`APP`), `invoiceId`, `amount` (decimal), `callback`. UX only. Respond in under 3 seconds.

### Status

`POST /payment/grabpay/one-time-charge/status`, body `{"mobiApiKey":"<MOBI_API_KEY>","service":"GRAB_CHARGE_STATUS","transactionId":"<txID>"}`. Response: `amount`, `mobiTransactionId`, `callbackUrl`, `currencyType`, `status` (`SUCCESSFUL`, `PENDING`, `FAILED`, `CANCEL`), `invoiceId`. Missing id returns HTTP 400.

### IPN (GET)

`amount`, `responseCode` (`0000`), `invoiceId`, `orderId`, `msgID`, `txID`, `paymentMethod` (`GRABPAY`), `txStatus`, `mode`. Optional POST as shared rules. A callback may be retried; be idempotent.

## ShopeePay and Touch 'n Go

Docs: `ewallet/shopee/*`, `ewallet/touch-n-go/*` (`genrate-qr`, `status-check`, `callBack-url`, `ipn`).

### Generate QR

`POST /payment/mobiezywayservice/jsonservice`, no headers.

| Field | Req | Notes |
|---|---|---|
| service | Yes | `SHOPEE_QRCODEECOM` or `TNG_QRCODEECOM` |
| amount | Yes | 12-digit sen |
| username | No | |
| invoiceId | Yes | unique |
| merchantCallback | Yes | callback URL |
| motoApiKey | Yes | `<MOBI_API_KEY>` |
| subMerchantMid | No | omit if unused |
| checkSum | Yes | see shared rules |

Response: `base64ImageQRCode` (a redirect URL despite the name), `orderId`, `mobiTxnId`, `trxId` (save for status). Description "QRCode hit success".

### Status

Same URL, `{"service":"TNG_AND_SPP_STATUSECOM","trxId":"<trxId>"}`. Response: `description`, `merchantCallback`, `tngTxnId`, `txID`, `txStatus`, `paymentMethod` (`SHOPEE PAY` or `TOUCH N GO`), `amount` (12-digit), `invoiceId`.

### Callback (browser form POST to merchantCallback)

`description`, `merchantCallback`, `tngTxnId`, `txID`, `txStatus`, `paymentMethod`, `amount`, `invoiceId`. UX only.

### IPN (GET)

`amount` (12-digit), `responseCode`, `invoiceId`, `orderId`, `msgID`, `txID`, `paymentMethod`, `txStatus`, `mode`. Optional POST as shared rules.

## DuitNow QR

Docs: `qr-payments/duitnow-qr`, `/payment-api`, `/status-check`, `/ipn`. Expiry 5 minutes. Production `godnp.gomobi.io` is IP-restricted.

Headers: `mobiApiKey` = 3DES-encrypted mobiKey (see `gomobi-api.md`), `Content-Type: application/json`.

### Payment

`POST /api/v1/duitnowqr/payment`. Build plain JSON, AES-CBC encrypt it (mobiKey, salt), send `{"requestSignature":"<base64>"}`.

```json
{
  "merchantInfo": {"subMerchantMid": "<MOBI_SUB_MID>"},
  "paymentInfo": {"invoiceId": "<unique>", "amount": "100.00", "remark": "<text>", "country": "MY", "paymentMethod": "PM-101"}
}
```

Rules from validation errors: amount greater than RM1, two decimals; invoiceId required, max 30, unique; remark required; country `MY`; paymentMethod `PM-101`. A note says the invoice id "is auto-generated by Mobi and includes a fixed DNQR segment" (sample `20241217DNQR98700012`).

Response `{responseCode, responseMessage, responseSignature}`. Decrypted: `transactionId`, `invoiceId`, `base64QRImage`.

### Status

`GET /api/v1/duitnowqr/status?invoiceId=<id>`, header `mobiApiKey`. Response codes `0000`, `0001`, `0002` with `responseSignature`. Decrypted:

```json
{"merchantInfo":{"merchantName":"...","customerName":"..."},
 "paymentInfo":{"transactionId":"...","invoiceId":"...","amount":"100.00","remark":"...","country":"MY","paymentMethod":"PM-101","transactionStatus":"SUCCESS","createdDate":"2024-12-17 11:06:45"}}
```

`transactionStatus`: `SUCCESS`, `FAILED`, `PENDING`.

### IPN

POST `{responseCode, responseMessage, responseSignature}`; decrypt with AES-CBC; same decrypted shape as status. "IPNs are sent only once per transaction": log every IPN, dedupe on `transactionId`, and requery status if none arrives.

Errors `E001` to `E022`: see `gomobi-codes.md`.

## Cash Over Counter (EZYCASH, 7-Eleven)

Docs: `cash-over-counter`, `/payment-api`, `/status-check`, `/ipn`. Payment code valid 48 hours. Header `mobiApiKey` (3DES).

### Payment

`POST /api/v1/ezycash/payment`, body `{"requestSignature":"<AES-CBC of plain JSON>"}`.

```json
{
  "merchantInfo": {"customerName": "<name>", "subMerchantMid": "<MOBI_SUB_MID>"},
  "paymentInfo": {"invoiceId": "<unique, max 30>", "amount": "100.00", "remark": "<max 20>", "country": "MY", "paymentMethod": "PM-103", "redirectURL": "https://..."}
}
```

One note says invoiceId max 20; validation says 30. Use 20 or fewer to be safe.

Decrypted response: `transactionId`, `invoiceId`, `txnData` = {`requestURL`, `requestMethod` (`POST`), `requestType` (`REDIRECT`), `requestData` = {`order_currency`, `order_title`, `order_pincode`, `order_barcode_type`, `order_txn_num`, `payTxnID`, `order_txn_amt`, `order_name`, `order_email`, `order_ctc_num`, `order_txn_datetime`, `order_num`, `order_desc`, `order_txn_due_datetime`, `order_channel` (`Cash-711`)}}. `requestType` `REDIRECT` with `requestMethod` `POST` implies a browser form post of `requestData` to `requestURL` (the sandbox sample points to a third-party cash processor host); the exact step is Not documented in official docs (checked 2026-10-21). The sample uses `order_barcode_type` `EAN-13`.

### Status

`GET /api/v1/ezycash/status?invoiceId=<id>`, header `mobiApiKey`. Codes `0000`, `0001`, `0002`; decrypted shape as DuitNow (paymentMethod `PM-103`).

### IPN

POST `{responseCode, responseMessage, responseSignature}`; decrypt AES-CBC. Retries exist for failed deliveries; be idempotent.
