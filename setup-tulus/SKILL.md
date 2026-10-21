---
name: setup-tulus
description: Set up, build, debug, review, and explain Tulus (also branded SnapNPay) payment integrations using official Tulus Developer Portal docs, the Tulus PHP SDK, the Laravel handoff, /v2/checkout signed order POST, hosted pay URLs, HMAC-SHA256 truncated signatures with the UUID API key as 16 raw bytes, direct (signed) and indirect (browser) callbacks to returnUrl, fpx_debit_auth_code mapping, Report API reconciliation on reports.tulus.my, AuthToken and API key auth, QR and DuitNow QR generators, zakat agency codes and form schemas, and staging vs production. Use when working with Tulus Digital, SnapNPay, tulus.my, pay.tulus.my, reports.tulus.my, zakat, wakaf, infaq or sedekah collection, FPX, cards (MPGS), DuitNow, Touch 'n Go, callbacks, reconciliation, or production readiness.
---

# Tulus (SnapNPay) Payment Gateway

Use this skill for Tulus payment gateway integration work. Tulus Digital Sdn Bhd runs one gateway under two brands, Tulus and SnapNPay; the API, field names and signing are the same.

## Source

Read `references/tulus-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, hosts, auth, checkout fields, the signature algorithm with verified test vectors, callback fields, report API, QR generators, and the known doc inconsistencies.

Read `references/tulus-codes.md` for the full `fpx_debit_auth_code` table, status mapping, zakat agency codes, zakat type and state enums, and form schema fields.

Read `references/tulus-partner-apis.md` only when the user is a partner pushing transactions into Tulus (ADB, Grab webhooks) or customising receipts and receipt emails.

Read `references/account-setup.md` for onboarding, where the AuthToken and API key live, staging vs production, env var names, and settlement notes.

Read `references/source-map.md` to pick the right official page.

## Core Workflow

1. Confirm the agency code (assigned at onboarding) and that the merchant has an API key (UUID) from the merchant dashboard. The API key is required to sign orders and verify callbacks.
2. Confirm the environment and checkout host with Tulus (see "Hosts" in `tulus-api.md`; docs and live behaviour differ).
3. Create a local pending payment keyed by your `refNo` before redirecting.
4. Server-side, build the order fields `agency`, `refNo`, `amount`, `email`, `returnUrl` (plus optional `cancelUrl`), add `exp = now + 60`, sign, and render an auto-submitting POST form to `/v2/checkout`.
5. Handle both POSTs on `returnUrl`: the direct callback (has `signature`, server-to-server, authoritative) and the indirect browser return (no `signature`, display only).
6. Verify the direct callback signature over all POST fields except `signature`, sorted, Go `url.QueryEscape` encoding, no path, no `exp`.
7. Settle only on verified `status=success`, after matching `refNo`, `agency` and the net `amount`; answer HTTP 200 quickly.
8. Reconcile pending or missed payments with the signed Report API (`/api/report/v1/fpxrequest` or `/api/report/v1/payment`).

## Core Facts

- Documented checkout: `POST https://tulus.my/v2/checkout`, `application/x-www-form-urlencoded`. On 2026-10-21 that URL returned 404 while `https://pay.tulus.my/v2/checkout` answered; the SDK playground posts to `https://pay.tulus.my/` (production) and `https://paydev.tulus.my/` (staging, FPX test banks). Confirm the host with Tulus.
- Reporting host: `https://reports.tulus.my` (also the merchant dashboard).
- Auth: AuthToken JWT (`Authorization: Bearer`) for reports, copied from the dashboard; or per-agency API key UUID used only as an HMAC key. The old `token` / username+password method is deprecated.
- Signature: `HMAC-SHA256(message, key)` hex, first 16 characters. Key is the UUID with dashes removed, hex-decoded to 16 raw bytes.
- Outbound message: `<path>?<ksorted RFC 3986 query including exp>`, for GET reports and POST checkout/QR alike. Checkout path is `/v2/checkout`.
- Callback message: ksorted `k=v&...` of all POST fields except `signature`, Go `url.QueryEscape` encoding (space is `+`), no path, no `exp`.
- `amount` is a ringgit string with 2 decimals; minimum `1.00` (FPX B2B `2.00`). `refNo` must not contain `~`; `~` appends extraData.
- Direct callback `status`: `success`, `pending`, `failed`. Retried about every 30 seconds until HTTP 2xx.
- Direct callback `amount` is net; `addCharge` is the platform fee; indirect return `amount` is gross.
- Unsigned or wrongly signed checkouts are rejected with HTTP 403 in production.
- Live test payments should use an amount ending in `12.34`.
- Refunds, cancel/void, recurring and tokenisation APIs: not documented in official docs (checked 2026-10-21).

## Critical Rules

- Keep the API key and AuthToken server-side only; the key never goes on the wire.
- Never fulfil from the indirect browser return or from `cancelUrl`.
- Never HMAC the ASCII UUID, never use the full 64-char digest, never add `exp` or a path when verifying callbacks.
- Do not add unsigned fields (for example a CSRF `_token`) to the checkout form; every posted field is checked.
- Exempt `returnUrl` from CSRF; respond 200 without redirects.
- Compare signatures in constant time.
- Settle idempotently: the same success can arrive more than once (retries and dashboard re-sends); a recorded success never regresses.
- Cross-check the signed net `amount` against your stored amount.
- Read `fpx.payment_method` to tell FPX from card (`mpgs`) and others; field names stay `fpx_*` for all methods.
- Do not use the downloadable WooCommerce plugin as a reference for verification: its signature check is disabled and it does not add `exp`.

## Completion Criteria

A Tulus integration is done only when:

1. The signer passes both official test vectors in `tulus-api.md`.
2. Checkout is signed server-side at redirect time (60-second `exp`) and posts only signed fields.
3. `returnUrl` is public HTTPS, CSRF-exempt, and handles direct and indirect POSTs separately.
4. Direct callbacks are verified, matched on `refNo`, `agency`, net `amount`, and settled once on `success`.
5. `pending` stays pending; a scheduled job reconciles via the signed Report API.
6. Tests cover bad signature (403), replayed success (no double fulfil), tampered amount, unsigned indirect POST, and `pending` then `success`.
7. Deployment docs list `TULUS_AGENCY`, `TULUS_API_KEY`, `TULUS_CHECKOUT_URL`, `TULUS_REPORT_URL`, `TULUS_RETURN_URL`, and staging vs production values.

## Loophole Review Loop

When asked whether a Tulus strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: checkout form, buyer browser, Tulus checkout host, bank/FPX, direct callback, indirect return, `cancelUrl`, database, fulfilment job, Report API, merchant dashboard.
2. Check bypasses: forged indirect POST, callback without `signature`, signature over the ASCII key, wrong encoding of `~` or space, stale `exp`, unsigned extra form field, amount gross vs net confusion, wrong agency, duplicate success, late `pending` after `success`, wrong checkout host, staging key on production.
3. Add fixes: server-side signing at redirect, constant-time verify, strict refNo/agency/amount match, row lock plus idempotent fulfil, terminal-state guard, signed Report API fallback, separate env config.
4. Re-read `references/tulus-api.md` sections relevant to any gap.
5. State remaining uncertainty: checkout host, staging credentials, payment methods and fees enabled for the agency, settlement timing, and any undocumented refund process.
