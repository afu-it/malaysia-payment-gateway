---
name: setup-gomobi
description: Set up, build, debug, review, and explain GoMobi payment integrations using official GoMobi docs, the Mobi hosted checkout API with HMAC-SHA256 request signing, AES-CBC (PBKDF2) payload encryption and checksums, 3DES mobiApiKey headers, FPX B2C/B2B direct form posts, cards with save-card tokens, Boost, GrabPay, ShopeePay and Touch 'n Go e-wallets, DuitNow QR, 7-Eleven cash over counter (EZYCASH), payouts, IPN callbacks, status requery, sandbox/live setup, IP whitelisting, and plugins. Use when working with GoMobi, Mobi, Mobi Asia, Mobiversa, gomobi.io, mobiApiKey, motoApiKey, loginId, FPX, DuitNow QR, e-wallets, cards, payouts, IPN settlement, or production readiness in Malaysia.
---

# GoMobi (Mobi) Payment Gateway

Use this skill for GoMobi integration work. GoMobi is now branded Mobi (Mobi Asia Sdn Bhd, formerly Mobiversa). Developer docs still live at `https://gomobi.io/documents`.

## Source

Read `references/gomobi-api.md` first. It is the entry point: environments, credentials, the full endpoint catalogue, hosted checkout signing, AES-CBC and 3DES algorithms, FPX, status, IPN rules, amount formats and doc conflicts.

Read `references/gomobi-ewallets-qr.md` for Boost, GrabPay, ShopeePay, Touch 'n Go, DuitNow QR and 7-Eleven cash (EZYCASH).

Read `references/gomobi-cards-payout.md` for card payment, saved-card (walletId) charges, and Malaysia payouts.

Read `references/gomobi-codes.md` for FPX bank codes, FPX and card response codes, payout BIC codes and error message lists.

Read `references/account-setup.md` for onboarding, credentials, sandbox vs live, env var names, IP whitelisting, settlement and pricing.

Read `references/source-map.md` to find the right docs page and to see coverage.

## Core Workflow

1. Identify environment (staging `m-uat.gomobi.io` family or production hosts) and product: hosted checkout, FPX direct, cards, e-wallet, DuitNow QR, cash, or payout.
2. Confirm Mobi has issued the right credentials for that product: mobiApiKey, loginId, mobiSecretKey, salt, FPX param1/param2, mid/tid, subMID, or checkout API key plus API secret.
3. Create a local pending payment with a unique order/invoice id before calling Mobi.
4. Build the request server-side with the product's exact signing or encryption scheme from `gomobi-api.md`.
5. Redirect the customer (checkout_url, FPX form, card form, wallet URL) or show the QR.
6. Treat the browser redirect or callback page as UX only.
7. Settle from the IPN after validating it (decrypt or recompute signature, match amount, order id, status), then confirm with the status API.
8. Requery pending payments after the documented expiry plus 5 to 10 minutes; do not mark failed too early.
9. Make IPN handling idempotent on Mobi's transaction id and your order id.

## Core Facts

- Hosted checkout: `POST https://middleware.gomobi.io/api/payment/v2/checkout/init` (staging `https://checkout.gomobi.io/...`), Bearer base64 API key, `X-Signature` HMAC-SHA256 over `METHOD + PATH + TIMESTAMP + NONCE + SHA256(body)`, session valid 10 minutes, single use.
- AES-CBC: AES/CBC/PKCS5Padding, key = PBKDF2WithHmacSHA256(mobiKey or param1, salt or param2, 65536 iterations, 256 bits), random 16-byte IV, output base64(IV || ciphertext).
- 3DES: DESede/ECB/PKCS5Padding, key = first 24 bytes of mobiSecretKey, plaintext = mobiKey, output base64. Used for the `mobiApiKey` header on DuitNow QR and EZYCASH, and to decrypt payout IPN and FPX POST IPN signatures.
- FPX: B2C RM1 to RM30,000, B2B RM2 to RM1,000,000, expires 15 minutes, success = `fpx_debitAuthCode` `00`.
- Amounts: e-wallets, cards and FPX status use 12-digit sen strings (`000000001000` = RM10.00); checkout, FPX deposit, DuitNow, cash and payout use decimal MYR.
- Expiry: FPX 15m, Grab 15m, TNG 10m, ShopeePay 5m, Boost 5m, DuitNow QR 5m, cash 48h.
- Generic codes: `0000` success, `0001` failure.
- Sandbox FPX: bank "SBI Bank A", user `1234`, password `1234`. Sandbox card: `5452 0100 0009 5323`, CVV `123`, expiry `12/28`. Sandbox only.
- No refund or void API is documented; FPX refunds are manual through Mobi.
- Callbacks cannot be resent manually by Mobi.
- Support: `csmobi@gomobi.io` (IPN IP list, support), `getsupport@gomobi.io` (checkout API secret). Portal: `https://gomobi.io/portal/login`.

## Critical Rules

- Keep mobiApiKey, mobiSecretKey, salt, param1/param2 and the checkout API secret server-side only. Never in frontend code.
- Never use the docs' sample keys; use the values Mobi issues, from env vars.
- Do not fulfil from the redirect page, callback form post, or QR display alone.
- Validate every IPN: decrypt `responseSignature`/`payoutEncryptedResponse`, or recompute `transaction_signature`, then check order id, amount (in the right unit), status, and sub-merchant.
- Confirm with the status endpoint before fulfilling high-value orders.
- Whitelist Mobi's IPN IPs (request the list from `csmobi@gomobi.io`) as hardening, not instead of validation. Payout and some production endpoints are IP-restricted on Mobi's side.
- Never mix staging and production hosts with the wrong credentials.
- Where the docs conflict (see "Known doc conflicts" in `gomobi-api.md`), test in sandbox and confirm with Mobi before shipping.
- Do not state fees; pricing is not published.

## Completion Criteria

A GoMobi integration is done only when:

1. Requests are built server-side with the documented signature, checksum or encryption for that product.
2. Local pending payment exists before redirect or QR display.
3. The IPN endpoint is registered with Mobi (static URL at onboarding or portal, or `callback` per request), reachable over HTTPS, and whitelisted for Mobi's IPs.
4. IPN handling verifies or decrypts, checks amount, order id and status, and is idempotent.
5. Status requery covers missed IPNs and pending states after expiry.
6. Tests cover bad signature/checksum, wrong amount unit, duplicate IPN, pending then success, expired, failed, and success.
7. Deployment docs list env vars from `account-setup.md` and keep staging and production separate.

## Loophole Review Loop

When asked whether a GoMobi strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, FPX/card HTML form post, wallet redirect, QR display, backend request builder, Mobi API, IPN receiver, decryption code, database, fulfilment job, payout job, portal settings.
2. Check bypasses: fake redirect POST with `fpx_debitAuthCode=00`, forged IPN GET, IPN not decrypted or signature not checked, 12-digit vs decimal amount mismatch, reused invoiceId, wrong subMID, sandbox/live key mix, expired QR settled late, pending marked failed too early, duplicate IPN, missed IPN (no manual resend), payout to wrong account, non-whitelisted server IP.
3. Add fixes: server-side request building, local pending record, IPN decryption and signature checks, strict amount/order/status matching, status requery, durable event log, idempotency keys, separate env config, IP whitelisting.
4. Re-read the relevant reference sections for any gap.
5. State remaining uncertainty where docs conflict or are silent: card encryption method, Boost/Grab/Shopee/TNG POST IPN signature order, Grab `tid`/`sessionId` handshake, checkout body-hash encoding, IPN retry policy, refunds.
