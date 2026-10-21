---
name: setup-finexus
description: Set up, build, debug, review, and explain Finexus payment integrations using official Finexus docs, the MyXaaS developer portal, PayMaster UPP hosted payment page, PayMaster API and Merchant Plug-In (MPI), HMAC-SHA256 secure hash (t001_SHT / t002_SHV), payment query 0780, reversal and refund 0400, DuitNow Pay online banking and FPX, dynamic and static DuitNow QR, Alipay+, card tokens, recurring and pre-auth, WooCommerce and OpenCart plugins, Kayaaku POS terminals, sandbox/live setup and settlement. Use when working with Finexus, PayMaster, MyXaaS, Kayaaku POS, Finexus Cards, FPX, DuitNow, cards, QR payments, return URL verification, requery, refunds, or production readiness.
---

# Finexus PayMaster Payment Gateway

Use this skill for Finexus online payments (PayMaster) and Finexus Kayaaku POS terminal integrations.

## Source

Read `references/finexus-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, field tables, MTIs, the secure hash method with test vectors, amount rules, query, reversal and refund, DuitNow Pay, QR, recurring, pre-auth, status codes and plugin cautions.

Read `references/account-setup.md` for registration, credentials, sandbox vs live, env var names, method activation, pricing link and settlement.

Read `references/finexus-kayaaku-pos.md` only for in-person terminals (App-to-App, Bluetooth, USB, HTTP JWE, pre-auth HTTPS, UCAT).

Read `references/source-map.md` to pick the official page for a topic and to see doc coverage.

## Core Workflow

1. Identify environment (sandbox or production) and the host Finexus issued; never guess the production host.
2. Identify surface: UPP hosted page (default), DuitNow Pay direct, dynamic QR, consumer presented QR, static QR, card token / recurring, pre-auth, plugin, or Kayaaku POS.
3. Choose MPI library or raw PayMaster API. Both send the same `name=value` messages.
4. Create a local pending payment with a unique `f263_MRN`, expected amount and currency before redirecting.
5. Build the 0280 message server side, hash it, and send the browser to `/upp/faces/upp/payment.xhtml`.
6. On the 0290 return (browser GET), verify `t002_SHV`, then match MRN, amount and currency.
7. Confirm with a 0780 query (`requery0780.xhtml`) before fulfilling; only trust it when `f274_QRespCode=00`.
8. Settle idempotently on result `00` for the mode in `f283_UPP_PM`; keep `22` / `68` / no response as pending and keep polling.
9. Handle dynamic QR via the server-to-server MT0250 callback (verify hash, reply HTTP 200) plus query fallback.
10. Use 0400 for void or reversal same day and refund (`993000`) after settlement.

## Core Facts

- Messages are `name=value` fields: header `h001_MTI`..`h004_TTM`, body `fNNN_*`, trailer `t001_SHT=SH2`, `t002_SHV`.
- Hash: `UPPER(HEX(HMAC_SHA256(key=SecretKey, data=<fields joined by & without trailer> + SecretKey)))`.
- Hash raw values, keep empty fields, exclude `f289_*` and `f290_Signature`; verify responses over the decoded query string before `&t001_SHT=`.
- Docs test vector: key `00112233445566778899AABBCCDDEEFF`, 0880 example gives `6C825DC0...3A962892`.
- MPI flags a bad hash as `TxnStatDetCde=5015`.
- Amount `f007_TxnAmt` is 12 digits with implied decimals (`f019_ExpTxnAmt=2`); RM 1.00 = `000000000100`; currency `458`, country `MY`, service `FNX`.
- UPP payment 0280 to 0290, query 0780 to 0790, reversal / refund 0400 to 0410, DuitNow Pay 0880 to 0890, bank listing 0360 to 0370, dynamic QR 0180 to 0190 plus 0250 callback.
- Paths: `/upp/faces/upp/payment.xhtml`, `/upp/faces/upp/requery0780.xhtml`, `/upp/faces/reversal0400.xhtml`, `/upp/faces/duitnowpaypayment.xhtml`, `/upp/faces/banklisting.xhtml`, `/upp/faces/generateQrPayload.xhtml`.
- Docs mask hosts as `xxx.finexusgroup.com`; examples use `sandbox.finexusgroup.com`; the MPI ships with `dev.finexusgroup.com:10501`.
- Query rule: no response after 30 s, query every 5 s up to 30 min (dynamic QR: up to 5 min).
- Production blocks an IP that floods requests for 1 min, plus 1 min per further attempt.
- Void / reversal before settlement (same day); refund after; DuitNow QR is refund only; no partial reversal.
- Onboarding is through the MyXaaS application form; Finexus issues MID, Secret Key, TID and URLs.

## Critical Rules

- Keep the Secret Key, MID config, RSA and EC private keys server side only.
- Never mark paid from the browser return alone; verify the hash and confirm with a 0780 query.
- Do not copy the official WooCommerce plugin's return handler: it trusts `result_mode=succ` from the URL without a hash check.
- Use constant-time comparison for `t002_SHV` (the plugin uses `==`).
- Hash fields in the documented message order; if a hash fails, check order, empty fields, URL decoding (`+` to space) and the `^` substitution for `&` in return URLs.
- Match `f263_MRN`, `f007_TxnAmt`, `f010_CurrCode` and `f001_MID` against the local record before settling.
- Make settlement idempotent across return, query, QR callback and static QR push (dedupe `trxRefNo`).
- Treat pending (`22`), timeout (`68`), query `96` and missing responses as unknown, not failed.
- Use a new MRN for each payment and for each DuitNow refund.
- Never invent production hosts, test cards, fees or settlement times; ask Finexus.

## Completion Criteria

A Finexus integration is done only when:

1. 0280 (or 0880 / 0180) messages are built and hashed server side with the correct MID and Secret Key per environment.
2. The return handler verifies `t002_SHV` with constant-time compare and rejects mismatches.
3. Settlement runs only after a 0780 query with `f274_QRespCode=00` and result `00`, or a verified QR callback, and checks MRN, amount, currency and MID.
4. Pending and timeout outcomes are polled per the documented schedule and stop cleanly.
5. Reversal and refund use 0400 with the right processing code and are idempotent.
6. Tests cover the docs hash vector, bad hash, wrong amount, wrong MRN, duplicate return or callback, pending then success, and failed codes.
7. Deployment config has `FINEXUS_MERCHANT_ID`, `FINEXUS_SECRET_KEY`, `FINEXUS_PAYMASTER_BASE_URL`, return URLs, and sandbox / live separation.

## Loophole Review Loop

When asked whether a Finexus strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, return URLs, backend message builder, Secret Key storage, PayMaster host, query endpoint, QR callback, static QR push, database, fulfilment job, refund job, plugin code, Kayaaku POS device.
2. Check bypasses: forged return query, replayed success URL, hash over wrong order or encoded values, wrong key or environment, amount or currency swap, reused MRN, pending treated as failed, query `12` treated as failed too early, duplicate callback, refund twice, plugin `result_mode` trust, rate limit block during polling.
3. Add fixes: server-side hashing, constant-time verify, query confirmation, strict field matching, durable event log, idempotency on MRN, separate env config, backoff on queries.
4. Re-read the relevant sections of `references/finexus-api.md`.
5. State remaining uncertainty: production host, hash field order for optional fields, enabled methods, settlement timing, test cards, and Kayaaku POS signing differences must be confirmed with Finexus.
