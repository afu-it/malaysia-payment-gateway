# Antom API Reference

Entry point for Antom (Ant International) merchant payment integrations, scoped to what a Malaysian merchant uses. Checked against official docs on 2026-10-21.

## Reference Files

| File | Use for |
| --- | --- |
| `antom-checkout.md` | pay, createPaymentSession, consult field tables; Checkout Page, Element, API-only modes |
| `antom-payment-ops.md` | inquiryPayment, capture, cancel, refund, inquiryRefund field tables and rules |
| `antom-notifications.md` | notifyPayment, notifyCapture, notifyRefund fields, verification, acknowledgement, retries |
| `antom-errors.md` | Result and error codes per API |
| `antom-codes.md` | Payment method enums, Malaysian method limits, FPX bank codes, card brands, statuses, locales |
| `account-setup.md` | Onboarding, keys, sandbox, go-live, settlement, pricing location |
| `source-map.md` | Every page used, grouped by topic, plus coverage |

## Source URLs

- Docs index for agents: `https://docs.antom.com/llms.txt`
- API overview (message structure, headers): `https://docs.antom.com/ac/ams/api_fund.md`
- Antom APIs (domains, sandbox paths): `https://docs.antom.com/ac/ams/api.md`
- Signing and verification: `https://docs.antom.com/ac/ams/digital_signature.md`
- Idempotency: `https://docs.antom.com/ac/ams/idempotency.md`
- Message encoding: `https://docs.antom.com/ac/ams/me.md`
- Amount object rules: `https://docs.antom.com/ac/ref/cc.md`
- Server SDKs: `https://docs.antom.com/ac/sdks/server_sdks.md`

## Environments And Domains

All APIs use HTTPS `POST` with a JSON body. API version is `v1` and sits in the path.

| Region | Domain |
| --- | --- |
| Asia | `https://open-sea-global.alipay.com` (recommended) or `https://open-sea.alipay.com` |
| North America (non-US merchants) | `https://open-na-global.alipay.com` (recommended) or `https://open-na.alipay.com` |
| North America (US merchants) | `https://open.antglobal-us.com` |
| Europe | `https://open-de-global.alipay.com`; `https://open-eu.alipay.com` is in-store only |

Use the gateway URL shown in Antom Dashboard > Developer > Quick start > Integration resources and tools > Integration resources. The Java SDK constant `EndPointConstants.SG` is `https://open-sea-global.alipay.com`.

| Environment | Path |
| --- | --- |
| Production | `https://<domain>/ams/api/<endpoint>` |
| Sandbox (online payments) | `https://<domain>/ams/sandbox/api/<endpoint>` |
| Sandbox (in-store payments) | `https://<domain>/ams/api/<endpoint>` |

- Sandbox client IDs start with `SANDBOX_`. The official Python SDK switches to sandbox mode when the client ID starts with `SANDBOX_` and replaces `/ams/api` with `/ams/sandbox/api` in the path before signing.
- The general FAQ says to remove `sandbox` from the path when going live: `/ams/sandbox/api/v1/payments/pay` becomes `/ams/api/v1/payments/pay`.
- Conflict to flag: the signature guide examples use client ID `SANDBOX_5X00000000000000` with path `/ams/api/v1/payments/pay`. Follow the SDK and api.md rule (sandbox path) and confirm with the Dashboard integration resources.
- Test mode and live mode need different API keys.

## Request Headers

| Header | Required | Notes |
| --- | --- | --- |
| `signature` | Yes | `algorithm=RSA256, keyVersion=1, signature=<generated_signature>` |
| `Content-Type` | No | `application/json; charset=UTF-8` |
| `client-id` | Yes | Your client ID from the Dashboard |
| `request-time` | Yes | Docs: timestamp accurate to milliseconds, for example `1685599933871` |
| `agent-token` | No | ISV use only |

- Header names are case-insensitive.
- The official Python SDK sends `Request-Time` as an ISO 8601 string, not milliseconds. Whatever format you send, sign the exact same string.

## Response Headers And Body

- Response headers: `signature`, `client-id`, `response-time` (ISO 8601, for example `2019-05-28T12:12:14+08:00`).
- Every response body has `result` with `resultStatus` (`S`, `F`, `U`), `resultCode` (max 64), `resultMessage` (max 256).
- `result` is the API call result. It is not always the payment result. For APMs, check `paymentStatus` in inquiryPayment; for cards, Apple Pay, and Google Pay, ship on capture result.
- The signature may be absent when Antom returns `SIGNATURE_INVALID`. The Python SDK accepts an unsigned response only when `resultStatus` is `F` or `U`.

## Signature Method

### Sign a request

1. Build the content to sign (one space between method and URI, a newline, then dot-separated values):

```text
POST <http-uri>
<client-id>.<request-time>.<request-body>
```

`http-uri` is the path plus query string, for example `/ams/api/v1/payments/pay` (or the sandbox path you actually call).

2. Generate the signature:

```text
generated_signature = urlEncode(base64Encode(sha256withRSA(content_to_be_signed, <MERCHANT_PRIVATE_KEY>)))
```

3. Send header `signature: algorithm=RSA256, keyVersion=1, signature=<generated_signature>`.

The Java sample formats the content with `String.format("POST %s\n%s.%s.%s", uri, clientId, requestTime, body)`. The Python SDK uses PKCS1 v1.5 padding with SHA-256 and `quote_plus` for URL encoding.

### Verify a response

1. Get the Antom public key from Dashboard > Developer > Quick start > Integration resources and tools > Integration resources. It appears only after you upload or generate your own public key.
2. Build `POST <http-uri>\n<client-id>.<response-time>.<response-body>` using the raw response body. Do not parse and re-serialize JSON.
3. Take the value after `signature=` in the response `signature` header.
4. Verify: `sha256withRSA_verify(base64Decode(urlDecode(target_signature)), content_to_be_validated, <ANTOM_PUBLIC_KEY>)`.

### Verify a notification

1. Read `client-id`, `request-time`, and `signature` from the notification request headers.
2. Build `POST <http-uri>\n<client-id>.<request-time>.<notify-body>` using the raw body.
3. Verify with the Antom public key as above.
4. Do not sign your acknowledgement.

- `http-uri` for a notification is the path of your notification endpoint as received. The docs do not spell this out separately for notifications (checked 2026-10-21); the Java `WebhookTool.checkSignature(requestUri, httpMethod, clientId, requestTime, signature, notifyBody, antomPublicKey)` takes it as `requestUri`.
- The notifications page shows a sample `Request-Time` header in ISO 8601 form. Use the header value exactly as received.

## Message Encoding

- Byte data is Base64 encoded. URLs are URL-encoded.
- All non-array values, including amounts, are sent as strings.

## Amounts

- `currency`: 3-letter ISO 4217 code. `value`: positive integer in the smallest unit, as a string.
- MYR has 2 decimal places: RM1.00 is `"100"`, RM13.14 is `"1314"`.
- Minimum payment and refund amounts vary per payment method. See `antom-codes.md`.
- If `order.goods` is sent, `order.orderAmount.value` must equal the sum of `goodsUnitAmount.value` times `goodsQuantity`.

## Idempotency

| API | Idempotency key |
| --- | --- |
| pay | `paymentRequestId` |
| refund | `refundRequestId` |
| capture | `captureRequestId` |

- Repeating a request with the same key after a final result (`S` or `F`) returns the same result.
- Changing amount, currency, or payment method under the same `paymentRequestId` returns `REPEAT_REQ_INCONSISTENT`.
- capture: `F` cannot be retried with the same ID; `U` can be retried and returns the final result.

## Endpoint Catalogue

Paths are under `/ams/api` (or `/ams/sandbox/api` in sandbox). All are `POST`.

### One-time Payments (in scope)

| Path | Purpose |
| --- | --- |
| `/v1/payments/consult` | List available payment methods, logos, FPX bank list for an amount and terminal |
| `/v1/payments/createPaymentSession` | Create a session for Hosted or Embedded Checkout Page or Payment Element |
| `/v1/payments/pay` | Create a payment (API-only, card S2S) and get redirect URLs or code |
| `/v1/payments/inquiryPayment` | Query payment status and capture or refund transactions |
| `/v1/payments/capture` | Capture an authorized card, Apple Pay, or Google Pay payment (manual capture) |
| `/v1/payments/cancel` | Cancel an unpaid or same-day payment (full only) |
| `/v1/payments/refund` | Full or partial refund |
| `/v1/payments/inquiryRefund` | Query refund status |
| `/v1/payments/updateAmount` | Increase an authorized amount before capture |
| `/v1/payments/inquireExchangeRate` | Exchange rates between payment and supported currencies |
| `/v1/customer/initializeAuthentication` | Get a link to collect device and browser data ahead of 3DS |

### Notifications Antom sends to you

| Notification | Purpose |
| --- | --- |
| notifyPayment (One-time Payments) | Payment result, pending, or amount update result |
| notifyCapture (One-time Payments) | Capture result |
| notifyRefund | Refund result |
| notifyDispute | Dispute created or judged |
| notifyAuthorization | Tokenized Payment authorization created or token cancelled |
| notifyVaulting | Vaulting result |
| notifySubscription | Subscription status change |
| notifyPayment (Subscription) | Subscription period payment result |
| notifyReversal | Reversal result |

### Tokenized Payment, vaulting, subscriptions (brief scope)

| Path | Purpose |
| --- | --- |
| `/v1/authorizations/consult` | Get the wallet authorization URL |
| `/v1/authorizations/applyToken` | Exchange `authCode` for an access token |
| `/v1/authorizations/revoke` | Revoke a user's authorization |
| `/v1/payments/pay` (Tokenized Payment) | Charge with an access token |
| `/v1/payments/createPaymentSession` (Tokenized Payment, EasySafePay) | Session for client SDK in those products |
| `/v1/vaults/createVaultingSession` | Session for card vaulting SDK |
| `/v1/vaults/vaultPaymentMethod` | Vault a payment method before payment |
| `/v1/vaults/inquireVaulting` | Query vaulting status |
| `/v1/vaults/fetchNonce` | Collect card data without PCI qualification |
| `/v1/subscriptions/create` | Create a subscription and get an authorization URL |
| `/v1/subscriptions/change` | Change a subscription (the page description is copied from a notification page; read the page fields) |
| `/v1/subscriptions/update` | Update plan, description, period, or amount |
| `/v1/subscriptions/cancel` | Cancel or terminate a subscription |
| `/v1/subscriptions/inquireSubscription` | Get one subscription |
| `/v1/subscriptions/inquireSubscriptionList` | List subscriptions by filter |
| `/v1/subscriptions/inquireSubscriptionPayment` | List payments of a subscription |

### Disputes and other

| Path | Purpose |
| --- | --- |
| `/v1/payments/acceptDispute` | Accept a dispute |
| `/v1/payments/supplyDefenseDocument` | Upload defense evidence |
| `/v1/payments/downloadDisputeEvidence` | Download evidence template or uploaded files |
| `/v1/payments/uploadInvoiceShippingFile` | Upload invoice or AWB (UPI, Net Banking, Indian cards only) |
| `/v1/customs/declare` | Send customs declaration |
| `/v1/customs/inquiryDeclarationRequests` | Query customs declaration status |

Out of scope with their own endpoint families: Billing (`/v1/billing/*`, `/v1/meter/*`, `/v1/tax/*`), Antom Business Account (`/v1/aba/*`), ISV and marketplace (`/v1/merchants/*`, payouts, transfers, reserve), APO, Antom Shield risk APIs (`/v1/risk/*`). See `source-map.md`.

## Result Handling For pay

| resultStatus | resultCode | Action |
| --- | --- | --- |
| `S` | `SUCCESS` | Payment succeeded. For cards, Apple Pay, Google Pay, still wait for the capture result |
| `U` | `PAYMENT_IN_PROCESS` with a URL | Redirect the buyer to `normalUrl`, `applinkUrl`, or `schemeUrl` |
| `U` | `PAYMENT_IN_PROCESS` without a URL | Call inquiryPayment, or retry with a new `paymentRequestId` |
| `U` | Other | Close the order or retry with a new `paymentRequestId` |
| `F` | Any | Fail; retry only with a new `paymentRequestId`. No notification is sent for synchronous `F` |

No response (timeout): close the order or retry with a new `paymentRequestId`. Poll inquiryPayment every 2 seconds until a final status or a notification arrives.

## Server SDKs

| Language | Repository |
| --- | --- |
| Java | `https://github.com/alipay/global-open-sdk-java` (Maven `com.alipay.global.sdk:global-open-sdk-java`, Java 6+) |
| Python | `https://github.com/alipay/global-open-sdk-python` |
| PHP | `https://github.com/alipay/global-open-sdk-php` |
| .Net | `https://github.com/alipay/global-open-sdk-dotnet` |
| Go | `https://github.com/alipay/global-open-sdk-go` |

Java client: `new DefaultAlipayClient(EndPointConstants.SG, <MERCHANT_PRIVATE_KEY>, <ANTOM_PUBLIC_KEY>, <CLIENT_ID>)`. Python client: `DefaultAlipayClient(gateway_url, client_id, merchant_private_key, alipay_public_key)`.

Web client SDK: npm `@alipay/ams-checkout` (`AMSElement`, `AMSCheckoutPage`, or `loadAntom()` then `antom.createElement()`). Payment Element needs SDK version 1.46.0 or later.

## AI Tooling From Antom

- Antom Integration Skill: `https://docs.antom.com/ac/ref/skill.md` (source `https://github.com/ant-intl/antom-ai-tools`).
- Antom MCP server `https://mcp.antom.com` (OAuth, allowlisted merchants only): `https://docs.antom.com/ac/ref/mcp.md`.
