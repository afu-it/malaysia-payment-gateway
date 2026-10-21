---
name: setup-omise
description: Set up, build, debug, review, and explain Omise (formerly Opn Payments) payment integrations using official Omise docs, Charges API, Sources API, Tokens API on vault.omise.co, Omise.js, 3-D Secure, authorize_uri redirects, webhooks with Omise-Signature HMAC-SHA256 verification, Events API, refunds, voids, captures, Customers and Schedules for recurring charges, transfers and recipients, Payment Links+, test mode, API keys, API versions, and Malaysian payment methods. Use when working with Omise Malaysia, MYR, FPX, DuitNow Online Banking/Wallets, DuitNow QR, Touch 'n Go, Boost, GrabPay, ShopeePay, Maybank QR, Atome, Google Pay, Maybank installments, cards, webhook settlement, refunds, payouts, or production readiness.
---

# Omise Payment Gateway

Use this skill for Omise payment gateway integration work for Malaysian merchants (account country Malaysia, funding currency MYR).

## Source

Read `references/omise-api.md` before giving factual API guidance or writing integration code. It is the entry point: environments, authentication, API versions, the charge flow, webhook signature, statuses, amount rules, rate limits, IPs, and the full Endpoint catalogue. It links to the topic files:

- `references/omise-charges.md`: Charge object, create/capture/reverse/expire, card tokens, Omise.js, 3-D Secure.
- `references/omise-malaysia-methods.md`: FPX, DuitNow OBW, DuitNow QR, Touch 'n Go, Boost, GrabPay, ShopeePay, Maybank QR, Atome, Google Pay, installments, Source API fields, bank codes, limits.
- `references/omise-webhooks.md`: signature verification, event list, Events API, dynamic webhooks.
- `references/omise-refunds-disputes.md`: refunds, voids, refund windows, disputes and evidence.
- `references/omise-recurring.md`: Customers, Cards, Schedules, Occurrences, charge schedule bulk APIs.
- `references/omise-payouts-reporting.md`: Balance, Transactions, Transfers, Recipients, Malaysian bank codes, Receipts, Search.
- `references/omise-payment-links.md`: Payment Links+ API.
- `references/omise-testing-errors.md`: test mode, test cards, simulation, error codes, failure codes.

Read `references/account-setup.md` for registration, live account documents, where keys and webhook secrets live, payment method activation, holding period, payouts, and pricing links.

Read `references/source-map.md` to pick the official page behind any fact, and for coverage notes.

## Core Workflow

1. Identify mode: test keys (`pkey_test_...`, `skey_test_...`) or live keys. Same API hosts for both; the key decides the mode.
2. Identify surface: Omise.js pre-built form, Omise.js custom form, server-only source charge, mobile SDK, plugin (WooCommerce, Magento, Shopify, OpenCart, PrestaShop), Payment Links+, or Schedules.
3. Confirm the payment method is enabled on the account (each non-card method is enabled by emailing support) and check live availability with `GET /capability`.
4. Create a local pending order with the expected amount in sen and currency `MYR`.
5. Browser: create a card token (`vault.omise.co/tokens`) or a source (`api.omise.co/sources`) with the public key. Or create the source inline in the charge on the server.
6. Server: `POST https://api.omise.co/charges` with the secret key, `amount`, `currency`, `card` or `source`, `return_uri`, `metadata[order_id]`, and a unique `ref_id` for idempotency. Store the `chrg_...` id.
7. If `status` is `pending` and `authorize_uri` is set, redirect the customer to `authorize_uri`. Atome QR is offline: show `source.scannable_code.image.download_uri` instead.
8. Treat the return to `return_uri` as UX only. Settle from a verified `charge.complete` webhook and a fresh `GET /charges/{id}`.
9. Verify `Omise-Signature` (HMAC-SHA256, hex) over `<Omise-Signature-Timestamp>.<raw body>` with the Base64-decoded webhook secret; accept if any comma-separated signature matches.
10. Mark paid only when the re-fetched charge has `status=successful` (`paid=true`), matching id, amount, currency, `livemode`, and order metadata. Make it idempotent.

## Core Facts

- API base: `https://api.omise.co`. Token vault: `https://vault.omise.co` (tokens only; tokens on api.omise.co return `invalid_domain`).
- Auth: HTTP Basic, key as username, blank password (`-u $OMISE_SECRET_KEY:`). Public key only creates/views tokens and sources; secret key does everything else.
- Request bodies are form-encoded (`-d "source[type]=fpx"`). Responses are JSON.
- Latest API version `2019-05-29`; override per request with header `Omise-Version`. Webhook payloads use the account default version at event time.
- Amounts are integers in the smallest unit: `100` = RM1.00. Malaysia charge limits `100` to `10000000` (RM1 to RM100,000); methods have tighter limits. Search API uses the main unit instead.
- Charge `status`: `pending`, `successful`, `failed`, `expired`, `reversed`. Failure detail in `failure_code` and `failure_message`.
- Source `flow`: `redirect` and `app_redirect` use `charge.authorize_uri`; `offline` uses `charge.source.scannable_code` or `references`.
- Malaysian source types: `fpx`, `duitnow_obw`, `duitnow_qr`, `touch_n_go`, `boost`, `grabpay`, `shopeepay`, `shopeepay_jumpapp`, `maybank_qr`, `atome`, `atome_qr`, `installment_mbb`. Cards use tokens.
- FPX and DuitNow OBW need `bank`; FPX banks come from `payment_methods[name=fpx].banks` in `GET /capability`, not the top-level `banks` (that list is for transfers).
- Webhook headers: `Omise-Signature`, `Omise-Signature-Timestamp`. Secrets are per mode, rotated with a 24 hour dual-signature window.
- `charge.complete` fires for 3DS and asynchronous methods, not for non-3DS card charges; read the synchronous charge response there.
- Omise does not guarantee webhook retries (webhooks page). Reconcile with `GET /events` or `GET /charges/{id}`.
- Dashboard webhook URLs: `https://dashboard.omise.co/test/webhooks` and `https://dashboard.omise.co/live/webhooks`. One static endpoint per mode; per-charge `webhook_endpoints` (max 2 HTTPS URLs) overrides it.
- Webhook source IPs: `54.169.118.227`, `52.74.199.175`, `18.139.13.19`. API IPs: `76.223.19.23`, `13.248.150.88`. JSON list: `https://cdn.omise.co/ips.json`.
- Rate limit errors are HTTP 429 `too_many_requests`. No numeric limit is published; Omise advises at most 10 parallel workers.
- Refunds: `POST /charges/{id}/refunds` with `amount`; under 15 refunds per charge, within 365 days at most, method windows are shorter (FPX six months, DuitNow/TnG/Boost/ShopeePay/Maybank QR 180 days, GrabPay three months, Atome 60 days). Refund before settlement is processed as a void.
- Uncaptured card authorizations expire after 7 days (Malaysia, non-Japan accounts).
- Payouts (Malaysia): 3 day hold, transfers batched once per business day, deposit in 3 business days, minimum RM1.00 transferable.
- Multi-currency is not available for Malaysian accounts: charge and settle in MYR.
- Status page: `https://status.omise.co`.

## Critical Rules

- Keep the secret key and webhook secret server-side. Never put the secret key in Omise.js, mobile apps, or repos.
- Never send raw card data through your server unless PCI DSS certified. Use Omise.js or the mobile SDKs to get a `tokn_...`.
- Tokens are single use. Store customers and cards (`cust_...`, `card_...`) for reuse, never tokens.
- Do not fulfil from `return_uri`, Omise.js callbacks, or an unverified webhook. Verify the signature over the raw body, then re-fetch the charge.
- Use constant-time comparison and support two signatures during secret rotation.
- Source `amount` and `currency` must equal the charge `amount` and `currency`.
- Send a unique `ref_id` per order attempt on charge create so a retried request cannot double-charge.
- Use `return_uri` over HTTPS; for app-switch methods on mobile apps use a deep link or app link.
- Keep test and live keys, webhook URLs and webhook secrets separate.
- Handle unknown event types and unknown fields without failing.
- Do not quote fees from memory. Link `https://www.omise.co/en/pricing/malaysia`.

## Completion Criteria

An Omise integration is done only when:

1. Charges are created server-side with the secret key; the browser only sees the public key.
2. Card flow handles both an immediate `successful` response and a `pending` charge with `authorize_uri` (3DS).
3. Every redirect method sends the customer to `authorize_uri` and returns to an HTTPS `return_uri`.
4. The webhook endpoint (HTTPS, valid certificate) verifies `Omise-Signature` over `timestamp.rawBody` with the decoded secret.
5. Settlement re-fetches the charge and checks id, `status`, `paid`, `amount`, `currency`, `livemode`, and order metadata before marking paid, once.
6. A reconciliation job re-fetches pending charges or lists events, since webhook retries are not guaranteed.
7. Tests cover bad signature, rotated dual signature, duplicate event, wrong amount/currency, `failed` and `expired` charges, a 3DS success via test card, and an FPX charge marked successful and failed in test mode.
8. Deployment docs list `OMISE_PUBLIC_KEY`, `OMISE_SECRET_KEY`, `OMISE_WEBHOOK_SECRET`, the webhook URL per mode, and enabled methods.

## Loophole Review Loop

When asked whether an Omise strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser, Omise.js iframe, mobile SDK, your charge API, api.omise.co, vault.omise.co, bank/wallet page at `authorize_uri`, `return_uri` handler, webhook receiver, raw-body parser, database, fulfilment, refund job, dashboard settings.
2. Check bypasses: forged return redirect, forged webhook, signature over re-serialized JSON, wrong-mode secret, only first signature checked during rotation, token reuse, source amount differing from order, missing `ref_id` on retry, non-3DS card with no webhook, pending charge never reconciled, expired QR or ShopeePay charge, method not enabled, off-us ShopeePay refund impossible, partial refund on installments, refund after window.
3. Add fixes: server-side charge creation, raw-body HMAC with constant-time compare, re-fetch before settle, unique index on charge id, `ref_id`, reconciliation via Events API, explicit `expires_at`, capability check, separate mode config.
4. Re-read the relevant reference file for any gap.
5. State remaining uncertainty that depends on account settings: 3DS enablement, enabled methods, method limits, holding period, fees, and webhook retry behavior.
