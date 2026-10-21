# Antom Source Map

Use this map to pick the right official page before answering or coding. All URLs are on `https://docs.antom.com` unless stated. Checked 2026-10-21.

## Indexes

- `https://docs.antom.com/llms.txt`
- `https://docs.antom.com/llms-auto.txt`

Use for: finding pages. The Markdown version of each page is the page URL with `.md`.

## API Basics And Signature

- `https://docs.antom.com/ac/ams/api.md` (Antom APIs: domains, sandbox and production paths)
- `https://docs.antom.com/ac/ams/api_fund.md` (Overview: message structure, headers, result object)
- `https://docs.antom.com/ac/ams/digital_signature.md` (Sign a request and verify the signature)
- `https://docs.antom.com/ac/ams/idempotency.md`
- `https://docs.antom.com/ac/ams/me.md` (Message encoding)
- `https://docs.antom.com/ac/ref/cc.md` (Amount object rules, MYR decimals)
- `https://docs.antom.com/ac/questions/general_question.md` (sandbox to live path change)

Use for: domains, sandbox path, headers, RSA256 signing, response and notification verification, idempotency keys, amount format.

## Server And Client SDKs

- `https://docs.antom.com/ac/sdks/server_sdks.md`
- `https://docs.antom.com/ac/sdks/web.md`, `https://docs.antom.com/ac/sdks/web_loadAntom.md`, `https://docs.antom.com/ac/sdks/web_antom-createElement.md`
- `https://github.com/alipay/global-open-sdk-java` (`EndPointConstants.java`, `WebhookTool.java`)
- `https://github.com/alipay/global-open-sdk-python` (`default_alipay_client.py`, `tools/signature_tool.py`, `tools/webhook_tool.py`, `model/antom_path_constants.py`)

Use for: SDK packages, sandbox path rewrite, signing implementation details, endpoint path constants, web SDK package `@alipay/ams-checkout`, locales.

## One-time Payments API Reference

- `https://docs.antom.com/ac/ams/payment_cashier.md` (pay)
- `https://docs.antom.com/ac/ams/session_cashier.md` (createPaymentSession)
- `https://docs.antom.com/ac/ams/consult.md` (consult)
- `https://docs.antom.com/ac/ams/paymentri_online.md` (inquiryPayment)
- `https://docs.antom.com/ac/ams/capture.md` (capture)
- `https://docs.antom.com/ac/ams/paymentc_online.md` (cancel)
- `https://docs.antom.com/ac/ams/refund_online.md` (refund)
- `https://docs.antom.com/ac/ams/ir_online.md` (inquiryRefund)

Use for: field tables, result handling, error codes in `antom-checkout.md`, `antom-payment-ops.md`, `antom-errors.md`.

## Notifications

- `https://docs.antom.com/ac/ams/paymentrn_online.md` (notifyPayment)
- `https://docs.antom.com/ac/ams/notify_capture.md` (notifyCapture)
- `https://docs.antom.com/ac/ams/notify_refund.md` (notifyRefund)
- `https://docs.antom.com/ac/cashierpay/notifications.md`
- `https://docs.antom.com/ac/merchant_service/notification.md`

Use for: notification fields, URL precedence, acknowledgement body, retry schedule, Dashboard notification testing.

## Integration Guides

- `https://docs.antom.com/ac/cashierpay/HOSTEDCKP.md` (Hosted Checkout Page)
- `https://docs.antom.com/ac/cashierpay/element.md` (Payment Element)
- `https://docs.antom.com/ac/cashierpay/embeddedckp.md` (Embedded Checkout Page)
- `https://docs.antom.com/ac/cashierpay/apm_api.md` (APM API-only)
- `https://docs.antom.com/ac/cashierpay/urls.md` (URLs returned per method and terminal)
- `https://docs.antom.com/ac/cashierpay/capture.md`
- `https://docs.antom.com/ac/cashierpay/cancel.md`
- `https://docs.antom.com/ac/cashierpay/refund.md`
- `https://docs.antom.com/ac/cashierpay/payment_status_desc.md`
- `https://docs.antom.com/integration_guide_en.md`

Use for: integration modes, `productScene`, redirect handling, capture timing, cancel windows, refund rules, status handling, go-live checklist.

## Payment Methods (Malaysia)

- `https://docs.antom.com/ac/pm/enumeration_values.md` (enums, card brands, FPX bank list)
- `https://docs.antom.com/ac/antomop/fpx.md`
- `https://docs.antom.com/ac/antomop/touchngo.md`
- `https://docs.antom.com/ac/antomop/boost.md`
- `https://docs.antom.com/ac/antomop/grabpay_.md`
- `https://docs.antom.com/ac/antomop/shopeepay.md`
- `https://docs.antom.com/ac/merchant_service/enable_payment.md` (Dashboard-enabled method list with acquirers)

Use for: `paymentMethodType` values, limits, refund periods, acquirer entities, supported integration modes, FPX bank codes.

## Dashboard, Keys, Onboarding, Settlement

- `https://docs.antom.com/ac/merchant_service/antom_dashboard.md`
- `https://docs.antom.com/ac/merchant_service/merchant_onboard.md`
- `https://docs.antom.com/ac/merchant_service/key_config.md`
- `https://docs.antom.com/ac/merchant_service/settlements.md`
- `https://docs.antom.com/ac/merchant_service/settlement_faq.md`
- `https://www.antom.com/contact-us/`

Use for: account registration, key generation and replacement, integration resources, go-live steps, settlement statuses, where fees are shown.

## Sandbox And Testing

- `https://docs.antom.com/ac/ref/sandbox.md`
- `https://docs.antom.com/ac/ref/testwallet.md`
- `https://docs.antom.com/ac/ref/test_resource.md`
- `https://docs.antom.com/ac/ref/card.md`
- `https://docs.antom.com/ac/ref/wallet.md`
- `https://docs.antom.com/ac/ref/other.md`
- `https://docs.antom.com/ac/questions/sandbox_question.md`

Use for: sandbox credentials, test wallet install, test cards, simulating failures and 3DS, testing online banking.

## Antom AI Tooling

- `https://docs.antom.com/ac/ref/skill.md` (Antom Integration Skill, `https://github.com/ant-intl/antom-ai-tools`)
- `https://docs.antom.com/ac/ref/mcp.md` (MCP server)

Use for: pointing users to Antom's own agent tooling.

## Other API Pages Scanned For The Endpoint Catalogue

All 54 pages under `https://docs.antom.com/ac/ams/` were scanned for title, endpoint, and purpose, including Tokenized Payment (`authconsult`, `accesstokenapp`, `authrevocation`, `payment_agreement`, `session_autodebit`), vaulting (`vaulting_session`, `vault_method`, `inquire_vaulting`, `fetchnouce`), subscriptions (`create_sub`, `change_sub`, `update`, `cancel_sub`, `inquire_sub`, `inquire_sublist`, `inquire_subpayment`), disputes (`accept`, `supply_evidence`, `download`, `notify_dispute`), `updateAmount`, `exchange_rate`, `init_auth`, `uploadInvoiceShippingFile`, `declare`, `inquirydeclare`, and the notify pages.

Use for: the endpoint catalogue in `antom-api.md`. Read the page itself before building against any of these.

## Coverage

- Pages enumerated: 851 unique `.md` URLs (433 from `llms.txt`, 842 from `llms-auto.txt`, merged). All 851 were downloaded.
- Pages read in full or in part: 55 docs pages (every docs URL listed in the sections above, excluding the scan-only section). Embedded Checkout Page, web SDK, and general FAQ pages were read in part.
- Pages scanned for title, endpoint, and purpose only: the other `ac/ams` API pages (54 in that folder in total).
- Source code read from official SDK repositories: 7 files (Java and Python).

Left out, and why:

- Billing (`ac/billing`, 93 pages), metering, and tax APIs: separate subscription billing product, not part of a basic Malaysian checkout.
- ISV and platform (`ac/isv`, 88 pages), marketplace payouts, transfers, and reserve: for platforms onboarding sub-merchants.
- APO (`ac/apo`, 73 pages): separate acquiring product with its own API family.
- Antom Business Account (`ac/aba`, 68 pages): business account and FX services, not checkout.
- Antom Shield (`ac/antomshield`) risk APIs, omnichannel and in-store (`ac/omnichannel`, `ac/scantopay`): in-store or risk products.
- Mobile SDK method references (`ac/sdks`, 72 pages): only the server SDK and web SDK entry pages were read; read the iOS and Android pages when building a native app.
- Payment method pages for other countries (most of `ac/antomop`, 111 pages): not used by a Malaysian merchant, except cross-border wallets noted in `antom-codes.md`.
- Chinese duplicates (`*_zh-cn`): same content as English pages.
- Disputes, reconciliation reports, EasySafePay, and Tokenized Payment guides: catalogued but not detailed. Read the pages when the merchant uses them.
