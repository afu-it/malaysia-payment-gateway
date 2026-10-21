# 2C2P Batch Services, Reconciliation, SoftPOS, Plugins And SNAP

Entry point: `2c2p-api.md`. Checked against the official docs on 2026-10-21.

## SFTP Batch Services

Account: fill in the SFTP setup form and send it to 2C2P support; 2C2P creates demo and production SFTP accounts. The docs list server `52.76.184.174`, port 22. Upload request files to `/input`; collect response files from `/output`.

Schedule (times are GMT+7, which is one hour behind Malaysia time):

| Job | Time (GMT+7) |
| --- | --- |
| Reconciliation report drop | Before 4:00 AM |
| Authorization (simple) pickup / drop | 5:00 AM / before 6:00 AM |
| Offline authorization pickup / drop | 2:00 PM / before 6:00 AM next day |
| Tokenization and refund pickup / drop | 2:00 AM / before 3:00 AM |
| QuickPay and promo code pickup / drop | Every hour / within the next hour |

File formats (every file has one header record `H` and detail records `D`, line break record delimiter):

| Batch | File name | Delimiter | Encryption |
| --- | --- | --- | --- |
| Authorization | `auth_YYYYMMDD_fileName.csv` | `;` | GPG with the 2C2P PGP public key |
| Offline authorization | `offlineauth_YYYYMMDD_fileName.csv` | `;` | GPG |
| Card tokenization | `token_YYYYMMDD_fileName.csv` | `;` | GPG |
| Refund | `refund_YYYYMMDD_fileName.csv` | `;` | none |
| QuickPay | `quickpay_YYYYMMDD_fileName.csv` | `\|` | none |
| Promo code | `promocode_YYYYMMDD_fileName.csv` | `\|` | none |

Batch authorization and tokenization files carry full PANs (PCI DSS scope). Prefer API tokenisation unless you are PCI certified.

Record fields:

- Authorization request header: `recordType`, `version`, `merchantID`, `createDateTime`, `processDateTime`, `totalRecord`. Detail: `recordType`, `recordNo`, `uniqueTransactionCode` C 20, `desc` C 50, `amount` D 12,2, `currencyCode` N 3, `pan` N 16, `expiryMonth`, `expiryYear` N 4, `storeCardUniqueID` O, `securityCode` O, `cardholderName`, `cardholderEmail` O, `userDefined1..5` O, `ippTransaction`, `installmentPriod`, `interestType`, `recurring`, `recurringPrefix`, `recurringAmount`, `recurringAllowAccumulate`, `recurringMaxAccumulateAmount`, `recurringInterval`, `recurringCount`, `recurringChargeNextDate`, `recurringChargeOnDate` (field names spelled as in the docs).
- Authorization response header: `recordType`, `version`, `merchantID`, `completeDateTime`, `totalRecords`. Detail: `recordType`, `recordNo`, `uniqueTransactionCode`, `tranRef`, `approvalCode`, `refNumber`, `amt`, `currencyCode`, `pan`, `storeCardUniqueID`, `userDefined1..5`, `ippTransaction`, `installmentPriod`, `interestType`, `recurringUniqueID`, `respCode` C 2, `respDesc`.
- Offline authorization: same as authorization without IPP and recurring fields.
- Card tokenization request detail: `recordType`, `recordNo`, `pan`, `expiryMonth`, `expiryYear`, `cardholderName`, `userDefined1..5`. Response detail: `recordType`, `recordNo`, `pan`, `storeCardUniqueID`, `userDefined1..5`, `respCode`, `respDesc`.
- Refund request header: `recordType`, `version`, `merchantID` C 25, `requestorID` C 50, `createDateTime`, `processDateTime`, `totalRecord`. Detail: `recordType`, `recordNo`, `transactionInvoiceNo` C 50, `refundAmount` N 12 (12 digits, no decimal point), `note` C 250 O. Response detail adds `tranRef`, `respCode`, `respDesc`. Refund batch codes: `00` success, `01` mandatory field missing, `02` invalid or blank invoice number, `03` refund amount format wrong, `04` invalid transaction status, `05` invalid transaction, `06` refund rejected or over transaction amount, `07` insufficient balance, `08` over merchant amount, `09` field length mismatch, `89` database error, `99` system error.
- QuickPay request detail: `orderIdprefix`, `description`, `currency`, `amount`, `allowMultiplePayment`, `maxTransaction`, `expiry`, `userData1..5`, `promotion`, `categoryId`, `resultUrl2`, `paymentOption`, `ippInterestType`, `paymentExpiry`, `toEmails`, `ccEmails`, `bccEmails`, `emailSubject`, `emailMessage`, `toMobiles`, `smsMessage`, `request3DS`, `enableStoreCard`, `recurring` (M), recurring fields. Response detail: `qpID`, `orderIdPrefix`, `currency`, `amount`, `expiry`, `url`, `successEmails`, `successMobiles`, `failedEmails`, `failedMobiles`, `respCode`, `respDesc`.
- Promo code request detail: `merchantID` O, `promoCode` C 20, `promoDesc`, `cardType`, `binList`, `allowMasterpass`, `ippMode`, `ippProductCodeType`, `ippProductCodeList`, `limitByTransactionCount`, `maxAuthorisedTransaction`, `limitPerCard`, `maxTransactionPerCard`. Response detail: `merchantID`, `promoCode`, `respCode` C 4, `respDesc`.

Source: https://developer.2c2p.com/docs/batch-services-authorization, https://developer.2c2p.com/docs/batch-services-authorization-offline, https://developer.2c2p.com/docs/batch-services-card-tokenization, https://developer.2c2p.com/docs/batch-services-refund, https://developer.2c2p.com/docs/batch-services-quickpay, https://developer.2c2p.com/docs/batch-services-promo-code

## Reconciliation Reports

Daily CSV pushed to the merchant SFTP folder (no request needed), comma delimited. File name `Reconcile2c2p_B_v2.4_<MID>_<CURRENCY>_yyyy-MM-dd_HHmmss[_all|_ipp|_nonIpp].csv`, for example `..._MYR_...`. Header record `TYPE_TABLE = H`, detail records `TYPE_TABLE = D`.

Non-IPP columns (1 to 76): `BATCH_NO`, `LINE_NO`, `TYPE_TABLE`, `BANK_CODE`, `ACCOUNT_NO`, `PAYIN_DATE`, `VAT_NO`, `TRANSACTION_DATE`, `TRANSACTION_TIME`, `SETTLEMENT_DATE`, `SETTLEMENT_TIME`, `MERCHANT_ID`, `ORDER_ID`, `INVOICE_NO`, `ORDER_DATE`, `ORDER_TIME`, `PAYMENT_DATE`, `PAYMENT_TIME`, `PAYMENT_TYPE`, `CHANNEL_CODE`, `AGENT_CODE`, `AMOUNT`, `DISCOUNT`, `TRANSACTION_FEE`, `SERVICE_FEE`, `COMMISSION_FEE`, `VAT_SERVICE_FEE`, `COM_VAT_AMOUNT`, `TAX_INVOICE_NO`, `TAX_INVOICE_DATE`, `TOTAL_SERVICE_FEE_AMOUNT`, `TOTAL_VAT_AMOUNT`, `TOTAL_DEDUCT_AMOUNT`, `WHT_NO`, `WHT_AMOUNT`, `TOTAL_PAYIN_AMOUNT`, `SHIPPING_FEE`, `CURRENCY_CODE`, `APPROVAL_CODE`, `ECI`, `MASKED_PAN_OR_PAYMENT_REF_CODE`, `USER_DEFINED_1` to `USER_DEFINED_5`, `MDR`, `REFUND_NOTE`, `ISSUER_COUNTRY`, `CARD_BRAND`, `TRANSACTION_ID`, `MERCHANT_NAME`, `QR_REF_ID`, `CARDHOLDER_NAME`, `PRODUCT_DESCRIPTION`, `PROMOTION_CODE`, `ACQUIRER_DISCOUNT`, `ACQUIRER_DISCOUNT_CURRENCY`, `MCP_AMOUNT`, `MCP_CURRENCY`, `MCP_EXRATE`, `EWT_AMOUNT`, `CUSTOM_DATA_ONE`, `CUSTOM_DATA_TWO`, `REDEEMED_POINTS`, `REDEEM_POINTS_AMOUNT`, `VOUCHERDESCRIPTION`, `REDEEMVOUCHERAMOUNT`, `REDEEMEDVOUCHERSCOUNT`, `PAYMENTTYPE_NAME`, `BACKEND_REFERENCE`, `REFUND_CHARGEBACK_FEE`, `REFUND_CHARGEBACK_VAT`, `CARDTYPE`, `CARDLEVEL`, `AGENT_CHANNEL_CODE`. Older versions name some columns differently (for example `ORDERID_INVOICENO`, `PAYMENT_CHANNEL`, `PAYMENT_AGENT_SERVICE`).

IPP report (1 to 85): the same first 47 columns, then `IPP_AMOUNT` (monthly principal), `IPP_INTEREST_BY` (`M` merchant / `C` cardholder), `IPP_TERM` (months), `INT_RATE`, `IPP_INTEREST_AMOUNT`, `IPP_VAT`, `IPP_MDR`, `BANK_PROMO_CODE`, `BANK_PROMO_NAME`, then `REFUND_NOTE` onward as in the non-IPP report.

`PAYMENT_TYPE` (AN 3): `001` cards and wallets, `002` cash channels, `003` direct debit, `004` others, `005` refund, `006` instalment (IPP), `007` loyalty payment, `008` chargeback, `010` other charges, `011` authorize charges, `012` void charges, `013` sale / capture interchange charges, `014` refund interchange charges, `015` chargeback interchange charges, `016` loyalty refund (v2.4+), `017` refund reversal.

Version history: v1.1 `ISSUER_COUNTRY`; v1.2 `CARD_BRAND`; v1.3 `TRANSACTION_ID`, `MERCHANT_NAME`; v1.5 QR ref, cardholder, product, promo; v1.6 MCP fields; v1.7 `EWT_AMOUNT`, custom data; v1.8 loyalty / voucher and `PAYMENT_TYPE_NAME`; v1.9 payment type `008`; v2.0 / v2.1 backend reference and refund chargeback fee / VAT, 5-decimal amounts; v2.2 `CARDTYPE`, `CARDLEVEL`; v2.3 `AGENT_CHANNEL_CODE` and column renames; v2.4 type `016`.

Use the report to reconcile payouts and fees against your ledger; it does not replace per-transaction notification or inquiry.

Source: https://developer.2c2p.com/docs/batch-services-reconcile-report-info, https://developer.2c2p.com/docs/batch-services-reconcile-report-non-ipp, https://developer.2c2p.com/docs/batch-services-reconcile-report-ipp, https://developer.2c2p.com/changelog/recon-file-update-v21-v19

## SoftPOS (tap on phone, Android NFC)

Requires an NFC Android device and a merchant account with SoftPOS enabled. Four integration models:

| Model | How | Notes |
| --- | --- | --- |
| Standalone app | 2C2P SoftPOS app only | No code; no backend notifications. |
| Mobile SDK | SoftPOS SDK embedded in your Android app, `pay(paymentToken)` returns `PaymentResultResponse` | Payment Token created server-side as in `2c2p-api.md`. |
| App-to-App | Android Intents to the SoftPOS app through the `poslib` wrapper (MSA POS Lib v1.2.10 or newer) | Sale, auth, void, query. |
| Device-to-App (Pay Server) | HTTP over LAN to the SoftPOS app | `POST /api/v1/activation`, `POST /api/v1/trans` (sale, auth or void intent in `PosRequest`), `GET /api/v1/cancel`, `GET /api/v1/trans/status/{posMessageId}` (one sample shows `/api/v1/trans/{id}`). Enable Pay Server in the app under Menu > Settings. |

SoftPOS backend notification (all modes except standalone): `POST` JSON over HTTPS (TLS 1.2+) to the endpoint given at onboarding; respond HTTP 200 with `{}` within 30 seconds. Envelope: `category` (`PAYMENT`), `event` (`Sales`, `Preauth`, `Capture`, `Void`, `Refund`), `version` (for example `4.3`), `payload` (JWT; claims match the Payment Inquiry response), `invoiceNo`, `respCode`, `respDesc`. Verify the JWT signature with the key provided at onboarding, check `merchantID`, and handle unknown `version` values. The device result is not authoritative; the notification is.

Source: https://developer.2c2p.com/docs/softpos-overview, https://developer.2c2p.com/docs/softpos-notification, https://developer.2c2p.com/docs/softpos-device-to-app-overview, https://developer.2c2p.com/docs/overview-apptoapp, https://developer.2c2p.com/docs/softpos-dta-activation-api, https://developer.2c2p.com/docs/softpos-dta-transaction-api, https://developer.2c2p.com/docs/softpos-dta-void-api, https://developer.2c2p.com/docs/softpos-dta-cancel-api, https://developer.2c2p.com/docs/softpos-dta-enquiry-api, https://developer.2c2p.com/docs/sdk-softpos-initialize

## Plugins (hosted redirect, no API code)

| Platform | Supported versions | Setup |
| --- | --- | --- |
| Shopify | Basic plan and above | Shopify Admin > Settings > Payments > Additional payment methods > search 2C2P > Install > Activate. Give 2C2P your `*.myshopify.com` Shop ID at onboarding; 2C2P configures the backend. |
| WooCommerce | 2026 build: WooCommerce 8.x to 10.x (HPOS compatible); 2024 build: 2.x to 7.x | Download the plugin zip from the docs page, upload to `/wp-content/plugins/`, activate, configure under Plugins > 2C2P Redirect API for WooCommerce > Settings. |
| Magento 2 | 2.2.x, 2.3.x, 2.4.x | Copy module files, run `php bin/magento setup:upgrade`, flush cache, configure under Stores > Configuration > Sales > Payment Methods. |
| OpenCart 3 | Version 3 | Copy files, Extensions > Payments > 2C2P > enable and edit. |

Common plugin settings: Enable, Title, Merchant ID and Secret Key (from the my2C2P portal), Mode (Sandbox or Production), Stored Card Payment, 123 Payment Expiry (8 to 720 hours), Language. Plugins use the Redirect API, so settlement still depends on the backend notification reaching your store.

Source: https://developer.2c2p.com/docs/client-api, https://developer.2c2p.com/docs/shopify, https://developer.2c2p.com/docs/woocommerce, https://developer.2c2p.com/docs/magento2, https://developer.2c2p.com/docs/opencart-3

## SNAP API (Indonesia only, out of scope for Malaysia)

SNAP is Bank Indonesia's open API standard. Listed so an agent does not mistake it for a Malaysian option. Endpoints: `POST /snap/v1.0/access-token/b2b` (OAuth), `/snap/v{version}/debit/payment-host-to-host`, `/debit/status`, `/debit/cancel`, `/debit/refund`, `/transfer-va/create-va`, `/transfer-va/status`, `/qr/qr-mpm-generate`, `/qr/qr-mpm-query`, `/qr/qr-mpm-refund`, `/registration-account-binding`, `/registration-account-inquiry`. Headers `X-TIMESTAMP`, `X-CLIENT-KEY`, `X-SIGNATURE`, `X-PARTNER-ID`, `Authorization: Bearer`; OAuth signature `base64(SHA256withRSA("{MERCHANT_ID}|{X-TIMESTAMP}", merchant_private_key))`; merchant public key uploaded with key ID `snap`. Not used for MYR.

Source: https://developer.2c2p.com/docs/snap-overview, https://developer.2c2p.com/docs/snap-oauth
