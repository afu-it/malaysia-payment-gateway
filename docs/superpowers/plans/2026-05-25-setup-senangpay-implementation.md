# Setup SenangPay Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a complete `setup-senangpay` provider skill package with source-backed API/account references and README discoverability.

**Architecture:** Follow the existing one-folder-per-provider skill structure used by `setup-fiuu`, `setup-billplz`, and `setup-toyyibpay`. Keep the main `SKILL.md` concise and action-oriented, while detailed factual material lives in `references/*.md` for progressive disclosure.

**Tech Stack:** Markdown skill files, YAML agent metadata, repository README documentation, git verification.

---

## File Structure

Create and modify these files:

- Create: `setup-senangpay/SKILL.md` — provider skill entrypoint with source rules, workflows, facts, critical rules, and loophole review loop.
- Create: `setup-senangpay/agents/openai.yaml` — Kilo/skill UI metadata matching existing provider packages.
- Create: `setup-senangpay/references/senangpay-api.md` — detailed senangPay API/reference facts from public docs.
- Create: `setup-senangpay/references/account-setup.md` — dashboard, credentials, sandbox, package, pricing caveats, and setup guidance.
- Create: `setup-senangpay/references/source-map.md` — use-case-to-source URL map for agents.
- Modify: `README.md` — add `setup-senangpay` to the skills table and usage examples.

Do not create application code. This repository is a skill package repository.

Implementation source of truth:

- Design doc: `docs/plans/2026-05-25-setup-senangpay-design.md`
- Existing patterns: `setup-fiuu/SKILL.md`, `setup-fiuu/agents/openai.yaml`, `setup-fiuu/references/fiuu-api.md`, `setup-billplz/SKILL.md`, `setup-toyyibpay/SKILL.md`

## Chunk 1: Create Skill Skeleton

### Task 1: Create `setup-senangpay` directories

**Files:**
- Create directory: `setup-senangpay/`
- Create directory: `setup-senangpay/references/`
- Create directory: `setup-senangpay/agents/`

- [ ] **Step 1: Verify parent directory**

Run: `ls`

Expected: Repository root lists existing provider folders like `setup-fiuu`, `setup-billplz`, and `README.md`.

- [ ] **Step 2: Create directories**

Run: `mkdir -p setup-senangpay/references setup-senangpay/agents`

Expected: No output.

- [ ] **Step 3: Verify directories exist**

Run: `ls setup-senangpay`

Expected: Output includes `agents` and `references`.

### Task 2: Create agent metadata

**Files:**
- Create: `setup-senangpay/agents/openai.yaml`

- [ ] **Step 1: Write metadata file**

Create `setup-senangpay/agents/openai.yaml` with exactly:

```yaml
interface:
  display_name: "Setup SenangPay"
  short_description: "Set up senangPay payments"
  default_prompt: "Use $setup-senangpay to integrate senangPay payments."
```

- [ ] **Step 2: Verify file contents**

Run: `git diff -- setup-senangpay/agents/openai.yaml`

Expected: Diff shows only the four YAML lines above.

## Chunk 2: Create Main Skill Entrypoint

### Task 3: Write `setup-senangpay/SKILL.md`

**Files:**
- Create: `setup-senangpay/SKILL.md`

- [ ] **Step 1: Write skill file**

Create `setup-senangpay/SKILL.md` with this content:

```markdown
---
name: setup-senangpay
description: Set up, build, debug, review, and explain senangPay payment integrations using official senangPay guide pages, Manual Integration API, Direct API, callbacks, return URLs, query status APIs, refund API, sandbox/live setup, Merchant ID, Secret Key, Hash Type, shopping cart plugins, e-commerce integrations, recurring payments, payout API, tokenisation, split settlements, and DOKU migration references. Use when working with senangPay hosted checkout, payment forms, callbacks, SHA256/MD5 hash verification, Direct API client sessions, JavaScript Web SDK, FPX bank lists, refunds, package capabilities, sandbox testing, or senangPay dashboard setup.
---

# SenangPay

Use this skill for senangPay payment gateway integration work.

## Source

Read `references/senangpay-api.md` before giving factual API guidance or writing integration code. It contains source URLs, integration surfaces, environment URLs, endpoint roles, parameter rules, hash formulas, callback behavior, query APIs, Direct API, FPX bank list, refund API, and settlement/security cautions.

Read `references/account-setup.md` when user asks how to register, which credentials are needed, where keys come from, sandbox setup, dashboard setup, package capabilities, pricing, support, or migration context.

Read `references/source-map.md` when choosing which senangPay guide page applies to hosted checkout, Direct API, plugins, callbacks, query APIs, refunds, recurring, tokenisation, split settlement, payout, sandbox, or migration.

## Core Workflow

1. Identify environment: sandbox or production.
2. Identify integration surface: hosted/manual payment form, shopping cart plugin, e-commerce platform integration, Direct API, recurring payment, refund API, payout API, tokenisation, or split settlement.
3. Confirm credentials and dashboard setup: Merchant ID, Secret Key, Hash Type, Return URL, and Callback URL.
4. Create a local pending payment before sending the customer to senangPay.
5. Generate request hash server-side using the correct formula and account Hash Type.
6. Redirect to hosted checkout or create a Direct API client session server-side.
7. Treat Return URL as browser UX only; do not fulfil from return alone.
8. Verify callback/return hash server-side, requery order or transaction status when needed, then settle only after amount, order ID, status, merchant, environment, and provider transaction reference match.
9. Make settlement idempotent because senangPay can send multiple callbacks and later callbacks can supersede earlier statuses.

## Core Facts

- Production hosted checkout URL: `https://app.senangpay.my/payment/{merchant_id}`.
- Sandbox hosted checkout URL: `https://sandbox.senangpay.my/payment/{merchant_id}`.
- Hosted/manual checkout accepts GET or POST parameters.
- Hosted/manual checkout amount uses two decimal places, for example `25.50`.
- Direct API amount uses cents, for example `200` means RM 2.00.
- Manual request hash source sequence: `{secret_key}{detail}{amount}{order_id}`.
- Return/callback status values: `1` successful, `0` failed, `2` pending authorization.
- Return/callback verification source sequence: `{secret_key}{status_id}{order_id}{transaction_id}{msg}`.
- Query order status endpoint: `GET https://app.senangpay.my/apiv1/query_order_status`.
- Query transaction status endpoint: `GET https://app.senangpay.my/apiv1/query_transaction_status`.
- Direct API endpoints: `POST https://api.senangpay.my/payment/v3` and `POST https://api.sandbox.senangpay.my/payment/v3`.
- Direct API requires Enterprise package, authorised merchant access, and outgoing server IP whitelisting.
- Merchant ID and Secret Key are in Dashboard > Settings > Profile > Shopping Cart Integration Link.
- SHA256 is recommended. MD5 is legacy/deprecated and should only be used for systems that explicitly require it.
- Callback URL uses HTTP POST and must output exactly `OK` without HTML tags for successful receipt.

## Critical Rules

- Keep Secret Key and private hash inputs server-side only.
- Do not mark paid from Return URL, frontend event, SDK response, or browser redirect alone.
- Prefer SHA256/HMAC-SHA256 and avoid MD5 unless supporting an explicit legacy integration.
- Always verify hash with the exact sequence documented for the active flow.
- Verify local order, expected amount, status, transaction ID/reference, merchant ID, and environment before marking paid.
- Log callback events durably and make processing idempotent before returning `OK`.
- Expect duplicate callbacks for the same transaction.
- Expect pending or failed callbacks before a later successful callback; latest callback can be the most up-to-date status.
- Avoid callback redirects and session-dependent callback handlers.
- Separate sandbox and production credentials, URLs, callback URLs, Return URLs, and dashboard settings.
- For Direct API, never expose Secret Key in browser JavaScript and remember UUID is valid for only 60 seconds.
- For refund API, store local refund attempts and do not retry blindly after duplicate, partial-refund, refund-window, or amount-exceeded errors.
- If package capability, payment method availability, settlement timing, current pricing, IP whitelisting, migration behavior, or dashboard state matters, verify live senangPay dashboard/docs/support before claiming certainty.

## Loophole Review Loop

When asked whether a senangPay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: browser Return URL, backend checkout creator, senangPay hosted checkout, Direct API session endpoint, JavaScript SDK, callback receiver, query APIs, database, fulfilment job, refund job, dashboard settings, and package/channel activation.
2. Check bypasses: fake success return, forged callback, wrong hash type, wrong hash sequence, wrong amount format, wrong order ID, wrong merchant ID, sandbox/live mix, duplicate callback, failed callback followed by success, pending marked paid, stale Direct API UUID, Direct API Secret Key leak, unwhitelisted Direct API server IP, inactive FPX bank, duplicate refund request, partial refund unsupported, and refund after 180-day window.
3. Add fixes: server-side local pending payment, server-side hash generation, callback hash verification, status requery, strict amount/status/order checks, idempotency keys, callback event log, exact `OK` response after durable recording, environment-separated config, bank status filtering, package capability checks, and refund idempotency.
4. Re-read `references/senangpay-api.md` sections relevant to any gap.
5. State remaining uncertainty if it depends on current package, channel enablement, pricing, dashboard configuration, DOKU migration state, or senangPay support approval.
```

- [ ] **Step 2: Verify metadata and references**

Run: `git diff -- setup-senangpay/SKILL.md`

Expected: Diff shows `name: setup-senangpay`, source references to all three reference files, and critical rules that prevent Return URL-only fulfilment.

## Chunk 3: Create API Reference

### Task 4: Write `references/senangpay-api.md`

**Files:**
- Create: `setup-senangpay/references/senangpay-api.md`

- [ ] **Step 1: Write API reference**

Create `setup-senangpay/references/senangpay-api.md` with this content:

```markdown
# senangPay API Reference

Use this reference for factual senangPay API guidance. Verify current dashboard settings, package capability, payment method availability, pricing, settlement timing, and support approval when those details matter.

## Source URLs

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
- `https://senangpay.com/social-selling/`
- `https://senangpay.com/online-store/`
- `https://senangpay.com/enterprise-solutions/`
- `https://senangpay.com/pricing/`

## Integration Surfaces

senangPay supports multiple payment surfaces:

- Hosted/manual payment form for shopping cart integrations.
- Payment forms for social selling: universal form, unique form, and product collection form.
- Shopping cart plugins and e-commerce platform integrations.
- Direct API with JavaScript Web SDK for Enterprise and authorised merchants.
- Recurring payments for instalments and subscriptions.
- Mass payments, Payout API, tokenisation, split settlement, IPP, and refund API.

Choose the simplest integration that satisfies the merchant requirement. Hosted/manual checkout is the normal custom integration path. Direct API is advanced and restricted.

## Hosted / Manual Integration API

Use this flow when a merchant has a custom shopping cart and wants to redirect customers to senangPay hosted checkout.

### Dashboard Setup

In the senangPay dashboard:

1. Go to `Settings > Profile`.
2. Find `Shopping Cart Integration Link`.
3. Copy `Merchant ID` and `Secret Key`.
4. Configure Return URL.
5. Configure Callback URL.
6. Select Hash Type. SHA256 is recommended; MD5 is legacy and not recommended.

### Payment URLs

Production:

```text
https://app.senangpay.my/payment/{merchant_id}
```

Sandbox:

```text
https://sandbox.senangpay.my/payment/{merchant_id}
```

Parameters can be sent by GET or POST.

### Request Parameters

| Parameter | Required | Notes |
| --- | --- | --- |
| `detail` | Yes | Payment description, max 500 characters. Underscores are converted to spaces. Allowed characters are letters, digits, dot, comma, dash, and underscore. |
| `amount` | Yes | Decimal amount with two decimal places, for example `25.50`. |
| `order_id` | Yes | Merchant order reference, max 100 characters. Only letters, digits, and dash. |
| `hash` | Yes | Integrity hash generated from Secret Key, detail, amount, and order ID. |
| `name` | Yes in docs | Prefills customer name. Not part of request hash. Customer may overwrite. |
| `email` | Yes in docs | Prefills customer email. Not part of request hash. Customer may overwrite. |
| `phone` | Yes in docs | Prefills customer phone. Not part of request hash. Customer may overwrite. |
| `timeout` | No | Payment timeout in seconds. Minimum is 60 seconds. Omit for no timeout. |

### Request Hash

Recommended SHA256 HMAC source sequence:

```text
{secret_key}{detail}{amount}{order_id}
```

HMAC key:

```text
{secret_key}
```

Legacy MD5 uses the same source string but should not be used unless a legacy system requires it.

### Return and Callback Parameters

senangPay sends these default fields back to merchant systems:

| Parameter | Meaning |
| --- | --- |
| `status_id` | `1` successful, `0` failed, `2` pending authorization. |
| `order_id` | Merchant order ID sent to senangPay. |
| `msg` | Status message, may contain underscores. |
| `transaction_id` | senangPay transaction ID. |
| `hash` | Integrity hash from senangPay. |

Verification hash source sequence:

```text
{secret_key}{status_id}{order_id}{transaction_id}{msg}
```

Use the account's configured Hash Type. Prefer SHA256 HMAC. Treat mismatched hashes as tampered data and do not settle.

## Return URL

Return URL is browser-facing. Use it to show the customer a result page, not to unlock paid access.

Return URL parameters can be customized only when the receiving system needs non-default field names. If the default parameters work, leave Return URL Parameters empty. senangPay says WooCommerce and OpenCart plugin users should leave the field empty.

Supported placeholders include:

- `[NAME]`
- `[EMAIL]`
- `[PHONE]`
- `[AMOUNT]`
- `[TXN_STATUS]`
- `[ORDER_ID]`
- `[TXN_REF]`
- `[MSG]`
- `[HASH]`
- `[TXN_TYPE]`

Return URL parameter customization also applies to Callback URL. For custom parameter hashes, URL encode values and calculate the hash over the configured query string appended to the merchant Secret Key. MD5 is deprecated; SHA256 is recommended.

## Callback URL

Callback URL is an alternative/backend notification for shopping cart integrations and is recommended for data integrity.

Key behavior:

- senangPay sends callback data by HTTP POST.
- Callback sends the same parameters as Return URL unless custom Return URL Parameters are configured.
- Callback URL must output exactly `OK` without HTML tags when successful.
- Avoid HTTP redirects; 302 can cause callback failure.
- Callback URL must not depend on sessions.
- Non-secure HTTP callback ports may need whitelisting by senangPay support.
- Multiple callbacks can arrive for the same transaction.
- If a transaction is pending, callbacks can start 5 minutes after transaction start.
- The last callback is 1 hour after transaction start.
- A failed status can be followed by a successful status because earlier bank queries may not be final.
- The latest callback data is considered the most up-to-date transaction data.

Implementation rule: record callback events durably, verify hash, process idempotently, then return `OK` only after the durable path succeeds.

## Query APIs

Use query APIs to re-check status from the backend before settlement or when callback/return ordering is unclear.

### Query Order Status

Endpoint:

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

For SHA256, use HMAC-SHA256 with Secret Key as the HMAC key. Docs also show MD5 for legacy hash type.

### Query Transaction Status

Endpoint:

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

For SHA256, use HMAC-SHA256 with Secret Key as the HMAC key. Docs also show MD5 for legacy hash type.

## Direct API

Direct API is an advanced flow for Enterprise package users and authorised merchants only. Merchants must contact senangPay and whitelist the server outgoing IP address.

Direct API has two parts:

1. Server creates a Client Session.
2. Browser uses JavaScript Web SDK with the returned UUID.

### Client Session Endpoint

Sandbox:

```text
POST https://api.sandbox.senangpay.my/payment/v3
```

Production:

```text
POST https://api.senangpay.my/payment/v3
```

Authentication:

- Basic Auth username: Merchant ID.
- Basic Auth password: blank.

Parameters include:

| Parameter | Required | Notes |
| --- | --- | --- |
| `payment_method` | Yes | Values include `fpx`, `boost`, `tng`, `grabpay`, `grabPl`, `shopeepay`, `shopback`, `cc`, `jcb_direct`. |
| `fpx_bank_code` | Conditional | Required when `payment_method` is `fpx`. |
| `shopeepay_platform` | Conditional | Required when `payment_method` is `shopeepay`; `desktop` or `mobile`. |
| `filter_card` | No | Optional array for card filters. |
| `customer_name` | Yes | Customer name. |
| `customer_email` | Yes | Customer email. |
| `customer_phone` | Yes | Customer phone. |
| `order_id` | Yes | Unique merchant transaction reference. |
| `amount` | Yes | Amount multiplied by 100. Example `200` means RM 2.00. |
| `detail` | Yes | Payment description. |
| `hash` | Yes | SHA256 hash as documented. |

Hash source sequence:

```text
{secret_key}{customer_name}{customer_email}{customer_phone}{order_id}{amount}{detail}{payment_method}
```

Response includes:

- `result`: `1` successful, `0` failed.
- `msg`: description.
- `uuid`: client token for JavaScript Web SDK.

The UUID is valid for only 60 seconds.

### JavaScript Web SDK

Include jQuery and senangPay SDK assets.

Production JS:

```text
https://senangpay.my/sdk/senangpay_v1.0.1.min.js
```

Sandbox JS:

```text
https://senangpay.my/sdk/senangpay_v1.0.1_sandbox.min.js
```

CSS:

```text
https://senangpay.my/sdk/senangpay_v1.0.1.min.css
```

HTML target:

```html
<div id="senangpay_payment"></div>
```

JavaScript:

```js
const senangpay = new Senangpay(merchantid);
senangpay.pay(uuid);
```

Do not generate hashes or expose Secret Key in browser code.

### FPX Bank List API

Production:

```text
GET https://api.senangpay.my/fpx_bank_list
```

Sandbox:

```text
GET https://api.sandbox.senangpay.my/fpx_bank_list
```

Authentication uses Basic Auth with Merchant ID as username and blank password.

Response contains `fpx_bank_list` entries with:

- `fpx_bank_code`
- `name`
- `status`: active or inactive
- `type`: b2b or b2c
- `logo`

Do not show inactive banks as selectable.

## Refund API

Base URLs:

```text
https://api.senangpay.my
https://api.sandbox.senangpay.my
```

Get token endpoint:

```text
/merchant/get_token
```

The guide shows merchant ID in the `Authorization` header for token generation.

Create refund endpoint:

```text
POST /merchant_service/create_refund_request
```

Use token header:

```text
Authorization: Bearer <token>
```

JSON body:

```json
{
  "transaction_reference": "1729057564864250",
  "refund_amount": "2"
}
```

Successful response creates a refund with fields such as `refund_reference_no`, `amount`, `type`, `status`, `message`, and `approval_code`.

Documented invalid responses include:

- Transaction does not exist for merchant.
- Refund request for transaction reference already exists.
- Partial refund is not supported.
- Requested refund amount exceeds transaction amount.
- Transaction date exceeds allowed refund window: 180 days.
- Authentication failed.

Use local refund idempotency. Do not retry blindly when the provider reports duplicate or unsupported refund state.

## Product and Package Caveats

Public pages describe these products:

- Social selling: online payment form, digital payments, mobile/tap-to-phone payments.
- Online store: shopping cart plugins, e-commerce integrations, Direct API, recurring payments, mass payments, IPP.
- Enterprise: Payout API, tokenisation, split payments, custom rates, advanced technical functions.

Feature availability can depend on package, add-ons, channel activation, dashboard state, and senangPay approval. Direct API is Enterprise and authorised only. Always verify current package and dashboard before promising availability.

## Settlement Checklist

Before marking a local payment as paid:

1. Local payment exists and is pending or in a transition-safe state.
2. Hash verifies with the configured Hash Type and exact source sequence.
3. `order_id` maps to exactly one local payment.
4. Amount matches expected local amount with the right unit for the active flow.
5. Status is successful and final enough for the business rule.
6. Provider transaction ID/reference is stored.
7. Environment matches local config: sandbox vs production.
8. Callback/query event is recorded durably.
9. Idempotency guard prevents duplicate fulfilment.
10. Return URL was not used as the sole authority.
```

- [ ] **Step 2: Verify key formulas are present**

Run: `grep -n "{secret_key}{detail}{amount}{order_id}\|{secret_key}{status_id}{order_id}{transaction_id}{msg}\|payment/v3" setup-senangpay/references/senangpay-api.md`

Expected: Output includes the manual request hash, return/callback hash, and Direct API endpoint sections.

## Chunk 4: Create Account Setup Reference

### Task 5: Write `references/account-setup.md`

**Files:**
- Create: `setup-senangpay/references/account-setup.md`

- [ ] **Step 1: Write account setup reference**

Create `setup-senangpay/references/account-setup.md` with this content:

```markdown
# senangPay Account Setup

Use this reference when explaining senangPay registration, dashboard setup, sandbox setup, credentials, package capabilities, pricing caveats, or support requirements.

## Source URLs

- `https://senangpay.com/pricing/`
- `https://senangpay.com/social-selling/`
- `https://senangpay.com/online-store/`
- `https://senangpay.com/enterprise-solutions/`
- `https://guide.senangpay.com/getting-started`
- `https://guide.senangpay.com/account-and-subscriptions`
- `https://guide.senangpay.com/testing-sandbox`
- `https://guide.senangpay.com/merchant-id-and-secret-key`
- `https://guide.senangpay.com/manual-integration-api`
- `https://guide.senangpay.com/direct-api`
- `https://guide.senangpay.com/payments-settlements-and-charges`
- `https://guide.senangpay.com/frauds-and-disputes`
- `https://guide.senangpay.com/rules-and-regulations`
- `https://guide.senangpay.com/senangpay-to-doku-platform-migration`

## Registration and Dashboard

senangPay merchant registration starts from the senangPay website and dashboard. The public guide groups onboarding under Getting Started:

- Merchant account registration.
- Requirements or restrictions.
- Approval timing.
- Required document submission.
- Dashboard login.
- Website policy templates.

Do not guess merchant eligibility, approval time, or required documents for a specific business. Use the current guide/dashboard/support when those details matter.

## Sandbox

senangPay provides sandbox testing for webstore or app development.

Useful URLs:

- `https://guide.senangpay.com/testing-sandbox`
- `http://sandbox.senangpay.my`
- Hosted sandbox checkout: `https://sandbox.senangpay.my/payment/{merchant_id}`
- Direct API sandbox: `https://api.sandbox.senangpay.my/payment/v3`
- FPX bank list sandbox: `https://api.sandbox.senangpay.my/fpx_bank_list`
- Refund API sandbox base: `https://api.sandbox.senangpay.my`

Sandbox and production credentials must be separated. Do not reuse production Merchant ID, Secret Key, callback URLs, or Return URLs in sandbox unless the dashboard explicitly says to.

## Credentials

For manual/hosted and plugin integrations:

1. Log in to senangPay dashboard.
2. Go to `Settings > Profile`.
3. Find `Shopping Cart Integration Link`.
4. Copy `Merchant ID`.
5. Copy `Secret Key`.
6. Select Hash Type.
7. Configure Return URL.
8. Configure Callback URL.
9. Save changes.

Secret Key can be regenerated. Regenerating it can break existing deployed integrations until environment variables and webhook verification code are updated.

## Hash Type

senangPay dashboard supports Hash Type selection.

- SHA256 is recommended.
- MD5 is not recommended, is older, and is described as vulnerable/deprecated.
- Always choose SHA256 unless a system provider explicitly confirms otherwise.

Agents should state which Hash Type their code expects and document it in environment setup.

## Required Environment Variables

Recommended app environment variables:

```text
SENANGPAY_ENV=sandbox
SENANGPAY_MERCHANT_ID=...
SENANGPAY_SECRET_KEY=...
SENANGPAY_HASH_TYPE=sha256
SENANGPAY_RETURN_URL=https://example.com/payments/senangpay/return
SENANGPAY_CALLBACK_URL=https://example.com/api/payments/senangpay/callback
```

For Direct API:

```text
SENANGPAY_DIRECT_API_ENABLED=false
SENANGPAY_API_BASE_URL=https://api.sandbox.senangpay.my
```

Do not expose `SENANGPAY_SECRET_KEY` to browser or mobile clients.

## Return URL and Callback URL Setup

Return URL is where senangPay redirects the buyer after payment processing. Use it for customer experience.

Callback URL is a backend notification URL for shopping cart integrations. It is recommended for data integrity and should be used for settlement.

Callback requirements:

- Must handle HTTP POST.
- Must not require a user session.
- Must avoid redirects.
- Must return exactly `OK` without HTML tags after successful durable processing.
- Non-secure HTTP ports may require whitelisting from senangPay support.

## Packages and Capabilities

Public pages describe package tiers and feature groups such as Starter, Advance, and Enterprise. Pricing, fees, and package names can change, especially during promotions.

Source-observed examples from public pricing pages include:

- Starter and Advance annual packages for small businesses.
- Enterprise custom package with custom rates and advanced features.
- FPX, cards, e-wallets, BNPL, payment forms, dashboard, plugins, e-invoice/quotation, API integration, recurring, payout, tokenisation, split settlement, faster settlement, and foreign cards.

Do not hardcode current pricing in app docs unless the user explicitly asks and you verify the live pricing page. Prefer wording such as "verify current pricing and package capabilities in the senangPay pricing page/dashboard."

## Direct API Access

Direct API is available only for Enterprise package users and restricted to authorised merchants. Merchants must contact senangPay and whitelist their server outgoing IP address.

Do not build Direct API as the default integration unless the user confirms:

- Enterprise package or approval.
- Direct API authorisation.
- Server outgoing IP whitelisting.
- Required payment methods.

## Product Selection Guidance

Choose by merchant need:

- Social seller without website: senangPay payment forms.
- Standard custom web app: hosted/manual payment checkout with callback and query verification.
- WooCommerce/OpenCart/Magento/etc.: use official plugin/platform guide first.
- Seamless/advanced checkout: Direct API only if Enterprise and authorised.
- Subscription/installment: recurring payment references and package capability checks.
- Marketplace or multi-party settlement: split settlement and Enterprise support checks.
- Refund automation: Refund API with local idempotency and approval workflow awareness.
- Vendor/customer payouts: Payout API and Enterprise support checks.

## Support and Operational Caveats

Use senangPay support or dashboard for:

- Package upgrades/downgrades.
- Payment method/channel enablement.
- Direct API approval and IP whitelisting.
- Non-secure callback port whitelisting.
- Current pricing and transaction rates.
- Settlement timing and payout schedules.
- DOKU platform migration behavior.
- Fraud/dispute handling and prohibited items.
- Required documents or business restrictions.

## Production Readiness Checklist

- Merchant ID and Secret Key configured server-side.
- Hash Type is SHA256 unless legacy integration requires MD5.
- Return URL and Callback URL set in dashboard.
- Public HTTPS callback URL deployed.
- Callback handler returns exact `OK` only after durable event recording.
- Sandbox and production settings separated.
- Local pending payment created before checkout redirect/session.
- Duplicate callbacks are harmless.
- Failed-then-success callback transitions are handled.
- Query status path exists for reconciliation.
- Secret Key regeneration process is documented.
- Package-specific features are verified before launch.
```

- [ ] **Step 2: Verify setup caveats are present**

Run: `grep -n "Direct API\|SHA256\|Callback URL\|SENANGPAY_SECRET_KEY" setup-senangpay/references/account-setup.md`

Expected: Output includes Direct API access constraints, SHA256 recommendation, callback requirements, and secret env var.

## Chunk 5: Create Source Map Reference

### Task 6: Write `references/source-map.md`

**Files:**
- Create: `setup-senangpay/references/source-map.md`

- [ ] **Step 1: Write source map**

Create `setup-senangpay/references/source-map.md` with this content:

```markdown
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
```

- [ ] **Step 2: Verify source map coverage**

Run: `grep -n "manual-integration-api\|direct-api\|refund-api\|pricing\|senangpay-to-doku" setup-senangpay/references/source-map.md`

Expected: Output includes all core source links.

## Chunk 6: Update README

### Task 7: Add README entries

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add skill table row**

In `README.md`, add this row after `setup-fiuu` or in the provider list near the other setup skills:

```markdown
| `setup-senangpay` | senangPay hosted/manual checkout, Direct API, callbacks, query status, refunds, plugins, recurring, payout, tokenisation, split settlement |
```

- [ ] **Step 2: Add usage example**

In the "How To Ask Your Agent" section, add this example after the Fiuu example:

```markdown
```text
Use $setup-senangpay to integrate senangPay hosted checkout, Callback URL verification, query status reconciliation, and idempotent settlement.
```
```

- [ ] **Step 3: Verify README entries**

Run: `grep -n "setup-senangpay\|senangPay hosted" README.md`

Expected: Output includes the table row and prompt example.

## Chunk 7: Verify Package

### Task 8: Validate content and formatting

**Files:**
- Verify: `README.md`
- Verify: `setup-senangpay/SKILL.md`
- Verify: `setup-senangpay/agents/openai.yaml`
- Verify: `setup-senangpay/references/senangpay-api.md`
- Verify: `setup-senangpay/references/account-setup.md`
- Verify: `setup-senangpay/references/source-map.md`

- [ ] **Step 1: Check created files**

Run: `ls setup-senangpay && ls setup-senangpay/references && ls setup-senangpay/agents`

Expected: Output includes `SKILL.md`, `references`, `agents`, `senangpay-api.md`, `account-setup.md`, `source-map.md`, and `openai.yaml`.

- [ ] **Step 2: Check skill metadata**

Run: `grep -n "^name: setup-senangpay\|^description:" setup-senangpay/SKILL.md`

Expected: Output includes `name: setup-senangpay` and a description mentioning senangPay integrations.

- [ ] **Step 3: Check required security language**

Run: `grep -n "Return URL\|Secret Key\|idempotent\|SHA256\|OK" setup-senangpay/SKILL.md setup-senangpay/references/*.md`

Expected: Output includes rules about Return URL not being fulfilment authority, Secret Key server-side, idempotent processing, SHA256 preference, and exact `OK` callback response.

- [ ] **Step 4: Check diff whitespace**

Run: `git diff --check`

Expected: No output.

- [ ] **Step 5: Review final diff**

Run: `git diff -- README.md setup-senangpay docs/plans/2026-05-25-setup-senangpay-design.md docs/superpowers/plans/2026-05-25-setup-senangpay-implementation.md`

Expected: Diff only contains the SenangPay design doc, implementation plan, new skill package, and README updates.

### Task 9: Commit only if explicitly requested

**Files:**
- Stage only if user explicitly asks for commit.

- [ ] **Step 1: Check git status**

Run: `git status --short`

Expected: Shows new docs and `setup-senangpay` plus README after implementation.

- [ ] **Step 2: Do not commit automatically**

Because this environment forbids committing unless explicitly requested by the user, stop before commit unless the user has explicitly asked for a commit.

- [ ] **Step 3: If user explicitly asks for commit, commit relevant files**

Run only after explicit commit request:

```bash
git add README.md docs/plans/2026-05-25-setup-senangpay-design.md docs/superpowers/plans/2026-05-25-setup-senangpay-implementation.md setup-senangpay
git commit -m "feat(senangpay): add setup skill"
```

Expected: Commit succeeds without force, amend, or hook skipping.
```
