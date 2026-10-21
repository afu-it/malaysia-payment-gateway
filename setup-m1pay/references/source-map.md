# M1Pay Source Map

Use this map to pick the right source before answering or coding. All sources checked on 2026-10-21.

The official Integration Document is an Angular single-page app at `https://www.m1pay.com.my/`, with hash routes under `#/docs/`. The page text lives in the site bundle `https://www.m1pay.com.my/main.js`; that bundle was read directly to capture every table and code sample.

## Coverage

- Index used: the docs side menu and Angular route table in `main.js` (routes `docs/introduction/overview`, `docs/introduction/get-started`, `docs/payment-service`, `docs/merchant-integration/M1Pay-OAuth2`, `docs/merchant-integration/Initiate-M1Pay-Payment-Request`, `docs/merchant-integration/Get-Transaction`, `docs/merchant-integration/Calling-Merchant-Callback-URL`, `docs/APPENDIX`, `docs/APPENDIX/error-codes-fpx`).
- Pages enumerated: 9. Pages read: 9 (all).
- Linked PDFs: `https://www.m1pay.com.my/assets/documents/M1Pay_IntegrationGuideline.pdf` and `https://www.m1payall.com/images/pdf/M1PAY-API-Documentation-V1.0.8-latest.pdf` (the docs "Download" link). Both return HTTP 404 as of 2026-10-21; the web docs replace them.
- No `llms.txt`, `sitemap.xml`, `robots.txt`, OpenAPI or Postman collection on `www.m1pay.com.my` (all 404).
- Official SDK/plugin: none found on an M1Pay or MobilityOne domain. The WordPress.org plugin `m1pay-buy-now-button` was closed on 2022-10-26 (guideline violation) and is unavailable.
- Left out: GitHub repos under the personal account `amaleena123` (M1PAY-PHP, M1PAY-LARAVEL, M1PAY-Woocommerce, M1PAY-Guidelines). They are personal projects with a personal support email and a "use at your own discretion" disclaimer, not published on an M1Pay domain, so no facts were taken from them. An integrator may read them as unofficial examples.
- Global-only areas: none; M1Pay is Malaysia-only (MYR).

## Overview And Security

- `https://www.m1pay.com.my/#/docs/introduction/overview`

Use for: HTTPS requirement, IP address filtering, `signedData` purpose, private key regeneration, OAuth2 model, payment flow overview.

## Account And Credentials

- `https://www.m1pay.com.my/#/docs/introduction/get-started`

Use for: Merchant Code, Private Key, Secret Key; IP address, Web Redirect URL, Host-to-Host Callback URL (255 character limit, one per account).

## Payment Wall vs Individual Channels

- `https://www.m1pay.com.my/#/docs/payment-service`

Use for: choosing the hosted Payment Wall (omit `channel`) or individual channel selection on the merchant site.

## OAuth2 Token

- `https://www.m1pay.com.my/#/docs/merchant-integration/M1Pay-OAuth2`

Use for: sandbox and production Keycloak token URLs, `grant_type`, `client_id`, `client_secret`, `access_token`.

## Create Transaction

- `https://www.m1pay.com.my/#/docs/merchant-integration/Initiate-M1Pay-Payment-Request`

Use for: create transaction URLs, header, request fields, response `transactionId`/`signedData`, Payment Wall rules, channel table, FPX B2C/B2B1 examples, UAT channel limits, FPX bank list URLs, test bank `SBI Bank A`.

## FPX Bank List (live endpoints)

- `https://gateway.m1payall.com/m1payfpx/api/bank-list/B2C`
- `https://gateway.m1payall.com/m1payfpx/api/bank-list/B2B1`
- `https://gateway.m1pay.com.my/fpx/api/bank-list/B2C`
- `https://gateway.m1pay.com.my/fpx/api/bank-list/B2B1`

Use for: the bank list JSON shape (`id`, `bankId`, `title`, `businessModel`, `fpxOnline`), observed by GET on 2026-10-21.

## Status Requery

- `https://www.m1pay.com.my/#/docs/merchant-integration/Get-Transaction`

Use for: Get Transaction URLs, Bearer header, `transactionId` path parameter, response fields including `transactionStatus`, `fpxTxnId`, `model`, `failureReason`.

## Callback

- `https://www.m1pay.com.my/#/docs/merchant-integration/Calling-Merchant-Callback-URL`

Use for: when M1Pay calls back, 15 retries at 10-minute intervals, form-urlencoded body fields, status list, per-channel success values, callback verification string.

## Signing And Verification

- `https://www.m1pay.com.my/#/docs/APPENDIX`

Use for: request signing string order, 2-decimal amount rule, SHA1withRSA, hex output, Java/PHP/C# signing and verification samples, X.509 certificate verification, `ErrorCode [09]`/`[03]` strings.

## Error Codes

- `https://www.m1pay.com.my/#/docs/APPENDIX/error-codes-fpx`

Use for: FPX debit authentication status names and codes returned in `failureReason`.

## Registration, Pricing, Settlement

- `https://www.m1pay.com.my/` (homepage: Register button, channels, footer contact `bd@mobilityone.com.my`, `+603-9213 0669`)
- `https://www.m1pay.com.my/assets/documents/Form-A1M1Pay.pdf`

Use for: application steps, due-diligence documents, MDR schedule and payout terms (link only), settlement report timing, refund and chargeback terms.

## Portals And Demo

- Production merchant portal: `https://www.m1pay.com.my/merchant-portal/`
- UAT merchant portal: `https://m1payall.com/merchant-portal/`
- Demo ("Try Sandbox"): `https://m1payall.com/payment-test/`

Use for: where to set IP, Web Redirect URL, Callback URL and read Client ID/Secret; seeing the hosted flow. The demo uses an internal endpoint; do not treat it as the merchant API.
