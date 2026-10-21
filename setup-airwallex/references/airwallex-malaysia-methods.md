# Airwallex Payment Methods for Malaysia

Checked 2026-10-21 against each official payment method page. Limits, timelines and availability change; confirm in the web app (**Payments > Payment methods**) before quoting them to a merchant. Pricing is not copied here; see `account-setup.md`.

## Source URLs

- Payment methods overview: `https://www.airwallex.com/docs/payments/payment-methods`
- FPX: `https://www.airwallex.com/docs/payments/payment-methods/apac/fpx` and `.../apac/fpx/desktopmobile-website-browser`
- DuitNow: `https://www.airwallex.com/docs/payments/payment-methods/apac/duitnow` and `.../apac/duitnow/desktopmobile-website-browser`
- Touch 'n Go: `https://www.airwallex.com/docs/payments/payment-methods/apac/touch-'n-go` and its sub-pages (QR, mobile web, mobile app, recurring)
- Boost: `https://www.airwallex.com/docs/payments/payment-methods/apac/boost` and `.../boost/desktopmobile-website-browser`
- GrabPay: `https://www.airwallex.com/docs/payments/payment-methods/apac/grabpay` and `.../grabpay/desktopmobile-website-browser-grabpay`
- ShopeePay (beta): `https://www.airwallex.com/docs/payments/payment-methods/apac/shopeepay-beta` and sub-pages
- SPayLater (beta): `https://www.airwallex.com/docs/payments/payment-methods/apac/spaylater-beta`
- Atome: `https://www.airwallex.com/docs/payments/payment-methods/apac/atome` and `.../atome/desktopmobile-website-browser`
- 7-Eleven Malaysia: `https://www.airwallex.com/docs/payments/payment-methods/apac/7-eleven-malaysia`
- Cards: `https://www.airwallex.com/docs/payments/payment-methods/global/cards`
- Apple Pay: `https://www.airwallex.com/docs/payments/payment-methods/global/apple-pay` and `.../apple-pay/enable-apple-pay`
- Google Pay: `https://www.airwallex.com/docs/payments/payment-methods/global/google-paytm`
- Onboarding requirements: `https://www.airwallex.com/docs/payments/payment-methods/payment-method-onboarding-requirement`
- Config API: `https://www.airwallex.com/docs/api/payments/config`

## Properties at a glance (MYR)

| Method | API `type` | Min / max (MYR) | Session timeout | Settlement | Refunds | Chargebacks | Recurring | Hold | Onboarding |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| FPX (online banking) | `fpx` | 1.01 / 2,000 (and bank limit) | Bank dependent | T+3 business days | Full and partial within 180 days | No | No | No | 15-20 business days |
| DuitNow QR | `duit_now` | 1.00 / 1,100 (higher on request) | 5 minutes | T+2 business days | Full and partial within 180 days | No | No | No | 7-14 business days |
| Touch 'n Go eWallet | `tng` | 0.10 / 20,000 | 10 minutes | T+1 business days | Full and partial within 1 year | Yes | Yes | No | 7-14 business days |
| Boost | `boost` | 1.00 / 2,000 | 3 hours | T+3 business days | Full and partial within 180 days | No | No | No | 7-14 business days |
| GrabPay (MY) | `grabpay` | 1.00 / 9,999 (PayLater by Grab 6,000; PayLater only for GrabPay SG today) | 10 minutes | T+3 business days | Full and partial within 90 days | No | No | No | 5-10 business days |
| ShopeePay (beta) | `shopee_pay` | varies / 9,999 | 15 minutes | T+2 business days (MY) | Full and partial within 90 days | Yes | No | No | 5-10 business days |
| SPayLater (beta, BNPL) | `spaylater` | varies / 9,000 | 10 minutes | T+2 business days (MY), subject to reserves | Full and partial within 90 days | Yes | No | No | 5-10 business days |
| Atome (BNPL) | `atome` | 10 / up to credit limit | 12 hours | T+4 business days, subject to reserves; MY settles in USD or default currency | Full and partial within 60 days | No | No | No | 1-2 weeks |
| 7-Eleven Malaysia (cash) | `seven_eleven` | 1.10 / 2,500 | 48 hours | T+12 business days, subject to reserves | No | No | No | No | Up to 8 business days |
| Visa / Mastercard (MY entity) | `card` | n/a | n/a | 3 business days after capture; MYR like-for-like | Yes | Yes | Yes | Yes (7 days, 30 with pre-auth) | Instant |
| Apple Pay / Google Pay | `applepay` / `googlepay` | n/a | n/a | As cards | Yes | Yes | Yes (card rules) | Yes | Enable in **Payments > Settings** |

Notes from the method pages:

- FPX, DuitNow, Boost, 7-Eleven: settlements below 100.00 USD may be delayed until open settlements reach 100.00 USD.
- Only businesses registered in MY settle in MYR for Boost, GrabPay, ShopeePay, SPayLater; others settle in USD or the default settlement currency.
- FPX refunds can fail when there are not enough same-day transactions to cover the refund; retry another day.
- GrabPay, ShopeePay and SPayLater refunds need equivalent or higher transaction volume.
- DuitNow: CIMB bank is only available for Malaysian entity merchants. The page lists availability for businesses registered in HK, AU, SG, UK, EU, US; confirm availability for a Malaysian entity with Airwallex.
- FPX statement descriptor: one of "MOLPay", "MOLP", "MOLPay EC", "NetBuilder EC". DuitNow descriptor: your registered business name.
- SPayLater options are shown only in the shopper's Shopee app, not mobile or desktop browser.
- Cards for a MY-registered business: Visa and Mastercard process all currencies; like-for-like settlement for Visa includes MYR, USD, HKD, EUR, GBP, AUD, SGD, JPY, CAD, CHF, NZD, THB; Mastercard the same without THB; other currencies settle to your default currency. Amex, JCB, UnionPay availability for MY-registered businesses is not listed on the cards page.
- All of FPX, DuitNow, TNG, Boost, GrabPay, ShopeePay, Atome, 7-Eleven work with HPP, Drop-in, Mobile SDK, Native API and Payment Links (ShopeePay is not listed for Embedded Elements). Subscription APIs: none of them.

## Checking what is enabled

- `GET /api/v1/pa/config/payment_method_types?transaction_currency=MYR&country_code=MY&active=true` lists activated methods with `name`, `transaction_mode` (`oneoff` or `recurring`), `flows` (`qrcode`, `mobile_web`, `mobile_app`), `transaction_currencies`.
- `GET /api/v1/pa/config/banks?payment_method_type=fpx` lists FPX banks (`bank_name`, `display_name`, `resources.logo_url`). Airwallex recommends fetching dynamically rather than hardcoding.
- Sandbox defaults enable cards, Apple Pay, Google Pay, WeChat Pay, Alipay, UnionPay, Airwallex Pay. Malaysian local methods must be requested in the sandbox web app or via support.
- Confirming with a method that is not activated returns `payment_method_not_allowed`.

## FPX bank codes (`payment_method.fpx.bank_name`)

| `bank_name` | Bank |
| --- | --- |
| `affin` | Affin Bank |
| `agrobank` | Agrobank |
| `alliance` | Alliance Bank Malaysia |
| `ambank` | AmBank |
| `bank_islam` | Bank Islam Malaysia |
| `muamalat` | Bank Muamalat Malaysia |
| `bank_of_china` | Bank of China (guide table only) |
| `rakyat` | Bank Rakyat |
| `bsn` | Bank Simpanan Nasional |
| `cimb_clicks` | CIMB Clicks |
| `hong_leong` | Hong Leong Bank |
| `hsbc` | HSBC |
| `kfh` | Kuwait Finance House |
| `maybank2u` | Maybank2u |
| `mbsb` | MBSB Bank (guide table only) |
| `ocbc` | OCBC Bank |
| `public_bank` | Public Bank Berhad |
| `rhb` | RHB Bank |
| `standard_chartered` | Standard Chartered |
| `uob` | United Overseas Bank |

The confirm API reference enum lists 18 values (without `bank_of_china` and `mbsb`); the FPX guide table lists 20. Use the banks API at runtime.

## Native API confirm payloads (server side)

All examples: `POST /api/v1/pa/payment_intents/{id}/confirm` with `Authorization: Bearer <token>`, after creating the intent with `currency: "MYR"` and a `return_url`.

FPX (`bank_name` required; shopper fields optional):

```json
{ "request_id": "<UUID>",
  "payment_method": { "type": "fpx",
    "fpx": { "bank_name": "maybank2u", "shopper_name": "<NAME>", "shopper_email": "<EMAIL>", "shopper_phone": "<PHONE>" } } }
```

Response: `status: "REQUIRES_CUSTOMER_ACTION"`, `next_action: { "type": "redirect", "method": "GET", "url": "..." }`. Redirect the shopper; result arrives by webhook.

DuitNow QR:

```json
{ "request_id": "<UUID>", "payment_method": { "type": "duit_now", "duit_now": { "shopper_name": "<NAME>" } } }
```

Response `next_action.type: "redirect"`; the shopper sees the DuitNow QR page and scans it within 5 minutes.

Touch 'n Go (`flow` required: `qrcode`, `mobile_web`, `mobile_app`; `os_type` `ios` or `android` required for the mobile flows):

```json
{ "request_id": "<UUID>", "payment_method": { "type": "tng", "tng": { "flow": "qrcode" } } }
```

- `qrcode` returns `next_action.type: "render_qrcode"` with `qrcode` text (render it yourself) and a `url`.
- `mobile_web` returns a `redirect` URL. In an app WebView add header `Referer: https://api.airwallex.com` when loading `next_action.url` or the TNG app may not open.
- `mobile_app` returns a redirect URL to open the TNG app; set `return_url` to your app scheme.
- TNG guides suggest polling retrieve after the shopper returns to `return_url`, in addition to webhooks.

Boost:

```json
{ "request_id": "<UUID>", "payment_method": { "type": "boost", "boost": { "shopper_name": "<NAME>", "shopper_email": "", "shopper_phone": "" } } }
```

GrabPay (API reference field: `grabpay.shopper_name`; the guide example also sends `"flow": "qrcode" | "mobile_web" | "mobile_app"`):

```json
{ "request_id": "<UUID>", "payment_method": { "type": "grabpay", "grabpay": { "flow": "qrcode" } } }
```

ShopeePay (API reference `type: "shopee_pay"` with `shopee_pay.shopper_name/email/phone`; guide examples use `"type": "shopeepay"` with `flow` `qrcode`, `mobile_web`, or `mobile_app`). Test which form your API version accepts in sandbox.

Atome (`shopper_phone` in E.164):

```json
{ "request_id": "<UUID>", "payment_method": { "type": "atome", "atome": { "shopper_phone": "+60123456789" } } }
```

7-Eleven Malaysia:

```json
{ "request_id": "<UUID>", "payment_method": { "type": "seven_eleven", "seven_eleven": { "shopper_name": "<NAME>", "shopper_email": "<EMAIL>", "shopper_phone": "<PHONE>" } } }
```

Card (Native API, PCI-DSS ROC only): `"payment_method": { "type": "card", "card": { "number", "expiry_month", "expiry_year", "cvc", "name", "billing" } }`, optional `payment_method_options.card.auto_capture`, `three_ds_action`, and `device_data`. See `airwallex-payment-intents.md`.

## Apple Pay and Google Pay

- Enable in **Payments > Settings** (account must be enabled for online payments). Accept Apple's terms.
- Web: Airwallex handles Apple merchant validation and decryption. HPP, Payment Links and Shopify need no domain step. For Drop-in/Elements on your own domain, register the domain and host the Apple file at `https://<DOMAIN>/.well-known/apple-developer-merchantid-domain-association` with `content-type: application/octet-stream`. API: `POST /api/v1/pa/config/applepay/registered_domains/add_items`.
- Native iOS: upload an Apple payment processing certificate created from the CSR in **Payments > Settings > iOS certificates**.
- Google Pay is available on HPP, Drop-in and the Google Pay Element once enabled.

## Website requirements before activation

Payment methods are only activated if your site shows: business name, registration number and contact details; content matching your stated industry; Terms, shipping and refund, and privacy policies; a checkout with clear pricing; no redirects to external sites. Visa and Mastercard also require the governing law in your terms to match your registered country.

## Config API field tables
### List all Available Payment Method Types

`GET /api/v1/pa/config/payment_method_types`

Retrieve configuration for available payment method types with different transaction modes. Note that the payment method types can be duplicate when it supports different transaction modes. A transaction mode refer to how merchant trigger transactions. It can be one-off and recurring. For oneoff, refer to Payment Intent API. For recurring, refer to Payment Consent API.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `active` | boolean | no | Indicate whether the payment method type is active. |
| `country_code` | string | no | The supported country code of the shopper. |
| `page_num` | integer | no | Page number starting from 0. |
| `page_size` | integer | no | Number of payment method types to be listed per page. Default value is 100. |
| `transaction_currency` | string | no | The supported transaction currency. `transaction_currency` is required when `country_code` is given. |
| `transaction_mode` | string | no | The supported transaction mode. One of `oneoff`, `recurring`. Values: `oneoff`, `recurring` |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `has_more` | boolean | no | A flag which identifies whether there are more results. |
| `items` | array[object] | no | List items. |
| `items.active` | boolean | no | Indicate whether the payment method type is active. |
| `items.flows` | array[string] | no | The supported flows for the payment method type and the transaction mode. One of `qrcode`, `mobile_web`, `mobile_app`. Values: `qrcode`, `mobile_web`, `mobile_app` |
| `items.name` | string | no | The type of payment method. Can be `card`, `wechatpay`, `alipaycn`, `alipayhk`, `kakaopay`, `tng`, `truemoney`, `dana`, `gcash`, etc. |
| `items.transaction_currencies` | array[string] | no | The supported transaction currencies for the payment method type and the transaction mode. |
| `items.transaction_mode` | string | no | Indicate in which mode you trigger transactions with the payment method type. One of `oneoff`, `recurring`. Values: `oneoff`, `recurring` |

### List all Available Bank Names

`GET /api/v1/pa/config/banks`

Some payment method types such as `online_banking` require the `bank_name` to be filled when confirming a Payment Intent. You may call this API to retrieve a list of available banks and let the shopper choose one before confirming a Payment Intent. Refer to confirm.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `country_code` | string | no | Country code to filter the available banks. Use the two-character ISO Standard Country Codes. |
| `page_num` | integer | no | Page number starting from 0. |
| `page_size` | integer | no | Number of banks to be listed per page. Default value is 100. |
| `payment_method_type` | string | yes | The payment method type to find the available banks. For other payment methods that don't require `bank_name`, an empty list will be returned. |

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `has_more` | boolean | no | A flag which identifies whether there are more results. |
| `items` | array[object] | no | List items. |
| `items.bank_name` | string | yes | The code name of the bank. Pass it to confirm Payment Intent if `bank_name` is needed. |
| `items.display_name` | string | yes | The bank name for display. |
| `items.resources` | object | yes | The resources provided to render your bank list. |

### List all Apple Pay Domains

`GET /api/v1/pa/config/applepay/registered_domains`

Get all the web domains registered in Apple Pay.

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `items` | array[string] | no | All registered domains. |

### Add Apple Pay Domains

`POST /api/v1/pa/config/applepay/registered_domains/add_items`

Register one or more web domains to Apple Pay.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `items` | array[string] | yes | A list of domains to be registered. |

### Remove Apple Pay Domains

`POST /api/v1/pa/config/applepay/registered_domains/remove_items`

Remove one or more registered web domains from Apple Pay.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `items` | array[string] | yes | A list of domains to be removed. |
| `reason` | string | no | The reason for removing these domains. Maximum length is 500. |

### Retrieve the Current Reserve Plan

`GET /api/v1/pa/config/reserve_plan`

Retrieves reserve plan configuration details for your Airwallex account.

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `delayed` | object | no | `DELAYED` reserve plan. |
| `delayed_rolling` | object | no | `DELAYED_ROLLING` reserve plan. |
| `rolling` | object | no | `ROLLING` reserve plan. |
| `type` | string | no | The type of the reserve plan. Possible values: -   `DELAYED`: All funds will settle to your wallet `delayed.delay_in_days` days after transaction. -   `ROLLING`: `rolling.rolling_percentage` percent of funds will be held for `rolling.rol... Values: `DELAYED`, `delayed.delay_in_days`, `ROLLING`, `rolling.rolling_percentage`, `rolling.rolling_window_in_days`, `DELAYED_ROLLING` |

Documented errors: `account_not_authorised_for_operation`; `credentials_invalid`, `credentials_expired`; `internal_server_error`; `resource_not_found`

### List all Convertible Shopper Currencies

`GET /api/v1/pa/config/convertible_shopper_currencies`

Retrieves a list of shopper currencies enabled for creating a conversion quote, which can be used as selectable options for payment currency conversion.

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `items` | array[string] | no | A list of shopper currencies available for currency conversion. Each currency is represented by a 3-letter ISO-4217 code. |

### Retrieve the BIN Info

`GET /api/v1/pa/reference/bin/lookup`

Retrieve the BIN info using the PAN passed via header param called `x-pan`.

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `card_brand` | string | yes | One of `visa`, `mastercard`, `union pay`, `american express`. Values: `visa`, `mastercard`, `union pay`, `american express` |
| `card_type` | string | yes | - Visa: One of `credit`, `debit`, `prepaid`, `charge`, `deferred_debit`, `not_applicable`. - Mastercard: One of `debit`, `prepaid`, `credit`, `non_mastercard`. - Union Pay: One of `debit`, `prepaid`, `credit`. - American Express: One of ... Values: `credit`, `debit`, `prepaid`, `charge`, `deferred_debit`, `not_applicable`, `non_mastercard`, `consumer`, `corporate`, `small_business`, `debit, prepaid`, `prepaid_reloadable` |
| `commercial_card` | boolean | yes | A boolean field referring whether the card is commercial or not. |
| `issuer_country_code` | string | no | Issuer country code. |
| `issuer_name` | string | no | Issuer name. |
| `product_code` | string | no | - Visa: it is `Product ID` description. For more details please refer to base-ii-clearing-data-codes Product ID Values section. - Mastercard: it will be `IP0040T1.Card_Program_ID`. - Union Pay: it will be one of `UNKNOWN`, `UNION_PAY_CLA... Values: `UNKNOWN`, `UNION_PAY_CLASSIC`, `RESERVED_2`, `UNION_PAY_GOLD`, `UNION_PAY_PLATINUM`, `UNION_PAY_DIAMOND`, `RESERVED_6` |
| `product_description` | string | no | Description for `product_code`. |
| `product_subtype_code` | string | no | - Visa: it will be production `sub-type id` description. For more details please refer to base-ii-clearing-ep-release-4-reports, 'EP-302: Account Range Table (ARDEF) Field Descriptions' field `SU TY`. - Mastercard: it will be `IP0040T1.... |
| `product_subtype_description` | string | no | Description for `product_subtype_code`. |

