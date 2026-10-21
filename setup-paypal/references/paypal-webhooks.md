# PayPal Webhooks

Sources: `/api/rest/webhooks`, `/api/rest/webhooks/rest`, `/api/rest/webhooks/simulator`, `/api/rest/webhooks/events-dashboard`, `/api/rest/webhooks/event-names`, `/payment-methods/webhooks`, `/subscriptions/webhooks`, Webhooks Management v1 schema (`https://developer.paypal.com/api/webhooks/v1/schema.json`, spec 1.11). Checked 2026-10-21.

## Setup

- Subscribe a listener URL per REST app, in Developer Dashboard > Apps & Credentials > app > Webhooks, or with `POST /v1/notifications/webhooks`. Up to 10 webhook URLs per app. Subscribe specific event types or `*` for all.
- Each subscribed URL gets a Webhook ID. Store it per environment: it is part of the signed message.
- Only events from that app are delivered. Payments made with another REST app on the same account do not reach this listener.
- Listener must be HTTPS on port 443 and return 2xx. Non-2xx, no response, or connection failure: PayPal retries up to 25 times over 3 days, then marks Failed (resend manually from the Webhooks Events dashboard, or `POST /v1/notifications/webhooks-events/{event_id}/resend`).
- Delivery failures with no status code are commonly a firewall blocking 443, or the domain being flagged by URL filtering services.
- Dashboard event viewer: Live and Sandbox Webhooks Events, filter last 30 days, resend.

## Signature Headers

| Header | Meaning |
| --- | --- |
| `paypal-transmission-id` | Unique transmission ID |
| `paypal-transmission-time` | Transmission timestamp |
| `paypal-transmission-sig` | Base64 asymmetric signature |
| `paypal-cert-url` | URL of the X.509 certificate holding the public key |
| `paypal-auth-algo` | Algorithm, for example `SHA256withRSA` |

## Method 1: Self Verification (PayPal's preferred method)

Signed message, exactly:

```text
<paypal-transmission-id>|<paypal-transmission-time>|<webhook_id>|<crc32>
```

- `webhook_id`: your stored Webhook ID (not in the request).
- `crc32`: CRC32 checksum of the raw HTTP body bytes, as a decimal number. Compute on the original raw body; do not parse and re-serialize. PayPal's sample computes the hex CRC32 and parses it as an unsigned integer: `parseInt("0x" + crc32(rawBody).toString("hex"))`.
- Verify `base64decode(paypal-transmission-sig)` against the message with SHA256 RSA using the public key in the PEM certificate downloaded from `paypal-cert-url`. Download once and cache.

Pseudo-code (mirrors PayPal's Node sample):

```text
raw      = request body bytes (no JSON parsing yet)
crc      = unsigned_decimal(crc32(raw))
message  = headers["paypal-transmission-id"] + "|" +
           headers["paypal-transmission-time"] + "|" +
           WEBHOOK_ID + "|" + crc
pem      = cached_download(headers["paypal-cert-url"])
ok       = rsa_sha256_verify(public_key(pem), message, base64decode(headers["paypal-transmission-sig"]))
if not ok: log and do not process
event    = json_parse(raw)
```

Node core of PayPal's sample:

```javascript
const crc = parseInt("0x" + crc32(rawBody).toString("hex"));
const message = `${transmissionId}|${timeStamp}|${WEBHOOK_ID}|${crc}`;
const verifier = crypto.createVerify("SHA256");
verifier.update(message);
const ok = verifier.verify(certPem, Buffer.from(headers["paypal-transmission-sig"], "base64"));
```

Hardening that PayPal does not document but you should add (our recommendation): only fetch `paypal-cert-url` over HTTPS from a PayPal host (`api.paypal.com`, `api-m.paypal.com`, `api.sandbox.paypal.com`, `api-m.sandbox.paypal.com`), and cache by URL.

Mock events from the Webhooks simulator use the Webhook ID string `WEBHOOK_ID` for self verification.

## Method 2: Postback Verification

`POST /v1/notifications/verify-webhook-signature` with a Bearer token from the same environment. Send `webhook_event` exactly as received; parsing and re-serializing "can fail verification". Not usable for simulator mock events. Accept only `verification_status == "SUCCESS"`.

```bash
curl -X POST https://api-m.sandbox.paypal.com/v1/notifications/verify-webhook-signature \
  -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -d '{"transmission_id":"<hdr>","transmission_time":"<hdr>","cert_url":"<hdr>",
       "auth_algo":"<hdr>","transmission_sig":"<hdr>","webhook_id":"<WEBHOOK_ID>",
       "webhook_event": <RAW_BODY_JSON_UNCHANGED>}'
```

Practical note: building the JSON request by string concatenation with the raw body keeps the event byte-identical.

## After Verification

1. Deduplicate on event `id` (store processed IDs). Retries resend the same event.
2. Use `event_type` and `resource` (`resource_type`, `resource_version`) to route. For `PAYMENT.CAPTURE.*`, `resource` is the capture; the order ID is in `resource.supplementary_data.related_ids.order_id`, and links include `up` to the order. GrabPay docs: "Upon receiving each webhook, fetch the latest order details using Show order details".
3. Re-fetch current state (`GET /v2/checkout/orders/{id}` or `GET /v2/payments/captures/{id}`) and settle idempotently.
4. Return 2xx quickly; do heavy work async.

## Events To Subscribe (Orders v2 checkout)

| Event | Meaning and action |
| --- | --- |
| `CHECKOUT.ORDER.APPROVED` | Buyer approved. Capture (or authorize) if your server has not already. |
| `CHECKOUT.PAYMENT-APPROVAL.REVERSED` | Problem after approval, before capture; PayPal cancels the order and refunds the buyer. Notify buyer. |
| `PAYMENT.CAPTURE.PENDING` | Initiated but not complete. Do not fulfil. |
| `PAYMENT.CAPTURE.COMPLETED` | Fulfil after checks. |
| `PAYMENT.CAPTURE.DENIED` | Cancel fulfilment. (Payments v2 table also lists `PAYMENT.CAPTURE.DECLINED`; subscribe to both.) |
| `PAYMENT.CAPTURE.REFUNDED` | Merchant refunded. |
| `PAYMENT.CAPTURE.REVERSED` | PayPal reversed the capture (for example chargeback). |
| `PAYMENT.REFUND.PENDING` / `PAYMENT.REFUND.FAILED` | Refund status changes. |
| `PAYMENT.AUTHORIZATION.CREATED` / `PAYMENT.AUTHORIZATION.VOIDED` | Authorize flow. Voided also fires when the 30-day validity ends. |
| `CUSTOMER.DISPUTE.CREATED` / `UPDATED` / `RESOLVED` | Disputes. |

Subscriptions: `BILLING.SUBSCRIPTION.CREATED`, `.ACTIVATED`, `.UPDATED`, `.EXPIRED`, `.CANCELLED`, `.SUSPENDED`, `.PAYMENT.FAILED`; `PAYMENT.SALE.COMPLETED` (each subscription payment), `PAYMENT.SALE.REFUNDED`, `PAYMENT.SALE.REVERSED`; `BILLING.PLAN.CREATED`, `.UPDATED`, `.ACTIVATED`, `.PRICING-CHANGE.ACTIVATED`, `.DEACTIVATED`; `CATALOG.PRODUCT.CREATED`, `.UPDATED`.

Vault: `VAULT.PAYMENT-TOKEN.CREATED`, `VAULT.PAYMENT-TOKEN.DELETED`, `VAULT.PAYMENT-TOKEN.DELETION-INITIATED`.

Payouts: `PAYMENT.PAYOUTSBATCH.DENIED|PROCESSING|SUCCESS`; `PAYMENT.PAYOUTS-ITEM.BLOCKED|CANCELED|FAILED|HELD|REFUNDED|RETURNED|SUCCEEDED|UNCLAIMED`.

Invoicing: `INVOICING.INVOICE.CANCELLED|CREATED|PAID|REFUNDED|SCHEDULED|UPDATED`.

Other names on the event page (platform onboarding, managed accounts, tokenization, legacy Payments v1 sales and orders, `CHECKOUT.CHECKOUT.BUYER-APPROVED` for Express Checkout, `PAYMENTS.PAYMENT.CREATED`, `IDENTITY.AUTHORIZATION-CONSENT.REVOKED`) are listed at `/api/rest/webhooks/event-names` and are not needed for a standard Malaysian merchant integration.

## Webhooks Management API

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/v1/notifications/webhooks` | Create webhook (`url`, `event_types[].name`) |
| `GET` | `/v1/notifications/webhooks` | List webhooks |
| `GET` / `PATCH` / `DELETE` | `/v1/notifications/webhooks/{webhook_id}` | Show, update, delete |
| `GET` | `/v1/notifications/webhooks/{webhook_id}/event-types` | List subscriptions |
| `POST` / `GET` | `/v1/notifications/webhooks-lookup` | Create / list lookups |
| `GET` / `DELETE` | `/v1/notifications/webhooks-lookup/{webhook_lookup_id}` | Show / delete lookup |
| `POST` | `/v1/notifications/verify-webhook-signature` | Verify signature |
| `GET` | `/v1/notifications/webhooks-event-types` | List available events |
| `GET` | `/v1/notifications/webhooks-events` | List event notifications (`page_size`, `start_time`, `end_time`, `transaction_id`, `event_type`) |
| `GET` | `/v1/notifications/webhooks-events/{event_id}` | Show event |
| `POST` | `/v1/notifications/webhooks-events/{event_id}/resend` | Resend event |
| `POST` | `/v1/notifications/simulate-event` | Simulate event (`webhook_id` or `url`, `event_type`, `resource_version`) |

## Field Tables

#### verify_webhook_signature (POST /v1/notifications/verify-webhook-signature)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `auth_algo` | string | yes | The algorithm that PayPal uses to generate the signature and that you can use to verify the signature. Extract this value from the `PAYPAL-AUTH-ALGO` response header, which is received with the webhook notification. [maxLength=100; pattern=^[a-zA-Z0-9]+$] |
| `cert_url` | string | yes | The X.509 public key certificate. Download the certificate from this URL and use it to verify the signature. Extract this value from the `PAYPAL-CERT-URL` response header, which is received with the webhook notification. [maxLength=500] |
| `transmission_id` | string | yes | The ID of the HTTP transmission. Contained in the `PAYPAL-TRANSMISSION-ID` header of the notification message. [maxLength=50; pattern=^(?!\d+$)\w+\S+] |
| `transmission_sig` | string | yes | The PayPal-generated asymmetric signature. Appears in the `PAYPAL-TRANSMISSION-SIG` header of the notification message. [maxLength=500; pattern=^(?!\d+$)\w+\S+] |
| `transmission_time` | string | yes | The date and time of the HTTP transmission, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). Appears in the `PAYPAL-TRANSMISSION-TIME` header of the notification message. [maxLength=100] |
| `webhook_id` | string | yes | The ID of the webhook as configured in your Developer Portal account. [maxLength=50; pattern=^[a-zA-Z0-9]+$] |
| `webhook_event` | object (event) | yes | A webhook event notification. |

#### verify_webhook_signature_response

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `verification_status` | string | yes | The status of the signature verification. [enum: SUCCESS, FAILURE] |

#### webhook (POST /v1/notifications/webhooks)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `id` | string | no | The ID of the webhook. [readOnly] |
| `url` | string | yes | The URL that is configured to listen on `localhost` for incoming `POST` notification messages that contain event information. [maxLength=2048] |
| `event_types` | array<event_type> (definitions-event_type_list) | yes | An array of events to which to subscribe your webhook. To subscribe to all events, including events as they are added, specify the asterisk wild card. To replace the `event_types` array, specify the ... [maxItems=500] |
| `event_types.name` | string | yes | The unique event name.Note: To subscribe to all events, including events as they are added, specify an `*` as the value to represent a wildcard. |
| `event_types.description` | string | no | A human-readable description of the event. [readOnly] |
| `event_types.status` | string | no | The status of a webhook event. [readOnly] |
| `event_types.resource_versions` | array<resource_versions> (resource_versions_list) | no | Identifier for the event type example: 1.0/2.0 etc. [readOnly] |
| `links` | array<link_description> (definitions-link_description_list) | no | An array of request-related [HATEOAS links](/docs/api/reference/api-responses/#hateoas-links/). [readOnly] |
| `links.href` | string | yes | The complete target URL. To make the related call, combine the method with this [URI Template-formatted](https://tools.ietf.org/html/rfc6570) link. For pre-processing, include the `$`, `(`, and `)` ... |
| `links.rel` | string | yes | The [link relation type](https://tools.ietf.org/html/rfc5988#section-4), which serves as an ID for a link that unambiguously describes the semantics of the link. See [Link ... |
| `links.method` | string | no | The HTTP method required to make the related call. [enum: GET, POST, PUT, DELETE, HEAD, CONNECT, OPTIONS, PATCH] |

#### event (webhook body)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `id` | string | no | The ID of the webhook event notification. [readOnly] |
| `create_time` | string | no | The date and time when the webhook event notification was created, in [Internet date and time format](https://tools.ietf.org/html/rfc3339#section-5.6). [readOnly] |
| `resource_type` | string | no | The name of the resource related to the webhook notification event. [readOnly] |
| `event_version` | string (event_version) | no | The event version in the webhook notification. [pattern=^([0-9]+.[0-9]+)$] |
| `event_type` | string | no | The event that triggered the webhook event notification. [readOnly] |
| `summary` | string | no | A summary description for the event notification. [readOnly] |
| `resource_version` | string (resource_version) | no | The resource version in the webhook notification. [pattern=^([0-9]+.[0-9]+)$] |
| `resource` | object (resource) | no | The resource that triggered the webhook event notification. [readOnly] |
| `links` | array<link_description> (link_description_list) | no | An array of request-related [HATEOAS links](/docs/api/reference/api-responses/#hateoas-links). [readOnly] |

#### simulate_event (POST /v1/notifications/simulate-event)

| Field | Type | Req | Notes (from schema) |
| --- | --- | --- | --- |
| `webhook_id` | string | no | The ID of the webhook. If omitted, the URL is required. [maxLength=50; pattern=^[a-zA-Z0-9]+$] |
| `url` | string | no | The URL for the webhook endpoint. If omitted, the webhook ID is required. [maxLength=2048] |
| `event_type` | string | yes | The event name. Specify one of the subscribed events. For each request, provide only one event. [maxLength=50; pattern=^[a-zA-Z0-9.]+$] |
| `resource_version` | string | no | The identifier for event type ex: 1.0/2.0 etc. |
