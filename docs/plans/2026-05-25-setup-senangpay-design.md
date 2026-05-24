# Setup SenangPay Skill Design

Date: 2026-05-25

## Goal

Create a complete `setup-senangpay` Kilo skill package for integrating senangPay payments in Malaysian apps. The skill should give agents enough source-backed context to build, debug, review, and explain senangPay payment flows without inventing account-specific facts.

## Scope

Add a provider package matching the existing gateway-skill pattern:

- `setup-senangpay/SKILL.md`
- `setup-senangpay/agents/openai.yaml`
- `setup-senangpay/references/senangpay-api.md`
- `setup-senangpay/references/account-setup.md`
- `setup-senangpay/references/source-map.md`
- Root `README.md` skill table and prompt examples

The package should cover hosted/manual integration, Direct API, return URL, callback URL, query APIs, refund API, sandbox, dashboard credentials, plugins/e-commerce platforms, recurring, payout, tokenisation, split settlement, pricing/package caveats, and DOKU migration pointers.

## Source Material

Use these public senangPay sources as the basis for references:

- `https://senangpay.com/social-selling/`
- `https://senangpay.com/online-store/`
- `https://senangpay.com/enterprise-solutions/`
- `https://senangpay.com/pricing/`
- `https://guide.senangpay.com/developer-tools`
- `https://guide.senangpay.com/manual-integration-api`
- `https://guide.senangpay.com/direct-api`
- `https://guide.senangpay.com/merchant-id-and-secret-key`
- `https://guide.senangpay.com/return-url`
- `https://guide.senangpay.com/what-is-callback-url-and-how-to-set-it`
- `https://guide.senangpay.com/query-order-status`
- `https://guide.senangpay.com/query-transaction-status`
- `https://guide.senangpay.com/refund-api`
- `https://guide.senangpay.com/testing-sandbox`
- `https://guide.senangpay.com/getting-started`
- User-provided guide categories: account/subscriptions, payments/settlements/charges, troubleshooting, dashboard/reporting, frauds/disputes, setup/integrations, payments/refunds, support/feedback, rules/regulations, migration, videos

## Key Product Facts

senangPay has several product surfaces:

- Social selling: online payment forms, digital payments, mobile payments/tap-to-phone.
- Online stores: shopping cart plugins, e-commerce partner integrations, Direct API, recurring payments, mass payments, IPP.
- Enterprise solutions: Payout API, tokenisation, split payments/settlements, lower transaction rates, custom API integrations.
- Plugins and platforms include WordPress/WooCommerce, Joomla/Hikashop, OpenCart, Magento, SHOPLINE, Shopify, EasyStore, SiteGiant, OnPay, PrestaShop, Ordersini, Orderla, and WhatsMenu.
- Sandbox is available at `sandbox.senangpay.my`; production hosted checkout uses `app.senangpay.my`.

## Integration Design

### Hosted / Manual Integration API

Main user flow:

1. Create a local pending payment before redirecting to senangPay.
2. Read Merchant ID and Secret Key from senangPay dashboard under `Settings > Profile > Shopping Cart Integration Link`.
3. Configure Return URL and Callback URL in the same dashboard section.
4. Generate request hash server-side.
5. Send buyer to senangPay hosted payment URL by GET or POST.
6. Treat Return URL as browser UX, not fulfilment authority.
7. Verify callback/return hash server-side.
8. Query order/transaction status when needed.
9. Settle idempotently only after local payment, order ID, amount, status, merchant, and provider transaction reference match.

Live hosted URL:

```text
https://app.senangpay.my/payment/{merchant_id}
```

Sandbox hosted URL:

```text
https://sandbox.senangpay.my/payment/{merchant_id}
```

Request parameters:

- `detail`: required, max 500 characters; only documented allowed characters; underscores become spaces.
- `amount`: required, 2 decimal places, for example `25.50`.
- `order_id`: required, max 100 characters; only letters, digits, and dash.
- `hash`: required.
- `name`, `email`, `phone`: documented required fields but not part of request hash; customer may overwrite on payment form.
- `timeout`: optional seconds; minimum 60; omitted means no timeout.

Request hash:

- Recommended: HMAC-SHA256.
- Legacy: MD5; not recommended.
- Source string sequence: `{secret_key}{detail}{amount}{order_id}`.
- HMAC key: Secret Key.

Return/callback parameters:

- `status_id`: `2` pending authorization, `1` successful, `0` failed.
- `order_id`: merchant order ID.
- `msg`: status message; may contain underscores.
- `transaction_id`: senangPay transaction ID.
- `hash`: integrity value.

Return/callback verification source string:

```text
{secret_key}{status_id}{order_id}{transaction_id}{msg}
```

Verify with SHA256 HMAC when configured, or MD5 only for legacy systems.

### Callback URL

Callback URL is the safer backend notification path for shopping cart/manual integrations.

Rules for agents:

- Use callback for payment settlement; do not fulfil from Return URL alone.
- senangPay sends callback data by HTTP POST.
- Callback URL must return exactly `OK` in plain HTML/text without tags for success.
- Avoid redirects on callback URL; HTTP 302 can break callback handling.
- Callback URL must not depend on sessions.
- Callback URL with non-secure HTTP ports may need whitelisting.
- Multiple callbacks can arrive for one transaction.
- Pending transactions can get callbacks starting 5 minutes after initiation.
- Final callback may arrive up to 1 hour after initiation.
- A failed status can be followed by successful status; latest callback is most up to date.

### Return URL Parameters

Return URL parameters are optional customization. Leave blank when the app can consume senangPay defaults. WooCommerce and OpenCart plugin users should leave it empty.

Supported placeholders include:

- `[NAME]`, `[EMAIL]`, `[PHONE]`, `[AMOUNT]`
- `[TXN_STATUS]`, `[ORDER_ID]`, `[TXN_REF]`, `[MSG]`, `[HASH]`, `[TXN_TYPE]`

The same customization also applies to callback URL. If custom parameters are used, verify the hash using the configured parameter string, URL-encoded values, and the merchant Secret Key. MD5 is deprecated; SHA256 is recommended.

### Query APIs

Order status query:

```text
GET https://app.senangpay.my/apiv1/query_order_status
```

Parameters:

- `merchant_id`
- `order_id`
- `hash`

Hash source sequence:

```text
{merchant_id}{secret_key}{order_id}
```

Transaction status query:

```text
GET https://app.senangpay.my/apiv1/query_transaction_status
```

Parameters:

- `merchant_id`
- `transaction_reference`
- `hash`

Hash source sequence:

```text
{merchant_id}{secret_key}{transaction_reference}
```

Use SHA256 HMAC with Secret Key if the account hash type is SHA256. MD5 appears in docs for legacy mode.

### Direct API

Direct API is not the default flow. It is for Enterprise package users and authorised merchants only. Server outgoing IP must be whitelisted.

Client Session endpoint:

```text
POST https://api.senangpay.my/payment/v3
POST https://api.sandbox.senangpay.my/payment/v3
```

Authentication:

- Basic Auth username: Merchant ID.
- Password: blank.

Supported payment methods include `fpx`, `boost`, `tng`, `grabpay`, `grabPl`, `shopeepay`, `shopback`, `cc`, and `jcb_direct`.

Direct API amount uses cents, unlike hosted/manual API which uses decimal string. Example: `200` means RM2.00.

Client Session hash source sequence:

```text
{secret_key}{customer_name}{customer_email}{customer_phone}{order_id}{amount}{detail}{payment_method}
```

JavaScript Web SDK uses returned `uuid`; UUID validity is 60 seconds. Do not expose Secret Key client-side.

SDK assets:

- Production JS: `https://senangpay.my/sdk/senangpay_v1.0.1.min.js`
- Sandbox JS: `https://senangpay.my/sdk/senangpay_v1.0.1_sandbox.min.js`
- CSS: `https://senangpay.my/sdk/senangpay_v1.0.1.min.css`

### FPX Bank List API

Direct API FPX integrations can query bank availability:

```text
GET https://api.senangpay.my/fpx_bank_list
GET https://api.sandbox.senangpay.my/fpx_bank_list
```

Authentication uses Basic Auth with Merchant ID as username and blank password. Response includes bank code, name, active/inactive status, type (`b2b` or `b2c`), and logo. Do not show inactive banks as selectable.

### Refund API

Refund API base URLs:

```text
https://api.senangpay.my
https://api.sandbox.senangpay.my
```

Get token:

```text
/merchant/get_token
```

The guide shows merchant ID in the `Authorization` header. Token is then passed as:

```text
Authorization: Bearer <token>
```

Create refund request:

```text
POST /merchant_service/create_refund_request
```

JSON body:

- `transaction_reference`
- `refund_amount`

Documented errors include duplicate refund request, transaction not found, partial refund not supported, amount exceeds transaction amount, refund window over 180 days, and authentication failed.

Agents should store refund attempts idempotently and never retry blindly after duplicate/refund-window errors.

## Account and Environment Design

Account setup reference should explain:

- Merchant registration starts at senangPay registration pages and dashboard.
- Sandbox account registration is separate and available from `http://sandbox.senangpay.my` / guide pages.
- Credentials live under `Settings > Profile > Shopping Cart Integration Link`.
- Required credentials: Merchant ID, Secret Key, Hash Type, Return URL, Callback URL.
- Secret Key can be regenerated; regenerating can break existing integrations if not redeployed.
- Hash Type should be SHA256 unless a legacy provider explicitly requires MD5.
- Direct API requires Enterprise package, authorisation, and IP whitelisting.
- Feature availability depends on package, add-ons, channel activation, merchant status, and dashboard settings.

## Security and Settlement Rules

Skill must enforce these rules:

- Keep Secret Key server-side only.
- Prefer SHA256/HMAC-SHA256; treat MD5 as legacy/deprecated.
- Never mark paid from browser Return URL alone.
- Use callback and/or query APIs for backend verification.
- Create local pending payment before sending user to senangPay.
- Verify hash, order ID, amount, status, provider transaction ID, and environment.
- Handle duplicate callbacks idempotently.
- Accept that latest callback can supersede earlier callback.
- Do not treat early failed callback as final if later success is documented as possible.
- Log callback events durably before returning `OK`.
- Return `OK` only after durable validation/recording path is complete.
- Separate sandbox and production credentials, merchant IDs, URLs, and webhook/callback config.
- For Direct API, do not expose session hash inputs or Secret Key in browser code.
- For refunds, prevent duplicate refund requests with local idempotency keys.

## File Design

### `SKILL.md`

Short provider-specific skill file with:

- Source references to read first.
- Core workflow for Manual API, Direct API, callbacks, queries, refunds.
- Core facts with URLs and hash formulas.
- Critical rules matching settlement posture.
- Loophole review loop for fake return, forged callback, duplicate callback, pending/final transition, wrong amount, wrong order, sandbox/live mix, Direct API IP whitelist, and refund duplicates.

### `references/senangpay-api.md`

Detailed API reference. Include endpoints, parameters, hash source strings, status meanings, callback behavior, Direct API notes, bank list, refund API, package caveats, and source URLs.

### `references/account-setup.md`

Account, dashboard, credentials, sandbox, package/feature, pricing caveat, migration, and support guidance. Avoid hardcoding fast-changing prices except as source-observed examples with a note to verify current pricing.

### `references/source-map.md`

Map use cases to source docs: hosted checkout, Direct API, callbacks, return URL, credentials, query status, refund API, sandbox, plugins, recurring, tokenisation, split settlement, payout, migration, pricing.

### `agents/openai.yaml`

Match existing package style:

```yaml
interface:
  display_name: "Setup SenangPay"
  short_description: "Set up senangPay payments"
  default_prompt: "Use $setup-senangpay to integrate senangPay payments."
```

### `README.md`

Add `setup-senangpay` to skill table and examples.

## Testing and Verification Plan

After implementation:

- Verify folder/file names match skill metadata.
- Check README contains `setup-senangpay` in table, prompt examples, and installation examples where relevant.
- Review formulas for exact sequence and clear SHA256-vs-MD5 language.
- Verify all source URLs are present in reference docs.
- Run `git diff --check`.
- Run any repository validation if available.

## Open Constraints

- Do not claim live current pricing as permanent; pricing and package capabilities can change.
- Do not claim account-specific payment method availability; verify dashboard/support.
- Do not claim Direct API access without Enterprise, authorisation, and IP whitelist.
- Do not claim DOKU migration behavior beyond source docs unless fetched and summarized.
