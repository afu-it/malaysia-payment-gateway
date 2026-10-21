---
name: setup-asiapay
description: Set up, build, debug, review, and explain AsiaPay (PayDollar) Malaysia payment integrations using official AsiaPay developer docs, Client Post Through Browser (payForm.jsp hosted page), Direct Client Side and Server Side Direct Connection (payComp.jsp), SHA-1/SHA-256 secureHash request signing, datafeed (Return Value Link) verification with the src|prc|successcode|Ref|PayRef|Cur|Amt|payerAuth hash, the "OK" acknowledgement, Merchant API orderApi.jsp (Query, Capture, Void, RequestRefund, Cancel, QueryRequestLog), onlineApi.jsp OnlineReversal refunds, GenTxnXML settlement report, MemberPay tokens, PaySDK, 3DS 2.0, test vs live URLs, and merchant admin setup. Use when working with AsiaPay, PayDollar, AsiaPay Malaysia, MYR currCode 458, FPX, DuitNow, Touch 'n Go, GrabPay, cards, e-wallets, datafeed settlement, refunds, or production readiness.
---

# AsiaPay (PayDollar) Payment Gateway

Use this skill for AsiaPay payment gateway integration work. AsiaPay Malaysia sells the PayDollar gateway; the API, hosts and field names are the PayDollar ones (`paydollar.com`). SiamPay and PesoPay are the Thai and Philippine brands of the same platform and are out of scope.

## Source

Read `references/asiapay-api.md` before giving factual API guidance or writing integration code. It holds the endpoint catalogue, test and live URLs, every payment request field, the exact secureHash string orders, datafeed fields, Merchant API inputs and outputs, status values, response codes, test cards and gotchas.

Read `references/account-setup.md` when the user asks how to sign up, where the merchant ID, Secure Hash Secret or API login live, how to enable datafeed or secure hash, sandbox vs live accounts, payment method activation in Malaysia, or pricing.

Read `references/source-map.md` to pick the right official page, or to see what the docs cover and leave out.

## Core Workflow

1. Identify environment: test (`test.paydollar.com/b2cDemo`) or production (`www.paydollar.com/b2c2`).
2. Pick the connection method: Client Post Through Browser (hosted page, default for web), Direct Client Side Connection (merchant card form, needs PCI scope), Server Side Direct Connection (server post, wallets/app-to-app), or PaySDK (mobile).
3. Confirm admin setup: Secure Hash enabled by AsiaPay, Secure Hash Secret copied, datafeed URL ("Return Value Link") enabled, Merchant API login issued, payment methods activated.
4. Create a local pending order with a unique `orderRef` before posting to PayDollar.
5. Build the form server-side: `merchantId`, `orderRef`, `currCode=458`, `amount`, `payType`, `payMethod`, `lang`, the three URLs, and `secureHash = SHA(merchantId|orderRef|currCode|amount|payType|secret)`.
6. Auto-submit the form to `payForm.jsp` from the browser. Treat `successUrl`, `failUrl` and `cancelUrl` as display only.
7. Receive the datafeed POST, print `OK` first, then verify `secureHash` over `src|prc|successcode|Ref|PayRef|Cur|Amt|payerAuth|secret`.
8. Settle idempotently only when hash, `MerchantId`, `Ref`, `Amt`, `Cur` and `successcode=0` all match.
9. If the datafeed is missing or `successcode=2` (pending), requery with Merchant API `actionType=Query`.
10. Run refunds and voids server-side through `onlineApi.jsp` (`OnlineReversal`) or `orderApi.jsp` (`Void`, `RequestRefund`).

## Core Facts

- Test hosted page: `https://test.paydollar.com/b2cDemo/eng/payment/payForm.jsp`; live: `https://www.paydollar.com/b2c2/eng/payment/payForm.jsp`.
- Server Side Direct: `.../eng/directPay/payComp.jsp`; Direct Client Side: `.../eng/dPayment/payComp.jsp` (same two bases).
- Merchant API: `.../eng/merchant/api/orderApi.jsp`; online void/refund: `.../eng/merchant/api/onlineApi.jsp`; settlement report: `.../GenTxnXML`.
- All requests are `application/x-www-form-urlencoded` POSTs, UTF-8, and field names are case sensitive.
- MYR currency code is `"458"`. One merchant ID accepts one currency only.
- `amount` is in ringgit with up to 2 decimals (`Number(12,2)`), not sen.
- Request secureHash: `merchantId|orderRef|currCode|amount|payType|secureHashSecret`, SHA-1 by default or SHA-256 if set on the account.
- Datafeed secureHash: `src|prc|successcode|Ref|PayRef|Cur|Amt|payerAuth|secureHashSecret`, same algorithm.
- Once Secure Hash is enabled, requests without a valid hash are dropped. The secret is rotated every 2 years.
- Datafeed `successcode`: `0` success, `1` failure, `2` pending, other values error.
- The datafeed page must print `OK`; without it PayDollar treats the datafeed as lost. Only ports 80 and 443 are supported.
- Merchant API auth is `merchantId` + `loginId` + `password` in the POST body, issued by AsiaPay on request.
- Hold payments (`payType=H`) must be captured within 14 days or the hold is released.
- Test merchant IDs are 6 digits; production merchant IDs are 4 or 8 digits.
- Datafeed source IPs (PDF guide v3.67): test `203.105.61.183` to `203.105.61.218`; production `103.149.149.1` to `103.149.149.62`.

## Critical Rules

- Keep the Secure Hash Secret, API `loginId` and `password` server-side only. Compute `secureHash` on the server and render it into the form.
- Hash the exact `amount` string you post (`10` and `10.00` give different hashes).
- Never mark paid from `successUrl`, `failUrl`, `cancelUrl`, the `Ref` query value, or PaySDK client results alone. Use the datafeed or Merchant API Query.
- Verify the datafeed hash against the raw POST values before any type conversion. Accept the datafeed if any comma-separated value in `secureHash` matches (official plugin behaviour during secret rotation).
- Use constant-time comparison for hashes.
- Print `OK` to acknowledge before slow work; never let a processing error suppress the `OK` and trigger retries you cannot dedupe.
- Expect several datafeeds for one `orderRef` (customer retries after Back). Once paid, ignore later failures for the same `orderRef`.
- Treat `successcode=2` and statuses `Pending`, `Pending_3D`, `Pending_feedback` as not final; requery later.
- Do not mix test and production merchant IDs, URLs, secrets or API logins.
- Never store card number, expiry or CVV. Direct Client Side and Server Side card posts put the merchant in PCI scope.

## Completion Criteria

An AsiaPay integration is done only when:

1. The payment form is built server-side with a correct secureHash and posts to the right environment URL.
2. Redirect pages only show status; they do not fulfil orders.
3. The datafeed URL is enabled in merchant admin for the right environment and reachable on 80/443 with a trusted TLS chain.
4. The datafeed handler prints `OK`, verifies the hash, and checks `MerchantId`, `Ref`, `Amt`, `Cur` and `successcode` before settling.
5. Settlement is idempotent across duplicate and retried datafeeds and Query fallbacks.
6. Merchant API Query is used for missing datafeeds and pending results; refund/void code handles `resultCode=-1`.
7. Tests cover bad hash, wrong amount/currency/ref, duplicate datafeed, success-then-failure for one `orderRef`, pending then accepted, and refund failure.
8. Deployment docs list `ASIAPAY_MERCHANT_ID`, `ASIAPAY_SECURE_HASH_SECRET`, `ASIAPAY_HASH_ALGO`, `ASIAPAY_API_LOGIN_ID`, `ASIAPAY_API_PASSWORD`, the base URL and the datafeed URL per environment.

## Loophole Review Loop

When asked whether an AsiaPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser form post, payment page, redirect URLs, datafeed receiver, Merchant API caller, database, fulfilment job, refund job, merchant admin settings.
2. Check bypasses: forged datafeed, hash not enabled on the account, wrong algorithm (SHA-1 vs SHA-256), amount string mismatch, parsed instead of raw values, rotated secret, wrong merchant ID or currency, replayed success redirect, duplicate `orderRef`, success followed by failure datafeed, pending marked failed too early, missing `OK`, test/live mix, refund past the window, capture past 14 days.
3. Add fixes: server-side hash, raw-value verification, constant-time compare, strict field checks, unique `orderRef`, event log with idempotency key `PayRef`, Query fallback, IP range check as defence in depth, separate env config.
4. Re-read the relevant section of `references/asiapay-api.md` for any gap.
5. State remaining uncertainty that depends on the merchant account: hash algorithm, enabled payment methods and their `payMethod` codes, refund window, datafeed retry setting, and production PaySDK hosts.
