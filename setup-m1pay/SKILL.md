---
name: setup-m1pay
description: Set up, build, debug, review, and explain M1Pay (MobilityOne) payment integrations using official M1Pay docs, Keycloak OAuth2 client-credentials tokens, the M1Pay Payment Wall transaction API, individual payment channels, FPX bank list API, Get Transaction status requery, Host-to-Host callback URL, SHA1withRSA signedData signing and verification, UAT/sandbox vs production setup, merchant portal keys, IP whitelisting, and FPX error codes. Use when working with M1Pay, MobilityOne, m1pay.com.my, m1payall.com, FPX B2C/B2B1, cards, eM-ONEi, Alipay, Boost, UMobile, e-wallet checkout, callback settlement, or production readiness.
---

# M1Pay Payment Gateway

Use this skill for M1Pay (MobilityOne Sdn Bhd) payment gateway integration work.

## Source

Read `references/m1pay-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, environments, OAuth2 token call, create transaction fields, Get Transaction fields, callback fields, the exact signing and verification strings, status values, channels, FPX bank list, FPX error codes, and documented gaps.

Read `references/account-setup.md` when the user asks how to register, which credentials are needed, where the Client ID, Client Secret and key files live, sandbox (UAT) setup, merchant portal settings, env var names, payment channels, pricing, or payout timing.

Read `references/source-map.md` when choosing which official M1Pay page applies, or to check what the docs do and do not cover.

## Core Workflow

1. Identify environment: UAT (sandbox, `m1payall.com` hosts) or production (`m1pay.com.my` hosts).
2. Confirm merchant portal setup: whitelisted server IP, Web Redirect URL, Host-to-Host Callback URL, Client ID, Client Secret, private key file, M1Pay public certificate.
3. Create a local pending payment with a unique `merchantOrderNo` before calling M1Pay.
4. Get an `access_token` server-side from the Keycloak token endpoint (`grant_type=client_credentials`).
5. Build the signing string, sign it with SHA1withRSA using the merchant private key, hex-encode it as `signedData`.
6. `POST` the create transaction call (form-urlencoded, `Authorization: Bearer <token>`); store the returned `transactionId`; redirect the customer to the returned M1Pay URL.
7. Treat the browser return to the Web Redirect URL as UX only. Call Get Transaction server-side to read the real status.
8. On the Host-to-Host callback, verify `signedData` with the M1Pay public certificate over the documented callback string before trusting it.
9. Settle idempotently only when `merchantOrderNo`, amount, currency and the channel's success status all match.
10. Requery with Get Transaction when the callback is missing or the redirect result is unclear.

## Core Facts

- Token: `POST https://keycloak.m1pay.com.my/auth/realms/master/protocol/openid-connect/token` (sandbox) or `.../realms/m1pay-users/...` (production).
- Create transaction: `POST https://gateway.m1payall.com/m1paywall/api/transaction` (sandbox) or `https://gateway.m1pay.com.my/wall/api/transaction` (production).
- Get Transaction: `GET .../m1paywall/api/m-1-pay-transactions/{transactionId}` (sandbox) or `.../wall/api/m-1-pay-transactions/{transactionId}` (production).
- FPX bank list: `GET .../bank-list/B2C` and `.../bank-list/B2B1` (public, no auth).
- Request and callback bodies are `x-www-form-urlencoded`; currency is the constant `MYR`.
- Amount is in ringgit (`1000` or `1000.00`, no commas); the signing string needs 2 decimals (`10.00`).
- Request signing string: `productDescription|transactionAmount|exchangeOrderNo|merchantOrderNo|transactionCurrency|emailAddress|merchantId`.
- Callback verification string: `transactionAmount|fpxTxnId|sellerOrderNo|status|merchantOrderNo`.
- Algorithm both ways: SHA1withRSA (RSA PKCS#1 v1.5 with SHA-1), signature as a hex string.
- Success status differs per channel: FPX `APPROVED`, CARD_PAYMENT `CAPTURED`, EMONEI/ALIPAY/UMOBILE `SUCCESSFUL`, BOOST `COMPLETED`.
- Omit `channel` and `fpxBank` to show the M1Pay Payment Wall; sending them empty returns an error.
- Callback retries up to 15 times at 10-minute intervals.
- UAT supports only `ONLINE_BANKING` and `CARD_PAYMENT`; test FPX bank is `SBI Bank A`.
- Merchant server IPs must be registered with M1Pay (IP filtering).

## Critical Rules

- Keep Client Secret, private key file and tokens server-side only.
- Never mark paid from the Web Redirect URL alone; requery Get Transaction.
- Verify callback `signedData` with the M1Pay certificate before parsing it into business state.
- Compare against the success value for the channel actually used; a generic "success" check is wrong.
- Format `transactionAmount` with exactly 2 decimals in the signing string, and send the same value in the body.
- Use the token, key, certificate and hosts of one environment only; never mix UAT and production.
- Make callback and requery handling idempotent; retries can arrive up to 15 times.
- Do not invent refund, cancel, recurring, payout or settlement APIs; M1Pay does not document them publicly.
- Flag undocumented details (callback ack body, redirect parameters, token lifetime, extra e-wallet channel codes) and ask M1Pay support to confirm.

## Completion Criteria

An M1Pay integration is done only when:

1. Token, create transaction and Get Transaction calls run server-side from a whitelisted IP.
2. `signedData` is generated with SHA1withRSA over the exact documented string with a 2-decimal amount.
3. The returned `transactionId` is stored against the local `merchantOrderNo` before redirecting.
4. The redirect handler only triggers a server-side Get Transaction requery.
5. The callback handler verifies `signedData` with the M1Pay public certificate and rejects failures.
6. Settlement checks `merchantOrderNo`, amount, `MYR`, channel and the channel's success status.
7. Duplicate callbacks, retries and requeries cannot double-fulfil.
8. Tests cover bad signature, wrong amount, wrong order number, failed/expired status, duplicate callback, and success per channel.
9. Deployment docs list `M1PAY_ENV`, `M1PAY_MERCHANT_ID`, `M1PAY_CLIENT_ID`, `M1PAY_CLIENT_SECRET`, `M1PAY_PRIVATE_KEY_PATH`, `M1PAY_PUBLIC_CERT_PATH`, redirect URL, callback URL and server IP.

## Loophole Review Loop

When asked whether an M1Pay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser redirect, merchant checkout, backend token client, Keycloak, M1Pay wall API, callback receiver, form parser, database, fulfilment job, merchant portal settings, key files.
2. Check bypasses: fake redirect success, forged callback, wrong field order in the signing string, amount without 2 decimals, uppercase vs lowercase hex mismatch, wrong certificate, UAT/production mix, expired token, wrong channel success value, reused `merchantOrderNo`, duplicate callback, callback never sent, IP not whitelisted, rotated private key.
3. Add fixes: server-side requery, strict signature verification, constant-time comparisons for local fields, per-channel success map, unique order numbers, durable event log, idempotent settlement, separate env config, key rotation runbook.
4. Re-read `references/m1pay-api.md` sections relevant to any gap.
5. State remaining uncertainty where the docs are silent: refunds, callback ack, redirect parameters, token expiry, extra channel codes, `fpxTxnId` for non-FPX callbacks, empty `exchangeOrderNo` handling.
