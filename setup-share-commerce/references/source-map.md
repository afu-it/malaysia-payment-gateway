# Share Commerce Source Map

Use this map to pick the right official source before answering or coding. All pages checked 2026-10-21; docs pages show "Last updated on 22nd Sep 2026".

## Coverage

- Index used: the docs home page `https://devdocs.share-commerce.com/` (Raneto knowledge base). No `llms.txt`, `robots.txt` or working `sitemap.xml` (404/500). No OpenAPI or Postman collection published.
- Pages enumerated from the index: 29 (plus the home page). Pages read: 29 of 29.
- Official plugin repositories enumerated on `github.com/share-commerce`: 6. Read: WooCommerce 8.x, WooCommerce 5.9.0, OpenCart 3, OpenCart 4 (ocmod zip), Magento 2.3.6, PrestaShop 1.7 (request builders and callback handlers).
- Left out: the separate in-store terminal docs site `https://devdocsterminal.share-commerce.com/` (not online payments). The local Terminal Payment API page on the main docs is summarised only.

## Overview and Onboarding

- Overview: `https://devdocs.share-commerce.com/Overview`
- Pre-requisite: `https://devdocs.share-commerce.com/PreRequisite`
- Company site: `https://share-commerce.com/`, `https://share-commerce.com/solutions-and-services/payment-gateway-solutions/`, `https://share-commerce.com/contact-us/`

Use for: process flow (redirect plus server callback), onboarding items (server IP, callback URL), credentials issued per environment, contact.

## Hosted Payment

- Request: `https://devdocs.share-commerce.com/Payment/PaymentRequest`
- Response Redirect / Callback: `https://devdocs.share-commerce.com/Payment/RedirectCallback`
- Seemless Payment Request: `https://devdocs.share-commerce.com/Payment/SeemlessPaymentRequest`

Use for: base URLs, request field order, Rule A signing, Versioning-dependent sign fields, redirect and callback fields, iframe checkout and postMessage.

## Status and Void

- Enquiry: `https://devdocs.share-commerce.com/Payment/Enquiry`
- Payment Void: `https://devdocs.share-commerce.com/Payment/PaymentVoid`

Use for: requery request, same-day void, Rule B raw-body signing in the `SCSign` header.

## Pre-Auth

- Pre Auth (Legacy): `https://devdocs.share-commerce.com/PreAuth/PreAuth`
- Create Checkout (Pre Auth): `https://devdocs.share-commerce.com/PreAuth/CreateCheckout`
- Pre Auth (Void): `https://devdocs.share-commerce.com/PreAuth/PreAuthVoid`
- Capture / Sale: `https://devdocs.share-commerce.com/PreAuth/CaptureSale`
- Capture / Sale (Void): `https://devdocs.share-commerce.com/PreAuth/CaptureSaleVoid`
- Enquiry: `https://devdocs.share-commerce.com/PreAuth/Enquiry`
- Response: `https://devdocs.share-commerce.com/PreAuth/Response`

Use for: card authorisation and capture, the two response shapes, integer RespCode on CreateCheckout, signature placement differences.

## Recurring

- Create Request: `https://devdocs.share-commerce.com/Recurring/CreateRequest`
- Response (Redirect): `https://devdocs.share-commerce.com/Recurring/ResponseRedirect`
- Enquiry: `https://devdocs.share-commerce.com/Recurring/Enquiry`
- Callback / Enquiry Response: `https://devdocs.share-commerce.com/Recurring/CallbackEnquiryResponse`

Use for: subscription plan fields, frequency rules, recharge options, cycle list, recurring signatures.

## Code Tables

- Appendix 1 Response Code: `https://devdocs.share-commerce.com/Appendix1`
- Appendix 2 Payment Method: `https://devdocs.share-commerce.com/Appendix2`
- Appendix 3 Payment Type: `https://devdocs.share-commerce.com/Appendix3`
- Appendix 4 Recurring Status: `https://devdocs.share-commerce.com/Appendix4`
- Appendix 5 Transaction Status: `https://devdocs.share-commerce.com/Appendix5`

Use for: `RespCode`, `PaymentMethod`, `PaymentType`, `RecurringStatus`, `TxnStatus` enums.

## Plugins (docs pages and official source)

- `https://devdocs.share-commerce.com/Plugins/WooCommerce` and `https://github.com/share-commerce/Plugin.WooCommerce-8.x` (`includes/class-wc-gateway-sharecommerce.php`), `https://github.com/share-commerce/Plugin.WooCommerce-5.9.0` (`src/scpay.php`)
- `https://devdocs.share-commerce.com/Plugins/OpenCart` and `https://github.com/share-commerce/Plugin.Opencart3` (`catalog/controller/extension/payment/scpay.php`), `https://github.com/share-commerce/Plugin.Opencart4`
- `https://devdocs.share-commerce.com/Plugins/Magento` and `https://github.com/share-commerce/Plugin.Magento.2.3.6` (`Helper/SCPayRequest.php`, `Controller/Payment/Response.php`)
- `https://devdocs.share-commerce.com/Plugins/Prestashop` and `https://github.com/share-commerce/Plugin.Prestashop.1.7.-` (`controllers/front/payment.php`, `callback.php`, `validation.php`)
- `https://devdocs.share-commerce.com/Plugins/EasyStore`

Use for: plugin install steps, real request field order in production code, callback verification by iterating received params, `OK` callback reply, `MerchantOrderNo` uniqueness pattern.

## Terminal (in-store)

- Payment Api: `https://devdocs.share-commerce.com/Terminal`

Use for: LAN terminal transaction types only. Not part of online checkout.

## Not Documented (checked 2026-10-21)

Refund API, payout/settlement API, rate limits, Share Commerce callback source IPs, callback retry policy, public sandbox test cards, FPX bank codes, DuitNow QR code, KYC document list, settlement schedule, pricing, signing input for Payment Enquiry, algorithm for the Seemless response `Signature` field.
