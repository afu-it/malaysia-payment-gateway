# KineticPay Source Map

Use this map to pick the right source before answering or coding. All checked 2026-10-21.

## Coverage

- Index used: the docs site navigation in its Next.js build manifest (`/_next/static/<build>/_buildManifest.js`) and the nav config in the `_app` bundle. The site has no `llms.txt`, `sitemap.xml`, OpenAPI or Postman file (all 404).
- Routes enumerated in the manifest: 12 content routes (`/`, `/authentication`, `/banks`, `/companies`, `/contacts`, `/conversations`, `/errors`, `/groups`, `/messages`, `/pagination`, `/quickstart`, `/sandbox`) plus framework routes `_app` and `_error`. Pages read: 12 of 12.
- `/contacts`, `/conversations`, `/groups`, `/messages` are leftover template pages (chat-app examples, not linked from the nav, not KineticPay payment features). Read and excluded.
- Nav entries with no page (`href: "/#"`): Bills, Collections, Collections Form, Transactions, Settlements, Points. Nothing to read; marked as not documented.
- Because the payment flow is not on the docs site, the official WordPress.org plugins were read in full for API calls: 4 plugins, all PHP files that call KineticPay.
- GitHub org `https://github.com/kineticpay-my`: 0 public repositories.

## Product And Account

- Homepage: `https://kineticpay.my/`
- Docs home: `https://docs.kineticpay.my/`
- Register: `https://manage.kineticpay.my/register`
- API key page: `https://manage.kineticpay.my/api-key`

Use for: product positioning, payment methods, settlement claim, pricing link, registration.

## API v2 Guides

- Quickstart: `https://docs.kineticpay.my/quickstart`
- Authentication: `https://docs.kineticpay.my/authentication`
- Pagination: `https://docs.kineticpay.my/pagination`
- Errors: `https://docs.kineticpay.my/errors`

Use for: base URL `https://manage.kineticpay.my/api/v2`, Bearer auth, required headers, `/ping`, rate limit 60/min and headers, HTTP status codes, `page`/`perPage`/`meta.pagination`.

## API v2 Resources

- FPX Banks: `https://docs.kineticpay.my/banks`
- Companies: `https://docs.kineticpay.my/companies`

Use for: `GET /bank` fields and bank codes, `GET /company` fields and company approval.

## Sandbox

- Sandbox: `https://docs.kineticpay.my/sandbox`

Use for: sandbox toggle on the Collection page, same endpoints as live.

## Payment Flow (bill create, callback, requery)

- WooCommerce plugin page: `https://wordpress.org/plugins/kineticpay-for-woocommerce/`
  - Download: `https://downloads.wordpress.org/plugin/kineticpay-for-woocommerce.zip` (3.0)
  - `src/kineticpay.php`: `/payment/create` fields, `html` response, `encoded_transaction_data` callback with `record.transaction_status_code`.
  - `index.php`: FPX bank list, `/payment/status` requery with `code` and `id`.
  - `readme.txt`: merchant key location (Collection page), setup steps.
- GiveWP plugin: `https://wordpress.org/plugins/kineticpay-for-givewp/` (`includes/Kineticpay_WPConnect.php`, `includes/give-kineticpay-gateway.php`)
- Gravity Forms plugin: `https://wordpress.org/plugins/kineticpay-for-gravityforms/` (`helpers/kineticpay_wpconnect.php`, `class-gf-kineticpay.php`, `helpers/class-gf-field-kineticpay.php`)
- Forminator plugin: `https://wordpress.org/plugins/kineticpay-for-forminator/` (`classes/kineticpay-gateway.php`, `classes/public.php`)

Use for: create bill request fields, amount format, invoice pattern, callback URLs, requery-on-return pattern, success code `"00"`, response handling, bank code list.

## Observed Behaviour (not documented, probed read-only 2026-10-21)

- `GET https://manage.kineticpay.my/payment/status?merchant_key=INVALID&invoice=TEST1` returned HTTP 200 with `{"error":{"merchant_key":["The selected merchant key is invalid."],"invoice":["The selected invoice is invalid."]}}`.
- `GET https://manage.kineticpay.my/api/v2/ping` without a key returned HTTP 401 `{"message":"Unauthenticated."}`.

Use for: error shape handling. Re-check before relying on it.

## Not Covered By Any Official Public Source

Refunds, cancel/void, capture, recurring/tokenisation, payouts, signed webhooks, settlement report API, Bills/Collections/Transactions/Settlements/Points APIs, card and e-wallet fields, full status code list, full status response schema, full callback payload, sandbox test data, IP allowlist setup, WHMCS module source. Confirm with `support@kineticpay.my`.
