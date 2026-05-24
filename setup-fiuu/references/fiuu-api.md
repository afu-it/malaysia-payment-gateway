# Fiuu Payment Gateway Reference

## Sources

Read/checked on 2026-05-25:

- FAQ: `https://fiuu.com/faq/`
- Seamless Integration page: `https://fiuu.com/seamless-integration/`
- GitHub organization: `https://github.com/FiuuPayment`
- API spec repository: `https://github.com/FiuuPayment/Documentation-Fiuu_API_Spec`
- Official API PDF in repo: `[official API] Fiuu API Spec for Merchant (v13.93).pdf`
- Cheatsheet/best practices: `https://github.com/FiuuPayment/Cheatsheet-BestPractices-Fiuu_API`
- JavaScript Seamless Integration: `https://github.com/FiuuPayment/Integration-Fiuu_JavaScript_Seamless_Integration`
- Inpage Checkout: `https://github.com/FiuuPayment/Integration-Fiuu_Inpage_Checkout`
- PHP SDK: `https://github.com/FiuuPayment/SDK-Fiuu_PHP`
- Flutter XDK docs: `https://pub.dev/documentation/fiuu_mobile_xdk_flutter/latest/`
- Flutter XDK repo: `https://github.com/FiuuPayment/Mobile-XDK-Fiuu_Flutter`

## Product Surfaces

- Hosted Payment Page: merchant redirects/submits customer to Fiuu-hosted payment page.
- Seamless Integration: merchant site displays channel selection; card entry still goes through Fiuu page for non-PCI mode.
- Inpage Checkout: card-focused iframe checkout.
- Mobile XDK: native/mobile plugins for Flutter, Android, iOS, React Native, Expo, Cordova, Ionic Capacitor, NativeScript, MAUI, and examples.
- SDKs: PHP and Java repos exist; use official source-map before choosing.
- Shopping cart plugins: WooCommerce, Magento, OpenCart, WHMCS, Shopify Hosted Payment, Drupal Commerce, VirtueMart/Joomla.
- Direct Server Integration: host-to-host, no Fiuu UI; requires PCI-DSS scope and direct request to Fiuu support.
- Recurring API: token-based subscription/MIT flow; feature/tokenization must be enabled by Fiuu.
- Merchant request APIs: status requery, capture, reversal, refund, settlement/reporting, channel status/success rate, static QR, token APIs.
- POS/offline: Offline Payment API, Virtual Terminal SDK, cloudECR, ECR Cloud docs.

## Environment

Hosted payment page:

```text
Production: https://pay.fiuu.com/RMS/pay/{MerchantID}/{Payment_Method}
Sandbox:    https://sandbox-payment.fiuu.com/RMS/pay/{MerchantID}/{Payment_Method}
```

API host guidance from official PDF:

```text
Production API host: api.fiuu.com
Sandbox API host:    sandbox-api.fiuu.com
```

Sandbox credentials are separate and commonly use a sandbox merchant ID prefix such as `SB_...`. Keep sandbox and production Merchant ID, Verify Key, Secret Key, password, URLs, channel settings, and webhooks separate.

## Credentials

- Merchant ID: identifies merchant/domain.
- Verify Key: used for request-side hashes such as `vcode`.
- Secret Key: top secret response/API verification key; never expose publicly.
- Password/API credentials: used by some SDK/API surfaces.
- Mobile XDK also uses username, password, merchant ID, app name, verification key, and optionally Secret Key for backend verification.

## Hosted Payment Request

Common request parameters include:

- `amount`: mandatory, integer or up to 2 decimal points; comma is not allowed.
- `orderid`: mandatory, alphanumeric, up to 40 chars.
- `bill_name`, `bill_email`, `bill_mobile`, `bill_desc`.
- `country`, for Malaysia use `MY`.
- `currency`, for Malaysia commonly `MYR`.
- `vcode`: mandatory for normal payment integrity unless using a documented optional/open-amount case.
- `returnurl`, `callbackurl`, `cancelurl` where relevant.
- `channel` or `{Payment_Method}` when bypassing hosted channel selection.

Most API requests are form encoded. Prefer `application/x-www-form-urlencoded` unless a specific Fiuu doc says otherwise. If both GET and POST carry the same parameter, be careful: many server frameworks prioritize POST.

## vcode

`vcode` protects merchant-to-Fiuu payment request integrity.

Standard formula:

```text
vcode = md5({amount}{merchantID}{orderID}{verify_key})
```

Extended vcode formula when the merchant portal setting is enabled:

```text
vcode = md5({amount}{merchantID}{orderID}{verify_key}{currency})
```

Rules:

- Generate on the backend from server-owned amount/order data.
- Keep Verify Key server-side.
- Use exact amount string format expected by Fiuu, usually two decimal places.
- Do not let browser/mobile decide amount, currency, merchant ID, or order ID.
- Fiuu provides a vcode verification tool at `https://api.fiuu.com/RMS/query/vcode.php`.

## Payment Status Endpoints

Fiuu has three endpoint concepts:

- Return URL: browser/frontend redirect endpoint. Useful for UX only; not a durable source of truth.
- Notify/Notification URL: server-to-server backend webhook. Recommended primary settlement signal.
- Callback URL: deferred/non-realtime status webhook, including cash-channel status changes.

Rules:

- Implement backend handlers for Notify/Notification URL and Callback URL.
- Verify `skey` before trusting payload fields.
- Compare `orderid`, `tranID`, `domain`, `amount`, `currency`, and status against your local pending payment.
- Return/ACK only after durable record or queue write.
- When IPN is enabled and ACK fails, Fiuu can retry webhook delivery at intervals. Design idempotent handlers.

## skey

Payment response `skey` protects Fiuu-to-merchant result integrity.

Formula from official API PDF:

```text
pre_skey = md5({txnID}{orderID}{status}{merchantID}{amount}{currency})
skey = md5({paydate}{merchantID}{pre_skey}{appcode}{secret_key})
```

Typical payload fields:

- `tranID`
- `orderid`
- `status`
- `domain`
- `amount`
- `currency`
- `appcode`
- `paydate`
- `skey`

Rules:

- Use Secret Key only on the backend.
- Compare with constant-time comparison where available.
- Verify the local stored order, amount, and currency before settlement.
- If mismatch, reject or requery status rather than fulfilling.

## Status Handling

Common web/mobile status codes:

- `00`: success.
- `11`: failed.
- `22`: pending. Pending is expected for some channels such as cash or FPX B2B/M2E approval flows.

Do not treat pending as paid. Requery or wait for notification/callback.

## Requery, Capture, Refund

- Direct Status Requery asks the channel for realtime aligned status.
- Indirect Status Requery returns Fiuu-side status.
- Reversal is for void-like same-day flows where supported.
- Full/partial refund can be supported up to provider/channel limits and may be allowed for a longer period than reversal.
- Capture is relevant for pre-authorization flows.

Always use the exact endpoint/formula from the current official PDF for these APIs.

## Seamless Integration

Fiuu Seamless Integration lets merchant site display channel selection and launch the channel flow without sending customers to the standard hosted channel-selection page.

Important constraints:

- Non-PCI Seamless still sends card number entry through Fiuu, so merchant does not host raw card entry.
- Domain registration/whitelisting is required for Seamless. Register in portal or via support.
- Multiple subdomains can be accepted, but docs note one domain per Merchant ID.
- Embedded page/iframe is not allowed for Seamless.
- Use Channel Status API for subscribed channels when merchant controls channel selection.
- Some in-app browsers may not return to the merchant website after selected bank flows.

## Inpage Checkout

Inpage Checkout is similar to Seamless but card-focused and uses an iframe for card input. Use its repo/wiki for exact integration details and current browser/card constraints.

## Mobile XDK

Flutter package: `fiuu_mobile_xdk_flutter`.

Minimums from pub.dev latest docs checked on 2026-05-25:

- Android SDK Version 27 or higher.
- Android API level 19 or higher.
- Android target version Android 4.4 or higher.
- Xcode 13 or higher.
- iOS target 16 or higher.

Flutter payment details commonly include:

- `mp_dev_mode`
- `mp_username`
- `mp_password`
- `mp_merchant_ID`
- `mp_app_name`
- `mp_verification_key`
- `mp_amount`
- `mp_order_ID`
- `mp_currency`
- `mp_country`
- `mp_channel`
- `mp_bill_description`
- `mp_bill_name`
- `mp_bill_email`
- `mp_bill_mobile`

XDK environment values:

```text
mp_core_env=1 Production V1 https://pay.fiuu.com/RMS/API/xdk/
mp_core_env=2 Production V2 https://xdk.fiuu.com/
mp_core_env=3 UAT V2        https://uat-xdk.fiuu.com/
mp_core_env=4 Sandbox V2    https://sandbox-xdk.fiuu.com/
default       Production V2 https://xdk.fiuu.com/
```

Other channel result checksum:

```text
chksum = MD5(mp_merchant_ID + msgType + txn_ID + amount + status_code + merchant_private_secret_key)
```

Google Pay result verification:

```text
VrfKey = md5(Amount + secret_key + Domain + TranID + StatCode)
```

XDK caveat: built-in checksum validation can return false when private Secret Key is used. Prefer sending result to backend and verifying there.

## Loopholes And Fixes

- Loophole: fulfilling from Return URL or mobile result.
  Fix: use backend Notify/Callback verification plus requery when needed.
- Loophole: trusting frontend amount/order/currency.
  Fix: create local pending payment first and hash server-side values.
- Loophole: exposing Secret Key or Verify Key.
  Fix: keep secrets in backend env vars or server secret store.
- Loophole: accepting duplicate notifications.
  Fix: unique constraint on `tranID` or `(provider, orderid, status/source)` plus idempotent settlement.
- Loophole: ignoring pending/deferred channels.
  Fix: model pending separately and wait for callback/requery.
- Loophole: using Seamless without domain registration.
  Fix: register/approve domain before production.
- Loophole: using Direct Server/card entry without PCI scope.
  Fix: use Hosted/Seamless non-PCI/Inpage or obtain proper approval/compliance.
