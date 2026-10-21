# Adyen Webhooks Reference

Use this file when building or debugging the webhook receiver. Webhooks are the source of truth for payment state on Adyen. Checked 2026-10-21.

## Source URLs

- `https://docs.adyen.com/development-resources/webhooks`
- `https://docs.adyen.com/development-resources/webhooks/configure-and-manage`
- `https://docs.adyen.com/development-resources/webhooks/handle-webhook-events`
- `https://docs.adyen.com/development-resources/webhooks/secure-webhooks`
- `https://docs.adyen.com/development-resources/webhooks/secure-webhooks/verify-hmac-signatures`
- `https://docs.adyen.com/development-resources/webhooks/webhook-types`
- `https://docs.adyen.com/development-resources/webhooks/webhook-types/additional-settings`
- `https://docs.adyen.com/development-resources/webhooks/troubleshoot`
- `https://docs.adyen.com/development-resources/logs-resources/webhook-event-logs`
- Official HMAC validator source: `https://github.com/Adyen/adyen-node-api-library/blob/main/src/utils/hmacValidator.ts`
- Webhook OpenAPI: `https://github.com/Adyen/adyen-openapi/blob/main/json/Webhooks-v1.json`

## Set Up (Customer Area)

Source: `https://docs.adyen.com/development-resources/webhooks/configure-and-manage`

1. Expose a public HTTPS endpoint, no redirects, TLSv1.2 or TLSv1.3. Live ports: 443, 8443 or 8843. Test also allows HTTP on 80, 8080, 8888.
2. Customer Area > **Developers** > **Webhooks** > **Create new webhook** > add **Standard webhook**. Needed roles: **Merchant admin** or **Merchant technical integrator**.
3. General: Enabled, Version, Description, Merchant accounts (all, include only, exclude). Adyen recommends configuring on the company account and filtering merchant accounts.
4. Server configuration: URL, Method (**JSON**, HTTP POST, or SOAP; use JSON), Encryption protocol.
5. Security: OAuth 2.0 (Standard webhook only; token TTL at least 3599 seconds) or Basic authentication (username and password), and **HMAC key** (generate, copy, store).
6. Events: pick event types. Default event types cannot be disabled (see below).
7. Optional **Additional settings** (extra `additionalData` fields).
8. Save configuration. Use **Test configuration** to send a test event.

Live: configure again in `https://ca-live.adyen.com/` (no copy function from test), and use the live HMAC key, which is different from the test key.

Each HMAC key is linked to one webhook endpoint. Generate a key per endpoint, and a new key when you go live. After rotating, keep accepting the previous key for some time because propagation takes time.

## Payload Structure (Standard webhook, JSON)

Source: `https://docs.adyen.com/development-resources/webhooks/webhook-types`

```json
{
  "live": "false",
  "notificationItems": [
    {
      "NotificationRequestItem": {
        "additionalData": { "hmacSignature": "<BASE64_SIGNATURE>" },
        "amount": { "value": 1130, "currency": "MYR" },
        "eventCode": "AUTHORISATION",
        "eventDate": "2026-10-21T10:00:00+08:00",
        "merchantAccountCode": "<MERCHANT_ACCOUNT>",
        "merchantReference": "<YOUR_ORDER_REF>",
        "originalReference": "<ONLY_FOR_MODIFICATIONS>",
        "paymentMethod": "visa",
        "pspReference": "<16_CHAR_PSP_REFERENCE>",
        "reason": "",
        "success": "true"
      }
    }
  ]
}
```

- `live` is a string, `"true"` or `"false"`. Reject events whose `live` does not match the environment of the endpoint.
- JSON and HTTP POST webhooks contain one `NotificationRequestItem` per request. SOAP can carry up to six. Still loop over `notificationItems`.
- `success` is a string `"true"` / `"false"`. For some codes (for example `REPORT_AVAILABLE`, `EXPIRE`) it is always `"true"`.
- `originalReference` = PSP reference of the original payment, for modification events (CAPTURE, REFUND, CANCELLATION, CANCEL_OR_REFUND and others).
- `reason`: failure text when `success` is false. On card AUTHORISATION success it can hold `authCode:last4:expiry` (example `033899:1111:03/2030`). On `REPORT_AVAILABLE` it is the download URL.
- `eventDate` is ISO 8601. Use it, not arrival order, to order events.
- Full field table: `adyen-fields.md` (`NotificationRequestItem`).

## HMAC Signature Verification (Standard webhook)

Source: `https://docs.adyen.com/development-resources/webhooks/secure-webhooks/verify-hmac-signatures`, confirmed against the official Node library `hmacValidator.ts`.

Where: `NotificationRequestItem.additionalData.hmacSignature` (one signature per item).

String to sign: these 8 values, in this exact order, joined with `:` (colon). Use an empty string for an empty or missing value.

1. `pspReference`
2. `originalReference`
3. `merchantAccountCode`
4. `merchantReference`
5. `amount.value` (integer as decimal text, for example `1130`)
6. `amount.currency`
7. `eventCode`
8. `success` (`true` or `false`, as received)

Algorithm:

1. Payload bytes = UTF-8 of the joined string.
2. Key bytes = the HMAC key from Customer Area **decoded from hexadecimal** (it is a hex string, not raw text).
3. HMAC-SHA256(key bytes, payload bytes).
4. Base64-encode the digest.
5. Compare with `hmacSignature` in constant time.

Official test vector (verified 2026-10-21):

```text
HMAC key (hex): 44782DEF547AAA06C910C43932B1EB0C71FC68D9D0C057550C48EC2ACF6BA056
signed string : 7914073381342284::TestMerchant:TestPayment-1407325143704:1130:EUR:AUTHORISATION:true
expected      : coqCmt/IZ4E3CzPvMY8zTjQVL5hYJUiBRg8UU+iCWo0=
```

Pseudo-code:

```text
function verifyItem(item, hexKey):
    sig = item.additionalData.hmacSignature        # missing -> reject
    parts = [item.pspReference, item.originalReference or "",
             item.merchantAccountCode, item.merchantReference,
             str(item.amount.value), item.amount.currency,
             item.eventCode, item.success]
    data = join(parts, ":")
    mac = base64(HMAC_SHA256(hexDecode(hexKey), utf8(data)))
    return constantTimeEqual(base64Decode(mac), base64Decode(sig))
```

Python:

```python
import base64, hashlib, hmac

def adyen_item_is_valid(item: dict, hex_key: str) -> bool:
    sig = (item.get("additionalData") or {}).get("hmacSignature")
    if not sig:
        return False
    amount = item.get("amount") or {}
    parts = [
        item.get("pspReference", ""), item.get("originalReference", "") or "",
        item.get("merchantAccountCode", ""), item.get("merchantReference", ""),
        str(amount.get("value", "")), amount.get("currency", ""),
        item.get("eventCode", ""), item.get("success", ""),
    ]
    data = ":".join(parts).encode("utf-8")
    mac = hmac.new(bytes.fromhex(hex_key), data, hashlib.sha256).digest()
    return hmac.compare_digest(mac, base64.b64decode(sig))
```

Node (official library):

```js
const { hmacValidator } = require('@adyen/api-library');
const validator = new hmacValidator();
for (const wrapper of body.notificationItems) {
  const item = wrapper.NotificationRequestItem;
  if (!validator.validateHMAC(item, process.env.ADYEN_HMAC_KEY)) { /* reject item */ }
}
```

Notes:

- The signature covers only those 8 fields. `additionalData`, `paymentMethod`, `eventDate` and `reason` are not signed. Do not trust unsigned fields for money decisions without cross-checking.
- Unlike some gateways, the standard-webhook signature is not over the raw body, so parsing JSON first is fine. Keep `amount.value` as an integer and `success` as the received string.
- Official libraries: Java `HMACValidator`, PHP `HmacSignature`, .NET `WebhooksHandler`, Node `hmacValidator`, Ruby `hmac_validator.rb`, Python `Adyen/util.py` (links on the source page).

### HMAC in the header (other webhook types)

Some non-standard webhooks (for example **Recurring tokens life cycle events**, Management and Balance Platform webhooks) send the signature in HTTP headers instead:

- `hmacsignature`: Base64 signature.
- `protocol`: `HmacSHA256`.

Compute HMAC-SHA256 with the hex-decoded key over the **raw request body exactly as received** (do not deserialize first), Base64-encode, compare in constant time. Official Node: `validateHMACSignature(hexKey, signatureHeader, rawBody)`.

## Accept, Store, Then Process

Source: `https://docs.adyen.com/development-resources/webhooks/handle-webhook-events`

1. Verify (HMAC, plus Basic auth or OAuth if configured). Do not accept insecure messages.
2. Store the message (database or queue).
3. Respond with a 2xx status (for example `200` or `202`) within **10 seconds**. Do not run business logic before responding.
4. Process asynchronously.

If no 2xx arrives within 10 seconds, the webhook is marked failing and all events to that endpoint go to a retry queue.

Duplicates: you can get the same event twice. Duplicates share `eventCode` and `pspReference`; `eventDate` and other fields can differ. Use the latest one. Idempotency key suggestion: `(eventCode, pspReference, success)` plus `originalReference` for modifications.

Be ready for new, unknown event codes: accept them with 2xx and ignore.

## Retry Queue

Source: `https://docs.adyen.com/development-resources/webhooks/troubleshoot`

- After initial failed attempts, the endpoint becomes **Failing** and events wait in a retry queue for up to **30 days**.
- Retry intervals: 30 s, 30 s, 1 min, 2 min, 5 min, 10 min, 15 min, 30 min, 1 h, 2 h, 4 h, 8 h.
- Customer Area > Developers > Webhooks > failing endpoint > **Troubleshoot**: per-event error, **Retry**, or **Ignore** (permanent).
- Common failures: 401 (auth or HMAC set up wrong), 404 (URL unreachable), 422 (your business logic rejected it; acknowledge first), 429 (you rate-limited Adyen after recovery), 500.
- Alerts: Notification center > Settings > Developers and Integrations > Webhooks. Initial failure alert after 5 retries; ongoing failure after 7 days; recovery alert.
- A disabled webhook queues events and delivers them when re-enabled. Webhook configurations inactive for more than six months are deleted.

## Network Allowlisting

Source: `https://docs.adyen.com/development-resources/webhooks/secure-webhooks`

Adyen does not publish a fixed list of webhook source IPs. Resolve `out.adyen.com` by DNS (`dig +short out.adyen.com`) and refresh regularly (hourly suggested), or allowlist by domain. Allowlisting is hardening only; keep HMAC verification. Source also: `https://docs.adyen.com/development-resources/security/integration-security/allowlisting`

## Standard Webhook Event Codes

Source: `https://docs.adyen.com/development-resources/webhooks/webhook-types`

Default (always on):

| `eventCode` | Meaning | Reference fields |
| --- | --- | --- |
| `AUTHORISATION` | Outcome of a payment. `success` true = authorised. Rely on this for order state, for sync and async methods. | `pspReference` = payment |
| `AUTHORISATION_ADJUSTMENT` | Outcome of an authorised amount adjustment. | `originalReference` = payment |
| `CANCELLATION` | Outcome of a cancel made with the payment PSP reference. | `originalReference` = payment |
| `CANCEL_OR_REFUND` | Outcome of a reversal. | `originalReference` = payment |
| `CAPTURE` | Outcome of a manual capture request. Not sent for automatic captures. For delayed automatic capture, enable it (see below). | `originalReference` = payment |
| `CAPTURE_FAILED` | Scheme rejected a capture after a successful CAPTURE. Technical failures are re-captured by Adyen within 10 business days. | `originalReference` = payment |
| `EXPIRE` | The authorisation expired. `success` always true. | |
| `HANDLED_EXTERNALLY` | Payment handled outside Adyen. | |
| `ORDER_OPENED` / `ORDER_CLOSED` | Partial payments order created / finished (`success` false = not fully paid, earlier partial payments cancelled or refunded). | |
| `REFUND` | Refund validations outcome. `success` true = sent to scheme (can still fail later). | `originalReference` = payment |
| `REFUND_FAILED` | Scheme rejected a refund after REFUND success. | `originalReference` = payment |
| `REFUNDED_REVERSED` | Refunded funds came back to your account. | `originalReference` = payment |
| `REFUND_WITH_DATA` | Outcome of an unreferenced refund. | |
| `REPORT_AVAILABLE` | New report. `pspReference` = file name, `reason` = download URL. | |
| `VOID_PENDING_REFUND` | Outcome of cancelling an unreferenced POS refund. | |
| `AUTORESCUE` / `CANCEL_AUTORESCUE` | Auto Rescue ended / cancelled. | |
| `TECHNICAL_CANCEL` | Outcome of a cancel made with your payment reference (`POST /cancels`). | `originalReference` = payment |
| `ACH_NOTIFICATION_OF_CHANGE`, `DIRECT_DEBIT_NOTICE_OF_CHANGE_NOTIFICATION` | US/CA direct debit only. | |
| Dispute events | `CHARGEBACK`, `CHARGEBACK_REVERSED`, `NOTIFICATION_OF_CHARGEBACK`, `INFORMATION_SUPPLIED`, `NOTIFICATION_OF_FRAUD`, `PREARBITRATION_LOST`, `PREARBITRATION_WON`, `PREARBITRATION_OPEN`, `PREARBITRATION_ACCEPTED`, `PREARBITRATION_DECLINED`, `PREARBITRATION_ISSUER_WITHDRAWN`, `SCHEME_ARBITRATION`, `REQUEST_FOR_INFORMATION`, `SECOND_CHARGEBACK`, `DISPUTE_DEFENSE_PERIOD_ENDED`, `ISSUER_RESPONSE_TIMEFRAME_EXPIRED`, `ISSUER_COMMENTS` | |
| Payout events | `PAYOUT_EXPIRE`, `PAYOUT_DECLINE`, `PAYOUT_THIRDPARTY`, `PAIDOUT_REVERSED` | |

Adyen does **not** send a webhook when a payment settles. Use the Settlement details report for settlement.

Non-default (enable on the Standard webhook page, Events row):

- `OFFER_CLOSED`: offer expired, for example shopper abandoned. Cards: offers expire after 12 hours by default. Touch 'n Go: 30 minutes.
- `REFUND_NOT_CLEARED`, `SETTLED_REVERSED`: Sales day payout adjustments.
- `RECURRING_CONTRACT`: token created (legacy; Adyen recommends the Recurring tokens life cycle webhook). Must also be enabled on the Webhooks settings page.

Webhooks settings page (Developers > Webhooks > Settings): `RECURRING_CONTRACT`, `POSTPONED_REFUND`, `AUTHENTICATION` (standalone 3DS), `CAPTURE` for **delayed automatic capture** (also enable on the Standard webhook page).

Risk settings page: `MANUAL_REVIEW_ACCEPT`, `MANUAL_REVIEW_REJECT`.

The Webhooks-v1 OpenAPI lists 43 `eventCode` values; the full enum is in `json/Webhooks-v1.json`.

## Other Webhook Types (separate endpoint each)

Source: `https://docs.adyen.com/development-resources/webhooks/webhook-types`

- **Generic pending webhook**: `eventCode` `PENDING` for pending redirect payment methods (Basic auth only).
- **Recurring tokens life cycle events**: `recurring.token.created`, `recurring.token.updated`, `recurring.token.disabled`, `recurring.token.alreadyExisting`. Body: `type`, `environment` (`test`/`live`), `createdAt`, `eventId`, `data.merchantAccount`, `data.shopperReference`, `data.storedPaymentMethodId`, `data.type`, `data.operation`. HMAC in header. See `adyen-tokenization-links.md`.
- **Account settings details**, **Account data**, **Payment method webhook** (`paymentMethod.created` and others), Adyen Giving `DONATION`, terminal webhooks.

## Additional Settings (extra `additionalData`)

Source: `https://docs.adyen.com/development-resources/webhooks/webhook-types/additional-settings`

Enable per webhook under **Additional Settings**. Examples: Add Acquirer Result (`authCode`, `cvcResult`, `avsResult`, `referred`), Add Raw Acquirer Result (`refusalReasonRaw`, `acquirerCode`), Include ARN (`arn`, CAPTURE and REFUND only), Include Acquirer Reference (`acquirerReference`), Include Alias Info, Include Authorised Amount (zero auth). Configure the same settings in live as in test.

## Settlement Rules For The Receiver

1. Verify HMAC for each item; reject if missing.
2. Check `live` and `merchantAccountCode` match this environment and account.
3. Store raw event with a unique key (`eventCode`, `pspReference`, `originalReference`, `success`).
4. Return 2xx fast.
5. Worker: for `AUTHORISATION` with `success` `"true"`, find the local order by `merchantReference`, check `amount.value` and `amount.currency` equal what you asked for, then mark paid once. For `success` `"false"`, mark failed unless a later success exists.
6. Track modifications by `originalReference` (payment) plus `pspReference` (modification).
