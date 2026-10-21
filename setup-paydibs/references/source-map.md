# Paydibs Source Map

Use this map to pick the right source before answering or coding. The docs site `https://v3api-docs.paydibs.com` is a JavaScript app; there is no llms.txt, sitemap or OpenAPI file (all return the app shell, checked 2026-10-21). Open pages in a browser.

## Overview And Versioning

- Introduction: `https://v3api-docs.paydibs.com/docs/introduction`
- Revision history: `https://v3api-docs.paydibs.com/docs/revision`
- Billing cycle rules (recurring): `https://v3api-docs.paydibs.com/docs/recurring/billing-cycle-rules`

Use for: prerequisites (test account, Merchant ID, Merchant Password, payment URL), data types, allowed special characters, E-KYC character rules, production URL by email, and the docs version (4.0, 24 Sep 2026).

## Hosted Checkout (PAY)

- PAY request: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/pay-request`
- PAY response: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/pay-response`

Use for: `/PPG/trigger`, mandatory/optional/conditional fields, amount format, `&` to `;` rule, E-KYC, Cash on Delivery fields, `PaySessId`, MerchantRURL form post, callback content type and JSON body, 5 retries, response fields.

## Status Query

- QUERY request: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/query-request`
- QUERY response: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/query-response`

Use for: `TxnType=QUERY`, request fields, `PAYExists`, JSON samples for found, not found and hash mismatch, TLS 1.2 note.

## Refund

- REFUND request: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/refund-request`
- REFUND response: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/refund-response`

Use for: `/PPG/refund`, `RefundReason`, `RefundResult`, `RefundID`, failure samples.

## Sign (Hashing)

- Merchant request sign: `https://v3api-docs.paydibs.com/docs/sign-hashing/merchant-request-sign`
- Merchant response sign: `https://v3api-docs.paydibs.com/docs/sign-hashing/merchant-response-sign`
- E-mandate sign: `https://v3api-docs.paydibs.com/docs/sign-hashing/e-mandate-sign`
- Direct debit sign: `https://v3api-docs.paydibs.com/docs/sign-hashing/direct-debit-sign`

Use for: SHA512 string-to-sign order for PAY, QUERY, REFUND, MIT, hosted session, REVOKE, BINDOPT, in-store QR and every response, with worked examples.

## Status Codes And Enums

- Appendices: `https://v3api-docs.paydibs.com/docs/appendices`

Use for: TxnType list, PTxnStatus, PAYExists, currency codes, method codes, token type, payment option code, E-KYC result, country/address type, recurring enums (Mode, Src, Bhv, BillCycTyp, Freq, weekday), ID type, debit frequency.

## Hosted Payment Session (Cards)

- Initiate session: `https://v3api-docs.paydibs.com/docs/hosted-session/initiate`
- SDK integration guide: `https://v3api-docs.paydibs.com/docs/hosted-session/integration-guide`
- SDK API reference: `https://v3api-docs.paydibs.com/docs/hosted-session/api-reference`
- SDK file: `https://v3api-docs.paydibs.com/HOSTED_SESSION_SDK/sdk/v1/sdk.js`

Use for: `/PPG/create-pay-session`, session fields and response, SDK configure/mount/updateSession, events, theme, error codes, and the `stg`/`prod` iframe hosts in sdk.js.

## Recurring

- Initiation (CIT): `https://v3api-docs.paydibs.com/docs/recurring/initiation`
- Subsequent payments (MIT): `https://v3api-docs.paydibs.com/docs/recurring/subsequent-payments`
- Billing cycle rules: `https://v3api-docs.paydibs.com/docs/recurring/billing-cycle-rules`

Use for: recurring PAY fields, `RcurId`, MIT request/response, `MITSuccess`, Paydibs-scheduled charges.

## Tokens And Linked Accounts

- Bind option request/response: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/bind-option-request`, `.../bind-option-response`
- Revoke token request/response: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/revoke-token-request`, `.../revoke-token-response`

Use for: `BINDOPT`, `REVOKE`, card tokens, TNG AutoDebit.

## E-Mandate And Direct Debit

- `https://v3api-docs.paydibs.com/docs/e-mandate-feature/enrolment`
- `https://v3api-docs.paydibs.com/docs/e-mandate-feature/maintenance`
- `https://v3api-docs.paydibs.com/docs/e-mandate-feature/termination`
- `https://v3api-docs.paydibs.com/docs/e-mandate-feature/create-direct-debit-instruction`
- `https://v3api-docs.paydibs.com/docs/e-mandate-feature/create-direct-debit-instruction-transaction`
- `https://v3api-docs.paydibs.com/docs/e-mandate-feature/cancel-direct-debit-instruction`

Use for: `/PPG/trigger-emandate` (`EMDT_CRT`, `EMDT_MTN`, `EMDT_TMN`), `/PPG/create-ddi`, `/PPG/create-ddi-txn`, `/PPG/cancel-ddi`, their field tables, Sign formulas and DDI error codes. Distilled in `paydibs-emandate.md`.

## In-Store

- In-store pay request/response: `https://v3api-docs.paydibs.com/docs/merchant-api-reference/in-store-pay-request`, `.../in-store-pay-response`

Use for: `/PPG/offline/trigger-qr` DuitNow QR.

## Company, Onboarding, Legal

- Home: `https://paydibs.com/`
- Online payment: `https://paydibs.com/online-payment/`
- Payment Gateway Stack: `https://paydibs.com/payment-gateway-stack/`
- Contact / sales enquiry: `https://paydibs.com/contact-us/`
- Service agreement: `https://paydibs.com/paydibs-service-agreement/`

Use for: product positioning, onboarding path, regulatory status, PCI-DSS claim, settlement and chargeback terms, pricing contact.

## Which Reference File Holds What

- `paydibs-api.md`: endpoint catalogue, PAY, PAY response, QUERY, REFUND, all Sign formulas, environments.
- `paydibs-codes.md`: Appendices, Introduction data types and special characters, revision history, DDI and SDK error codes.
- `paydibs-recurring.md`: recurring initiation, subsequent payments, billing cycle rules.
- `paydibs-hosted-session.md`: hosted session initiate, integration guide, API reference, sdk.js.
- `paydibs-tokens-instore.md`: bind option, revoke token, in-store pay request/response.
- `paydibs-emandate.md`: six e-mandate/direct debit pages plus e-mandate and direct debit Sign pages.

## Coverage

Checked 2026-10-21. Index used: the docs app router and lazy-loaded page chunks in `https://v3api-docs.paydibs.com/assets/index-533f0cc5.js` (the site has no llms.txt, llms-full.txt, sitemap.xml or OpenAPI file; those URLs return the app shell).

- Routed docs pages enumerated: 31. Read: 31 (100%).
  - Introduction, Revision History (2)
  - Merchant API Reference: pay request/response, query request/response, refund request/response, bind option request/response, revoke token request/response, in-store pay request/response (12)
  - Recurring: initiation, subsequent payments, billing cycle rules (3)
  - E-Mandate Feature: enrolment, maintenance, termination, create DDI, create DDI transaction, cancel DDI (6)
  - Hosted Payment Session: initiate, integration guide, API reference (3)
  - Sign (Hashing): merchant request, merchant response, e-mandate, direct debit (4)
  - Appendices (1)
- Official SDK file read: `HOSTED_SESSION_SDK/sdk/v1/sdk.js` (1 of 1).
- Marketing/legal pages read: home, online payment, payment gateway stack, contact, service agreement (5).
- Excluded: 7 route names present in the bundle but not mounted in the navigation (terminal callback request/response, void request/response, settlement request/response, terminal request/response Sign). They have no page content in the published build, so nothing could be read. Re-check later.
- Official plugins/SDK repos: none found on Paydibs domains (the MPG listing names WooCommerce, but no plugin source is published by Paydibs in the docs).
- Sign examples on the four Sign pages: 26 published (10 merchant request, 6 merchant response, 4 e-mandate, 6 direct debit). Reproduced 25 of 26 with SHA512 over UTF-8 (the `${Paydibs}` placeholder in the merchant examples is the literal MerchantID `Paydibs`). Not reproducible: the PAY request example (its shown source string does not hash to the shown value). Also reproduced the FLX create-DDI request sample on the DDI page. The MIT request/response samples on the recurring pages use an unshown password and could not be checked.

## Corrections In This Revision (2026-10-21)

- `paydibs-api.md` Data Types: earlier text said `EkycBuyerId` does not support `/`, `&` or `'`. The Introduction note names only slash and ampersand; fixed.
- E-Mandate and Direct Debit were listed as "not distilled"; now fully distilled in `paydibs-emandate.md`.
- Added the endpoint catalogue, full field tables for PAY response, QUERY, REFUND, MIT, BINDOPT, REVOKE, in-store QR, hosted session and e-mandate/DDI, and the complete enum set.

## Documentation Inconsistencies Noted

- MIT success sample returns `Method: "MC"`, not a listed method code.
- CIT table heads the Paydibs-managed schedule fields with `SRC = MER`, while the process text says Paydibs-managed is `Src = PG`.
- REVOKE response: Sign formula is published but the field table and samples have no `Sign`.
- BINDOPT invalid-data sample has a duplicated `BindOptDesc` key and no `BindOptResult`/`Sign`.
- Create DDI sample response echoes a `PymRefNum` that differs from its request.
- Billing cycle rules list "every 30 minutes" as an interval example though no minute frequency exists.

