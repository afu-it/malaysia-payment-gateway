# iPay88 Account Setup

Use this reference for iPay88 Malaysia onboarding, credentials, testing, go-live and operations.

Source: iPay88's own Technical Specification v1.6.4.4 (11 Sep 2024), not published on an iPay88 domain; obtained from a community mirror (`https://github.com/karyamedia/ipay88`). Merchants receive the current spec from iPay88 Support; confirm against it. Brand and website facts below were checked on 2026-10-21.

## Brand and website status

- `www.ipay88.com.my` did not resolve on 2026-10-21.
- `https://www.ipay88.com/` redirects to `https://my.nttdatapay.com/en/payment-services/adaptis-e-commerce` (NTT DATA Payment Services Malaysia, "Adaptis E-Commerce"). That site has no developer section.
- The gateway itself still runs on `https://payment.ipay88.com.my`; its pages show "iPay88.com", Customer Careline `+60-3-2261 4668` (8.30 am to 6.00 pm, Mon to Fri) and `support@ipay88.com.my`.
- The spec's footer names the company as iPay88 (M) Sdn Bhd.
- iPay88's archived support page (`https://web.archive.org/web/2024/https://www.ipay88.com/support/`) gave support hours Mon to Fri, 9 am to 6 pm (closed weekends and public holidays) and a Technical Support line `+6017 677 5846`. The site is offline; confirm current contacts with NTT DATA. Bank Negara lists the licensed entity as NTT DATA eCommerce Solutions Sdn. Bhd. (formerly iPay88 (M) Sdn. Bhd.).

## Getting a merchant account

The spec does not describe the commercial sign-up. Contact iPay88 / NTT DATA Payment Services Malaysia sales through `https://my.nttdatapay.com/en/contact-us` or `support@ipay88.com.my`. The documents needed for KYC are not listed in the spec; ask during sign-up.

## Integration prerequisites (spec section 1.3 and FAQ)

1. Receive the Merchant Code and Merchant Key from iPay88.
2. Return a completed Merchant Checklist to `support@ipay88.com.my` (FAQ 2). Older specs called this "Appendix A. Merchant Checklist".
3. Give iPay88 Support the Request URL (your checkout page that posts to iPay88). It must be IP or domain based; localhost is not allowed.
4. Give iPay88 Support the server IP addresses for your ResponseURL and BackendURL so they can be whitelisted.
5. Assign a technical person before integration.
6. Ask for "Appendix I.pdf" (MYR PaymentId and currency list) and, for foreign currency, "Appendix II.pdf". Also ask for the "Sample Code to Generate HMACSHA512 Signature" folder.
7. Optional: send a logo or banner for the payment page and email (max 600 x 100 px, JPG, BMP or GIF).

## Where credentials live

- Merchant Code and Merchant Key are issued by iPay88, not generated in a dashboard (spec section 3.2). The key is re-issued only if iPay88 suspects compromise or fraud.
- Report login and password for `https://payment.ipay88.com.my/ePayment/report` are also issued by iPay88.
- To change Request URL, contact number, company name or bank account, email `support@ipay88.com.my` (FAQ 9).

## Sandbox vs live

No separate public sandbox is documented. The spec says:

- Test transactions must come from the registered Request URL.
- Use MYR 1.00 (see `ipay88-api.md` section 9 for other currencies).
- Any valid credit card can be used for testing.

Ask iPay88 Support whether your account starts in a test mode and whether test payments are reversed. Notify iPay88 Support of the intended live date at least 3 working days in advance.

## Environment variables

```text
IPAY88_MERCHANT_CODE=<MERCHANT_CODE>
IPAY88_MERCHANT_KEY=<MERCHANT_KEY>
IPAY88_ENTRY_URL=https://payment.ipay88.com.my/epayment/entry.asp
IPAY88_REQUERY_URL=https://payment.ipay88.com.my/epayment/enquiry.asp
IPAY88_WS_REQUERY_URL=https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx
IPAY88_RESPONSE_URL=https://shop.example.com/payment/ipay88/response
IPAY88_BACKEND_URL=https://shop.example.com/payment/ipay88/backend
```

Keep `IPAY88_MERCHANT_KEY` server-side only.

## Payment method activation

- Methods (cards, online banking, e-wallets) are enabled per merchant account by iPay88. Sending a `PaymentId` for a method not enabled returns "Payment not allowed".
- Per-transaction limits are set on the account; exceeding them returns "Overlimit per transaction".
- Foreign currencies use the multi-currency gateway (Appendix II).

## Settlement and payouts

Settlement timing and payout schedule are not in the spec. Confirm in your merchant agreement and the online report.

## Pricing

Do not copy fee numbers. Get current pricing from NTT DATA Payment Services Malaysia: `https://my.nttdatapay.com/en/payment-services/adaptis-e-commerce`. Pricing changes; do not hardcode it in application logic.

## Go-live checklist

1. Request signature matches the official tester page for a sample order.
2. MYR 1.00 test paid from the registered Request URL; ResponseURL and BackendURL both verified it.
3. BackendURL outputs only `RECEIVEOK`.
4. Re-query returns `00` for the test RefNo.
5. Server IPs whitelisted with iPay88; live date notified 3 working days ahead.
6. `no-reply@ipay88.com.my` whitelisted in the merchant mailbox for notification emails (not a payment trigger).
