# PayNet API Reference (entry point)

Use this file first. It says which PayNet product fits the caller, where each product's hosts and auth are, and lists every documented endpoint. Product detail lives in the split files named below. Checked 2026-10-21 against `https://docs.paynet.my` (current) and `https://docs.developer.paynet.my` (old site, marked "no longer being updated and may not contain the latest information").

## Who can integrate what

PayNet (Payments Network Malaysia Sdn Bhd) runs the shared rails. It does not onboard ordinary merchants for direct payment acceptance.

| Caller | What they can integrate with PayNet directly | Detail file |
|---|---|---|
| Ordinary merchant (online shop, SaaS) | Nothing directly for DuitNow Pay. PayNet says a merchant "will need to be affiliated with our merchant acquirers' network". Use a bank acquirer or licensed gateway. | Use that gateway's skill |
| FPX merchant or TPA (Third Party Acquirer) approved by PayNet | FPX B2C, B2B1, B2B2, bulk refund, status enquiry, bank list. Merchant docs and sample code are public. | `paynet-fpx.md` |
| Acquirer (bank or non-bank RPP participant) | DuitNow Pay (needs RPP Online Banking/Wallets, Consent and AutoDebit already enabled), RPP v3 webhooks and APIs, DuitNow QR acquiring, reports | `paynet-duitnow-pay.md`, `paynet-rpp-v3.md`, `paynet-duitnow-qr.md` |
| Issuer (bank or e-wallet) | RPP issuer APIs, FPX bank webview, NAD, consent issuer side | Catalogue only (see Coverage in `source-map.md`) |
| Anyone generating a DuitNow QR for an acquirer | QR string format (public spec) | `paynet-duitnow-qr.md` |

Commercial and availability questions: `commercial@paynet.my`. Onboarding and company verification: `ask@paynet.my`. Older DuitNow v1/v2 or DOBW merchant documents: `solution.advisory@paynet.my` (stated on the docs site).

## Products and their rails

| Product | What it is | Typical payer flow |
|---|---|---|
| FPX | Direct debit from Malaysian internet banking, run on the `mepsfpx.com.my` hosts | Merchant form-POSTs an AR message, buyer logs in to the bank, FPX sends direct and indirect AC |
| DuitNow Online Banking/Wallets (DOBW) | RPP request-to-pay to a bank or wallet app | Acquirer gets `endToEndId` + signature, payer is redirected to the bank URL |
| DuitNow Pay | PayNet's acquirer-facing checkout over DOBW, Consent and AutoDebit, with a PayNet-hosted page or acquirer self-hosted page | `POST /v1/payment/intent` then redirect to `redirectUrl` |
| DuitNow Consent + AutoDebit | Payer authorises a mandate once; acquirer debits later | Consent registration then real-time debit |
| DuitNow Request | Request-to-pay that lands in the payer's banking inbox with a longer expiry | `POST /v3/intents/initiate` |
| DuitNow Reversal | Return funds of a completed credit transfer | `POST /v3/payments/reverse` |
| DuitNow QR | National QR (merchant-presented and consumer-presented) | Payer's app scans, issuer does account enquiry then QR payment |
| JomPAY | Bill payment network | Biller registration, validation, notification |
| MyDebit | Domestic debit card scheme; tokenisation APIs for acquirer/merchant | Token provisioning |

## Environments and hosts

| Area | UAT / sandbox | Production | Source |
|---|---|---|---|
| FPX merchant messages (AR) | `https://uat.mepsfpx.com.my/FPXMain/seller2DReceiver.jsp` | Not documented in official docs (checked 2026-10-21) | FPX PHP sample |
| FPX status enquiry (AE) | `https://uat.mepsfpx.com.my/FPXMain/sellerNVPTxnStatus.jsp` | Not documented | FPX PHP sample |
| FPX bank list (BE) | `https://uat.mepsfpx.com.my/FPXMain/RetrieveBankList` | Not documented | FPX PHP sample |
| FPX B2B2 | `https://uat.mepsfpx.com.my/FPXMain/B2B2Payment` | Not documented | FPX PHP sample |
| FPX T&C link | `https://uat.mepsfpx.com.my/FPXMain/termsAndConditions.jsp` | `https://www.mepsfpx.com.my/FPXMain/termsAndConditions.jsp` | `/docs/fpx/browser-redirection` |
| FPX bridge for DuitNow redirect | `https://uat.mepsfpx.com.my/FPXMain/r2pReceiver.jsp` | `https://www.mepsfpx.com.my/FPXMain/r2pReceiver.jsp` | `/docs/duitnow-pay/browser-redirection` |
| FPX Bank Webview API | `https://sandbox.api.paynet.my/fpx/banks/v1` | Not documented | Bank Webview OpenAPI |
| FPX simulators | `https://simulator.fpx.uat.inet.paynet.my/uatSeller1.10/sellermain.jsp`, `https://simulator.fpx.uat.inet.paynet.my/UatBuyerBankSim1.7/B2BAuth.jsp` | n/a | `/docs/fpx/resources` |
| DuitNow Pay API | Base host not documented publicly; given at onboarding. Sample `redirectUrl`: `https://duitnow-pay.uat.inet.paynet.my:8443/?sessionId=...` | Not documented | `/docs/duitnow-pay/integration/paynet-hosted-page/payment-intent` |
| RPP v3 Request-to-Pay API (internet) | `https://p1ice.rpp.uat.inet.paynet.my:8443` | `https://ice.rpp.prod.inet.paynet.my:8443` | `/docs/online-banking-wallet/connectivity-requirement` |
| RPP v3 Request-to-Pay API (MPLS) | `https://p1ice.rpp.uat.mpls.paynet.my:8443` | `https://icep2.rpp.prod.mpls.paynet.my` | same |
| RPP v3 Consent and AutoDebit API (internet) | `https://api.rpp.nextswitch.uat.inet.paynet.my` (443) | `https://api.rpp.nextswitch.prod.inet.paynet.my` (443) | same |
| RPP v3 Consent and AutoDebit API (MPLS) | `https://api.rpp.nextswitch.uat.mpls.paynet.my` | `https://api.rpp.nextswitch.prod.mpls.paynet.my` | same |
| RPP Phase-1 API (domestic and cross-border QR) | `https://api.uat.inet.paynet.my` | `https://api.prod.inet.paynet.my` | same |
| Financial Reports API | `https://api.reports.uat.inet.paynet.my`; simulator `https://certification.api.developer.inet.paynet.my/open-sim/pds` | `https://api.reports.paynet.my` | Reports OpenAPI |
| Merchant Profile API (beta) | `https://api.mssot.uat.inet.paynet.my`; simulator `https://certification.api.developer.inet.paynet.my/open-sim/mssot` | `https://api.mssot.prod.inet.paynet.my` | Merchant Profile OpenAPI |
| MyDebit tokenisation (mock) | `https://mock.developer.paynet.my/merchant-api/v2` | Not documented | TSP OpenAPI |
| JomPAY | Registration `https://brs.jompay.uat.mpls.paynet.my:8446/api/brs/{version}`; validation `https://sandbox.api.paynet.my/JomPAY/{version}/bank` | Not documented | JomPAY OpenAPI |
| PTS / ID Trust (fraud) | `https://openapi.ptis.uat.inet.paynet.my`, `https://openapi.nfp.uat.inet.paynet.my` | `https://openapi.ptis.prod.inet.paynet.my` | PTIS OpenAPI |

PayNet outbound (webhook source) IPs for RPP v3 over the internet: UAT `103.145.216.167` (plus `103.145.217.167` for Consent/AutoDebit); production `103.145.216.163` (primary), `103.145.217.120` (secondary); Phase-1 QR production `103.145.216.90`, `103.145.217.90`. PayNet says to obtain current IPs and domains in the Developer Portal onboarding stage or from `ask@paynet.my`, and that inbound hosts are behind Cloudflare. FPX is the opposite: merchants must NOT whitelist FPX IPs.

## Auth and signature by product

| Product | Request auth | Response / callback verification | Detail |
|---|---|---|---|
| FPX merchant messages | `fpx_checkSum` = hex uppercase of RSA-SHA1 signature over pipe-joined field values in field-name order, merchant private key | Verify AC `fpx_checkSum` with the FPX certificate (`fpxuat_current.cer`, then `fpxuat.cer`) | `paynet-fpx.md` |
| FPX Bank Webview | OAuth 2.0 plus `X-Signature` header (RSA-SHA1 over key fields) | Not documented | `paynet-fpx.md` |
| DuitNow Pay | `Authorization: <JWT>` signed RS256 with acquirer private key; claims `exp`, `iss`, `sub`, `jti`, `key`, `data` | Verify responses with PayNet's public key from the Developer Portal. Webhook signing is not documented. | `paynet-duitnow-pay.md` |
| RPP v3 (DuitNow, QR, Consent, AutoDebit, Reversal, Network Admin) | `Authorization: Bearer <JWS>`; claims `iss`, `iat`, `exp`, `key`, `jti`, `ds` (SHA-256 of minified body); header `kid` = certificate serial | Verify response JWS with PayNet certificate, check `ds` against the body | `paynet-rpp-v3.md` |
| Financial Reports, Merchant Profile | OAuth 2.0 client credentials at `/token`, `Bearer` token valid 1 hour; some endpoints add `X-Timestamp` and `X-Signature` = HMAC-SHA256(client_secret, timestamp) hex, plus `X-Fiid` | n/a | Below |

## Endpoint catalogue

Every endpoint in the published OpenAPI specs and docs pages, grouped by product. "WH" means a webhook the participant hosts and PayNet calls. Paths are relative to the hosts above.

### FPX merchant services (form POST or host-to-host, `application/x-www-form-urlencoded`)

| Method | Path / message | Purpose |
|---|---|---|
| POST (browser form) | `/FPXMain/seller2DReceiver.jsp` with `fpx_msgType=AR` | Authorisation request (B2C `fpx_msgToken=01`, B2B1 `02`) |
| WH POST | Merchant direct URL receives AC | Direct (host-to-host) authorisation response; reply `OK` |
| WH POST (browser) | Merchant indirect URL receives AC | Indirect authorisation response through the buyer's browser |
| POST | `/FPXMain/sellerNVPTxnStatus.jsp` with `fpx_msgType=AE` | Status enquiry; returns AC fields |
| POST | `/FPXMain/RetrieveBankList` with `fpx_msgType=BE` | Bank list; returns BC |
| POST | `/FPXMain/B2B2Payment` | B2B2 batch AR over MPLS |
| POST | AF message (host-to-host) | Bulk refund submission; returns AB |
| POST | AQ message | Bulk refund status; returns AB |
| POST | AX message | Merchant cancellation request; returns AXA |

### FPX Bank Webview (banks only) `https://sandbox.api.paynet.my/fpx/banks/v1`

| Method | Path | Purpose |
|---|---|---|
| GET | `/pending/banks/{bankId}/dcl` | DCL (debit) pending inquiry |
| GET | `/pending/banks/{bankId}/dcl/model/{model}` | DCL inquiry by model |
| GET | `/pending/banks/{bankId}/ccl` | CCL (credit) pending inquiry |
| GET | `/pending/banks/{bankId}/ccl/model/{model}` | CCL inquiry by model |
| POST | `/pending/banks/{bankId}/transactions/{transactionId}/actions/{action}/dcl` | DCL authorise |
| POST | `/pending/banks/{bankId}/transactions/{transactionId}/actions/{action}/ccl` | CCL authorise |
| GET | `/downtime/banks/{bankId}/model/{model}` | Downtime inquiry |
| POST | `/downtime/banks/{bankId}/model/{model}` | Downtime submit (`periodType`, `startDate`, `startTime`, `endDate`, `endTime`) |
| DELETE | `/downtime/banks/{bankId}/schedulers/{schedulerId}` | Downtime cancel |
| POST / WH | `/notifications/downtime` | Downtime notification |

### DuitNow Pay (acquirer)

| Method | Path | Purpose |
|---|---|---|
| POST | `/v1/payment/intent` | PayNet-hosted page: one-time payment (`dataType` `01`) or save payment method (`02`); returns `redirectUrl` |
| GET | `/v2/bw/banks` | Bank list for self-hosted page (call every 30 minutes) |
| POST | `/v1/bw/checkout` | Self-hosted one-time checkout (RTP 861); returns `endToEndId` + `endToEndIdSignature` |
| POST | `/v1/bw/consent` | Self-hosted save payment method (consent 811) |
| POST | `/v1/bw/autodebit` | Debit an active consent |
| POST | `/v1/bw/refund` | Refund (within 90 days) |
| GET | `/v1/bw/refund?refundId=` | Refund status (once, at least 1 hour after the request) |
| GET | `/v1/bw/rtp?checkoutId=` | Payment status v1 (30 s rate limit) |
| GET | `/v2/bw/checkout-status?paymentMethod=&checkoutId=` | Payment status v2, also AutoDebit (30 s per transaction) |
| GET | `/v1/bw/checkout?endToEndId=` | Checkout details (map `endToEndId` back to `checkoutId`) |
| GET | `/v1/bw/consent?consentId=` | Saved payment method (consent) details |
| GET | `/v1/bw/consent/request?checkoutId=` | Saved payment registration status (30 s rate limit) |
| DELETE | `/v1/bw/consent?consentId=` | Terminate consent |
| WH | Update Checkout Details | Maps `checkoutId` to `rtpEndToEndId` or `consentEndToEndId` + `consentId` |
| WH | Update Payment Status | Payment result (success, and a rejected variant) |
| WH | Update Consent Details | Consent result |
| WH | Update Refund Status | Refund result |
| GET | `/v1/bw/autodebit-batch-key` | Temporary storage credentials (1 hour) for AutoDebit batch upload and download |
| WH | Update AutoDebit Batch Status | Batch file processing result |
| Batch | AutoDebit batch file and report list | `/docs/duitnow-pay/integration/duitnow-autodebit-batch`, `/docs/duitnow-pay/batchfilelist`, `/docs/duitnow-pay/reportlist` (file formats; summarised in `paynet-duitnow-pay.md`) |

### RPP DuitNow v3, acquirer side

| Method | Path | Spec | Purpose (RPP transaction codes) |
|---|---|---|---|
| WH POST | `/webhooks/v3/payments/transfer-ct` | Online Banking, Request, Transfer (acquirer) | Receive credit transfer (010, 110, 060, 070) |
| WH POST | `/webhooks/v3/intents/reverse/initiate` | Online Banking, Request (acquirer) | Receive refund intent (854) |
| WH POST | `/webhooks/v3/intents/reject` | Request (acquirer) | Intent rejected (852) |
| WH POST | `/webhooks/v3/accounts/enquire-ct` | Transfer (acquirer), Consent (issuer) | Account enquiry for credit transfer (510) |
| POST | `/v3/proxies/enquire` | Request, Consent, Transfer, NAD | Proxy enquiry (610, 620) |
| POST | `/v3/accounts/enquire-ct` | Request, Consent, Transfer | Account enquiry (510) |
| POST | `/v3/intents/initiate` | Request (acquirer) | Request-to-pay by proxy (851) or account (853) |
| POST | `/v3/payments/reverse` | Reversal (acquirer) | Reverse a credit transfer (011, 012) |
| POST | `/v3/auto-debit/pre-auth` | AutoDebit (acquirer) | Pre-authorisation (871) |
| POST | `/v3/payments/debit-ct` | AutoDebit (acquirer) | Real-time debit (210, 230) |
| WH POST | `/webhooks/v3/auto-debit/direct-debit` | AutoDebit (acquirer) | Receive direct debit credit (080) |
| POST | `/v3/consents/register` | Consent | Consent registration (801, 802, 821, 823) |
| POST | `/v3/consents/enquire` | Consent | Consent enquiry (751, 752) |
| POST | `/v3/consents/enquire-reg` | Consent | Retrieve consent detail (812) |
| POST | `/v3/consents/{consentId}/status-updates` | Consent | Consent status update (803, 813, 822) |
| POST | `/v3/consents/{consentId}/terminate` | Consent | Consent termination (763, 764) |
| WH POST | `/webhooks/v3/consents/register` | Consent | Consent registration notice (801, 802, 811, 821) |
| WH POST | `/webhooks/v3/consents/status-updates` | Consent | Consent update notice (803, 822) |
| WH POST | `/webhooks/v3/consents/reject` | Consent (acquirer) | Consent rejected (804) |
| WH POST | `/webhooks/v3/accounts/enquire-xc` | MPM QR, Cross-border inbound QR (acquirer) | QR account enquiry (520, 521) |
| WH POST | `/webhooks/v3/payments/transfer-xc` | MPM QR, Cross-border inbound QR (acquirer) | QR payment (030, 031, 040) |
| POST | `/v3/transactions/enquire` | CPM QR acquirer, MPM QR issuer, Transfer issuer | Transaction enquiry (630), one transaction per call |
| POST | `/v3/payments/cpmqr` | Old-site CPM QR acquirer spec | Debit request for consumer-presented QR (direct or pre-auth) |
| POST | `/v3/admin/sign-on`, `/v3/admin/sign-off`, `/v3/admin/echo` | Network Administration | Connectivity (000) |
| WH POST | `/webhooks/v3/admin/event` | Network Administration | System event notification |

### RPP DuitNow v3, issuer side (catalogue only)

| Method | Path | Purpose |
|---|---|---|
| POST | `/v3/payments/transfer-ct` | Credit transfer |
| POST | `/v3/intents/enquire` | Retrieve an intent |
| POST | `/v3/intents/{transactionId}/update` | Update intent status |
| POST | `/v3/intents/{transactionId}/forward` | Forward an intent |
| POST | `/v3/intents/reject` | Reject intent |
| WH POST | `/webhooks/v3/intents/initiate`, `/webhooks/v3/intents/forward` | Intent received / forwarded |
| WH POST | `/webhooks/v3/payments/reverse` | Reversal received |
| WH POST | `/webhooks/v3/auto-debit/pre-auth`, `/webhooks/v3/auto-debit/cancel`, `/webhooks/v3/payments/debit-ct` | AutoDebit issuer side |
| POST | `/v3/consents/merchant-product-list`, `/v3/consents/{consentId}/amendments`, `/v3/consents/{consentId}/transfer`, `/v3/consents/reject` | Consent issuer side |
| POST | `/v3/accounts/enquire-xc`, `/v3/payments/transfer-xc` | MPM QR issuer: enquiry and payment |
| POST | `/v3/accounts/enquire-xc-cross-border`, `/v3/payments/transfer-xc-cross-border` | Cross-border outbound QR issuer |
| WH POST | `/webhooks/v3/debits/authorize`, `/webhooks/v3/debits/initiate` | CPM QR issuer (new site); old site `/webhooks/v3/payments/cpmqr` |
| POST | `/v3/proxies/register`, `/v3/proxies/{registrationId}/update`, `/v3/proxies/{registrationId}/transfer`; WH `/webhooks/v3/proxies/transfer` | NAD proxy maintenance |

### Operations, reports and other

| Method | Path | Purpose |
|---|---|---|
| POST | `/token` (reports host) | OAuth 2.0 client credentials token (form body `grant_type=client_credentials`, `client_id`, `client_secret`), 1 hour |
| GET | `/v1/reports/types` | List v1 report types (NextSwitch) |
| POST | `/v1/reports/download` | Download v1 report |
| POST | `/v2/reports/query` | GraphQL report query: daily itemised (I01), cycle, daily and monthly summary data (report IDs named in errors: S01, S02, D01, I01, M01) |
| GET | `/v2/files/{file_id}` | Download an itemised report file (one-time URL) |
| GET / PUT | `/v1/merchants` | Merchant Profile list / create-update (beta, "Q3 2025") |
| POST | `/token/provision`, `/token/data`, `/token/suspend`, `/token/resume`, `/token/delete`, `/token/status`, `/card/metadata`, `/notification/details` | MyDebit tokenisation for acquirer/merchant |
| WH POST | `/CNP/MY/Notification/TokenUpdatedStatus`, `/CNP/MY/Notification/CardMetaDataUpdatedNotification` | MyDebit token notifications to merchant |
| POST | `/card/suspend`, `/card/resume`, `/card/delete`, `/card/pan-update`; WH `/CNP/MY/Notification/TokenProvisionStatus` | MyDebit tokenisation for issuer |
| POST | `/registerbiller` | JomPAY biller registration |
| POST | `/validatepayment`, `/notifycompletion` | JomPAY biller validation |
| WH POST | `/JomPAY/MY/Notification/BillNotification` | JomPAY biller payment notification |
| POST | `/pts-api/v1/transactions/screen`, `/pts-api/v1/transactions/feedback` | Pre-transaction fraud screening |
| POST | `/oauth2/token`, `/oauth2/token/refresh`; GET `/nfp-api/v1/server/ping`; POST `/nfp-api/v1/server/mule-check` | ID Trust / National Fraud Portal |

## Common rules across products

- TLS 1.2 only (FPX auth page and v3 construct-header page).
- Amounts: FPX `fpx_txnAmount` as a decimal string in MYR (sample `100.00`); DuitNow Pay `amount` string with two decimals, max 18 chars; v3 `interbankSettlementAmount.value` number in MYR, up to 2 decimals; QR tag `54` digits with optional `.`. Currency is MYR (`458` in QR).
- Rate limits apply per route and globally: HTTP 429 with `{"error":"Quota has been exceeded"}`. PayNet strongly discourages load testing against the sandbox, and recommends limiting concurrent batch enquiries and using separate Developer Portal projects per team (`/docs/API/rate-limit`).
- API version goes in the URL (`{version}`) where supported (`/docs/API/versioning`); v3 uses an optional `x-api-version` header and may return `x-deprecated`, `x-deprecation-date`, `x-sunset-date` headers.
- Webhook configuration for v3 projects: Developer Portal project, Edit profile, Webhook tab, with source IPs listed there.
- Postman collections can be exported from each API reference page on the docs site.

## Financial Reports API (operations)

Auth: `POST {reports host}/token` with `Accept: application/json`, `Content-Type: application/x-www-form-urlencoded`, body `grant_type=client_credentials&client_id=<CLIENT_ID>&client_secret=<CLIENT_SECRET>`. Response `access_token`, `token_type` `Bearer`, `expires_in` `3600`.

Signature for endpoints that require it: `X-Timestamp` = Unix seconds as a string, `X-Signature` = hex HMAC-SHA256 with key `client_secret` over the timestamp string. `X-Fiid` carries the FI ID. Errors: 400 for missing `X-Fiid`, timestamp or signature, "Timestamp not within valid range", "Invalid signature"; 401 missing authorization; 403 forbidden or wrong `X-Fiid`; E0016 expired one-time link.

Flow: itemised report (I01) is asynchronous: send the query, poll the same query until status `ready` (statuses `new`, `processing`, `ready`, `error`), then download once from the given URL. Summary reports return JSON directly. Retention 90 days (E0001), monthly 3 months (E0011). Rate limit E0014: retry after 1 minute. Only one product per GraphQL query (E0015). Full error list in `paynet-codes.md`.

## Known gaps and conflicts (state these to the user)

- DuitNow Pay API base URLs are not published; they come from onboarding.
- FPX production endpoint URLs for AR, AE and BE are not published in the docs or samples.
- The FPX AR/AC field length and format table is not on the public docs site; field names come from the official sample source code.
- FPX `fpx_version`: the official sample uses `6.0`; the docs BE example uses `7.0`. Use the version PayNet assigns.
- v3 JWS algorithm: docs text says RS512; the docs' own Java, Python, PHP and Go samples use RS256.
- DuitNow Pay webhook authentication is not documented (only "verify API response" with PayNet's public key). Use IP allowlisting and requery the status API before fulfilment.
- Several v3 specs are marked beta or "specification review"; access needs PayNet approval.
