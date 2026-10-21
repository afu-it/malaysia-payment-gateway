# GoMobi Source Map

Use this map to pick the right official page. Base: `https://gomobi.io/documents/<route>`. Checked 2026-10-21. The docs are an Angular single-page app; pages render client-side, so a plain HTTP fetch returns only the shell. There is no `llms.txt`, sitemap or OpenAPI file.

## General

- `https://gomobi.io/documents/getting-started`
- `https://gomobi.io/documents/error-codes`
- `https://gomobi.io/documents/sandbox-environments`

Use for: product list by country, generic `0000`/`0001` codes, sandbox FPX and card test data.

## Authentication and Encryption

- `https://gomobi.io/documents/authentication`
- `https://gomobi.io/documents/authentication/aes-cbc`
- `https://gomobi.io/documents/authentication/3des`

Use for: AES-CBC PBKDF2 encryption/decryption per product (FPX, e-wallet, DuitNow, cards, EZYCASH), 3DES mobiApiKey header and IPN decryption, code samples in Java, Python, PHP and C#.

## Hosted Checkout

- `https://gomobi.io/documents/checkout/overview`
- `https://gomobi.io/documents/checkout/malaysia`
- `https://gomobi.io/documents/checkout/malaysia/authorization`
- `https://gomobi.io/documents/checkout/malaysia/api`

Use for: checkout init endpoint, Bearer key, HMAC-SHA256 X-Signature, X-Timestamp, X-Nonce, body fields, order summary, session expiry, API secret issuance.

## FPX

- `https://gomobi.io/documents/payin/malaysia/internet-banking/fpx`
- `https://gomobi.io/documents/payin/malaysia/internet-banking/fpx/get-available-banks`
- `https://gomobi.io/documents/payin/malaysia/internet-banking/fpx/initiate-deposit`
- `https://gomobi.io/documents/payin/malaysia/internet-banking/fpx/payment-response`
- `https://gomobi.io/documents/payin/malaysia/internet-banking/fpx/ipn`
- `https://gomobi.io/documents/payin/malaysia/internet-banking/fpx/status`

Use for: B2C/B2B limits, bank list, deposit form and checkSum, redirect fields, debit auth codes, IPN GET/POST formats and transaction_signature, status requery.

## Cards

- `https://gomobi.io/documents/payin/malaysia/cards`
- `https://gomobi.io/documents/payin/malaysia/cards/card-payment`
- `https://gomobi.io/documents/payin/malaysia/cards/saved-card`

Use for: card form post, carddetails format, save card and walletId, WALLET_DIRECT charges, card errors.

## E-wallets

- Boost: `https://gomobi.io/documents/payin/malaysia/ewallet/boost`, `.../boost/genrate-qr`, `.../boost/status-check`, `.../boost/ipn`
- GrabPay: `https://gomobi.io/documents/payin/malaysia/ewallet/grab`, `.../grab/create-one-time-link`, `.../grab/callBack-url`, `.../grab/status`, `.../grab/ipn`
- ShopeePay: `https://gomobi.io/documents/payin/malaysia/ewallet/shopee`, `.../shopee/genrate-qr`, `.../shopee/callBack-url`, `.../shopee/status-check`, `.../shopee/ipn`
- Touch 'n Go: `https://gomobi.io/documents/payin/malaysia/ewallet/touch-n-go`, `.../touch-n-go/genrate-qr`, `.../touch-n-go/callBack-url`, `.../touch-n-go/status-check`, `.../touch-n-go/ipn`

Use for: QR/link generation, checkSum over minified JSON, status services, callback and IPN fields, expiry and minimums, error descriptions.

## DuitNow QR

- `https://gomobi.io/documents/payin/malaysia/qr-payments/duitnow-qr`
- `https://gomobi.io/documents/payin/malaysia/qr-payments/duitnow-qr/payment-api`
- `https://gomobi.io/documents/payin/malaysia/qr-payments/duitnow-qr/status-check`
- `https://gomobi.io/documents/payin/malaysia/qr-payments/duitnow-qr/ipn`

Use for: encrypted requestSignature payload, 3DES header, status GET, encrypted IPN, E-codes.

## Cash Over Counter (EZYCASH)

- `https://gomobi.io/documents/payin/malaysia/cash-over-counter`
- `https://gomobi.io/documents/payin/malaysia/cash-over-counter/payment-api`
- `https://gomobi.io/documents/payin/malaysia/cash-over-counter/status-check`
- `https://gomobi.io/documents/payin/malaysia/cash-over-counter/ipn`

Use for: 7-Eleven cash payment, txnData redirect data, status, IPN, E-codes, 48-hour validity.

## Payouts

- `https://gomobi.io/documents/payout/overview`
- `https://gomobi.io/documents/payout/malaysia`
- `https://gomobi.io/documents/payout/malaysia/api`
- `https://gomobi.io/documents/payout/malaysia/ipn`
- `https://gomobi.io/documents/payout/malaysia/status`

Use for: PAYOUT_TXN_REQ fields, e-wallet payouts, BIC code table, 3DES payout IPN, PAYOUT_TXN_DETAIL status, validation errors.

## Plugins

- `https://gomobi.io/documents/payin/malaysia/plugins/shopify`
- `https://gomobi.io/documents/payin/malaysia/plugins/woocommerce`
- `https://gomobi.io/documents/payin/malaysia/plugins/prestashop`
- `https://gomobi.io/documents/payin/malaysia/plugins/opencart`
- `https://gomobi.io/documents/payin/malaysia/plugins/magento`

Use for: plugin install and configuration steps.

## Support

- `https://gomobi.io/documents/support/whitelist-ip`
- `https://gomobi.io/documents/support/faq`
- `https://gomobi.io/documents/support/contact`

Use for: IPN and payout IP whitelisting, settlement and operations FAQ, transaction timeouts, FPX and card status code tables, support channels.

## Company and Pricing

- `https://www.mobi.xyz/product`
- `https://www.mobi.xyz/contact`
- `https://gomobi.io/portal/login`

Use for: brand and product positioning, same-day settlement claim, sales contact (pricing is not published), portal login.

## Coverage

- Enumeration: the route table in the docs' main JavaScript bundle (the SPA router), 95 routes in total. No sitemap, llms.txt or OpenAPI exists.
- Read in full: 59 routes (every Malaysia and general page above). Content came from each page's lazy-loaded bundle, including field tables, code samples and error tables.
- Left out: 36 routes for Indonesia (`payin/indonesia/...`: virtual account, OVO, DANA, LinkAja, ShopeePay Indonesia, QRIS; `payout/indonesia/...`) and India (`payout/india/...`). Reason: non-MYR markets, not usable by a Malaysian merchant collecting MYR. They share the same AES-CBC/3DES patterns.
- Also checked: `https://www.mobi.xyz/sitemap.xml` (marketing pages only; product and contact pages used). `https://gomobi.io/documents_v2/` exists as another SPA build; not used.
- Not found in official docs (checked 2026-10-21): refund/void API, card status or IPN API, hosted checkout webhook or status API, published IPN IP list, published pricing, KYB document list, e-wallet/DuitNow/cash sandbox test data.
