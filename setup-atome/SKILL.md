---
name: setup-atome
description: Set up, build, debug, review, and explain Atome Buy Now Pay Later (BNPL) payment integrations using official Atome Open API v2 docs, HTTP Basic auth, create payment, redirectUrl and appPaymentUrl checkout, server-to-server callbacks, X-Signature HMAC-SHA256 callback verification, Get Payment Information requery, cancel, full and partial refunds, payment states, MYR sen amounts and minimums, test vs production endpoints, IP allowlisting, plugins, and mobile SDKs. Use when working with Atome Malaysia, apaylater, BNPL instalment checkout, Atome callbacks, Atome refunds, or production readiness.
---

# Atome BNPL Payment Gateway

Use this skill for Atome (apaylater) online BNPL integration work.

## Source

Read `references/atome-api.md` before giving factual API guidance or writing integration code. It contains the endpoint catalogue, base URLs, Basic auth, the 10-step checkout flow, full create payment and nested field tables, payment object and states, requery, cancel, refund, callback fields, `X-Signature` HMAC-SHA256, IP allowlist, amount rules, enums, the full error code table, and test notes.

Read `references/atome-plugins-sdk.md` when the merchant uses Shopify, WooCommerce, Magento, PrestaShop, OpenCart, EasyStore, Wix, or Salesforce, or opens `appPaymentUrl` from an iOS, Android, React Native, or Flutter app.

Read `references/account-setup.md` when the user asks how to register, where credentials come from, test vs production setup, env var names, plugins, Malaysia specifics, pricing, or settlement.

Read `references/source-map.md` when choosing which official Atome page applies, or to see the list of gaps that only the Atome Account Manager can answer.

## Core Workflow

1. Identify environment: test (`api.apaylater.net`) or production (`api.apaylater.com`).
2. Identify surface: native API (web redirect or app `appPaymentUrl`) or a platform plugin (Shopify, WooCommerce, Magento, PrestaShop, OpenCart, EasyStore, Wix, Salesforce).
3. Confirm credentials from the Atome Account Manager: API username, API password, and callback signing secret plus its encoding.
4. Run `POST /auth` once with `countryCode: "MY"` and the callback URL to prove credentials and callback reachability.
5. Create a local pending payment, then `POST /payments` server-side with a unique `referenceId`, `MYR` amount in sen, `callbackUrl`, `paymentResultUrl`, `paymentCancelUrl`, `customerInfo.mobileNumber`, `shippingAddress`, and `items`.
6. Send the customer to `redirectUrl` (browser) or `appPaymentUrl` (in-app via SDK).
7. Treat `paymentResultUrl` and `paymentCancelUrl` as browser UX only.
8. On callback, verify `X-Signature` over the raw body, read `referenceId`, then call `GET /payments/{referenceId}`.
9. Settle idempotently only when the fetched status is `PAID` and `referenceId`, amount, currency, and environment match.
10. Call cancel when your own checkout hold expires; refund with a unique `refundId`.

## Core Facts

- Test base: `https://api.apaylater.net/v2`. Production base: `https://api.apaylater.com/v2`. TLS 1.2+.
- Auth: `Authorization: Basic base64(username:password)`, issued by the Atome Account Manager.
- Endpoints: `POST /auth`, `POST /payments`, `GET /payments/{referenceId}`, `POST /payments/{referenceId}/cancel`, `POST /payments/{referenceId}/refund`.
- `POST /payments` is idempotent by `referenceId` (`[0-9a-zA-Z-_]{1,40}`).
- Amounts are integers in the smallest unit; MYR is sen. MYR minimum is RM10.00 (`1000`). Max is not documented.
- Required create fields: `currency`, `amount`, `callbackUrl`, `paymentResultUrl`, `customerInfo` (with `mobileNumber` in E.164), `shippingAddress` (`countryCode`, `lines`, `postCode`), `items` (`itemId`, `name`, `quantity`, `price`).
- Unpaid payments auto-cancel after 12 hours (default and max). `expirationTime` can shorten it, minimum 300 seconds.
- Statuses: `PROCESSING`, `PAID`, `FAILED`, `CANCELLED`, `REFUNDED`.
- `PAID` means the first instalment was received; it stays `PAID` if the shopper misses later instalments.
- Partial refund keeps `PAID`; full refund becomes `REFUNDED`. Refunds are synchronous and idempotent by `refundId`.
- Callback body is only `{"referenceId": "..."}`; Atome retries 5 times in 5 hours, then emails the merchant.
- Callbacks are signed with HMAC-SHA256 over the raw body in the `X-Signature` header with a shared secret; encoding and secret come from the Account Manager.
- Malaysia callback source IPs: `47.254.230.20`, `47.254.238.153`; test `47.254.247.54`.
- In Malaysia, shoppers pay in the Atome app (developer portal FAQ lists MY as app only).

## Critical Rules

- Keep the API password and callback secret server-side only.
- Never mark paid from `paymentResultUrl`, the app return, or the callback body. Always settle from `GET /payments/{referenceId}`.
- Verify `X-Signature` against the raw request bytes before parsing JSON; use constant-time comparison. Do not guess hex vs Base64; confirm with Atome.
- Store `referenceId`, expected amount (sen), currency, environment, and local order id before redirecting.
- Reuse the same `referenceId` when retrying a create after a timeout or 5xx; generate a new one only for a genuinely new attempt.
- Return HTTP 200 to callbacks after durable recording; make processing idempotent across the 5 retries.
- If your checkout reserves stock or a booking for less than 12 hours, call cancel when the hold expires. If cancel returns `PAYMENT_NOT_CANCELLABLE`, requery and fulfil or refund.
- Never send decimal amounts; `12.34` is invalid, send `1234`.
- Branch on error `code`, never on `message`.
- Do not mix test credentials with the production base URL, or the reverse.
- Do not reverse an order because a shopper missed a later instalment.

## Completion Criteria

An Atome integration is done only when:

1. `POST /auth` succeeds for the target environment with `countryCode: "MY"`.
2. `POST /payments` runs server-side with Basic auth, a stored unique `referenceId`, and amount in sen at or above RM10.00.
3. The browser redirect only shows a "confirming" page and triggers a server requery; it never fulfils.
4. The callback handler verifies `X-Signature` (with the confirmed encoding), requeries, and settles idempotently.
5. Settlement checks status `PAID`, `referenceId`, `amount`, `currency`, environment, and local order ownership.
6. Cancel is wired to local checkout expiry; refunds send a unique `refundId` and handle `AMOUNT_NOT_VALID` and `UNABLE_TO_REFUND`.
7. Tests cover invalid signature, duplicate callback, callback for unknown `referenceId`, `FAILED`, `CANCELLED`, partial refund (stays `PAID`), full refund (`REFUNDED`), and successful `PAID`.
8. Deployment docs list `ATOME_BASE_URL`, `ATOME_API_USERNAME`, `ATOME_API_PASSWORD`, `ATOME_CALLBACK_SECRET`, callback URL, result and cancel URLs, and test/production separation.

## Loophole Review Loop

When asked whether an Atome strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser, merchant app webview, backend payment creator, Atome API, Atome app, callback receiver, raw-body parser, database, fulfilment job, refund job, cancel job, and plugin settings.
2. Check bypasses: fake `paymentResultUrl` hit, forged callback, signature over parsed JSON, wrong secret or encoding, test/production mix, decimal amount, amount below RM10.00, reused `referenceId` across orders, stale requery, duplicate callback, payment completed after local hold expired without cancel, duplicate refund without `refundId`, refund above `refundableAmount`, inactive Atome account blocking refund, and firewall blocking Atome IPs.
3. Add fixes: server-side create, local pending record first, raw-body HMAC check, mandatory requery, strict amount/currency/reference match, idempotent settlement keyed on `referenceId`, cancel on expiry, `refundId` idempotency, separate env config, IP allowlist, and `/auth` connectivity test.
4. Re-read `references/atome-api.md` sections relevant to any gap.
5. State remaining uncertainty if it depends on the callback secret encoding, maximum amount, MDR, settlement schedule, rate limits, test shopper accounts, or merchant contract terms.
