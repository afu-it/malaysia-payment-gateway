# PayCools Source Map

Use this map to pick the right source before answering or coding. All docs pages are also served as Markdown by appending `.md`.

## Coverage

- Index used: `https://docs.paycools.com.my/en/llms.txt` and `https://docs.paycools.com.my/en/sitemap.xml` (both list the same 25 pages).
- English pages enumerated: 25. English pages read in full: 25.
- Chinese mirror (`https://docs.paycools.com.my/zh/llms.txt`, 25 pages) read as a cross-check; it adds one detail (WebPay `mobile` starts with `01`, max 10 digits).
- Checked: 2026-10-21.
- Old routes referenced inside the OpenAPI blocks (for example `/en/PaymentOrder/PaymentOrderCreate.html`, `/en/Account.html`) now return 404.
- Not found anywhere official: API base URL, UAT host, retry policy, notification IPs, rate limits.
- Excluded: npm package `paycools` (published by an individual, repo `nswbmw/paycools-node`, not PayCools); no official GitHub, npm, Packagist, or WordPress plugin found.

## Product And Company

- Home: `https://www.paycools.com.my/`
- About: `https://www.paycools.com.my/about/`
- Contact: `https://www.paycools.com.my/contact/`
- QR Code product: `https://www.paycools.com.my/products/qrcode/`
- Payment Links product: `https://www.paycools.com.my/products/paymentLink/` (returned 403 to automated fetch)

Use for: operator identity, onboarding steps, sales contact, product scope, free setup claim.

## Request Envelope And Signatures

- Merchant Unified API Param: `https://docs.paycools.com.my/en/9367977m0`
- Unified API Param (PayCools to merchant): `https://docs.paycools.com.my/en/9367979m0`

Use for: `{appId, sign, param}` envelope, `{sign, param}` notification envelope, SHA256withRSA, 4096-bit keys, PKCS#8/X.509 formats, Java sign/verify samples, example signature (verified with openssl).

## Response Codes

- API Code Collection: `https://docs.paycools.com.my/en/9367978m0`

Use for: `10000` success, `21000` to `21016` errors, account validation failure codes `-20001` to `-20006`.

## WebPay (hosted redirect)

- 4.1 WebPay Create: `https://docs.paycools.com.my/en/507514603e0`
- 4.2 Retrieve WebPay Status: `https://docs.paycools.com.my/en/507514604e0`
- 4.3 WebPay Payment Notification: `https://docs.paycools.com.my/en/507514605e0`
- WebPay Channel Code List: `https://docs.paycools.com.my/en/9368011m0`

Use for: FPX, DuitNow RPP, card, and e-wallet checkout, `payUrl`, status requery, notification fields and events, channel codes.

## DuitNow QR

- 3.1 QR Code Create: `https://docs.paycools.com.my/en/507514599e0`
- 3.2 QR Code Detail: `https://docs.paycools.com.my/en/507514600e0`
- 3.3 QR Code Payment List: `https://docs.paycools.com.my/en/507514601e0`
- 3.4 QR Code Payment Notification: `https://docs.paycools.com.my/en/507514602e0`
- QR Code Channel Code List: `https://docs.paycools.com.my/en/9368008m0`

Use for: dynamic DuitNow QR creation, `qrCodeContent`, QR status, payments on a QR, QR notification.

## Refunds

- 5.1 Refund: `https://docs.paycools.com.my/en/507514606e0`
- 5.2 Get Refund Payments: `https://docs.paycools.com.my/en/507514607e0`
- 5.3 Refund Notification: `https://docs.paycools.com.my/en/507514608e0`
- Refund Support Channel List: `https://docs.paycools.com.my/en/9368019m0`
- Refund Status Enum: `https://docs.paycools.com.my/en/9368021m0`

Use for: full/partial refund support per channel, refund fields, refund status values, fee not returned.

## Disbursement (payouts)

- 2.1 Account Validation: `https://docs.paycools.com.my/en/508846511e0`
- 2.2 Check Account Validation Order Status: `https://docs.paycools.com.my/en/508846512e0`
- 2.3 Create Payout: `https://docs.paycools.com.my/en/507514596e0`
- 2.4 Check Payout Status: `https://docs.paycools.com.my/en/507514597e0`
- 2.5 Disburse callback notification api: `https://docs.paycools.com.my/en/507514598e0`
- Disburse API Supported Types (bank and e-wallet codes): `https://docs.paycools.com.my/en/9368003m0`
- Account Verification Related Enum List: `https://docs.paycools.com.my/en/9392778m0`

Use for: payee validation (bank or DuitNow proxy), name match, payouts to banks and e-wallets, `idempotencyId`, payout status and callback, `transferCode` lists.

## Account

- 1.1 Balance Inquiry: `https://docs.paycools.com.my/en/507514595e0`

Use for: merchant balance before payouts.
