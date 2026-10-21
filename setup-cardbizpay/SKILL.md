---
name: setup-cardbizpay
description: Set up, build, debug, review, and explain CardBizPay payment integrations using official CardBizPay developer docs, Merchant API hosted payment page (Payment/Pymt), Payment Return URL callbacks, HMAC-SHA512 base64 signatures, payment enquiry, void, refund, refund enquiry, MDR enquiry, Open Payment API card tokenization (AddCard, RemoveCard, Sale, PreAuth, Capture, PreAuthCancel, Void, Refund, TrxnEnq), staging/production setup, Merchant No, Access Key, Secret Key, and status codes. Use when working with CardBizPay or Cardbiz Payment Services, FPX B2C/B2B, DuitNow, cards, e-wallets, saved cards, pre-authorization, callback verification, requery, refunds, or production readiness.
---

# CardBizPay Payment Gateway

Use this skill for CardBizPay (Cardbiz Payment Services Sdn Bhd) payment gateway integration work.

## Source

Read `references/cardbizpay-api.md` before giving factual API guidance or writing integration code. It is the entry point: environments, endpoint catalogue, the signature algorithm, the Merchant API (hosted payment page, enquiry, void, refund, MDR), status codes, and documented doc defects.

Read `references/cardbizpay-opa.md` when the merchant needs saved cards, merchant-initiated charges, pre-authorization and capture, or the Open Payment API (`/opa/*`).

Read `references/account-setup.md` when the user asks how to register, which credentials exist, who configures the return/notify URL, sandbox access, payment methods, settlement, or pricing.

Read `references/source-map.md` to find the official page behind a fact, or to see what the public docs do not cover.

## Core Workflow

1. Identify environment: staging (`https://uat.cardbizpay.com/paymentgateway`) or production (`https://www.cardbizpay.com/paymentgateway`).
2. Identify API family: Merchant API (`/Payment/*`, hosted payment page) or Open Payment API (`/opa/*`, card tokens).
3. Confirm onboarding: merchant profile created by the CardBizPay administrator, Merchant No, Access Key, Secret Key used for signing (the docs name both and do not say whether they are the same value), API access enabled on the profile, Payment Return URL (Merchant API) or Notify URL (Open Payment API) registered with CardBizPay, payment methods subscribed.
4. Create a local pending order with a unique `OrderId` (max 25 chars for `/Payment/Pymt`) and the exact amount string you will send.
5. Build the request server-side, compute `Signature` = base64(HMAC-SHA512(secret, concatenated values in documented field order)), and auto-submit an HTML form POST from the browser to `[Base URL]/Payment/Pymt`.
6. Receive the response form POST at the Payment Return URL. Verify `Signature` over the received values in the documented response field order before trusting any field.
7. Check `RespCode = "00"`, `StatusCode = "A"`, `MerchantNo`, `OrderId`, and `TrxnAmount` against the local order. Store `TrxnId` (needed for void and refund).
8. Requery with `GET /Payment/PymtEnq` when the return post is missing, unverifiable, or not final (`N`, `W`).
9. Settle idempotently: one order transitions to paid once, no matter how many posts or enquiries arrive.

## Core Facts

- All requests are HTML form posts; enquiries are documented as `GET`. Field names are case-sensitive PascalCase (`MerchantNo`, `OrderId`, `TrxnAmount`).
- Signature: HMAC-SHA512 keyed with the merchant Secret Key, output base64. String to sign = values only, no separators, in the documented field order, skipping `Signature` and any optional field whose value is empty.
- Official test vector (reproduces): `1000000000000001M12345678910.50` with key `Ws123456Ws987654` gives `++lLP7XD4fK3t7PsBz50LdVU35yutApDvZdmlltNXuBsrQnF0B/ed1uxAjSN+3anRjJWrDelYCZoWbsyTzAuMg==`.
- Optional (`O`) fields must still be posted as parameters, with an empty value if unused.
- Amounts are ringgit strings, up to 2 decimals, no thousand separators (`"10"`, `"10.5"`, `"10.50"`). No currency field is documented.
- `ReqDate` format `yyyyMMddHHmmss`. Open Payment API rejects a `ReqDate` more than 60 minutes from host time.
- `PymtMethodSelect` values: empty (all), `Card`, `FPX`, `FPX B2C`, `FPX B2B`, `e-Wallet`, `DuitNow`; comma-separate multiple.
- `CustMobile` is required in practice when e-Wallet is subscribed; `CustEmail` must be supplied to avoid declines.
- Response code `00` = approved / accepted. Payment status: `A` approved, `D` declined, `N` pending (cancelled after 3 days unpaid), `W` awaiting status, `V` voided, `C` cancelled, `Q` partially refunded, `R` fully refunded; Open Payment API adds `P` pre-auth and `T` pre-auth cancelled.
- Void is same-day only and returns 100%; refunds may be full or multiple partial, each with a unique `RefundId` (max 20).
- Payment page session counter is 15 minutes.
- The Payment Return URL is not a request field; CardBizPay configures it on the merchant profile (editable in the Merchant Portal if you have a login).

## Critical Rules

- Keep the Secret Key server-side only; compute signatures on the server, never in browser JavaScript.
- Never mark paid from the browser landing alone or from an unsigned or badly signed post.
- Verify the response signature over the exact received string values; do not reformat amounts before hashing.
- Sign the exact strings you post. If you send `"10.5"`, sign `"10.5"`, and use the same string for later void, refund, and enquiry calls ("should be the same value when calling Payment API").
- Compare signatures in constant time.
- Use separate Merchant No and Secret Key per environment; never mix staging credentials with the production base URL.
- `OrderId` must be unique per transaction (status `22` duplicate, `13` already paid).
- Treat `N` and `W` as not final; requery, do not fail the order early.
- Make return-URL handling idempotent; duplicate posts and enquiry results must not double-fulfil.
- Where the docs are ambiguous (listed in `references/cardbizpay-api.md` under Doc Defects), confirm with CardBizPay in staging before going live.

## Completion Criteria

A CardBizPay integration is done only when:

1. Payment requests are built and signed server-side with the documented field order and all optional parameters present.
2. The Payment Return URL (and Notify URL for Open Payment API) is registered with CardBizPay for each environment.
3. The return handler verifies `Signature` over the documented response field order, with empty optional fields skipped.
4. Settlement checks `RespCode`, `StatusCode`, `MerchantNo`, `OrderId`, `TrxnAmount`, and stores `TrxnId`.
5. `PymtEnq` requery covers missing or non-final results; void and refund use stored `TrxnId` and the original amount string.
6. Tests cover bad signature, wrong amount, wrong order, duplicate post, declined (`D`), pending (`N`/`W`), and approved (`A`), plus the official signature test vector.
7. Deployment docs list `CARDBIZPAY_MERCHANT_NO`, `CARDBIZPAY_SECRET_KEY`, `CARDBIZPAY_BASE_URL`, and the registered return URL per environment.

## Loophole Review Loop

When asked whether a CardBizPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser form post, CardBizPay payment page, Payment Return URL handler, Notify URL handler, enquiry calls, database, fulfilment, void/refund jobs, Merchant Portal settings.
2. Check bypasses: forged return post, signature over wrong field order, amount reformatted before hashing, empty optional field wrongly included, staging key in production, reused `OrderId`, stale enquiry, duplicate post, pending `N`/`W` treated as failed, void after the same-day window, refund over the remaining amount, token used with the wrong `CustId`.
3. Add fixes: server-side signing, strict field-order tables, constant-time compare, local pending order first, `PymtEnq` fallback, idempotent state transitions, per-environment config, unique `RefundId` and `RequestId`.
4. Re-read the relevant sections of `references/cardbizpay-api.md` or `references/cardbizpay-opa.md`.
5. State remaining uncertainty: whether the return post is browser or server-to-server, enquiry response format, Transaction List encoding, profile-level API enablement, subscribed methods, and pricing. These are not settled by public docs.
