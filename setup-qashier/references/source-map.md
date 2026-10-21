# Qashier Source Map

Use this map to pick the right official source before answering or coding. All checked 2026-10-21.

## Coverage

- Index used: `https://docs.qashier.com/sitemap.xml` (Docusaurus site). It lists 7 pages; all 7 were read.
- The two API reference pages are rendered by Redocusaurus from raw OpenAPI files, which were read in full: `payment-online-checkout-api.yaml` (1201 lines) and `payment-instore-api.yaml` (671 lines).
- `llms.txt`, `openapi.json`, `swagger.json` are not published (they return the site's HTML shell). The GitHub link in the docs footer (`github.com/qashierpos/qashier-docs`) returns "Repository not found". No official SDK, npm, packagist, or WordPress.org plug-in was found.
- Left out: nothing from the developer docs. Global-only content: Singapore, Philippines, and Thailand test cards are kept but labelled; currency examples `SGD`/`PHP` noted only.

## Docs Home And Intro

- `https://docs.qashier.com/`
- `https://docs.qashier.com/docs/intro`

Use for: what the docs cover, support route (account manager).

## Online Checkout (primary)

- Guide: `https://docs.qashier.com/docs/guides/checkout-session`
- API reference: `https://docs.qashier.com/api/payment-online/`
- Raw spec: `https://docs.qashier.com/redocusaurus/payment-online-checkout-api.yaml`

Use for: base URLs, Basic Auth, idempotency, Request-Id, Checkout Session create/list/retrieve/expire, session fields and statuses, refunds, webhook events and `X-Qashier-Signature` algorithm, errors, rate limit (100 rps, guide only), payment method behaviour, test cards.

## In-Store Terminal Partner API

- API reference: `https://docs.qashier.com/api/payment-instore/`
- Raw spec: `https://docs.qashier.com/redocusaurus/payment-instore-api.yaml`

Use for: partner headers, HMAC-SHA256 Base64 request signing, IP whitelist, timestamp window, payment method list, initiate, cancel, void, status, notification callback, error messages.

## Template Pages (not real API)

- `https://docs.qashier.com/docs/api/payments-example`
- `https://docs.qashier.com/docs/api/openapi-reference-example`

Use for: nothing. They demonstrate doc embedding with a placeholder `POST /v1/payments` Bearer example. Do not integrate against them.

## Product, Plug-ins, Pay-by-Link, Pricing

- QashierPay Online: `https://qashier.com/my/qashierpay-online`
- Pay-by-Link: `https://qashier.com/my/pay-by-link/`
- API Integration: `https://qashier.com/my/api-integration`
- E-commerce inventory sync: `https://qashier.com/my/ecommerce-integration`
- MDR rates: `https://qashier.com/my/mdr-rates`
- Merchant back office: `https://hq.qashier.com`

Use for: integration options (plug-ins, custom API, Pay-by-Link), accepted method families, BSN acquiring partner, next-day settlement claims, pricing links.
