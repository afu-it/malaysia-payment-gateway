# M1Pay API Reference

Use this reference for factual M1Pay API guidance. Everything here comes from the official M1Pay Integration Document at `https://www.m1pay.com.my/#/docs/introduction/overview` (an Angular app; pages are hash routes). Checked 2026-10-21. Where the docs are silent, this file says "Not documented in official docs (checked 2026-10-21)". Confirm those points with M1Pay support (contacts in `account-setup.md`) before launch.

## Source URLs

- Overview: `https://www.m1pay.com.my/#/docs/introduction/overview`
- Get Started: `https://www.m1pay.com.my/#/docs/introduction/get-started`
- Payment Service: `https://www.m1pay.com.my/#/docs/payment-service`
- Step 1 OAuth2: `https://www.m1pay.com.my/#/docs/merchant-integration/M1Pay-OAuth2`
- Step 2 Initiate Payment Request: `https://www.m1pay.com.my/#/docs/merchant-integration/Initiate-M1Pay-Payment-Request`
- Step 3 Get Transaction: `https://www.m1pay.com.my/#/docs/merchant-integration/Get-Transaction`
- Step 4 Callback URL: `https://www.m1pay.com.my/#/docs/merchant-integration/Calling-Merchant-Callback-URL`
- Appendix, Signing Data: `https://www.m1pay.com.my/#/docs/APPENDIX`
- Appendix, FPX error codes: `https://www.m1pay.com.my/#/docs/APPENDIX/error-codes-fpx`

## Integration Surfaces

- M1Pay Payment Wall: hosted page listing every channel the merchant subscribes to. Omit `channel` and `fpxBank`.
- Individual payment channels: merchant checkout shows channel logos; send `channel` (and `fpxBank` for FPX) so the customer goes straight to that channel or bank login.
- Merchant portal for keys, URLs, IP, invoicing (no-code). See `account-setup.md`.

Recurring, tokenisation, refunds, payouts and plugins: Not documented in official docs (checked 2026-10-21).

## Environments

| Item | UAT (sandbox) | Production |
| --- | --- | --- |
| OAuth2 token | `https://keycloak.m1pay.com.my/auth/realms/master/protocol/openid-connect/token` | `https://keycloak.m1pay.com.my/auth/realms/m1pay-users/protocol/openid-connect/token` |
| Create transaction | `https://gateway.m1payall.com/m1paywall/api/transaction` | `https://gateway.m1pay.com.my/wall/api/transaction` |
| Get Transaction | `https://gateway.m1payall.com/m1paywall/api/m-1-pay-transactions/{transactionId}` | `https://gateway.m1pay.com.my/wall/api/m-1-pay-transactions/{transactionId}` |
| FPX banks B2C | `https://gateway.m1payall.com/m1payfpx/api/bank-list/B2C` | `https://gateway.m1pay.com.my/fpx/api/bank-list/B2C` |
| FPX banks B2B1 | `https://gateway.m1payall.com/m1payfpx/api/bank-list/B2B1` | `https://gateway.m1pay.com.my/fpx/api/bank-list/B2B1` |
| Merchant portal | `https://m1payall.com/merchant-portal/` | `https://www.m1pay.com.my/merchant-portal/` |

Note the token host is the same for both; only the realm differs (`master` for sandbox, `m1pay-users` for production). Path prefixes differ too (`/m1paywall/` vs `/wall/`, `/m1payfpx/` vs `/fpx/`).

## Endpoint Catalogue

Every operation in the official docs:

| # | Method | Path (production host shown) | Purpose | Auth |
| --- | --- | --- | --- | --- |
| 1 | POST | `keycloak.m1pay.com.my/auth/realms/m1pay-users/protocol/openid-connect/token` | Get OAuth2 `access_token` | client_id + client_secret in body |
| 2 | POST | `gateway.m1pay.com.my/wall/api/transaction` | Initiate payment request, returns M1Pay payment URL | Bearer token + `signedData` |
| 3 | GET | `gateway.m1pay.com.my/wall/api/m-1-pay-transactions/{transactionId}` | Get transaction details and status (requery) | Bearer token |
| 4 | GET | `gateway.m1pay.com.my/fpx/api/bank-list/B2C` | FPX retail (B2C) bank list | None |
| 5 | GET | `gateway.m1pay.com.my/fpx/api/bank-list/B2B1` | FPX corporate (B2B1) bank list | None |
| 6 | POST (inbound) | Merchant Host-to-Host Callback URL | M1Pay pushes payment status to merchant | `signedData` (verify with M1Pay cert) |
| 7 | Browser | Merchant Web Redirect URL | Customer returns after payment | None (UX only) |

Not documented in official docs (checked 2026-10-21): refund, cancel, void, capture, recurring/token, payout/disbursement, settlement report API.

## Security Requirements (Overview page)

- HTTPS is required for all messages between merchant and M1Pay.
- IP address filtering: the merchant must register its server's static outgoing IP with M1Pay.
- Data message protection: request data is signed with the merchant private key (`signedData`); M1Pay validates it. M1Pay "strongly recommends" the merchant validate response messages the same way.
- The private key can be regenerated at any time in the merchant dashboard. Rotating it means updating your server key file at the same moment.
- OAuth2: merchant code is the username (`client_id`) and secret key the password (`client_secret`).

## Step 1: OAuth2 Token

```text
POST {token_url}
Content-Type: application/x-www-form-urlencoded
```

Request body:

| Parameter | M/O | Type (size) | Notes |
| --- | --- | --- | --- |
| `grant_type` | M | String (18) | Constant `client_credentials` |
| `client_id` | M | Alphanumeric | Client ID from the M1Pay Merchant Portal (docs also call it the merchant code) |
| `client_secret` | M | Alphanumeric | Client Secret from the M1Pay Merchant Portal |

Response body:

| Parameter | M/O | Type | Notes |
| --- | --- | --- | --- |
| `access_token` | M | Alphanumeric | Use in Step 2 and Step 3 |

The docs say other fields appear in the response and advise using only `access_token`. Token lifetime and refresh rules: Not documented in official docs (checked 2026-10-21). Practical approach: request a fresh token per payment or cache it briefly and retry once on 401.

## Step 2: Initiate Payment Request

```text
POST {create_transaction_url}
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer <access_token>
```

Request body:

| Parameter | M/O | Type | Notes |
| --- | --- | --- | --- |
| `merchantId` | M | Alphanumeric | Merchant code |
| `transactionAmount` | M | Numeric | Ringgit amount, format `1000` or `1000.00`, no commas. Use 2 decimals to match the signing string. |
| `transactionCurrency` | M | String | Constant `MYR` |
| `merchantOrderNo` | M | Alphanumeric | Unique merchant reference |
| `emailAddress` | M | Email | Customer email for notification |
| `phoneNumber` | M | Numeric | Customer phone |
| `productDescription` | M | Alphanumeric | Max 30 characters; allowed `A-Z`, `0-9`, `-`, `.` |
| `channel` | M (see note) | String | Channel code from the table below. Omit it (with `fpxBank`) to show the Payment Wall. |
| `fpxBank` | O | Alphanumeric | Required when `channel=ONLINE_BANKING` and you want to skip bank selection |
| `exchangeOrderNo` | O | Alphanumeric | Unique tracking value; part of the signing string |
| `skipConfirmation` | O | Boolean | Skip the M1Pay confirmation page |
| `signedData` | M | Alphanumeric | Hex SHA1withRSA signature, see Signing |

Note: the field table marks `channel` mandatory, but the same page says to omit `channel` and `fpxBank` for the Payment Wall, and that sending them with empty values returns an error. So either send a real value or leave the parameter out entirely.

Response: "Resulting M1Pay URL" carrying these parameters:

| Parameter | Notes |
| --- | --- |
| `transactionId` | Unique M1Pay transaction id; needed for Get Transaction |
| `signedData` | Used as verification key at M1Pay |

Redirect the customer's browser to that URL. The exact response body shape (plain URL vs JSON) is not spelled out; parse defensively and log the raw body during UAT. Store `transactionId` against `merchantOrderNo` before redirecting.

### FPX bank selection examples (from the docs)

```text
Retail (B2C):     channel=ONLINE_BANKING  fpxBank=Maybank2U
Corporate (B2B1): channel=ONLINE_BANKING  fpxBank=Maybank2E
UAT test bank:    fpxBank=SBI Bank A
```

The examples use the bank `title` from the bank list (not `bankId`). Confirm with M1Pay if you plan to send `bankId`.

### Payment channels

| `channel` | Description | Success status |
| --- | --- | --- |
| `ONLINE_BANKING` | FPX online banking in Malaysia | `APPROVED` |
| `CARD_PAYMENT` | Debit or credit card | `CAPTURED` |
| `UMOBILE` | UMobile | `SUCCESSFUL` |
| `EMONEI` | eM-ONEi wallet | `SUCCESSFUL` |
| `ALIPAY` | Alipay e-wallet | `SUCCESSFUL` |
| `BOOST` (status list only) | Boost | `COMPLETED` |

The prose also names Boost, TnG, Shopee and GrabPay as individual channels, but their exact `channel` codes are not in the channel table (only `BOOST` appears, in the callback success list). Get the codes for your subscribed channels from M1Pay support. DuitNow QR channel code: Not documented in official docs (checked 2026-10-21).

UAT supports only `ONLINE_BANKING` and `CARD_PAYMENT`. Test credentials come from M1Pay support.

### FPX bank list response

Public GET, no auth. Live response observed 2026-10-21 is a JSON array:

```json
[{ "id": 1, "bankId": "ABB0233", "title": "Affin Bank", "businessModel": "B2C", "fpxOnline": true }]
```

Fields: `id`, `bankId`, `title`, `businessModel` (`B2C` or `B2B1`), `fpxOnline`. Hide banks where `fpxOnline` is false. Fetch at checkout time or cache for a few minutes; do not hardcode.

## Step 3: Get Transaction (status requery)

Use after the customer returns to the result page, and any time you need to confirm status.

```text
GET {get_transaction_url}/{transactionId}
Authorization: Bearer <access_token>
```

Path parameter: `transactionId` (M, Numeric) from Step 2.

Response fields:

| Field | Type | Notes |
| --- | --- | --- |
| `merchantId` | Alphanumeric | Merchant code |
| `productDescription` | Alphanumeric | Product description |
| `transactionAmount` | Numeric | Amount |
| `transactionCurrency` | Alphabet | Currency |
| `transactionStatus` | Alphabet | Last status of the transaction (values below) |
| `merchantOrderNo` | Alphanumeric | Your order number |
| `phoneNumber` | Numeric | Customer phone |
| `emailAddress` | Email | Customer email |
| `channel` | Alphanumeric | Channel used |
| `fpxTxnId` | Numeric | FPX transaction id |
| `model` | Alphanumeric | `B2B1` or `B2C` (FPX only) |
| `failureReason` | Alphabet | E-wallet failure reason, or FPX debit authentication error (see codes) |

No signature on this response is documented. It is a server-to-server TLS call with your Bearer token, so treat it as the source of truth.

## Step 4: Host-to-Host Callback

M1Pay POSTs to the Callback URL set in the merchant portal (max 255 characters, one per merchant account). The Step 4 page says it is used when a real-time result first fails and is later updated to successful or captured; the Get Started page says M1Pay sends the payment status result there. Build for both: do not assume a callback for every transaction, and always requery on redirect.

Retries: if your server cannot receive it, M1Pay retries up to 15 times at 10-minute intervals.

```text
POST {merchant_callback_url}
Content-Type: application/x-www-form-urlencoded
```

Body fields:

| Field | Type | Notes |
| --- | --- | --- |
| `transactionAmount` | Alphanumeric | Amount |
| `merchantOrderNo` | Alphanumeric | Your order number |
| `sellerOrderNo` | Alphabet | M1Pay seller order number |
| `status` | Alphabet | Status value (below) |
| `description` | Alphanumeric | Free text |
| `signedData` | Alphanumeric | Hex signature to verify |
| `reasonStatus` | Alphabet | Reason status for e-wallet transactions |
| `fpxTxnId` | (not in table) | Used in the verification string and read by the official PHP sample (`$_POST['fpxTxnId']`) |

The descriptions in the official callback table are copy-pasted from other tables (for example `transactionAmount` is described as "Merchant code"). Rely on the field names and the verification string, not those descriptions.

Expected acknowledgement response (HTTP code or body): Not documented in official docs (checked 2026-10-21). Return HTTP 200 quickly after durable storage.

## Status Values

`REQUEST`, `APPROVED`, `ROLLBACK`, `UNSUCCESSFUL`, `PENDING`, `CANCELLED`, `FAILED`, `CAPTURED`, `SUCCESSFUL`, `COMPLETED`, `CANCEL`, `COMPLETED_ACK`, `EXPIRED`, `REFUNDED`, `REVERSED`.

Success per channel: FPX `APPROVED`; `CARD_PAYMENT` `CAPTURED`; `EMONEI`, `ALIPAY`, `UMOBILE` `SUCCESSFUL`; `BOOST` `COMPLETED`.

Meaning of `COMPLETED_ACK`, `ROLLBACK`, `REVERSED`, `REFUNDED`: Not documented in official docs (checked 2026-10-21). Treat `REFUNDED`, `REVERSED` and `ROLLBACK` as reversing a prior settlement; treat `REQUEST` and `PENDING` as not final.

## Signing (merchant to M1Pay)

Two steps: concatenate, then sign with the merchant private key file (`{merchant_code}.key`, PEM RSA key).

Signing string (Java sample, pipe separated, this exact order):

```text
productDescription|transactionAmount|exchangeOrderNo|merchantOrderNo|transactionCurrency|emailAddress|merchantId
```

- `transactionAmount` must have 2 decimal places (`10` becomes `10.00`).
- Algorithm: `SHA1withRSA` (Java, C#) or `sha1WithRSAEncryption` (PHP `openssl_sign`), which is RSA PKCS#1 v1.5 with SHA-1.
- Encoding: signature bytes as hex. Java and PHP samples emit uppercase hex; the C# sample emits lowercase. Use uppercase to match the primary samples.
- Input bytes: UTF-8 (C# sample); Java uses `getBytes()` default charset.
- Empty `exchangeOrderNo`: not documented. The PHP sample concatenates the value as is, so an omitted field becomes an empty string between pipes. Safest: always send a value (for example reuse your order number).

Node.js sketch:

```js
import { createSign } from "node:crypto";
const amount = Number(order.amount).toFixed(2);
const toSign = [desc, amount, exchangeOrderNo, merchantOrderNo, "MYR", email, merchantId].join("|");
const signedData = createSign("RSA-SHA1").update(toSign, "utf8").sign(privateKeyPem, "hex").toUpperCase();
```

PHP (from the official sample, trimmed):

```php
$raw = $p['productDescription'].'|'.$p['transactionAmount'].'|'.$p['exchangeOrderNo'].'|'
     .$p['merchantOrderNo'].'|'.$p['transactionCurrency'].'|'.$p['emailAddress'].'|'.$p['merchantId'];
$key = openssl_get_privatekey(file_get_contents('/path/{merchant_code}.key'));
openssl_sign($raw, $sig, $key, 'sha1WithRSAEncryption');
$p['signedData'] = strtoupper(bin2hex($sig));
```

## Verifying (M1Pay to merchant)

Verify callback `signedData` with M1Pay's X.509 public certificate (`public.crt` / `merchantXXXXXXXX.crt` in the samples).

Verification string:

```text
transactionAmount|fpxTxnId|sellerOrderNo|status|merchantOrderNo
```

- Use the received values exactly as posted (do not reformat the amount).
- Hex-decode `signedData` to bytes, then verify with SHA1withRSA against the certificate public key.
- The official PHP sample reads `$_POST['transactionAmount ']` with a trailing space. That is a typo; use `transactionAmount`.
- Java sample error strings: `ErrorCode :[09]` when data does not match the signature, `ErrorCode : [03]` on exception.

Node.js sketch:

```js
import { createVerify } from "node:crypto";
const s = [b.transactionAmount, b.fpxTxnId ?? "", b.sellerOrderNo, b.status, b.merchantOrderNo].join("|");
const ok = createVerify("RSA-SHA1").update(s, "utf8").verify(m1payCertPem, b.signedData, "hex");
if (!ok) return res.status(400).end();
```

PHP (official sample, fixed):

```php
$data = $_POST['transactionAmount'].'|'.$_POST['fpxTxnId'].'|'.$_POST['sellerOrderNo'].'|'
      .$_POST['status'].'|'.$_POST['merchantOrderNo'];
$pub = openssl_get_publickey(file_get_contents('/path/public.crt'));
$ok = openssl_verify($data, hex2bin($_POST['signedData']), $pub, 'sha1WithRSAEncryption') === 1;
```

`fpxTxnId` for non-FPX callbacks: not documented. Log raw callbacks in UAT and confirm with M1Pay.

## Settlement Flow

1. Local order `pending` with `merchantOrderNo`, amount, `MYR`, chosen channel.
2. Token, sign, create transaction; save `transactionId`.
3. Redirect to the M1Pay URL.
4. On return to the Web Redirect URL: call Get Transaction with the stored `transactionId`; never trust query parameters. Redirect query parameters are Not documented in official docs (checked 2026-10-21).
5. On callback: verify `signedData`, then (recommended) requery Get Transaction.
6. Mark paid only if `merchantOrderNo`, amount, currency and the per-channel success status match, inside a transaction guarded by a unique key on `transactionId`.
7. Handle later `REFUNDED`, `REVERSED`, `ROLLBACK` by reversing fulfilment.

## FPX Debit Authentication Error Codes

Returned in `failureReason` for FPX.

| Status | Code | Status | Code |
| --- | --- | --- | --- |
| APPROVED | 00 | INVALID_MERCHANT | 03 |
| INVALID_SELLER_OR_ACQUIRING_BANK | 05 | TRANSACTION_PENDING | 09 |
| INVALID_TRANSACTION | 12 | INVALID_AMOUNT | 13 |
| INVALID_BUYER_ACCOUNT | 14 | INVALID_RESPONSE | 20 |
| FORMAT_ERROR | 30 | INVALID_BANK | 31 |
| NO_CREDIT_ACCOUNT | 39 | DUPLICATE_SELLER_ORDER_NUMBER | 45 |
| INVALID_SELLER_EXCHANGE_OR_SELLER | 46 | INVALID_CURRENCY | 47 |
| MAXIMUM_TRANSACTION_LIMIT_EXCEEDED | 48 | MERCHANT_SPECIFIC_LIMIT_EXCEEDED | 49 |
| INVALID_SELLER_FOR_MERCHANT_SPECIFIC_LIMIT | 50 | INSUFFICIENT_FUNDS | 51 |
| NO_BUYER_ACCOUNT_NUMBER | 53 | TRANSACTION_NOT_PERMITTED | 57 |
| TRANSACTION_TO_MERCHANT_NOT_PERMITTED | 58 | INVALID_SERIAL_NUMBER | 70 |
| TRANSACTION_NOT_FOUND | 76 | INVALID_BUYER_NAME_OR_BUYER_ID | 77 |
| DECRYPTION_FAILED | 78 | HOST_DECLINE_WHEN_DOWN | 79 |
| BUYER_CANCEL_TRANSACTION | 80 | INVALID_TRANSACTION_MODEL | 83 |
| INVALID_TRANSACTION_TYPE | 84 | INTERNAL_ERROR_AT_BANK_SYSTEM | 85 |
| DEBIT_FAILED_EXCEPTION_HANDLING | 87 | CREDIT_FAILED_EXCEPTION_HANDLING | 88 |
| TRANSACTION_NOT_RECEIVED_EXCEPTION_HANDLING | 89 | BANK_INTERNET_BANKING_UNAVAILABLE | 90 |
| INVALID_BUYER_BANK | 92 | SYSTEM_MALFUNCTION | 96 |
| MAC_ERROR | 98 | PENDING_B2B | 99 |
| BLOCKED_BANK | BB | TRANSACTION_CANCELLED_BY_CUSTOMER | BC |
| INVALID_APPLICATION_TYPE | DA | INVALID_EMAIL_FORMAT | DB |
| INVALID_MAXIMUM_FREQUENCY | DC | INVALID_FREQUENCY_MODE | DD |
| INVALID_EXPIRY_DATE | DE | INVALID_E_MANDATE_BUYER_BANK_ID | DF |
| INTERNAL_ERROR | FE | TRANSACTION_REJECTED_NOT_IN_FPX_OPERATION_HOUR | OE |
| TRANSACTION_TIMEOUT | OF | INVALID_ACQUIRING_BANK_CODE | SB |
| INVALID_SRC_IP_ADDRESS_B2B2 | XA | INVALID_SELLER_EXCHANGE_IP | XB |
| SELLER_EXCHANGE_ENCRYPTION_ERROR | XC | INVALID_MESSAGE | XE |
| INVALID_NUMBER_OF_ORDERS | XF | INVALID_SELLER_EXCHANGE | XI |
| INVALID_FPX_TRANSACTION_MODEL | XM | TRANSACTION_REJECTED_DUPLICATE_SELLER_EXCHANGE_ORDER_NUM | XN |
| DUPLICATE_SELLER_EXCHANGE_ORDER_NUMBER | XO | SELLER_NOT_BELONG_TO_EXCHANGE | XS |
| INVALID_TRANSACTION_TYPE_2 | XT | SELLER_EXCHANGED_DATE_DIFFERENCE_EXCEEDED | XW |
| BUYER_SESSION_TIMEOUT_AT_INTERNET_BANKING_LOGIN | 1A | FAILED_PROVIDE_NECESSARY_INFO_LOGIN | 1B |
| BUYER_CHOOSE_CANCEL_AT_LOGIN | 1C | BUYER_SESSION_TIMEOUT_AT_ACCOUNT_SELECTION_PAGE | 1D |
| FAILED_PROVIDE_NECESSARY_INFO_ACCOUNT_SELECTION_PAGE | 1E | BUYER_CHOOSE_CANCEL_AT_ACCOUNT_SELECTION_PAGE | 1F |
| BUYER_SESSION_TIMEOUT_AT_TAC_REQ_PAGE | 1G | FAILED_PROVIDE_NECESSARY_INFO_TAC_REQ_PAGE | 1H |
| BUYER_CHOOSE_CANCEL_AT_TAC_REQ_PAGE | 1I | BUYER_SESSION_TIMEOUT_AT_CONFIRMATION_PAGE | 1J |
| FAILED_PROVIDE_NECESSARY_INFO_AT_CONFIRMATION_PAGE | 1K | BUYER_CHOOSE_CANCEL_AT_CONFIRMATION_PAGE | 1L |
| INTERNET_BANKING_SESSION_TIMEOUT | 1M | TRANSACTION_AMOUNT_IS_LOWER_THAN_MINIMUM | 2A |
| EXPIRED_ON_M1 | EX | | |

## Not Documented (checked 2026-10-21)

- Refund, cancel, void, capture, recurring/token, payout, settlement report APIs.
- Rate limits, token lifetime, idempotency keys, minimum/maximum amount.
- Web Redirect URL query parameters and method.
- Callback acknowledgement format.
- Channel codes for TnG, ShopeePay, GrabPay, DuitNow QR, Maybank QRPay.
- API error response format for Steps 1 to 3.
- M1Pay IP addresses for callback allowlisting.
- Public test card numbers (ask M1Pay support).
