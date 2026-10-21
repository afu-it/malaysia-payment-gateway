# PayNet RPP DuitNow v3 Reference (participant APIs)

RPP (Real-time Retail Payments Platform) v3 APIs are for RPP participants: banks, e-wallets and licensed non-bank acquirers. A merchant cannot call them. This file covers the acquirer (crediting agent) side used to accept DuitNow payments, plus the shared JWS security. Field tables are generated from PayNet's published OpenAPI specs (`https://docs.paynet.my/api-reference/*`, v3.0.0). Checked 2026-10-21.

Availability warning from the specs: DuitNow Online Banking Acquirer and Consumer-Presented QR Acquirer are marked "specification review stage and not yet available to the general public"; DuitNow Request, AutoDebit and Consent are "beta stage"; Reversal, QR MPM and Network Administration say test access "will only be granted upon approval from PayNet". Contact `commercial@paynet.my` before development.

## Hosts

See `paynet-api.md` (Environments and hosts). Request-to-Pay workflow APIs use `p1ice.rpp.uat.inet.paynet.my:8443` / `ice.rpp.prod.inet.paynet.my:8443`; Consent and AutoDebit use `api.rpp.nextswitch.uat.inet.paynet.my` / `api.rpp.nextswitch.prod.inet.paynet.my`; QR (Phase-1) uses `api.uat.inet.paynet.my` / `api.prod.inet.paynet.my`. MPLS and TCP/IP options exist. TCP/IP needs a PayNet client connector, 4-byte length prefix per message, persistent connections, 4 ports per FI connecting round-robin to 4 RPP servers (`/docs/online-banking-wallet/communication-protocol`). TLS 1.2 only.

## HTTP headers

| Header | Direction | Req | Notes |
|---|---|---|---|
| `Authorization` | both | Y | `Bearer <JWS>` (see below) |
| `x-client-id` | both | Y | BIC of the participant (PayNet Client Id on webhooks) |
| `x-business-message-id` | both | Y | Same value as `appHeader.businessMessageId` |
| `x-gps-coordinates` | request | Y on intents/initiate, optional elsewhere | Sender location in decimal degrees |
| `x-ip-address` | request | Y on intents/initiate, optional elsewhere | IPv4 or IPv6 where the transaction started |
| `x-api-version` | request | N | Target version; latest if omitted |
| `x-deprecated`, `x-deprecation-date`, `x-sunset-date` | response | N | Deprecation signals |

`/docs/online-banking-wallet/construct-header` also lists security headers for "all incoming requests": `Content-Security-Policy: script-src 'self' ajax.cloudflare.com`, `Strict-Transport-Security: max-age=1000`, `X-Xss-Protection: 1; mode=block`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Custom-Header: Success`.

## Business Message Identifier

Format `YYYYMMDDBBBBBBBBXXXOCCSSSSSSSS` (max 35):

- `YYYYMMDD` date, `BBBBBBBB` first 8 characters of your BIC, `XXX` transaction code (3 digits), `O` originator (`O` OFI, `H` RPP, `R` RFI), `CC` channel, `SSSSSSSS` 8-digit serial.
- Channels: `RB` retail internet banking, `CB` corporate internet banking, `RM` retail mobile, `CM` corporate mobile, `OT` other, `BW` business web, `BA` business app (both for FI-initiated or merchant-initiated merchant transactions), `QR` DuitNow QR only.
- `endToEndId` uses the same format. `transactionId` links legs (for 060/070 it is the `businessMessageId` of the original intent).
- `creationDateTime` is Malaysian local time with offset, `YYYY-MM-DDThh:mm:ss.sss+08:00`. Error U305 fires when the BizMsgId date does not match `creationDateTime` for QR.

## JWS message signature (`/docs/online-banking-wallet/message-signature`)

Request:

1. Minify the JSON body and compute SHA-256 hex. For an empty body use SHA-256 of the empty string.
2. Header `{"alg": "RS512", "typ": "JWT"}` per the text. The page's Java, Python, PHP and Go samples use `RS256` and add `"kid": "<certificate serial number>"`. Confirm the algorithm with PayNet; the OpenAPI example token header decodes to `RS512`.
3. Claims:

| Claim | Value |
|---|---|
| `iss` | Your BIC |
| `iat` | Issued-at (Unix seconds) |
| `exp` | Must be within 1 hour of `iat` (samples use `iat + 900`) |
| `key` | JWT credential key assigned at onboarding |
| `jti` | `businessMessageId` |
| `ds` | SHA-256 hex of the minified request body |

4. Sign `base64url(header) + "." + base64url(claims)` with your private key; token = that string + `.` + base64url(signature).
5. Send `Authorization: Bearer <token>`.

Response verification: read the JWS from the response `Authorization` header, decode the header (`alg`, `typ`, `kid` = PayNet certificate serial number), verify the signature with that PayNet certificate, then recompute SHA-256 of the minified response body and compare with `ds`.

Docs sample: body `{"data":{"businessMessageId":"20230412BOEEMYK1000ORB00000001","clientMessage":"Client hello"}}` gives `ds` `8fc1f5ed05596aa2952e68ac221f31ee8a87641315c7b091f0bd41266d380739`.

```text
body_min = json_minify(body)              # same bytes you send
ds = sha256_hex(body_min)                 # sha256_hex("") if no body
header = {alg: "RS256" or "RS512" as agreed, typ: "JWT", kid: CERT_SERIAL}
claims = {iss: BIC, iat: now, exp: now + 900, key: JWT_KEY, jti: business_message_id, ds: ds}
signing_input = b64url(json(header)) + "." + b64url(json(claims))
token = signing_input + "." + b64url(RSA_PKCS1v15_sign(hash_for(alg), private_key, signing_input))
headers: Authorization: Bearer token, x-client-id: BIC, x-business-message-id: business_message_id
```

Certificates: centralised certificate setup and renewal for DuitNow v3 is described under `/docs/getting-started/centralised-configuration/duitnow-v3/*` and `/docs/certificate-management/*` (PayNet-signed or authorised-CA-signed, key rotation, modulus extraction). Older XML-signature flavours of the same services (enveloped signature, exclusive canonicalisation, RSA-SHA256) appear on old-site pages; use JWS for v3.

## Common response blocks

Success responses (200/201) carry `appHeader` (with `originalBusinessMessageId`), optional `data`, and `resp{status, reason{name, code, description, details, additionalCode}}`. `status` is `ACTC`, `ACSP` or `RJCT`; `reason.code` is a U/P/E/D code from `paynet-codes.md`.

Error responses (400) carry `appHeader{originalBusinessMessageId, rejectionDateTime}` and `resp{status: "RJCT", reason{name, code, description, details (copy of the rejected message), additionalCode, errorLocation}}`. Reject reason codes `0001` to `0062` are in `paynet-codes.md`.

When your webhook receives a request, answer with the documented 200 body: `appHeader` with a `businessMessageId`, `originalBusinessMessageId` = the request's `businessMessageId`, the request's `endToEndId` and `transactionId`, plus `resp.status` and `resp.reason` (the spec example for acceptance is `ACSP` with `U000`). Which reject codes an acquirer should return for each failure is not spelled out per webhook in the specs; the end-to-end return code tables in `paynet-codes.md` are the closest guide.

## Repeat handling and store-and-forward

- If you got no response, resend the same request with `appHeader.possibleDuplicate=true` (`PssblDplct` in ISO terms). RPP returns the original status if it has the transaction, or drops the message if it never arrived; then use transaction enquiry (`/docs/online-banking-wallet/repeat-handling`).
- `copyDuplicate`: OFI sends `CODU` on the first attempt and `COPY` on retries; RPP uses `COPY` for SAF resends and `EXPY` for expired retries.
- SAF: RPP resends selected transactions to the RFI up to a retry limit on timeout. The RFI cannot reject SAF transactions. After max retries the transaction stays in SAF as timeout until cleared manually (`/docs/online-banking-wallet/store-and-forward`). Your webhook must treat SAF resends as duplicates of the same `endToEndId`.
- Reason `U002` means "Success/ Transaction Accepted Stored in SAF" (status `ACTC`).

## Validations PayNet recommends (`/docs/online-banking-wallet/validations`)

Message: format, field type, length, signature. Business: mandatory and conditional fields, BizMsgId, date tolerance, transaction code, timeout, issuer and acquirer checks, repeat handling, min and max amount, accepted source of funds, JomPAY Ref-1 check digit. Show the customer beneficiary name, amount, recipient reference, payment description and merchant name.

## Transaction enquiry

`POST /v3/transactions/enquire` (630) queries one transaction by the original `businessMessageId` (`appHeader.transactionId`) with `debtorAgent.id` and `creditorAgent.id`; it returns `data.transaction.status` and `reason`. Published in the CPM QR acquirer, MPM QR issuer and Transfer issuer specs.

## Acquirer flows

### DuitNow Online Banking/Wallets and DuitNow Request: receiving the credit

PayNet calls `POST /webhooks/v3/payments/transfer-ct` on the acquirer for codes 010 (P2A), 110 (pay-to-proxy), 060 (DuitNow Request credit) and 070 (DOBW credit). The acquirer validates the creditor account and returns `ACSP`/`U000` with `data.creditorAccount`. For 060/070, `debtorAccount.id` is encrypted (up to 256 chars) and `appHeader.transactionId` is the `businessMessageId` of the original intent (861/851/853); use it to find the merchant order.

### Refund intent (854)

PayNet calls `POST /webhooks/v3/intents/reverse/initiate` on the merchant's acquirer bank for a "Request-to-Refund" from a merchant. `appHeader.transactionId` is the original request's `businessMessageId` (861 for DuitNow Pay-initiated, 070 for RPP back office-initiated); `creditorAccount.id` is the encrypted creditor account number (the payer receiving the refund).

### DuitNow Request (851/853)

The acquirer calls `POST /v3/intents/initiate`. Requires `x-gps-coordinates` and `x-ip-address`. `merchant.id` and `merchant.productId` are mandatory when a merchant initiates or when consent registration is bundled. PayNet notifies rejection (852) at `/webhooks/v3/intents/reject`; the credit arrives at `/webhooks/v3/payments/transfer-ct` with code 060.

### Reversal (011, 012)

`POST /v3/payments/reverse` returns funds of a completed credit transfer; the debtor is the original creditor. 011 (clear) covers 010, 110, 030, 040; 012 (masked) covers 060, 070, 210, 220, 230.

### AutoDebit (871, 210, 230, 080)

Pre-authorisation `POST /v3/auto-debit/pre-auth` (871), real-time debit `POST /v3/payments/debit-ct` (210 from bank, 230 pre-auth), and PayNet calls `POST /webhooks/v3/auto-debit/direct-debit` (080) on the acquirer.

### Consent

Register `POST /v3/consents/register`, enquire `/v3/consents/enquire` (751/752) and `/v3/consents/enquire-reg` (812), update `/v3/consents/{consentId}/status-updates`, terminate `/v3/consents/{consentId}/terminate`. PayNet webhooks: `/webhooks/v3/consents/register`, `/webhooks/v3/consents/status-updates`, `/webhooks/v3/consents/reject` (804).

### Network administration

`POST /v3/admin/sign-on`, `/v3/admin/sign-off`, `/v3/admin/echo` (code 000) and the PayNet webhook `/webhooks/v3/admin/event` (`eventCode` such as 2001 bank availability, 2003 liquidity limit breached, 2004 settlement notification).

## Field tables

Repeated blocks: `appHeader.businessMessageId` and `endToEndId` follow the Business Message Identifier format above; `resp.reason.*` is the common response block.


### Credit Transfer webhook `POST /webhooks/v3/payments/transfer-ct` (010, 110, 060, 070)

#### Request body (PayNet to acquirer)

| Field | Type | Req | Notes |
|---|---|---|---|
| `settlementCycleNumber` | string | Y | max 35 ; Settlement cycle number set by RPP |
| `interbankSettlementDate` | string | Y | Settlement date of the transaction. Expressed in Malaysian local date (YYYY-MM-DD). |
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.transactionId` | string | Y | max 35 ; Transaction Identifier. For 010 and 110, use the businessMessageId of this request. For 060 and 070, use the businessMessageId of Initiate Intent request. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.possibleDuplicate` | boolean |  | Indicates a possible duplicate of a request that was sent in an earlier attempt. To be set to TRUE when re-sending a request to PayNet. |
| `appHeader.copyDuplicate` | string |  | max 4 ; enum: CODU, COPY, EXPY ; OFI uses CODU (copy of a duplicate) during the first attempt of a request, and uses COPY (copy) when retrying the same request. RPP uses COPY to indicate a SAF (Store-and-Forward) and EXPY (expired retry) to indicate a retry reque... |
| `interbankSettlementAmount` | object | Y | Amount of the transaction |
| `interbankSettlementAmount.value` | number | Y | Value of the transaction in MYR, supports up to 2 decimal places. Special handling for Cross Border transaction: For static Cross Border QR, value is calculated by converting the foreign currency amount using FX rate in Cross Bord... |
| `interbankSettlementAmount.currency` | string |  | max 3 ; Currency of the transaction. Default to MYR. |
| `debtor` | object | Y | Debtor information |
| `debtor.name` | string | Y | max 140 ; Name of debtor / sender |
| `debtor.id` | string | Y | max 35 ; Debtor Identifier refers to the ID used to register the customer. For details, please refer to the list of identification types provided here. |
| `debtor.type` | string |  | max 35 ; enum: RET, COR ; Indicates if debtor is Corporate or Retail |
| `debtorAccount` | object | Y | Debtor account information |
| `debtorAccount.id` | string | Y | max 256 ; Debtor account number; up to 34 characters for 010 and 110 for clear account number; up to 256 characters for 060 and 070 for encrypted account number |
| `debtorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, LOAN, DEFAULT ; Debtor account type |
| `debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `debtorAgent` | object | Y |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI |
| `creditor` | object | Y | Information of the creditor, who is the recipient of funds. |
| `creditor.name` | string | Y | max 140 ; Name of the creditor. For 110, use `RPP` as the creditor name. |
| `creditor.type` | string |  | max 35 ; enum: RET, COR ; Indicates if creditor is Corporate or Retail |
| `creditor.id` | string |  | max 20 ; Required if `secondaryValidationIndicator` is `true`, this is the creditor ID number. |
| `creditor.idType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Required if `secondaryValidationIndicator` is `true`, this describes the creditor ID type. |
| `creditorAccount` | object | Y | Information of the receiving account |
| `creditorAccount.id` | string | Y | max 34 ; Creditor Account number. |
| `creditorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, LOAN, DEFAULT, PROXY ; Creditor Account type. |
| `creditorAgent` | object | Y | Information of the receiving FI. |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the receiving FI |
| `recipientReference` | string | Y | max 140 ; Reference note for creditor. For allowable special character details, please refer to the list provided here |
| `paymentDescription` | string |  | max 140 ; Description for the transaction. For allowable special character details, please refer to the list provided here |
| `secondaryValidationIndicator` | boolean |  | If `true`, the creditor account number will be validated against the creditor ID number. |
| `acceptedSourceOfFunds` | array |  | Accepted sources of fund or line of credit ; items enum: CASA, CREDIT_CARD, WALLET |
| `proxyLookupReference` | string |  | max 35 ; Required for Pay-to-Proxy (110), use the BusinessMessageId of Proxy Enquiry request. |

#### Response body the acquirer returns (200)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Message Identifier |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.originalBusinessMessageId` | string | Y | max 35 ; Message Identifier from the request. |
| `appHeader.transactionId` | string | Y | max 35 ; Transaction Identifier from the request. |
| `data` | object | Y |  |
| `data.creditorAccount` | object |  |  |
| `data.creditorAccount.id` | string | Y | max 34 ; Creditor Account number. |
| `data.creditorAccount.type` |  | Y | enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, LOAN, DEFAULT |
| `data.creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `data.creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `data.creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `data.creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `data.creditorAccount.customerCategory` | string |  | max 20 ; enum: RET, COR, OTH ; Customer category returned by the RFI |
| `resp` | object | Y |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, ACSP, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |
| `resp.reason.name` | string | Y | max 120 ; Name of the corresponding response code |
| `resp.reason.code` | string | Y | max 20 ; Response code of the request |
| `resp.reason.description` | string | Y | max 2000 ; Description of the response code |
| `resp.reason.details` | string |  | max 2000 ; Additional information related to the status of the request |
| `resp.reason.additionalCode` | string |  | max 20 ; Additional response code for PayNet troubleshooting |


### Refund Intent webhook `POST /webhooks/v3/intents/reverse/initiate` (854)

#### Request body (PayNet to acquirer)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.transactionId` | string | Y | max 35 ; The `businessMessageId` of the original Request; DN Pay initiated refund intent - 861, RPP BO initiated refund intent - 070. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.possibleDuplicate` | boolean |  | Indicates a possible duplicate of a request that was sent in an earlier attempt. To be set to TRUE when re-sending a request to PayNet. |
| `appHeader.copyDuplicate` | string |  | max 4 ; enum: CODU, COPY, EXPY ; OFI uses CODU (copy of a duplicate) during the first attempt of a request, and uses COPY (copy) when retrying the same request. RPP uses COPY to indicate a SAF (Store-and-Forward) and EXPY (expired retry) to indicate a retry reque... |
| `initiator` | object | Y | Information about the Originating Party who initiated the request. |
| `initiator.name` | string | Y | max 140 ; Name of originating party |
| `interbankSettlementAmount` | object | Y | The amount requested to be paid by the Debtor to the Creditor. |
| `interbankSettlementAmount.value` | number | Y | Value of the transaction in MYR, supports up to 2 decimal places. Special handling for Cross Border transaction: For static Cross Border QR, value is calculated by converting the foreign currency amount using FX rate in Cross Bord... |
| `interbankSettlementAmount.currency` | string |  | max 3 ; Currency of the transaction. Default to MYR. |
| `requestedExecutionDateTime` | string | Y | Date when payment is expected to be executed/refused by the debtor. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `expiryDateTime` | string | Y | Date when the intent expires. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `debtor` | object | Y | Debtor information |
| `debtor.name` | string | Y | max 140 ; Name of debtor |
| `debtor.type` | string |  | max 35 ; enum: RET, COR ; Indicates if debtor is Corporate or Retail |
| `debtor.id` | string |  | max 140 ; Debtor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 Cou... |
| `debtor.idType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor ID type. |
| `debtorAgent` | object | Y |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the Debiting FI. |
| `debtorAccount` | object | Y | Debtor account information |
| `debtorAccount.name` | string |  | max 70 ; Debtor account name |
| `debtorAccount.id` | string | Y | max 35 ; Debtor account number. For Request-to-Pay by Proxy (851), use debtor's proxy registration Id. For Request-to-Pay by Account (853) and Refund (854), use debtor's account number. |
| `debtorAccount.type` | string |  | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, PROXY ; Debtor account type. For Request-to-Pay by Proxy (851), use 'PROXY' For Request-to-Pay by Account (853) and Refund (854), use debtor's account type. |
| `debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `creditor` | object | Y | Information of the creditor, who is the recipient of funds. |
| `creditor.name` | string | Y | max 140 ; Name of the creditor. |
| `creditor.type` | string |  | max 35 ; enum: RET, COR ; Indicates if creditor is Corporate or Retail |
| `creditor.id` | string |  | max 140 ; Creditor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 C... |
| `creditor.idType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor ID type. |
| `creditor.secondaryId` | string |  | max 140 ; Creditor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ... |
| `creditor.secondaryIdType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor Secondary ID type. |
| `creditorAgent` | object | Y |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the Crediting FI. |
| `creditorAccount` | object | Y | Creditor account information |
| `creditorAccount.name` | string |  | max 70 ; Creditor account name. |
| `creditorAccount.id` | string | Y | max 256 ; Creditor account number. For Refund (854), use the encrypted creditor account number. |
| `creditorAccount.type` | string |  | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET ; Creditor account type. |
| `creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Creditor's resident status |
| `creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `firstPartyPaymentValidation` | string |  | max 20 ; enum: NO_VALIDATION, DEBTOR_NAME, DEBTOR_ID, DEBTOR_NAME_ID ; Indicates if Debiting FI should perform validation on debtor information. |
| `amountNotEditable` | boolean |  | Indicates if the amount in the Request-to-Pay can be changed by the Debtor. To be set to TRUE if the amount cannot be changed. |
| `acceptedSourceOfFunds` | array | Y | Accepted sources of fund or line of credit. ; items enum: CASA, CREDIT_CARD, WALLET |
| `recipientReference` | string | Y | max 140 ; Reference note for creditor. |
| `paymentDescription` | string |  | max 140 ; Description for the transaction. |
| `merchant` | object |  |  |
| `merchant.msicCode` | string |  | Merchant MSIC code, provided by PayNet. |

#### Response body the acquirer returns (200)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.transactionId` | string | Y | max 35 ; The `businessMessageId` of the original Request; DN Pay initiated refund intent - 861, RPP BO initiated refund intent - 070. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Message Identifier |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.originalBusinessMessageId` | string | Y | max 35 ; Message Identifier from the request. |
| `data` | object | Y |  |
| `data.initiator` | object | Y | Information about the Originating Party who initiated the request. |
| `data.initiator.name` | string | Y | max 140 ; Name of originating party |
| `resp` | object | Y |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, ACSP, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |


### Initiate an Intent `POST /v3/intents/initiate` (851, 853)

#### Request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.transactionId` | string | Y | max 35 ; The `businessMessageId` of the original Request; DN Pay initiated refund intent - 861, RPP BO initiated refund intent - 070. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.possibleDuplicate` | boolean |  | Indicates a possible duplicate of a request that was sent in an earlier attempt. To be set to TRUE when re-sending a request to PayNet. |
| `appHeader.copyDuplicate` | string |  | max 4 ; enum: CODU, COPY, EXPY ; OFI uses CODU (copy of a duplicate) during the first attempt of a request, and uses COPY (copy) when retrying the same request. RPP uses COPY to indicate a SAF (Store-and-Forward) and EXPY (expired retry) to indicate a retry reque... |
| `initiator` | object | Y | Information about the Originating Party who initiated the request. |
| `initiator.name` | string | Y | max 140 ; Name of originating party |
| `interbankSettlementAmount` | object | Y | The amount requested to be paid by the Debtor to the Creditor. |
| `interbankSettlementAmount.value` | number | Y | Value of the transaction in MYR, supports up to 2 decimal places. Special handling for Cross Border transaction: For static Cross Border QR, value is calculated by converting the foreign currency amount using FX rate in Cross Bord... |
| `interbankSettlementAmount.currency` | string |  | max 3 ; Currency of the transaction. Default to MYR. |
| `requestedExecutionDateTime` | string | Y | Date when payment is expected to be executed/refused by the debtor. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `expiryDateTime` | string | Y | Date when the intent expires. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `debtor` | object | Y | Debtor information |
| `debtor.name` | string | Y | max 140 ; Name of debtor |
| `debtor.type` | string |  | max 35 ; enum: RET, COR ; Indicates if debtor is Corporate or Retail |
| `debtor.id` | string |  | max 140 ; Debtor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 Cou... |
| `debtor.idType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor ID type. |
| `debtorAgent` | object | Y |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the Debiting FI. |
| `debtorAccount` | object | Y | Debtor account information |
| `debtorAccount.name` | string |  | max 70 ; Debtor account name |
| `debtorAccount.id` | string | Y | max 35 ; Debtor account number. For Request-to-Pay by Proxy (851), use debtor's proxy registration Id. For Request-to-Pay by Account (853) and Refund (854), use debtor's account number. |
| `debtorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, PROXY ; Debtor account type. For Request-to-Pay by Proxy (851), use 'PROXY' For Request-to-Pay by Account (853) and Refund (854), use debtor's account type. |
| `debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `creditor` | object | Y | Information of the creditor, who is the recipient of funds. |
| `creditor.name` | string | Y | max 140 ; Name of the creditor. |
| `creditor.type` | string | Y | max 35 ; enum: RET, COR ; Indicates if creditor is Corporate or Retail |
| `creditor.id` | string |  | max 140 ; Creditor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 C... |
| `creditor.idType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor ID type. |
| `creditor.secondaryId` | string |  | max 140 ; Creditor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ... |
| `creditor.secondaryIdType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor Secondary ID type. |
| `creditorAgent` | object | Y |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the Crediting FI. |
| `creditorAccount` | object | Y | Creditor account information |
| `creditorAccount.name` | string |  | max 70 ; Creditor account name. |
| `creditorAccount.id` | string | Y | max 256 ; Creditor account number. For Refund (854), use the encrypted creditor account number. |
| `creditorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET ; Creditor account type. |
| `creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Creditor's resident status |
| `creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `firstPartyPaymentValidation` | string |  | max 20 ; enum: NO_VALIDATION, DEBTOR_NAME, DEBTOR_ID, DEBTOR_NAME_ID ; Indicates if Debiting FI should perform validation on debtor information. |
| `amountNotEditable` | boolean |  | Indicates if the amount in the Request-to-Pay can be changed by the Debtor. To be set to TRUE if the amount cannot be changed. |
| `acceptedSourceOfFunds` | array | Y | Accepted sources of fund or line of credit. ; items enum: CASA, CREDIT_CARD, WALLET |
| `recipientReference` | string | Y | max 140 ; Reference note for creditor. |
| `paymentDescription` | string |  | max 140 ; Description for the transaction. |
| `merchant` | object |  |  |
| `merchant.id` | string |  | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. Mandatory if Request-to-Pay is initiated by a Merchant or if Consent Registration is bundled with Request-to-Pay. |
| `merchant.productId` | string |  | max 35 ; Product identification assigned by PayNet during merchant registration. Mandatory if Request-to-Pay is initiated by a Merchant or if Consent Registration is bundled with Request-to-Pay. |

#### Response body (201)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.transactionId` | string | Y | max 35 ; The `businessMessageId` of the original Request; DN Pay initiated refund intent - 861, RPP BO initiated refund intent - 070. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Message Identifier |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.originalBusinessMessageId` | string | Y | max 35 ; Message Identifier from the request. |
| `data` | object | Y |  |
| `data.initiator` | object | Y | Information about the Originating Party who initiated the request. |
| `data.initiator.name` | string | Y | max 140 ; Name of originating party |
| `resp` | object | Y |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |

#### Reject Intent webhook request body (852)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. In this 852 Reject Intent, it is end to end ID of the Request-To-Pay that is being updated. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `debtorAgent` | object | Y |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the debiting FI. |
| `creditorAgent` | object | Y |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the crediting FI. |
| `creditorAccount` | object |  |  |
| `creditorAccount.id` | string |  | max 35 ; Creditor account number. |
| `blockFutureRequests` | boolean |  | Indicate that the debtor intends to block future Request-to-Pay. |
| `merchant` | object |  |  |
| `merchant.id` | string |  | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. |
| `merchant.productId` | string |  | max 35 ; Product identification assigned by PayNet during merchant registration. |


### Reverse a Credit Transfer `POST /v3/payments/reverse` (011, 012)

#### Request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.transactionId` | string | Y | max 35 ; The `businessMessageId` of the original Credit Transfer from the originating Issuer to the Acquirer. Example for 011, use the `businessMessageId` of the original 010 or 011 transaction. Refer to the list of supported Credit Transf... |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.possibleDuplicate` | boolean |  | Indicates a possible duplicate of a request that was sent in an earlier attempt. To be set to TRUE when re-sending a request to PayNet. |
| `appHeader.copyDuplicate` | string |  | max 4 ; enum: CODU, COPY, EXPY ; OFI uses CODU (copy of a duplicate) during the first attempt of a request, and uses COPY (copy) when retrying the same request. RPP uses COPY to indicate a SAF (Store-and-Forward) and EXPY (expired retry) to indicate a retry reque... |
| `interbankSettlementAmount` | object | Y | Amount of the transaction |
| `interbankSettlementAmount.value` | number | Y | Value of the transaction in MYR, supports up to 2 decimal places. Special handling for Cross Border transaction: For static Cross Border QR, value is calculated by converting the foreign currency amount using FX rate in Cross Bord... |
| `interbankSettlementAmount.currency` | string |  | max 3 ; Currency of the transaction. Default to MYR. |
| `debtor` | object | Y | Debtor information. This is also the creditor information in the original Credit Transfer request. |
| `debtor.name` | string | Y | max 140 ; Name of the debtor, which is the name of the creditor in the original Credit Transfer request. |
| `debtor.type` | string |  | max 35 ; enum: RET, COR ; Indicates if debtor is Corporate or Retail. Default to “RET” when no value is specified. |
| `debtorAccount` | object | Y | Debtor account information. This is also the creditor account information in the original Credit Transfer request. |
| `debtorAccount.id` | string | Y | max 34 ; Debtor account number, which is the creditor account number in the original Credit Transfer request. |
| `debtorAccount.type` | string | Y | max 20 ; enum: DEFAULT, CURRENT, SAVINGS, CREDIT_CARD, WALLET, LOAN ; Debtor account type, which is the creditor account type in the original Credit Transfer request. |
| `debtorAgent` | object | Y |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI, which is the receiving FI in the original Credit Transfer request. |
| `creditor` | object | Y | Information of the creditor, who is the recipient of funds. This is also the debtor information in the original Credit Transfer request. |
| `creditor.name` | string | Y | max 140 ; Name of the creditor, which is the name of the debtor in the original Credit Transfer request. |
| `creditor.type` | string |  | max 35 ; enum: RET, COR ; Indicates if creditor is Corporate or Retail. Default to “RET” when no value is specified. |
| `creditorAccount` | object | Y | Information of the receiving account. This is also the debtor account in the original Credit Transfer request. |
| `creditorAccount.id` | string | Y | max 256 ; Receiving account information, same as the debtor account number in the original Credit Transfer request. Supports up to 34 characters for 011 for clear account number and up to 256 characters for 012 for encrypted account number. |
| `creditorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, LOAN, DEFAULT, PROXY ; Creditor account type, which is the debtor account type in the original Credit Transfer request. |
| `creditorAgent` | object | Y |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the receiving FI, which is the originating FI in the original Credit Transfer request. |
| `paymentDescription` | string |  | max 140 ; Description for the transaction. For allowable special character details, please refer to the list provided here |
| `acceptedSourceOfFunds` | array |  | Accepted sources of fund or line of credit. ; items enum: CASA, CREDIT_CARD, WALLET |

#### Response body (200)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Message Identifier |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.originalBusinessMessageId` | string | Y | max 35 ; Message Identifier from the request. |
| `appHeader.transactionId` | string | Y | max 35 ; Transaction Identifier from the request. |
| `data` | object | Y |  |
| `data.settlementCycleNumber` | string |  | max 35 ; Settlement cycle number set by RPP |
| `data.interbankSettlementDate` | string |  | Settlement date of the transaction. Expressed in Malaysian local date (YYYY-MM-DD). |
| `data.creditor` | object | Y | Information on the recipient |
| `data.creditor.name` | string | Y | max 140 ; Name of the recipient |
| `data.creditorAccount` | object | Y | Information on the receiving account |
| `data.creditorAccount.id` | string | Y | max 34 ; Receiving account information, same as the debtor account number in the original Credit Transfer request. |
| `data.creditorAccount.type` | string |  | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, DEFAULT ; Recipient account type |
| `resp` | object | Y |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, ACSP, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |


### AutoDebit (acquirer)

#### Pre-authorisation (871) request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `creditor` | object |  | Information of the creditor, who is the recipient of funds. |
| `creditor.name` | string | Y | max 140 ; Name of the creditor. |
| `creditorAccount` | object |  | Information of the receiving account |
| `creditorAccount.id` | string | Y | max 34 ; Creditor account number. |
| `creditorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, LOAN, DEFAULT, PROXY ; Creditor account type, participant to return from Account Enquiry response's creditor account type. |
| `creditorAccount.name` | string |  | max 70 ; Creditor account name. |
| `creditorAgent` | object | Y | Information of the receiving FI. |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the receiving FI. |
| `merchant` | object | Y |  |
| `merchant.id` | string | Y | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. |
| `merchant.productId` | string | Y | max 35 ; Product identification assigned by PayNet during merchant registration. |
| `bill` | object | Y |  |
| `bill.reference` | string | Y | max 140 ; Bill reference number for the transaction. |
| `bill.description` | string |  | max 140 ; Description of the bill or transaction. |
| `consent` | object |  |  |
| `consent.id` | string | Y | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. |
| `preAuth` | object | Y |  |
| `preAuth.amount` | number | Y | Pre-Authorized Amount, which must be within the limit of consent maximum amount. |
| `preAuth.currency` | string |  | max 3 ; Currency of the Pre-authorised Amount. Default to MYR. |
| `preAuth.expiryDateTime` | string |  | Expiry Date/Time of Pre-auth transaction. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |

#### Pre-authorisation response body (200)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Message Identifier |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.transactionId` | string |  | max 35 ; Transaction Identifier |
| `data` | object | Y |  |
| `data.settlementCycleNumber` | string |  | max 35 ; Settlement cycle number set by RPP |
| `data.interbankSettlementDate` | string | Y | Settlement date of the transaction. Expressed in Malaysian local date (YYYY-MM-DD). |
| `data.creditor` | object | Y | Information on the creditor |
| `data.creditor.name` | string | Y | max 140 ; Name of the creditor |
| `data.creditorAccount` | object | Y | Information on the receiving account |
| `data.creditorAccount.id` | string | Y | max 34 ; Creditor Account number. |
| `data.creditorAccount.type` | string |  | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, LOAN, DEFAULT, PROXY ; Creditor Account type. |
| `data.creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `data.creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `data.creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `data.creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `data.creditorAccount.customerCategory` | string |  | max 20 ; enum: RET, COR, OTH ; Customer category returned by the RFI |
| `data.debtorAccount` | object |  | Debtor account information |
| `data.debtorAccount.id` | string |  | max 34 ; Debtor account number |
| `data.preAuth` | object | Y |  |
| `data.preAuth.id` | string | Y | max 35 ; Pre-Authorization ID |
| `resp` | object | Y |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, ACSP, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |

#### Real-time Debit (210/230) request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.possibleDuplicate` | boolean |  | Indicates a possible duplicate of a request that was sent in an earlier attempt. To be set to TRUE when re-sending a request to PayNet. |
| `appHeader.copyDuplicate` | string |  | max 4 ; enum: CODU, COPY, EXPY ; OFI uses CODU (copy of a duplicate) during the first attempt of a request, and uses COPY (copy) when retrying the same request. RPP uses COPY to indicate a SAF (Store-and-Forward) and EXPY (expired retry) to indicate a retry reque... |
| `interbankSettlementAmount` | object | Y | Amount of the transaction |
| `interbankSettlementAmount.value` | number | Y | Value of the transaction in MYR, supports up to 2 decimal places. |
| `interbankSettlementAmount.currency` | string |  | max 3 ; Currency of the transaction. Default to MYR. |
| `creditor` | object |  | Information of the creditor. |
| `creditor.name` | string |  | max 140 ; Name of the creditor. name. |
| `creditorAccount` | object | Y | Information of the receiving account |
| `creditorAccount.id` | string | Y | max 34 ; Creditor account number. |
| `creditorAccount.type` | string | Y | max 35 ; enum: CURRENT, SAVINGS, CREDIT_CARD ; Creditor account type. |
| `creditorAccount.name` | string |  | max 70 ; Name of the creditor account. |
| `creditorAgent` | object | Y | Information of the receiving FI. |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the receiving FI - Crediting Agent BIC |
| `bill` | object | Y |  |
| `bill.reference` | string | Y | max 140 ; Bill reference number for the transaction. |
| `bill.description` | string |  | max 140 ; Description of the bill or transaction. |
| `consent` | object | Y | Information of the consent. |
| `consent.id` | string | Y | max 35 ; Consent Registration Id |
| `merchant` | object | Y | Information of the merchant. |
| `merchant.id` | string | Y | max 35 ; Merchant Id |
| `merchant.productId` | string | Y | max 35 ; Used by Debiting Agent to inform customer of which Merchant Product ID |
| `preAuth` | object |  | Pre-authorisation Information |
| `preAuth.id` | string |  | max 35 ; Pre-authorisation ID. |
| `preAuth.amount` | number | Y | Pre-authorised Amount. The amount limit must be within consent maximum amount. |
| `preAuth.currency` | string |  | max 3 ; Currency of the Pre-authorised Amount. Default to MYR. |
| `preAuth.expiryDateTime` | string |  | Expiry Date/Time of Pre-auth transaction. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |

#### Real-time Debit response body (200)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Message Identifier |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.transactionId` | string |  | max 35 ; Transaction Identifier. |
| `data` | object | Y |  |
| `data.preAuth` | object |  | Pre-authorisation Information |
| `data.preAuth.id` | string | Y | max 35 ; Pre-authorised Amount. RPP will populate this field for Pre-Auth transaction 871. Must be returned in 871 and 230 by Debiting Agent. |
| `resp` | object | Y |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, ACSP, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |

#### Direct Debit webhook (080) request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID to track a payment from start to finish. Required to use the same format as `businessMessageId`. |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.possibleDuplicate` | boolean |  | Indicates a possible duplicate of a request that was sent in an earlier attempt. To be set to TRUE when re-sending a request to PayNet. |
| `appHeader.copyDuplicate` | string |  | max 4 ; enum: COPY ; RPP uses COPY to indicate a SAF (Store-and-Forward) to indicate a retry request. |
| `interbankSettlementAmount` | object | Y | Amount of the transaction |
| `interbankSettlementAmount.value` | number | Y | Value of the transaction in MYR, supports up to 2 decimal places. |
| `interbankSettlementAmount.currency` | string |  | max 3 ; Currency of the transaction. Default to MYR. |
| `debtor` | object | Y | Debtor information |
| `debtor.name` | string | Y | max 140 ; Name of debtor / sender |
| `debtor.id` | string |  | max 35 ; Debtor Id |
| `creditor` | object | Y | Creditor information |
| `creditor.name` | string | Y | max 140 ; Name of creditor |
| `debtorAccount` | object | Y | Debtor account information |
| `debtorAccount.id` | string | Y | max 256 ; Debtor account number; up to 34 characters for 010 and 110 for clear account number; up to 256 characters for 060 and 070 for encrypted account number |
| `debtorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, LOAN, DEFAULT ; Debtor account type |
| `debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `debtorAgent` | object | Y |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI |
| `creditorAccount` | object | Y | Information of the receiving account |
| `creditorAccount.id` | string | Y | max 34 ; Creditor Account number. |
| `creditorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, LOAN, DEFAULT ; Creditor Account type. |
| `creditorAgent` | object | Y | Information of the receiving FI. |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the receiving FI |
| `categoryPurpose` | string | Y | Payment Type Information - Category Purpose |
| `recipientReference` | string | Y | max 140 ; Reference note for creditor. For allowable special character details, please refer to the list provided here |
| `paymentDescription` | string |  | max 140 ; Description for the transaction. For allowable special character details, please refer to the list provided here |
| `secondaryValidationIndicator` | boolean | Y | If `true`, the creditor account number will be validated against the creditor ID number. |

#### Direct Debit webhook response the acquirer returns (200)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Message Identifier |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.originalBusinessMessageId` | string | Y | max 35 ; Message Identifier from the request. |
| `appHeader.endToEndId` | string | Y | max 35 ; End to end ID of 220 transaction. |
| `data` | object | Y |  |
| `data.interbankSettlementDate` | string | Y | Settlement date of the transaction. Expressed in Malaysian local date (YYYY-MM-DD). |
| `data.settlementCycleNumber` | string |  | max 3 ; Settlement Cycle when the payment will be settled. E.g. 001,002… |
| `data.creditor` | object | Y | Information on the creditor |
| `data.creditor.name` | string | Y | max 140 ; Name of the creditor |
| `data.creditorAccount` | object | Y | Information on the receiving account |
| `data.creditorAccount.id` | string | Y | max 34 ; Creditor Account number. |
| `data.creditorAccount.type` | string |  | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, DEFAULT ; Creditor Account type. |
| `data.creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `data.creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `data.creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `data.creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `data.creditorAccount.customerCategory` | string | Y | max 20 ; enum: RET, COR, OTH ; Customer category returned by the RFI |
| `resp` | object |  |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, ACSP, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |


### Consent (acquirer)

#### Consent Registration request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.transactionId` | string |  | max 35 ; Transaction Identifier. For 823 use this field to pass in the Business Message Id |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `creditor` | object | Y | Information of the creditor, who is the recipient of funds. |
| `creditor.name` | string | Y | max 140 ; Name of the creditor. |
| `creditor.id` | string | Y | max 140 ; Creditor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 C... |
| `creditor.idType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor ID type. |
| `creditor.secondaryId` | string | Y | max 140 ; Creditor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ... |
| `creditor.secondaryIdType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor Secondary ID type. |
| `creditorAgent` | object |  |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the Receiving FI. |
| `creditorAccount` | object |  | Information on the creditor account |
| `creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Creditor's resident status |
| `creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `debtor` | object |  | Debtor information |
| `debtor.name` | string | Y | max 140 ; Name of debtor |
| `debtor.id` | string | Y | max 140 ; Debtor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 Cou... |
| `debtor.idType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor ID type. |
| `debtor.secondaryId` | string | Y | max 140 ; Debtor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the AL... |
| `debtor.secondaryIdType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor Secondary ID type. |
| `debtor.firstPartyPaymentValidation` | string |  | max 20 ; enum: NO_VALIDATION, DEBTOR_NAME, DEBTOR_ID, DEBTOR_NAME_ID ; Indicates if Issuer should perform validation on debtor information. |
| `debtorAgent` | object |  |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI |
| `debtorAccount` | object |  | Debtor account information |
| `debtorAccount.id` | string | Y | max 34 ; Debtor account number |
| `debtorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET, PROXY ; Debtor account type |
| `debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `consent` | object | Y |  |
| `consent.maxAmount` | number | Y | Maximum Direct Debit Amount |
| `consent.maxAmountCurrency` | string | Y | max 3 ; Currency code for the maximum direct debit amount. Default MYR. |
| `consent.frequency` | string | Y | max 20 ; enum: UNLIMITED, DAILY, WEEKLY, MONTHLY, QUARTERLY, YEARLY ; Frequency mode of the Direct Debit to be permitted by this consent |
| `consent.effectiveDate` | string | Y | Effective Consent Date. Expressed in Malaysian local date (YYYY-MM-DD). |
| `consent.expiryDate` | string | Y | Date when the consent expires. Expressed in Malaysian local date (YYYY-MM-DD). |
| `consent.debtorCanTerminate` | boolean |  | Indicates if the consent can be terminated by the Debtor. |
| `consent.acceptedSourceOfFunds` | array | Y | Accepted sources of fund or line of credit. response. ; items enum: CASA, CREDIT_CARD, WALLET |
| `requestExpiryDateTime` | string | Y | Date and time to indicate when the consent registration request expires. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `merchant` | object | Y |  |
| `merchant.id` | string | Y | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. |
| `merchant.productId` | string | Y | max 35 ; Product identification assigned by PayNet during merchant registration. |
| `reference` | object |  |  |
| `reference.ref1` | string | Y | max 140 ; Reference field 1. |
| `reference.ref2` | string |  | max 140 ; Reference field 2. |

#### Consent Registration response body (200)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.endToEndId` | string | Y | max 35 ; Business Message ID provided in the request. |
| `data` | object | Y |  |
| `data.originalMandateId` | string | Y | max 35 ; Original Mandate Identification |
| `data.consent` | object | Y |  |
| `data.consent.id` | string |  | max 35 ; Consent ID assigned by RPP |
| `data.consent.type` | string |  | max 4 ; enum: DDPT, ABAL ; Consent Type Indicator Code |
| `data.consent.status` | string |  | max 4 ; enum: ACTIVE, INACTIVE, SUSPEND_BY_CUSTOMER, SUSPEND_BY_BANK, ACTIVATE_BY_BANK, PENDING, PENDING_AUTHORISATION, CANCELLED, EXPIRED, REJECT ; Status of the Consent. |
| `data.consent.effectiveDate` | string |  | Effective Consent Date. Expressed in Malaysian local date (YYYY-MM-DD). |
| `data.consent.expiryDate` | string |  | Date when the consent expires. Expressed in Malaysian local date (YYYY-MM-DD). |
| `data.consent.maxAmount` | number | Y | Maximum Direct Debit Amount |
| `data.consent.maxAmountCurrency` | string | Y | max 3 ; Currency code for the maximum direct debit amount. Default MYR. |
| `data.consent.frequency` | string | Y | max 20 ; enum: UNLIMITED, DAILY, WEEKLY, MONTHLY, QUARTERLY, YEARLY ; Frequency mode of the Direct Debit to be permitted by this consent |
| `data.consent.debtorCanTerminate` | boolean |  | Indicates if the consent can be terminated by the Debtor. |
| `data.consent.acceptedSourceOfFunds` | array |  | Accepted sources of fund or line of credit. response. ; items enum: CASA, CREDIT_CARD, WALLET |
| `data.merchant` | object |  |  |
| `data.merchant.id` | string |  | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. |
| `data.merchant.name` | string |  | max 140 ; Merchant Name |
| `data.merchant.msicCode` | string |  | Merchant MSIC code, provided by PayNet. |
| `data.creditor` | object |  |  |
| `data.creditor.name` | string |  | max 140 ; Name of the creditor. |
| `data.debtor` | object | Y | Debtor information |
| `data.debtor.name` | string |  | max 140 ; Name of debtor |
| `data.debtor.id` | string |  | max 140 ; Debtor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 Cou... |
| `data.debtor.idType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor ID type. |
| `data.debtor.secondaryId` | string |  | max 140 ; Debtor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the AL... |
| `data.debtor.secondaryIdType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor Secondary ID type. |
| `data.debtorAccount` | object | Y | Debtor account information |
| `data.debtorAccount.id` | string | Y | max 34 ; Debtor account number |
| `data.debtorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET ; Debtor account type |
| `data.debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `data.debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `data.debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `data.debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `data.debtorAgent` | object | Y |  |
| `data.debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI |
| `data.firstPartyPaymentValidation` | string |  | max 20 ; enum: NO_VALIDATION, DEBTOR_NAME, DEBTOR_ID, DEBTOR_NAME_ID ; Indicates if Issuer should perform validation on debtor information. |
| `data.reference` | object |  |  |
| `data.reference.ref1` | string | Y | max 140 ; Reference field 1. |
| `data.reference.ref2` | string |  | max 140 ; Reference field 2. |
| `resp` | object | Y |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |

#### Consent Enquiry (751/752) request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.pageKey` | string |  | max 35 ; Page index. Page returned will start with element after the element indicated by the pageKey. |
| `appHeader.endToEndIdSignature` | string |  | max 1024 ; Signed BiszMsgIdr of Consent to be retrieved |
| `originalMandateId` | string | Y | max 35 ; Original Mandate Identification. |
| `debtor` | object |  |  |
| `debtor.secondaryId` | string |  | max 140 ; Value of secondaryId, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the A... |
| `debtor.secondaryIdType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Secondary ID type. |

#### Retrieve Consent Detail (812) request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.pageKey` | string |  | max 35 ; Page index. Page returned will start with element after the element indicated by the pageKey. |
| `appHeader.endToEndIdSignature` | string |  | max 1024 ; Signed BiszMsgIdr of Consent to be retrieved |
| `originalMandateId` | string | Y | max 35 ; Original Mandate Identification. |
| `debtor` | object |  |  |
| `debtor.secondaryId` | string |  | max 140 ; Value of secondaryId, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the A... |
| `debtor.secondaryIdType` | string |  | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Secondary ID type. |

#### Consent Status Update request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.transactionId` | string |  | max 35 ; Transaction Identifier. 803 - EndtoEndId of the original 801/802 Consent Registration Request. 813 - EndtoEndId of the original 811 Consent Registration Request. 822 - EndtoEndId of the original 821 Consent Registration Request. |
| `creditor` | object | Y | Information of the creditor, who is the recipient of funds. |
| `creditor.name` | string | Y | max 140 ; Name of the creditor. |
| `creditor.id` | string | Y | max 140 ; Creditor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 C... |
| `creditor.idType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor ID type. |
| `creditor.secondaryId` | string | Y | max 140 ; Creditor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ... |
| `creditor.secondaryIdType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor Secondary ID type. |
| `creditorAgent` | object |  |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the Receiving FI. |
| `creditorAccount` | object |  | Information on the creditor account |
| `creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Creditor's resident status |
| `creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `debtor` | object |  | Debtor information |
| `debtor.name` | string | Y | max 140 ; Name of debtor |
| `debtor.id` | string | Y | max 140 ; Debtor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 Cou... |
| `debtor.idType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor ID type. |
| `debtor.secondaryId` | string | Y | max 140 ; Debtor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the AL... |
| `debtor.secondaryIdType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor Secondary ID type. |
| `debtor.firstPartyPaymentValidation` | string |  | max 20 ; enum: NO_VALIDATION, DEBTOR_NAME, DEBTOR_ID, DEBTOR_NAME_ID ; Indicates if Issuer should perform validation on debtor information. |
| `debtorAgent` | object |  |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI |
| `debtorAccount` | object |  | Debtor account information |
| `debtorAccount.id` | string | Y | max 34 ; Debtor account number |
| `debtorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET ; Debtor account type |
| `debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `consent` | object | Y |  |
| `consent.maxAmount` | number | Y | Maximum Direct Debit Amount |
| `consent.maxAmountCurrency` | string | Y | max 3 ; Currency code for the maximum direct debit amount. Default MYR. |
| `consent.frequency` | string | Y | max 20 ; enum: UNLIMITED, DAILY, WEEKLY, MONTHLY, QUARTERLY, YEARLY ; Frequency mode of the Direct Debit to be permitted by this consent |
| `consent.effectiveDate` | string | Y | Effective Consent Date. Expressed in Malaysian local date (YYYY-MM-DD). |
| `consent.expiryDate` | string | Y | Date when the consent expires. Expressed in Malaysian local date (YYYY-MM-DD). |
| `consent.debtorCanTerminate` | boolean |  | Indicates if the consent can be terminated by the Debtor. |
| `consent.status` | string |  | max 35 ; enum: PENDING, PENDING_AUTHORISATION, ACTIVE, INACTIVE ; Consent status to be updated to. |
| `consent.acceptedSourceOfFunds` | array | Y | Accepted sources of fund or line of credit. response. ; items enum: CASA, CREDIT_CARD, WALLET |
| `merchant` | object |  |  |
| `merchant.id` | string | Y | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. |
| `merchant.productId` | string | Y | max 35 ; Product identification assigned by PayNet during merchant registration. |
| `reference` | object |  |  |
| `reference.ref1` | string | Y | max 140 ; Reference field 1. |
| `reference.ref2` | string |  | max 140 ; Reference field 2. |

#### Consent Termination request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `merchant` | object |  |  |
| `merchant.id` | string | Y | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. |
| `merchant.productId` | string | Y | max 35 ; Product identification assigned by PayNet during merchant registration. |
| `reason` | string | Y | max 256 ; Reason for cancellation |

#### Consent Registration webhook request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `creditor` | object | Y | Information of the creditor, who is the recipient of funds. |
| `creditor.name` | string | Y | max 140 ; Name of the creditor. |
| `creditor.id` | string | Y | max 140 ; Creditor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 C... |
| `creditor.idType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor ID type. |
| `creditor.secondaryId` | string | Y | max 140 ; Creditor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ... |
| `creditor.secondaryIdType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor Secondary ID type. |
| `creditorAgent` | object |  |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the Receiving FI. |
| `creditorAccount` | object |  | Information on the creditor account |
| `creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Creditor's resident status |
| `creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `debtor` | object |  | Debtor information |
| `debtor.name` | string | Y | max 140 ; Name of debtor |
| `debtor.id` | string | Y | max 140 ; Debtor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 Cou... |
| `debtor.idType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor ID type. |
| `debtor.secondaryId` | string | Y | max 140 ; Debtor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the AL... |
| `debtor.secondaryIdType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor Secondary ID type. |
| `debtor.firstPartyPaymentValidation` | string |  | max 20 ; enum: NO_VALIDATION, DEBTOR_NAME, DEBTOR_ID, DEBTOR_NAME_ID ; Indicates if Issuer should perform validation on debtor information. |
| `debtorAgent` | object |  |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI |
| `debtorAccount` | object |  | Debtor account information |
| `debtorAccount.id` | string | Y | max 34 ; Debtor account number |
| `debtorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET ; Debtor account type |
| `debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `consent` | object |  |  |
| `consent.id` | string |  | max 35 ; BIC of the originating FI |
| `consent.type` | string |  | max 20 ; enum: DDPT ; Consent Type indicator code. |
| `consent.status` | string |  | max 35 ; enum: PENDING, PENDING_AUTHORISATION, ACTIVE, INACTIVE ; Consent status to be updated to. |
| `consent.maxAmount` | number | Y | Maximum Direct Debit Amount |
| `consent.frequency` | string | Y | max 20 ; enum: UNLIMITED, DAILY, WEEKLY, MONTHLY, QUARTERLY, YEARLY ; Frequency mode of the Direct Debit to be permitted by this consent |
| `consent.effectiveDate` | string | Y | Effective Consent Date. Expressed in Malaysian local date (YYYY-MM-DD). |
| `consent.expiryDate` | string | Y | Date when the consent expires. Expressed in Malaysian local date (YYYY-MM-DD). |
| `consent.debtorCanTerminate` | boolean |  | Indicates if the consent can be terminated by the Debtor. |
| `consent.acceptedSourceOfFunds` | array | Y | Accepted sources of fund or line of credit. response. ; items enum: CASA, CREDIT_CARD, WALLET |
| `requestExpiryDateTime` | string | Y | Date and time to indicate when the consent registration request expires. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `merchant` | object | Y |  |
| `merchant.id` | string | Y | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. |
| `merchant.productId` | string | Y | max 35 ; Product identification assigned by PayNet during merchant registration. |
| `merchant.msicCode` | string |  | Merchant MSIC code. |
| `reference` | object |  |  |
| `reference.ref1` | string | Y | max 140 ; Reference field 1. |
| `reference.ref2` | string |  | max 140 ; Reference field 2. |

#### Consent Update webhook request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.transactionId` | string |  | max 35 ; Transaction Identifier. 803 - EndtoEndId of the original 801/802 Consent Registration Request. 813 - EndtoEndId of the original 811 Consent Registration Request. 822 - EndtoEndId of the original 821 Consent Registration Request. |
| `creditor` | object | Y | Information of the creditor, who is the recipient of funds. |
| `creditor.name` | string | Y | max 140 ; Name of the creditor. |
| `creditor.id` | string | Y | max 140 ; Creditor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 C... |
| `creditor.idType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor ID type. |
| `creditor.secondaryId` | string | Y | max 140 ; Creditor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ... |
| `creditor.secondaryIdType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION ; Creditor Secondary ID type. |
| `creditorAgent` | object |  |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the Receiving FI. |
| `creditorAccount` | object |  | Information on the creditor account |
| `creditorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Creditor's resident status |
| `creditorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `creditorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `creditorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `debtor` | object |  | Debtor information |
| `debtor.name` | string | Y | max 140 ; Name of debtor |
| `debtor.id` | string | Y | max 140 ; Debtor ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the ALPHA- 3 Cou... |
| `debtor.idType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor ID type. |
| `debtor.secondaryId` | string | Y | max 140 ; Debtor Secondary ID, do not include hyphens ('-'), only alphanumeric characters are allowed. Passport Number should be followed by ALPHA-3 country code of the country of issuance, e.g. E394029340VSGP. Refer to this list for the AL... |
| `debtor.secondaryIdType` | string | Y | max 20 ; enum: NEW_IC, ARMY_POLICE, PASSPORT, BIZ_REGISTRATION, MBNO ; Debtor Secondary ID type. |
| `debtor.firstPartyPaymentValidation` | string |  | max 20 ; enum: NO_VALIDATION, DEBTOR_NAME, DEBTOR_ID, DEBTOR_NAME_ID ; Indicates if Issuer should perform validation on debtor information. |
| `debtorAgent` | object |  |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI |
| `debtorAccount` | object |  | Debtor account information |
| `debtorAccount.id` | string | Y | max 34 ; Debtor account number |
| `debtorAccount.type` | string | Y | max 20 ; enum: CURRENT, SAVINGS, CREDIT_CARD, WALLET ; Debtor account type |
| `debtorAccount.residentStatus` | string |  | max 20 ; enum: RESIDENT, NON_RESIDENT, UNAVAILABLE ; Debtor’s resident status |
| `debtorAccount.productType` | string |  | max 20 ; enum: ISLAMIC, CONVENTIONAL ; Product type of the account |
| `debtorAccount.shariaCompliance` | string |  | max 20 ; enum: YES, NO, UNKNOWN ; Whether or not the account is Sharia compliant |
| `debtorAccount.accountHolderType` | string |  | max 20 ; enum: SINGLE, JOINT, GOVERNMENT, CORPORATE, NGO ; Account holder type |
| `consent` | object | Y |  |
| `consent.id` | string |  | max 35 ; BIC of the originating FI |
| `consent.type` | string |  | max 20 ; enum: DDPT ; Consent Type indicator code. |
| `consent.status` | string |  | max 35 ; enum: PENDING, PENDING_AUTHORISATION, ACTIVE, INACTIVE ; Consent status to be updated to. |
| `consent.maxAmount` | number | Y | Maximum Direct Debit Amount |
| `consent.maxAmountCurrency` | string | Y | max 3 ; Currency code for the maximum direct debit amount. Default MYR. |
| `consent.frequency` | string | Y | max 20 ; enum: UNLIMITED, DAILY, WEEKLY, MONTHLY, QUARTERLY, YEARLY ; Frequency mode of the Direct Debit to be permitted by this consent |
| `consent.effectiveDate` | string | Y | Effective Consent Date. Expressed in Malaysian local date (YYYY-MM-DD). |
| `consent.expiryDate` | string | Y | Date when the consent expires. Expressed in Malaysian local date (YYYY-MM-DD). |
| `consent.debtorCanTerminate` | boolean |  | Indicates if the consent can be terminated by the Debtor. |
| `consent.acceptedSourceOfFunds` | array | Y | Accepted sources of fund or line of credit. response. ; items enum: CASA, CREDIT_CARD, WALLET |
| `requestExpiryDateTime` | string | Y | Date and time to indicate when the consent registration request expires. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `merchant` | object |  |  |
| `merchant.id` | string | Y | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. |
| `merchant.productId` | string | Y | max 35 ; Product identification assigned by PayNet during merchant registration. |
| `merchant.msicCode` | string |  | Merchant MSIC code. |
| `reference` | object |  |  |
| `reference.ref1` | string | Y | max 140 ; Reference field 1. |
| `reference.ref2` | string |  | max 140 ; Reference field 2. |

#### Consent Reject webhook (804) request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.transactionId` | string |  | max 35 ; 801/802 Business Message Id |
| `debtorAgent` | object |  |  |
| `debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI |
| `creditorAgent` | object |  |  |
| `creditorAgent.id` | string | Y | max 35 ; BIC of the Receiving FI. |
| `merchant` | object | Y |  |
| `merchant.id` | string | Y | max 35 ; Merchant ID for PayNet to look up Merchant crediting information. |
| `merchant.productId` | string | Y | max 35 ; Product identification assigned by PayNet during merchant registration. |
| `blockFutureRequests` | boolean |  | Indicate that the debtor intends to block future Request-to-Pay. |


### Transaction enquiry `POST /v3/transactions/enquire` (630)

#### Enquire Transactions (630) request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.transactionId` | string | Y | max 35 ; The `businessMessageId` of the original payment transaction. |
| `debtorAgent` | object | Y |  |
| `debtorAgent.id` | string | Y | max 35 ; Query transactions matching this BIC of originating FI. |
| `creditorAgent` | object | Y |  |
| `creditorAgent.id` | string | Y | max 35 ; Query transactions matching this BIC of receiving FI. |

#### Enquire Transactions response body (200)

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `appHeader.originalBusinessMessageId` | string | Y | max 35 ; Message Identifier from the request. |
| `appHeader.transactionId` | string | Y | max 35 ; The `businessMessageId` of the original payment transaction. |
| `data` | object | Y |  |
| `data.transaction` | object | Y |  |
| `data.transaction.status` | string | Y | max 4 ; enum: ACTC, ACSP, RJCT ; Returns the status of the request |
| `data.transaction.reason` | object | Y | Additional information on the status |
| `data.transaction.reason.name` | string | Y | max 120 ; Name of the corresponding response code |
| `data.transaction.reason.code` | string | Y | max 20 ; Response code of the request |
| `data.transaction.reason.description` | string | Y | max 2000 ; Description of the response code |
| `data.transaction.reason.details` | string |  | max 2000 ; Additional information related to the status of the request |
| `data.transaction.reason.additionalCode` | string |  | max 20 ; Additional response code for PayNet troubleshooting |
| `data.settlementCycleNumber` | string | Y | max 35 ; Settlement cycle number set by RPP |
| `data.interbankSettlementAmount` | object | Y | Amount of the transaction |
| `data.interbankSettlementAmount.value` | number | Y | Value of the transaction in MYR, supports up to 2 decimal places. Special handling for Cross Border transaction: For static Cross Border QR, value is calculated by converting the foreign currency amount using FX rate in Cross Bord... |
| `data.interbankSettlementAmount.currency` | string |  | max 3 ; Currency of the transaction. Default to MYR. |
| `data.convertedAmount` | object |  | Converted amount from MYR, for use with foreign currency. |
| `data.convertedAmount.value` | number | Y | Value of the amount |
| `data.convertedAmount.currency` | string |  | max 3 ; Currency of the transaction. Default to MYR. |
| `data.interbankSettlementDate` | string | Y | Settlement date of the transaction. Expressed in Malaysian local date (YYYY-MM-DD). |
| `data.debtor` | object | Y | Debtor information |
| `data.debtor.name` | string | Y | max 140 ; Name of the debtor |
| `data.debtorAgent` | object |  |  |
| `data.debtorAgent.id` | string | Y | max 35 ; BIC of the originating FI |
| `data.creditor` | object |  | Creditor information |
| `data.creditor.name` | string | Y | max 140 ; Name of the creditor |
| `resp` | object | Y |  |
| `resp.status` | string | Y | max 4 ; enum: ACTC, ACSP, RJCT ; Returns the status of the request |
| `resp.reason` | object | Y | Additional information on the status |


### Network administration

#### Sign On request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object | Y |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |

#### System Event Notification webhook request body

| Field | Type | Req | Notes |
|---|---|---|---|
| `appHeader` | object |  |  |
| `appHeader.businessMessageId` | string | Y | max 35 ; Business Message Identifier (see format above) |
| `appHeader.creationDateTime` | string | Y | Message creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
| `eventCode` | string | Y | max 40 ; Event code used to identify the event |
| `eventParameter` | array |  | max 35 ; Additional information about the event |
| `eventDescription` | string |  | max 350 ; Description of the event |
| `eventTime` | string |  | Event creation date and time. Expressed in Malaysian local time with UTC offset format (YYYY-MM-DDThh:mm:ss.sss+/-hh:mm). |
