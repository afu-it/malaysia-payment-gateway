---
name: setup-leanx
description: Set up, build, debug, review, and explain LeanX (Lean.x, Leanis Solutions) payment integrations using official LeanX docs, auth-token header, optional X-Signature HMAC-SHA256 request signing, Collections and Bills, hosted payment page (create-bill-page), own-page silent bills (create-bill-silent) with payment_service_id, FPX B2C/B2B, cards, e-wallets, DuitNow QR, BNPL, JWT-encoded callbacks decoded with the Hash Key, manual-checking-transaction status requery, payouts via DuitNow, bank account validation, pool balance, WooCommerce/OpenCart plugins, the Credit Card JS library, sandbox (api.leanx.dev) vs production (api.leanx.io). Use when working with LeanX checkout, FPX, DuitNow QR, e-wallet or card collection, payout disbursement, callback settlement, or production readiness.
---

# LeanX Payment Gateway

Use this skill for LeanX (Lean.x) payment collection and payout integration work.

## Source

Read `references/leanx-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, base URLs, auth header, X-Signature algorithm, bill creation fields, payment service list, callback JWT format, status API, payout flow, error codes, and documented inconsistencies.

Read `references/account-setup.md` when the user asks how to register, where the Auth Token / UUID / Hash Key live, sandbox vs production portals, env var names, plugin setup, payout pool funding, or pricing.

Read `references/source-map.md` to pick the official docs page for a topic and to see docs coverage.

## Core Workflow

1. Identify environment: sandbox (`https://api.leanx.dev`) or production (`https://api.leanx.io`).
2. Identify surface: hosted payment page (`create-bill-page`), own bank-selection page (`list-payment-services` + `create-bill-silent`), DuitNow QR (silent bill with QR `metadata`), card form (Credit Card JS library), plugin, or payout.
3. Confirm portal setup: API key created and enabled, Auth Token, UUID, Hash Key, a Collection UUID, X-Signature on or off, production IP whitelist if used.
4. Create a local pending payment with your own unique `invoice_ref` before calling LeanX.
5. Create the bill server-side with the `auth-token` header (plus `x-signature`, `x-timestamp`, `x-nonce` when X-Signature is enabled).
6. Redirect the customer to the returned URL (`data.redirect_url` for the payment page, `data.payment_url` for a silent bill) or render the QR from `data.metadata`.
7. Treat `redirect_url` as browser UX only; never fulfil from it.
8. Receive the callback (`{"data": "<JWT>", "response_code": 2100}`), verify and decode the HS256 JWT with the Hash Key, then requery `manual-checking-transaction`.
9. Settle idempotently only after invoice number, `invoice_ref`, amount, `invoice_status == "SUCCESS"`, environment and merchant match.
10. Run a requery job for pending bills: callbacks are sent only for successful payments, so failures and missed callbacks are found only by polling.

## Core Facts

- Production API base `https://api.leanx.io`; sandbox `https://api.leanx.dev`. All paths are under `/api/v1/merchant/...`.
- Auth: header `auth-token: <MERCHANT_ID>|<UUID>|<secret>` (the full Auth Token string from the portal API page). The token is static.
- Optional X-Signature: HMAC-SHA256 (hex) over `HTTP_METHOD|UUID|URL_PATH|TIME_STAMP|AUTH_TOKEN|NONCE` keyed with the Hash Key; send `x-signature`, `x-timestamp` (Unix seconds), `x-nonce` (UUID4). Valid 5 minutes.
- Hosted page: `POST /api/v1/merchant/create-bill-page` (optional query `invoice_no`). Own page: `POST /api/v1/merchant/create-bill-silent` with `payment_service_id`.
- Every bill needs a `collection_uuid`; one collection can be reused indefinitely.
- `amount` is Decimal(10,2) in MYR ringgit (for example `10.00`), not sen.
- Payment service list: `POST /api/v1/merchant/list-payment-services` with `payment_type`, `payment_status`, `payment_model_reference_id` (1 = B2C/default, 2 = FPX B2B).
- Payment types: `WEB_PAYMENT` (FPX), `CI_VISA_MASTERCARD`, `GLOBAL_CARD_PAYMENT`, `DIGITAL_PAYMENT` (e-wallet), `BUY_NOW_PAY_LATER`, `SWITCH_PAYPAL`, `GLOBAL_CARD_PAYMENT_STRIPE`, `DUITNOW_ONLINE_BANKING`, `QR_PAYMENT` (DuitNow QR).
- Callback body is JSON `{"data": "<HS256 JWT>", "response_code": 2100}`; decode with the Hash Key. Sent only for successful transactions; retried up to 3 times unless you return HTTP 200.
- Status requery: `POST /api/v1/merchant/manual-checking-transaction?invoice_no=<bill invoice_no or your invoice_ref>`. LeanX calls this method recommended over the callback.
- Bill statuses seen: `SUCCESS` (`invoice_status_id` 2), `PENDING` (1), `FAILED`; plugins also treat `FPX_STOP_CHECK_FROM_PS` and `SENANGPAY_STOP_CHECK_FROM_PS` as failed.
- Success envelope: `response_code: 2000`, `description: "SUCCESS"`; errors use numeric codes such as 4008 Invalid HMAC signature, 5688 Invalid Token, 4626 Bill Invoices Already Exist.
- Refunds, recurring billing and tokenisation are not documented in the public API docs (checked 2026-10-21).
- Payouts: `POST /api/v1/merchant/create-payout-invoice` from a prefund pool (`virtual_pool_reference`), RM2.00 minimum pool balance, status via `get-payout-transaction-by-id`.
- Production supports an IP whitelist feature; sandbox does not. IP addresses are not published.

## Critical Rules

- Keep Auth Token and Hash Key server-side only; the Hash Key both signs requests and verifies callbacks.
- Do not mark paid from `redirect_url`, browser state, or the card library result.
- Verify the callback JWT locally with a JWT library pinned to `HS256` and the Hash Key; reject any other `alg`. Do not post the Hash Key to a remote decode endpoint.
- After decoding, confirm with `manual-checking-transaction` before settling; compare amount as a decimal string.
- Generate a unique `invoice_ref` per attempt; duplicates return 4626 or 3614.
- Respond HTTP 200 to callbacks only after durable processing; make processing idempotent across the 3 retries.
- Use matching base URL, Auth Token, Hash Key, Collection UUID and payment service IDs per environment; service IDs differ between sandbox and production (card is `1100` in production).
- Refresh the payment service list daily rather than hardcoding bank IDs.
- For payouts, validate the account first, key status checks by `external_invoice_ref`, and treat `PENDING` / `STATUS_PENDING_APPROVAL` as non-final.

## Completion Criteria

A LeanX integration is done only when:

1. Bill creation runs server-side with `auth-token` (and X-Signature headers if enabled) and a unique `invoice_ref`.
2. Redirect handling only triggers a status requery; it never fulfils directly.
3. Callback handler verifies the HS256 JWT with the Hash Key, rejects bad signatures, and returns 200 only after persistence.
4. Settlement checks invoice number, `invoice_ref`, amount, `invoice_status`, environment and merchant before marking paid.
5. A scheduled requery resolves pending bills to SUCCESS or FAILED, since failed payments send no callback.
6. Tests cover invalid JWT, wrong `alg`, duplicate callback, wrong amount, unknown invoice, FAILED and PENDING status, and success.
7. Deployment docs list `LEANX_API_URL`, `LEANX_AUTH_TOKEN`, `LEANX_UUID`, `LEANX_HASH_KEY`, `LEANX_COLLECTION_UUID`, callback URL, redirect URL, and sandbox/live separation.

## Loophole Review Loop

When asked whether a LeanX strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, payment page, own bank-selection page, QR page, card library, backend bill creator, LeanX API, callback receiver, JWT verifier, database, requery job, payout job, portal settings.
2. Check bypasses: fake redirect, forged callback, JWT with `alg: none` or wrong key, Hash Key leaked to a remote decoder, sandbox/live key mix, wrong collection, reused `invoice_ref`, amount mismatch, stale or missing callback, failed payment never reported, duplicate payout, payout to wrong account name, insufficient pool balance, X-Signature clock skew.
3. Add fixes: local pending record first, server-side bill creation, local HS256 verification, status requery before settlement, decimal amount compare, unique refs, idempotent updates, scheduled requery, bank validation before payout, NTP-synced clock.
4. Re-read the relevant sections of `references/leanx-api.md`, including "Documented inconsistencies".
5. State remaining uncertainty: callback JWT key encoding, exact `URL_PATH` for signed requests with query strings, refund process, settlement timing, enabled payment services, and IP whitelist values must be confirmed with LeanX or in sandbox.
