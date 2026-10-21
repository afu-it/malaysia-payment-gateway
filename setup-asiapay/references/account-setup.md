# AsiaPay Account Setup

Use this reference when explaining AsiaPay Malaysia registration, merchant admin setup, sandbox vs production, credentials, Malaysian payment methods, pricing and support.

## Source URLs

- `https://www.asiapay.com.my/`
- `https://www.paydollar.com/en/contact-us.html`
- `https://developer.asiapay.com/integration-guide/transaction-security-by-secure-hash/enable-secure-hash`
- `https://developer.asiapay.com/integration-guide/data-feed-handling/data-feed-setup`
- `https://developer.asiapay.com/integration-guide/functions-of-merchant-api/introduction`
- `https://developer.asiapay.com/integration-guide/mobile-integration/tokenization-flow/get-started-with-paydollar-api`
- `https://developer.asiapay.com/integration-guide/payment-methods-by-regions-countries`
- `https://developer.asiapay.com/integration-guide/faq/common-problems`
- `https://developer.asiapay.com/integration-guide/faq/support`
- `https://github.com/asiapay-lib/asiapay-Woocommerce` (README)

## Who AsiaPay Malaysia Is

- AsiaPay Malaysia is a wholly owned subsidiary of AsiaPay Limited (Hong Kong), established in 2011. Its gateway product is PayDollar (`https://www.asiapay.com.my/`).
- Kuala Lumpur office: Business Suite 19A-17-3 & 3A, Level 17, UOA Centre, No. 21, Jalan Pinang, 50450 Kuala Lumpur. Tel +603 2161 8188. Sales email `sales@asiapay.com.my`.
- The site lists Visa, Mastercard, American Express, JCB, UnionPay, Alipay, Boost, GrabPay, DuitNow, FPX, ShopBack, TNG Wallet, TrueMoney, Atome, SPayLater and PayLater by Grab.

## Registration

There is no self-serve sign-up flow on asiapay.com.my (checked 2026-10-21; its "Get Started" buttons are placeholders). Contact sales:

1. Email `sales@asiapay.com.my` or call the KL office, or use `https://www.paydollar.com/en/contact-us.html`.
2. Ask for a PayDollar test merchant account and the integration guide.
3. Provide business and KYC documents as requested by AsiaPay. The document list is not published (checked 2026-10-21).
4. Agree pricing, settlement currency (MYR), payment methods and connection method.
5. After testing, AsiaPay issues the production merchant ID and production admin access.

## Merchant Administration

- Test admin: `https://test.paydollar.com/b2cDemo/eng/merchant/index.jsp`.
- Production admin: `https://admin.paydollar.com/b2c2/eng/merchant/index.jsp`.
- Test merchant IDs are 6 digits. Production merchant IDs are 4 or 8 digits. Each pairs only with its own environment URLs.
- One merchant ID accepts one currency. Ask sales for another merchant ID for a second currency.

## Credentials And Where They Live

| Item | Where | Notes |
| --- | --- | --- |
| Merchant ID | Issued by AsiaPay | Public value, posted in every form. |
| Secure Hash Secret | Admin > Profile > Payment Information | AsiaPay must enable Secure Hash first (email `service@paydollar.com`). Rotated every 2 years. Server-side only. |
| Hash algorithm | Account setting (SHA-1 default, SHA-256 optional) | Confirm with AsiaPay which one your account uses. |
| Merchant API login ID and password | Issued by AsiaPay on request by email | Used for Query, Capture, Void, Refund, Cancel, settlement report. Server-side only. |
| PaySDK client secret and RSA public key | Admin > Profile > Account Information > Payment Account Information | PaySDK requires your iOS bundle ID and Android application ID registered with AsiaPay. |
| Client library and sample code | Admin > Support > Developer Corner | Java, PHP, ASP, ASP.NET hash helpers. |

## Datafeed Setup

1. Admin > Profile > Payment Options (the WooCommerce README path is Profile > Profile Settings > Payment Options).
2. Enable "Return Value Link" and enter the datafeed URL, e.g. `https://shop.example/api/asiapay/datafeed`. Port 80 or 443 only.
3. Optionally enable "Auto retry failed data feed" and "Data Feed Failure Alert by Email".
4. Confirm delivery in Transaction Details with output columns "Data Feed Ind." and "Data Feed Return".
5. Repeat separately for the test and production accounts.

## Other Admin Operations

- Capture or reverse hold transactions: Admin > Operation > Transaction Detail.
- Iframe embedding of the payment page: give AsiaPay the domain to enable.
- API IP whitelisting: give AsiaPay your server IPs if required.
- Technical support: `it@paydollar.com`, hotline (852) 8226 7981 or (852) 3173 1990 (Hong Kong). Give PayRef (preferred) or Ref.

## Environment Variables

Use these names unless the app already has a convention:

```text
ASIAPAY_ENV=test                      # test | production
ASIAPAY_BASE_URL=https://test.paydollar.com/b2cDemo   # production: https://www.paydollar.com/b2c2
ASIAPAY_MERCHANT_ID=<MERCHANT_ID>
ASIAPAY_CURRENCY_CODE=458             # MYR
ASIAPAY_SECURE_HASH_SECRET=<SECURE_HASH_SECRET>
ASIAPAY_HASH_ALGO=sha1                # sha1 | sha256, must match the account
ASIAPAY_API_LOGIN_ID=<API_LOGIN_ID>
ASIAPAY_API_PASSWORD=<API_PASSWORD>
ASIAPAY_DATAFEED_URL=https://shop.example/api/asiapay/datafeed
```

Keep test and production values in separate environments. Never commit them.

## Payment Methods In Malaysia

The developer portal Malaysia table lists: cards (Amex, China UnionPay, Discover, JCB, Mastercard, Visa), FPX, e-wallets (Alipay, WeChat Pay, PayPal, Boost, Touch 'n Go, Scan & Pay by MAE, GrabPay, ShopBack Pay), BNPL (Atome, PayLater by Grab).

- Activation of each method is done by AsiaPay on your account; it is not self-serve in the docs.
- Documented `payMethod` codes for Malaysian methods include `FPX`, `DuitNow`, `TouchnGo`, `GRABPAY`, `ATOME`, `M2U`, `CIMBCLICK`, `MEPS`, `SHOPEEPAY`, card codes and `ALL`. No code is published for Boost, MAE, ShopBack Pay, PayLater by Grab or SPayLater (checked 2026-10-21); use `ALL` or ask AsiaPay.
- FPX bank list and DuitNow QR specifics are not documented publicly (checked 2026-10-21).

## Pricing And Settlement

- No public Malaysian rate card. Pricing is quoted by sales: contact `sales@asiapay.com.my` or `https://www.paydollar.com/en/contact-us.html`. Do not quote numbers from memory.
- Settlement timing and payout schedule are not documented in the developer docs (checked 2026-10-21). Use the GenTxnXML settlement report (`settleflag`, `settledate`, `batchid`) for reconciliation.
- Voids work only before the settlement cut-off; `RequestRefund` is documented as within 14 days; holds must be captured within 14 days.

## Plugins

Official plugins are published at `https://github.com/asiapay-lib` for WooCommerce, Magento 1.x/2.x, OpenCart, PrestaShop, Shopify, Drupal, OsCommerce, ZenCart, VirtueMart, UberCart, LiteCommerce and Ecshop, plus PaySDK Android/iOS libraries and demos. The WooCommerce plugin uses Client Post Through Browser and the datafeed URL `https://<site>/index.php?wc-api=wc_paydollar`; WooCommerce 8.3+ needs Classic Checkout per its README.
