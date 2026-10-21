# SPayLater Promotional Messaging (Widget, SDKs, Shopify App)

Use this file when a merchant wants "Buy Now, Pay Later with SPayLater" / "As low as RM X/mth" messaging on product, category, home or cart pages. The widget is display only: it does not take payment. Payment still goes through the ShopeePay APIs in `spaylater-api.md`.

Sources (checked 2026-10-21):

- `https://product.shopeepay.com/products/functionalities/spl-promotional-widget/about`
- `https://product.shopeepay.com/products/functionalities/spl-promotional-widget/web`
- `https://product.shopeepay.com/products/functionalities/spl-promotional-widget/android`
- `https://product.shopeepay.com/products/functionalities/spl-promotional-widget/ios`
- `https://product.shopeepay.com/products/shopify-plug-in/spaylater-on-site-messaging-app`
- `https://product.shopeepay.com/products/brand-guidelines`

## Concepts

- Figures are calculated from a default rate fixed per market and applied to every merchant. No personalized prequalification, no customer data in the widget.
- Placements: Homepage (banner, no price), Category/listing (short instalment line per item), Product page (instalment line + info icon that opens the tenor pop-up), Cart (same as product page).
- Compliance (confirm with the ShopeePay account manager and your legal team): no guaranteed-approval wording; SPayLater must already be live at your checkout before you show the widget; amounts are illustrative examples; do not change the default copy (centrally reviewed per market); custom copy needs ShopeePay legal sign-off.

## Credentials

- Access Key (SPL plugin key) and Store ID (`store_ext_id`) from the ShopeePay account manager. Same Access Key for test and live (stated on Android and iOS pages).
- Country, currency and language must match the market of your SPayLater agreement, or nothing renders.

## Web

Loader script (pick one):

```html
<!-- non-production -->
<script src="https://deo.shopeemobile.com/shopee/test/shopee-shopeepayfe-test-global/spl-widget/loader.js"></script>
<!-- production -->
<script src="https://deo.shopeemobile.com/shopee/shopee-shopeepayfe-live-global/spl-widget/loader.js"></script>
```

Init once per page:

```html
<script>
  SPLWidget.init({ access_key: '<SPL_ACCESS_KEY>', store_ext_id: '<STORE_EXT_ID>', currency: 'MYR', language: 'en', country: 'MY' });
</script>
```

| `SPLWidget.init` param | Type | Required | Notes |
| --- | --- | --- | --- |
| `access_key` | string | Yes | SPL plugin key. Public in the page by design. |
| `store_ext_id` | string | Yes | ShopeePay store external ID. |
| `currency` | string | Yes | ISO 4217, `MYR`. |
| `language` | string | Yes | BCP 47, e.g. `en`. |
| `country` | string | Yes | ISO 3166-1 alpha-2, `MY`. |

Anchors:

```html
<div data-spl-widget="true" data-page-type="home_page"></div>
<div data-spl-widget="true" data-page-type="product_page" data-amount="50000" data-product-id="SKU-001"></div>
```

| Attribute | Default | Notes |
| --- | --- | --- |
| `data-spl-widget="true"` | | Recommended marker. Legacy `class="spl-widget"` still works; do not use both. |
| `data-page-type` | `product_page` | `home_page`, `category_page`, `product_page`, `cart_page`. Tenor pop-up only on `product_page` (cart same as product per docs). |
| `data-amount` | 0 | Price x100 (RM 500.00 = `50000`). Required for category and product pages. |
| `data-product-id` | auto | SKU. Required for category and product pages. |

SPA: call `SPLWidget.refresh()` after client-side navigation or data change. `SPLWidget.onModalClose(cb)` runs when the tenor modal closes.

CSP additions:

```text
script-src  'self' https://deo.shopeemobile.com;
connect-src 'self' https://*.shopeemobile.com https://*.shopeepay.com;
img-src     'self' data: https://*.shopeemobile.com;
frame-src   'self' https://*.shopeepay.com;
```

Behaviour: about 12 KB, async, Shadow DOM (no CSS leakage, styling not configurable), self-hides if pricing cannot load. Modern Chrome, Firefox, Safari, Edge (latest 2 majors); no IE. No React/Vue component; use the script.

Troubleshooting: blank means script blocked or CSP; no price means wrong country/currency or missing `data-amount`; disappears after navigation means missing `refresh()`; works in test not live means still on test loader or store not activated for live.

## Android

- SDK `shopeepay-merchant-sdk.aar` from the account manager (not on Maven). Put in `libs/`, add `implementation files("libs/shopeepay-merchant-sdk.aar")`.
- Dependencies: `com.squareup.okhttp3:okhttp:3.12.1`, `com.google.code.gson:gson:2.10.1`, `com.github.bumptech.glide:glide:4.13.2`, `androidx.core:core-ktx:1.10.1`, `androidx.recyclerview:recyclerview:1.3.2`, `androidx.constraintlayout:constraintlayout:2.1.4`.
- Init in `Application.onCreate()`:

```kotlin
val config = ShopeePayMerchantConfig.Builder()
    .context(applicationContext).accessKey("<SPL_ACCESS_KEY>")
    .env(Environment.SANDBOX) // Environment.LIVE for production
    .country(Country.MY).currency(Currency.MYR).language(Language.EN).build()
ShopeePayMerchantSdk.init(config)
```

- View: `com.shopeepay.merchant.sdk.biz.installment.ShopeePayMerchantView`; call `merchantView.render(storeExtId = "<STORE_EXT_ID>", productId = "sku", amount = 129900L, pageType = ShopeePayMerchantPageType.PRODUCT_PAGE)`. Page types `HOME_PAGE`, `CATEGORY_PAGE`, `PRODUCT_PAGE`. `productId` and `amount` not needed on home.
- RecyclerView lists: call `ShopeePayMerchantSdk.requestData(SPPMerchantInfoRequest(storeExtId, pageType = CATEGORY_PAGE, productInfoList = listOf(SPPMerchantInfo(productId, amount))))` once, then `render()` per row from cache.
- Ineligible region or no program: view collapses to zero height.

## iOS

- iOS 12+, Xcode 15+. CocoaPods `SDWebImage`, `Masonry`.
- XCFrameworks from the account manager: `ShopeePayKit.xcframework`, `SDBProtectFramework.xcframework`; set Do Not Embed (static), Other Linker Flags `-ObjC`.
- Link: UIKit, Foundation, NetworkExtension, DeviceCheck, StoreKit, `libresolv.9.tbd`, `libc++.tbd`.
- Init in `application:didFinishLaunchingWithOptions:`:

```objc
SPLConfiguration *c = [[SPLConfiguration alloc] initWithAccessKey:@"<SPL_ACCESS_KEY>" storeExtId:@"<STORE_EXT_ID>"
  country:SPLCountryMY currency:SPLCurrencyMYR language:SPLLanguageEN environment:SPLEnvironmentUAT]; // SPLEnvironmentLive for release
[[ShopeePayLaterKit sharedKit] configureWithConfiguration:c];
```

- Views: `SPLBannerView` (home); `SPLInstallmentMessageView` with `loadWithProductId:@"sku" amount:@10000 pageType:SPLPageTypeProduct` (or `SPLPageTypeCategory`). Amount in smallest unit.
- Debug: `[[ShopeePayLaterKit sharedKit] setLogEnabled:YES]` (disable for release). Swift works through the Objective-C API.

## Shopify

1. Install the ShopeePay Payment App first (Malaysia: `https://apps.shopify.com/shopeepay-malaysia`), activate with the 32-digit merchant Access Key, keep Test Mode disabled for live, and toggle off any channel (such as SPayLater) not in your agreement.
2. Then install the SPayLater On-Site Messaging app from the Shopify App Store. It only works after ShopeePay is live at checkout.
3. Theme editor: Products > Default product > Product information > Price > Add block > "SPayLater on-site message". Cart: set cart type to "Page", then Subtotal > Add block. Text size and logo size are adjustable separately for PC and mobile web.
4. Store currency and billing address must match the ShopeePay account region (Settings > General).

## Branding

SPayLater logo minimum sizes: horizontal 60 px screen / 20 mm print, vertical 50 px / 15 mm, icon 20 px / 10 mm; 0.5x clear space (0.25x for icon). Acceptance mark: rectangle 35 px / 15 mm, square 35 px / 8 mm. Do not use the standalone "SPayLater" logotype. Show SPayLater in the same format as other payment options. Branding PDF: `https://product.shopeepay.com/static/files/ShopeePay_SPayLater_Online_Branding_Guide.pdf`.
