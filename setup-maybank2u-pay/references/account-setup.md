# Maybank2u Pay Account Setup

Use this reference when explaining Maybank2u Pay registration, documents, Payee Codes, the technical requirement form, UAT sign-off, reporting, settlement, and the separate Cards Payment Gateway (Maybank eBPG). Checked 2026-10-21.

## Source URLs

- `https://m2upay.maybank2u.com.my` (merchant application portal)
- `https://m2upay.maybank2u.com.my/assets/faq/index.html`
- `https://m2upay.maybank2u.com.my/assets/sdk/index.html`
- `https://payee.maybank2u.com.my/payee/LogIn.do` (Maybank Payee Report Portal, linked from the portal)

## Product Positioning

Maybank2u Pay is a suite of Maybank payment services for businesses. The two online services are:

- Online Banking Payment Gateway: customers pay from a Maybank current or savings account through Maybank2u. This is what the SDK and this skill cover.
- Cards Payment Gateway (Maybank eBPG): Visa, Mastercard and American Express under 3D-Secure, with a hosted "Payment Gateway CheckOut Window" and a "Merchant Administration Interface". Its integration spec is not published; Maybank arranges integration after the Letter of Offer.

The portal also takes applications for POS, mPOS, QRPayBiz (DuitNow QR app), Maybank2u Biller, JomPAY, FPX and AutoDebit. Those are not API checkout integrations covered here. Service is Malaysia-only. (FAQ)

## Eligibility

- Must be a Maybank Business customer with a business current account. (FAQ)
- Eligible: sole proprietorships, partnerships, Sdn Bhd, Bhd, associations, charities, government, professional bodies, law firms, joint management bodies, clinics incorporated in Malaysia. (FAQ)
- Cards Payment Gateway additionally requires local registration, an office site (not a home address), and a website meeting Maybank's checklist (SSL of at least 128-bit, 3D Secure only, company and contact details, product details and total cost, currency shown, online receipt, refund and delivery policy, no sharing with sub-merchants, fraud mitigation, PCI-DSS if storing cards). (FAQ)

## Documents By Business Type

From the portal application form (required unless marked optional):

| Business type | Documents |
|---|---|
| Public Listed (Berhad), Private Limited (Sdn Bhd) | Company Registration Form (Form 24, 49, Section 14, 17, M&A, Form D, Form E, etc); Business License; optional: Changes in Register of Directors, Managers & Secretaries; Change of Company Name |
| Sole Proprietary, Partnership | Company Registration Form; Business License; optional: Extract of Registration of Business Name (Sarawak only); Form B (Sabah only) |
| Government | Circular of Ministry of Finance |
| Joint Management Body | Minutes of Meeting with list of Committees; Perakuan Penubuhan Badan Pengurusan Bersama |
| NGOs/Association | Company Registration Form; Business License; Minutes of Meeting |
| Professional Body: Law Firm, Other Professional Body | Company Registration Form; Business License; Valid Practising License; optional: Changes in Register of Directors |
| Professional Body: Clinic | Company Registration Form; Business License; Form B/C/F (Akta Kemudahan dan Perkhidmatan Jagaan Kesihatan Swasta); Perakuan Amalan Tahunan; optional: Changes in Register of Directors |

## Application Flow (Online Banking Payment Gateway)

1. Register and log in at `https://m2upay.maybank2u.com.my`, choose the service, describe the business, upload documents, submit. Status updates come by email and on the portal dashboard. (FAQ)
2. Once approved, a Test (UAT) Payee Code appears in the portal ("Online Banking UAT/ Live Payee Code" page). Start integrating while the Live Payee Code is processed. (FAQ, portal)
3. Submit the "Payment Notification Technical Requirement" (portal):
   - Response URL and Port Number for UAT and for Production. Example: website `www.website.com`, Response URL `www.website.com/payment_notification`. It is not a redirect URL.
   - SSL certificate for each environment (`.cer`, `.crt` or one zip). Incorrect certificate means notifications fail.
   - Developer details (type, name, email, contact number) are stored with the service record.
   - Changes to the Response URL take 5 working days.
4. Integrate and test with the SDK (playground, then UAT).
5. UAT confirmation: upload a screenshot of the payment pop-up in UAT and a screenshot of the RPN received, then confirm UAT passed with no open defects. Maybank is not responsible for failures from incomplete payee UAT. (Portal)
6. Receive the Production Payee Code; switch Payee Code and `envType` to 2; go live. (SDK docs, FAQ)

Processing time depends on how fast the documents are provided. (FAQ)

## Credentials And Env Vars

There are no API keys or secrets. Configure:

```text
M2UPAY_PAYEE_CODE=<PAYEE_CODE_FOR_THIS_ENV>
M2UPAY_ENV_TYPE=1            # 0 playground, 1 UAT, 2 production
M2UPAY_CALLBACK_URL=https://<your-domain>/m2upay/return
M2UPAY_RPN_ALLOWED_IPS=202.162.18.247   # UAT; production 202.162.17.179
```

The Response URL itself is registered in the portal, not sent in the request.

## Pricing

- Applying is free, no setup fee. (FAQ)
- Online banking and card Merchant Rate, plus any setup or annual fee for cards, are given in the Letter of Offer after approval. Pricing is not published; check the FAQ and the Letter of Offer: `https://m2upay.maybank2u.com.my/assets/faq/index.html`.

## Settlement And Reporting

- Cards: settled funds go to the merchant's current account the next business day; transactions on the Maybank eBPG Portal; settlement reports on the e-statement Merchant Portal. (FAQ)
- Online banking: daily report via the Maybank Merchant Portal (FAQ); the portal names a "Daily Payment Summary Report" and "Payment Raw Data File (in details)" in the Maybank Payee Report Portal, credentials emailed after approval. Settlement timing for online banking is not stated separately.

## Cards Payment Gateway (Maybank eBPG) Process

- Apply online, receive and sign a Letter of Offer, then Maybank contacts you to arrange website integration. (FAQ)
- Integration testing takes about 3 days; you have 6 months to complete integration and testing or the offer lapses. Notify the bank 3 days before going live. (FAQ)
- Technical integration documents are provided by Maybank, not published. Do not invent eBPG field names.

## Support

- SDK support: `maybank2upay@maybank.com` (replies next business day). (SDK docs)
- Merchant and cards enquiries: `merchantinquiry@maybank.com`. (FAQ)
- Customisation requests: by email to the contacts listed in the FAQ, subject to bank approval.
- Branding: do not modify the Maybank logo; refer to the product as "Maybank2u Pay". (FAQ)
