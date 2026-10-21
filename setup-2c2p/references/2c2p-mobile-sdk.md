# 2C2P PGW Mobile SDK Reference

Use this reference when a merchant accepts 2C2P payments inside a native or cross-platform mobile app (Android, iOS, Flutter, React Native). It covers the PGW SDK, PGW UI SDK, PGW SDK Helper (Apple Pay, Google Pay, deep links) and SecurePay SDK. Server-side Payment Token, backend notification and Payment Inquiry live in `2c2p-api.md`.

Checked 2026-10-21 against developer.2c2p.com. Most SDK sample payloads use Thai or Singapore demo merchants (`JT01`, `JT04`, `THB`, `SGD`); swap in your own merchant ID and `MYR`.

## Golden Rule: The App Never Decides "Paid"

- The SDK result (`TransactionCompleted`, `PaymentUIResponseCode.TransactionCompleted`, Apple/Google Pay `TokenGeneratedSuccess`, deep link `PaymentTransactionStatusInquiry`) only means the customer finished the flow. It is not proof of payment.
- The SDK flow itself says: 2C2P sends the backend notification to the merchant server, then "Merchant application uses transaction ID to retrieve transaction result from merchant server."
- The SDK Transaction Status Inquiry is for displaying a result to the customer. 2C2P states merchants must not use Transaction Status Inquiry to acknowledge payment status; use the server-side Payment Inquiry API.
- Settle only on the server: verify the backend notification JWT (HS256, merchant secret key) or call Payment Inquiry from the server, then match `invoiceNo`, `amount`, `currencyCode`, `merchantID` and `respCode` `0000`.
- Never ship the merchant secret key, the JWT signing code or the Payment Token call in the app. The app only ever receives the `paymentToken` string from your server.

Source: https://developer.2c2p.com/docs/sdk-how-it-work, https://developer.2c2p.com/docs/api-transaction-status-inquiry, https://developer.2c2p.com/docs/api-sdk-transaction-status-inquiry

## How It Works

Card payment flow as documented:

1. Customer checks out in the app.
2. App sends checkout info to the merchant server.
3. Merchant server requests a payment token from 2C2P (Payment Token API, JWT HS256).
4. Merchant server passes the `paymentToken` to the app.
5. App sends the payment request to 2C2P through the SDK (`proceedTransaction`).
6. 2C2P sends the payment to the acquirer (with 3DS when required).
7. 2C2P sends the backend notification to the merchant server (`backendReturnUrl`).
8. 2C2P returns the result / transaction reference to the SDK.
9. App asks the merchant server for the transaction result.
10. App shows the result.

3DS card flow adds: 2C2P sends an authentication URL to the app, the app opens it in a WebView (SDK gives `TransactionAuthenticateRedirect` / `TransactionAuthenticateFullRedirect`), the issuer OTP completes, then 2C2P authorises, sends the backend notification and relays the status to the SDK.

Two integration styles:

- **Payment UI** (`PaymentUIBuilder`): 2C2P renders the full payment UI inside the app. Least code.
- **Custom UI**: the app renders its own screens, uses Payment Option / Payment Option Details to list channels, builds a payment with a builder class (`CardPaymentBuilder`, `DigitalPaymentBuilder`, and so on) and calls `proceedTransaction`.

Source: https://developer.2c2p.com/docs/sdk-how-it-work, https://developer.2c2p.com/docs/sdk-method-card-payments, https://developer.2c2p.com/docs/sdk-payment-authorization, https://developer.2c2p.com/docs/sdk-payment-features

## Install Coordinates

| Package | Android (Gradle) | iOS (Swift Package) | Min OS |
| --- | --- | --- | --- |
| PGW SDK | `implementation("com.2c2p:pgw-sdk:4.7.1")` | `https://github.com/2C2P/PGWSDK-SwiftPackage` | iOS 12.0+, Android 19+ |
| PGW UI SDK | `implementation("com.2c2p:pgw-ui-sdk:4.0.5")` | `https://github.com/2C2P/PGWUISDK-SwiftPackage` | iOS 13.0+, Android 24+ |
| PGW SDK Helper | `implementation("com.2c2p:pgw-sdk-helper:4.2.1")` (changelog lists up to 4.2.3) | `https://github.com/2C2P/PGWSDKHELPER-SwiftPackage` | iOS 12.0+, Android 19+ |
| SDK Helper ZaloPay add-on (optional) | `implementation("com.2c2p:pgw-sdk-helper-zalopay:3.1.0")` | n/a | |
| Google Pay dependency (optional) | `implementation("com.google.android.gms:play-services-wallet:19.4.0")` | n/a | |
| SecurePay SDK | `implementation("com.2c2p:secure-pay-sdk:2.1.0")` | `https://github.com/2C2P/SecurePaySDK-SwiftPackage` | iOS 12.0+, Android 19+ |
| Flutter | `pgw_sdk: ^4.1.0` plus `webview_flutter: ^4.14.1`, `meta: ^1.19.0` in pubspec.yaml | iOS via Swift Package Manager (`flutter config --enable-swift-package-manager`) | iOS 15.0+, Android 24+, Flutter >=3.41.0, Dart ^3.11.0 |
| React Native | `npm install @2c2p/pgw-sdk-react-native@4.0.5` (or `yarn add`) | `npm explore @2c2p/pgw-sdk-react-native -- npm run ios-frameworks`, then `bundle exec pod install` | Android minSdk 24 |
| React Native Expo | `npx expo install @expo/config-plugins expo-build-properties @2c2p/pgw-sdk-react-native@4.0.5` | config via app.json plugin | |

iOS install notes:

- Swift Package Collection URL: `https://swiftpackageindex.com/2C2P/collection.json` (Xcode > Package Dependencies > add collection).
- Add this to Build Phases > Run Script: `bash "${BUILT_PRODUCTS_DIR}/PGW.framework/integrate-dynamic-framework.sh"`.
- 2C2P links Apple forum threads about Xcode 16 missing dSYM warnings.

Android UI SDK prerequisites: `ndk { abiFilters 'armeabi-v7a', 'arm64-v8a', 'x86_64' }`, Java 17 `compileOptions`, and `packaging { resources { excludes += "META-INF/versions/9/OSGI-INF/MANIFEST.MF" } }`. iOS UI SDK needs `LSApplicationQueriesSchemes` (`zalo`, `zalopay`, `zalopay.api.v2`, `scbeasy`, `momo`, `line`) and `CADisableMinimumFrameDurationOnPhone` in Info.plist.

Flutter Android: update `settings.gradle` plugin management (example pins `com.android.application` 9.4.0, Kotlin 2.4.20) and add ProGuard keeps for `com.ccpp.pgw.sdk.flutter.**` and `com.fasterxml.jackson.**`. React Native Android: minSdk 24, compile/target 36, `reactNativeArchitectures=armeabi-v7a,arm64-v8a,x86_64`, ProGuard keeps for `com.ccpp.pgw.sdk.reactnative.**`, Gradle 8.13. React Native iOS also needs deep link handling, Apple Pay setup and the Payment UI Info.plist entries.

EOL warning: 2C2P marks old SDK versions End of Life when the backend SSL certificate changes, and EOL versions stop executing payments. Versions 4.3.3 and older are marked EOL. Keep the SDK current.

React Native exports: `RTNPGW`, `APIEnvironment`, `APIResponseCode`, `ApplePayPaymentResponseCode`, `DeepLinkPaymentResponseCode`, `GooglePayAPIEnvironment`, `GooglePayPaymentResponseCode`, `InstallmentInterestTypeCode`, `PaymentCustomDataCode`, `PaymentChannelCode`, `PaymentInputCode`, `PaymentNotificationPlatformCode`, `PaymentUIResponseCode`, `QRTypeCode`, `ZaloPayAPIEnvironment`, `ZaloPayPaymentResponseCode`.

CocoaPods: Not documented in official docs for the native iOS SDK (checked 2026-10-21); only Swift Package Manager and older zip frameworks. React Native uses `pod install` for its own wrapper.

Source: https://developer.2c2p.com/docs/sdk-import-sdk, https://developer.2c2p.com/docs/sdk-download-sdk, https://developer.2c2p.com/docs/sdk-download-ui-sdk, https://developer.2c2p.com/docs/sdk-download-sdk-helper, https://developer.2c2p.com/docs/sdk-download-securepay, https://developer.2c2p.com/docs/sdk-download-sdk-flutter, https://developer.2c2p.com/docs/sdk-download-sdk-react-native

## Initialize the SDK

Initialize once in the `Application` (Android) or `AppDelegate` (iOS) class, before any payment call.

```kotlin
class CustomApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        val params = PGWSDKParamsBuilder(this, APIEnvironment.Sandbox).build() // Production for live
        PGWSDK.initialize(params)
    }
}
```

```swift
let params: PGWSDKParams = PGWSDKParamsBuilder(apiEnvironment: APIEnvironment.Sandbox).build()
PGWSDK.initialize(params: params)
```

Flutter: `PGWSDK().initialize({'apiEnvironment': APIEnvironment.production}, (error) {...})`. React Native: `RTNPGW.initialize(JSON.stringify({'apiEnvironment': APIEnvironment.production}))`.

`PGWSDKParamsBuilder` fields: constructor `(context, apiEnvironment)` on Android, `(apiEnvironment)` on iOS (M); `clientId` (O, UUID); `locale` (O, ISO 639); `log` (O, Boolean).

`APIEnvironment` values: `Production` (all countries except Indonesia), `ProductionIndonesia` (Indonesia only), `Sandbox` (testing). Malaysian merchants use `Sandbox` then `Production`. The SDK environment must match the server environment that issued the payment token (sandbox token with `Sandbox`).

Locale rule (applies to every request class): empty locale uses the payment token locale; a set locale wins; if `clientId` exists, the user preference locale is used.

Source: https://developer.2c2p.com/docs/sdk-initial-sdk, https://developer.2c2p.com/docs/sdk-classes-payment-request, https://developer.2c2p.com/docs/sdk-classes-payment-enums

## Server Side: Payment Token for the App

The server calls the normal Payment Token API (see `2c2p-api.md`). SDK guides add a `paymentChannel` filter per method. Examples documented:

| Method | Token request extras |
| --- | --- |
| Global card | `"paymentChannel": ["GCARD"], "request3DS": "Y"` |
| Local card | `"paymentChannel": ["PCC"]` |
| Card tokenization without auth | `["GCARD"]`, `"request3DS": "Y"`, `"tokenizeOnly": true` |
| Customer token payment | `["GCARD"]`, `"request3DS": "Y"`, `"customerToken": ["<CUSTOMER_TOKEN>"]` |
| IPP | `["GCARD"]`, `"installmentPeriodFilter": [6]`, `"interestType": "M"`, `"productCode"`, `"promotionCode"` |
| LIPP | `["LCARDIPP"]` plus the IPP fields |
| RPP | `["GCARD"]`, `"recurring": true`, `invoicePrefix`, `recurringAmount`, `recurringInterval`, `recurringCount`, `allowAccumulate`, `maxAccumulateAmount`, `chargeNextDate` (`ddMMyyyy`) |
| E-wallets, Apple Pay, Google Pay | `["DPAY"]` (Google Pay sample adds `"request3DS": "Y"`) |
| E-wallet tokenization without auth | `["DPAY"]`, `"tokenizeOnly": true` |
| QR | `["QR"]` |
| BNPL | `["GBNPL"]` |
| Internet banking | `["IMBANK"]` |
| Web payment (bank redirect, incl. FPX banks) | `["WEBPAY"]` |
| Pay at counter | `["COUNTER"]` |
| Self service machines | `["SSM"]` |
| Deep link | `["DEEPLINK"]` |
| Online direct debit | `["ODD"]` |
| Third party | `["GTPTY", "LTPTY"]` |
| Card loyalty points | `["GCARD"]` plus `loyaltyPoints: [{providerID, externalMerchantId, accountNo, redeemAmount, rewards:[{quantity}]}]` |
| Digital currency (Triple-A) | `["DCURRENCY"]` |

Always set `backendReturnUrl` on the server so the notification reaches you even if the app dies mid-flow.

Source: https://developer.2c2p.com/docs/sdk-method-credit-debit-card, https://developer.2c2p.com/docs/sdk-customer-tokenization-without-authorization, https://developer.2c2p.com/docs/sdk-payment-with-customer-token, https://developer.2c2p.com/docs/sdk-installment-payment-plan, https://developer.2c2p.com/docs/sdk-installment-payment-plan-local, https://developer.2c2p.com/docs/sdk-recurring-payment-plan, https://developer.2c2p.com/docs/sdk-method-qr-payment, https://developer.2c2p.com/docs/sdk-card-loyalty-point-payment, https://developer.2c2p.com/docs/api-payment-token

## Standard Custom-UI Pattern (All Methods)

Every method page follows the same six steps:

1. Server generates payment token.
2. App receives `paymentToken`.
3. Build a `PaymentCode` and a payment request with the right builder.
4. Wrap it: `TransactionResultRequestBuilder(paymentToken).with(paymentRequest).build()`.
5. Call `PGWSDK.getInstance().proceedTransaction(...)` (Android) / `PGWSDK.shared.proceedTransaction(...)` (iOS) and branch on `responseCode`.
6. Call Transaction Status Inquiry for display, and let the server confirm via Payment Inquiry.

```kotlin
val paymentCode = PaymentCode("CC")
val paymentRequest = CardPaymentBuilder(paymentCode, "4111111111111111").apply {
    expiryMonth(12); expiryYear(2026); securityCode("123")
}.build()
val request = TransactionResultRequestBuilder(paymentToken).apply { with(paymentRequest) }.build()

PGWSDK.getInstance().proceedTransaction(request, object : APIResponseCallback<TransactionResultResponse> {
    override fun onResponse(response: TransactionResultResponse) {
        when (response.responseCode) {
            APIResponseCode.TransactionAuthenticateRedirect,
            APIResponseCode.TransactionAuthenticateFullRedirect -> openWebView(response.data)   // 3DS / bank page
            APIResponseCode.TransactionExternalApplication -> openApp(response.data)            // wallet deep link
            APIResponseCode.TransactionQRPayment -> showQr(response.type, response.data)        // then poll status
            APIResponseCode.TransactionPaymentSlip -> showSlip(response.data)                   // pending
            APIResponseCode.TransactionCompleted -> askServerForResult()                        // not proof of payment
            else -> showError(response.responseDescription)
        }
    }
    override fun onFailure(error: Throwable) { showError(error.message) }
})
```

```swift
PGWSDK.shared.proceedTransaction(transactionResultRequest: request, { (response: TransactionResultResponse) in
    if response.responseCode == APIResponseCode.TransactionAuthenticateRedirect
        || response.responseCode == APIResponseCode.TransactionAuthenticateFullRedirect {
        let redirectUrl: String = response.data // open PGWWebViewController
    } else if response.responseCode == APIResponseCode.TransactionCompleted {
        // ask your server; server confirms with Payment Inquiry
    } else { /* show error */ }
}) { (error: NSError) in /* show error */ }
```

The Kotlin `when` form above condenses the documented if/else samples; method names, callback types, builders and enum values are as documented.

Source: https://developer.2c2p.com/docs/sdk-method-credit-debit-card, https://developer.2c2p.com/docs/sdk-api-do-payment, https://developer.2c2p.com/docs/response-code-payment-flow

## Method Catalogue (Builder, PaymentCode, Expected Response)

| Method | Builder | PaymentCode example | Builder fields used in docs | Main response codes |
| --- | --- | --- | --- | --- |
| Global card | `CardPaymentBuilder(paymentCode, cardNo)` | `PaymentCode("CC")` | `expiryMonth`, `expiryYear`, `securityCode` | AuthenticateRedirect / FullRedirect, Completed |
| Local card | `CardPaymentBuilder` | `PaymentCode("KBZ")` (Myanmar sample) | `expiryMonth`, `expiryYear`, `pin` | same |
| Card + tokenize | `CardPaymentBuilder` | `"CC"` | `tokenize(true)` | same |
| Tokenize without auth | `CardPaymentBuilder` | `"CC"` | token request has `tokenizeOnly: true` | same |
| Customer token payment | `CustomerTokenPaymentBuilder(paymentCode, customerToken)` | `"CC"` | `securityCode` | same |
| IPP | `CardPaymentBuilder` | `PaymentCode("IPP")` | `installmentInterestType(InstallmentInterestTypeCode.Merchant)`, `installmentPeriod(6)` | same |
| LIPP | `CardPaymentBuilder` | `PaymentCode("LIPP")` | same as IPP | same |
| RPP | `CardPaymentBuilder` | `"CC"` | recurring settings come from the token request | same |
| Customer address | `CardPaymentBuilder` + `UserAddressBuilder` | `"CC"` | `userAddress(UserAddressBuilder().userBillingAddress(...).build())` | same |
| Loyalty points | `LoyaltyPointPaymentBuilder(paymentCode)` | `"CC"` | `cardNo`, expiry, `securityCode`, `loyaltyPoints(list)` | same |
| E-wallet (TNG, Boost, GrabPay, Alipay, others) | `DigitalPaymentBuilder(paymentCode)` | `PaymentCode("TNG")`, `"BOOST"`, `"GRAB"`, `"ALIPAY"` | `name`, `email`, `mobileNo` (some wallets `accountNo`) | AuthenticateRedirect / FullRedirect, ExternalApplication |
| E-wallet tokenize without auth | `DigitalPaymentBuilder` | `PaymentCode("GRAB")` | token request has `tokenizeOnly: true` | same as e-wallet |
| Apple Pay | `DigitalPaymentBuilder` + Helper | `PaymentCode(channelCode: "APPLEPAY")` | `token(applePayToken)` | Completed |
| Google Pay | `DigitalPaymentBuilder` + Helper | `PaymentCode("GOOGLEPAY")` | `token(googlePayToken)` | AuthenticateRedirect / FullRedirect, Completed |
| QR | `QRPaymentBuilder(paymentCode)` | `PaymentCode("VEMVQR")` | `type(QRTypeCode.URL)` | QRPayment |
| BNPL | `BuyNowPayLaterPaymentBuilder(paymentCode)` | `PaymentCode("HOOLAH")` (sample) | `name`, `email`, `installmentPayLaterPeriod` (optional) | AuthenticateRedirect / FullRedirect, Completed |
| Internet banking | `InternetBankingBuilder(paymentCode)` | `PaymentCode("123", "THUOB", "IBANKING")` | `name`, `email`, `mobileNo` | PaymentSlip |
| Web payment | `WebPaymentBuilder(paymentCode)` | `PaymentCode("123", "THBBL", "WEBPAY")` | `name`, `email`, `mobileNo` | AuthenticateRedirect / FullRedirect |
| Web payment card (WPC) | `WebPaymentBuilder` | `PaymentCode("123", "VNABB", "WEBPAY")` | plus `cardNo`, expiry, issued month/year, `securityCode` | AuthenticateRedirect / FullRedirect, Completed |
| Pay at counter | `PayAtCounterBuilder` | `PaymentCode("123", "BIGC", "OVERTHECOUNTER")` | `name`, `email`, `mobileNo` | PaymentSlip |
| Self service machine | `SelfServiceMachineBuilder` | `PaymentCode("123", "THUOB", "ATM")` | same | PaymentSlip |
| Deep link | `DeepLinkPaymentBuilder` | `PaymentCode("123", "THSCB", "DEEPLINK")` | same | AuthenticateRedirect / FullRedirect, ExternalApplication |
| Online direct debit | `OnlineDirectDebitBuilder` | `PaymentCode("123", "THKTB", "ODD")` | same | AuthenticateRedirect / FullRedirect |
| Third party | `CardPaymentBuilder(paymentCode)` | `PaymentCode("UPOP")` | `name`, `email`, `mobileNo` | AuthenticateRedirect / FullRedirect, Completed |
| Digital currency | `DigitalCurrencyBuilder` | `PaymentCode("TRIPLEA")` | `name`, `email`, `mobileNo` | AuthenticateRedirect / FullRedirect |
| Any type | `UniversalPaymentBuilder(paymentCode)` | `PaymentCode(PaymentChannelCode.Group.CreditCard)` | union of all fields | per channel |

Malaysian channel codes to plug in (from the payment channel list, not from SDK samples): e-wallets `TNG`, `BOOST`, `GRAB`, `SHPPAY` (ShopeePay), `ALIPAY`, `WECHAT`, `SHOPBACK`; DuitNow QR `DUITNOW` (group QRC); BNPL `ATOME`, `GRABBNPL`, `SPAYLATER`; FPX banks run as WEBPAY channel `123` with agent codes such as `MYM2U`, `MYCIMBCLICKS`, `MYPBB`, `MYRHB`, `MYHLB`, so the SDK form is `PaymentCode("123", "MYM2U", "WEBPAY")`. That last construction follows the documented `PaymentCode(channelCode, agentCode, agentChannelCode)` pattern; 2C2P does not publish an SDK sample with a Malaysian bank, so test it in sandbox (sandbox test banks `MYSBIA`, `MYSBIB`, `MYSBIC`). Always prefer the codes returned by Payment Option Details for your account.

Other documented wallet pages follow the identical `DigitalPaymentBuilder` pattern and are not Malaysian: GCash, LINE Pay, ZaloPay (needs Helper `proceedZaloPayPayment`), OK Dollar, MoMo, WavePay, TrueMoney, Masterpass, PayPal, M-Pitesan, KBZPay, SPA, CB Pay, OVO, LinkAja.

Source: https://developer.2c2p.com/docs/sdk-classes-payment-request, https://developer.2c2p.com/docs/sdk-method-touch-n-go, https://developer.2c2p.com/docs/sdk-method-boost-wallet, https://developer.2c2p.com/docs/sdk-method-grab-pay, https://developer.2c2p.com/docs/sdk-method-alipay, https://developer.2c2p.com/docs/sdk-method-qr-payment, https://developer.2c2p.com/docs/sdk-method-buy-now-pay-later, https://developer.2c2p.com/docs/sdk-method-internet-banking, https://developer.2c2p.com/docs/sdk-method-web-payment, https://developer.2c2p.com/docs/sdk-method-web-payment-card-wpc, https://developer.2c2p.com/docs/sdk-method-pay-at-counter, https://developer.2c2p.com/docs/sdk-method-self-service-machines, https://developer.2c2p.com/docs/sdk-deep-link-payment, https://developer.2c2p.com/docs/sdk-online-direct-debit, https://developer.2c2p.com/docs/sdk-method-third-party-payment, https://developer.2c2p.com/docs/sdk-method-triplea, https://developer.2c2p.com/docs/sdk-method-credit-debit-card-local, https://developer.2c2p.com/docs/sdk-user-address-for-payment, https://developer.2c2p.com/docs/sdk-card-loyalty-point-payment, https://developer.2c2p.com/docs/reference-payment-channels, https://developer.2c2p.com/docs/sdk-payment-methods

## Payment UI (2C2P-Rendered Screens)

```kotlin
val paymentUIRequest = PaymentUIBuilder(activity).build()          // iOS: PaymentUIBuilder(uiViewController:)
val request = TransactionResultRequestBuilder(paymentToken).apply { with(paymentUIRequest) }.build()
PGWSDK.getInstance().proceedTransaction(request, object : APIResponseCallback<TransactionResultResponse> {
    override fun onResponse(response: TransactionResultResponse) {
        if (response.responseCode == APIResponseCode.TransactionCompleted) {
            val token = response.paymentToken  // run Transaction Status Inquiry for display, server confirms
        } else { /* show error */ }
    }
    override fun onFailure(error: Throwable) { /* show error */ }
})
```

Requires the PGW UI SDK package. `PaymentUIResponseCode`: `TransactionCompleted` 0000 (use payment token for transaction status inquiry), `TransactionCancelled` 0001 (user cancelled), `TransactionFailed` 0002, `TransactionRetryFailed` 0003 (re-submit the UI payment request), `ValidationFailed` 0099, `InitializationFailed` 0099 (iOS).

Source: https://developer.2c2p.com/docs/sdk-api-payment-ui, https://developer.2c2p.com/docs/sdk-classes-payment-enums, https://developer.2c2p.com/docs/sdk-download-ui-sdk

## Card Tokenization, Token Payments, IPP, RPP

- **Tokenize with authorisation**: set `tokenize(true)` on `CardPaymentBuilder` (also on WebPayment, Digital, ODD, Universal, Loyalty builders). When true, 2C2P returns the customer token on Payment Inquiry and in the backend notification (`customerToken`, `customerTokenExpiry`). Store it server-side.
- **Tokenize without authorisation**: server sends `tokenizeOnly: true` in the token request; app submits card details normally; no charge. 2C2P warns merchants must obtain customer consent before storing card details.
- **Token payment**: server passes `customerToken: ["<token>"]` in the token request; app uses `CustomerTokenPaymentBuilder(paymentCode, customerToken).securityCode("123")`. Customers still enter CVV; show the masked PAN and collect CVV via secure entry. `expiryMonth` / `expiryYear` on this builder are only for updating the token expiry.
- **List stored cards**: `CustomerTokenInfoRequest(paymentToken)` returns `customerTokens` with `channelCode`, `token`, `accountNo` (masked), `expiry`, `name`, `email`, `status` (`A` available, `EX` expiring within 6 months, `ED` expired), `iconUrl`, `logoUrl`, `cardBrand`.
- **IPP / LIPP**: token request filters plans (`installmentPeriodFilter`, `interestType` `A`/`C`/`M`, `productCode`, `promotionCode`); app sets `installmentInterestType(InstallmentInterestTypeCode.Customer|Merchant)` and `installmentPeriod(months)`. Plans are visible in the my2C2P dashboard under IPP Plans or via Payment Option Details.
- **RPP**: recurring parameters live only in the server token request; the app sends a normal card payment. 2C2P then charges automatically on schedule. Changes go through the Recurring Payment Plan Maintenance API (server side, see `2c2p-api.md`).

Source: https://developer.2c2p.com/docs/sdk-customer-tokenization, https://developer.2c2p.com/docs/sdk-customer-tokenization-without-authorization, https://developer.2c2p.com/docs/sdk-payment-with-customer-token, https://developer.2c2p.com/docs/sdk-e-wallet-tokenization-without-authorization, https://developer.2c2p.com/docs/sdk-api-customer-tokens-information, https://developer.2c2p.com/docs/sdk-installment-payment-plan, https://developer.2c2p.com/docs/sdk-installment-payment-plan-local, https://developer.2c2p.com/docs/sdk-recurring-payment-plan, https://developer.2c2p.com/docs/sdk-payment-features, https://developer.2c2p.com/docs/sdk-classes-payment-info

## Handling Redirects, 3DS and Deep Links

**3DS / bank redirect (`TransactionAuthenticateRedirect`, `TransactionAuthenticateFullRedirect`)**: open `response.data` in a WebView wired to the SDK client so the SDK can detect completion.

- Android: `webView.webViewClient = PGWWebViewClient(transactionStatusCallback, webViewClientCallback)`. Mandatory settings: `javaScriptEnabled = true`, `domStorageEnabled = true` (some bank pages such as AmBank need it). `PGWWebViewTransactionStatusCallback.onInquiry(paymentToken)` fires at the end: run Transaction Status Inquiry and close the WebView.
- iOS: `WKWebView` with JavaScript enabled and `navigationDelegate = PGWWebViewNavigationDelegate({ paymentToken in ... })` (sample class `PGWWebViewController`). Optional `navigationCallback(didStartProvisionalNavigation:decidePolicyForNavigationAction:didFinishNavigation:)` for logging.

**External app (`TransactionExternalApplication`, flow 1004)**: hand off to PGW SDK Helper.

```kotlin
val providerRequest = PaymentProviderBuilder(activity).apply { transactionResultResponse(response) }.build()
PGWSDKHelper.getInstance().proceedDeepLinkPayment(providerRequest, object : PaymentResultResponseCallback<DeepLinkPaymentResultResponse> {
    override fun onResponse(r: DeepLinkPaymentResultResponse) {
        if (r.responseCode == DeepLinkPaymentResponseCode.PaymentTransactionStatusInquiry) { /* status inquiry by payment token */ }
    }
    override fun onFailure(error: Throwable) { /* show error */ }
})
```

iOS deep link prerequisites:

1. In `AppDelegate.application(_:open:options:)` call `PGWSDKHelper.shared.universalPaymentResultObserver(url: url)` (SwiftUI: `.onOpenURL { PGWSDKHelper.shared.universalPaymentResultObserver(url: $0) }`).
2. Add a custom URL scheme under URL Types in the format `pgw{your app bundle identifier}`.
3. Add target app schemes to `LSApplicationQueriesSchemes`. Documented list is Thai/Vietnam apps only: `line`, `momo`, `scbeasy`, `bualuangmbanking`, `ktbnext` (UAT `ktbnextuat`), `krungsri-kma`. Schemes for Touch 'n Go, Boost, GrabPay or ShopeePay: Not documented in official docs (checked 2026-10-21).

`TransactionResultResponse.fallbackData` (flow 1004 only) lets users without the native app pay via web. `DeepLinkPaymentResponseCode`: `PaymentTransactionStatusInquiry` 0000, `PaymentTokenNotFound` 0001, `PaymentCancelled` 0002, `OpenAppWithDataSuccess` / `OpenAppWithFallbackDataSuccess` 0003, `RetryOpenAppWithFallbackData` 0004, `OpenAppWithDataAndFallbackFailed` 0005, `ValidationFailed` / `InvalidPaymentResultResponseCallback` 0099, Android-only `ThirdPartySDKFailed`, `LaunchPaymentActivityFailed`, `PaymentInfoInitializationFailed` 0099.

**External browser (`TransactionExternalBrowser`, flow 1002)**: open the URL; on app resume call Transaction Status Inquiry.

**QR (`TransactionQRPayment`, flow 1005)**: `response.type` is `QRTypeCode.URL` (load image from URL), `Base64` (decode image) or `Raw` (render QR from string). `expiryTimer` (ms) and `expiryDescription` come with it. 2C2P: keep repeating Transaction Status Inquiry until the customer pays or cancels.

**Payment slip (`TransactionPaymentSlip`, flow 1003)**: show the slip from `response.data`; status is pending until the customer pays offline. `additionalInfo` may carry `qrData`, `barcodeData`, `referenceNo`, `paymentExpiry`.

Source: https://developer.2c2p.com/docs/sdk-handle-pgw-payment-authentication, https://developer.2c2p.com/docs/sdk-references-handle-deeplink-payment-flow-by-pgw-sdk-helper, https://developer.2c2p.com/docs/sdk-helper-apis-interface, https://developer.2c2p.com/docs/sdk-helper-payment-enums, https://developer.2c2p.com/docs/sdk-method-qr-payment, https://developer.2c2p.com/docs/sdk-classes-apis-interface, https://developer.2c2p.com/docs/response-code-payment-flow

## Apple Pay (iOS)

Prerequisites:

1. Apple Developer: create a Merchant ID (recommended `merchant.com.{your.app.id}`).
2. Create an Apple Pay Payment Processing Certificate using a CSR obtained from the 2C2P PGW merchant portal or 2C2P merchant support; upload the resulting `apple_pay.cer` back to the 2C2P PGW merchant portal.
3. Enable Apple Pay on the App ID, add the Apple Pay capability in Xcode Signing & Capabilities, select the Merchant ID, add `PassKit.framework`.

Flow: token request with `["DPAY"]`; call Payment Option Details and pick the channel whose `context.code.channelCode == "APPLEPAY"` to get `context.info.paymentProvider`; then:

```swift
let providerRequest = PaymentProviderBuilder().paymentProvider(paymentProvider!).build()
PGWSDKHelper.shared.proceedApplePayPayment(paymentProviderRequest: providerRequest, { (r: ApplePayPaymentResultResponse) in
    guard r.responseCode == ApplePayPaymentResponseCode.TokenGeneratedSuccess else { return }
    let payment = DigitalPaymentBuilder(paymentCode: PaymentCode(channelCode: "APPLEPAY"))
        .name("<NAME>").email("<EMAIL>").token(r.token).build()
    // TransactionResultRequestBuilder(paymentToken:).with(payment) then proceedTransaction; expect TransactionCompleted
}) { (error: NSError) in /* show error */ }
```

`supportApplePayPayment(request, response)` checks device support. `ApplePayPaymentResponseCode`: `TokenGeneratedSuccess` / `SupportedDevice` 0000, `TokenGenerateFailed` 0001, `PaymentCancelled` 0002, `UnsupportedDevice` 0003, `ValidationFailed` / `InvalidPaymentResultResponseCallback` 0099. Result also carries optional `fxRateId`.

Source: https://developer.2c2p.com/docs/sdk-method-apple-pay, https://developer.2c2p.com/docs/sdk-references-apple-pay-prerequisite, https://developer.2c2p.com/docs/sdk-helper-apis-interface, https://developer.2c2p.com/docs/sdk-helper-payment-enums

## Google Pay (Android)

Prerequisites:

- Dependencies: `com.2c2p:pgw-sdk:4.7.1`, `com.2c2p:pgw-sdk-helper:4.2.1`, `com.google.android.gms:play-services-wallet:19.4.0`; for Google Pay inside a WebView also `androidx.webkit:webkit:1.14.0`.
- Google Pay inside WebView needs Google Play services 25.18.30+ and Android WebView (Chrome) 137+, `<queries>` intents `org.chromium.intent.action.PAY`, `IS_READY_TO_PAY`, `UPDATE_PAYMENT_DETAILS` in AndroidManifest.xml, and `WebSettingsCompat.setPaymentRequestEnabled(settings, true)` when `WebViewFeature.PAYMENT_REQUEST` is supported.
- Follow Google Pay brand guidelines; in the Google Pay console choose integration type "Gateway", upload app screenshots and submit for approval; production requires a release-signed app and production environment. Optional: join the Google Pay test card suite group.

Flow: token request with `["DPAY"]`; find `"GOOGLEPAY"` in Payment Option Details `channels`; then:

```kotlin
val providerRequest = PaymentProviderBuilder(activity).apply { paymentProvider(paymentProvider) }.build()
PGWSDKHelper.getInstance().proceedGooglePayPayment(providerRequest, object : PaymentResultResponseCallback<GooglePayPaymentResultResponse> {
    override fun onResponse(r: GooglePayPaymentResultResponse) {
        if (r.responseCode == GooglePayPaymentResponseCode.TokenGeneratedSuccess) {
            val payment = DigitalPaymentBuilder(PaymentCode("GOOGLEPAY")).apply { name("<NAME>"); email("<EMAIL>"); token(r.token) }.build()
            // proceedTransaction; handle AuthenticateRedirect (3DS) and Completed
        }
    }
    override fun onFailure(error: Throwable) {}
})
```

Overloads take a `googlePayAPIEnvironment` integer; `supportGooglePayPayment(...)` checks availability. `GooglePayPaymentResponseCode`: `TokenGeneratedSuccess` / `SupportedDevice` 0000, `TokenGenerateFailed` 0001, `PaymentCancelled` 0002, `UnsupportedDevice` 0003, `PaymentFailed` 0004, `PayloadReadyToRequestFailed` / `PayloadPaymentDataRequestFailed` 0005, 0099 validation and SDK failures. Result includes `token`, optional `fxRateId`, `googlePayResult` (`statusCode`, `message`, `paymentData`).

Source: https://developer.2c2p.com/docs/sdk-method-google-pay, https://developer.2c2p.com/docs/sdk-references-google-pay-prerequisite, https://developer.2c2p.com/docs/sdk-helper-apis-interface, https://developer.2c2p.com/docs/sdk-helper-payment-enums, https://developer.2c2p.com/docs/sdk-helper-payment-info-classes

## SDK API Methods

All are on `PGWSDK.getInstance()` / `PGWSDK.shared`, take a request built from `paymentToken`, and return `APISuccess` on success unless noted.

| Method | Request | Purpose / key fields |
| --- | --- | --- |
| `paymentOption` | `PaymentOptionRequest(paymentToken)` | Merchant config and enabled channels. Response: `merchantInfo`, `userInfo`, `channels` (categories), `transactionInfo`, `configurationInfo`. |
| `paymentOptionDetail` | `PaymentOptionDetailRequest(paymentToken)` with `categoryCode`, `groupCode` (M), e.g. `PaymentChannelCode.Category.GlobalCardPayment` + `PaymentChannelCode.Group.GlobalInstallmentPaymentPlan` | Channel list for a group: `name`, `iconUrl`, `validation`, `channels`, `totalChannel`. Source of Apple/Google Pay `paymentProvider`. |
| `proceedTransaction` | `TransactionResultRequest` (`payment` or `paymentUI`) | Do Payment. Response `TransactionResultResponse`. |
| `transactionStatus` | `TransactionStatusRequest(paymentToken)`, `additionalInfo = true` optional | Display-only status. Success branch checks `TransactionNotFound` or `TransactionCompleted`. Response: `channelCode`, `invoiceNo` (only when 2000), `additionalInfo` (merchant, transaction, `resultInfo`). |
| `customerTokenInfo` | `CustomerTokenInfoRequest(paymentToken)` | Stored cards list. |
| `exchangeRate` | `ExchangeRateRequest(paymentToken)`, `bin` optional | DCC rates: `providerCode`, `expire` (Unix time), `fxRates`. Use `fxRateId` in builders. |
| `userPreference` | `UserPreferenceRequest(paymentToken)` | Recent channels, recent transactions, locale (needs `clientId`). |
| `systemInitialization` | `SystemInitializationRequest()` | Locales, icons and other resources (PGW Initialization). |
| `paymentNotification` | `PaymentNotificationRequest(paymentToken)` with `platform`, `recipientId`, `recipientName` | Sends the receipt to the customer (Email, WhatsApp, Facebook, WeChat, Line). Not the merchant webhook. |
| `cancelTransaction` | `CancelTransactionRequest(paymentToken)`, `clientId` M | Lets the customer cancel an ongoing or pending transaction. Response: `channelCode`, `invoiceNo`. |
| `loyaltyPointInfo` | `LoyaltyPointInfoRequest(paymentToken)`, `providerId` M, optional `referenceId`, `cardNo`, expiry | Rewards list, terms, provider info. |

`TransactionStatusAdditionalInfo.resultInfo` (`PaymentResultAdditionalInfo`) fields: `redirectImmediately`, `autoRedirect`, `autoRedirectTimer` (ms), `frontendReturnUrl`, `frontendReturnData` (base64), `frontendRedirectMethod`, `responseCode` (`0000` success, `0001` failed, `0003` cancelled, `0999` general error), `responseDescription`. Use these to drive the result screen only.

Source: https://developer.2c2p.com/docs/sdk-api-payment-option, https://developer.2c2p.com/docs/sdk-api-payment-option-details, https://developer.2c2p.com/docs/sdk-api-do-payment, https://developer.2c2p.com/docs/api-sdk-transaction-status-inquiry, https://developer.2c2p.com/docs/sdk-api-pgw-initialization, https://developer.2c2p.com/docs/sdk-api-customer-tokens-information, https://developer.2c2p.com/docs/sdk-api-exchange-rate, https://developer.2c2p.com/docs/sdk-api-user-preference, https://developer.2c2p.com/docs/sdk-api-payment-notification, https://developer.2c2p.com/docs/sdk-api-cancel-transaction, https://developer.2c2p.com/docs/sdk-api-loyalty-point-information, https://developer.2c2p.com/docs/sdk-classes-apis-interface, https://developer.2c2p.com/docs/sdk-classes-payment-info

## TransactionResultResponse and Response Codes

`TransactionResultResponse` fields: `paymentToken` (M), `channelCode` (M), `invoiceNo` (M, only returned when responseCode is 2000), `type` (C, QR data type), `data` (M, URL / deep link / QR / slip depending on code), `fallbackData` (C, flow 1004), `expiryTimer` (C, ms, flow 1005), `expiryDescription` (C, flow 1005), `additionalInfo` (C), `responseCode`, `responseDescription`.

`APIResponseCode` enum, matched to the numeric Payment Flow Codes by their identical descriptions (the enum page itself does not print the numbers):

| Enum | Flow code | Action |
| --- | --- | --- |
| `TransactionAuthenticateRedirect` | 1000 | Load URL in iframe/WebView; close it when the result URL loads. |
| `TransactionAuthenticateFullRedirect` | 1001 | Full redirect to third-party page. |
| `TransactionExternalBrowser` | 1002 | Open external browser or app without callback; on resume call Transaction Status. |
| `TransactionPaymentSlip` | 1003 | Show payslip; status PENDING. |
| `TransactionExternalApplication` | 1004 | Open external app; app calls back. |
| `TransactionQRPayment` | 1005 | Show QR; poll Transaction Status. |
| `TransactionCompleted` | 2000 | Flow finished. Server confirms via backend notification or Payment Inquiry before showing paid. |
| `TransactionInProgress` | Not numbered on this page | Keep inquiring until a result. |
| `TransactionNotFound` | Not numbered on this page | Show result. |
| `TransactionInquiryFailed` | Not numbered on this page | Show result. |
| `APISuccess` | Not numbered on this page | Generic success for non-payment APIs. |

Any other code means failed or rejected: perform a payment inquiry (server) for full status. Numeric codes beyond the flow codes are in the Payment Response Codes list in `2c2p-api.md`.

Source: https://developer.2c2p.com/docs/sdk-classes-apis-interface, https://developer.2c2p.com/docs/sdk-classes-payment-enums, https://developer.2c2p.com/docs/response-code-payment-flow

## Other Enums

- `InstallmentInterestTypeCode`: `Customer`, `Merchant`.
- `QRTypeCode`: `All`, `Raw`, `Base64`, `URL`.
- `PaymentInputCode`: `Mandatory`, `Ignore`, `Optional` (field requirement hints from Payment Option Details).
- `PaymentNotificationPlatformCode`: `Facebook`, `WhatsApp`, `WeChat`, `Line`, `Email`.
- `PaymentCustomDataCode`: `Text`, `ImageUrl`, `WebUrl`, `FileUrl`.
- `PaymentChannelCode.Category`: `GlobalCardPayment`, `LocalCardPayment`, `LocalCardInstallmentPaymentPlan`, `DigitalPayment`, `QRPayment`, `WebPayment`, `SelfServiceMachine`, `PayAtCounter`, `InternetBanking`, `GlobalBuyNowPayLater`, `DeepLinkPayment`, `OnlineDirectDebit`, `DigitalCurrency`.
- `PaymentChannelCode.Group`: `CreditCard`, `ProprietaryCard`, `GlobalInstallmentPaymentPlan`, `LocalInstallmentPaymentPlan`, `GlobalThirdPartyPayment`, `LocalThirdPartyPayment`, `EWalletPayment`, `MasterPass`, `QRCodePayment`, `CardSchemeQRPayment`, `ThaiQRPayment`, `SingaporeQRPayment`, `WebPayment`, `AutomatedTellerMachine`, `Kiosk`, `BankCounter`, `OverTheCounter`, `InternetBanking`, `BuyNowPayLater`, `CardWalletPayment`, `DeepLinkPayment`, `CardLoyaltyPointPayment`, `OnlineDirectDebit`, `CryptoCurrency`.
- `PaymentChannelCode.Channel`: `LinePay`, `MoMoWallet`, `ShopeePay`, `OneTwoThree`, `GooglePay`, `CardLoyaltyPointPayment`, `ApplePay`, `ZaloPay`, `TripleAWallet`.
- `PaymentChannelCode.AgentChannel`: `WebPayment`, `AutomatedTellerMachine`, `Kiosk`, `BankCounter`, `OverTheCounter`, `InternetBanking`, `DeepLinkPayment`, `OnlineDirectDebit`.
- `ZaloPayPaymentResponseCode` (Vietnam only): `PaymentSuccess` 0000, `PaymentCancelled` 0001, `PaymentFailed` 0002, 0099 failures.

Source: https://developer.2c2p.com/docs/sdk-classes-payment-enums, https://developer.2c2p.com/docs/sdk-helper-payment-enums

## Key Classes (Summary)

- `PaymentCode(channelCode)` or `PaymentCode(channelCode, agentCode, agentChannelCode)`.
- Builders all accept optional `name`, `email`, `mobileNo`, `userAddress`, `paymentExpiry` (`yyyy-MM-dd HH:mm:ss`, later payments rejected), `customerNote`, `fxRateId`. Card builders add `expiryMonth`, `expiryYear`, `securityCode`, `pin`, `bank`, `country`, `tokenize`, `installmentInterestType`, `installmentPeriod`, `installmentPayLaterPeriod`, `loyaltyPoints`. Every builder also has a `(channelCode, ...)` string constructor fed from Payment Option Details.
- `UserBillingAddress`: `address1`, `address2`, `address3`, `city`, `state`, `postalCode`, `countryCode` (all M), wrapped by `UserAddressBuilder().userBillingAddress(...)`.
- `LoyaltyPoint` (`providerId`, `redeemAmount`, `rewards`) and `LoyaltyPointReward` (`id`, `quantity`), values from Loyalty Point Info.
- `PaymentProviderRequest`: Android constructor `(activity)`, iOS `uiViewController`, plus `paymentProvider`, `transactionResultResponse`, `fxRate`.
- `PaymentProvider`: `id`, `channels`, `merchantInfo`, `transactionInfo`, `terms`, `termsConsent`, `capabilities`.
- `TransactionInfo`: `amount`, `currencyCode`, `invoiceNo`, `productDescription`, `recurring`, `paymentItemInfos`, `showFXRate`.
- Full class list (validation, configuration, DCC, MCP, installment plan, merchant info, system locale, user preference and more) is on the Payment Info Classes page; these are read-only response models.

Source: https://developer.2c2p.com/docs/sdk-classes-payment-request, https://developer.2c2p.com/docs/sdk-classes-payment-info, https://developer.2c2p.com/docs/sdk-helper-apis-interface

## SecurePay SDK (Card Data Encryption for Do Payment API)

Use when the app collects card data but your server (not the PGW SDK) calls the Do Payment API. The SecurePay SDK turns card details into a secure pay token so raw PAN never reaches your server.

```kotlin
val sdk = SecurePaySDK(APIEnvironment.Sandbox)   // com.ccpp.pgw.sdk.android.securepay.APIEnvironment: Production, Sandbox
val payload = Payload().apply {
    cardNo = "4111111111111111"; expiryMonth = 12; expiryYear = 2026
    securityCode = "123"
    merchantId = "<MERCHANT_ID>"   // or paymentToken = "<PAYMENT_TOKEN>"; one of them is required
}
val secureToken = sdk.token(payload)  // send to server, server passes it to Do Payment
```

`Payload` fields: `cardNo`, `expiryMonth`, `expiryYear`, `issuedMonth`, `issuedYear`, `securityCode`, `pin`, `merchantId` (C), `paymentToken` (C), `maskedCardNo`. iOS: `SecurePaySDK(apiEnvironment: SecurePayAPIEnvironment.Sandbox)`, `SecurePayPayload`. Version 2.1.0 added `merchantId` and `paymentToken` to the payload.

Source: https://developer.2c2p.com/docs/sdk-securepay-initialize, https://developer.2c2p.com/docs/sdk-securepay-apis-interface, https://developer.2c2p.com/docs/sdk-securepay-payment-request, https://developer.2c2p.com/docs/sdk-securepay-payment-enums, https://developer.2c2p.com/docs/sdk-download-securepay

## Mobile Integration Checklist

1. Server creates the payment token (JWT HS256) with `backendReturnUrl` and stores a pending order keyed by `invoiceNo`.
2. App initializes the SDK with the environment matching the token.
3. App runs Payment UI or a custom builder flow and handles every response code (WebView with `PGWWebViewClient` / `PGWWebViewNavigationDelegate`, Helper for deep links, QR polling, payslip pending).
4. App calls Transaction Status Inquiry only to show a result screen, then asks your server.
5. Server marks paid only from a verified backend notification or a server-side Payment Inquiry with matching invoice, amount, currency and `respCode` `0000`, idempotently.
6. iOS: URL scheme `pgw{bundle id}`, `universalPaymentResultObserver`, Apple Pay cert uploaded to the 2C2P portal. Android: Google Pay "Gateway" approval and release signing.
7. Keep SDK versions out of EOL.

Source: https://developer.2c2p.com/docs/sdk-how-it-work, https://developer.2c2p.com/docs/sdk-handle-pgw-payment-authentication, https://developer.2c2p.com/docs/sdk-references-handle-deeplink-payment-flow-by-pgw-sdk-helper, https://developer.2c2p.com/docs/sdk-download-sdk
