# OxPay Source Map

Use this map to pick the right official source before answering or coding. All checked 2026-10-21.

## Coverage

- Index used: `https://docs.oxpayfinancial.com/sitemap.xml` (9 URLs) plus the Docusaurus sidebar (same 9 pages) and the Redoc config in the site bundle, which points at the OpenAPI file.
- `llms.txt` and `llms-full.txt`: not published (HTTP 404).
- Enumerated: 9 docs pages + 1 OpenAPI file (9 paths/operations, 5 webhooks, about 150 schemas) + 6 portal guide PDFs linked from Getting Started. Read: all 16.
- The `/api-reference` HTML page is an empty shell rendered client-side from the OpenAPI file; the OpenAPI file was read in full instead.
- Also read for account context: `oxpay.com.my` home, payment, solutions, support, contact, about pages.
- Not found: official GitHub/npm/packagist SDK, WordPress.org plugin (the Malaysia support page says plugins are supplied on request). Public OxPay repos on GitHub are third-party and were not used.
- Left out on purpose: OxPay Lite app usage, ffastpay terminal guides and the "Clear Cache" browser PDF, since they do not affect API integration. Payouts are included but their examples are Singapore-only.

## Product And Market Overview

- Docs introduction: `https://docs.oxpayfinancial.com/`
- Malaysia site: `https://oxpay.com.my/`, `https://oxpay.com.my/payment/`, `https://oxpay.com.my/solutions/`, `https://oxpay.com.my/about/`

Use for: positioning, entity name, marketed methods, the fact that the API is REST/JSON with bearer token plus signature.

## Getting Started And Onboarding

- Getting Started: `https://docs.oxpayfinancial.com/getting-started`
- Support (merchant application form, API specs, plugins note): `https://oxpay.com.my/support/`
- Contact: `https://oxpay.com.my/contact/`

Use for: prerequisites (MID, TID, portal, Basecamp), integration options (hosted page vs merchant hosted), user flows, end-to-end onboarding, "no UAT environment" note, go-live approval.

## API Reference

- Redoc page: `https://docs.oxpayfinancial.com/api-reference`
- OpenAPI 3.0.0 file (v2.0.2): `https://docs.oxpayfinancial.com/redocusaurus/oxpay-payments-api-reference.yaml`

Use for: servers (live and UAT), required headers, every endpoint, request/response schemas, brand and state enums, examples, webhook payloads and Signature header on webhooks, payouts.

## Security, Credentials And Signature

- Security: `https://docs.oxpayfinancial.com/security`
- Credentials: `https://docs.oxpayfinancial.com/credentials`
- Integrity: `https://docs.oxpayfinancial.com/integrity`

Use for: HTTPS, API key auth, rate limiting statement, PCI DSS, where to create keys, HMAC-SHA256 signature over compact JSON with Python/Java/PHP samples.

## Responses And Errors

- HTTP status codes: `https://docs.oxpayfinancial.com/responses/http-status-codes`
- Operation status: `https://docs.oxpayfinancial.com/responses/operation-status`
- Reason codes: `https://docs.oxpayfinancial.com/responses/reason-codes`

Use for: HTTP codes, `headers.status_code` 0/1/2/3, per transaction type next actions.

## Merchant Portal Guides (OxPay PDFs on Google Drive, linked from Getting Started)

- How to get Signature Key and API Key: `https://drive.google.com/open?id=1LyQvS_HcMn6c1ehpsEDdMcWQoKfzjHrw`
- Merchant Portal User Guide: `https://drive.google.com/open?id=1nhayv5wHQT1s3Ukg8_ezadoCxrmOavM_`
- [MFA] How to access Merchant Portal: `https://drive.google.com/open?id=14ihP5s61mrRdyqqhDUN-hkibO8fU7bAF`
- How to create new account in OxPay: `https://drive.google.com/open?id=1YE94oOnVOICmf3BrJWZxeuRNsLeccoc3`
- OxPay Lite User Guide: `https://drive.google.com/open?id=1EfsnHmX-bPgm3c_67ihuXNxLcdKGXYvC`
- Clear Cache: `https://drive.google.com/open?id=1hNKs9W0idZ5aq_TB_QoypZKWGSIApwLU`

Use for: portal and login URLs, self sign-up, MFA, API key creation steps, portal sections, settlement/daily/monthly reports, compliance-before-disbursement rule, support hotline and Freshdesk.

## Operations

- Merchant portal: `https://merchant.oxpayfinancial.com`
- Help desk: `https://oxpay.freshdesk.com/support/home`

Use for: transaction lookup, reports, key management, support tickets. No public status page was found.
