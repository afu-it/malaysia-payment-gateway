# LekirPay Source Map

Use this map to pick the right source before answering or coding. All checked 2026-10-21.

## Coverage

- Index used: the docs sidebar on `https://app.lekirpay.com/doc/*` plus links hidden in HTML comments. No `llms.txt`, `sitemap.xml`, OpenAPI, or Postman collection exists (`/llms.txt` and `/sitemap.xml` return 404).
- Pages enumerated: 12 docs pages under `app.lekirpay.com/doc/` plus 1 downloadable sample (`/download/client.rar`). Pages read: 12 of 12, plus the sample.
- Visible in sidebar (9): index, flow, token, payment/secure, payment/normal, payment/callback, paymentstatus, paymentGroup/create, paymentGroup/data.
- Hidden in sidebar HTML comments (3 pages + sample): introtoken, paymentbulk, transferrequest, Sample Program (Laravel `client.rar`).
- Additional official source: GiveWP plugin `lekirpay-givewp` 3.1.2 source (all PHP files read).
- Left out: nothing in scope. Hidden payout/transfer pages are documented as legacy only because their host `lekirpay.co` does not resolve.

## Product, Pricing, Terms

- Website: `https://lekirpay.com/`
- Pricing: `https://lekirpay.com/#pricing`
- Terms and conditions: `https://lekirpay.com/terms-and-condition/`
- Privacy policy: `https://lekirpay.com/privacy-policy/`
- Signup: `https://app.lekirpay.com/validate`

Use for: company identity, plans, MYR-only, FPX/card availability, settlement claims, refund policy, KYC wording.

## API Basics And Auth

- Introduction: `https://app.lekirpay.com/doc/index`
- Auth and API flow: `https://app.lekirpay.com/doc/flow`
- Requesting tokens: `https://app.lekirpay.com/doc/token`
- Issuing access tokens (hidden, legacy auth-code wording): `https://app.lekirpay.com/doc/introtoken`

Use for: live and sandbox base URLs, content types, HTTP status codes, credential names (ID, Lekirkey, Lekir-Signature key), dashboard location of keys, `client_credentials` token request and response.

## Payment Creation

- Send payment: `https://app.lekirpay.com/doc/payment/normal`
- Send secure payment: `https://app.lekirpay.com/doc/payment/secure`

Use for: request fields, required flags, `reference_no` length, `method`, `charge_to_payer`, `group_id`, response fields, Lekir-Signature checksum string order and worked example.

## Callback

- Payment callback notification: `https://app.lekirpay.com/doc/payment/callback`

Use for: callback POST fields, `type` values (`callback`, `redirect`). Pair with the plugin `return_listener` for the requery pattern.

## Status

- Payment status: `https://app.lekirpay.com/doc/paymentstatus`

Use for: requery endpoint, response fields, status enum (`Created`, `Proceed`, `Paid`, `Failed`).

## Payment Groups

- Create payment group: `https://app.lekirpay.com/doc/paymentGroup/create`
- Get payment group data: `https://app.lekirpay.com/doc/paymentGroup/data`

Use for: creating and listing groups, `group_id` values for create payment.

## Hidden Legacy Pages

- Bulk payment: `https://app.lekirpay.com/doc/paymentbulk`
- Transfer request: `https://app.lekirpay.com/doc/transferrequest`
- Sample program (2018 Laravel client, "Lunapay" naming, staging hosts): `https://app.lekirpay.com/download/client.rar`

Use for: completeness only. Do not build on them; the hosts are legacy and the sample targets `staging.lunapay.co` and localhost.

## Official Plugin

- Plugin page: `https://wordpress.org/plugins/lekirpay-givewp/`
- Source zip: `https://downloads.wordpress.org/plugin/lekirpay-givewp.3.1.2.zip`
- Files read: `give-lekirpay.php`, `includes/lekirpay_WPConnect.php`, `includes/lekirpay_API.php`, `includes/give-lekirpay-gateway.php`, `includes/admin/*`, `readme.txt`.

Use for: sandbox host used in code (`matkilau.lekirpay.com`), form-urlencoded request style, callback handling via status requery with `status == 'Paid'`, checksum string order confirmation, settings names, dashboard key location, support links. Do not copy its secure-payment path (bug noted in `lekirpay-api.md`).

## Support

- Contact: `https://lekir.tech/ask-us-now/`
- Ticket desk: `https://lekirpay.freshdesk.com/support/login`

Use for: confirming sandbox host, KYC documents, callback retry policy, refund process, and anything marked "not documented".
