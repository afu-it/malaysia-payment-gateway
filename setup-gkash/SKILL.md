---
name: setup-gkash
description: Set up, build, debug, review, and explain Gkash payment integrations using official Gkash docs, host-to-host checkout (POST /api/payment/form), legacy hosted payment form, SHA512 signatures, server-side callback with "OK" acknowledgement, return URL handling, transaction status query, refunds, pre-auth and capture, recurring card tokens (epkey), sandbox/staging setup, CID and signature key handling, and production readiness. Use when working with Gkash Malaysia, pay.asia, api.gkash.my, FPX internet banking, cards, e-wallets, DuitNow QR, BNPL, installments, WooCommerce or OpenCart Gkash plugins, callback signature mismatches, or Gkash settlement and reconciliation.
---

# Gkash Payment Gateway

Use this skill for Gkash online payment gateway integration work.

## Source

Read `references/gkash-api.md` before giving factual API guidance or writing integration code. It contains environment hosts, checkout request and response fields, the exact SHA512 string-to-sign for every request and callback, callback and return handling, status query, refund, pre-auth, recurring, and documented gaps.

Read `references/gkash-offline.md` for e-wallet QR scan or display (`paymentid` codes), static DuitNow QR, SoftPOS `RemotePay`, Soundbox, and e-Invoice QR. Those `/apim/merchant/*` calls use a different signature case rule.

Read `references/account-setup.md` when the user asks how to register, which credentials are needed, where the CID and signature key come from, sandbox setup, merchant portal, payment method activation, pricing, or settlement timing.

Read `references/source-map.md` when choosing which Gkash docs page applies, or when a fact must be re-checked against the official source.

## Core Workflow

1. Identify environment: staging (`https://api-staging.pay.asia`) or production (`https://api.gkash.my`).
2. Identify surface: host-to-host checkout (current, recommended), legacy browser form post, pre-auth and capture, recurring, plugin (WooCommerce, OpenCart), or offline (e-wallet QR, terminal, Soundbox).
3. Confirm credentials: `CID` (merchant/user ID, for example `M161-U-xxx`) and signature key, for the same environment.
4. Create a local pending payment with a unique `v_cartid` before calling Gkash.
5. Build the request signature server-side: `SHA512(UPPER(key;CID;v_cartid;amountNoDot;v_currency))`, send as lowercase hex.
6. `POST /api/payment/form` (JSON) from the server; store `poId`; then 302 to `redirect.url` or render `redirect.html`.
7. Handle the server-to-server callback (form POST to `callbackurl`): verify signature, settle idempotently, respond with body `OK`.
8. Treat `returnurl` (browser form POST) as UX only, even after verifying its signature.
9. Use `POST /api/payment/query` to resolve pending or missing callbacks before deciding final state.
10. Issue refunds server-side with `POST /api/payment/refund` and record the result.

## Core Facts

- Staging host: `https://api-staging.pay.asia`. Production host: `https://api.gkash.my` (from Gkash's official WooCommerce plugin and iOS SDK; doc pages show staging only).
- Checkout (host-to-host): `POST /api/payment/form`, `Content-Type: application/json`, `version` `1.5.5`.
- Legacy checkout: browser form `POST /api/paymentform.aspx` (`application/x-www-form-urlencoded`), same fields and signature.
- Auth is per-request SHA512 signature only; no API key header, no OAuth.
- Request signature: `UPPER(signatureKey;CID;v_cartid;v_amount;v_currency)` hashed with SHA512. Amount is 2 decimals with the dot removed (`100.00` -> `10000`, `0.1` -> `010`).
- Callback and return signature: `UPPER(signatureKey;CID;POID;cartid;amount;currency;status)`, SHA512, compared against the posted lowercase `signature`.
- The whole string is uppercased, including the signature key and the status text (`88 - TRANSFERRED`).
- Status values: `88 - Transferred` (paid), `66 - Failed`, `11 - Pending`. Parse the leading number.
- Callback must return HTTP 200 with body `OK`.
- Currency: `MYR` only. `v_cartid` must be unique; only `-`, `_`, `.` special characters allowed.
- Query: `POST /api/payment/query` (JSON, `version` `1.3.0`). Refund: `POST /api/payment/refund` (JSON, `version` `1.3.1`). Both sign `key;CID;cartid;amount;currency`.
- Refund statuses: `00 - Refund Successful`, `01 - Refund Failed`, `02 - Partial Refund Successful`.
- Pre-auth: add `v_preauth=PREAUTH` (included in signature); capture via `POST /api/payment/submit` with `v_preauth=<POID>`.
- Recurring: add `recurringtype` (`MONTHLY`/`ANNUAL`); callback returns `epkey`; charge again via `POST /api/payment/submit` with `epkey`; each charge returns a new `epkey`.
- Endpoint catalogue (14 operations) and enums: `references/gkash-api.md`. No documented void, payout, settlement report, error codes, or FPX bank list.
- Sandbox sign-up: `https://doc.gkash.my/getting-started`; sandbox merchant portal: `https://demo.pay.asia`.

## Critical Rules

- Keep the signature key server-side only. Never compute signatures in the browser or mobile client.
- Never mark paid from `returnurl`, the checkout response (`11 - Pending`), or a frontend redirect.
- Verify the callback signature before any state change; use constant-time comparison on lowercase hex.
- Build the signature from the exact posted values (`cartid`, `amount`, `currency`, `status`), but settle only after they also match your stored order and your own `CID`.
- Format the signature amount from a decimal, never from float string concatenation; `1` must become `100`.
- Generate a fresh unique `v_cartid` per attempt; retries with the same cart ID will collide.
- Make callback processing idempotent on `POID` and `v_cartid`; reply `OK` for already-settled duplicates.
- Treat `11 - Pending` as not final; requery later instead of failing the order.
- Do not mix staging and production CID, key, or host.
- Do not trust `description` for logic; Gkash says it varies by acquirer.
- Store the latest `epkey` after every recurring charge; the old one is replaced.

## Completion Criteria

A Gkash integration is done only when:

1. Checkout is created server-side with a correct SHA512 signature and a unique `v_cartid`.
2. The browser is sent only to `redirect.url` or `redirect.html` returned by Gkash.
3. The callback handler verifies the signature, checks CID, cart ID, amount, currency, and status against the stored order, and returns `OK`.
4. Return URL shows a result page but does not fulfil on its own.
5. Pending and missing callbacks are resolved with the query API.
6. Duplicate callbacks, replays, and late callbacks on closed orders are idempotent.
7. Tests cover the documented signature vectors, invalid signature, wrong amount/cart ID/CID, `66 - Failed`, `11 - Pending`, and `88 - Transferred`.
8. Deployment docs list `GKASH_CID`, `GKASH_SIGNATURE_KEY`, `GKASH_BASE_URL`, callback URL, return URL, and staging/production separation.

## Loophole Review Loop

When asked whether a Gkash strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser, checkout creator, Gkash API, redirect page, callback receiver, return URL, database, fulfilment job, query job, refund job, recurring job, merchant portal.
2. Check bypasses: forged return POST, forged callback, signature built from stored instead of posted values or vice versa, missing uppercase step, float amount formatting, wrong CID, staging key on production, reused cart ID, pending treated as failed, duplicate callback, callback before order commit, refund replay, stale `epkey`.
3. Add fixes: server-only signing, decimal amount formatter with test vectors, strict CID/cart/amount/currency match, constant-time compare, durable event log, unique constraints on `POID` and `v_cartid`, query fallback, idempotent refunds.
4. Re-read `references/gkash-api.md` sections relevant to any gap.
5. State remaining uncertainty: production endpoint paths for v2 checkout, callback retry policy, sync response signature formula, refund `amount` semantics, IP allowlist, rate limits, and test cards are not documented publicly.
