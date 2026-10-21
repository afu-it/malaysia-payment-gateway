# Maybank2u Pay Source Map

Use this map to pick the right official source before answering or coding. All sources are Maybank-owned (`m2upay.maybank2u.com.my`, `maybank2u.com.my`). Checked 2026-10-21.

## Coverage

- Index used: the SDK docs page itself (single page, anchors `#backend`, `#realtime-payment-notification`) and its download links. `/llms.txt`, `/robots.txt` and `/sitemap.xml` on `m2upay.maybank2u.com.my` all return the Angular portal shell, so no machine index exists.
- Enumerated: 1 SDK docs page, 3 SDK zips (Java, JavaScript, PHP), 1 API guide inside the Java jar, 1 FAQ page, 1 portal app bundle (application, technical requirement and Payee Code screens). Read: all 7.
- Searched for, not found as official: Maybank-published WooCommerce/WordPress plugin, npm/packagist package, GitHub repo, OpenAPI or Postman collection, public Maybank eBPG card integration spec.
- Left out: POS, mPOS, QRPayBiz, Maybank2u Biller, JomPAY, FPX and AutoDebit FAQ sections (not online API checkout); eBPG technical details (not published).

## Integration Flow, Request, Encryption

- SDK docs: `https://m2upay.maybank2u.com.my/assets/sdk/index.html`
- PHP SDK: `https://m2upay.maybank2u.com.my/assets/sdk/sdk/PHP%20SDK%20v1.0.zip` (`m2upay_backend/M2UPay.php`)
- JavaScript SDK: `https://m2upay.maybank2u.com.my/assets/sdk/sdk/JS%20SDK%20v1.0.zip` (`m2upay_backend/m2upay_backend.js`, `Sample/server.js`, `Sample/index.html`, `Sample/js/paymentsctrl.js`)
- Java SDK: `https://m2upay.maybank2u.com.my/assets/sdk/sdk/Java%20SDK%20v1.0.zip` (`m2u_backend/M2UPayment.jar` classes `PaymentGateway`, `PGParams`; `Sample/src/main/java/com/example/SampleController.java`)

Use for: implementation flow, `getEncryptionString` parameters and lengths, `envType` values, action URLs, send-string format, AES-128-ECB key/salt/iterations, URL encoding, return shape, `FAIL` result, Java method name.

## Front End Pop-up

- `m2upay_frontend.js` in each SDK zip (identical logic)
- SDK docs "Frontend" section

Use for: `m2upay.initPayment(encString, action, subService)`, form fields `q` and `i`, pop-up window name and size, `OT` sub-service.

## Realtime Payment Notification

- SDK docs "Payment Notification" section: `https://m2upay.maybank2u.com.my/assets/sdk/index.html#realtime-payment-notification`
- Portal "Payment Notification Technical Requirement" screen at `https://m2upay.maybank2u.com.my` (after login; text read from the public app bundle)

Use for: RPN JSON fields, cents amount, status codes, reply JSON and reply codes, Maybank IPs, JSON format, all-transactions notification, Response URL and port, SSL certificate upload, 5-working-day change lead time.

## Browser Return Callback

- `docs/PaymentAPIReference-V1.docx` inside `M2UPayment.jar` in the Java SDK zip

Use for: `Callbackurl` redirect parameters, older UAT URL, `i` values `OT`/`DPE`/`OPR`, mandatory amount and reference matching statement.

## Account, Eligibility, Pricing, Settlement

- FAQ: `https://m2upay.maybank2u.com.my/assets/faq/index.html`
- Portal application screens (documents per business type, Payee Code page, UAT confirmation): `https://m2upay.maybank2u.com.my`
- Payee Report Portal login: `https://payee.maybank2u.com.my/payee/LogIn.do`

Use for: eligibility, application steps, documents, Test and Live Payee Codes, UAT screenshots, no setup fee, Letter of Offer pricing, next-business-day card settlement, reports, eBPG requirements and timelines, support emails, branding.
