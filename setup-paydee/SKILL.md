---
name: setup-paydee
description: Set up, build, debug, review, and explain Paydee payment integrations using official Paydee developer docs, the Payment Acceptance Gateway API (MPI) spec v5.0.0, per-session RSA key exchange (mkReq), hosted payment page (mercReq), non-hosted requests (mpReq), Get Payment Channels, SHA256withRSA MPI_MAC signing and verification, Inquiry, Void, Refund, PreAuth and Sales Complete, recurring and installment (split payment), response and referral codes, payment status values, sandbox test cards, and merchant onboarding. Use when working with Paydee Malaysia, FPX, e-wallets (Boost, GrabPay, ShopeePay, TNG), DuitNow QR, Visa/Mastercard with EMV 3DS, Trade Credit, callback or redirect verification, status requery, or production readiness.
---

# Paydee Payment Gateway

Use this skill for Paydee (Paydee Sdn Bhd, formerly Synergy Cards) Payment Acceptance Gateway integration work. Paydee calls the merchant-facing API the "MPI" (Merchant Plug-In) server.

## Source

Read `references/paydee-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, environments, key exchange, request and response field tables, the exact MAC string-to-sign orders, status and error codes, void/refund rules, recurring fields, test cards, and documented conflicts between official sources.

Read `references/paydee-codes.md` when mapping any `errorCode`, `MPI_ERROR_CODE`, `MPI_REFERRAL_CODE`, `MPI_AUTH_STATUS`, or `MPI_STATUS_REASON` value.

Read `references/account-setup.md` when the user asks how to become a merchant, where credentials come from, sandbox versus production access, RSA key enrollment, payment method activation, settlement, or pricing.

Read `references/source-map.md` to pick the right official page or file (developer site, spec PDF, Swagger JSON, message-table XLSX) and to see what could not be read.

## Core Workflow

1. Identify environment: test (`https://devlink.paydee.co/mpi`, from Paydee's published Swagger) or production (endpoint released by Paydee after login/onboarding).
2. Identify surface: hosted payment page (`/mercReq`, Paydee renders checkout) or non-hosted (`/mpReq`, merchant page, plus all server-side INQ/VSALES/REFUND/RECURR/TERMINATE/SALESCOMPL calls).
3. Create a local pending payment with a new unique transaction ID (max 20 chars, minimum 6 digits; all-digit IDs are safest because the request tables type it `N`) before any Paydee call.
4. Generate a fresh RSA-2048 key pair for this payment session. Call `POST /mkReq` (JSON) with `merchantId`, `purchaseId` (= the transaction ID), `pubKey` (Base64Url), optional `paymentMethod`, and `mac` if key-exchange MAC is enrolled. Store Paydee's returned session `pubKey`.
5. Build the `MPI_*` form fields, compute `MPI_MAC` = Base64Url(SHA256withRSA(merchant session private key, concatenated values in the documented order)).
6. Hosted: auto-post the form from the cardholder browser to `/mercReq`. Non-hosted SALES: post to `/mpReq` with `MPI_PAYMENT_CHANNEL_ID` from `/channels`.
7. On the redirect and on the callback, verify the response `MPI_MAC` with Paydee's session public key over the documented response field order before trusting any field.
8. Settle only when `MPI_MERC_ID`, `MPI_TRXN_ID`, amount and currency match the local record, `MPI_ERROR_CODE` is `000` and `MPI_STATUS` is `1` (Payment Success, or `7` Payment Settled on a later INQ). Make settlement idempotent.
9. If no response, timeout, or browser closed: run `INQ` (new key exchange, new transaction ID, `MPI_ORI_TRXN_ID` = original) before retrying a sale.
10. Void before cut-off, otherwise refund; both are asynchronous, so confirm with `INQ` (Paydee suggests about 30 minutes later) and read `MPI_STATUS`.

## Core Facts

- Every API call is HTTPS POST except Get Payment Channels (`GET /channels`, JSON body per spec).
- Endpoints: `/mkReq` (key exchange, JSON), `/mercReq` (hosted page, form data), `/mpReq` (non-hosted and server operations, form data `application/x-www-form-urlencoded`), `/channels` (channel list).
- Test server in the official Swagger: `https://devlink.paydee.co/mpi`. Production URL is locked behind developer-portal login.
- Key exchange must precede every other call. `purchaseId` in `/mkReq` equals `MPI_TRXN_ID` in the next request; each new operation (INQ, VOID, REFUND, RECURR) uses a new ID and its own key exchange.
- Signature: SHA256withRSA, Base64Url encoded, signed with the sender's private key, verified with the peer's public key exchanged in `/mkReq`.
- Amounts are integers in minor units (MYR 10.23 = `1023`); currency is ISO 4217 numeric (`458` = MYR). The XLSX says Paydee only accepts MYR.
- `MPI_PURCH_DATE` is `yyyyMMddHHmmss` (24h).
- `MPI_EMAIL` is mandatory; billing address is conditional (C1); at least one phone pair is required (C2). Visa rules effective 12 Feb 2024 drive these.
- Payment methods (`paymentMethod` / `MPI_CUST_PAYMENT_METHOD`): `CC`, `EWALLETS`, `FPX`, `TRD` (Trade Credit), `DNQR` (DuitNow QR).
- `MPI_STATUS`: `0` requested, `1` success, `2` cancelled, `3`/`4` reverse requested/success, `5`/`6` refund requested/success, `7` settled, `8` recurring terminated, `-1` failed, `-2` reverse failed, `-3` refund failed, `-4` sales request failed.
- Paydee asks that the checkout page be implemented in an iframe per EMV 3-D Secure 2.0.
- SALESCOMPL is rejected if the final amount exceeds the PREAUTH amount by more than 20%.
- Online banking (FPX) refund function was removed in spec v3.0.3 (1 July 2025).
- Published sandbox test cards: Visa `4100000000000100`, Mastercard `5100000000000107` (frictionless success); challenge cards use OTP `123456`. Sandbox only.

## Critical Rules

- Keep RSA private keys server-side. Never send card data from a non-PCI server; non-hosted card fields (`MPI_PAN`, `MPI_CVV2`) put you in PCI DSS scope.
- Never mark paid from the browser redirect alone; verify `MPI_MAC` on both redirect and callback, and use INQ as the authority when in doubt.
- Build the MAC string from exactly the documented field order, using empty strings for absent fields. Confirm whitespace handling with Paydee (samples show spaces removed).
- Verify with the Paydee public key from the same session's `/mkReq` response, not a cached key from another session.
- Treat `004` (in processing) and void/refund acknowledgements as non-final.
- Never reuse a transaction ID; a duplicate is rejected (`594`) and the pair (merchant ID, transaction ID) identifies the transaction.
- Keep test and production merchant IDs, keys and endpoints separate.

## Completion Criteria

A Paydee integration is done only when:

1. Each payment session generates its own RSA key pair and runs `/mkReq` server-side; Paydee's session key is stored with the payment.
2. Request `MPI_MAC` is produced over the documented order and accepted by the test server (no `5A0`).
3. Redirect and callback handlers verify response `MPI_MAC`, merchant ID, transaction ID, amount and currency before settlement.
4. Settlement requires `MPI_ERROR_CODE=000` and `MPI_STATUS=1`, is idempotent, and handles duplicates.
5. INQ fallback exists for timeout, closed browser and pending states; void/refund outcomes are confirmed by INQ.
6. Non-hosted flows load `/channels` and disable blocked (`=B`) channels.
7. Tests cover bad MAC, wrong amount, wrong transaction ID, failed card (`4100000000300005`), challenge flow, and duplicate callback.
8. Deployment docs list `PAYDEE_ENV`, `PAYDEE_ENDPOINT`, `PAYDEE_MERCHANT_ID`, enrolled key-exchange key path (if used), and the callback/redirect URL agreed with Paydee.

## Loophole Review Loop

When asked whether a Paydee strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: cardholder browser, hosted page, merchant checkout page, `/mkReq` caller, key store, MAC builder, callback receiver, redirect handler, INQ job, void/refund job, recurring scheduler, database.
2. Check bypasses: forged redirect, MAC verified against the wrong session key, MAC string built with wrong order or whitespace, missing empty fields, reused transaction ID, amount in ringgit instead of sen, test/production mix, trusting `004`, trusting a void/refund ack, blocked channel shown as selectable, SALESCOMPL above 120%, recurring sent without original transaction ID.
3. Add fixes: per-session keys, strict field order, constant-format amount, server-side INQ confirmation, idempotency key on `(merchant_id, trxn_id)`, durable callback log, separate env config.
4. Re-read `references/paydee-api.md` sections relevant to any gap, especially "Documented conflicts".
5. State remaining uncertainty: callback URL setup, production endpoint, recurring field names, whitespace in MAC string, response MAC order, and IP allowlists are not fully specified in public docs and must be confirmed with Paydee (help@paydee.co).
