# toyyibPay Reference Notes

Sources read on 2026-05-24:

- Official API reference: `https://toyyibpay.com/apireference/`
- Onboarding manual PDF: `https://www.toyyibpay.com/Manual.pdf`
- DuitNow QR manual PDF: `https://toyyibpay.com/downloads/user-manual-dnqr.pdf`
- WorkDo setup guide: `https://workdo.io/documents/payment-gateway-toyyibpay/`
- Current marketing/home page: `https://www.toyyibpay.com/`
- Contact page: `https://www.toyyibpay.com/contact/`
- Pricing page: `https://www.toyyibpay.com/pricing-plans/`
- Merchant terms PDF: `https://www.toyyibpay.com/toyyibPay-tnc.pdf`
- Alternate merchant terms PDF: `https://toyyibpay.com/pdf/toyyibPay-Merchant-Terms-and-Conditions.pdf?v=2025`
- Official WordPress plugin page: `https://wordpress.org/plugins/toyyibpay-for-woocommerce/`
- Official WordPress plugin SVN readme/source: `https://plugins.svn.wordpress.org/toyyibpay-for-woocommerce/trunk/`
- Current about page: `https://www.toyyibpay.com/about/`
- Brochure PDF with social handles: `https://www.toyyibpay.com/Brochure.pdf`

Community or third-party sources found but not authoritative:

- GitHub/SDK/plugin forks and marketplace snippets may help with examples, but do not treat them as source of truth for fields, hashes, status values, or security rules unless verified against official docs.
- Package registries include unofficial or unclear-ownership packages for JavaScript, Python, PHP/Laravel, and Flutter. Audit ownership, recency, dependency health, callback security, and amount handling before using any package.

## Environments

- Production: `https://toyyibpay.com`
- Sandbox: `https://dev.toyyibpay.com`
- API path: `/index.php/api/{endpoint}` for most documented API calls.
- Official sandbox instruction: register at `https://dev.toyyibpay.com`, then replace `toyyibpay.com` with `dev.toyyibpay.com` and use sandbox credentials.

## Request Format

- Method: `POST` for documented APIs, except `getBank` sample uses a non-POST curl call.
- Content type: `multipart/form-data` or `application/x-www-form-urlencoded`.
- Keep all secret keys on server-side routes or backend jobs.

## Dashboard Setup

Onboarding manual covers:

- Account registration and login.
- Profile, Category, All Bill, Create Bill, Settlement, Payment Channel, reports, password, logout.
- Account types: personal saving, business/company current, society/organization current.
- Payment channels are managed under `My Payment Channel`.
- Categories group bills. Bills belong to a category.

WorkDo setup guide says to configure:

- Secret Key from toyyibPay account/API reference flow.
- Category Code for transactions.
- Supported currency: Malaysian Ringgit (MYR).
- Supported country: Malaysia.
- Main payment method described: local online banking via FPX.

Official WordPress plugin page says WooCommerce setup needs:

- WooCommerce installed and activated.
- WordPress Dashboard -> Plugins -> Add New -> search `toyyibPay for WooCommerce`.
- WooCommerce -> Settings -> Payments -> toyyibPay -> Manage.
- Category Code and Secret Key from toyyibPay Admin Dashboard.
- Enable gateway, choose payment channels, optionally enable DuitNow QR and admin fee settings.
- Plugin page says it supports FPX online banking, credit/debit card, DuitNow QR, split payment, HPOS, and WooCommerce Blocks checkout.
- Stable plugin version checked: `2.0.1`; WordPress tested up to `6.9`; WooCommerce tested up to `9.5`; requires PHP `7.0`.

Use plugin docs for WordPress admin steps only. For settlement/security behavior, still require callback hash verification and idempotent local fulfilment.

Official plugin source behavior checked:

- Production mode uses `https://toyyibpay.com/index.php/api/createBill` and redirects to `https://toyyibpay.com/{BillCode}`.
- Development mode uses `https://dev.toyyibpay.com/index.php/api/createBill` and redirects to `https://dev.toyyibpay.com/{BillCode}`.
- WooCommerce bill creation sends fixed price bills with `billPriceSetting=1`, `billPayorInfo=1`, `billAmount = order_total * 100`, `billExternalReferenceNo = order_id`, `billReturnUrl`, and `billCallbackUrl`.
- Callback handler computes `md5(userSecretKey . status . order_id . refno . "ok")` from POST values and compares with `hash_equals`.
- Return URL handler treats browser redirect separately and validates WooCommerce order key before showing order received/failed state.
- Plugin schedules a bill requery after checkout: 6 minutes for DuitNow QR, 3 minutes for other payments.
- Requery calls `getBillTransactions` with `billpaymentStatus=1` and completes order only when response has successful status.
- Plugin settings note sandbox test banks: `SBI Bank A` for success, `SBI Bank B` for fail, `SBI Bank C` for random possibilities, username/password `1234`/`1234`.
- Plugin settings warn customer phone number is required during WooCommerce checkout.
- Plugin settings warn DuitNow QR must be activated on the account or checkout can fail.
- Plugin settings warn DuitNow QR currently does not support Split Payment; when both are enabled and customer pays via DuitNow QR, split is ignored and merchant receives full settlement amount.
- Plugin settings say split payment works with FPX and credit card payments, with one receiver username, percentage or fixed amount settings.

Do not copy plugin logic blindly into non-WordPress apps: strengthen it with explicit amount, Bill Code, local reference, status, and idempotency checks.

## Unofficial SDK and Package Notes

Registry/package checks on 2026-05-24 found these non-official or unclear-ownership options:

- npm `toyyibpay-js-sdk`: latest `1.0.2`, last modified in 2022 registry metadata, old dependencies listed (`axios` 0.x, `qs` 6.x), GitHub repo `Akim95/toyyibpay-js-sdk`.
- PyPI `toyyibpay`: latest `0.1.0`, uploaded 2025-06-21, metadata says "Official Python SDK" but owner/project URLs are not to official toyyibPay organization; treat as unofficial until toyyibPay confirms.
- Packagist `tarsoft/toyyibpay`: Laravel package, latest `v0.3.0`, source `tarsofttech/toyyibpay-laravel`, supports Laravel Illuminate `^6` through `^11`.
- pub.dev `toyyibpay`: Flutter helper, latest `2.1.0`, repository `goutam2597/toyyibpay`, creates bills, opens WebView checkout, parses return URL, and optionally verifies via API.

Package safety checklist:

- Confirm package owner is official or acceptable for project risk.
- Check latest release date, open issues, security advisories, dependency age, and maintenance activity.
- Inspect source for server-side secret handling; never expose User Secret Key in browser/mobile apps.
- Verify callback hash formula exactly against official API reference.
- Verify `billAmount` conversion between cents and ringgit.
- Verify Return URL is not used as final settlement.
- Verify retries, duplicates, and idempotent fulfilment.
- Verify sandbox and production credentials cannot mix.
- Prefer direct server-side API integration when package quality is uncertain.

## Support, Social, and Current Public Pages

Official current site says:

- Direct support operation: 9:00am to 5:00pm.
- After working hours support via email is available 24 hours, with stated 1 to 3 hour response time.
- Online support is through official Facebook and WhatsApp.
- Footer social links include Facebook, Instagram, TikTok, and LinkedIn.
- Contact page lists Corporate Office and ToyyibTech Hub addresses.
- Terms PDF contact email: `support@toyyibpay.com`.
- Brochure/social references include `fb.me/toyyibpay`, `m.me/toyyibpay`, `@toyyibpaymy`, and `@toyyibpay`.
- Current homepage links include registration, login, API reference, manual PDF, official WordPress plugin, pricing, solutions, risk management, Shariah certificate, and privacy/terms PDFs.

Use social/contact links for support and user-facing docs only. Do not use social posts as API truth unless they link back to official documentation.

Official social links found:

- Facebook: `https://www.facebook.com/toyyibPay/`
- Instagram: `https://www.instagram.com/toyyibpaymy`
- LinkedIn: `https://www.linkedin.com/company/toyyibpay`
- TikTok: `https://www.tiktok.com/@toyyibpay`
- Messenger references: `m.me/toyyibpay`

## Pricing and Account-Dependent Facts

Pricing page checked on 2026-05-24:

- Santai FPX: RM 0.00 per transaction for B2C, RM 2.00 per transaction for B2B; eligibility says non-profit organizations; settlement says next 10 business days.
- Standard FPX: RM 1.00 per transaction for B2C, RM 2.00 per transaction for B2B; eligibility says all users; settlement says next 1-4 business days.
- Card payments: local card 1.50%, foreign card 3.5%; onboarding/yearly fees shown on pricing page; currency MYR only; settlement speed next 4 business days.
- DuitNow QR: 1.00% or RM 1.00 per transaction; eligibility subject to provider approval; bank coverage all local banks and ewallet supported by DuitNow; settlement next 2 business days.
- Pricing page says settlement speed is subject to best effort.

Pricing, fees, eligibility, approval, and settlement timing can change. Verify live page/account dashboard before quoting to users or writing production docs.

## Products and Channels

Current homepage/solutions pages mention:

- Payment Link and QR Code.
- Online support through official Facebook and WhatsApp.
- Auto settlement with daily automated procedure.
- Split Payment to divide authorized transactions between configured parties.
- Comprehensive Dashboard and reports.
- Integration through APIs and plugins.
- Payment Gateway for FPX, QRPay, debit card, and credit card.
- Direct Debit for recurring payments.
- POS Terminal listed as coming soon.
- Current homepage positions ToyyibPay as Shariah-compliant and says it supports businesses, government agencies, and NGOs.

Treat marketing product names as high level. For exact API availability, use API reference or account dashboard.

## Risk, Compliance, and Data Notes

Risk page says framework covers:

- Operational risk through continuous monitoring, internal controls, and redundancy protocols.
- Financial risk through secure fund flows, verified merchant onboarding, and compliance with payment facilitation guidelines.
- Data security risk through strict access control under PDPA standards.
- Reputational and compliance risk through audits, governance reviews, and partner due diligence.

Use these as due diligence notes. They do not replace app-level security controls, callback verification, access control, idempotency, logging, and reconciliation.

## Legal and API Use Notes

Merchant terms PDF checked on 2026-05-24:

- Latest PDF found says Last Updated: 15 January 2025.
- Terms define Platform as websites, mobile applications, APIs, and Webhooks.
- Terms define API/Webhook as technology used for integration and communication with other systems.
- Terms state users agree to comply with API documentation and guidelines.
- Terms restrict harmful, disabling, overburdening, interfering, scraping, mining, or extracting data without prior written consent.
- Terms disclaim liability for API errors, omissions, interruption, or failure.
- Terms say fees, settlement periods, and reimbursement schedules may be checked on website or by contacting toyyibPay.

Do not build scraping or unsupported API behavior into integrations.

## Main Payment Flow

1. Customer starts payment in merchant app.
2. Merchant app creates local pending payment.
3. Merchant app creates toyyibPay Bill through API.
4. toyyibPay returns `BillCode`.
5. Merchant redirects customer to bill URL, for example `https://toyyibpay.com/{BillCode}`.
6. Customer pays through selected channel.
7. toyyibPay sends server-side callback to merchant callback URL.
8. Merchant may check payment status with API.
9. Merchant settles local payment only after verification.

## Endpoints

Use production host below, or replace host with `dev.toyyibpay.com` for sandbox.

| Purpose | Endpoint |
| --- | --- |
| Create Category | `POST https://toyyibpay.com/index.php/api/createCategory` |
| Create Bill | `POST https://toyyibpay.com/index.php/api/createBill` |
| Get Bill Transactions | `POST https://toyyibpay.com/index.php/api/getBillTransactions` |
| Get Category Details | `POST https://toyyibpay.com/index.php/api/getCategoryDetails` |
| Inactive Bill | `POST https://toyyibpay.com/index.php/api/inactiveBill` |
| Create Account, enterprise only | `POST https://toyyibpay.com/index.php/api/createAccount` |
| Get Bank | `https://toyyibpay.com/index.php/api/getBank` |
| Get User Status, enterprise only | `POST https://toyyibpay.com/index.php/api/getUserStatus` |
| Get Settlement Summary | `POST https://toyyibpay.com/api/getSettlementSummary` |
| Get Settlement Summary, enterprise | `POST https://toyyibpay.com/admin/api/getSettlementSummary` |
| Check DuitNow QR Status | `POST https://toyyibpay.com/index.php/api/checkDuitNowQRStatus` |

## Create Category

Required fields:

- `catname`
- `catdescription`
- `userSecretKey`

Response contains `CategoryCode`.

Use a stable category for each product/payment grouping. Store Category Code in environment or gateway settings when the app has a fixed category.

## Create Bill

Important fields:

- `userSecretKey`: User Secret Key.
- `categoryCode`: Category Code.
- `billName`: displayed bill title; docs mention max 30 alphanumeric characters, space, and `_`.
- `billDescription`: docs mention max 100 alphanumeric characters, space, and `_`.
- `billPriceSetting`: `1` for fixed amount, `0` for dynamic payer-entered amount.
- `billPayorInfo`: `1` to require payer info, `0` for open bill without payer info.
- `billAmount`: amount in cents, for example `100` equals RM 1.00. Use `0` only for dynamic bills.
- `billReturnUrl`: browser return URL after payment completion.
- `billCallbackUrl`: callback URL for payment transaction status.
- `billExternalReferenceNo`: merchant reference. Use local payment/order ID.
- `billTo`, `billEmail`, `billPhone`: payer information.
- `billSplitPayment`: optional; `1` enables split payment, unavailable for dynamic bills.
- `billSplitPaymentArgs`: optional JSON, for example `[{"id":"johndoe","amount":"200"}]`.
- `billPaymentChannel`: `0` FPX, `1` credit card, `2` both FPX and credit card.
- `billContentEmail`: optional extra email message, limited to 1000 characters.
- `billChargeToCustomer`: optional charging behavior. Docs note charge to customer is unavailable for card payment channel.
- `billChargeToPrepaid`: optional, deducts transaction charge from prepaid account for FPX when charges are on bill owner.
- `billExpiryDate`: optional date/time, sample `17-12-2020 17:00:00`.
- `billExpiryDays`: optional 1 to 100 days. `billExpiryDate` has priority when both are set.
- `enableFPXB2B`: optional, `1` enables FPX corporate banking.
- `chargeFPXB2B`: optional, `0` charge customer, `1` charge bill owner. Default is `1` when FPX B2B is enabled.
- `enableDuitNowQR`: optional, `1` enables DuitNow QR; account must have DuitNow QR activated.
- `chargeDuitNowQR`: required when `enableDuitNowQR=1`; `0` charge bill owner, `1` charge customer.

Response contains `BillCode`. Hosted URL is `https://toyyibpay.com/{BillCode}` in production, or sandbox host in sandbox.

## Callback

Callback URL receives `POST` data. Docs state callback URL cannot be localhost.

Standard callback fields:

- `refno`: payment reference number.
- `status`: `1` success, `2` pending, `3` fail.
- `reason`: reason for status.
- `billcode`: Bill Code or permanent link.
- `order_id`: external reference from merchant, if specified.
- `amount`: payment amount received.
- `transaction_time`: transaction status datetime.
- `hash`: MD5 hash for callback verification.

Hash verification:

```text
MD5(userSecretKey + status + order_id + refno + "ok")
```

Reject callback before settlement if hash mismatches. Compare computed hash to received `hash` using a timing-safe comparison where available.

## DuitNow QR Callback

DuitNow QR callback uses similar data:

- `refno`
- `status`
- `reason`
- `billcode`
- `order_id`
- `amount`
- `status_id`
- `msg`
- `transaction_id`: toyyibPay invoice number.
- `dnqr_transaction_id`: DuitNow QR transaction ID.
- `hash`

Docs note DuitNow QR callback does not include `transaction_time` or `fpx_transaction_id`.

Identify DuitNow QR payments through `billpaymentChannel = "DuitNow QR"` in `getBillTransactions`.

## Return URL

Return URL receives `GET` parameters:

- `status_id`: `1` success, `2` pending, `3` fail.
- `billcode`
- `order_id`

Do not mark paid from Return URL alone. Use it to show a pending/success screen while server-side callback/API verification completes.

## Get Bill Transactions

Request fields:

- `billCode`
- `billpaymentStatus`: optional status filter.

Status filter descriptions:

- `1`: successful transaction.
- `2`: pending transaction.
- `3`: unsuccessful transaction.
- `4`: pending.

Response fields include:

- `billName`
- `billDescription`
- `billTo`
- `billEmail`
- `billPhone`
- `billStatus`
- `billpaymentStatus`
- `billpaymentChannel`
- `billpaymentAmount`
- `billpaymentInvoiceNo`
- `billSplitPayment`
- `billSplitPaymentArgs`
- `billpaymentSettlement`
- `billpaymentSettlementDate`
- `SettlementReferenceNo`
- `billPaymentDate`
- `billExternalReferenceNo`

Use this API to reconcile callback data before fulfilment when practical.

## Inactive Bill

Endpoint: `inactiveBill`.

Fields:

- `secretKey`: User Secret Key.
- `billCode`: Bill Code.

Possible responses include:

- `{"status":"success","result":"Bill status changed to inactive"}`
- `{"status":"failed","result":"Bill has pending transaction process"}`
- `{"status":"failed","result":"Bill is inactive"}`

Use this for order cancellation or expired checkout cleanup, but never assume inactive succeeds during pending transaction processing.

## DuitNow QR Setup

DuitNow QR manual says:

- Portal Create Bill: tick Enable DuitNow QR, then save bill.
- Portal Update Bill: All Bill -> More Info -> Update -> tick Enable DuitNow QR, then save.
- API Create Bill: send `enableDuitNowQR` and `chargeDuitNowQR`.
- `enableDuitNowQR=1` enables DuitNow QR; `0` disables.
- `chargeDuitNowQR=1` customer pays fee; `0` bill owner pays fee.

Activation check:

- `POST /index.php/api/checkDuitNowQRStatus`
- Field: `userSecretKey`
- Activated response: `{"status":"success","duitnowqr_activated":true}`
- Not activated response: `{"status":"success","duitnowqr_activated":false}`
- Error response: `{"status":"error","message":"Invalid userSecretKey"}`

## Enterprise APIs

Enterprise-only APIs include user creation, user status, bank lookup, and settlement summary.

Do not use Enterprise User Secret Key in normal merchant checkout unless the project is explicitly building an enterprise partner flow.

## Settlement Strategy

Only mark local payment paid when all are true:

- Callback hash valid.
- Local payment exists by `order_id`/`billExternalReferenceNo`.
- Provider is `toyyibpay`.
- Local `BillCode` matches callback `billcode`.
- Status is success (`1`) from callback and/or fetched transaction.
- Amount matches expected MYR amount. Normalize cents vs ringgit strings carefully.
- Duplicate callback or retry has no extra side effects.
- Fulfilment happens once after transaction recording.

Recommended database constraints:

- Unique local payment reference.
- Unique `(provider, bill_code)`.
- Unique callback event by provider reference/invoice where available.
- Idempotent fulfilment marker on order/subscription/entitlement.

## Common Failure Fixes

- Callback not received: do not use localhost; expose public HTTPS callback and ensure firewall/router allows it.
- Hash mismatch: verify concatenation order exactly, use User Secret Key for same environment, and do not trim/mutate callback values before hashing unless app stores normalized copies separately.
- Paid page shown but order pending: Return URL is not settlement; wait for callback or fetch `getBillTransactions`.
- Wrong amount marked paid: compare local expected cents to callback/fetched amount after explicit conversion.
- Sandbox/live issue: use `dev.toyyibpay.com` plus sandbox credentials; do not mix live Category Code with sandbox key.
- DuitNow QR missing: confirm account activation with `checkDuitNowQRStatus`, then include `enableDuitNowQR=1` and `chargeDuitNowQR`.
- Pending transaction not cancelled: `inactiveBill` can fail when pending transaction process exists; leave payment pending or expire locally with reconciliation.
- WooCommerce order mismatch: confirm Category Code and Secret Key match same environment, channel settings match account activation, and order completion depends on verified callback/API state, not browser return alone.
