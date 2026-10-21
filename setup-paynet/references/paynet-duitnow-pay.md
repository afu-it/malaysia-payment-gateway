# PayNet DuitNow Pay Reference (acquirer API)

DuitNow Pay is PayNet's online checkout product over DuitNow Online Banking/Wallets (one-time), DuitNow Consent (save payment method) and DuitNow AutoDebit (recurring). It is consumed by an acquirer, not by a merchant. PayNet: "To be a part of the DuitNow Pay platform, you will need to be affiliated with our merchant acquirers' network." Acquirers must already be RPP participants with Online Banking/Wallets, Consent and AutoDebit enabled. Checked 2026-10-21.

Sources: `https://docs.paynet.my/docs/duitnow-pay/*` (overview, user-journey, registration/*, integration/*, browser-redirection, response-codes, mapping-tables, transaction-codes, batchfilelist, reportlist, csr-renewal, branding-guidelines).

## Hosts

API base URL: Not documented in official docs (checked 2026-10-21); PayNet provides it during onboarding. The documented sample `redirectUrl` is `https://duitnow-pay.uat.inet.paynet.my:8443/?sessionId=<id>`. All examples below are paths only.

## Authentication (`/docs/duitnow-pay/integration/api-authentication`)

- Generate a 2048-bit RSA key: `openssl genpkey -algorithm RSA -out duitnow_pay_priv.key -pkeyopt rsa_keygen_bits:2048`, then a CSR (`/integration/generate-key-pair`). Upload the CSR in the Developer Portal (System Verification server configuration, and again at go-live). Get PayNet's public key from the Developer Portal ("One-Time Configuration For Your Application", and "View Production Configuration Details").
- Every request carries `Authorization: <JWT>`. The docs say "The JWT token should be included in the 'Authorization' request header" and do not show a `Bearer` prefix. Confirm the exact header form with PayNet.
- JWT alg `RS256`, signed with the acquirer private key. Claims:

| Claim | Value |
|---|---|
| `exp` | Expiry; must be within 1 hour |
| `iss` | BIC assigned during onboarding |
| `sub` | Merchant ID assigned during merchant registration |
| `jti` | UUID v4 generated per request (samples reuse the `checkoutId`) |
| `key` | Project ID assigned during onboarding (`<api-key-name>` in samples) |
| `data` | The request payload body, as a JSON object |
| `iat` | Set in the Java and Python samples |

- Verify API responses with PayNet's public key.
- Required headers on every endpoint (`/integration/construct-header`): `X-Ip-Address` (max 100, IPv4 or IPv6 of the payer; `0` if unknown) and `X-Gps-Coordinates` (max 35, decimal degrees such as `40.689263, 74.044505`; `0` if unknown).

Pseudo-code:

```text
body = {...request fields...}
claims = { iss: BIC, sub: MERCHANT_ID, jti: uuid4(), key: PROJECT_ID, data: body,
           iat: now, exp: now + 3600 }
jwt = RS256_sign(acquirer_private_key, claims)
POST <DUITNOW_PAY_BASE>/v1/payment/intent
  Authorization: <jwt>
  X-Ip-Address: <payer ip or 0>
  X-Gps-Coordinates: <lat, lng or 0>
  Content-Type: application/json
  body
```

Certificate renewal: through PayNet's One Stop Portal, Certificate Management, "Renew Certificate" (button appears 90 days before expiry); either let PayNet generate a CSR or upload your own (`/docs/duitnow-pay/csr-renewal`).

## Webhooks

The acquirer gives PayNet one webhook URL plus the IP range to whitelist at System Verification and at go-live. PayNet sends: Update Checkout Details, Update Payment Status (success, and a separate rejected variant), Update Consent Details, Update Refund Status, Update AutoDebit Batch Status. The acquirer "shall provide an acknowledgement back to API Gateway" for each. How PayNet signs or authenticates webhook calls: Not documented in official docs (checked 2026-10-21). Treat webhooks as untrusted until confirmed with a status enquiry.

## One-time payment, PayNet-hosted page

`POST /v1/payment/intent`

| Field | Type | Max | Req | Notes |
|---|---|---|---|---|
| `dataType` | String | 2 | Y | `01` payment (checkout WebView), `02` save payment method |
| `transactionFlow` | String | 2 | N | `01` retail (default), `02` corporate; checker must approve within 5 days; only for `dataType=01` |
| `checkoutId` | String | 36 | Y | UUID v4 from the acquirer; your idempotency key |
| `sourceOfFunds` | Array | n/a | N (Y for `02`) | `01` CASA, `02` credit card (not supported), `03` eWallet |
| `amount` | String | 18 | Y | MYR with two decimals, e.g. `10.00` |
| `merchantName` | String | 100 | N | Shown on the checkout WebView |
| `merchantReferenceId` | String | 140 | Y | Shown to payer at the issuer |
| `merchant.productId` | String | 35 | Y | Assigned by PayNet at merchant registration |
| `customer.name` | String | 140 | Y | Payer name |
| `customer.identityValidation` | String | 2 | Y | `00` none, `01` name, `02` ID, `03` name and ID |
| `customer.identificationType` | String | 2 | N | `01` NRIC, `02` army, `03` passport, `04` registration no., `05` mobile |
| `customer.identification` | String | 140 | Cond. | Required if type sent. NRIC without hyphens; passport with country (e.g. `E394029340VSGP`); mobile with country code (`+60...`) |
| `consent.maxAmount` | String | 18 | for `02` | Two decimals |
| `consent.effectiveDate` | String | 10 | for `02` | `YYYY-MM-DD` |
| `consent.expiryDate` | String | 10 | for `02` | `YYYY-MM-DD` |
| `consent.frequency` | String | 2 | for `02` | `01` unlimited, `02` daily, `03` weekly, `04` monthly, `05` quarterly, `06` yearly |
| `language` | String | 2 | Y | `en` or `bm` |

Response: `data.id` (session ID, max 100), `data.redirectUrl` (max 140), `message` `OK` or a reason code. Open `redirectUrl` to load DuitNow Pay. For `dataType=02`, if any consent field is missing, PayNet shows a consent management page so the payer fills it.

Flow: payer picks a bank on the DuitNow Pay page, PayNet sends "Update Checkout Details" (maps `checkoutId` to `rtpEndToEndId`), payer authorises at the issuer, PayNet sends "Update Payment Status", and the issuer redirects the payer to the acquirer's redirect URL configured at onboarding, carrying only the `endToEndId`.

## One-time payment, self-hosted page

1. `GET /v2/bw/banks` (call every 30 minutes). Response `data.retail[]` and `data.corporate[]`, each with `bicCode`, `name`, `browser`, `androidAppId`, `iosAppId`, `url` (bank redirect URL), `isConsent`, `isActive`, `isFpx`, `isBank` (false = e-wallet), `isObw`, `priority` (lower first; `0` means sort alphabetically). Listing rules with `isActive=true`: one-time payment lists OBW-only banks as OBW, FPX-only banks as FPX, banks with both only under OBW, neither not listed; save payment lists only `isConsent=true`.
2. `POST /v1/bw/checkout` with `checkoutId`, `transactionFlow`, `issuer` (bank name from the list), `sourceOfFunds` (required here), `amount`, `merchant.productId`, `merchantName`, `merchantReferenceId`, `customer{...}` as above. Response `data.endToEndId` (35), `data.endToEndIdSignature` (1024, signed by RPP), `data.issuer`, `data.paymentMethod` `01`, `message` (e.g. `U000`).
3. Redirect the payer (see Browser redirection).
4. Receive the same webhooks as the hosted flow.

## Save payment method (DuitNow Consent)

- Hosted: `POST /v1/payment/intent` with `dataType=02`, `sourceOfFunds` and the `consent` object.
- Self-hosted: `POST /v1/bw/consent` with `checkoutId`, `issuer`, `sourceOfFunds`, `merchant.productId`, `merchantReferenceId`, `customer{...}`, `consent{maxAmount, effectiveDate, expiryDate, frequency}`. Response `endToEndId`, `endToEndIdSignature`, `issuer`. Follow the consent UI requirements on `/integration/self-hosted-page/initiate-consent` (colour and font may change, guidance must stay clear).
- Webhook "Update Checkout Details" for consent: `checkoutId`, `consentEndToEndId`, `consentId`, `issuer`.
- Webhook "Update Consent Details": `checkoutId`, `endToEndId`, `consentStatus{consentId, code (consent status), message}`, `originalIssuer`, `originalAccountNumber` (last 4), `issuer`, `accountNumber` (last 4). The original fields are for consent transfers between participants.
- `GET /v1/bw/consent/request?checkoutId=` (30 s rate limit): `messageId`, `transactionStatus` (`PDNG`, `ACTV`, `CANC`, `PDAU`, `EXPI`), `consentId`, `issuer`.
- `GET /v1/bw/consent?consentId=`: `messageId`, `consent{consentId, consentStatus, effectiveDate, expiryDate, frequency, allowTerminatedByDebtor ("false")}`, `issuer`, `accountNumber` (last 4).
- `DELETE /v1/bw/consent?consentId=`: returns `messageId`.

## AutoDebit (recurring)

`POST /v1/bw/autodebit`

| Field | Max | Req | Notes |
|---|---|---|---|
| `checkoutId` | 36 | Y | New UUID v4 per debit |
| `consentId` | 35 | Y | From the consent webhook or enquiry |
| `amount` | 18 | Y | Two decimals |
| `merchantName` | 100 | N | |
| `merchantReferenceId` | 140 | Y | |

Response `data.messageId`, `data.transactionStatus` (`CLEARED` success, `REJECTED` rejected by RPP or debiting agent), `data.issuer`, `message` (reason or issuer rejection code). AutoDebit status can also be checked with `GET /v2/bw/checkout-status?paymentMethod=02&checkoutId=`.

AutoDebit batch (`/integration/duitnow-autodebit-batch`): `GET /v1/bw/autodebit-batch-key` returns temporary storage credentials valid 1 hour (`accessKeyId`, `secretAccessKey`, `sessionToken`, `expiration`, `uploadTargetUrl`). Upload `.txt` batch files (max 500 records each) to `<BIC>/<SI>/<YYYYMMDD>/Uploads/`, each with a `.sig` file holding an RS256 JWT whose claims are `key` (consumer key), `iat`, `jti` (UUID), `ds` (SHA-256 hex of the file content). Response files appear in Downloads. PayNet sends "Update AutoDebit Batch Status" with `request_filename`, `response_filename`, `total_records`, `successful_records`, `failed_records`, `status`, `message`. Batch file formats (RTPB001, RTPB002, CRB001, DDB001, DDB002 and others) are on `/docs/duitnow-pay/batchfilelist`; RTPB001 naming `YYYYMMDD_BBBBBBBB_RTPB001_SSSSSSSS_NNNNN`, max 99,999 files a day.

## Payment status webhook and enquiry

Webhook "Update Payment Status" fields: `checkoutId`, `endToEndId`, `paymentStatus{payerName, code (ACSP/ACTC/RJCT), substate, message (reason code)}`, `issuer`, `paymentMethod` (`01`). It fires on success. A separate "Update Payment Status (rejected)" webhook carries the same shape without `payerName`.

`substate` values: `RECEIVED` pending, `CLEARED` successful credit, `REJECTED` rejected by debiting agent, `PENDAUTH` pending corporate authorisation.

Settle on `substate=CLEARED`. Do not rely on `code`: the docs' samples show `CLEARED` with `code` `ACTC` and `message` `U002`, and `REJECTED` with `code` `ACSP` and `U000`.

Corporate flow (`transactionFlow=02`): first webhook `PENDAUTH`, then `CLEARED` after the authoriser approves. Run an enquiry on day 5 to confirm the final status.

Enquiry:

- v2 (latest): `GET /v2/bw/checkout-status?paymentMethod=01|02&checkoutId=`; response `endToEndId`, `transactionStatus`, `issuer`, `paymentMethod`, `message`. Supports AutoDebit.
- v1: `GET /v1/bw/rtp?checkoutId=`; adds `messageId`.
- Rate limit: once every 30 seconds per transaction. PayNet: "Acquirer should perform retrieve payment status after 60 mins before updating the final status for the transaction."
- `GET /v1/bw/checkout?endToEndId=` returns `checkoutId` with `rtpEndToEndId`, `issuer`, `paymentMethod` (one-time), or `checkoutId`, `consentEndToEndId`, `consentId`, `issuer` (consent). Use it when the payer returns with only `endToEndId` and "Update Checkout Details" never arrived.

Decision table (one-time payment; consent uses the same pattern with Update Consent Details and Enquire Payment Method Details):

| Checkout Details webhook | Payer redirected back | Payment Status webhook | Action |
|---|---|---|---|
| missing | yes | received | No enquiry, payment successful |
| received | yes | missing | Enquire payment status |
| missing | yes | missing | Enquire checkout details, then payment status |
| missing | no | missing | Not stated in the docs table (no ID known yet; enquire by your `checkoutId` with the status API) |

## Refunds

`POST /v1/bw/refund`, allowed up to 90 days from capture, for one-time and AutoDebit payments.

| Field | Max | Req | Notes |
|---|---|---|---|
| `refundId` | 36 | Y | UUID v4, your refund idempotency key |
| `checkoutId` | 36 | Y | Original payment |
| `paymentMethod` | 2 | Y | Same as original |
| `amount` | 18 | Y | Two decimals; partial refunds are implied by E003 "Refund amount exceed refund percentage allowed" |
| `merchantReferenceId` | 140 | Y | Shown to the payer |

Response `data.endToEndId`, `data.refundStatus` (always `REFUND_PENDING`), `message` `OK`.

Webhook "Update Refund Status": `refundId`, `checkoutId`, `endToEndId`, `paymentStatus{code, substate, message}`, `issuer`, `paymentMethod`, `amount`. Substates: `REFUND_PENDING`, `REFUND_ACCEPTED` (sent to issuing bank), `REFUND_REJECTED` (see message), `REFUND_EXCEPTION` (retry, tell PayNet if it persists).

`GET /v1/bw/refund?refundId=`: only if the webhook failed, once, at least 1 hour after the refund request. Returns `checkoutId`, `endToEndId`, `code`, `refundStatus`, `issuer`, `paymentMethod`, `amount`.

## Browser redirection (`/docs/duitnow-pay/browser-redirection`)

Redirect the payer to the bank `url` from the bank list, adding query parameters:

| Parameter | Value |
|---|---|
| `EndtoEndId` | `endToEndId` (for consent, the mandate ID) |
| `EndtoEndIdSignature` | `endToEndIdSignature` from the 861 (payment) or 811 (consent) response |
| `DbtrAgt` | BIC of the selected debiting bank |

URL-encode values (`?` `%3F`, `/` `%2F`, `&` `%26`, `+` `%2B`, `=` `%3D`). Paths: DOBW `https://<domain>/RPP/MY/Redirect/RTP`, AutoDebit consent `https://<domain>/RPP/MY/Redirect/Consent`, both GET. FPX bridge: UAT `https://uat.mepsfpx.com.my/FPXMain/r2pReceiver.jsp`, PROD `https://www.mepsfpx.com.my/FPXMain/r2pReceiver.jsp`; when the payer was sent through this bridge, the bank returns to the merchant with HTTP POST. The bank also returns to the merchant using the same parameter specification.

Example (sandbox values from the docs):

```text
https://www.testbankA.my/RPP/MY/Redirect/RTP?EndtoEndId=20201104MAXIMYKL861ORB21001206&EndtoEndIdSignature=SXAj...%3D&DbtrAgt=TEST0021
```

## Transaction and event codes used by DuitNow Pay

From `/docs/duitnow-pay/transaction-codes` and mapping tables: 650 list banks, 861 request-to-pay (redirect), 862 retrieve transaction info, 863 update payment status, 864 RTP status enquiry, 865 cancellation, 070 credit transfer (RTP redirect), 854 refund intent, 811 consent registration (redirect), 812 retrieve consent, 813 update consent status, 814 consent status enquiry, 815 consent cancellation, 210/220/230 real-time debit, 011/012 reversal. Event codes: `PMT001` payment status, `CMN001` consent update, `CMN002` consent termination, `CRN001` consent update status, `RED001` RTP redirect, `RED002` consent registration redirect. Full lists in `paynet-codes.md`.

## Reports

`/docs/duitnow-pay/reportlist`: RPP-S02 cycle summary, RPP-D01 daily summary, RPP-I01 daily itemised, RPP-M01 participant fee monthly, RTP SAF files (RTPSS02, RTPSR02, RTPSE02), RTP payment reports (RTPPS02, RTPPR02, RTPPE02), On-Us report, consent reports (CMR02, CMR04, CRSS02, CRSR02, CRSE02), merchant files (MRR02, MMR02, MPMR02). The Financial Reports API in `paynet-api.md` serves these programmatically.

## UI and branding

Follow the DuitNow Pay Figma design guideline, DuitNow Brand Guidelines v2.0, the DuitNow Pay logo pack and issuer logo pack linked on `/docs/duitnow-pay/branding-guidelines`.

## Settlement checklist

1. Store `checkoutId` (UUID v4) with amount, merchant `productId` and order before calling PayNet.
2. On any webhook, match `checkoutId`, then confirm with `GET /v2/bw/checkout-status` before fulfilment because webhook authentication is not documented.
3. Fulfil only on `CLEARED`. Keep `RECEIVED` and `PENDAUTH` pending. Fail on `REJECTED` and record `message`.
4. Make webhook handlers idempotent on `checkoutId` + `endToEndId` (and `refundId` for refunds); return the acknowledgement even on duplicates.
5. Schedule a 60-minute final enquiry and a day-5 enquiry for corporate flows, respecting the 30-second rate limit.
