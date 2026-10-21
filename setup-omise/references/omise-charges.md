# Omise Charges, Cards, Omise.js and 3-D Secure

Checked 2026-10-21 against the Malaysia view of docs.omise.co.

## Source URLs

- Charge API: `https://docs.omise.co/en/charges-api/malaysia`
- Token API: `https://docs.omise.co/en/tokens-api/malaysia`
- Credit card payments: `https://docs.omise.co/en/charging-cards/malaysia`
- Collecting cards: `https://docs.omise.co/en/collecting-card-information/malaysia`
- Omise.js: `https://docs.omise.co/en/omise-js/malaysia`
- 3-D Secure: `https://docs.omise.co/en/3d-secure/malaysia`
- How to implement 3-D Secure: `https://docs.omise.co/en/how-to-implement-3-D-Secure/malaysia`
- Capture: `https://docs.omise.co/en/capture/malaysia`
- Authorize and hold: `https://docs.omise.co/en/does-omise-support-authorize-and-hold-only/malaysia`
- Merchant advice: `https://docs.omise.co/en/merchant-advice/malaysia`
- Security best practices: `https://docs.omise.co/en/security-best-practices/malaysia`
- Google Pay: `https://docs.omise.co/en/googlepay/malaysia`

## Create a charge

`POST https://api.omise.co/charges` (secret key). Token creation happens on the client; charge creation must happen on the server.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `amount` | integer | yes | Smallest unit. If charging a source, must equal the source amount. `0` triggers a 3DS card validation without payment (3DS must be enabled on the account) |
| `currency` | string | yes | `MYR`. Must equal the source currency when charging a source |
| `card` | string | conditional | Unused token id (`tokn_...`), or a card id (`card_...`) together with `customer`. Do not send `customer` with a token |
| `customer` | string or object | conditional | Customer id with at least one card (default card used unless `card` given). Can be an object to create a customer inline: `customer[card]`, `customer[email]`, `customer[description]`, `customer[metadata]`, or `customer[id]` + `customer[card]` to add a card |
| `source` | string or object | conditional | Source id (`src_...`) or inline source object (`source[type]=fpx`, `source[bank]=maybank2u`). Required if `card` and `customer` are absent |
| `return_uri` | string | conditional | Where the payer lands after 3DS or any redirect authorization. Required when the account uses 3DS or the method redirects. For app-switch methods inside a mobile app, use a deep link or app link |
| `capture` | boolean | no | Default `true`. `false` authorizes only. Valid for cards (and PayPay, Rabbit LINE Pay). Method pages say DuitNow OBW, DuitNow QR and Atome support manual and automatic capture |
| `authorization_type` | string | no | `final_auth` (default for manual capture) or `pre_auth` (single partial capture, Omise Thailand contracts only) |
| `description` | string | recommended | Helps fraud analysis |
| `ip` | string | recommended | Customer IP (IPv4 or IPv6) for fraud analysis |
| `metadata` | object | no | Custom key/values, max field length 15,000 characters. Put your order id here |
| `ref_id` | string | no | Your idempotency reference; each value usable once per account |
| `expires_at` | string | no | ISO 8601 UTC expiry. Supported by `atome`, `atome_qr`, `shopeepay`, `shopeepay_jumpapp` (and other markets' methods); list not exhaustive |
| `webhook_endpoints` | array | no | Up to 2 HTTPS URLs (not localhost, not an IP, not a numeric-only host). Events for this charge and its refunds go there instead of the account webhook |
| `first_charge` | string | no | Charge id of the first charge in a recurring cycle, to improve approval |
| `recurring_reason` | string | no | Credential-on-file reason: blank, `unscheduled`, `standing_order`, `subscription`, `installment`, `partial_shipment`, `delayed_charge`, `no_show`, `resubmission` |
| `transaction_indicator` | string | no | `MIT` (merchant initiated) or `CIT` (customer initiated) |
| `zero_interest_installments` | boolean | no | Merchant absorbs installment interest; must match the source value |
| `platform_fee` | object | no | `fixed` and/or `percentage` (platform setups) |
| `linked_account` | string | no | Direct debit (Thailand) |
| `sub_merchant_country` | string | conditional | Two-letter ISO 3166 (sub-merchant setups) |

```bash
# Card (token from Omise.js)
curl https://api.omise.co/charges -u "$OMISE_SECRET_KEY:" \
  -d "amount=150000" -d "currency=MYR" -d "card=tokn_test_xxx" \
  -d "return_uri=https://shop.example.my/orders/1001/return" \
  -d "description=Order 1001" -d "ip=203.0.113.1" \
  -d "metadata[order_id]=1001" -d "ref_id=order-1001-attempt-1"

# FPX, source created inline
curl https://api.omise.co/charges -u "$OMISE_SECRET_KEY:" \
  -d "amount=400000" -d "currency=MYR" \
  -d "return_uri=https://shop.example.my/orders/1001/return" \
  -d "source[type]=fpx" -d "source[bank]=maybank2u"
```

### Charge failure codes (Charge API)

| `failure_code` | Meaning |
| --- | --- |
| `confirmed_amount_mismatch` | Final amount from the payment channel does not match the charge |
| `failed_fraud_check` | Card marked as fraudulent |
| `failed_processing` | General processing failure |
| `insufficient_balance` / `insufficient_fund` | Insufficient funds or credit limit reached |
| `invalid_account_number` / `invalid_account` | No valid account for the payment method |
| `payment_cancelled` | Payer cancelled |
| `payment_rejected` | Issuer rejected |
| `expired_card` | Card expired |
| `stolen_or_lost_card` | Card reported stolen or lost |
| `timeout` | Provider did not respond in time |
| `payment_expired` | Payment expired (listed on DuitNow, wallet and Atome pages) |

Declined card charges can also carry `merchant_advice` (text on what to do next) and, when enabled on request, `merchant_advice_code` (integer). Test with Visa `4111 1111 1117 0018`; the charge amount selects the advice code (for example amount `9132` returns "Card has been temporarily blocked...").

## Charge object

| Field | Type | Notes |
| --- | --- | --- |
| `object` | string | `charge` |
| `id` | string | `/chrg(_test)?_[0-9a-z]+/` |
| `livemode` | boolean | Live or test |
| `location` | string | API path |
| `amount` | integer | Smallest unit |
| `currency` | string | ISO 4217 |
| `status` | string | `failed`, `expired`, `pending`, `reversed`, `successful` |
| `authorized` | boolean | Authorized |
| `authorized_amount` | integer | Max authorized amount |
| `authorized_at` | string | ISO 8601 |
| `authorization_type` | string | `pre_auth` or `final_auth` |
| `authorize_uri` | string | Redirect the payer here when pending (3DS, banks, wallets) |
| `authenticated_by` | string | `PASSKEY` or `3DS`, only when that flow ran |
| `return_uri` | string | Your return URL |
| `capture` | boolean | Auto capture set |
| `capturable` | boolean | Can be captured now |
| `captured_amount` | integer | Captured amount |
| `multi_capture` | boolean | Multiple partial captures allowed |
| `paid` | boolean | Captured (paid) |
| `paid_at` | string | ISO 8601 |
| `reversed`, `reversible`, `reversed_at` | boolean, boolean, string | Authorization reversal |
| `expired`, `expires_at`, `expired_at` | boolean, string, string | Expiry |
| `failure_code`, `failure_message` | string | When `status=failed` |
| `merchant_advice`, `merchant_advice_code` | string | Decline advice |
| `card` | card | Card charged (see Card object in `omise-recurring.md`) |
| `source` | source | Source charged (see `omise-malaysia-methods.md`) |
| `customer` | id/expandable | Customer |
| `description`, `metadata` | string, object | Yours |
| `ref_id` | string | Your idempotency reference |
| `ip` | string | Customer IP |
| `fee`, `fee_vat` | integer | Omise fee and VAT on fee |
| `interest`, `interest_vat` | integer | Installment interest |
| `funding_amount`, `funding_currency` | integer, string | Amount in settlement currency |
| `net` | integer | `funding_amount` after fees, interest and VAT |
| `transaction_fees` | object | `fee_flat`, `fee_rate`, `vat_rate` (strings) |
| `platform_fee` | object | `fixed`, `amount`, `percentage` |
| `refunded_amount` | integer | Total refunded |
| `refunds` | list | Refund objects |
| `refundable`, `partially_refundable` | boolean | Refund possible, partial refund possible |
| `voided`, `can_perform_void` | boolean | Voided instead of refunded; void possible |
| `disputable`, `dispute` | boolean, dispute | Dispute |
| `transaction` | id/expandable | Balance transaction |
| `schedule` | id/expandable | Schedule that created it |
| `link` | id/expandable | Link that created it |
| `zero_interest_installments` | boolean | Merchant absorbs interest |
| `approval_code`, `acquirer_reference_number` | string | Card network references |
| `missing_3ds_fields` | object | Missing cardholder fields for 3DS (test mode, or live with 3DS compliance check) |
| `three_ds_info` | object | 3DS details when enabled for the account |
| `branch`, `terminal`, `device` | expandable | Physical POS |
| `linked_account`, `merchant_name`, `merchant_uid`, `merchant_info`, `unmanaged_payment`, `created_at` | various | Direct debit, PayFac, dashboard-only, timestamps |

## Other charge operations

| Operation | Endpoint | Parameters and notes |
| --- | --- | --- |
| Retrieve | `GET /charges/{id}` | Use for settlement checks |
| List | `GET /charges` | `from`, `to`, `limit`, `offset`, `order` |
| Update | `PATCH /charges/{id}` | `description`, `metadata` |
| Capture | `POST /charges/{id}/capture` | Optional `capture_amount` (smallest unit; required only for single partial capture), `sub_merchant_country`. After expiry: `expired_charge` error; other problems: `failed_capture` |
| Reverse | `POST /charges/{id}/reverse` | Releases an uncaptured authorization. Errors: `failed_reverse` |
| Expire | `POST /charges/{id}/expire` | Pending, not yet authorized charges only. Charge API lists `alipay_cn`, `alipay_hk`, `barcode_alipay`, `dana`, `gcash`, `kakaopay`, `paypay`, `touch_n_go`; ShopeePay and Atome pages also document it. Errors: `failed_expire` |
| Mark paid / failed | `POST /charges/{id}/mark_as_paid`, `/mark_as_failed` | Test charges only, not all methods |
| Events | `GET /charges/{id}/events` | Event history |

## Cards

### Card token

`POST https://vault.omise.co/tokens` with the public key. Tokens are single use; a used token returns `used_token`.

| Parameter | Required | Notes |
| --- | --- | --- |
| `card[name]` | yes | Cardholder name |
| `card[number]` | yes | Card number |
| `card[expiration_month]` | yes | `M` or `MM` |
| `card[expiration_year]` | yes | `YY` or `YYYY` |
| `card[security_code]` | recommended | CVV |
| `card[email]`, `card[phone_number]` | recommended | Phone in `+60...` format recommended |
| `card[street1]`, `card[street2]`, `card[city]`, `card[state]`, `card[postal_code]`, `card[country]` | recommended | Billing address; `country` otherwise derived from IIN |

Token object: `object=token`, `id` (`tokn_...`), `livemode`, `location`, `card` (Card object), `charge_status`, `created_at`, `used`. `GET https://vault.omise.co/tokens/{id}` retrieves it (public key only). Since 1 April 2020 `card.security_code_check` always returns `true`; do not use it to judge validity.

Tokenization (Google Pay): `tokenization[method]=googlepay`, `tokenization[data]=<Google Pay token>` plus optional `billing_*` fields; the resulting card has `tokenization_method=googlepay` and cannot be attached to a customer.

### Ways to charge a card

- Token directly: `card=tokn_...` (one-time).
- Saved customer default card: `customer=cust_...`.
- Saved specific card: `customer=cust_...` and `card=card_...` (card id, not a token).
- Card charge limits in Malaysia: `100` to `10000000` (RM1 to RM100,000).

### Statuses with and without 3DS

- 3DS enabled: `pending` means not yet authenticated (`authorized=false`) or awaiting manual capture.
- 3DS disabled: `pending` only means authorized and awaiting manual capture.
- `successful` means `authorized=true` and `paid=true`.

## 3-D Secure

- Only 3DS2 is supported (3DS1 retired October 2022). 3DS is an account-level setting: Omise's fraud team makes it mandatory for some business types, otherwise request it from support@omise.co. It is enabled on test first, then live.
- Flow: token via Omise.js, charge with `return_uri`, response has `authorize_uri`, redirect, cardholder authenticates (frictionless or challenge), returns to `return_uri`, then fetch the charge. A completed redirect does not mean success.
- Allow 1 to 3 minutes for authentication, up to 10 minutes when the cardholder enrols for the first time. Use a waiting state, not a hard timeout.
- A missing or malformed `return_uri` is a common enrollment failure; test error `missing_return_uri`.
- `charge.complete` is sent for 3DS charges; non-3DS card charges do not trigger it.
- Trade-off documented by Omise: with 3DS on, every purchase redirects to the bank, so automatic or recurring charges cannot be processed through it directly. Storing the card on a Customer avoids re-entering card details, but the cardholder still authenticates each payment. For merchant-initiated repeat charges the Charge API offers `recurring_reason`, `transaction_indicator=MIT` and `first_charge`; confirm with Omise how these interact with 3DS on your account.

## Authorization and capture

- Automatic capture (default) captures immediately, or after a configured capture delay.
- Full manual capture: create with `capture=false` (and `authorization_type=final_auth`), then `POST /charges/{id}/capture`. One capture only.
- Single partial capture (`pre_auth` plus `capture_amount`) is for merchants contracted with Omise Thailand only. Multiple partial capture is not supported.
- Sending `capture_amount=0` captures the full amount. If you will not deliver, let the authorization expire or reverse it.
- Uncaptured card authorizations expire after 7 days (30 days for Japan-registered merchants per the errors page; 29 days per the capture page). Capturing after that returns `expired_charge`. Authorized amounts cannot be changed.

## Omise.js

Load `https://cdn.omise.co/omise.js`. HTTPS is required on the checkout page.

### Pre-built form

```html
<form id="checkoutForm" method="POST" action="/charge">
  <script type="text/javascript" src="https://cdn.omise.co/omise.js"
          data-key="<OMISE_PUBLIC_KEY>" data-amount="12345" data-currency="MYR"
          data-default-payment-method="credit_card"
          data-other-payment-methods="fpx,duitnow_obw,duitnow_qr,grabpay,boost,maybank_qr">
  </script>
</form>
```

- Your `action` endpoint receives `omiseToken` (cards) or `omiseSource` (other methods); the other is null.
- JavaScript variant: `OmiseCard.configure({ publicKey })`, then `OmiseCard.open({ amount, currency, defaultPaymentMethod, otherPaymentMethods, onCreateTokenSuccess(nonce), onFormClosed() })`. `nonce` starting with `tokn_` is a token, otherwise a source. Also `OmiseCard.configureButton(selector, config)` and `OmiseCard.attach()`. `configure` must be called before `open`.
- Parameters (data attribute / JS): `data-key`/`publicKey` (required), `data-amount`/`amount` (required), `data-currency`/`currency` (default `THB`, so set `MYR`), `data-default-payment-method`/`defaultPaymentMethod` (default `credit_card`), `data-other-payment-methods`/`otherPaymentMethods`, `data-button-label`, `data-submit-label`, `data-frame-label`, `data-frame-description`, `data-image`, `data-hide-amount`, `data-locale` (`en`, `ja`, `th`), `data-location` (`postal_code` and `city` fields), `data-submit-form-target`, plus `data-googlepay-merchant-id` (required live), `data-googlepay-request-billing-address`, `data-googlepay-request-phone-number`.
- The pre-built form's documented Malaysian methods: `boost`, `duitnow_obw`, `duitnow_qr`, `fpx`, `grabpay`, `maybank_qr`. The same table lists `credit_card` for Thailand, Singapore, Japan and `touch_n_go` and `googlepay` without Malaysia, which conflicts with the method pages; confirm in test mode. `shopeepay` is listed for Thailand only there.

### Custom form

- `Omise.setPublicKey("<OMISE_PUBLIC_KEY>")`.
- `Omise.createToken("card", { name, number, expiration_month, expiration_year, security_code, ... }, (statusCode, response) => {})`; success is `statusCode === 200` and `response.id`; errors carry `response.code` and `response.message`.
- `Omise.createSource("fpx", { amount, currency: "MYR", bank }, (statusCode, response) => {})`.
- Do not give card inputs a `name` attribute (they would post to your server). Use `data-omise` or `id`. Disable analytics on the checkout page.

### Mobile

iOS (`github.com/omise/omise-ios`) and Android (`github.com/omise/omise-android`) SDKs create tokens and sources with the public key, same as Omise.js.

## Security rules from Omise

- Card data must go from the client device to Omise's vault (Omise.js, SDKs, plugins). Only PCI DSS certified merchants may handle card data, and must tell Omise.
- Do not modify plugin source to handle cards.
- Keep the secret key in environment variables, use 2FA on the dashboard, review sessions at `https://dashboard.omise.co/sessions`.
- Supply `ip`, `description`, customer email and billing address to improve fraud scoring.
