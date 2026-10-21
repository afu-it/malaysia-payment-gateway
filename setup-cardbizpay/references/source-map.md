# CardBizPay Source Map

Use this map to pick the right official source before answering or coding. All pages checked 2026-10-21.

## Coverage

- Index used: WordPress REST listing of the `guide` post type, `https://cardbizpay.my/wp-json/wp/v2/guide?per_page=100&page=1..2` (`X-WP-Total: 134`), cross-checked with `https://cardbizpay.my/guide-sitemap.xml` and the section links on `https://cardbizpay.my/developer/open-payment-api/`.
- Enumerated: 134 guide posts + 3 developer pages (`/developer/`, `/developer/merchant-api/`, `/developer/open-payment-api/`).
- Read: all 134 guide posts and all 3 developer pages.
  - 78 are API documentation (about 39 Merchant API, 39 Open Payment API, including duplicated intro/URL/signature/code sections). All are reflected in `cardbizpay-api.md` and `cardbizpay-opa.md`.
  - 15 are merchant FAQ/guide pages: 12 with real content (registration, documents, settlement, refunds, dashboard, transactions, reports, failure reasons, login, bank account, email change; used in `account-setup.md`) and 3 empty (`how-to-configure-merchant-access`, `where-do-i-submit-my-documents`, `permanent-and-editable-details-required-for-registration`).
  - 41 are "Lorem ipsum" placeholders. Nothing to extract. Two API section posts are also empty headers (`payment-apis`, `2-merchant-integration-2`).
- Also read: `https://cardbizpay.my/pricing/`, `/contact/`, `/payment-channel/`, `/security/`, `/tokenization/`, `/faqs/`, `/payment-link/`, `/e-wallet-payment/`, and `https://cardbiz.com.my/cardbiz-payment-services/cardbizpay/`.
- Searched with no result: `https://cardbizpay.my/llms.txt` (404), WordPress.org plugin directory (`cardbizpay`, `cardbiz payment`: 0 results), GitHub repositories (`cardbizpay`: 0; `cardbiz`: unrelated projects). No OpenAPI, Postman collection, SDK, or PDF guide is published.
- Not covered by public docs: currency field, rate limits, IP allowlists, return-post transport and retries, Notify URL payload, Transaction List wire format, enquiry response format (except MdrEnq JSON), test cards/banks, bank and wallet codes, recurring scheduling API, payouts, settlement report API.

## Developer Hub

- `https://cardbizpay.my/developer/`
- `https://cardbizpay.my/developer/merchant-api/` (body currently empty, page modified 2025-12-29)
- `https://cardbizpay.my/developer/open-payment-api/`

Use for: which API families exist, and the Open Payment API section list.

## Merchant API: Integration Basics

- Introduction: `https://cardbizpay.my/guide/1-introduction/`
- Prerequisites (Return URL, Merchant No, Access Key, API enablement): `https://cardbizpay.my/guide/1-1-pre-requisite/`
- Transaction flow: `https://cardbizpay.my/guide/1-2-transaction-flow-diagram/`
- Page previews (15-minute session counter): `https://cardbizpay.my/guide/1-3-pages-preview/`, `/guide/1-4-sample-merchant-checkout-page/`, `/guide/1-5-cardbiz-payment-page-with-saved-credit-card/`, `/guide/1-6-cardbiz-payment-page-without-saved-credit-card/`, `/guide/1-7-sample-merchant-received-payment/`
- Base URLs: `https://cardbizpay.my/guide/2-merchant-integration/`, `/guide/2-1-url/`
- HTTP methods: `https://cardbizpay.my/guide/2-3-http-methods/`

Use for: environments, onboarding prerequisites, end-to-end flow.

## Signature

- Security control: `https://cardbizpay.my/guide/2-2-security-control/`
- HMAC-SHA512: `https://cardbizpay.my/guide/2-2-1-hash-signature/`
- Calculation and test vector: `https://cardbizpay.my/guide/2-2-2-signature-calculation/`
- OPA copies (example output does not reproduce): `/guide/2-2-security-control-2/`, `/guide/2-2-1-hash-signature-2/`, `/guide/2-2-2-signature-calculation-2/`

Use for: string-to-sign rules, key, base64 output, verification duty.

## Merchant API: Codes

- Code table intro: `https://cardbizpay.my/guide/2-4-code-table/`
- Response codes: `https://cardbizpay.my/guide/2-4-1-status-code/`
- Payment status codes: `https://cardbizpay.my/guide/2-4-2-payment-status-code/`

Use for: `RespCode` and `StatusCode` handling.

## Merchant API: Operations

- Payment overview: `https://cardbizpay.my/guide/3-payment-apis/`, `/guide/3-1-payment/`
- Pymt request / response: `/guide/3-1-1-request/`, `/guide/3-1-2-response/`
- Payment enquiry: `/guide/3-2-payment-enquiry/`, `/guide/3-3-2-request/` (titled 3.2.1), `/guide/3-2-2-reponse/`
- Void: `/guide/3-3-void-payment/`, `/guide/3-3-1-request/`, `/guide/3-3-2-reponse/`
- Void enquiry: `/guide/3-4-void-payment-enquiry/`, `/guide/3-4-1-request/`, `/guide/3-4-2-reponse/`
- Refund: `/guide/3-5-refund-payment/`, `/guide/3-5-1-request/`, `/guide/3-5-2-reponse/`
- Refund enquiry: `/guide/3-6-refund-status-enquiry/`, `/guide/3-6-1-request/`, `/guide/3-6-2-reponse/`
- MDR enquiry: `/guide/3-7-mdr-enquiry/`, `/guide/3-7-1-request/`, `/guide/3-7-2-reponse/`

(All under `https://cardbizpay.my`.) Use for: field tables, field order for signing, void/refund rules.

## Open Payment API

- Intro and prerequisites (Notify URL): `https://cardbizpay.my/guide/1-introduction-2/`, `/guide/1-1-pre-requisite-of-merchant-integration/`
- URL, methods, codes: `/guide/2-1-url-2/`, `/guide/2-3-http-methods-2/`, `/guide/2-4-code-table-2/`, `/guide/2-4-1-status-code-2/`, `/guide/2-4-2-payment-status-code-2/`
- AddCard: `/guide/3-1-add-card/`, `/guide/3-1-1-request-2/`, `/guide/3-1-1-response/`
- RemoveCard: `/guide/3-2-remove-card/`, `/guide/3-2-1-request/`, `/guide/3-2-1-response/`
- Sale: `/guide/3-3-sale/`, `/guide/3-3-1-request-2/`, `/guide/3-3-2-response/`
- PreAuth: `/guide/3-4-pre-authorization/`, `/guide/3-4-1-request-2/`, `/guide/3-4-2-response/`
- Capture: `/guide/3-5-capture-preauth/`, `/guide/3-5-1-request-2/`, `/guide/3-5-2-response/`
- PreAuthCancel: `/guide/3-6-pre-auth-cancellation/`, `/guide/3-6-1-request-2/`, `/guide/3-6-2-response/`
- Void: `/guide/3-7-transaction-enquiry/` (titled 3.7 Void), `/guide/3-7-1-request-2/`, `/guide/3-7-2-response/`
- Refund: `/guide/3-8-refund/`, `/guide/3-8-1-request/`, `/guide/3-8-2-response/`
- TrxnEnq: `/guide/3-9-transaction-enquiry/`, `/guide/3-9-1-request/`, `/guide/3-9-2-response/`

Use for: tokenization, pre-auth/capture, OPA-specific codes and Transaction List rules.

## Merchant Operations And FAQ

- Registration: `https://cardbizpay.my/guide/how-do-i-register/`, `/guide/required-documents-to-submit/`
- Payments and refunds FAQ: `/guide/payment-refunds/`
- Settlement FAQ: `/guide/settlement/`
- Failure reasons: `/guide/common-reason-failed/`
- Portal: `/guide/dashboard-overview/`, `/guide/transactions/`, `/guide/transactions-2/`, `/guide/reports/`
- Account: `/guide/why-cant-i-log-into-my-account/`, `/guide/how-to-update-the-merchants-bank-account-details/`, `/guide/can-i-change-the-registered-email-address/`

Use for: onboarding, settlement timing, refunds policy, portal features, support routes.

## Product, Pricing, Contact

- Product page: `https://cardbiz.com.my/cardbiz-payment-services/cardbizpay/`
- Site: `https://cardbizpay.my/`, `/payment-channel/`, `/payment-link/`, `/e-wallet-payment/`, `/tokenization/`, `/security/`, `/faqs/`
- Pricing (link only): `https://cardbizpay.my/pricing/`, `https://cardbiz.com.my/cardbiz-payment-services/cardbizpay-rates/`
- Contact: `https://cardbizpay.my/contact/`
- Merchant Portal: `https://cardbizpay.com/Merchant`

Use for: positioning, method list, PCI DSS claim, pricing links, support contacts.
