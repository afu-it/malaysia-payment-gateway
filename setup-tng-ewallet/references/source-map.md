# Touch 'n Go eWallet Source Map

Use this map to pick the right source before answering or coding. Mini program doc base: `https://miniprogram.tngdigital.com.my/docs/miniprogram_tngd/mpdev/`.

## Coverage

- Index used: the docs site's embedded table of contents (hydrate data served with `?ssrScript=1` on any doc page). No `llms.txt`, `sitemap.xml`, OpenAPI file, or public SDK repo exists (all 404 or absent, checked 2026-10-21).
- Pages enumerated in the TOC: 281. Pages fetched and read: 281, plus 7 slugs linked from pages but missing from the TOC (2 had content: `xww2sz`, `api_device_permission_showauthguide`; 5 were empty: `api_idempotency`, `capabilities`, `api_openapi_signcontract`, `api_openapi_tngdsigncontract`, `api_openapi_getsiteinfo`). 3 TOC pages were empty (`api_ui_feedback_hidetoast`, `api_ui_keyboard_hidekeyboard`, `api_ui_selector-query_query_selectorquery-selectviewport`).
- About 237 TOC pages are UI framework, components, canvas, storage, map, and device JSAPIs. They were scanned for payment, signature, notify, refund, and settlement terms; only `component_open_web-view` (use `my.tradePay`, not H5, for payment) and `mini-program-studio_api-implementation-difference` (IDE simulates `my.tradePay`) were relevant.
- Also read: TNGD WooCommerce plugin v1.0.1 full source, TNGD merchant page, 3 merchant help centre articles.

## Partner-Only (not public)

- Open API host names (sandbox and production), Postman collection, test users and wallets.
- Notification request headers and the exact verify string for `notifyPayment`.
- `Encrypt` (RSA_AES) procedure, rate limit numbers, refund window length.
- JSAPIs `my.signContract`, `my.tngdSignContract`, `my.getSiteInfo` (named in docs, pages empty).
- A formal web acquiring (`alipayplus.acquiring.*`) spec. Only the plugin source is public.
- Production web acquiring base URL and the notification URL registration method.

Merchants get these from TNGD during onboarding (see `account-setup.md`).

## Open API Fundamentals And Security

- API fundamentals: `rekqh-qm`
- Message transmission security: `msg_transmission_security`
- Message encoding: `encode`
- Call an API via adding a signature: `call_api`
- Data dictionary: `vs3pkf`
- Error codes: `error_codes`

Use for: URL pattern, headers, RSA256 string to sign, response verification, Base64 and URL encoding, Amount and other models, enums, common error codes.

## Payments

- Pay: `v1_pay` (older version at `xww2sz`)
- Inquiry payment: `dl7y4p`
- Notify payment: `edgctc`
- Refund: `v1_refund`
- Inquiry refund: `tp3ntm`
- `my.tradePay`: `api_openapi_tradepay`
- web-view payment rule: `component_open_web-view`

Use for: cashier and agreement payments, product codes, payment factors, result logic, notification ack rules, refunds, polling interval, tradePay codes.

## Authorization And User Data

- Prepare (auto debit binding): `v1_auth_prepare`
- Apply token: `qfd7r1`
- Cancel token: `uh7agv`
- User info by access token: `lab2r2`
- `my.getAuthCode`: `api_openapi_getauthcode`
- `my.getOpenUserInfo`: `api_openapi_getopenuserinfo`
- Send message: `v2_miniprogram_send`

Use for: auto debit, OAuth scopes, token refresh, user profile and eKYC data, templated messages.

## Settlement, Onboarding, And Platform

- Partner settlement process: `partner_settlement_process`
- Mini program project planning: `project_planning`
- Developer guide: `developer-guide`
- Quick start overview: `quick-start_overview`
- Apply for an account: `quick-start_create-account`
- Release mini program: `quick-start_release-mini-program`
- App Container: `app-container`
- Deeplinks: `deeplink`
- `my.request` (domain whitelist): `api_network_request`

Use for: T+1 SFTP CSV, onboarding milestones, credential handover, mini program release, deeplinks, outbound request whitelist.

## Web Acquiring (Plugin Source)

- Plugin page: `https://wordpress.org/plugins/touch-n-go-ewallet/`
- Plugin info API: `https://api.wordpress.org/plugins/info/1.2/?action=plugin_information&request[slug]=touch-n-go-ewallet`
- Source zip: `https://downloads.wordpress.org/plugin/touch-n-go-ewallet.1.0.1.zip` (`includes/class-wc-gateway-tngd.php`, `touch-n-go-ewallet.php`)

Use for: `alipayplus.acquiring.order.create/query/refund` and `refund.query` paths, envelope, signature over `request`, `acquirementStatus` values, notification fields, return and cron fallback.

## Merchant Product And Pricing

- Merchant page: `https://www.tngdigital.com.my/business/merchant`
- MDR: `https://merchantsupport.tngdigital.com.my/hc/en-us/articles/15599821031321-How-much-is-the-MDR`
- Payment Link: `https://merchantsupport.tngdigital.com.my/hc/en-us/articles/360061912153-Payment-Link`
- Daily settlement report: `https://merchantsupport.tngdigital.com.my/hc/en-us/articles/360057944133-How-do-I-view-daily-settlement-report-in-Merchant-Dashboard`
- Onboarding request form: `https://tngdigitalbizhelp.zendesk.com/hc/en-my/requests/new?ticket_form_id=59240529886361`

Use for: self-serve merchant registration, no-code options, pricing links, dashboard settlement reports.

## Left Out

- Mini program UI, framework, component, and device JSAPI pages: not related to payment acceptance.
- FPX, cards, DuitNow QR acquiring: TNGD documents no API for these. TNG eWallet is also offered by third-party gateways, which are outside this skill.
