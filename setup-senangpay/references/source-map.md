# senangPay Source Map

Use this map to pick the right senangPay source before answering or coding.

## Primary Developer Sources

| Use case | Source |
| --- | --- |
| Developer tools index | `https://guide.senangpay.com/developer-tools` |
| Hosted/manual shopping cart API | `https://guide.senangpay.com/manual-integration-api` |
| Merchant ID, Secret Key, Hash Type | `https://guide.senangpay.com/merchant-id-and-secret-key` |
| Callback URL behavior and `OK` response | `https://guide.senangpay.com/what-is-callback-url-and-how-to-set-it` |
| Return URL parameters and custom placeholders | `https://guide.senangpay.com/return-url` |
| Query by order ID | `https://guide.senangpay.com/query-order-status` |
| Query by transaction reference | `https://guide.senangpay.com/query-transaction-status` |
| Direct API and JavaScript Web SDK | `https://guide.senangpay.com/direct-api` |
| Refund API | `https://guide.senangpay.com/refund-api` |
| Sandbox overview | `https://guide.senangpay.com/testing-sandbox` |

## Product and Business Sources

| Use case | Source |
| --- | --- |
| Social selling, payment forms, digital payments, tap-to-phone | `https://senangpay.com/social-selling/` |
| Online store, plugins, e-commerce platforms, Direct API, recurring, mass payments | `https://senangpay.com/online-store/` |
| Enterprise, Payout API, tokenisation, split payments | `https://senangpay.com/enterprise-solutions/` |
| Pricing, package features, transaction rates | `https://senangpay.com/pricing/` |
| Video guides | `https://senangpay.com/edu-videos/` |

## Guide Categories

| Use case | Source |
| --- | --- |
| Platform migration | `https://guide.senangpay.com/senangpay-to-doku-platform-migration` |
| Company/about context | `https://guide.senangpay.com/about-us` |
| Registration and documents | `https://guide.senangpay.com/getting-started` |
| Packages, activation, account settings | `https://guide.senangpay.com/account-and-subscriptions` |
| Charges, payment methods, settlement, billing | `https://guide.senangpay.com/payments-settlements-and-charges` |
| Payment and technical troubleshooting | `https://guide.senangpay.com/troubleshooting` |
| Dashboard and reporting | `https://guide.senangpay.com/dashboard-and-reporting` |
| Fraud, security, disputes | `https://guide.senangpay.com/frauds-and-disputes` |
| E-commerce platforms, payment forms, web plugins | `https://guide.senangpay.com/setup-and-integrations` |
| Payments and refunds | `https://guide.senangpay.com/payments-and-refunds` |
| Support channels and sunsetted products | `https://guide.senangpay.com/support-and-feedback` |
| Prohibited items, SSM, terms | `https://guide.senangpay.com/rules-and-regulations` |

## Integration Choice Map

- If the app has a custom checkout and can redirect users, start with Manual Integration API.
- If the merchant uses WooCommerce, OpenCart, Magento, Joomla/Hikashop, Shopify, SHOPLINE, EasyStore, SiteGiant, OnPay, PrestaShop, Ordersini, Orderla, or WhatsMenu, read setup/integration guides before writing custom code.
- If the merchant asks for no hosted form or direct selected-payment-method checkout, read Direct API and verify Enterprise/authorised/IP whitelist requirements.
- If the merchant needs subscriptions or scheduled charges, read recurring payment API references from Developer Tools.
- If the merchant needs marketplace-like fund distribution, read split settlement and Enterprise solution sources.
- If the merchant needs payouts, read Payout API and Enterprise solution sources.
- If the merchant needs refunds, read Refund API and payments/refunds sources.
- If the merchant asks about fees, package capability, channel support, settlement timing, or dashboard state, verify live pricing/dashboard/support.

## Security Source Map

- Hash Type and Secret Key: `merchant-id-and-secret-key`.
- Request hash and return/callback verification: `manual-integration-api`.
- Callback delivery, duplicate callbacks, final callback timing, and exact `OK`: `what-is-callback-url-and-how-to-set-it`.
- Custom return/callback placeholders and deprecated MD5 note: `return-url`.
- Direct API server-only session hash and UUID expiry: `direct-api`.
- Refund duplicate/window/partial support errors: `refund-api`.
