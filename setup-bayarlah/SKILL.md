---
name: setup-bayarlah
description: Set up, build, debug, review, and explain bayarlah (Beez Fintech) payment integrations using official bayarlah docs, the Upay Web API, terminal activation and daily check-in, MD5 request signing with vendor and terminal keys, DuitNow QR and WeChat Pay pre-order (precreate), WeChat payment code (CPM) pay, WeChat WAP redirect gateway, query polling, notify_url callbacks with SHA256WithRSA verification, Event Order Notification push with MD5 app_secret verification, refunds, cancel and revoke, Terminal Order Push, and Mini Program plugin signing. Use when working with bayarlah Malaysia, DuitNow QR, cross-border WeChat Pay, e-wallet QR acceptance, POS or website QR checkout, payment callbacks, order polling, refunds, or production readiness.
---

# bayarlah Payment Gateway

Use this skill for bayarlah (Beez Fintech Sdn Bhd) Upay Web API integration work.

## Source

Read `references/bayarlah-api.md` before giving factual API guidance or writing integration code. It contains the endpoint catalogue, domains, activation and check-in, the request signature, every request and response field table, callback and event-push verification, polling schedules, refunds, cancellation, status values, and error codes.

Read `references/account-setup.md` when the user asks how to get API parameters, what to send to bayarlah, where activation codes come from, env var names, WeChat refund authorization, settlement timing, or pricing.

Read `references/source-map.md` when choosing which bayarlah docs page applies, or to check coverage and what the docs do not publish.

## Core Workflow

1. Get `vendor_sn`, `vendor_key` and `app_id` from bayarlah technical support (there is no self-serve sandbox in the docs).
2. Generate a store activation code in the merchant platform `https://s.bayarlah.net`, then call `/terminal/activate` once per terminal, signed with `vendor_sn` + `vendor_key`.
3. Persist the returned `terminal_sn` and `terminal_key` server-side; call `/terminal/checkin` on the first payment call of each calendar day and save the new key.
4. Create a local pending order with a unique `client_sn` before calling bayarlah.
5. For website or kiosk QR: call `/upay/v2/precreate` with `payway` (`"1000"` DuitNow, `"3"` WeChat), amount in cents, and `notify_url`; render `qr_code` or `qr_code_image_url`.
6. For WeChat in-browser: 302-redirect to `https://qr.bayarlah.net/gateway` with an uppercase MD5 query `sign`; treat `return_url` as display only.
7. Poll `/upay/v2/query` after a successful pre-order and also accept `notify_url` callbacks; stop when either gives a final `order_status`.
8. Verify callbacks: `notify_url` uses SHA256WithRSA with the bayarlah public key; Event Order Notification uses `MD5(body + app_secret)`.
9. Settle idempotently only when `client_sn`, `sn`, `total_amount`, terminal and `order_status = PAID` match.
10. Refund with `/upay/v2/refund` and a stable `refund_request_no`; use `/upay/v2/cancel` for unknown-state WeChat payments.

## Core Facts

- API domain: `https://api.bayarlah.net` (only one domain is documented; no sandbox domain is published).
- All API requests are JSON, UTF-8, `Content-Type: application/json`; all response values are strings.
- Request signature: `sign = MD5(raw_body_bytes + key)`, sent as `Authorization: <sn> <sign>`.
- Activation signs with `vendor_sn`/`vendor_key`; all other calls sign with `terminal_sn`/`terminal_key`.
- Amounts are in cents (sen), numeric strings up to 10 digits (`"100"` = RM1.00).
- Responses wrap results: top-level `result_code` (`"200"` or `"400"`), then `biz_response.result_code` and `biz_response.data`.
- Pre-order success: `biz_response.result_code = "PRECREATE_SUCCESS"` or `data.order_status = "CREATED"`.
- Final order statuses: `PAID`, `PAY_CANCELED`, `REFUNDED`, `PARTIAL_REFUNDED`, `CANCELED`.
- Pre-order payment window is about 4 minutes; documented polling schedule ends with a final poll at 6 minutes.
- `notify_url` callback retries at 1s, 5s, 30s, 600s; reply with the string `success`.
- Activation codes are valid 7 days and have limited uses; never hard-code them.
- Refunds allowed within three months of payment; cancel/revoke only on the same day, once per order.
- `client_sn` cannot be reused after a failed payment; create a new one.

## Critical Rules

- Keep `vendor_key`, `terminal_key`, `app_secret` and the RSA public key config server-side only.
- Sign the exact bytes you send; do not re-serialize JSON after signing.
- Store the newest `terminal_key` atomically after check-in; the old key stops working.
- Do not mark paid from `return_url`, a displayed QR, or a frontend result; use query or a verified callback.
- Verify callback signatures over the raw body before parsing JSON, using constant-time comparison.
- Use the correct verification method per channel: RSA for `notify_url`, MD5 + `app_id` check for Event Order Notification.
- Make settlement idempotent across polling, `notify_url` retries and event-push retries.
- Keep `refund_request_no` unchanged when retrying the same refund; change it for a new refund.
- When a `pay` (CPM) call does not return a clear result, query, then call `cancel` if still unknown.
- Use HTTPS for any URL bayarlah pushes to.
- Do not assume unpublished values (full payway list, sub_payway list, hex case of the API MD5, RSA signature encoding); confirm with bayarlah support.

## Completion Criteria

A bayarlah integration is done only when:

1. Activation runs once, and `terminal_sn`/`terminal_key` persist in durable server storage.
2. Daily check-in runs before the first payment call of the day and saves the new key.
3. Every API call is signed as `MD5(body + key)` over the exact sent bytes with the right `sn`.
4. Pre-order stores `sn`, `client_sn`, amount and expected `payway` before showing the QR.
5. Polling follows the documented schedule and stops on a final status or a verified callback.
6. Callback handlers verify signatures over the raw body and reply `success`.
7. Settlement checks `client_sn`, `sn`, `total_amount`, `terminal_sn` and `order_status`, and is idempotent.
8. Tests cover bad signature, duplicate callback, wrong amount, expired pre-order, expired terminal key, refund retry, and a successful `PAID` order.
9. Deployment docs list the env vars in `references/account-setup.md`.

## Loophole Review Loop

When asked whether a bayarlah strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser, QR display, `return_url`, backend signer, terminal key store, bayarlah API, `notify_url` receiver, event-push receiver, raw-body parser, database, polling job, refund job, merchant platform.
2. Check bypasses: fake `return_url` success, forged callback, signature over parsed JSON, wrong key (vendor vs terminal), stale terminal key after check-in, lost check-in response, reused `client_sn`, amount in ringgit instead of cents, wrong `payway`, polling stopped too early, duplicate callbacks, partial refund double-sent, cancel after the same day, DuitNow refund assumptions.
3. Add fixes: server-side signing over sent bytes, atomic key rotation with retry using the old key, unique `client_sn`, cents conversion, raw-body verification, `app_id` check, durable event log, idempotency keys, stable `refund_request_no`.
4. Re-read `references/bayarlah-api.md` sections relevant to any gap.
5. State remaining uncertainty where the docs are silent: sandbox, payway appendix, MD5 hex case, RSA encoding, DuitNow refunds, rate limits, IP allowlists, settlement and pricing.
