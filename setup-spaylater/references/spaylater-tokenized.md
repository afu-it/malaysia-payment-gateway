# SPayLater Tokenized Flows (Account Linking, Link & Pay, Subscription, Auth & Capture)

Use this file when a merchant wants to show SPayLater instalment tenures on its own checkout page, charge a linked ShopeePay account, run recurring charges, or hold-then-capture. All calls use the Wallet API host (`https://api.uat.wallet.airpay.com.my` staging, `https://api.wallet.airpay.com.my` production) and the same `X-Airpay-ClientId` + `X-Airpay-Req-H` HMAC-SHA256 signing described in `spaylater-api.md`.

Sources (checked 2026-10-21):

- `https://product.shopeepay.com/integration/api/account-linking`
- `https://product.shopeepay.com/integration/api/link-and-pay`
- `https://product.shopeepay.com/integration/api/subscription`
- `https://product.shopeepay.com/integration/api/auth-and-capture`
- `https://product.shopeepay.com/products/online-payments/link-and-pay`
- `https://product.shopeepay.com/products/online-payments/subscription`
- `https://product.shopeepay.com/products/online-payments/auth-and-capture`

Availability per product and market (for example whether SPayLater can be charged through Subscription or Auth & Capture in Malaysia) is not stated. The docs only tie SPayLater explicitly to Link & Pay payment methods, `spaylater_info` in user info, and SPayLater errcode `1102`. Confirm with ShopeePay.

## 1. Account Linking

Flow: merchant calls Initiate Account Linking, redirects the user to `redirect_url_web` (ShopeePay linking agreement page), user agrees, user returns to `return_url` with an auth code, ShopeePay also sends a Notify Account Link callback. Merchant then calls Get Access Token (triggered by the callback with `update_type = 1` or by the frontend return) and stores the token securely. Unlink invalidates the token.

### 1a. Initiate Account Linking: `POST /v3/merchant-host/account/link`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Yes | Max 64. |
| `return_url` | string | Yes | Return after verification on ShopeePay. |
| `linking_reference_id` | string | No | Merchant linking ID, max 64. Auto-generated if absent. |
| `merchant_ext_id` | string | Yes | |
| `service_name` | string | No | |
| `service_id` | string | No | |
| `linking_description` | string | No | |
| `phone` | string | Yes | Country code, no plus. |
| `platform_type` | string | No | |
| `additional_info` | object | No | Custom metadata or risk data; ask ShopeePay. |

Response: `request_id`, `errcode` (string type in docs), `debug_msg`, `redirect_url_web` (do not domain-whitelist or IP-restrict). errcode: `-2`; `-1` server error (retry or reuse `linking_reference_id`); `0`; `1`; `2`; `11` duplicated; `305` invalid merchant; `300026` invalid or expired QR.

### 1b. Get Access Token: `POST /v3/merchant-host/access-token/get`

Fields: `request_id` (Yes), `linking_reference_id` (Yes). Response: `request_id`, `errcode`, `debug_msg`, `access_token` (store securely; used in later payments), `user_id_hash`, `linking_reference_id`, `merchant_ext_id`, `linking_status` (always `1` Active on success), `create_time`, `update_time`.

errcode: `-2`, `-1`, `0`, `1`, `2`, `9` banned, `14` deleted, `24` frozen, `27` not activated, `105` invalid auth code, `150` active linking count threshold reached, `305` invalid merchant, `907` unable to verify customer, `300059` bank account not linked.

### 1c. Unlink: `POST /v3/merchant-host/account/unlink`

Fields: `request_id` (Yes), `linking_reference_id` (Yes). Response: `request_id`, `errcode`, `debug_msg`, `merchant_ext_id`, `linking_status` (`3` Invalid on success), `create_time`, `update_time`, `access_token`, `user_id_hash`, `linking_reference_id`.

### 1d. Notify Account Link / Unlink Status (callback to merchant)

Body: `request_id`, `linking_reference_id`, `merchant_ext_id`, `update_type`:

- `1` user authorized the linking (merchant should now call Get Access Token)
- `2` linked successfully
- `3` token invalidated by ShopeePay
- `4` user unlinked
- `5` merchant unlinked

Verify the header signature as for other callbacks. The acknowledgement format for this callback is not documented (checked 2026-10-21); replying `{"errcode":0}` matches the transaction callback convention.

### 1e. Get User Info: `POST /v3/merchant-host/user-info/get`

Fields: `request_id` (Yes), `access_token` (Yes). Response: `request_id`, `errcode`, `debug_msg`, `linking_info` {`linking_reference_id`, `linking_status` (1 Active, 2 Inactive, 3 Invalid, 4 Pending user authorization, 5 Authorized pending Get Access Token, 6 Expired), `create_time`, `update_time`, `merchant_ext_id`, `user_id_hash`}, `user_info` (empty if account deleted, banned, frozen) {`phone_number` (masked), `wallet_balance` (x100), `spaylater_info` {`available_balance` (x100), `status_info` (int64, SPayLater status; values not documented)}, `coin_balance`, `kyc_passed` (bool)}.

`spaylater_info` is returned only if the merchant has SPayLater enabled. errcode: `-2`, `-1`, `0`, `1`, `2`, `24`, `27`, `305`.

### 1f. Get Coin Redemption Info: `POST /v3/merchant-host/coin/potential-redemption`

Fields: `request_id` (Yes), `access_token` (Yes), `payment_amount` (No, x100). Response: `max_coin_percentage`, `potential_payment_amount` (x100), `potential_coin_amount` (x100). errcode adds `702`, `705` coin redemption disabled. Temporarily unavailable in TH and VN.

## 2. Link & Pay

Two modes: Redirection (merchant shows "ShopeePay" as one option) and API-Based (merchant shows each linked method, including SPayLater tenures, on its own page). API-Based requires Get Payment Methods before Direct Payment.

### 2a. Get Payment Methods: `POST /v4/merchant-host/payment-methods/get`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Yes | |
| `access_token` | string | Yes | From Get Access Token. |
| `merchant_ext_id` | string | Yes | |
| `store_ext_id` | string | Yes | |
| `amount` | int64 | No | x100. If sent, availability and tenure pricing are prechecked for this amount. Send it for SPayLater. |
| `language` | string | No | `en` (default), `ms`, `id`, `th`, `vi`. |
| `additional_info` | JSON string | Conditional | Risk fields by scenario (see 2c). |

The sample request also sends `currency`; it is not in the field table.

Response: `request_id`, `errcode`, `debug_msg`, `saved_payment_methods[]`:

- `payment_method_type` (`ewallet`, `spay_later`, `giro_bank`, `card`), `display_name`, `description`, `icon_url`, `saved_payment_method_id`, `payment_option_reference` (pass to Direct Payment), `payment_method_availability` {`is_available`, `unavailable_reason`}, `payment_promo_info[]`, `transaction_fee`, `payable_amount`.
- `payment_method_details.spay_later` (when type is `spay_later`):
  - `credit_balance` (x100): available SPayLater credit.
  - `compliance_notices[]`: mandatory compliance copy to display on the merchant frontend; each `content` {`content_type` `text` or `link`, `text`, `bold`, `url`}.
  - `spaylater_options[]`: one per tenure, each with `saved_payment_method_id`, `payment_option_reference` (pass this to Direct Payment to charge that tenure), `display_name`, `payment_method_availability` {`is_available`, `unavailable_reason`}, `loan_tenure` (months, e.g. `6`), `monthly_payment` (x100), `payment_promo_info[]` {`applicability` `global` or `payment_method`, `promo_type` (Promotion Type V2, e.g. `INTEREST_WAIVER`), `promo_text`, `savings_amount`, `fee_amount_after_waived`}, `transaction_fee` (fee charged to the user, x100), `payable_amount` (x100).
- Other details: `ewallet.available_balance`; `card` {`last_4digits`, `card_type` `credit_card`/`debit_card`, `network` `visa`/`mastercard`/`amex`/`jcb`/`discovery`}; `giro_bank.last_4digits`.

Display rule: do not let users pick an option with `is_available: false`. Show `compliance_notices` for SPayLater.

### 2b. Direct Payment: `POST /v3/merchant-host/transaction/payment/direct`

| Field | Type | Req | Notes |
| --- | --- | --- | --- |
| `request_id` | string | Yes | |
| `payment_reference_id` | string | Yes (sample) | Merchant transaction ID. Present in samples and in the response description, missing from the field table. Send it. |
| `access_token` | string | Yes | |
| `merchant_ext_id` | string | Yes | |
| `store_ext_id` | string | Yes | |
| `amount` | int64 | Yes | x100. |
| `currency` | string | Yes | `MYR`. (The docs table repeats the amount description by mistake.) |
| `return_url` | string | Conditional | Required for Link & Pay Redirection and API-Based. |
| `payment_method_type` | string | Conditional | Required for API-Based, e.g. `spay_later`. |
| `payment_option_reference` | string | Conditional | Required for API-Based; for SPayLater use the chosen `spaylater_options[].payment_option_reference`. |
| `expiry_time` | uint32 | No | UNIX timestamp, default and max 1800 s after request. |
| `use_coin` | boolean | No (sample) | In samples; documented on Capture. |
| `additional_info` | JSON string | Conditional | Risk fields (2c). |

Response when verification is needed: `request_id`, `errcode`, `debug_msg`, `redirect_url` (send the user there; final state comes by Notify Transaction Status).

Response when no verification is needed: `request_id`, `errcode`, `debug_msg`, `transaction` {`reference_id`, `amount`, `status`, `transaction_type` (13), `transaction_sn`, `create_time`, `update_time`, `user_id_hash`, `merchant_ext_id`, `store_ext_id`, `payment_channel`, `payment_method_type`}. Not returned if the transaction fails.

errcode: `-2`, `-1`, `0`, `1`, `2`, `4`, `9`, `11`, `14`, `24`, `27`, `42`/`43` insufficient balance, `50`, `105`, `140` wallet limit, `1001` not allowed, `1100`/`1101`/`1200`/`1201`/`1500`/`1800`/`1801` internal payment module error, `1102` SPL credit limit reached, `1103` must link bank account, `1104`/`1907` ineligible channel, `1105` authentication failed, `1106` duplicated payment, `1906` transaction failed, `1908` maintenance, `300088` must complete verification, `300210` scam warning.

### 2c. `additional_info` risk fields (Link & Pay, Subscription)

JSON string. Keys: `field1`, `field2` (Required in table), `field3` (remarks for the clearing report); `payment_initiation` (Required): `MERCHANT_QR_PROPRIETARY`, `MERCHANT_QR_NATIONAL`, `CUSTOMER_QR_PROPRIETARY`, `CUSTOMER_QR_NATIONAL`, `ONLINE_CHECKOUT`, `SETTLEMENT`. For `ONLINE_CHECKOUT` send Device Info, User Account, Merchant Account, Order Details, Items.

- `merchant_device_info`: `device_identifier_info` {`device_id` (Required, merchant-generated), `iosudid`, `imei`}, `device_hardware_info` {`os_version`, `brand`, `model`}, `risk_tags` {`is_jailbreak`, `is_emulator`, `is_remote_control_enabled`, `is_gps_modified`, `has_fake_gps_app_installed`}, `location_info` {`latitude`, `longitude`, `location`}, `device_behavior_info` {`verification_completion_method` none/biometric/pin/password/pattern/other, `device_first_login_timestamp`, `device_risk_score` high/medium/low}, `network_risk_info.device_ip`, `additional_info`.
- `merchant_user_account_info`: `user_identifier_info` {`user_id` (Required), at least one of `phone_number`, `email`, `username`}, `account_time_info.account_registration_time`, `account_risk_info.account_risk_score`.
- `customized_partner_merchant_info` (payment facilitators): `merchant_basic_info` {`merchant_name`, `merchant_id`, `merchant_mcc`}, `merchant_address`, `store_basic_info` {`store_name`, `store_id`, `store_mcc`}, `store_address`.
- `order_detail`: `shipping_address`, `item_info_list[]` {id, name, quantity, currency, amount, category}, `item_info` (deprecated), `additional_info`, `description`, `qr_details` {`qr_type` static/dynamic, `qr_source` scan/upload, `national_qr_payload`}.
- `billing_detail`: `billing_id`, `billing_frequency` weekly/monthly/quarterly/annually/others, `is_recurring`, `is_retry`.

## 3. Subscription (Create Payment Order: Subscription)

Same endpoint as Direct Payment: `POST /v3/merchant-host/transaction/payment/direct`, called by the merchant when a payment is due. ShopeePay tries the user's linked methods one by one until one succeeds; the user gets a push notification.

The Subscription field table on the docs page appears copied from Get Payment Methods (lists `amount` optional and `language`). The sample request sends `request_id`, `payment_reference_id`, `merchant_ext_id`, `store_ext_id`, `access_token`, `amount`, `currency`, `use_coin`, `additional_info`. Follow the sample and confirm with ShopeePay. Response: `request_id`, `errcode`, `debug_msg`, `transaction` (as 2b). errcode list as 2b plus `5` processing and `1601` invalid promotion.

## 4. Auth & Capture

Hold funds on a linked account, capture once later (full or partial), or reverse.

### 4a. Create Auth: `POST /v3/merchant-host/transaction/auth/create`

Fields (presence not marked in docs): `request_id`, `auth_reference_id` (max 64), `merchant_ext_id`, `store_ext_id`, `access_token`, `amount` (x100), `currency`, `return_url`, `auth_expiry_time` (UNIX, max 14 days). Response: `request_id`, `errcode`, `debug_msg`, `redirect_url` (PIN verification page, valid 30 minutes, only if needed), `transaction` {`reference_id`, `amount`, `status`, `transaction_type` (1000), `transaction_sn`, `create_time`, `update_time`, `user_id_hash`, `merchant_ext_id`, `store_ext_id`}.

### 4b. Capture: `POST /v3/merchant-host/transaction/auth/capture`

Fields (all Required unless noted): `request_id`, `capture_reference_id` (max 64), `merchant_ext_id`, `store_ext_id`, `amount` (not more than the auth amount), `auth_reference_id`, `use_coin` (No), `promo_ids` (No). Only one capture per auth, before the auth expires. Response: `transaction` with `transaction_type` 1001 in the sample (the field text says 1000; Common Objects defines 1001 as Payment Captured).

### 4c. Reverse Auth: `POST /v3/merchant-host/transaction/auth/reverse`

Fields (all Required): `request_id`, `reverse_auth_reference_id` (max 64), `merchant_ext_id`, `store_ext_id`, `auth_reference_id`. Response: `transaction` (type 1002).

errcode for 4a to 4c: same list as Direct Payment (2b) plus `5` processing. Query status with Check Transaction Status using `transaction_type` 1000, 1001 or 1002 and the matching reference.
