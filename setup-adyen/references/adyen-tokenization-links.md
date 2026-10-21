# Adyen Tokenization (recurring) and Payment Links

Checked 2026-10-21. Field tables: `adyen-fields.md` (`StoredPaymentMethod*`, `PaymentLink*`, `RecurringToken*`).

## Source URLs

- `https://docs.adyen.com/online-payments/tokenization`
- `https://docs.adyen.com/online-payments/tokenization/create-tokens`
- `https://docs.adyen.com/online-payments/tokenization/make-token-payments`
- `https://docs.adyen.com/online-payments/tokenization/managing-tokens`
- `https://docs.adyen.com/development-resources/webhooks/webhook-types` (Recurring tokens life cycle events)
- `https://docs.adyen.com/unified-commerce/pay-by-link`
- `https://docs.adyen.com/unified-commerce/pay-by-link/create-payment-links/api`
- OpenAPI: `json/CheckoutService-v72.json`, `json/TokenizationNotification-v1.json`, `json/RecurringService-v68.json` in `https://github.com/Adyen/adyen-openapi`

## Tokenization

Scope for Malaysia: cards (and Apple Pay / Google Pay card tokens). None of the Malaysian local methods (FPX, DuitNow, TnG, GrabPay, Boost, Atome, Alipay, Alipay+) support recurring. Alipay+ rejects `storePaymentMethod: true`.

Requirements: API credential with the **Merchant Recurring role**; subscribe a webhook to **Recurring tokens life cycle events**.

### Create a token (with a payment)

| Where | Fields |
| --- | --- |
| `POST /sessions` (v70+) | `shopperReference` (min 3, no personal data), `storePaymentMethodMode`: `disabled` (default), `askForConsent` (Drop-in shows a checkbox, cards only), `enabled` (always store); `recurringProcessingModel`; `shopperInteraction: "Ecommerce"` |
| `POST /payments` | `shopperReference`, `storePaymentMethod: true`, `recurringProcessingModel`, `shopperInteraction: "Ecommerce"` |
| `POST /storedPaymentMethods` | Create a token without a payment (card details encrypted), `shopperReference`, `recurringProcessingModel` |

`recurringProcessingModel` values: `CardOnFile` (shopper-present one-click), `Subscription` (fixed schedule), `UnscheduledCardOnFile` (merchant-initiated, variable timing, for example top-ups).

Customer Area > **Payments** > **Checkout settings** > **Tokenization**: a Recurring toggle can tokenise automatically when `shopperReference` is sent. Token Groups (via Support) share tokens across merchant accounts.

### Get the token

Webhook `recurring.token.created` (HMAC in `hmacsignature` header over the raw body):

```json
{
  "createdAt": "2026-10-21T10:00:00+08:00",
  "environment": "test",
  "eventId": "<PSP_REFERENCE_OF_EVENT>",
  "type": "recurring.token.created",
  "data": {
    "merchantAccount": "<MERCHANT_ACCOUNT>",
    "shopperReference": "<SHOPPER_REFERENCE>",
    "storedPaymentMethodId": "<TOKEN>",
    "type": "visa",
    "operation": "..."
  }
}
```

Other types: `recurring.token.updated`, `recurring.token.disabled`, `recurring.token.alreadyExisting`. Some responses also carry `additionalData.tokenization.storedPaymentMethodId` (and `tokenization.shopperReference`, `tokenization.store.operationType`). Legacy: `RECURRING_CONTRACT` standard event.

### Pay with a token

| Use case | `shopperInteraction` | `recurringProcessingModel` | Other |
| --- | --- | --- | --- |
| One-click (shopper present) | `ContAuth` | `CardOnFile` | `paymentMethod.storedPaymentMethodId`, same `shopperReference`, `paymentMethod.encryptedSecurityCode` (CVC) unless Support disabled the CVC requirement |
| Subscription | `ContAuth` | `Subscription` | `paymentMethod.storedPaymentMethodId`, `shopperReference` |
| Unscheduled | `ContAuth` | `UnscheduledCardOnFile` | same |

In `/sessions` one-click, Drop-in shows stored methods when `shopperReference` matches; Adyen forces `recurringProcessingModel` to `CardOnFile`. Merchant-initiated payments go through `POST /payments` and return the result in the response and the `AUTHORISATION` webhook. `paymentMethod.brand` is needed only for co-badged cards.

Rules: tell the shopper the amount and schedule before storing; for subscriptions and trials follow the reminder and cancellation rules in `account-setup.md` (website requirements).

### Manage tokens

- `GET /storedPaymentMethods?shopperReference=<REF>&merchantAccount=<MERCHANT_ACCOUNT>` list.
- `DELETE /storedPaymentMethods/{storedPaymentMethodId}?shopperReference=<REF>&merchantAccount=<MERCHANT_ACCOUNT>` disable.
- Classic Recurring API v68 (`https://pal-test.adyen.com/pal/servlet/Recurring/v68`): `/disable`, `/listRecurringDetails`, `/notifyShopper`, `/scheduleAccountUpdater`; `/createPermit`, `/disablePermit` deprecated. Prefer the Checkout endpoints.

## Payment Links (Pay by Link)

Source: `https://docs.adyen.com/unified-commerce/pay-by-link/create-payment-links/api`

1. `POST /paymentLinks` with `merchantAccount`, `reference`, `amount` (minor units). Optional: `countryCode` (`MY`), `shopperLocale`, `description`, `expiresAt`, `reusable`, `shopperReference` + `storePaymentMethodMode` + `recurringProcessingModel` (tokenize), `lineItems`, `allowedPaymentMethods`, `blockedPaymentMethods`, `themeId`, `requiredShopperFields`.
2. Response: `id`, `url`, `expiresAt`, `status`.
3. Send `url` to the shopper (email, chat). From 2025-11-01 links cannot be embedded in an iframe.
4. Wait for the `AUTHORISATION` webhook (`success` true = paid; with manual capture you still need to capture). `additionalData.paymentLinkId` links it back.
5. `GET /paymentLinks/{linkId}` returns `status`: `active`, `completed`, `expired`, `paymentPending`.
6. Force expiry: `PATCH /paymentLinks/{linkId}` with `{ "status": "expired" }`.

Expiry: default 24 hours after creation; maximum 70 days (source: `PaymentLinkRequest.expiresAt` in Checkout OpenAPI v72). `reusable: true` makes a link payable by many shoppers. Customer Area also creates links without code (Pay by Link > Payment links), and the Pay by Link page requires your terms and conditions and policies to be set (see `account-setup.md`).
