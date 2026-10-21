# AffanPay Source Map

Use this map to pick the right source before answering or coding. All checked 2026-10-21.

## API Documentation

- Live API docs: `https://app.affanpay.my/api`
  - `#introduction`: JSON responses, disclaimer, API endpoint base.
  - `#token`: Bearer token, Settings > Developer > API tokens, shown once.
  - `#createbill`: `POST /api/v1/bill`, request field table, PHP example, response.
  - `#requery`: `GET /api/v1/requery`, `bill_id` / `payment_ref`, response with `payments[]` and status codes.
  - `#callback`: when callbacks fire, double send, headers, body fields, HMAC verification, secret key location, 200 vs 4xx handling.
- Sandbox API docs: `https://sandbox.affanpay.my/api`

Use for: every endpoint, field, status code, and the signature algorithm.

## Account And Onboarding

- Live registration: `https://app.affanpay.my/register`
- Sandbox registration: `https://sandbox.affanpay.my/register`
- Live sign-in: `https://app.affanpay.my/sign-in`
- Sandbox sign-in: `https://sandbox.affanpay.my/sign-in`

Use for: registration steps, required documents, sandbox vs live hosts.

## Product, Pricing, And Support

- Homepage: `https://affanpay.my/`
- Contact and FAQ: `https://affanpay.my/contact.html`
- About: `https://affanpay.my/about.html`
- Partnership: `https://affanpay.my/partnership.html`
- Bayar.io: `https://bayar.io`

Use for: channels offered, channel approval notes, pricing link, T+1 settlement, requirements FAQ, support contacts and hours.

## Coverage

- Index used: none published. `/llms.txt`, `/llms-full.txt`, and `/sitemap.xml` on `app.affanpay.my` return 404; `robots.txt` has no sitemap. The API docs are one page whose in-page navigation lists 5 sections (Introduction, Token, Create Bill, Requery, Callback).
- Enumerated: 1 API docs page (5 sections) plus its sandbox copy. Read: all 5 sections on both; the sandbox copy differs only in host name.
- Supporting pages read: homepage, contact/FAQ, live registration page. About, partnership, and sign-in pages checked for reachability only.
- Endpoints documented: 3 (Create Bill, Requery, outbound Callback). All are in `affanpay-api.md`.

## Checked But Not Found

- Official SDKs or GitHub organisation: none found. GitHub search returns `Imncem/affanpay-lite` and `maui2023/affanpay-laravel`, owned by individuals with no link from AffanPay, so they are excluded as unofficial.
- WordPress.org plugin: none (plugin search for "affanpay" returns 0 results).
- Homepage links "Integration Guides" and "FAQ" under Resources do not point to separate docs pages.
- Refunds, recurring, payouts, rate limits, IP allowlists, sandbox test data, error codes: not documented.
