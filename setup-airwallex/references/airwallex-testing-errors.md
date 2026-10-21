# Airwallex Sandbox, Testing and Errors

Checked 2026-10-21.

## Source URLs

- Sandbox overview: `https://www.airwallex.com/docs/developer-tools/sandbox-environment`
- Sandbox settings: `https://www.airwallex.com/docs/developer-tools/sandbox-environment/sandbox-settings`
- Simulate shopper action: `https://www.airwallex.com/docs/developer-tools/sandbox-environment/payments/simulate-shopper-action`
- Simulate payment disputes: `https://www.airwallex.com/docs/developer-tools/sandbox-environment/payments/simulate-payment-disputes`
- Test your integration (test cards): `https://www.airwallex.com/docs/payments/test-and-go-live/test-card-numbers`
- Error response codes: `https://www.airwallex.com/docs/payments/troubleshooting/error-response-codes`
- Issuer response codes: `https://www.airwallex.com/docs/payments/troubleshooting/response-codes/issuer-response-codes`
- API errors: `https://www.airwallex.com/docs/api/errors`

## Sandbox

- Sign up at `https://www.sandbox.airwallex.com/global/signup`; ready instantly. Web app `https://www.sandbox.airwallex.com/app/`. API `https://api.sandbox.airwallex.com`.
- Auto-provisioned: onboarding approved, Payments onboarded, MCC `5499`, max transaction 10,000 (local currency), HPP checkout enabled. PCI DSS flag `false` (ask support to enable Native API card flows).
- Default payment methods: Visa, Mastercard, Amex, Airwallex Pay, Apple Pay, Google Pay, WeChat Pay, Alipay CN, Alipay HK, UnionPay. Request FPX, DuitNow, TNG and other local methods in the sandbox web app or via support.
- Extra sandbox accounts must use entity name "New Business Sandbox" or "Sandbox Business" or they stay "In review".
- Sandbox files are deleted after 90 days. Sandbox has lower rate limits; do not load test there.
- Simulation APIs work only in sandbox. Platforms can pass `x-on-behalf-of`.

## Simulating local methods (FPX, DuitNow, TNG, Boost, ...)

1. Create and confirm an intent in sandbox; take `next_action.url`.
2. `POST /api/v1/simulation/pa/shopper_actions/pay` with `{ "url": "<next_action.url>" }` for success, `/reject` for failure, `/verify` for a Payment Consent `next_action.url`.
3. Success returns HTTP 200 with empty body; your webhook then receives the outcome.
4. Errors: `validation_error` (bad action or URL), `operation_not_supported` (action not valid for that URL), `internal_error`.

## Test cards (sandbox only, published by Airwallex)

Any CVC and future expiry. Amounts of the form `8x.xx` trigger errors; avoid them when you expect success.

| Card | Brand | Result |
| --- | --- | --- |
| `4035501000000008` | Visa debit | Always succeeds |
| `2223000048410010` | Mastercard credit | Always succeeds |
| `5354563134257854` | Mastercard debit | Always succeeds |
| `370636803809394` | Amex | Always succeeds |
| `3569599999097585` | JCB | Always succeeds |
| `6250941006528599` | UnionPay debit | Always succeeds |
| `5307837360544518` or `2223000010181375` | Mastercard | `80.05` Do not honor (05), `80.14` invalid card (14), `80.51` insufficient funds (51), `issuer_declined` |
| `4012000300001003` | Visa | `80.59` suspected fraud (59), `80.07` pick up card (07) |
| `4646464646464644` | Visa | `risk_declined` |
| `1111111111111111` | Invalid | `validation_error` |

3DS: `4012000300000005` frictionless success; `4012000300000088` challenge success (OTP `1234`); `4012000300000062` challenge success without device data (OTP `1234`); `4012000300000021` frictionless with device data; `4012000300000013` and `4012000300000047` fail at confirm (not enrolled); `4012000300000070` fails after OTP at `confirm_continue`; `4012000300000039` with `88.88` authorization fails. MIT with 3DS (SCA): `5100013455636697`.

Required go-live tests (HPP/Drop-in/Elements): success with `4035501000000008`; 3DS failure with `4012000300000013`; insufficient funds with `5307837360544518` and `80.51`; refund success; refund over captured amount fails; refund that breaks refund conditions fails; risk data collection. Record `payment_intent_id`, `payment_attempt_id`, webhook delivery logs.

## Go live

Generate production keys in `https://www.airwallex.com/app/`; ask your Account Manager to copy sandbox settings; configure risk settings and production webhooks; switch base URL to `https://api.airwallex.com`; run real low-value payments; confirm webhooks return 200.

## Error codes (HTTP 400 unless noted)

| Code | Meaning |
| --- | --- |
| `3ds_not_supported` | 3DS required but card does not support it |
| `amount_above_limit` / `amount_below_limit` | Outside method or account limits |
| `authentication_declined` | Shopper failed authentication |
| `card_brand_not_supported` | Brand not supported |
| `configuration_error` | Request conflicts with merchant configuration |
| `currency_not_supported` | Currency not supported |
| `duplicate_request` | `request_id` used before |
| `frequency_above_limit` | Frequency limit reached |
| `invalid_status_for_operation` | Resource status does not allow the operation (for example refund on unpaid intent) |
| `issuer_declined` | Card issuer declined; see `provider_original_response_code` |
| `issuer_unavailable` / `provider_unavailable` | Try later |
| `no_3ds_liability_shift` | Declined, liability shift not possible |
| `operation_not_supported` | Operation not supported |
| `payment_method_not_allowed` | Method not activated |
| `provider_declined` | Method provider declined |
| `quote_expired` | FX quote expired |
| `rejected_by_routing_rules` | Merchant routing rules |
| `resource_already_exists` / `resource_not_found` | Id conflict or missing (404 too) |
| `risk_declined` | Blocked for risk |
| `suspended_from_online_payments` | Account suspended |
| `validation_error` | Field invalid; `source` names the field |
| `unauthorized` (401) | Access denied / token invalid |
| `not_found` (404) | Endpoint does not exist |
| `internal_error` (500) | Retry later |
| `too_many_requests` (429) | Rate limited |
| `forbidden` (403) | IP not whitelisted or scope missing |

Error body: `{ "code", "source", "message", "trace_id", "details" }`. Keep `trace_id` for support. Airwallex.js has its own error codes page (listed above in Source URLs of `airwallex-checkout.md`).

## Endpoint field tables (login and simulation)
### Obtain Access Token

`POST /api/v1/authentication/login`

Obtain an access token by specifying your `x-client-id` and `x-api-key` in the HTTP request header. Include the access token you receive in the `Authorization: Bearer [token]` header when calling other endpoints. The access token remains valid for 30 minutes and can be reused until it expires. Always refer to the `expires_at` field for the exact expiration time. If your scoped API key includes permissions for multiple accounts, include the `x-login-as` header to specify the target account that the returned access token is authorized for. If the scoped API key also includes organization-leve...

Response (201/200):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `expires_at` | string | no | Token expiration time in ISO8601 format. |
| `token` | string | no | Authentication token returned. |

Documented errors: `credentials_invalid`, `credentials_expired`

### Simulate a Shopper Action

`POST /api/v1/simulation/pa/shopper_actions/{action}`

You can use this endpoint to simulate the shopper actions after obtaining a redirect URL for a PaymentIntent or PaymentConsent. Upon successful simulation of a shopper action on the given URL, a response with HTTP status `200 OK` and empty body will be returned. This endpoint is only available in the sandbox environment.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `action` | string | yes | Action to simulate on a given redirect URL. Supported values are: PaymentIntent: `pay` and `reject`. PaymentConsent: `verify`. Values: `pay`, `reject` |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `url` | string | yes | The url with which shoppers actions will be simulated after redirection. Usually obtained from the response of Confirm a PaymentIntent and Verify a PaymentConsent. |

### Create a Payment Dispute

`POST /api/v1/simulation/pa/payment_disputes/create`

By creating a dispute, we raise a dispute for the merchant to respond.

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `due_at` | string | yes | The due date of dispute. |
| `payment_intent_id` | string | yes | The unique identifier of PaymentIntent to raise a dispute against. |
| `reason_code` | string | yes | The reason code for the dispute. Must be a valid reason code for the given processor. |
| `stage` | string | yes | The stage of dispute. |
| `amount` | number | no | The amount of dispute in transaction currency. The original payment amount is used by default. |
| `comment` | string | no | The text description of why the dispute is raised. |
| `documents` | array[string] | no | The documents explain why the dispute is raised. The list of File unique identifiers submitted via File Service. |

### Resolve a Payment Dispute

`POST /api/v1/simulation/pa/payment_disputes/{id}/resolve`

By resolving a dispute, the dispute transitions to the final status. - When `in_favor_of` = `MERCHANT`, the dispute becomes `WON` or `REVERSED` - When `in_favor_of` = `CUSTOMER`, the dispute becomes `LOST`.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | PaymentDispute unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `in_favor_of` | string | yes | The party who is in favor of when resolving the dispute. Possible values: - `CUSTOMER` - `MERCHANT` Values: `CUSTOMER`, `MERCHANT` |
| `amount` | number | no | The resolved amount. The full disputed amount is used by default. |
| `comment` | string | no | The text explains why the dispute is resolved. |

### Escalate a Payment Dispute

`POST /api/v1/simulation/pa/payment_disputes/{id}/escalate`

By escalating a dispute, the evidence provided by the merchant is declined, and the dispute is escalated to the next stage.

Path/query parameters:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | PaymentDispute unique identifier. |

Request body:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `due_at` | string | yes | The new due date of the dispute. |
| `amount` | number | no | The adjusted amount of dispute. |
| `comment` | string | no | The text description of why the dispute is escalated. |
| `documents` | array[string] | no | The documents explain why the dispute is escalated. The list of File unique identifiers submitted via File Service. |

