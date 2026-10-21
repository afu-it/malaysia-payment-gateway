---
name: setup-paynet
description: Set up, build, debug, review, and explain PayNet payment integrations using official PayNet docs, covering FPX direct merchant and TPA integration (AR/AC/AE/BE messages, RSA-SHA1 fpx_checkSum, exchange certificates), DuitNow Pay acquirer APIs (payment intent, self-hosted checkout, DuitNow Consent, AutoDebit, refunds, webhooks, JWT RS256 auth), RPP DuitNow v3 participant APIs (JWS with SHA-256 payload digest, credit transfer and refund intent webhooks, DuitNow Request, Reversal, AutoDebit, Consent), DuitNow QR merchant-presented string generation (EMV TLV, CRC-16), PayNet Developer Portal onboarding, Certification Centre simulator, System Verification, go-live, connectivity and IP allowlists, financial reports API, and response codes. Use when working with PayNet, FPX, DuitNow, DuitNow Online Banking/Wallets, DuitNow Pay, DuitNow AutoDebit, DuitNow QR, JomPAY QR, RPP, mepsfpx.com.my, or when a merchant asks whether they can integrate with PayNet directly.
---

# PayNet (FPX, DuitNow, DuitNow QR)

Use this skill for work against PayNet's own rails. PayNet is national payment infrastructure, not a merchant gateway. Most merchants reach FPX and DuitNow through a bank acquirer or a licensed payment gateway, and should use that provider's skill instead.

## Source

Read `references/paynet-api.md` first. It is the entry point: who may integrate which product, environments and hosts, auth and signature per product, the full endpoint catalogue, and links to the detail files.

Read the detail file for the product in scope:

- `references/paynet-fpx.md` for FPX merchant or TPA integration (B2C, B2B1, B2B2, bulk refund, AE status enquiry, bank list, checksum, certificates, UAT banks).
- `references/paynet-duitnow-pay.md` for DuitNow Pay acquirer APIs (hosted and self-hosted checkout, saved payment method, AutoDebit, refunds, webhooks, JWT auth).
- `references/paynet-rpp-v3.md` for RPP DuitNow v3 participant APIs (JWS, credit transfer webhook, refund intent, DuitNow Request, Reversal, AutoDebit, Consent, QR webhooks, network administration).
- `references/paynet-duitnow-qr.md` for building or parsing a DuitNow QR string and the acquirer QR flow.
- `references/paynet-codes.md` for every status, reason, reject and response code.
- `references/account-setup.md` for Developer Portal registration, project roles, certification, go-live, and merchant registration.
- `references/source-map.md` to pick the official page for a question and to see coverage.

## Core Workflow

1. Identify the caller's role. Merchant, acquirer (bank or non-bank RPP participant), TPA for FPX, or issuer. If the user is a normal merchant, say they need an acquirer or gateway and stop unless they are an approved FPX merchant or TPA.
2. Identify the product: FPX, DuitNow Pay, DuitNow Online Banking/Wallets v3, DuitNow Request, AutoDebit/Consent, Reversal, or DuitNow QR.
3. Identify the environment: UAT (Certification Centre simulator or System Verification) or production.
4. Create a local pending payment with your own unique ID (`fpx_sellerOrderNo`/`fpx_sellerExOrderNo` for FPX, `checkoutId` UUID v4 for DuitNow Pay, `businessMessageId` for v3) before calling PayNet.
5. Sign every request with the product's method (FPX RSA-SHA1 checksum, DuitNow Pay JWT RS256, v3 JWS with `ds` digest). Keep private keys server-side.
6. Redirect the payer in the documented way (FPX full-window form POST; DuitNow bank URL with `EndtoEndId`, `EndtoEndIdSignature`, `DbtrAgt`).
7. Treat the browser return as UX only. Settle from the verified server-to-server message: FPX direct AC (reply `OK`), DuitNow Pay `Update Payment Status` webhook, or v3 webhook.
8. Requery when the server message is missing: FPX AE, DuitNow Pay `GET /v2/bw/checkout-status`, v3 `POST /v3/transactions/enquire`. Respect rate limits.
9. Settle idempotently on the documented final state only (FPX `fpx_debitAuthCode` `00`; DuitNow Pay substate `CLEARED`).

## Core Facts

- New docs: `https://docs.paynet.my`. Old docs `https://docs.developer.paynet.my` say they are no longer updated.
- Developer Portal: `https://developer.paynet.my`, sign-up through `https://account.paynet.my`. New companies must be verified by PayNet (`ask@paynet.my`) before go-live.
- DuitNow Pay merchants "need to be affiliated with our merchant acquirers' network". Acquirers must already be RPP participants with Online Banking/Wallets, Consent and AutoDebit enabled.
- FPX UAT: AR `https://uat.mepsfpx.com.my/FPXMain/seller2DReceiver.jsp`, AE `.../sellerNVPTxnStatus.jsp`, BE `.../RetrieveBankList`. Production FPX endpoint URLs are not in the public docs.
- FPX checksum: RSA with SHA-1 over pipe-joined values in field-name order, hex uppercase, sent as `fpx_checkSum`.
- FPX status: `00` success, `99` pending authorisation (B2B), `09` pending, anything else failed. Direct AC must be answered with `OK` or FPX resends.
- DuitNow Pay auth: `Authorization` JWT RS256 with claims `exp` (within 1 hour), `iss` (BIC), `sub` (Merchant ID), `jti` (UUID v4), `key` (Project ID), `data` (request body). Headers `X-Ip-Address` and `X-Gps-Coordinates` are required (`0` if unknown).
- DuitNow Pay API base host is not documented publicly. The sample `redirectUrl` host is `https://duitnow-pay.uat.inet.paynet.my:8443`.
- DuitNow Pay status polling: once per 30 seconds per transaction; final enquiry after 60 minutes; corporate flow enquiry on day 5. Refunds within 90 days.
- RPP v3 JWS: header `alg`/`typ`/`kid`, claims `iss`, `iat`, `exp` (within 1 hour), `key`, `jti` (= businessMessageId), `ds` (SHA-256 hex of the minified body). The text says RS512; the official code samples use RS256.
- DuitNow QR: EMV TLV, ID `00` first, CRC `63` last, CRC-16/CCITT polynomial `1021` init `FFFF`, AID `A0000006150001`, currency `458`, country `MY`.
- All PayNet APIs require TLS 1.2. Rate-limited calls return 429 `Quota has been exceeded`. PayNet strongly discourages sandbox load tests.

## Critical Rules

- Never tell a normal merchant they can sign up with PayNet for card or FPX acceptance. Route them to an acquirer or gateway.
- Never mark paid from the browser redirect, the indirect AC alone, `redirectUrl` return, or a QR display.
- For FPX, verify `fpx_checkSum` on every AC with the current FPX certificate, then the old one, and reject on failure.
- For FPX, do not whitelist FPX IPs, do not hardcode min/max limits, never use an iframe or pop-up, and show the FPX logo, operating hours and T&C link.
- For DuitNow Pay, settle on `paymentStatus.substate` = `CLEARED`, not on `code` (the docs' success samples show `ACTC`/`U002`).
- For v3, verify the response JWS signature with PayNet's certificate (`kid` = serial) and recompute `ds` over the received body.
- Use `possibleDuplicate`/`PssblDplct` = true only for a repeat of the same request; never send a new ID for a retry.
- Do not copy private keys from PayNet sample zips. Generate your own 2048-bit RSA key.
- Do not copy `CURLOPT_SSL_VERIFYPEER false` from the FPX PHP sample. Verify TLS.
- Many v3 specs say "beta" or "specification review". Confirm availability with `commercial@paynet.my` before building.

## Completion Criteria

A PayNet integration is done only when:

1. The caller's participant role and product are confirmed and the account is approved in the Developer Portal.
2. Requests are signed server-side with the correct method, and keys and certificates load from secret storage.
3. The server-to-server result (FPX direct AC, DuitNow Pay webhook, v3 webhook) is verified, acknowledged, and stored before fulfilment.
4. Settlement checks own order ID, amount, currency MYR, status code, and environment.
5. Missing callbacks are handled by the documented enquiry with the documented timing and rate limit.
6. Duplicate callbacks, repeats, SAF resends and late corporate approvals are idempotent.
7. Certification Centre and System Verification test cases are passed.
8. Tests cover bad signature, wrong amount, duplicate callback, pending `09`/`99` or `PENDAUTH`, rejected status, and success.

## Loophole Review Loop

When asked whether a PayNet strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser, merchant server, acquirer server, PayNet API gateway, FPX, issuer bank, webhook receiver, key store, certificate files, database, fulfilment job, refund job, Developer Portal settings.
2. Check bypasses: forged indirect AC, unsigned or wrongly ordered checksum, expired FPX certificate, old certificate not tried, JWT reused after expiry, `ds` not checked, webhook from unknown IP, wrong environment host, wrong amount, duplicate `fpx_sellerExOrderNo`, polling faster than 30 seconds, corporate approval after 5 days, refund over 90 days, SAF repeat processed twice.
3. Add fixes: server-side signing, full checksum verification with certificate rollover, JWS and `ds` verification, IP allowlist plus signature checks, strict amount and ID matching, durable message log, idempotency keys, environment-specific config, scheduled requery.
4. Re-read the relevant section of the reference files for any gap.
5. State remaining uncertainty where it depends on PayNet onboarding (hosts, IPs, credentials, certificates, product approval) or undocumented production details.
