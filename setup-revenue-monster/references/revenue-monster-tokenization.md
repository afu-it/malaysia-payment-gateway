# Revenue Monster Tokenization, Recurring, And Card-On-File

Recurring payment (RM's recurring engine), tokenized payment (your own billing engine), customer APIs, and card-on-file tokens. All calls are signed Open API calls (Bearer token plus `X-Signature`, `X-Nonce-Str`, `X-Timestamp`). Amounts are integers in sen. Checked 2026-10-21.

Sources:

- `https://doc.revenuemonster.my/docs/v2/payment/tokenization-payment`
- `https://doc.revenuemonster.my/docs/v2/payment/online-payment-backup` (Card-on-File Tokenization)
- `https://doc.revenuemonster.my/docs/ecom-plugin/lowCodeCheckout` (`customerId` for tokenization)

Hosts: production `https://open.revenuemonster.my`, sandbox `https://sb-open.revenuemonster.my`.

## Which Flow

| Need | Use |
| --- | --- |
| RM charges the card on a fixed schedule | Recurring Payment: `POST /v3/recurring-payment` |
| You decide when and how much to charge | Tokenized Payment: `POST /v3/tokenized-payment`, then `POST /v3/customer/{customer_id}/order` |
| Returning card customer in hosted checkout | Card-on-File: pass `customer.userId` in `POST /v3/payment/online`, then `GET /v3/payment/tokens/{customer_id}` |

## Create Recurring Customer

`POST /v3/recurring-payment`

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `storeId` | String | Yes | Store ID. |
| `email` | String | Yes | Customer email. |
| `name` | String | Yes | Customer name. |
| `countryCode` | String | Yes | For example `60`. |
| `phoneNumber` | String | Yes | Without country code, for example `187824152`. |
| `productName` | String | Yes | |
| `productDescription` | String | Yes | |
| `currency` | String | Yes | `MYR`. |
| `amount` | Uint64 | Yes | Sen per charge. |
| `redirectUrl` | String | Yes | Browser return after card bind. |
| `notifyUrl` | String | Yes | Called when a payment is made. |
| `recurringInterval` | String | Yes | `MONTHLY`, `WEEKLY`, `DAILY`. |
| `recurringTarget` | String | Yes | See target rules. |
| `recurringRepetition` | Uint64 | Yes | Number of charges. |
| `firstChargeDate` | String `YYYY-MM-DD` | No | Must match interval and target (WEEKLY target `0` means a Sunday; MONTHLY target `-1` means last day of month). |
| `chargeOnBind` | Bool | No | Charge immediately when the card is bound. |

Recurring target rules:

| Interval | Target | Behaviour |
| --- | --- | --- |
| `WEEKLY` | `0` to `6` | Sunday (`0`) to Saturday (`6`). |
| `MONTHLY` | `-1` | Last day of every month. |
| `MONTHLY` | `0` | First day of every month. |
| `MONTHLY` | `1` to `28` | Day of month. |
| `DAILY` | not documented | Target semantics for DAILY are not documented in official docs (checked 2026-10-21). |

Response `item` (Customer object):

| Field | Type | Notes |
| --- | --- | --- |
| `id` | String | Customer ID (use as `customer_id`). |
| `merchantId`, `storeId` | String | |
| `label` | String | Card first six and last four digits (after bind). |
| `email`, `countryCode`, `phoneNumber` | String | |
| `productName`, `productDescription` | String | |
| `isActive` | Bool (documented as String) | Becomes active after a successful bind. |
| `createdAt`, `updatedAt` | RFC3339 | |
| `clientKey` | String | Internal. |
| `redirectUrl`, `notifyUrl` | String | |
| `paymentUrl` | String | Send the customer here to bind their card. |
| `recurringPayment.amount` | Uint64 | |
| `recurringPayment.currency` | String | `MYR`. |
| `recurringPayment.recurringInterval` | String | |
| `recurringPayment.recurringTarget` | String | |
| `recurringPayment.recurringRepetition` | Uint64 | |

Plus `code` (`SUCCESS`) or `error.code`, `error.message`, `error.debug` (sandbox).

### Card Bind Redirect

`GET <redirectUrl>?status=...&customerId=...&reason=...`

| Param | Values |
| --- | --- |
| `status` | `SUCCESS`, `FAILED`, `CANCELLED` |
| `customerId` | Customer ID |
| `reason` | Optional failure reason |

Browser UX only. Confirm with `GET /v3/customer/{customer_id}` (`isActive`).

### Recurring Notify

Sent to `notifyUrl` only when a recurring charge succeeds. The docs table header says method `GET`, while the online payment notify is `POST`; accept both until verified in sandbox.

| Field | Type | Notes |
| --- | --- | --- |
| `eventType` | String | `RECURRING_PAYMENT` |
| `data.amount` | Uint64 | Sen. |
| `data.currency` | String | `MYR`. |
| `data.countryCode`, `data.phoneNumber`, `data.email`, `data.name` | String | Customer. |
| `data.customerId` | String | |
| `data.merchantId`, `data.storeId` | String | |
| `data.orderId` | String | Recurring transaction order ID; query it with `GET /v3/payment/transaction/order/{orderId}`. |
| `data.status` | String | `SUCCESS`. |
| `data.createdAt`, `data.updatedAt` | RFC3339 | |

## Create Tokenized Customer

`POST /v3/tokenized-payment`

| Field | Type | Required |
| --- | --- | --- |
| `storeId` | String | Yes |
| `email` | String | Yes |
| `name` | String | Yes |
| `countryCode` | String | Yes |
| `phoneNumber` | String | Yes |
| `productName` | String | Yes |
| `productDescription` | String | Yes |
| `redirectUrl` | String | Yes |

Response `item` is the Customer object without `recurringPayment` (includes `paymentUrl` for card bind). Redirect parameters are the same as Card Bind Redirect. No notify URL is taken on create; the response table lists a top-level `notifyUrl` field.

## Customer APIs (recurring and tokenized)

Available only after the customer has bound a card once.

| Method | Path | Purpose | Body | Response |
| --- | --- | --- | --- | --- |
| GET | `/v3/customer/{customer_id}` | Customer info | none | Customer object |
| GET | `/v3/customer/{customer_id}/orders` | Customer's orders | none | `item[]`: `id`, `recurringCustomerId`, `transactionId`, `amount`, `currency`, `createdAt`, `updatedAt` (plus `merchantId`, `storeId`) |
| PUT | `/v3/customer/{customer_id}/status` | Toggle active / inactive | none documented | Customer object |
| POST | `/v3/customer/{customer_id}/order` | Charge the bound card now | `currency` (`MYR`, required), `amount` (Uint64 sen, required), `title` (optional), `description` (optional) | Transaction Object |

`POST /v3/customer/{customer_id}/order` charges the stored card immediately; it is not a manual order record. Example body: `{"amount":100,"currency":"MYR","description":"Some order description","title":"Some order title"}` (keys sorted for signing). Check `item.status` and requery by `transactionId` before fulfilling.

## Card-On-File Tokenization (CoFT)

When a card payment is made in hosted checkout with `customer.userId` set (required when tokenization is enabled), RM stores a token for that customer ID. Call the token API after the transaction is `SUCCESS` or after notify; before that the token may not exist.

| Method | Path | Body | Purpose |
| --- | --- | --- | --- |
| GET | `/v3/payment/tokens/{customer_id}` | none | List tokens. |
| DELETE | `/v3/payment/tokens/{customer_id}` | `{"token":"<TOKEN>"}` | Remove a token. |

`customer_id` is the `customer.userId` you sent at checkout. Both return `item[]`:

| Field | Type | Notes |
| --- | --- | --- |
| `id` | String | Token ID. |
| `label` | String | |
| `provider` | String | `GOBIZ`. |
| `token` | String | Use this when paying. |
| `country` | String | `MY`. |
| `createdAt`, `updatedAt` | RFC3339 | |
| `card.name` | String | Cardholder name. |
| `card.method` | String | `MASTERCARD`, `VISA`. |
| `card.lastFourNo` | String | |
| `card.expMonth`, `card.expYear` | Uint64 | |

How a stored token is passed into Direct Payment Checkout is not documented in official docs (checked 2026-10-21); the docs only say tokens can be used "with Direct Payment Checkout API". Confirm the field with RM before building it.

Low-code checkout button: `customerId` attribute is "required for tokenization".
