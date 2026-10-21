---
name: setup-shopeepay-gateway
description: Set up, build, debug, review, and explain direct ShopeePay merchant integrations using official ShopeePay docs (product.shopeepay.com), Checkout with ShopeePay (jump app / redirect), Merchant Presented Mode dynamic QR, Customer Presented Mode scan, hosted ShopeePay Payment Gateway checkout sessions, Account Linking, Link & Pay, Subscription, Auth & Capture, refunds, invalidate order/QR, Notify Transaction Status callbacks, Check Transaction Status requery, X-Airpay-ClientId and X-Airpay-Req-H HMAC-SHA256 Base64 signatures, sandbox (UAT) and production airpay.com.my hosts, and the official WooCommerce and Shopify plugins. Use when working with ShopeePay Malaysia wallet acceptance, SPayLater through ShopeePay checkout, MYR wallet QR, DuitNow QR or online banking through the ShopeePay hosted gateway, callbacks, settlement, refunds, or production readiness.
---

# ShopeePay Direct Merchant Integration

Use this skill for direct ShopeePay merchant API work (ShopeePay Malaysia Sdn Bhd), not for ShopeePay accepted through another aggregator gateway.

## Source

Read `references/shopeepay-gateway-api.md` first for any factual API guidance or code. It holds hosts, headers, the signature algorithm with a verified test vector, the endpoint catalogue, Checkout with ShopeePay, MPM, CPM, invalidate, refund, Notify Transaction Status, Check Transaction Status, common enums, error codes, and the retry policy.

Read `references/shopeepay-gateway-hosted-gateway.md` for the separate ShopeePay Payment Gateway product (`/v1/checkout`, `/v1/refund`, webhook events) on the `api.gw.*` hosts.

Read `references/shopeepay-gateway-tokenized.md` for Account Linking, access tokens, Link & Pay, Subscription, Auth & Capture, and the risk `additional_info` object.

Read `references/account-setup.md` for onboarding, credentials, sandbox, plugins, callback URL registration, and Malaysia availability.

Read `references/source-map.md` to pick the right official page and to see doc coverage and known doc gaps.

## Core Workflow

1. Identify the product: Checkout with ShopeePay (online redirect), ShopeePay Payment Gateway (hosted multi-method checkout), MPM dynamic QR, CPM scan, Link & Pay, Subscription, or Auth & Capture.
2. Identify environment and host: wallet APIs `api.uat.wallet.airpay.com.my` (staging) / `api.wallet.airpay.com.my` (production); hosted gateway `api.gw.uat.airpay.com.my` / `api.gw.airpay.com.my`.
3. Create a local pending payment with a unique `payment_reference_id` and the amount in sen (RM x 100) before calling ShopeePay.
4. Serialize the JSON body once, sign those exact bytes with HMAC-SHA256 using the secret key, Base64 encode, and send `X-Airpay-ClientId` and `X-Airpay-Req-H` headers. All wallet API calls are server-side `POST`.
5. Check `errcode` (0 = success) in every response; treat `-1`, `-2`, `5`, timeouts, and "processing" as unknown, never as failed.
6. Redirect the customer to `redirect_url_http` (Checkout) or show `qr_content` (MPM). Treat `return_url` as UX only.
7. Verify the Notify Transaction Status callback signature over the raw body, match `payment_reference_id` and `amount`, then reply `{"errcode":0}`.
8. In Malaysia the callback is sent only once, so always run Check Transaction Status as a fallback on the documented backoff schedule.
9. Settle idempotently; invalidate abandoned orders or QRs; refund with a unique `refund_reference_id`.

## Core Facts

- Signature: Base64(HMAC-SHA256(secret_key, raw_request_body)) in header `X-Airpay-Req-H`; client ID in `X-Airpay-ClientId`.
- Protocol: HTTPS, POST, JSON, UTF-8, UNIX seconds, TLS 1.2 or 1.3, no SDK (direct REST only).
- Amount is an integer inflated by 100 (RM 10.00 = `1000`).
- Checkout with ShopeePay: `POST /v3/merchant-host/order/create`, returns `redirect_url_http`; Malaysia max validity 3600 s, default 1200 s.
- MPM dynamic QR: `POST /v3/merchant-host/qr/create`; Malaysia max 3600 s, default 1200 s; `qr_url` valid 5 minutes.
- CPM: `POST /v3/merchant-host/transaction/payment/create` with the scanned `payment_code`.
- Status: `POST /v3/merchant-host/transaction/check` with `reference_id`, `transaction_type` (13 payment, 15 refund) and original `amount`.
- Refund: `POST /v3/merchant-host/transaction/refund/create-new`; full or partial; 365-day window; idempotent on identical payload.
- Callback success (MY): `payment_status = 1` (CPM/MPM/Checkout) or `transaction_status = 3` (Link & Pay/Subscription).
- Transaction status enum: 1 initial, 2 processing, 3 successful, 4 failed, 6 expired, 7 invalidated.
- Callback URL is registered with the ShopeePay integration team during onboarding, not per request.
- Credentials (Client ID, Secret Key, `merchant_ext_id`, `store_ext_id`) are issued after NDA and commercial agreement; no self-serve sandbox.

## Critical Rules

- Keep the secret key server-side; never sign in a browser or mobile app.
- Sign the exact byte string you send; do not re-serialize between signing and sending.
- Verify callback `X-Airpay-Req-H` over the raw body with constant-time comparison before parsing.
- Do not mark paid from `return_url`, frontend state, or a customer screenshot alone.
- Validate `amount` and `payment_reference_id` against the original request before marking paid (docs require this).
- `payment_method` in callbacks and status is the product flow (Point of Initiation), not a reliable validation value; docs say do not default-validate on it.
- Do not whitelist domains or IPs for `redirect_url_http`; ShopeePay says not to.
- Use `request_id` unique per call and `payment_reference_id` unique per order; errcode 11 means duplicate request.
- Partial refunds: wait for the previous one to finish; refunds are blocked by default 12am to 5am local time (errcode 601).
- Ignore unknown response fields and field order; ShopeePay adds fields without notice.

## Completion Criteria

A ShopeePay integration is done only when:

1. All calls run server-side with correct `X-Airpay-ClientId` and `X-Airpay-Req-H`, and the signature unit test passes the documented test vector.
2. The local order stores `payment_reference_id`, `request_id`, amount in sen, currency `MYR`, environment, and state before redirect or QR display.
3. The callback handler verifies the signature on the raw body, checks reference and amount, is idempotent, and returns `{"errcode":0}`.
4. A requery job calls Check Transaction Status on the 5 s step backoff to 100 s, then every 5 minutes up to 24 hours.
5. Abandoned orders call Invalidate Order (Checkout) or Invalidate QR (MPM).
6. Refunds use unique `refund_reference_id`, handle errcode 5 and 320, and are re-checked with `transaction_type` 15.
7. Tests cover bad signature, wrong amount, duplicate callback, errcode -1/5/timeout, expired order, and refund failure.
8. Config documents `SHOPEEPAY_CLIENT_ID`, `SHOPEEPAY_SECRET_KEY`, `SHOPEEPAY_MERCHANT_EXT_ID`, `SHOPEEPAY_STORE_EXT_ID`, `SHOPEEPAY_ENV`, and the callback URL registered with ShopeePay.

## Loophole Review Loop

When asked whether a ShopeePay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser, ShopeePay app or web page, `return_url`, backend signer, ShopeePay API, callback receiver, raw-body parser, database, requery job, refund job, plugin settings.
2. Check bypasses: forged callback, signature over re-encoded JSON, wrong secret or host (UAT vs production, wallet vs `gw`), amount in ringgit instead of sen, reused `payment_reference_id`, errcode -1 treated as failed, missed single-shot MY callback, expired order paid late, duplicate refund, refund in block window, stale access token, unlinked account still charged.
3. Add fixes: server-only signing, raw-body HMAC, constant-time compare, strict reference/amount match, unique ids, durable callback log, requery backoff, invalidate on cancel, idempotent settlement and refund records.
4. Re-read the relevant reference sections for any gap.
5. State remaining uncertainty: response signature header name, callback IP ranges, sandbox test data, rate limits, hosted gateway method availability, and pricing are not published; confirm with the ShopeePay integration team.
