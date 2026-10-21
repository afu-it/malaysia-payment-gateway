---
name: setup-ipay88
description: Set up, build, debug, review, and explain iPay88 Malaysia payment integrations using iPay88's own OPSG Technical Specification v1.6.4.4 and the official iPay88 signature tester, covering the ePayment entry.asp redirect POST, HMAC-SHA512 request and response Signature, SignatureType, ResponseURL versus BackendURL handling, the RECEIVEOK acknowledgement, enquiry.asp re-query, the TxDetailsInquiry SOAP webservice requery, PaymentId values, Status codes, error messages, Appdeeplink for e-wallets, and Merchant Code/Merchant Key setup. Use when working with iPay88 (now operating under NTT DATA Payment Services Malaysia), FPX, cards, Boost, Touch 'n Go, ShopeePay, the SHA256 to HMAC-SHA512 migration, signature mismatch errors, backend post retries, or production readiness.
---

# iPay88 Malaysia Payment Gateway

Use this skill for iPay88 Malaysia (OPSG, "Online Payment Switching Gateway") integration work.

## Source

iPay88 does not publish its developer docs on an iPay88 domain. This skill is built from iPay88's own Technical Specification v1.6.4.4 (11 Sep 2024), obtained from a community mirror, plus official pages on `payment.ipay88.com.my` (signature tester, webservice WSDL) and iPay88's archived support FAQ. Where the mirror and an official page differ, the official page wins. Merchants receive the current spec from iPay88 Support; confirm against it.

Read `references/ipay88-api.md` before giving factual API guidance or writing integration code. It has the endpoint catalogue, every request/response field, the HMAC-SHA512 string-to-sign for request and response, the Xfield1 open question, backend post and `RECEIVEOK` rules, both re-query forms, status values and error messages.

Read `references/account-setup.md` for merchant onboarding, the Merchant Checklist, Request URL registration, IP whitelisting, test amounts, go-live notice, report login, env var names and payment method activation.

Read `references/source-map.md` to see which source backs which fact and what is not documented.

## Core Workflow

1. Confirm iPay88 has issued the Merchant Code and Merchant Key, registered the Request URL, and whitelisted the server IPs for ResponseURL and BackendURL.
2. Create a local pending order with a unique `RefNo` (max 30 chars), the expected amount and `MYR`.
3. Build the HMAC-SHA512 request `Signature` server-side, keyed with the Merchant Key.
4. Render an auto-submitting HTML form that POSTs to `https://payment.ipay88.com.my/epayment/entry.asp` with every request field present (optional fields may be empty but must exist).
5. On ResponseURL (browser POST): verify Signature, amount, currency and `RefNo`; show the result page. Treat it as UX plus a settlement path, never as trusted input without verification.
6. On BackendURL (server POST, success only): verify the same way, settle idempotently, then print exactly `RECEIVEOK` with no HTML.
7. Before fulfilment, or when no post arrives, re-query `enquiry.asp` (reply `00` = paid) or the SOAP webservice (Version 4, `Status` `1` = paid).
8. Settle once, whichever of ResponseURL, BackendURL or re-query arrives first.

## Core Facts

- Payment page: `POST https://payment.ipay88.com.my/epayment/entry.asp` (HTML form post, not a JSON API).
- Re-query: `https://payment.ipay88.com.my/epayment/enquiry.asp` with `MerchantCode`, `RefNo`, `Amount`; plain-text reply.
- Webservice requery: `https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx`, operation `TxDetailsInquiryCardInfo`, `Version` = `4`. SOAPAction `"https://www.mobile88.com/epayment/webservice/TxDetailsInquiryCardInfo"` (official WSDL); a plain form POST to `.../TxDetailsInquiry.asmx/TxDetailsInquiryCardInfo` also works.
- No separate sandbox host is documented. Testing runs on the production host with a real merchant account, from the registered Request URL, using MYR 1.00.
- `SignatureType` = `HMACSHA512`. iPay88 moved from SHA256 to HMAC-SHA512 in spec v1.6.4.4; SHA256 integrations are deprecated.
- Request signature (official tester): `HMAC_SHA512(key=MerchantKey, MerchantKey + MerchantCode + RefNo + Amount + Currency)`, Amount with "." and "," removed, lowercase hex, 128 chars.
- The spec also appends `Xfield1` to the request string. With an empty Xfield1 both forms are identical. See the open question in `references/ipay88-api.md`.
- Response signature: `HMAC_SHA512(key=MerchantKey, MerchantKey + MerchantCode + PaymentId + RefNo + Amount + Currency + Status)`.
- Response `Status`: `1` success, `0` fail. Webservice requery adds `6` pending and `20` authorised; an unknown RefNo returns empty `Status` with `Errdesc` `Record not found`.
- BackendURL receives success only; iPay88 posts to ResponseURL and BackendURL at the same time; retries backend up to 3 times without `RECEIVEOK`.
- `PaymentId` is optional; blank lets the customer choose on the iPay88 page. Numeric values are in iPay88's separate "Appendix I.pdf" (MYR) and "Appendix II.pdf" (multi-currency), issued by iPay88 Support.
- Refunds, recurring and tokenisation: sold as products, but no public API. The WSDL shows refund, capture and token fields on requery results only.
- Support: `support@ipay88.com.my`. Merchant report: `https://payment.ipay88.com.my/ePayment/report`.

## Critical Rules

- Keep the Merchant Key server-side only; never put it in browser code, even though the signature is posted from the browser.
- Compute the signature over the exact `Amount` string you post (for example `1,278.99` becomes `127899`).
- Verify the response Signature with a constant-time compare; also compare `Amount`, `Currency`, `RefNo` and `MerchantCode` with the local order.
- Never fulfil from ResponseURL alone without signature and amount checks; the browser can be tampered with or closed.
- The backend page must not depend on session or login state, and must output only `RECEIVEOK` after a successful update.
- Make settlement idempotent across ResponseURL, BackendURL and re-query; both posts arrive at once.
- Do not reuse a `RefNo` that was already paid ("Duplicate reference number").
- No backend post on failure: handle failed and abandoned payments with ResponseURL and re-query.
- For Appdeeplink (mobile app to e-wallet), no response data returns to the app; use the webservice requery and BackendURL.
- Use `HTTP_REFERER` from `https://payment.ipay88.com.my` only as an extra check, never as proof.

## Completion Criteria

An iPay88 integration is done only when:

1. The request form posts every field, with `SignatureType=HMACSHA512` and a signature that matches the official tester page.
2. ResponseURL and BackendURL both verify the response signature, amount, currency and `RefNo`.
3. BackendURL prints only `RECEIVEOK` after a durable update and has no session code.
4. Settlement is idempotent across both posts and re-query.
5. A re-query (`enquiry.asp` or webservice) runs before fulfilment or as fallback for missing posts.
6. Tests cover bad signature, wrong amount, duplicate posts, `Status=0`, re-query `Record not found`/`Incorrect amount`, and a paid MYR 1.00 test.
7. Deployment docs list `IPAY88_MERCHANT_CODE`, `IPAY88_MERCHANT_KEY`, ResponseURL, BackendURL, and the IPs whitelisted with iPay88.

## Loophole Review Loop

When asked whether an iPay88 strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: checkout page, browser form post, iPay88 payment page, bank/e-wallet, ResponseURL, BackendURL, re-query call, database, fulfilment job.
2. Check bypasses: forged ResponseURL post, replayed success post, amount edited in the form, signature built over a different amount format, Xfield1 mismatch, SHA256 left in place, duplicate RefNo, backend post and response post racing, missing `RECEIVEOK` causing retries, session-dependent backend page, failed payment never posted to backend, e-wallet deeplink with no response data, pending (`6`) treated as paid.
3. Add fixes: server-side signing, signature and amount checks on both URLs, constant-time compare, unique RefNo, row lock or unique constraint on settlement, re-query before fulfilment, plain `RECEIVEOK`, sessionless backend route.
4. Re-read `references/ipay88-api.md` sections relevant to any gap.
5. State remaining uncertainty: the spec came from a mirror, the Xfield1 rule, PaymentId values, refund and recurring products, and anything only iPay88 Support can confirm.
