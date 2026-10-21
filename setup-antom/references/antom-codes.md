# Antom Codes And Enums

Enums and payment method facts that matter to a Malaysian merchant. Checked against official docs on 2026-10-21.

## Source URLs

- Payment method enumeration values: `https://docs.antom.com/ac/pm/enumeration_values.md`
- FPX: `https://docs.antom.com/ac/antomop/fpx.md`
- Touch'n Go eWallet: `https://docs.antom.com/ac/antomop/touchngo.md`
- Boost: `https://docs.antom.com/ac/antomop/boost.md`
- GrabPay MY/PH: `https://docs.antom.com/ac/antomop/grabpay_.md`
- ShopeePay: `https://docs.antom.com/ac/antomop/shopeepay.md`
- Enable payment methods (Dashboard list): `https://docs.antom.com/ac/merchant_service/enable_payment.md`
- Amount object rules: `https://docs.antom.com/ac/ref/cc.md`
- Payment status guide: `https://docs.antom.com/ac/cashierpay/payment_status_desc.md`
- Web SDK createElement (locales): `https://docs.antom.com/ac/sdks/web_antom-createElement.md`

## Malaysian Payment Methods (One-time Payments)

| Method | `paymentMethodType` | Category | Acquirer entities | Min | Max | Refund period |
| --- | --- | --- | --- | --- | --- | --- |
| FPX | `ONLINEBANKING_FPX` | Online banking | AntomSG, AntomHK, AntomUS, AntomUK, AntomEU, AntomMY | RM1 | RM30,000 | 180 days |
| Touch'n Go eWallet | `TNG` | Digital wallet | AntomSG, AntomEU, AntomUS, AntomUK, AntomHK | RM0.05 | RM20,000 per transaction; RM25,000 per month; RM300,000 per year | 365 days |
| Boost | `BOOST` | Digital wallet | AntomSG, AntomHK, AntomEU, AntomUS, AntomUK | RM0.05 | RM5,000 per month (Tokenized Payment tab: RM50,000 per month) | 365 days |
| GrabPay MY | `GRABPAY_MY` | Digital wallet | AntomSG, AntomUS | RM0.01 | RM1,500 | 365 days |
| ShopeePay MY | `SHOPEEPAY_MY` | Digital wallet | AntomUS | RM0.01 | RM4,999 | 365 days |
| Cards | `CARD` | Card | See card pages and Dashboard list | Not documented in official docs (checked 2026-10-21) | Not documented in official docs (checked 2026-10-21) | Per card brand page |

Common to the five local methods above: processing currency MYR, redirect flow (ShopeePay also code-scanning), refund and partial refund supported, no chargeback or dispute, real-time result, default timeout 14 minutes.

Integration support:

- FPX: API-only, Hosted Checkout Page, Web Element, Mobile Element, payment links. Embedded Checkout Page is not supported. No Tokenized Payment, Subscription, or EasySafePay.
- TNG: API-only, Hosted and Embedded Checkout Page, Web and Mobile Element, payment links; Tokenized Payment (API-only); Subscription (AntomSG only); EasySafePay; Scan to Link. Plugins: WooCommerce and Shopify.
- Boost: API-only, Hosted and Embedded Checkout Page, Web and Mobile Element (no payment links); Tokenized Payment (API-only); Subscription (AntomSG, API-only). No EasySafePay or Scan to Link. Plugin: Shopify only.
- GrabPay MY: Tokenized Payment (API-only). No Subscription. `paymentResultInfo.funding` can be `WALLETS`, `BNPL_POSTPAID`, or `BNPL_INSTALLMENT`.
- DuitNow is not listed in the payment method enumeration (checked 2026-10-21).

Watch the acquirer column. Only FPX lists AntomMY (merchant entity MY). The wallets list non-Malaysian Antom entities. Confirm with Antom which entity contracts your Malaysian business.

## Tokenized And Subscription Enums For Malaysia

- Tokenized Payment: `BOOST`, `TNG`, `GRABPAY_MY`.
- Subscription Payment: `BOOST`, `TNG`.

## FPX Bank Codes

Returned by consult in `paymentOptionDetail.supportBanks` (`bankShortName`, `bankIdentifierCode`) and usable in `paymentMethod.paymentMethodMetaData.bankIdentifierCode`.

| Bank | `bankIdentifierCode` |
| --- | --- |
| Maybank | `MYM2U` |
| Bank Islam | `MYBISM` |
| RHB Bank | `MYRHB` |
| Hong Leong Bank | `MYHLB` |
| CIMB Bank | `MYCIMBCLICKS` |
| AmBank | `MYAMB` |
| Public Bank | `MYPBB` |
| Affin Bank | `MYABB` |
| Agro Bank | `MYAGB` |
| Alliance Bank | `MYABMB` |
| Bank Muamalat | `MYBMMB` |
| Bank of China | `MYBOC` |
| Bank Rakyat | `MYBKRM` |
| Bank Simpanan Nasional | `MYBSN` |
| HSBC Bank | `MYHSBC` |
| Kuwait Finance House | `MYKFH` |
| OCBC Bank | `MYOCBC` |
| Standard Chartered Bank | `MYSCB` |
| UOB Bank | `MYUOB` |

## Card Brands (GLOBAL region)

| Brand | Enum |
| --- | --- |
| Visa | `VISA` |
| Mastercard | `MASTERCARD` |
| American Express | `AMEX` |
| Discover | `DISCOVER` |
| Diners | `DINERS` |
| UnionPay | `CUP` |
| JCB | `JCB` |

Used in consult `supportCardBrands.cardBrand` and pay `paymentMethod.paymentMethodMetaData.selectedCardBrand`.

## Other Wallets A Malaysian Merchant May Accept

Cross-border wallets in the enum include `ALIPAY_CN`, `ALIPAY_HK`, `KAKAOPAY`, `GCASH`, `DANA`, `TRUEMONEY`, plus `APPLEPAY` and `GOOGLEPAY`. Availability depends on the acquirer entity and what is enabled in the Dashboard.

## Status And Type Enums

| Field | Values |
| --- | --- |
| `result.resultStatus` | `S`, `F`, `U` |
| inquiryPayment `paymentStatus` | `SUCCESS`, `FAIL`, `PROCESSING`, `CANCELLED`, `PENDING` |
| notifyPayment `notifyType` | `PAYMENT_RESULT`, `PAYMENT_PENDING`, `UPDATE_AMOUNT_RESULT` |
| notifyCapture `notifyType` | `CAPTURE_RESULT` |
| notifyRefund `notifyType` | `REFUND_RESULT` |
| `refundStatus` | `SUCCESS`, `PROCESSING` (inquiryRefund only), `FAIL` |
| `transactions.transactionType` | `REFUND`, `CAPTURE`, `INCREMENTAL` |
| `env.terminalType` | `WEB`, `WAP`, `APP`, `MINI_APP` |
| `env.osType` | `IOS`, `ANDROID` |
| `productCode` | `CASHIER_PAYMENT` (One-time Payments) |
| `productScene` | `CHECKOUT_PAYMENT`, `ELEMENT_PAYMENT` |
| `paymentFactor.captureMode` | `AUTOMATIC` (default), `MANUAL` |
| `paymentFactor.presentmentMode` (consult) | `TILE`, `UNIFIED` |
| `tokenizeMode` | `ENABLED`, `DISABLED`, `ASKFORCONSENT` |
| `interactionType` | `QR`, `REDIRECT` |
| `orderCodeForm.codeDetails.displayType` | `TEXT`, `SMALLIMAGE`, `MIDDLEIMAGE`, `BIGIMAGE` |
| consult `paymentMethodCategory` | `ALIPAY_PLUS`, `WALLET`, `CARD`, `ONLINE_BANKING`, `BANK_TRANSFER`, `OTC`, `MOBILE_BANKING_APP`, `BUY_NOW_PAY_LATER` |

## Currency

- MYR: 2 decimal places. `"100"` is RM1.00.
- Minimum payment and refund amounts differ per method; use the method page.

## Locales

- Web SDK `locale` includes `en_US` (default) and `ms_MY` (Malay), plus `zh_CN`, `zh_HK`, `in_ID`, `th_TH`, `vi_VN`, `tl_PH`, and others. Unsupported values fall back to English.
- pay and createPaymentSession accept a `locale` field (max 8). See the field tables in `antom-checkout.md`.
