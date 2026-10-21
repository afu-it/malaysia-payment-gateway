# 2C2P Payment Maintenance Reference

Refund, void, settle (capture), maintenance inquiry, refund status, refund notification, recurring plan maintenance, card token maintenance, customer token maintenance and FX rate inquiry. All are server-to-server only. Entry point: `2c2p-api.md`.

Checked against the official docs on 2026-10-21.

## Endpoints

| Environment | URL |
| --- | --- |
| Sandbox | `https://demo2.2c2p.com/PaymentAction/2.0/action` |
| Production | `https://t.2c2p.com/PaymentAction/2.0/action` |
| Production (Indonesia only) | `https://pgwcore.dp.alipay.com/PaymentAction/2.0/action` |

`POST`, `Content-Type: text/plain`, body is the signed and encrypted token (see below). The card token guide cURL samples post to `https://demo2.2c2p.com/2C2PFrontend/PaymentAction/2.0/action` while the same page's header shows `/PaymentAction/2.0/action`; use the header URL and confirm with 2C2P if it fails.

One endpoint serves several XML message types: `PaymentProcessRequest` (inquiry, refund, refund status, void, settle), `RecurringMaintenanceRequest`, `MaintenanceRequest` (card token), `FxRateRequest` and `FxRateListRequest`.

Source: https://developer.2c2p.com/docs/payment-maintenance-how-it-works, https://developer.2c2p.com/docs/payment-maintenance-refund-guide, https://developer.2c2p.com/docs/payment-maintenance-card-token-guide

## Security: JWE + JWS With Keys

Key setup (once per environment):

1. Download the 2C2P public key from Merchant Portal: Account > Options > 2C2P Public Keys.
2. Generate a merchant RSA key pair (the certificate generation guide uses 2048-bit keys and produces `.pem`, `.crt`, `.der`, `.pfx`). The public key you upload must be X.509.
3. Upload the merchant public key in Merchant Portal: Account > Options > Merchant Public Keys, and set it as default.

Request:

1. Build the XML (or JSON for customer token) message.
2. Encrypt it as JWE with the 2C2P public key. Algorithm `RSA_OAEP` (the guides write "RSA-OAEP"), content encryption `A256GCM`. The PHP sample sets header `kid` to `Default`.
3. Sign the JWE compact string as JWS with the merchant private key, algorithm `PS256`.
4. POST the JWS compact string as the raw `text/plain` body.

Response:

1. Verify the JWS signature with the 2C2P public key (`PS256`). Reject on failure.
2. Decrypt the inner JWE with the merchant private key (`RSA_OAEP`, `A256GCM`).
3. Parse the XML / JSON and check `respCode` (`00` success for Payment Action; `0000` for customer token).

```text
request_body  = JWS_PS256_sign( JWE_encrypt(xml, 2c2p_public_key, alg=RSA-OAEP, enc=A256GCM), merchant_private_key )
response_xml  = JWE_decrypt( JWS_PS256_verify(response_body, 2c2p_public_key), merchant_private_key )
```

```js
// Node.js with the "jose" library (sketch). Keys loaded from secret storage, never from the repo.
import { CompactEncrypt, CompactSign, compactVerify, compactDecrypt, importX509, importPKCS8 } from "jose";
const c2pPub = await importX509(process.env.TWOC2P_PUBLIC_CERT_PEM, "RSA-OAEP");
const c2pPubSig = await importX509(process.env.TWOC2P_PUBLIC_CERT_PEM, "PS256");
const merchPriv = await importPKCS8(process.env.TWOC2P_MERCHANT_PRIVATE_KEY_PEM, "PS256");
const merchPrivDec = await importPKCS8(process.env.TWOC2P_MERCHANT_PRIVATE_KEY_PEM, "RSA-OAEP");

async function maintenanceCall(xml) {
  const jwe = await new CompactEncrypt(new TextEncoder().encode(xml))
    .setProtectedHeader({ alg: "RSA-OAEP", enc: "A256GCM", kid: "Default" }).encrypt(c2pPub);
  const jws = await new CompactSign(new TextEncoder().encode(jwe))
    .setProtectedHeader({ alg: "PS256" }).sign(merchPriv);
  const res = await fetch(process.env.TWOC2P_MAINTENANCE_URL, { method: "POST", headers: { "Content-Type": "text/plain" }, body: jws });
  const { payload } = await compactVerify(await res.text(), c2pPubSig);
  const { plaintext } = await compactDecrypt(new TextDecoder().decode(payload), merchPrivDec);
  return new TextDecoder().decode(plaintext); // XML string
}
```

One sentence in the request recipe says "RSA-OAEP-256", but the C#, Java and PHP code on the same page and the JWE + JWS guide all use RSA-OAEP (SHA-1 OAEP) with A256GCM. Use RSA-OAEP. Whether RSA-OAEP-256 is also accepted: Not documented in official docs (checked 2026-10-21).

Source: https://developer.2c2p.com/docs/reference-jwt-with-key, https://developer.2c2p.com/docs/certificate-generation-guide, https://developer.2c2p.com/recipes/prepare-request-payload-with-jwt-jws-with-keys, https://developer.2c2p.com/recipes/process-response-payload-with-jwt-jws-with-keys

## Payment Process Request (`PaymentProcessRequest`, version 4.6)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `version` | C 5 | Y | `4.6`. |
| `timeStamp` | C 22 | N | `ddMMyyHHmmss`; server time if empty. (The void sample spells it `timestamp`.) |
| `merchantID` | C 15 | Y | |
| `processType` | C 2 | Y | `I` inquiry, `R` refund, `RS` refund status, `V` void / cancel, `S` settle / capture. |
| `invoiceNo` | C 50 | Y | Original transaction invoice. |
| `actionAmount` | N 13 | C | 2-decimal amount (`25.00`). Required for refund and settle; void samples also send it. Must not exceed the original; cumulative partial refunds must not exceed the settled amount. |
| `bankCode`, `accountName`, `accountNumber` | C 10, C 50, C 50 | N | APM refunds only. Not required for Malaysia. |
| `idempotencyID` | C | N | Retry key (the refund sample also shows `idempotencyId`). Codes 50 / 51 report idempotency conflicts. |
| `subMerchantList` / `subMerchant subMID`, `subAmount` | | N | Master merchant accounts only; `subAmount` total must equal `actionAmount`. |
| `notifyURL` | C 250 | N | Refund only. 2C2P posts the final refund status there. |
| `loyaltyPayments` / `loyaltyRefund` | | N | Loyalty refund: `loyaltyProvider`, `externalMerchantId`, `totalRefundRewardAmount`, `refundRewards/reward(type, quantity)`. |
| `userDefined1` to `userDefined5` | | N | Echoed. |

Field types come from the legacy Payment Action parameter page (v3.1). The v4.6 page publishes the XML structure only.

Samples (Malaysian demo merchant `JT07`, sandbox):

```xml
<PaymentProcessRequest><version>4.6</version><merchantID>JT07</merchantID><invoiceNo>260121085327</invoiceNo><actionAmount>25.00</actionAmount><processType>R</processType></PaymentProcessRequest>
<PaymentProcessRequest><version>4.6</version><merchantID>JT07</merchantID><invoiceNo>250221094450</invoiceNo><actionAmount>105.00</actionAmount><processType>V</processType></PaymentProcessRequest>
<PaymentProcessRequest><version>4.6</version><merchantID>JT07</merchantID><processType>S</processType><invoiceNo>INV14235354</invoiceNo></PaymentProcessRequest>
<PaymentProcessRequest><version>4.6</version><merchantID>JT07</merchantID><invoiceNo>260121085327</invoiceNo><actionAmount>25.00</actionAmount><processType>RS</processType></PaymentProcessRequest>
```

## Payment Process Response (`PaymentProcessResponse`)

| Field | Notes |
| --- | --- |
| `version`, `timeStamp` | |
| `respCode` | `00` success; see result codes in `2c2p-codes.md`. |
| `respDesc` | |
| `processType`, `invoiceNo` | Echo. |
| `amount` | Paid, refunded or captured amount. |
| `status` | Transaction status code (`A`, `S`, `RS`, `RF`, `RP`, `V`, ...); see `2c2p-codes.md`. |
| `approvalCode`, `referenceNo` | Host references (`referenceNo` is the refund reference on refunds). |
| `refundReferenceNo` | Refund reference (refund responses). |
| `transactionDateTime` | `yyyyMMddHHmmss`. |
| `paidAgent`, `paidChannel` | Agent and channel used. |
| `maskedPan`, `eci` | Card data. |
| `subMerchantList` | Master merchant only. |
| `refundList/refund` | Attributes `referenceNo`, `status`, `amount`, `dateTime`, `acquirerResponseCode`, `userDefined1..5` (refund status). |
| `paymentScheme`, `processBy` | Scheme and network codes (`VI`, `MA`, ...). |
| `acquirerResponseCode` | ISO 8583 code when available. |
| `idempotencyID` | Echo. |
| `amountSettled`, `amountRefunded`, `availableAmountforCapture` | Running totals. |
| `userDefined1` to `userDefined5` | Echo. |

Source: https://developer.2c2p.com/docs/api-payment-action-payment-process, https://developer.2c2p.com/docs/api-parameters-2, https://developer.2c2p.com/docs/payment-maintenance-inquiry-guide, https://developer.2c2p.com/docs/payment-maintenance-void-guide, https://developer.2c2p.com/docs/payment-maintenance-settle-guide, https://developer.2c2p.com/docs/payment-maintenance-refund-status-guide

## Rules Per Action

- Refund: only for settled transactions. Multiple partial refunds allowed up to the settled total. Refund availability depends on bank and channel.
- Async refunds: for some 123 (FPX / online banking) payments and wallets the first response is REFUND_PENDING (`status` `RP`, or result code 42). Either poll `processType=RS` or send `notifyURL`. The notification is an HTTP POST body `notifyResponse=<token>`, signed and encrypted with JWE + JWS like a maintenance response, containing a `PaymentProcessResponse`. Verify and decrypt before trusting it.
- Void: card payments only, same day as the authorisation and before the acquirer cut-off time (see cut-off table in `2c2p-codes.md`).
- Settle: captures a pre-authorised transaction (`transactionMode` pre-auth at token time). Uncaptured pre-auths are auto-voided by the acquirer after a period (commonly 7 days). Result code 34 means a duplicate capture was detected; inquire before retrying.
- Idempotency: send `idempotencyID` and keep it per action. Do not resend a refund on timeout without an `RS` or `I` check.
- Record every maintenance call (request XML minus secrets, response, status) for audit.

Source: https://developer.2c2p.com/docs/payment-maintenance-how-it-works, https://developer.2c2p.com/docs/payment-maintenance-refund-guide, https://developer.2c2p.com/docs/response-code-payment-maintenance-result-code

## Recurring Plan Maintenance (`RecurringMaintenanceRequest`, version 2.4)

Same endpoint and JWE + JWS. Use the `recurringUniqueID` from the first payment's notification.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `version` | C 5 | Y | `2.4`. |
| `timeStamp` | C 22 | Y | `ddMMyyHHmmss`. |
| `merchantID` | N 20 | Y | |
| `recurringUniqueID` | N 20 | Y | Plan ID. |
| `processType` | C 1 | Y | `I` inquiry, `U` update, `C` cancel. |
| `recurringStatus` | C 1 | Update | `Y` active, `N` inactive. |
| `amount` | N 12 | Update | 12-digit zero-padded, 2 implied decimals (`000000009990` = 99.90). |
| `allowAccumulate` | C 1 | Update | `Y` / `N`. |
| `maxAccumulateAmount` | N 12 | Update | 12-digit zero-padded. |
| `recurringInterval` | N 5 | Update | Days, max 365. Do not combine with `chargeOnDate`. |
| `recurringCount` | N 5 | Update | `0` = endless. |
| `chargeNextDate` | C 8 | Update | `DDMMYYYY` (the response sample shows `20210307`, which is `yyyyMMdd`). |
| `chargeOnDate` | C 8 | N | `ddMM`. Do not combine with `recurringInterval`. |

Response (`RecurringMaintenanceResponse`): `version`, `timeStamp`, `merchantID`, `recurringUniqueID`, `respCode` (`00` success), `respReason`, `recurringStatus`, `invoicePrefix`, `currency` (ISO 4217 numeric, MYR `458`), `amount`, `maskedCardNo`, `allowAccumulate`, `maxAccumulateAmount`, `recurringInterval`, `recurringCount`, `currentCount`, `chargeNextDate`, `chargeOnDate`.

Legacy recurring maintenance (PKCS7 era) adds `hashValue` = HMAC-SHA256 with the Secret Key over `version + merchantID + recurringUniqueID + processType + recurringStatus + amount + allowAccumulate + maxAccumulateAmount + recurringInterval + recurringCount + chargeNextDate + chargeOnDate`, uppercase hex. Use only if 2C2P has put you on the legacy endpoint.

Source: https://developer.2c2p.com/docs/payment-maintenance-recurring-payment-guide, https://developer.2c2p.com/docs/api-payment-action-recurring-maintenance, https://developer.2c2p.com/docs/recurring-payment-plan-maintenance, https://developer.2c2p.com/docs/api-parameters-2

## Card Token Maintenance (`MaintenanceRequest`, version 2.3)

Same endpoint and JWE + JWS. Full PAN handling puts you in PCI DSS scope; prefer hosted tokenisation (`tokenize` on Payment Token) unless you are PCI certified.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `version` | C 5 | M | `2.3`. |
| `timeStamp` | C 22 | O | `ddMMyyHHmmss`. |
| `merchantID` | C 15 | M | |
| `storeCardUniqueID` | C 20 | M except Add | Token from 2C2P. |
| `pan` | C 19 | Add, Update | Full card number. |
| `panExpiry` | C 4 | Add, Update | `MMYY`. |
| `cardholderName` | C 50 | Add, Update | Allowed characters `-_,'.A-Za-z&`. |
| `cardholderEmail` | C 50 | O | |
| `panBank` | C 50 | O | Issuer name. |
| `panCountry` | C 2 | O | ISO 3166-1 alpha-2. |
| `panCurrency` | C 3 | O | Numeric currency code. |
| `action` | C 1 | M | `A` add, `U` update, `D` delete, `I` inquiry. |

Response (`MaintenanceResponse`): `version`, `timeStamp`, `merchantID`, `respCode` (`00`), `respReason`, `storeCardUniqueID`, `panBank`, `panCountry`, `cardholderName`, `cardholderEmail`, `panMasked`. Legacy versions add `hashValue` (HMAC-SHA256).

Source: https://developer.2c2p.com/docs/payment-maintenance-card-token-guide, https://developer.2c2p.com/docs/api-payment-action-store-card-maintenance, https://developer.2c2p.com/docs/api-parameters-2

## Customer Token Maintenance (JSON, JWE + JWS)

`POST https://sandbox-pgw.2c2p.com/customertoken/3.0/CustomerToken/{add|update|get|delete}` (production `https://pgw.2c2p.com/...`; the docs list the production `get` URL with the sandbox host, which looks like a doc error). `Content-Type: text/plain`.

| Operation | Request fields | Response fields |
| --- | --- | --- |
| add | `merchantID` AN 15 M, `accountNo` AN 255 O (full card or external token), `tokenProvider` C 15 M (`CC` card, `MA` Mastercard token), `name` C 50 C, `email` C 50 O, `expiry` (`yyyy-MM-dd`) C, `accountIssuer` C 50 O, `accountIssuerCountry` C 2 O, `accountCurrency` C 3 O | `responseCode` (`0000`), `responseDesc`, `merchantID`, `token`, `tokenReference`, `accountNo` (masked), `name`, `email`, `expiry`, `channelCode`, `subChannelCode`, `accountIssuer`, `accountIssuerCountry`, `accountCurrency` |
| update | `merchantID` M, `token` M, `name` M, `email` O, `expiry` M, `accountIssuer`, `accountIssuerCountry`, `accountCurrency` O (sample also sends `tokenProvider`) | as add |
| get | `merchantID` M, `token` C | `responseCode`, `responseDesc`, `merchantID`, `token`, `name`, `email`, `expiry`, `channelCode`, `subChannelCode`, `accountIssuer`, `accountIssuerCountry`, `accountCurrency` |
| delete | `merchantID` M, `token` C | `responseCode`, `responseDesc`, `merchantID`, `token` |

Response codes follow the PGW payment response code list.

Source: https://developer.2c2p.com/docs/customer-token-maintenance, https://developer.2c2p.com/docs/customer-token-maintenance-api

## FX Rate Inquiry (`FxRateRequest` / `FxRateListRequest`)

| Message | Request | Response |
| --- | --- | --- |
| FX Rate Inquiry | `version` (table says 2.2, sample 2.1), `timeStamp`, `merchantID`, `currency` (numeric), `fxProvider` O | `version`, `timeStamp`, `merchantID`, `currency`, `fxProvider`, `rateQuoteId`, `rateExpiryDate`, `fxRate` D 15,7, `baseCurrency`, `responseCode` (`00`), `respReason` |
| FX Rate List Inquiry | `version`, `timeStamp`, `merchantID`, `fxProvider` O (legacy sample adds `hashValue`) | `version`, `timeStamp`, `merchantID`, `baseCurrency`, `fxProvider`, `currencyList/currency` (`currencyCode`, `fxRate`, `rateQuoteId`, `rateExpiryDate`), `responseCode`, `respReason` |

Only relevant for multi-currency pricing. Source: https://developer.2c2p.com/docs/payment-maintenance-fx-rate-inquiry-guide, https://developer.2c2p.com/docs/api-payment-action-fx-rate-inquiry

## Legacy Versions (only if 2C2P assigned them)

- Payment Action v3.x (PKCS7): `https://demo2.2c2p.com/2C2PFrontend/PaymentActionV2/PaymentAction.aspx`, production `https://t.2c2p.com/PaymentActionV2/PaymentAction.aspx`. `hashValue` = HMAC-SHA1 with the Secret Key (per the legacy parameter table). The exact field order for this hash: Not documented in official docs (checked 2026-10-21).
- Payment Process (Base64): `https://t.2c2p.com/PaymentActionV2/PaymentProcess.aspx`.
- IPP Option inquiry (`IppOptionRequest` v2.3): `version`, `timeStamp`, `merchantID`, `hashValue` (HMAC-SHA256 over `version + merchantID`). Response lists `ippBanks` with `bankName`, `bankShortName`, `bankTerms`, `bankLogoUrl`, `bankPromoUrl`, `bins/bin`, `installmentOptions/option` (`id`, `installmentPeriod`, `merInterestRate`, `cusInterestRate`, `minAmount`, `currencyCode`, `validFrom`, `validUntil`).
- 2C2P recommends new integrations use Payment Maintenance (JWE + JWS) instead.

Source: https://developer.2c2p.com/docs/api-parameters-2, https://developer.2c2p.com/docs/payment-action-references
