# Adyen API Reference (entry point)

Distilled from Adyen's official docs (`https://docs.adyen.com/`) and official OpenAPI and SDK sources on `https://github.com/Adyen`, for a Malaysian merchant accepting MYR online. Checked 2026-10-21. The docs use **Checkout API v72**.

Read this file first. Detail lives in topic files:

| File | Use for |
| --- | --- |
| `adyen-checkout.md` | Sessions flow, Hosted Checkout, Advanced flow, Drop-in config, actions, result codes, 3DS2 |
| `adyen-webhooks.md` | Webhook setup, payload, HMAC verification, event codes, retries |
| `adyen-modifications.md` | Capture, refund, cancel, reversal, amount update, lifecycle statuses, disputes API |
| `adyen-payment-methods-my.md` | FPX (18 issuers), DuitNow, Touch 'n Go, GrabPay, Boost, Atome, Alipay, Alipay+, WeChat Pay, cards for MY |
| `adyen-tokenization-links.md` | Tokens (recurring), payment links |
| `adyen-errors-testing.md` | Test cards, 3DS test data, result triggers, refusal reasons, all error codes |
| `adyen-fields.md` | Full request and response field tables generated from the official OpenAPI |
| `account-setup.md` | Account, credentials, Customer Area paths, env vars, go live, payouts, reports |
| `source-map.md` | Every page used, coverage |

## Environments

Source: `https://docs.adyen.com/development-resources/live-endpoints`

| | Test | Live |
| --- | --- | --- |
| Customer Area | `https://ca-test.adyen.com/` | `https://ca-live.adyen.com/` |
| Checkout API | `https://checkout-test.adyen.com/v72/{method}` | `https://<PREFIX>-checkout-live.adyenpayments.com/checkout/v72/{method}` |
| Classic Payments API | `https://pal-test.adyen.com/pal/servlet/Payment/{version}/{method}` | `https://<PREFIX>-pal-live.adyenpayments.com/pal/servlet/Payment/{version}/{method}` |
| Classic Recurring API | `https://pal-test.adyen.com/pal/servlet/Recurring/v68/{method}` | same `<PREFIX>-pal-live` host pattern |
| Disputes API | `https://ca-test.adyen.com/ca/services/DisputeService/v30/{method}` | live Customer Area host |
| Drop-in `environment` | `test` | `live` (EU), `live-us`, `live-au`, `live-nea`, `live-in`, `live-apse` |
| Client key prefix | `test_` | `live_` |

- `<PREFIX>`: live Customer Area > **Developers** > **API URLs** > **Prefix** (random hex plus company name). Never call live with the test host.
- Location-based live endpoints (US, AU, APSE and others) are set up through Support. Which region a Malaysian account uses is not stated in the docs (checked 2026-10-21); read it from **API URLs**. All calls for one payment must use the same region.
- Test and live are separate accounts. Settings, users, webhooks, API keys, client keys and HMAC keys are not copied to live.
- Connection: Adyen closes idle keepalive connections after 60 seconds; set client keepalive to 30 to 50 seconds. MTU 1500 recommended.

## Authentication

Sources: `https://docs.adyen.com/development-resources/api-authentication`, `https://docs.adyen.com/development-resources/api-credentials`, `https://docs.adyen.com/development-resources/client-side-authentication`

- Server: header `x-API-key: <API_KEY>` (preferred), or HTTP Basic with `ws_<ID>@Company.<COMPANY_ACCOUNT>` and its password.
- API credential: Customer Area > **Developers** > **API credentials** > pick or create (`ws_...`) > **Server settings** > **Authentication** > **API key** > Generate. Copy it then; it cannot be shown again. After regenerating, the old key keeps working for 24 hours. A new Basic auth password replaces the old one at once. Credentials cannot be deleted, only deactivated. Optional allowed IP ranges.
- Roles on the credential: **Checkout webservice role** and **Merchant PAL webservice role** (payments); **Merchant Recurring role** (tokens); **Checkout encrypted cardholder data** (Drop-in card data); **API PCI Payments role** only for raw card data (SAQ D). Source: `https://docs.adyen.com/development-resources/api-credentials/roles`
- Client key (frontend, not secret): same credential > **Client settings** > **Authentication** > **Client key**. Add **Allowed origins** (HTTPS in live; `http://localhost:<port>` allowed in test; wildcards allowed). A regenerated client key leaves the old one valid for 24 hours. The client key is not used for server API calls.
- User roles needed to manage credentials: **Merchant admin** or **Manage API credentials**.

## Idempotency And Retries

Source: `https://docs.adyen.com/development-resources/api-idempotency`

- Header `idempotency-key: <KEY>` on POST; max 64 characters; UUID v4 recommended. Scope: company account. Kept for at least 7 days (up to 14). Not shared across regions.
- Same key plus same request returns the stored response. Concurrent duplicate: HTTP 409 or 422 with `errorCode` `704`. Store unavailable: 503 with `703`.
- Retry only with the same key. Response header `transient-error: true` marks a safe retry. Use exponential backoff.
- Max request size 10 MB (HTTP 413). HTTP 429 or `errorCode` `705`: back off with jitter.

## Rate Limits, IP Allowlists

- Numeric API rate limits: Not documented in official docs (checked 2026-10-21). Handle 429 / `705`.
- Source: `https://docs.adyen.com/development-resources/security/integration-security/allowlisting`. Recommended: domain-based allowlisting of `adyen.com` and `adyenpayments.com`. Do not hardcode IPs or use custom DNS records (SNI mismatch).
- Outgoing (your server to Adyen) IP ranges published: `82.199.87.128/26`, `82.199.90.136/29`, `82.199.90.160/27`, `213.52.172.0/25`, `62.146.248.0/21`, `85.184.228.0/22`, `91.212.42.0/24`, `135.84.148.0/22`, `147.12.16.0/20`, `185.101.196.0/22`, `145.63.64.0/18`. Machine-readable: `https://docs.adyen.com/ip-ranges.txt`, `https://docs.adyen.com/ip-ranges.json`. Always resolve endpoints by DNS.
- Incoming (Adyen webhooks to you): no fixed list; resolve `dig +short out.adyen.com` regularly (hourly suggested on the webhook security page). Allowlisting is hardening only; keep HMAC verification.

## Amounts And Currency

Source: `https://docs.adyen.com/development-resources/currency-codes`

- `amount` = `{ "currency": "MYR", "value": <integer minor units> }`. MYR has 2 decimals: RM 1.00 = `100`, RM 49.90 = `4990`. Never send decimals.
- Webhooks return the same shape; compare integers.
- MYR payouts go only to a local Malaysian bank account (no SWIFT payouts for MYR). Source: `https://docs.adyen.com/account/supported-currencies`

## Endpoint Catalogue

Checkout API v72 (base above; all from `json/CheckoutService-v72.json`):

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/sessions` | Create a payment session (Sessions flow; `mode: "hosted"` for Hosted Checkout) |
| GET | `/sessions/{sessionId}?sessionResult=` | Get the result of a payment session |
| PATCH | `/sessions/{sessionId}` | Update a session (amount) when created with `payable: false` |
| POST | `/paymentMethods` | List available payment methods (and FPX issuers, stored methods) |
| POST | `/payments` | Start a payment (Advanced flow, token payments) |
| POST | `/payments/details` | Submit redirect or 3DS details |
| POST | `/payments/{paymentPspReference}/captures` | Capture an authorised payment |
| POST | `/payments/{paymentPspReference}/refunds` | Refund a captured payment |
| POST | `/payments/{paymentPspReference}/cancels` | Cancel an authorised payment |
| POST | `/cancels` | Cancel using your own payment reference (24 hours) |
| POST | `/payments/{paymentPspReference}/reversals` | Cancel or refund |
| POST | `/payments/{paymentPspReference}/amountUpdates` | Update an authorised amount (cards, pre-auth) |
| POST | `/paymentLinks` | Create a payment link |
| GET | `/paymentLinks/{linkId}` | Get a payment link (status) |
| PATCH | `/paymentLinks/{linkId}` | Update payment link status (force `expired`) |
| GET | `/storedPaymentMethods?shopperReference=&merchantAccount=` | List tokens |
| POST | `/storedPaymentMethods` | Create a token without a payment |
| DELETE | `/storedPaymentMethods/{storedPaymentMethodId}?shopperReference=&merchantAccount=` | Delete a token |
| POST | `/cardDetails` | Card brands and details from a BIN |
| POST | `/applePay/sessions` | Get an Apple Pay session (own certificate setups) |
| POST | `/orders` | Create an order (partial payments) |
| POST | `/orders/cancel` | Cancel an order |
| POST | `/paymentMethods/balance` | Gift card balance |
| POST | `/donationCampaigns` | List donation campaigns (Adyen Giving) |
| POST | `/donations` | Make a donation |
| POST | `/forward` | Forward stored card details to a third party |
| POST | `/paypal/updateOrder` | PayPal Express order update |
| POST | `/calculateTax` | Calculate tax for a transaction |
| POST | `/validateShopperId` | Validate shopper id |
| POST | `/verifyShopperTaxIdentification` | Verify shopper tax id |
| POST | `/originKeys` | Deprecated (replaced by client key) |

Other APIs used by a Malaysian online merchant:

| API | Method | Path | Purpose |
| --- | --- | --- | --- |
| Standard webhook (you host) | POST | your URL | `NotificationRequest` with `notificationItems[]` (`adyen-webhooks.md`) |
| Tokenization webhook (you host) | POST | your URL | `recurring.token.created`, `.updated`, `.disabled`, `.alreadyExisting` |
| Disputes API v30 | POST | `/retrieveApplicableDefenseReasons`, `/supplyDefenseDocument`, `/deleteDisputeDefenseDocument`, `/defendDispute`, `/acceptDispute` | Card chargeback defence |
| Recurring API v68 (classic) | POST | `/disable`, `/listRecurringDetails`, `/notifyShopper`, `/scheduleAccountUpdater` (`/createPermit`, `/disablePermit` deprecated) | Legacy token management |
| Report download | GET | URL from `REPORT_AVAILABLE` `reason` | Settlement and other reports, Report user credential |

Adyen MCP server (alpha, official): `npx -y @adyen/mcp --adyenApiKey=<API_KEY> --env=TEST` (live adds `--env=LIVE --livePrefix=<PREFIX>`); supports Checkout and Management API tools. Source: `https://docs.adyen.com/development-resources/mcp-server`, `https://github.com/Adyen/adyen-mcp`.

Out of scope here: Management API, Balance Platform / Adyen for Platforms, POS Terminal API, classic Payments API payment flows (see `source-map.md` Coverage).

## Flow Summary

1. Create a local order (`pending`, amount in sen, `MYR`).
2. Server: `POST /sessions` (or `/paymentMethods` + `/payments`) with `merchantAccount`, `amount`, `reference` = order id, `returnUrl`, `countryCode: "MY"`, `idempotency-key`.
3. Browser: Drop-in with `session.id` + `sessionData` (or redirect to Hosted Checkout `url`).
4. Redirect methods (FPX, TnG, GrabPay, Boost, Atome) return to `returnUrl` with `redirectResult` (or `sessionResult`). Submit details; show the result.
5. Webhook `AUTHORISATION`: verify HMAC, store, return 2xx within 10 seconds, then process.
6. Mark paid only if `success == "true"`, `merchantReference` matches, `amount.value` and `amount.currency` match, `merchantAccountCode` matches, `live` matches environment. Idempotent on `pspReference` + `eventCode`.
7. Capture (only if manual), refund, cancel via modification endpoints; outcomes arrive as webhooks.

## Key Request Fields

Full tables: `adyen-fields.md`. Most used:

`POST /sessions` (`CreateCheckoutSessionRequest`):

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `merchantAccount` | string | yes | Merchant account code |
| `amount` | Amount | yes | `currency` + integer `value` |
| `reference` | string | yes | Your order id, max 80 |
| `returnUrl` | string | yes | Max 1024 |
| `countryCode` | string | | `MY` |
| `mode` | string | | `embedded` (default) or `hosted` |
| `expiresAt` | date-time | | Default 1 hour, max 24 hours |
| `shopperReference` | string | | Tokens; min 3 |
| `storePaymentMethodMode` | string | | `disabled`, `askForConsent`, `enabled` |
| `recurringProcessingModel` | string | | `CardOnFile`, `Subscription`, `UnscheduledCardOnFile` |
| `allowedPaymentMethods` / `blockedPaymentMethods` | array | | Payment method types |
| `lineItems`, `shopperEmail`, `shopperName`, `telephoneNumber`, `billingAddress`, `deliveryAddress` | | | Required by Atome and GrabPay PayLater |
| `captureDelayHours` | integer | | Max 672 |
| `metadata` | object | | Key max 20, value max 80 |

`POST /payments` (`PaymentRequest`): the same core fields plus `paymentMethod` (required; `state.data.paymentMethod`, for FPX `{ "type": "molpay_ebanking_fpx_MY", "issuer": "<ISSUER_ID>" }`), `browserInfo`, `channel`, `origin`, `shopperIP`, `shopperInteraction`, `storePaymentMethod`, `authenticationData`, `additionalData`.

`PaymentResponse`: `resultCode`, `action`, `pspReference`, `merchantReference`, `refusalReason`, `refusalReasonCode`, `amount`, `additionalData`, `paymentMethod`.

Modification requests: `merchantAccount` (required), `amount` (captures, refunds, amount updates; required), `reference`, `merchantRefundReason` (refunds). Responses: `status: "received"`, `pspReference`, `paymentPspReference`.

## Webhook Signature (HMAC)

Source: `https://docs.adyen.com/development-resources/webhooks/secure-webhooks/verify-hmac-signatures`; confirmed against `https://github.com/Adyen/adyen-node-api-library/blob/main/src/utils/hmacValidator.ts`.

- Signature location: `NotificationRequestItem.additionalData.hmacSignature` (Base64).
- String to sign (8 fields, colon-separated, missing = empty string):
  `pspReference:originalReference:merchantAccountCode:merchantReference:amount.value:amount.currency:eventCode:success`
- Key: HMAC key from Customer Area (per webhook endpoint), **hex-decoded** to bytes.
- Algorithm: Base64(HMAC-SHA256(key bytes, UTF-8 string)). Compare in constant time.

```text
key   = hexDecode(<HMAC_KEY>)
data  = join(":", [pspReference, originalReference or "", merchantAccountCode,
                   merchantReference, str(amount.value), amount.currency,
                   eventCode, success])
valid = constantTimeEqual(base64(HMAC_SHA256(key, utf8(data))), item.additionalData.hmacSignature)
```

Official test vector: key `44782DEF547AAA06C910C43932B1EB0C71FC68D9D0C057550C48EC2ACF6BA056`, data `7914073381342284::TestMerchant:TestPayment-1407325143704:1130:EUR:AUTHORISATION:true`, expected `coqCmt/IZ4E3CzPvMY8zTjQVL5hYJUiBRg8UU+iCWo0=` (recomputed 2026-10-21, matches).

Other webhook types (Recurring tokens life cycle, Management) sign the **raw body** instead: header `hmacsignature`, header `protocol: HmacSHA256`, same hex key handling.

## Statuses

- `resultCode` (API, UX only): `Authorised`, `Pending`, `Received`, `Refused`, `Cancelled`, `Error`, `RedirectShopper`, `IdentifyShopper`, `ChallengeShopper`, `PresentToShopper`, `PartiallyAuthorised`, `AuthenticationFinished`, `AuthenticationNotRequired`.
- Session status: `completed`, `paymentPending`, `canceled`, `expired`.
- Payment link status: `active`, `completed`, `expired`, `paymentPending`.
- Webhook `success`: `"true"` / `"false"` per `eventCode`.
- Customer Area lifecycle: `adyen-modifications.md`.

## Requery

There is no Checkout endpoint to fetch a payment by `pspReference` or `reference` (checked 2026-10-21). Use webhooks (30-day retries, manual retry in Customer Area), `GET /sessions/{id}?sessionResult=` at return time, `GET /paymentLinks/{linkId}`, and reports. Details: `adyen-checkout.md`.

## Refunds, Capture, Cancel

Modification endpoints above. Async; outcome webhooks `REFUND`, `REFUND_FAILED`, `REFUNDED_REVERSED`, `CAPTURE`, `CAPTURE_FAILED`, `CANCELLATION`, `TECHNICAL_CANCEL`, `CANCEL_OR_REFUND`. Boost: full refunds only. GrabPay refunds need `reference` (max 32, `[A-Za-z0-9_-]`). Details: `adyen-modifications.md`.

## Recurring

Cards only for Malaysia (local methods do not support tokens). `shopperReference` + `storePaymentMethodMode` / `storePaymentMethod`, token from `recurring.token.created`, pay with `paymentMethod.storedPaymentMethodId` and `shopperInteraction: "ContAuth"`. Details: `adyen-tokenization-links.md`.

## Sandbox Data (test only)

- Card `4111 1111 4555 1142`, 03/2030, CVC 737. 3DS2 challenge card `4212 3456 7891 0006`, password `password`.
- `holderName` values trigger results (`DECLINED`, `PENDING`, `capture failed`, `refund failed`).
- Malaysian local methods use the Adyen Payment Simulator once added in the test Customer Area.
- Full lists: `adyen-errors-testing.md`.

## Common Errors

| Symptom | Cause | Fix |
| --- | --- | --- |
| 401 | Wrong or missing `x-API-key`, test key on live host | Use the key from the same environment |
| 403 `901` Invalid Merchant Account | Credential has no access to that merchant account | Check merchant account code and credential account access |
| 403 / `010` Not allowed | Missing role | Add Checkout webservice / Merchant PAL roles |
| 422 `14_xxx` | Validation (for example `14_0390` missing `redirectResult`) | Read `errorCode` in `adyen-errors-testing.md` |
| FPX missing in Drop-in | Not enabled, or `countryCode`/`currency` not `MY`/`MYR` | Add FPX in Customer Area; send `MY` + `MYR` |
| Webhook HMAC mismatch | Key not hex-decoded, wrong field order, `amount.value` formatted, test key in live | Follow the HMAC section exactly |
| Drop-in fails to load on your domain | Client key allowed origins do not include the page origin | Add the origin under Client settings > Allowed origins |
| Live call fails with test URL | Live needs the `<PREFIX>` host | Use the URL from **Developers** > **API URLs** |
