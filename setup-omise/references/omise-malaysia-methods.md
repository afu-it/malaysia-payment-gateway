# Omise Malaysian Payment Methods and Source API

Checked 2026-10-21 against the Malaysia view of docs.omise.co. Fees are not copied here; see `https://www.omise.co/en/pricing/malaysia`.

## Source URLs

- FPX: `https://docs.omise.co/en/fpx/malaysia`
- DuitNow Online Banking/Wallets: `https://docs.omise.co/en/duitnow-obw/malaysia`
- DuitNow QR: `https://docs.omise.co/en/duitnow-qr/malaysia`
- Touch 'n Go: `https://docs.omise.co/en/rms-touch-n-go/malaysia`
- Boost: `https://docs.omise.co/en/boost/malaysia`
- GrabPay: `https://docs.omise.co/en/grabpay/malaysia`
- ShopeePay QR: `https://docs.omise.co/en/shopeepay-qr/malaysia`
- ShopeePay App Redirection: `https://docs.omise.co/en/shopeepay-jumpapp/malaysia`
- Maybank QR: `https://docs.omise.co/en/maybank-qr/malaysia`
- Atome: `https://docs.omise.co/en/atome/malaysia`
- Atome QR: `https://docs.omise.co/en/atome-qr/malaysia`
- Google Pay: `https://docs.omise.co/en/googlepay/malaysia`
- Installments (redirect): `https://docs.omise.co/en/installment-payments/malaysia`
- Source API: `https://docs.omise.co/en/sources-api/malaysia`
- Capability API: `https://docs.omise.co/en/capability-api/malaysia`
- Integrations (methods by country): `https://docs.omise.co/en/integrations/malaysia`

## Enabling methods

Each method below says: "send an email requesting this feature to support@omise.co" and accept new terms. (The FPX page prints `support@opn.ooo`, a leftover of the Opn name.) Minimum API version `2017-11-02`. Confirm what is live on the account with `GET /capability` (`payment_methods[]` with `name`, `currencies`, `card_brands`, `installment_terms`, `banks`).

Omise's "payment methods by country" table for Malaysia: Boost, DuitNow Online Banking/Wallets, DuitNow QR, FPX, GrabPay, Maybank QR, Touch 'n Go, ShopeePay, Credit and Debit Cards, Google Pay. Atome (both flows) and Maybank installments also list Malaysia on their own pages.

## Common flow for every source method

1. Client (public key) or server (secret key): `POST https://api.omise.co/sources` with `type`, `amount`, `currency=MYR` and method fields. Or pass `source[...]` inline on the charge.
2. Server: `POST /charges` with `amount`, `currency`, `source`, `return_uri` (HTTPS). Amount and currency must equal the source's.
3. Charge returns `status=pending` and `authorize_uri`. Redirect the payer.
4. Payer authorizes at the bank or wallet, then returns to `return_uri`.
5. `charge.complete` webhook; retrieve the charge and confirm `status`.
6. Test mode: open `authorize_uri` yourself and choose Successful or Failed, or use dashboard Charges > charge > Actions > Mark as Successful / Mark as Failed.

## Method summary

| Method | `type` | Flow | Required extra fields | Min | Max | Refund window | Capture |
| --- | --- | --- | --- | --- | --- | --- | --- |
| FPX | `fpx` | redirect | `bank` | `100` (RM1) | `3000000` (RM30,000) | Partial or full within six months | Auto |
| DuitNow Online Banking/Wallets | `duitnow_obw` | redirect | `bank` | `100` | `2000000` (RM20,000) | Partial or full within 180 days | Manual and automatic |
| DuitNow QR | `duitnow_qr` | redirect | none | `100` | `2000000` (RM20,000) | Partial or full within 180 days | Manual and automatic |
| Touch 'n Go | `touch_n_go` | redirect | none | `100` | `999999` (RM9,999.99) | Partial or full within 180 days | Not stated |
| Boost | `boost` | redirect | none | `100` | `499900` (RM4,999) | Partial or full within 180 days | Not stated |
| GrabPay | `grabpay` | redirect | none | `100` | `150000` (RM1,500) | Partial or full within three months | Not stated |
| ShopeePay QR | `shopeepay` | redirect | none | `100` | `499900` (RM4,999) | Partial or full within 180 days; not for off-us | Not stated |
| ShopeePay App Redirection | `shopeepay_jumpapp` | app_redirect | none | `100` | `999900` (RM9,999) | Partial or full within 180 days; not for off-us | Not stated |
| Maybank QR | `maybank_qr` | redirect | none | `100` | `2000000` (RM20,000) | Partial or full within 180 days | Not stated |
| Atome | `atome` | redirect | `phone_number`, `items`, `shipping` (required in Malaysia) | `1000` (RM10) | `500000` (RM5,000) | Partial or full within 60 days; multiple partials allowed | Manual and automatic |
| Atome QR | `atome_qr` | offline | none | `1000` (RM10) | `500000` (RM5,000) | Partial or full within 60 days | Not stated |
| Maybank installments | `installment_mbb` | redirect | `installment_term` | see table below | | Full refund only | Not stated |
| Cards | token, not a source | 3DS redirect if enabled | | `100` | `10000000` (RM100,000) | Within 365 days (JCB/AMEX/Diners 160 days) | Auto or manual |
| Google Pay | token (`tokenization[method]=googlepay`) | card flow | Google Pay token | card limits | | card rules | card rules |

Limits are per transaction (Atome FAQ). Google Pay availability for Malaysia is stated on its page; the Omise.js pre-built form table does not list Malaysia for it.

## FPX

- Payer selects a bank, is redirected to the bank page, confirms, sees a receipt page, and can return to your site.
- Get live banks: `GET /capability`, read `payment_methods[name=fpx].banks[]` (`code`, `name`, `active`). Show only `active=true`. The top-level `banks` array is for transfers, not FPX.
- PayNet requires the FPX logo on the merchant payment page (branding guideline) for direct API integrations.
- Source fields: `amount` (required), `currency` `MYR` (required), `bank` (required), `email` (optional, 6 to 50 characters).
- Failure codes: `payment_rejected`, `failed_processing`, `invalid_account`, `insufficient_fund`.
- Example charge response: `status=pending`, `authorize_uri=https://pay.omise.co/payments/pay2_test_.../authorize`, `expires_at` 7 days after creation, `source.flow=redirect`.
- Support article for internet banking refunds says you contact the customer and pay the refund yourself in advance; the FPX page says FPX charges can be refunded via the API within six months. Prefer the FPX page, confirm with Omise for your account.

### FPX bank codes (documented list)

| Code | Bank | Code | Bank |
| --- | --- | --- | --- |
| `affin` | Affin Bank | `hongleong` | Hong Leong Bank |
| `alliance` | Alliance Bank (Personal) | `hsbc` | HSBC Bank |
| `agro` | AGRONet | `kfh` | KFH |
| `ambank` | AmBank | `maybank2e` | Maybank2E |
| `islam` | Bank Islam | `maybank2u` | Maybank2U |
| `muamalat` | Bank Muamalat | `ocbc` | OCBC Bank |
| `rakyat` | Bank Rakyat | `public` | Public Bank |
| `bocm` | Bank Of China | `rhb` | RHB Bank |
| `bsn` | BSN | `sc` | Standard Chartered |
| `cimb` | CIMB Clicks | `uob` | UOB Bank |

## DuitNow Online Banking/Wallets

- Payer picks a bank or wallet, is redirected, sees a summary, receives an OTP from the wallet app, confirms, sees a slip.
- Source fields: `amount`, `currency` `MYR`, `bank` (required), `platform_type` (optional `IOS` or `ANDROID`), `type=duitnow_obw`.
- Bank codes: `affin`, `agro`, `alliance`, `ambank`, `islam`, `muamalat`, `rakyat`, `bsn`, `cimb`, `hongleong`, `hsbc`, `kfh`, `maybank2u`, `ocbc`, `public`, `rhb`, `sc`, `uob` (no `maybank2e` or `bocm`).
- Malaysia only; charges must be MYR. `return_uri` must be HTTPS.
- Failure codes: `payment_expired`, `payment_rejected`, `failed_processing`, `invalid_account`, `insufficient_fund`.

## DuitNow QR

- Redirect flow to a QR page. Desktop: scan with a bank or wallet app. Mobile: screenshot the QR and load it in the bank app.
- Source fields: `amount`, `currency` `MYR`, `platform_type` (optional), `type=duitnow_qr`.
- Failure codes: `payment_expired`, `payment_rejected`, `failed_processing`, `invalid_account`, `insufficient_fund`.

## Touch 'n Go

- Redirect to a TnG QR page (desktop scan) or to the TnG app (mobile).
- Source fields: `amount`, `currency` `MYR`, `platform_type` (optional), `type=touch_n_go`.
- Failure codes: `payment_cancelled`, `payment_expired`, `payment_rejected`, `failed_processing`, `invalid_account`, `insufficient_fund`.
- `POST /charges/{id}/expire` is supported for `touch_n_go`.

## Boost

- Mobile: "Tap and Pay" (Boost app) or "Sign in to Pay" (phone number plus OTP). Desktop: QR.
- Source fields: `amount`, `currency` `MYR`, `platform_type` (optional), `type=boost`.
- Failure codes: `payment_expired`, `payment_rejected`, `failed_processing`, `invalid_account`, `insufficient_fund`.

## GrabPay

- Redirect to GrabPay checkout, phone number and OTP, choose a Grab payment option. Malaysia supports wallet balance, PayLater (Postpaid) and PayLater (Installments); credit card inside Grab is not supported.
- Source fields: `amount`, `currency`, `type=grabpay` (page example uses SGD; use `MYR`).
- Failure codes: `payment_expired`, `payment_rejected`, `failed_processing`, `invalid_account`, `insufficient_fund`.

## ShopeePay

Two types. Malaysia supports wallet balance and SPayLater.

- `shopeepay` (QR page): source fields `amount`, `currency` (`MYR`), `type`. Charge expires 60 minutes after creation by default; expire early with `POST /charges/{id}/expire`.
- `shopeepay_jumpapp` (opens the ShopeePay app): source fields `amount`, `currency`, `type`, `platform_type` (optional). Charge expires 20 minutes after creation by default; set `expires_at` on the charge to any time up to 60 minutes; expire early with the Expire API.
- Failure codes: `payment_cancelled`, `payment_expired`, `payment_rejected`, `failed_processing`, `invalid_account`, `insufficient_fund`.
- Refunds and voids are not available for off-us transactions (paid through a mobile banking app instead of ShopeePay).

## Maybank QR

- Redirect to a QR page; desktop scan or mobile screenshot into the bank app.
- Source fields: `amount`, `currency` `MYR`, `platform_type` (optional), `type=maybank_qr`.
- Failure codes: `payment_expired`, `payment_rejected`, `failed_processing`, `invalid_account`, `insufficient_fund`.

## Atome (redirect)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `amount` | integer | yes | Sen |
| `currency` | string | yes | `MYR` |
| `type` | string | yes | `atome` |
| `phone_number` | string | yes | With country code, for example `+60123456789` |
| `items` | array | yes | Each item: `sku`, `name`, `quantity`, `amount` (conditionally required for Atome), optional `category`, `brand`, `item_uri`, `image_uri` |
| `shipping` | Address | yes in Malaysia | `country`, `city`, `postal_code`, `state`, `street1`, `street2` |
| `name`, `email` | string | no | Customer |
| `billing` | Address | no | Billing address |

- Charge expires 12 hours after creation by default; `expires_at` may be 30 seconds to 12 hours from creation, otherwise the request is rejected. Manual Expire API also works.
- Failure codes: `payment_expired`, `payment_rejected`, `failed_processing`.
- Recurring or scheduled payments are not supported.

## Atome QR (offline)

- Same fields as Atome, all optional except `amount`, `currency`, `type=atome_qr`.
- Charge has no `authorize_uri` or `return_uri`. Show `charge.source.scannable_code.image.download_uri`; the customer scans with the Atome app.
- QR expires 12 hours after creation by default; `expires_at` 30 seconds to 12 hours.
- Test: dashboard charge > Actions > mark Successful or Failed.
- Failure codes: `payment_expired`, `payment_rejected`, `failed_processing`.

## Installments (Maybank)

| Source `type` | Term (months) | Minimum amount |
| --- | --- | --- |
| `installment_mbb` | 6 | MYR 500 |
| `installment_mbb` | 12 | MYR 1,000 |
| `installment_mbb` | 18 | MYR 1,500 |
| `installment_mbb` | 24 | MYR 2,000 |

- In Malaysia, merchants absorb all interest. `zero_interest_installments` must match between source and charge if sent.
- Source fields: `type`, `amount`, `currency`, `installment_term` (integer), optional `zero_interest_installments`.
- Redirect flow via `authorize_uri`, then webhook and status check.
- Installment refunds: full refund only; merchant-absorbed interest is not returned.

## Google Pay

- Use the Google Pay API with `allowedAuthMethods=["PAN_ONLY"]`, `allowedCardNetworks` from `GET /capability`, `gateway: "omise"`, `gatewayMerchantId: <OMISE_PUBLIC_KEY>`.
- Convert the Google Pay token to an Omise card token client-side (Omise.js `createToken` with type `tokenization`, or Android SDK) using `method=googlepay`, `data=<token>`, optional `billing_name`, `billing_city`, `billing_country`, `billing_postal_code`, `billing_state`, `billing_street1`, `billing_street2`, `billing_phone_number`.
- Charge the card token like any card; 3DS settings apply. These tokens cannot be attached to customers.
- Google Pay test environment always yields card `4111 1111 1111 1111`. Live needs a Google merchant ID from the Google Pay Business Console.

## Source API

### Create a source

`POST https://api.omise.co/sources` (public or secret key).

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `amount` | integer | yes | Smallest unit |
| `currency` | string | yes | ISO 4217 |
| `type` | string | yes | One of the types listed under Source types |
| `bank` | string | conditional | `fpx` (and `duitnow_obw` per its page) |
| `email` | string | conditional | `fpx` (optional per FPX page), `econtext` |
| `phone_number` | string | conditional | `atome` |
| `name` | string | conditional | `econtext` |
| `mobile_number` | string | conditional | `truemoney` |
| `installment_term` | integer | conditional | `installment_*` |
| `zero_interest_installments` | boolean | no | `installment_*` |
| `platform_type` | string | no | `WEB` (default), `IOS`, `ANDROID` |
| `items` | array | conditional | `atome` and some installments |
| `shipping`, `billing` | object | no / conditional | `country`, `city`, `postal_code`, `state`, `street1`, `street2` |
| `ip` | string | no | Payer IP |
| `barcode`, `store_id`, `store_name`, `terminal_id` | string | conditional | `barcode_alipay` |
| `promotion_code`, `supplier_id`, `qr_settings` | various | conditional | Thai methods |

### Source object

`object=source`, `id` (`/src(_test)?_[0-9a-z]+/`), `livemode`, `location`, `type`, `flow` (`redirect`, `offline`, `app_redirect`), `amount`, `currency`, `charge_status`, `bank`, `email`, `name`, `phone_number`, `mobile_number`, `platform_type`, `installment_term`, `zero_interest_installments`, `absorption_type`, `items`, `billing`, `shipping`, `scannable_code` (barcode object with `image.download_uri`), `references`, `provider_references` (bank or wallet transaction references; returned for `duitnow_obw`, `duitnow_qr`, `fpx`, `grabpay`, `maybank_qr`, `shopeepay`, `shopeepay_jumpapp`, `touch_n_go` among others, otherwise `null`), `receipt_amount`, `discounts`, `ip`, `barcode`, `store_id`, `store_name`, `terminal_id`, `promotion_code`, `supplier_id`, `qr_settings`, `created_at`.

`GET https://api.omise.co/sources/{id}` retrieves a source (public or secret key).

### Source types (full documented enum)

`alipay`, `alipay_cn`, `alipay_hk`, `alipayplus_mpm`, `alipayplus_upm`, `atome`, `atome_qr`, `bkash`, `boost`, `dana`, `duitnow_qr`, `duitnow_obw`, `easypaisa`, `econtext`, `fpx`, `gcash`, `grabpay`, `installment_bay`, `installment_bbl`, `installment_first_choice`, `installment_kbank`, `installment_ktc`, `installment_mbb`, `installment_paynext_extra_jumpapp`, `installment_paynext_extra_qr`, `installment_scb`, `installment_ttb`, `installment_uob`, `installment_wlb_bay`, `installment_wlb_bbl`, `installment_wlb_first_choice`, `installment_wlb_kbank`, `installment_wlb_ktc`, `installment_wlb_scb`, `installment_wlb_ttb`, `installment_wlb_uob`, `kakaopay`, `maybank_qr`, `mobile_banking_ocbc`, `mobile_banking_scb`, `mobile_banking_kbank`, `mobile_banking_bbl`, `mobile_banking_bay`, `mobile_banking_ktb`, `paynow`, `paypay`, `promptpay`, `qr_code_upi`, `rabbit_linepay`, `shopeepay`, `shopeepay_jumpapp`, `touch_n_go`, `truemoney`, `truemoney_jumpapp`, `truemoney_qr`, `wechat_pay`, `wechat_pay_mpm`, `wechat_pay_upm`.

Only the Malaysian subset above applies to a Malaysia account. Alipay+, Apple Pay (Singapore only), PromptPay, TrueMoney, Thai mobile banking, PayNow, Konbini and others are for other markets.

## Plugins and Malaysian methods

Per the integrations page: WooCommerce and Magento support cards, Boost, GrabPay, Google Pay, ShopeePay, Touch 'n Go, DuitNow OBW, DuitNow QR, FPX, Maybank QR and installments; Shopify supports the same minus Google Pay; OpenCart and PrestaShop support cards only for Malaysia-relevant methods. Server libraries: Node, PHP, Python, Ruby, Go, Elixir "support all payment methods"; Java and .NET list FPX but not every Malaysian wallet.
