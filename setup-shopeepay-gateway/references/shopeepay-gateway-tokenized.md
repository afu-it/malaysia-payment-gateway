# ShopeePay Tokenized Payments: Account Linking, Link & Pay, Subscription, Auth & Capture

Sources (checked 2026-10-21): `https://product.shopeepay.com/integration/api/account-linking`, `.../link-and-pay`, `.../subscription`, `.../auth-and-capture`, and the product pages under `https://product.shopeepay.com/products/online-payments/`. Host, headers, signature, amount rules, callbacks, status check and error codes are in `shopeepay-gateway-api.md`. All calls are `POST` on the wallet host (`https://api[.uat].wallet.airpay.com.my`).

Malaysia availability (functionalities page): ShopeePay wallet and SPayLater are live; linked bank and credit/debit cards are "Coming soon".

## Account Linking

Flow:

1. Customer chooses to link ShopeePay; server calls Initiate Account Linking.
2. Redirect to `redirect_url_web` (ShopeePay linking agreement page); do not whitelist domains or IPs for it.
3. Customer agrees; returns to `return_url` with an auth code; ShopeePay also sends Notify Account Link Status.
4. Call Get Access Token (on callback `update_type = 1` or on return) and store the token securely.
5. On unlink, call Account Unlinking; after that the account cannot be queried or charged.

### Initiate Account Linking

`POST /v3/merchant-host/account/link`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Required | Max 64 |
| `return_url` | string | Required | Return after verification |
| `linking_reference_id` | string | Optional | Merchant linking id, max 64; auto-generated if omitted |
| `merchant_ext_id` | string | Required | |
| `service_name` | string | Optional | |
| `service_id` | string | Optional | |
| `linking_description` | string | Optional | |
| `phone` | string | Required | Country code, no plus (e.g. `60123456789`) |
| `platform_type` | string | Optional | e.g. `app` |
| `additional_info` | object | Optional | `merchant_device_info`, `merchant_user_account_info` (risk data; see Risk Object) |

Response: `request_id`, `errcode`, `debug_msg`, `redirect_url_web`.

Errcodes: -2, -1 (retry new link or same `linking_reference_id`), 0, 1, 2, 11, 305, 300026.

### Get Access Token

`POST /v3/merchant-host/access-token/get`

Request: `request_id` (Required), `linking_reference_id` (Required, same as link request).

Response: `request_id`, `errcode`, `debug_msg`, `access_token` (store securely; used for payments), `user_id_hash`, `linking_reference_id`, `merchant_ext_id`, `linking_status` (1 Active on success), `create_time`, `update_time`.

Errcodes: -2, -1, 0, 1, 2, 9, 14, 24, 27, 105 (invalid auth code), 150 (active linking limit), 305, 907, 300059.

### Account Unlinking

`POST /v3/merchant-host/account/unlink`

Request: `request_id`, `linking_reference_id` (both Required). Response: same fields as Get Access Token with `linking_status` = 3 (Invalid).

### Notify Account Link Status (callback to merchant)

Body: `request_id`, `linking_reference_id`, `merchant_ext_id`, `update_type`:

| update_type | Meaning |
| --- | --- |
| 1 | User authorised the linking; call Get Access Token now |
| 2 | Linked successfully |
| 3 | Token invalidated by ShopeePay |
| 4 | User unlinked |
| 5 | Merchant unlinked |

Verify the signature header as for transaction callbacks. Response format for this callback is not separately documented; reply `{"errcode":0}` as for Notify Transaction Status.

Linking status enum (Get User Info): 1 Active, 2 Inactive, 3 Invalid, 4 Pending user authorisation, 5 Authorised pending Get Access Token, 6 Expired.

### Get User Info

`POST /v3/merchant-host/user-info/get`. Request: `request_id`, `access_token`. Response: `linking_info` {`linking_reference_id`, `linking_status`, `create_time`, `update_time`, `merchant_ext_id`, `user_id_hash`}; `user_info` (empty if account deleted, banned, frozen) {`phone_number` (masked), `wallet_balance` (x100), `spaylater_info` {`available_balance` x100, `status_info`}, `coin_balance`, `kyc_passed`}. Errcodes: -2, -1, 0, 1, 2, 24, 27, 305. For display only.

### Get Coin Redemption Info

`POST /v3/merchant-host/coin/potential-redemption`. Request: `request_id`, `access_token`, optional `payment_amount` (x100). Response: `max_coin_percentage`, `potential_payment_amount` (x100), `potential_coin_amount` (x100). Errcodes: -2, -1, 0, 1, 2, 27, 305, 702, 705. Unavailable in TH/VN.

## Link & Pay

Two modes: Redirection (ShopeePay shows its own page when needed) and API-Based (merchant renders the user's methods from Get Payment Methods and passes the choice).

### Get Payment Methods (API-Based, mandatory there)

`POST /v4/merchant-host/payment-methods/get`

Request: `request_id` (Req), `access_token` (Req), `merchant_ext_id` (Req), `store_ext_id` (Req), `amount` (Optional, x100, prechecks availability), `language` (Optional: `ms`, `en`, `id`, `th`, `vi`), `additional_info` (JSON string, Conditional; Risk Object).

Response: `request_id`, `errcode`, `debug_msg`, `saved_payment_methods[]`:

- `payment_method_type` (`ewallet`, `spay_later`, `giro_bank`, `card`), `display_name`, `description`, `icon_url`, `saved_payment_method_id`, `payment_option_reference` (pass to Direct Payment).
- `payment_method_availability` {`is_available` (do not allow selection when false), `unavailable_reason`}.
- `payment_method_details`: `ewallet.available_balance`; `card` {`last_4digits`, `card_type` `credit_card`/`debit_card`, `network` `visa`/`mastercard`/`amex`/`jcb`/`discovery`}; `giro_bank.last_4digits`; `spay_later` {`credit_balance`, `compliance_notices[].content` {`content_type` `text`/`link`, `text`, `bold`, `url`}, `spaylater_options[]` {`saved_payment_method_id`, `payment_option_reference`, `display_name`, `payment_method_availability`, `loan_tenure` (months), `monthly_payment`, `payment_promo_info`, `transaction_fee`, `payable_amount`}}.
- `payment_promo_info` {`applicability` `global`/`payment_method`, `promo_type` (Promotion Type V2), `promo_text`, `savings_amount`, `fee_amount_after_waived`}, `transaction_fee` (x100, charged to user), `payable_amount` (x100).

SPayLater compliance notices must be shown on the merchant frontend.

### Direct Payment

`POST /v3/merchant-host/transaction/payment/direct`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Required | |
| `access_token` | string | Required | From Get Access Token |
| `merchant_ext_id` | string | Required | |
| `store_ext_id` | string | Required | |
| `amount` | int64 | Required | x100 |
| `currency` | string | Required | `MYR` (docs description is a copy of the amount text) |
| `return_url` | string | Conditional | Required for Link & Pay Redirection and API-Based |
| `payment_method_type` | string | Conditional | Required for API-Based |
| `payment_option_reference` | string | Conditional | From Get Payment Methods; required for API-Based |
| `expiry_time` | uint32 | Optional | Unix timestamp; default and max 1800 s after request |
| `additional_info` | JSON string | Conditional | Risk Object |

Doc gap (checked 2026-10-21): the request table does not list `payment_reference_id`, yet the response says `reference_id` is "same value as the payment_reference_id in the request" and `expiry_time` refers to it. Send a unique `payment_reference_id` and confirm with ShopeePay during UAT.

Response when verification is required: `request_id`, `errcode`, `debug_msg`, `redirect_url` (send the user there; final result comes by Notify Transaction Status).

Response when not required: `transaction` {`reference_id`, `amount`, `status`, `transaction_type` 13, `transaction_sn`, `create_time`, `update_time`, `user_id_hash`, `merchant_ext_id`, `store_ext_id`, `promo_id_applied`, `payment_channel`, `payment_method_type`}; absent on failure.

Errcodes: -2, -1, 0, 1, 2, 4, 9, 11, 14, 24, 27, 42, 43, 50, 105, 140, 1001, 1100, 1101, 1102, 1103, 1104, 1105, 1106, 1200, 1201, 1500, 1800, 1801, 1906, 1907, 1908, 300088, 300210.

## Subscription

Same endpoint `POST /v3/merchant-host/transaction/payment/direct`, charged on the due date with no channel chosen; ShopeePay tries the user's linked methods sequentially until one succeeds and pushes a notification to the customer.

Documented request: `request_id`, `access_token`, `merchant_ext_id`, `store_ext_id` (Required); `amount` (int64, marked Optional, x100); `language` (Optional); `additional_info` (object, Conditional; for recurring send `payment_initiation` = `SUBSCRIPTION` and `billing_detail`). The table appears copied from Get Payment Methods and omits `currency` and `payment_reference_id`; send them as for Direct Payment and confirm with ShopeePay (doc gap, checked 2026-10-21).

Response: `transaction` {`reference_id` (= `payment_reference_id`), `amount`, `status`, `transaction_type` 13, `transaction_sn`, `create_time`, `update_time`, `user_id_hash`, `merchant_ext_id`, `store_ext_id`, `promo_id_applied`, `payment_channel`}.

Errcodes: as Direct Payment plus 5 (processing; requery) and 1601 (invalid promotion).

Callbacks for Link & Pay and Subscription use `reference_id`, `transaction_type`, `transaction_status` (3 = success).

## Auth & Capture

Hold funds on a linked account, capture later (once), or release.

### Create Auth

`POST /v3/merchant-host/transaction/auth/create`. Fields (no Required flags shown): `request_id`, `auth_reference_id` (max 64), `merchant_ext_id`, `store_ext_id`, `access_token`, `amount` (x100), `currency`, `return_url`, `auth_expiry_time` (Unix timestamp, max 14 days ahead).

Response: `request_id`, `errcode`, `debug_msg`, `redirect_url` (PIN verification page, valid 30 minutes, only when needed), `transaction` {`reference_id`, `amount`, `status`, `transaction_type` 1000, `transaction_sn`, `create_time`, `update_time`, `user_id_hash`, `merchant_ext_id`, `store_ext_id`}.

### Capture

`POST /v3/merchant-host/transaction/auth/capture`. Must happen before the auth expires; only one capture (full or partial) per auth.

Request: `request_id`, `capture_reference_id` (max 64), `merchant_ext_id`, `store_ext_id`, `amount` (not greater than auth amount), `auth_reference_id` (all Required); `use_coin` (boolean, Optional), `promo_ids` (Optional).

Response `transaction`: `reference_id` (capture ref), `amount`, `status`, `transaction_type` (table says 1000, sample shows 1001 Payment Captured; expect 1001), `transaction_sn`, `create_time`, `update_time`, `user_id_hash`, `merchant_ext_id`, `store_ext_id`, `promo_id_applied`.

Refund a capture with the Refund API (`reference_id` = capture reference; type list shows Capture Refund = 15). Requery with `reference_id` = `capture_reference_id`.

### Reverse Auth

`POST /v3/merchant-host/transaction/auth/reverse`. Request: `request_id`, `reverse_auth_reference_id` (max 64), `merchant_ext_id`, `store_ext_id`, `auth_reference_id` (all Required). Response `transaction` with `transaction_type` 1002.

Auth, Capture and Reverse errcodes: -2, -1, 0, 1, 2, 4, 5, 9, 11, 14, 24, 27, 42, 43, 50, 105, 140, 1001, 1100, 1101, 1102, 1103, 1104, 1106, 1200, 1201, 1500, 1800, 1801, 1907, 1908, 300088, 300210.

## Risk Object (`additional_info`)

Required content depends on `payment_initiation`:

| `payment_initiation` | Fields to send |
| --- | --- |
| (settlement remarks) | `field1`, `field2` (Required), `field3` (Optional): remarks in the ShopeePay Clearing Order Report |
| `MERCHANT_QR_PROPRIETARY`, `CUSTOMER_QR_PROPRIETARY` | Device info, user account, merchant account, order details |
| `MERCHANT_QR_NATIONAL`, `CUSTOMER_QR_NATIONAL` | As above plus `qr_details` if the merchant reads the QR |
| `ONLINE_CHECKOUT` | Device info, user account, merchant account, order details, items |
| `SUBSCRIPTION` | Recurring or scheduled billing |
| `TRANSFER` | Fund transfer without an order |

Objects:

- `merchant_device_info`: `device_identifier_info` {`device_id` (merchant-generated, Required inside), `iosudid`, `imei`}; `device_hardware_info` {`os_version`, `brand`, `model`}; `risk_tags` {`is_jailbreak`, `is_emulator`, `is_remote_control_enabled`, `is_gps_modified`, `has_fake_gps_app_installed`}; `location_info` {`latitude`, `longitude`, `location`}; `device_behavior_info` {`verification_completion_method` none/biometric/pin/password/pattern/other, `device_first_login_timestamp`, `device_risk_score` high/medium/low}; `network_risk_info` {`device_ip`}; `additional_info`.
- `merchant_user_account_info`: `user_identifier_info` {`user_id` (Required), and at least one of `phone_number`, `email`, `username`}; `account_time_info.account_registration_time`; `account_risk_info.account_risk_score` (high/medium/low).
- `customized_partner_merchant_info` (payment facilitators): `merchant_basic_info` {`merchant_name`, `merchant_id`, `merchant_mcc`}; `merchant_address` {`region`,`state`,`city`,`address`,`postal_code`}; `store_basic_info` {`store_name`,`store_id`,`store_mcc`}; `store_address`.
- `order_detail`: `shipping_address`; `item_info_list[]` {`id`,`name`,`quantity`,`currency`,`amount`,`category`} (`item_info` is deprecated); `additional_info`; `description`; `qr_details` {`qr_type` static/dynamic, `qr_source` scan/upload, `national_qr_payload`}.
- `billing_detail`: `billing_id`, `billing_frequency` (weekly, monthly, quarterly, annually, others), `is_recurring`, `is_retry`.

ShopeePay says to contact its team for which risk fields your account must send.
