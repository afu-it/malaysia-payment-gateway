# Adyen Modifications (capture, refund, cancel, reversal, amount update)

Use this file for anything after authorisation. All modification calls are asynchronous: the API response only says Adyen received the request (`status: "received"`). The outcome arrives in a webhook. Checked 2026-10-21. Field tables: `adyen-fields.md` (`Payment*Request` / `Payment*Response`).

## Source URLs

- `https://docs.adyen.com/online-payments/capture`
- `https://docs.adyen.com/online-payments/refund`
- `https://docs.adyen.com/online-payments/cancel`
- `https://docs.adyen.com/online-payments/reversal`
- `https://docs.adyen.com/online-payments/adjust-authorisation`
- `https://docs.adyen.com/online-payments/modify-payments` (overview)
- `https://docs.adyen.com/account/payments-lifecycle`

## Common Rules

- Path parameter `paymentPspReference` = `pspReference` of the original payment (from the `AUTHORISATION` webhook or `/payments` response).
- Body always has `merchantAccount`; `reference` is your own id for the modification (recommended; required for GrabPay refunds: max 32, `[A-Za-z0-9_-]`).
- Send an `idempotency-key` header; retry with the same key on network errors or `transient-error: true`.
- Response: `merchantAccount`, `paymentPspReference`, `pspReference` (new, for the modification), `reference`, `status: "received"`, plus `amount` where relevant.
- Webhook: `originalReference` = payment PSP reference, `pspReference` = modification PSP reference. `success: "true"` means accepted (for refunds, sent to the scheme), not always money moved.
- Malaysian local methods (FPX, DuitNow, TnG, GrabPay, Boost, Atome, Alipay, Alipay+, WeChat Pay) do not support separate capture; Boost supports full refunds only. See `adyen-payment-methods-my.md`.

## Capture

Source: `https://docs.adyen.com/online-payments/capture`

Default: immediate automatic capture. Options:

- Delayed automatic capture: Customer Area > **Settings** > **Account settings** (merchant account) > **Capture delay**: immediate, 1 to 7 days, or manual. Per payment: `captureDelayHours` (max 672 on v72).
- Manual capture: set capture delay to **Manual**, or per payment `additionalData.manualCapture: "true"`.

Manual capture request:

```bash
curl https://checkout-test.adyen.com/v72/payments/<PAYMENT_PSP_REFERENCE>/captures \
  -H "x-API-key: <API_KEY>" -H "content-type: application/json" \
  -H "idempotency-key: <UUID_V4>" \
  -d '{ "merchantAccount": "<MERCHANT_ACCOUNT>", "amount": { "value": 4990, "currency": "MYR" }, "reference": "CAP-ORDER-10001" }'
```

- Webhooks: `CAPTURE` (only for manual capture by default; for delayed automatic capture enable it on the Webhooks settings page), `CAPTURE_FAILED` later if the scheme rejects. Adyen retries technical capture failures for up to 10 business days.
- Partial capture: send a lower `amount`. Multiple partial captures need Support to enable them; the remainder can be cancelled.
- Authorisations expire. Adyen default validity for global card schemes is 28 days; card scheme rules can be shorter. After expiry an `EXPIRE` webhook may arrive and capture fails. Source: `https://docs.adyen.com/online-payments/adjust-authorisation`
- Sandbox: trigger a failed capture with `holderName: "capture failed"` on a test card payment.

## Refund

Source: `https://docs.adyen.com/online-payments/refund`

```bash
curl https://checkout-test.adyen.com/v72/payments/<PAYMENT_PSP_REFERENCE>/refunds \
  -H "x-API-key: <API_KEY>" -H "content-type: application/json" \
  -H "idempotency-key: <UUID_V4>" \
  -d '{ "merchantAccount": "<MERCHANT_ACCOUNT>", "amount": { "value": 1000, "currency": "MYR" }, "reference": "RF-ORDER-10001-1", "merchantRefundReason": "CUSTOMER REQUEST" }'
```

- Only a captured payment can be refunded. To undo an uncaptured authorisation, cancel instead (or use reversal when unsure).
- `amount` can be partial; several partial refunds are allowed up to the captured amount (method permitting).
- `merchantRefundReason` values: `FRAUD`, `CUSTOMER REQUEST`, `RETURN`, `DUPLICATE`, `OTHER`.
- Webhooks: `REFUND` (validation result), then possibly `REFUND_FAILED` (scheme rejected; funds stay with you) or `REFUNDED_REVERSED` (shopper account could not receive; funds returned to you).
- Timing: it can take up to 40 days for the shopper to receive the funds, depending on the bank. Source: `https://docs.adyen.com/account/payments-lifecycle`
- Unreferenced refunds (no original payment) need the API Payment RefundWithData role via Support and report `REFUND_WITH_DATA`.
- Sandbox: `holderName: "refund failed"` triggers a failed refund.

## Cancel

Source: `https://docs.adyen.com/online-payments/cancel`

- With the PSP reference: `POST /payments/{paymentPspReference}/cancels` body `{ "merchantAccount", "reference" }`. Webhook: `CANCELLATION`.
- With your own reference (when you never got a PSP reference, for example a timeout): `POST /cancels` body `{ "merchantAccount", "paymentReference": "<YOUR_PAYMENT_REFERENCE>", "reference" }`. Webhook: `TECHNICAL_CANCEL`. Only up to 24 hours after authorisation.
- Cancel works only before capture.

## Reversal (cancel or refund)

Source: `https://docs.adyen.com/online-payments/reversal`

- `POST /payments/{paymentPspReference}/reversals` body `{ "merchantAccount", "reference" }`. Adyen cancels if not captured, else refunds the full amount.
- Webhook: `CANCEL_OR_REFUND` (`additionalData.modification.action` says `cancel` or `refund`).
- Not usable after multiple partial captures, or for payments with split data. Use it for "undo this whole payment" cases, for example an order that cannot be fulfilled.

## Amount Update (authorisation adjustment)

Source: `https://docs.adyen.com/online-payments/adjust-authorisation`

- `POST /payments/{paymentPspReference}/amountUpdates` body `{ "merchantAccount", "amount", "reference", "industryUsage" }` (`delayedCharge`, `installment`, `noShow`). Cards with pre-authorisation only; needs pre-auth setup (`additionalData.authorisationType: "PreAuth"`).
- Webhook: `AUTHORISATION_ADJUSTMENT`.
- Not relevant to Malaysian local methods.

## Payment Lifecycle Statuses (Customer Area)

Source: `https://docs.adyen.com/account/payments-lifecycle`

| Status | Meaning |
| --- | --- |
| `Received` | Request received; shopper must still complete authorisation. |
| `Open` / `OfferCancelled` | Offer pending / shopper did not complete. |
| `Authorised` | Approved by the financial institution. |
| `Cancelled`, `Refused`, `Error`, `Expired` | Cancelled, declined, processing error, authorisation expired. |
| `SentForSettle` | Capture sent; cannot be cancelled now. |
| `Settled` | Funds received by Adyen (not yet paid out to you; depends on payout model). |
| `SettledReversed` | Sales day payout only: funds not received within 30 days after capture. |
| `CaptureFailed` | Capture failed. |
| `SentForRefund`, `Refunded`, `RefundFailed`, `RefundedReversed` | Refund steps. |
| `Chargeback`, `SecondChargeback`, `ChargebackReversed` | Dispute steps (cards). |

Settlement has no webhook. Use the Settlement details report (`account-setup.md`).

## Disputes API (cards)

Source: `https://docs.adyen.com/risk-management/disputes-api`, OpenAPI `https://github.com/Adyen/adyen-openapi/blob/main/json/DisputeService-v30.json`

Base test URL: `https://ca-test.adyen.com/ca/services/DisputeService/v30`. Needs an API key with the **API dispute management** role (ask Support if missing). Operations (all POST, body has `disputePspReference` and `merchantAccountCode`): `/retrieveApplicableDefenseReasons`, `/supplyDefenseDocument` (`defenseDocuments[]` with base64 `content`, `contentType`, `defenseDocumentTypeCode`), `/deleteDisputeDefenseDocument`, `/defendDispute` (`defenseReasonCode`), `/acceptDispute`. Dispute events arrive as webhooks (`CHARGEBACK`, `NOTIFICATION_OF_CHARGEBACK`, `REQUEST_FOR_INFORMATION`, and others). Malaysian local methods have no chargebacks.
