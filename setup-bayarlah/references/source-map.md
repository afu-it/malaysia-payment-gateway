# bayarlah Source Map

Use this map to pick the right source before answering or coding. All pages are on the official GitBook at `https://doc.bayarlah.my/` (append `.md` for Markdown) or the official site `https://www.bayarlah.my/`. Checked 2026-10-21.

## Coverage

- Index used: `https://doc.bayarlah.my/llms.txt` (21 pages), cross-checked with `https://doc.bayarlah.my/sitemap-pages.xml` (21 URLs) and `https://doc.bayarlah.my/llms-full.txt`.
- Pages enumerated: 21. Pages read: 21 (every page, via `llms-full.txt` and each page's `.md`).
- Section landing pages with no content beyond a link list: `bayarlah-public-document`, `upay-web-api/list-of-integration-interfaces`.
- Pages referenced in the docs but not published: the "appendix" with the full Payment Method List, Secondary Payment Method List, Transaction Type List and status code tables (`/appendix` and `/upay-web-api/appendix` return 404). The only published appendix is the error-code table on the redirect page.
- Images not transcribed: activation and check-in sequence diagrams, Event Order Notification flow diagrams (their text is covered by the pages).
- Other official sources checked: `https://www.bayarlah.my/merchant`, `/faqs`, `/payment-solution`, `/contact`.
- No official SDK, GitHub repo, WordPress plugin or Packagist package found. The npm package `bayarlah` is published by an unrelated individual (`razmans`), so it was not used.
- Left out: nothing global-only; all pages are in scope.

## Overview And Terms

- Overview: `https://doc.bayarlah.my/overview`
- Introduction to Terms: `https://doc.bayarlah.my/introduction-to-terms`
- Upay Web API intro: `https://doc.bayarlah.my/upay-web-api`

Use for: channels (DuitNow, WeChat, Alipay+ coming soon), B-to-C vs C-to-B, `{api_domain}`, vendor, `app_id`, store, activation code, terminal, `device_id`, PayNet integration, contact.

## Getting Credentials

- Preparation Before Integration: `https://doc.bayarlah.my/preparation-before-integration`

Use for: information to send bayarlah to get `vendor_sn`, `vendor_key`, `app_id`.

## Integration Flow And Polling

- The Integration Process: `https://doc.bayarlah.my/upay-web-api/the-integration-process`

Use for: activate, check-in, transact order; lost check-in response handling; pre-order polling (2s for 30s, then 5s).

## Signatures

- Signature Mechanism: `https://doc.bayarlah.my/upay-web-api/signature-mechanism`
- Callback Signature Verification: `https://doc.bayarlah.my/upay-web-api/callback-signature-verification`
- Redirect gateway signature rules: `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/redirect-payment-interface-wechat-only`
- Mini Program signature: `https://doc.bayarlah.my/more-payment-function-interfaces/mini-program-plugin-api-wechat-only`
- Event push signature: `https://doc.bayarlah.my/event-order-notification`

Use for: `MD5(body + key)` with `Authorization: sn sign`, which key signs which call, cents, string responses; SHA256WithRSA callback check; uppercase MD5 query signature; `MD5(body + app_secret)` push check.

## Terminal Management

- Activation: `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/activation-interface`
- Check-in: `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/check-in`

Use for: `/terminal/activate` and `/terminal/checkin` fields, activation code validity (7 days), key rotation, FAQ on terminals and device IDs.

## Payments

- Pre-order (DuitNow and WeChat QR, WAP, Mini Program): `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/pre-order-payment-interface`
- Payment code pay (CPM, WeChat): `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/payment-code-payment-interface-cpm-wechat-only`
- Redirect gateway (WeChat WAP): `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/redirect-payment-interface-wechat-only`
- Terminal Order Push: `https://doc.bayarlah.my/more-payment-function-interfaces/terminal-order-push`
- Mini Program plugin: `https://doc.bayarlah.my/more-payment-function-interfaces/mini-program-plugin-api-wechat-only`

Use for: request and response fields, payway and sub_payway values, QR fields, 4-minute window and polling schedule, `return_url` fields, error-code appendix, T9 order push.

## Status, Callbacks And Notifications

- Query: `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/query-interface`
- Callback (`notify_url`): `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/callback-interface`
- Event Order Notification: `https://doc.bayarlah.my/event-order-notification`

Use for: `/upay/v2/query` fields, final statuses, callback body and `success` reply, retry intervals, HTTPS requirement, push body fields, Python verification sample.

## Refunds And Cancellation

- Refund: `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/refund-wechat-and-card`
- Order Cancellation (cancel and revoke): `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/order-cancellation-wechat-only`

Use for: `refund_request_no` rules, partial refunds, three-month window, WeChat refund authorization, same-day cancel rules.

## Merchant, Settlement And Contact

- Merchant page: `https://www.bayarlah.my/merchant`
- FAQs: `https://www.bayarlah.my/faqs`
- Contact: `https://www.bayarlah.my/contact`
- Merchant platform: `https://s.bayarlah.net`

Use for: who can apply, sales contact, settlement timing, merchant portal, activation code generation. No public pricing page exists.
