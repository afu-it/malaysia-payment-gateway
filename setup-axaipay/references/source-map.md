# Axaipay Source Map

Use this map to pick the right official source before answering or coding. All sources checked 2026-10-21.

## Coverage

- Index used: the Freshdesk knowledge base tree. Root `https://support.axaipay.com/support/solutions` lists 3 categories (For Shoppers `73000222458`, Technical Support `73000254495`, For Axaipay Merchants `73000347755`) and 8 folders. Every folder page was enumerated, including the second page of the AutoDebit folder (`/folders/73000373068/page/2`).
- Enumerated: 30 KB articles. Read: 30, plus all attachments: eComPay API PDF v1.18 (37 pages), AutoDebit API PDF v1.6 (32 pages), Refund Request Form v2.0 (1 page), DirectDebit eMandate T&C (6 pages, skimmed for API-relevant content; none found), OpenCart handbook (21 pages).
- Official plugin source read: WooCommerce v2.0.0 (`class-axaipay-signature.php`, `class-axaipay-api.php`, `class-wc-gateway-axaipay.php`, main plugin file), PrestaShop alpha 1.0 (`axaipay.php`, `validation.php`), VirtueMart 1.0 (`axaipay.php`), OpenCart 1.0 (`catalog/controller/payment/axaipay.php`).
- Website pages read: home, `/online-payments`, `/autodebit`, `/qrpay`, `/3rd-party-integration`, `/contact-us`, `pages-sitemap.xml` (27 URLs; no developer or pricing page in it).
- `https://www.axaipay.com/llms.txt` returns the site shell, not an index. No OpenAPI file, Postman collection, npm or packagist package, or official GitHub organisation found. GitHub search for "axaipay" returns only third-party repos (not used).
- Every signature example in both PDFs was recomputed with HMAC-SHA512 + Base64. 12 of 13 printed digests match; the eComPay inquiry-by-`mchtTrxnId` digest does not match its own stated field order (see Known Doc Conflicts).
- Endpoints documented: 9 (6 Axaipay URLs, 3 merchant-side callback targets). All are in the catalogue in `axaipay-api.md`.
- Images (technical/process flow diagrams in both PDFs) are not text-extractable; the numbered step tables beside them were used.

## API Documentation

- eComPay article: `https://support.axaipay.com/support/solutions/articles/73000523456-axaipay-ecompay-api-technical-document-v1-18`
- eComPay PDF: `https://support.axaipay.com/helpdesk/attachments/73176544580`

Use for: hosted page and bypass page fields, request and response signatures, QR code transaction, inquiry and void, status/acquirer/payment method enums, FPX bank codes, error codes, test cards.

- AutoDebit article: `https://support.axaipay.com/support/solutions/articles/73000611021-axaipay-autodebit-api-documentation-v1-6`
- AutoDebit PDF: `https://support.axaipay.com/helpdesk/attachments/73081657622`

Use for: eMandate enrolment with and without product code, recurring callbacks, `trxnRecurringNo`, AutoDebit inquiry and its signatures.

- APIs folder: `https://support.axaipay.com/support/solutions/folders/73000373064`

## Official Plugins

- WooCommerce: `https://support.axaipay.com/support/solutions/articles/73000530285-axaipay-plugin-for-woocommerce` (zip `https://support.axaipay.com/helpdesk/attachments/73197568765`)
- PrestaShop: `https://support.axaipay.com/support/solutions/articles/73000617704-axaipay-plugin-for-prestashop` (zip `https://support.axaipay.com/helpdesk/attachments/73071618024`)
- VirtueMart: `https://support.axaipay.com/support/solutions/articles/73000617339-axaipay-plugin-for-virtuemart` (zip `https://support.axaipay.com/helpdesk/attachments/73070212526`)
- OpenCart: `https://support.axaipay.com/support/solutions/articles/73000601885-axaipay-plugin-for-opencart` (handbook and zip on Google Drive links in the article)
- Shopping Carts folder: `https://support.axaipay.com/support/solutions/folders/73000400936`

Use for: real-world signature code, callback handling, ack response, phone and amount formatting, plugin settings, sandbox toggles.

## Refunds

- `https://support.axaipay.com/support/solutions/articles/73000612883-how-do-i-process-a-refund-through-axaipay-`
- Refund Request Form: `https://support.axaipay.com/helpdesk/attachments/73059771805`

Use for: manual refund process, timelines, refund modes, fees.

## AutoDebit Shopper FAQ (folder `73000373068`)

- `.../articles/73000523457-what-is-autodebit-or-directdebit-emandate-`
- `.../articles/73000523458-directdebit-e-mandate-terms-and-conditions` (T&C PDF `https://support.axaipay.com/helpdesk/attachments/73003061819`)
- `.../articles/73000523459-how-to-enroll-directdebit-emandate-with-merchant-`
- `.../articles/73000523460-can-directdebit-emandate-work-with-any-account-`
- `.../articles/73000523461-once-it-s-set-up-do-i-have-to-do-anything-`
- `.../articles/73000523462-will-the-payment-always-be-on-the-same-date-every-month-`
- `.../articles/73000523463-what-if-changes-are-needed-`
- `.../articles/73000523464-how-do-i-cancel-directdebit-emandate-authorization-`
- `.../articles/73000523465-can-money-be-collected-from-my-account-after-i-have-cancelled-a-directdebit-emandate-authorisation-`
- `.../articles/73000589635-does-axaipay-merchants-offer-autodebit-or-directdebit-emandate-`
- `.../articles/73000589639-why-my-first-directdebit-emandate-billing-is-rm1-lesser-than-actual-recurring-amount-`

(`...` = `https://support.axaipay.com/support/solutions`)

Use for: customer enrolment steps, RM1 deduction, debit dates, eligible accounts, cancellation.

## Merchant Account And Security

- Prohibited activities: `.../articles/73000526931-prohibited-business-activities` and `https://www.axaipay.com/prohibited-activities`
- Logo upload: `.../articles/73000589633-how-to-upload-company-business-logo-to-axaipay-payment-page-`
- 2FA: `.../articles/73000672306-what-is-two-factor-authentication-2fa-`, `.../73000672307-how-2fa-protects-your-store-`, `.../73000672308-myth-vs-reality-does-2fa-slow-down-daily-operations-`, `.../73000672309-how-to-enable-two-factor-authentication-2fa-`

Use for: onboarding eligibility, portal settings, account security.

## Shopper And Security Center (no API content)

- `.../articles/73000514819-why-is-axai-on-my-bank-statement-`
- `.../articles/73000570782-why-am-i-receiving-payment-request-emails-from-axaipay-`
- `.../articles/73000570778-tips-to-spot-a-phishing-email-or-text`
- `.../articles/73000570780-what-to-do-if-you-receive-a-phishing-message`
- `.../articles/73000570781-tips-to-stay-safe-on-public-wi-fi`
- `.../articles/73000591066-8-tips-to-keep-your-card-safe-and-secure`

Use for: statement descriptor, genuine payment domain (`secured.axaipay.my`), support contact.

## Product And Pricing

- `https://www.axaipay.com/online-payments`
- `https://www.axaipay.com/autodebit`
- `https://www.axaipay.com/qrpay`
- `https://www.axaipay.com/3rd-party-integration`
- `https://www.axaipay.com/contact-us`

Use for: product positioning, no-monthly-fee claim, sales contact. No public rate card exists.

## Known Doc Conflicts (tell the integrator)

1. eComPay inquiry by `mchtTrxnId`: text and WooCommerce plugin say `mchtTrxnId + merchantId`; the PDF's sample digest matches `merchantId + mchtTrxnId`. Use the text order, confirm with support if `PY016`.
2. eComPay response signature example prints the string without spaces; the digest only matches with spaces kept.
3. `trxnFpxType` is `LCA` in callback docs and `Card` in inquiry docs; buyer name is `trxnBuyerName` in callbacks and `trxnFpxBuyerName` in inquiry.
4. AutoDebit sample forms use `mchtTxnId` and a stray `txnAmount`; the field tables use `mchtTrxnId` and no amount.
5. AutoDebit response signature header line lists fewer fields than the bullet list and example; the 13-field example is correct (digest verified).
6. Test card expiry `08/25` is past.
7. WooCommerce plugin skips signature checks on redirect for status 55.
