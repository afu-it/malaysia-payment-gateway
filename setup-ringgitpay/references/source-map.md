# RinggitPay Source Map

Use this map to pick the right source before answering or coding. All sources checked 2026-10-21.

## Source Trust Caveat

RinggitPay publishes no developer portal, API docs page, OpenAPI file, or SDK on its own domains. The sources are:

1. Official plugin on WordPress.org, published by the `ringgitpay1` account with author "RinggitPay" and Author URI `https://ringgitpay.biz`. Treated as official.
2. Two RinggitPay-authored PDFs (RinggitPay branding, document owner "RinggitPay Sdn Bhd", RinggitPay staff revision history). They were found only on a third-party GitHub repository, not on a RinggitPay domain. Their endpoints, field names and checksum formulas match the official plugin and the live endpoints exactly, so they are used, but treat them as "official document, unofficial copy" and ask RinggitPay for the current version (the plugin readme calls it the "RinggitPay Integration Document").

## Hosted Payment Page API

- Integration spec v1.13: `https://github.com/farisfang/ringgitpay_documentation/blob/main/RinggitPay_Integration_and_Testing_V1.13.pdf`
- Official plugin page: `https://wordpress.org/plugins/ringgitpay/`
- Official plugin source (1.0.4): `https://plugins.svn.wordpress.org/ringgitpay/tags/1.0.4/class-wc-ringgitpay-gateway.php`
- Plugin readme: `https://plugins.svn.wordpress.org/ringgitpay/tags/1.0.4/readme.txt`
- Plugin metadata API: `https://api.wordpress.org/plugins/info/1.0/ringgitpay.json`

Use for: UAT/production URLs, payment request fields, response fields, request/response/enquiry checksum formulas, checksum case, transaction enquiry, status codes, payer verification `rp_xtraInfo`, APIURL/RedirectURL/ReturnURL semantics, UAT FPX banks and simulator login, test cards, B2B pending flow, go-live checklist, plugin settings and status mapping, MYR-only.

## Direct Debit (e-Mandate) API

- Direct Debit guide v1.8: `https://github.com/farisfang/ringgitpay_documentation/blob/main/RP_DD_API_Integration_Guide_v1.8.pdf`
- Identity server discovery (live, official domain): `https://identity.ringgitpay.biz/.well-known/openid-configuration`

Use for: DD API base URLs, OAuth2 token request (password grant, scope `bills2u_api`), version rules, DDA create/view/update/manage, debit settings, DDI create/view/approve/cancel, response envelope, HTTP codes.

## Live Endpoint Checks

- `https://ringgitpay.co/payment`, `https://ringgitpay.com/payment`: 405 on GET, `Allow: POST`.
- `https://ringgitpay.co/transactionenquiry`, `https://ringgitpay.com/transactionenquiry`: 405 on GET, `Allow: POST`.

Use for: confirming the spec URLs are live and POST-only.

## Website (Product, Onboarding, Support)

- Home: `https://ringgitpay.biz/`
- Site index for agents: `https://ringgitpay.biz/llms.txt`
- Sitemap: `https://ringgitpay.biz/sitemap_index.xml`
- Sign-up form: `https://ringgitpay.biz/sign-up/`
- Support and FAQ: `https://ringgitpay.biz/support/`
- Instant Collect (payment links, payment gateway, digital invoice): `https://ringgitpay.biz/instant-collect/`
- Recurring billing (auto debit, e-Mandate, subscription plan): `https://ringgitpay.biz/recurring-billing/`
- Payment pages: `https://ringgitpay.biz/payment-pages/`
- Pricing: `https://ringgitpay.biz/pricing/`
- Merchant portal login: `https://biller.ringgitpay.biz/`

Use for: onboarding steps, sign-up fields, contacts, payment methods list, settlement timing ("credited in 2 days"), pricing link.

## Searched With No Developer Content

- `https://ringgitpay.biz/docs/`, `/developer/`, `/api/`: 404.
- `https://ringgitpay.com/docs`, `https://ringgitpay.co/docs`: 404.
- `developer.ringgitpay.biz`, `docs.ringgitpay.biz`, `api.ringgitpay.biz`: no DNS response.
- Packagist, npm: no RinggitPay packages. GitHub: only a WordPress.org mirror (`common-repository/ringgitpay`), the PDF rehost, and an unrelated automation repo.
- WordPress media library on ringgitpay.biz: no integration PDF.

## Coverage

- Enumerated: 2 integration PDFs (22 + 24 pages), 1 plugin (2 files: PHP class, readme; trunk differs from tag 1.0.4 only in the version header), 69 site URLs in `llms.txt`/sitemap (blog and marketing).
- Read in full: both PDFs (all pages, including the token, version and flow screenshots on DD pages 6-8 and spec page 6), the plugin PHP and readme, plugin metadata. Read for facts: home, sign-up, support, instant-collect, recurring-billing, pricing. Blog posts, event pages and landing variants (SP1, gs1, thank-you pages) were checked through `llms.txt` summaries only; they contain no integration content.
- Left out: e-invoice, payment links, payment pages and smart-recovery dashboard features (no API documented for them).
