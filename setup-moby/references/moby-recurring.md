# MOBY Recurring Payments, Tokens And Subscriptions

Use with `moby-api.md` (environments, auth, signature, errors). Checked 2026-10-21.

## Source URLs

- `https://docs.mobypay.my/docs/recurring-payments`
- `https://docs.mobypay.my/docs/api-reference` (Recurring Payments group)
- `https://dev.pay.mobycheckout.com/docs?api-docs.json` (tags `Tokens`, `Customers`, `Subscription`)
- `https://github.com/MobyPayTech/moby-plugins/tree/main/recurring-payments` (deprecated repo, extra response detail)
- `https://docs.mobypay.my/docs/guide-recurring-billing` (no-code dashboard Recurring Billing)

## Choose The Surface

- No-code: dashboard Recurring Billing (Billing Plans + Billing Links). MOBY runs the schedule, retries, reminders, refunds, customer portal. No API needed.
- Build it yourself: card tokenisation API (`/api/v2/tokens...`). You own the schedule and call "charge token" each cycle.
- Subscription API (`/api/merchant/subscription/...`): present only in the OpenAPI spec, not in the docs pages. Treat as less certain.

## Concepts

| Term | Meaning |
|---|---|
| Token | Reference to a saved card. Never store raw card numbers |
| Charge | One transaction against a token |
| Subscription | Billing plan: amount, frequency, retry logic |

All amounts are MYR decimal numbers (not cents). Base URLs are the same as the payment API: sandbox `https://dev.pay.mobycheckout.com`, production `https://pay.mobycheckout.com`. The deprecated GitHub README's sandbox host `https://dev-pay-refactor.mobycheckout.com/` is superseded by the current docs.

## Token Endpoints

### Create token (start tokenisation)

`POST /api/v2/tokens`, Bearer.

| Field | Type | Required |
|---|---|---|
| `client_id` | string | yes |
| `customer_email` | string (email) | yes |
| `customer_name` | string | yes |
| `customer_mobile` | string | yes (e.g. `60123456789`) |

201: `{ "success": true, "redirect_url": "https://pay.mobycheckout.com/mpgs/save-card?merchantId=...&customerName=..." }`. Redirect the customer there to enter card details on MOBY's page (the save-card route returns an HTML form, not JSON). "After they enter their card, MobyPay stores the token and calls your callback." The tokenisation callback payload and how to set its URL are not documented (checked 2026-10-21); retrieve the token with List tokens filtered by the customer's email or mobile.

Error example: `{ "success": false, "error": "Invalid customer information" }`.

### List tokens

`GET /api/v2/tokens?client_id=<CLIENT_ID>`, Bearer. Optional query: `email`, `name` (partial match), `mobile`, `page`, `per_page` (default 15).

Response `data[]` fields (docs API Reference / OpenAPI): `id`, `merchant_id`, `customer_id`, `gateway_id`, `provider` (e.g. `mastercard`), `card_type` (e.g. `debit`), `acquirer_token`, `token`, `firstSix`, `lastFour`, `card_expiry` (`MMYY`, e.g. `0139`), `agreement_id`, `payment_confirmation`, `transaction_id`, `created_at`, `updated_at`, `deleted_at`, `customer{ id, name, email, mobile, ... }`. Plus `links{first,last,prev,next}`, `meta{current_page, from, last_page, path, per_page, to, total}`, `success`.

The deprecated README shows an older flat shape (`current_page`, `per_page`, `total`, `last_page`, `data[]{provider, type, token, firstSix, lastFour, expiryYearMonth, customer}`). Parse defensively.

### Delete token

`DELETE /api/v2/tokens/{token}?client_id=<CLIENT_ID>`, Bearer. OpenAPI additionally requires a JSON body `{ "customer_mobile": "<mobile>" }`; the docs page does not. Send it.

200: `{ "success": true, "message": "Token deleted successfully" }`. 404: `{ "success": false, "error": "Token not found" }`.

### Charge token

`POST /api/v2/tokens/{token}/charges`, Bearer.

| Field | Type | Required | Notes |
|---|---|---|---|
| `client_id` | string | yes | Docs: body. OpenAPI: query parameter. Send in both to be safe |
| `amount` | number | yes | MYR, minimum 0 |
| `order_reference` | string | yes | Unique per charge. Your idempotency and status key |
| `3ds` | boolean | yes | `true`: response gives `redirect_url` for the customer. `false`: charged immediately |
| `return_url` | string | no | After 3DS |
| `callback_url` | string | no | Webhook for charge result |
| `merchant_reference` | string | no | |
| `skip_receipt` | boolean | no | Receipt emails go to customer and merchant unless `true` |
| `details` | string | no | Description |
| `custom_data` | object or null | no | Docs: must be a JSON object or null, strings/numbers/arrays rejected. OpenAPI and deprecated README: JSON string. Conflict: test in sandbox |
| `customer_mobile` | string | OpenAPI: yes | "Must be an existing customer mobile number". Not in the docs page. Send it |

Responses:

- `3ds: true`: `{ "success": true, "redirect_url": "https://secure-3ds.com/authenticate?ref=xyz" }`
- `3ds: false`: `{ "success": true, "message": "Successfully charged", "data": { "referenceNo", "status", "amount", ... } }`. The deprecated README example has `status: "pending"` with `transactions[]` (`transactionId`, `status`, `amount`, `payMethod` `MPGS`, `payAt`, `createdAt`). A `success: true` response is not proof of capture: check `data.status` or the charge status endpoint.
- Decline: `{ "success": false, "error": "DECLINED - Do not honor", "message": "05 - Do not honor", "data": null }`

The charge callback payload format is not separately documented. Assume the standard callback (`moby-api.md`) and verify its signature; if it differs, settle from `GET /api/v2/charges/{order_reference}`.

### Charge status

`GET /api/v2/charges/{order_reference}?client_id=<CLIENT_ID>`, Bearer.

Success:

```json
{ "success": true, "message": "Charge Success",
  "data": { "result": "SUCCESS",
    "response": { "gatewayCode": "APPROVED", "acquirerCode": "00", "acquirerMessage": "Approved",
      "transaction": { "id": "TXN-...", "amount": 100.50, "currency": "MYR", "timeOfRecord": "2025-04-25T12:34:56.000Z" } } } }
```

Failure: `success: false`, `data.result: "ERROR"`, `data.response.gatewayCode: "DECLINED"`, `gatewayRecommendation`, `acquirerCode` (e.g. `05`), `acquirerMessage`. 404 when not found.

Treat as paid only when `data.result` is `SUCCESS`, `gatewayCode` is `APPROVED`, and `transaction.amount` / `currency` match.

### Refund charge

`POST /api/v2/charges/{order_reference}/refund`, Bearer. Body: `amount` (required, at most the original), `client_id` (docs body; OpenAPI query). 201 `{ "success": true, "message": "Refund request submitted." }`. Refunds are queued and processed asynchronously; poll charge status or wait for a callback before marking refunded.

### Move customer mobile (OpenAPI only)

`POST /api/v2/customers/mobile`, Bearer. Body: `client_id`, `old_mobile` (e.g. `+60123456789`), `new_mobile`. Renames the latest customer with `old_mobile`, or adopts the existing `new_mobile` customer, and re-points every card token. 200 `{ "success": true, "message": "..." }`; 404 no customer for old mobile.

## Subscription API (OpenAPI only)

All Bearer, JSON body, `clientId` camelCase.

| Method | Path | Body fields |
|---|---|---|
| POST | `/api/merchant/subscription/list` | `clientId` |
| POST | `/api/merchant/subscription/create` | `clientId`, `title`, `description`, `term`, `period`, `retry_gap_days`, `retry_max_attempt`, `term_amount` (all required, strings in examples); optional `total_amount`, `increment_after_period`, `increment_amount` |
| POST | `/api/merchant/subscription/{subscriptionId}/update` | same as create |
| POST | `/api/merchant/subscription/subscriber/create` | `clientId`, `name`, `email` (required); `mobile`, `subscription_id` |
| POST | `/api/merchant/subscription/subscriber/add` | `clientId`, `subscription_id`, `customer_id` (all required) |
| POST | `/api/merchant/subscription/subscriber/get` | `subscription_id`, `clientId` (both optional filters) |
| POST | `/api/merchant/subscription/subscriber/create-bulk` | `clientId` (required); `subscribers[]{name,email,mobile}`, `subscription_id` |

Field meanings: `term` "Monthly, Daily, Yearly" (example value `"4"`, so the encoding is unclear), `period` number of units, `retry_gap_days`, `retry_max_attempt`, `term_amount` per-term charge, `increment_after_period` / `increment_amount` step-up pricing.

List response `subscriptions[]`: `id`, `title`, `description`, `term`, `period`, `retry_gap_days`, `retry_max_attempt`, `merchant_id`, `customer_id`, `payment_link`, `return_url`, `callback_url`, `total_amount`, `term_amount`, `increment_after_period`, `increment_amount`, `consent_url`, `consent_slug`, `reference_prefix`, `status` (1 active, 0 inactive), `created_at`, `updated_at`, `staff_id`. Create/update return `{ "status": true, "message": "Subscription created successfully!" }`. Subscriber create returns `id`; bulk returns `success[]` and `failure[]` (each with `error`).

How a subscriber authorises payment (consent URL flow) and how subscription charge results are notified are not documented (checked 2026-10-21). Confirm with MOBY before relying on this API; the dashboard Recurring Billing feature is fully documented.

## Dashboard Recurring Billing (no code)

Dashboard > Recurring Billing has Recurring Bills, Billing Plans, Billing Links, Settings.

1. Create a Billing Plan: name, description, interval (every N days/weeks/months/years), amount in MYR, "Allow customer to cancel", internal notes, status. Interval and amount are locked after creation. Maximum 30 years of cycles.
2. Create a Billing Link on the plan: purpose, reference, optional contact, expiry, usage cap, start now or after a trial, optional total cycles, optional one-time add-ons.
3. Share the link. Each signup becomes its own Recurring Bill. Customer email is required. Methods: card, Apple Pay, Google Pay.

Recurring Bill statuses: Authenticated (trial), Active, Pending (retrying), Halted (retries exhausted), Paused, Cancelled (final), Completed (final).

Daily schedule (Malaysia time): 05:00 reminders for charges within 3 days; 05:30 finalise end-of-cycle cancellations; 05:45 trials go Active; 06:00 charge bills due; 06:30 retries; 07:00 resolve unclear results; 07:15 card-expiry notices (within 30 days).

Retries: Max Retry Attempts 1 to 5, Days Between Retries 1 to 7, lifetime cap of 10 charge attempts per invoice. Declines count against the budget; temporary system issues do not; unusable payment method pauses the bill.

Refunds: from a bill's Payments tab, full refund of any paid invoice, marked refunded only after the gateway succeeds.

Customer portal: private link in every email, protected by a one-time email code (session at least 7 days). Customers can view history, download receipts, update card/wallet, cancel if the plan allows.
