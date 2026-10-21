# AhaPay Account Setup

Use this reference when explaining AhaPay merchant registration, credentials, sandbox vs production, plugin setup, limits, fees, payout, refunds, and support. Facts checked on 2026-10-21.

## Source URLs

- `https://ahapay.my/merchants` (merchant page and FAQ)
- `https://apidocs.ahapay.my/` (developer portal home)
- `https://apidocs.ahapay.my/docs/api/v2/Authentication`
- `https://apidocs.ahapay.my/docs/api/Callback-Security`
- `https://apidocs.ahapay.my/docs/platforms/WooCommerce/`
- `https://apidocs.ahapay.my/docs/platforms/EasyStore/`
- `https://wordpress.org/plugins/ahapay-buy-now-pay-later/` (official plugin readme and source)
- `https://portal.ahapay.my/login` (merchant portal login)
- `https://merchant-help.ahapay.my/hc/en-my` (merchant help center)

## Product Positioning

AhaPay is a Buy Now Pay Later service for Malaysian offline and online stores, operated by Aharetail Solutions Sdn. Bhd. (company registration 202401018053, Bangsar South, Kuala Lumpur).

- Customer pays in instalments; the merchant FAQ describes 3 months interest-free, the merchant page shows 4 equal payments with the first paid at checkout, and the plugin readme (v1.1.0) says 4 to 12 instalments with a client fee added on longer plans. Do not quote a fixed plan to customers; check current terms with AhaPay.
- Merchant receives the full order amount; AhaPay takes the non-payment risk ("you will receive the full amount for the order").
- In store: no special hardware. The merchant enters the amount in the AhaPay Agent app and shows the generated QR; the customer scans it in the AhaPay app.
- Online: API (`apidocs.ahapay.my`), WooCommerce plugin, EasyStore app. Shopify, Magento, PrestaShop, OpenCart and Wix plugins are listed as "Coming Soon".

## Eligibility And Documents

From the merchant FAQ:

1. The business must be legal, non-gambling, non-alcoholic, not in high-risk categories, and pass AhaPay's risk approval.
2. Documents:
   - SSM report (written "SMM report" on the page);
   - PnL for the last 3 to 6 months, or
   - transaction statement for the last 3 to 6 months.

## Registration

1. Go to `https://ahapay.my/merchants` ("For Business") and use "Connect" to sign up.
2. Fill in the business details.
3. AhaPay reviews the application and contacts you with next steps.
4. AhaPay sets your per-merchant transaction limit based on business type, products, monthly volume and average ticket.
5. During onboarding AhaPay issues the API key ("The API Key given while onboarding"). There is no self-serve key page documented.

## Credentials

| Credential | Where it comes from | Notes |
| --- | --- | --- |
| API key (Merchant / Trade Outlet) | Issued by AhaPay at onboarding; plugin docs say "Obtain API key from AhaPay merchant portal" | Separate keys for sandbox and production. Also the secret behind the callback `X-Api-Key-Hash` |
| Partner API key | Issued to partner platforms | Requests must include `external_merchant_id` |
| `external_merchant_id` | The partner's own internal ID for each merchant | Leave empty with a Merchant key |
| Merchant ID (EasyStore) | Provided by AhaPay | EasyStore asks for Merchant ID and API key |

Suggested env vars:

```text
AHAPAY_BASE_URL=https://api.bnpl-my.dev.mychili.id   # sandbox; production https://api.ahapay.my
AHAPAY_API_KEY=<AHAPAY_API_KEY>
AHAPAY_EXTERNAL_MERCHANT_ID=<PARTNER_MERCHANT_ID>     # partner keys only
```

Never commit the key. The WooCommerce plugin stores it encrypted (AES-256-CBC) and masks it in the settings field.

Key errors: 401 means an outdated or archived key, or a partner not bound to the trade outlet; 404 means outlet or partner not found; 500 means invalid key format or a key that was never created. Use `GET /v2/health` with the key to check it before going live.

## Sandbox vs Production

| | Sandbox | Production |
| --- | --- | --- |
| API base | `https://api.bnpl-my.dev.mychili.id` | `https://api.ahapay.my` |
| API key | Sandbox key from AhaPay | Live key from AhaPay |
| Plugin setting | Environment "Test" (`sandbox`, default) | Environment "Production" (`live`) |

No public sandbox signup, test customers or test data are published. Ask AhaPay support for sandbox access and a test customer.

## WooCommerce Setup (official plugin)

Requirements: WordPress 5.4+, WooCommerce 4.8+, PHP 7.4+ with OpenSSL, SSL certificate (production), store currency MYR.

1. Install "AhaPay Buy Now Pay Later" (`ahapay-buy-now-pay-later`) from Plugins > Add New, or upload the ZIP.
2. Activate it.
3. Go to WooCommerce > Settings > Payments > AhaPay Buy Now Pay Later > Manage.
4. Set Enable, Title, Description, Environment (Test or Production), API Key.
5. External Merchant ID: only for Partner API keys; leave empty for Trade Outlet keys.
6. Save. The plugin calls `GET /v2/health`; a green "Verified" status means the key works. A failed check does not save the new credentials.
7. Set Success Page and Error Page (browser return pages).
8. Optional: product labels, label colours, minimum amount (default 55 MYR), custom checkout CSS, debug mode.
9. Make sure `https://<your-site>/wp-json/ahapaybuynowpaylater/v1/callback` is publicly reachable (no auth wall, WAF block or cache).

Troubleshooting from the docs:

- Gateway not shown: store currency is not MYR, API key not set or invalid, or cart below the minimum.
- Red API status: check connectivity and key validity; logs are in WooCommerce > Status > Logs with Debug Mode on.
- Status not updating: callback URL not reachable or blocked.

## EasyStore Setup

1. EasyStore admin > Settings > Payment.
2. Search for AhaPay and add it.
3. Enter the Merchant ID and API Key provided by AhaPay.
4. Enable it and Save.

Get the API key from AhaPay support before starting.

## Limits

- Customer purchase limit: RM50 to RM3,500, varies by merchant (merchant FAQ).
- WooCommerce plugin default minimum: 55 MYR (editable).
- Optional daily sales limit agreed with AhaPay; once reached, all orders are rejected until 12am the next day. Check with `GET /v2/daily_limit_status`.
- Customers may fail checkout if they exceed their AhaPay spending limit, are not eligible, or have a frozen plan after missed repayments.

## Fees

AhaPay charges only a Merchant Discount Rate (MDR) per transaction, and from 1 June 2026 SST applies on the MDR amount. Do not hard-code the rate; read the current figure and worked example on the official merchant FAQ: `https://ahapay.my/merchants`. The FAQ says the fee covers internet acquiring, promotion in the AhaPay app, joint marketing, and non-return risk.

## Payout And Settlement

- Funds are paid to the merchant's bank account on an agreed schedule (merchant FAQ). The schedule is not published.
- The merchant dashboard shows sales and AhaPay disbursements, not customer repayments.
- No settlement report API is documented.

## Refunds

- The merchant approves the refund; AhaPay then returns money to the customer (merchant FAQ).
- Via API: `POST /v2/orders/{online_order_id}/refund`, full order only.
- Consumer refund policy: `https://ahapay.my/refund-policy`.

## Support

- Merchant email: `merchant@ahapay.my` (developer portal "Email Us").
- Merchant help center: `https://merchant-help.ahapay.my/hc/en-my`.
- Merchant portal: `https://portal.ahapay.my/login`.
- The FAQ states 24/7 support via email and live chat. When asking for help, give your Merchant ID and details of the issue.
