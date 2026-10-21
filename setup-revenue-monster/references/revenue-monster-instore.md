# Revenue Monster In-Store: Quick Pay, Terminal, POS, Deeplink

Offline payment flows. Quick Pay is billed at the offline subscription rate (docs). All Open API calls are signed (Bearer token plus `X-Signature`, `X-Nonce-Str`, `X-Timestamp`). Amounts in sen, minimum `10`. Checked 2026-10-21.

Sources:

- `https://doc.revenuemonster.my/docs/v2/payment/quick-pay`
- `https://doc.revenuemonster.my/docs/v2/payment/terminal-integration`
- `https://doc.revenuemonster.my/docs/pos/payment`, `/docs/pos/cancellation`, `/docs/pos/card-refund`, `/docs/pos/card-settlement`
- `https://doc.revenuemonster.my/docs/v2/payment/deeplink-integration`
- `https://doc.revenuemonster.my/docs/campaign/gourmet-card`
- `https://github.com/RevenueMonster/applink-demo` (deeplink demo linked from the docs)

## Shared Order Object

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `order.id` | String | Yes | Max 24, unique. |
| `order.title` | String | Yes | Max 32. |
| `order.currencyType` | String | Yes | `MYR`. |
| `order.amount` | Uint64 | Yes | Sen. |
| `order.detail` | String | No | Max 600. The POS pages spell it `details` and mark it required; the v2 pages use `detail`. Use `detail` (matches the Transaction Object). |
| `order.additionalData` | String | No | Max 128. |

## Quick Pay (merchant scans customer wallet code)

`POST /v3/payment/quickpay`

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `authCode` | String | Yes | Code from the customer's wallet QR / barcode. |
| `ipAddress` | String | Yes | IPv4 / IPv6 of the terminal or app. |
| `storeId` | String | Yes | |
| `order.*` | object | Yes | Shared Order Object. |
| `voucher.code` | String | No | RM voucher code (Quick Pay + Voucher). Scan the voucher QR first, then the wallet QR. |
| `loyalty.type` | String | No | `QRCODE`, `PHONENUMBER`, `ID` (Quick Pay + Spending Loyalty). |
| `loyalty.memberId` | String | When `type=ID` | |
| `loyalty.countryCode`, `loyalty.phoneNumber` | String | When `type=PHONENUMBER` | |
| `extraInfo.type` | String | No | `MEMBERSHIP` (Quick Pay + Membership Card). |
| `extraInfo.reference` | String | No | Membership card ID. |

Response: `item` Transaction Object, `code`, or `error.*`. With loyalty `QRCODE`, `item.extraInfo.loyalty.qrcode` must be printed on the receipt.

Membership card (only Alipay Gourmet Card is supported): verify first with `POST /v3/membership/verify` body `{"memberCardId","storeId"}`; response `code` only.

Common Quick Pay errors: `AUTH_CODE_EXPIRED`, `AUTH_CODE_INVALID`, `AUTH_CODE_MIS_MATCH`, `AUTH_CODE_USED`, `PAYMENT_TIMEOUT` (reverse by `orderId` then retry with a new order).

## Terminal Integration (Event API)

Event calls are relayed to an RM terminal, so they can be slow and fail if the terminal is offline. If your device has its own scanner, use Quick Pay instead.

### Payment

`POST /v3/payment/terminal/quickpay`

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `terminalId` | String | Yes | |
| `type` | String | Yes | `E-WALLET`, `CARD`, or `RETAIL-QR` (DuitNow Retail QR; terminal app 2.18.0 or later). |
| `receiptType` | Uint | Yes | `1` merchant and customer copy, `2` customer copy, `3` no receipt. |
| `cameraType` | String | Yes | `FRONT` or `BACK` (for `E-WALLET`). |
| `order.*` | object | Yes | Shared Order Object. |

Response: `item` Transaction Object (`platform` `TERMINAL`, `type` `BANK_CARD` for cards, `extraInfo.card` with `inputType`, `maskNo`, `referenceId`, `secondaryReferenceId`; `payee.userId` for e-wallet).

### Other Events

`POST /v3/event/terminal`

| `type` | Extra fields | Response |
| --- | --- | --- |
| `REFUND` (card) | `data.transactionId` (required), `data.receiptType`, `data.reason`, `data.email` (required, matches refund PIN owner), `data.pin` (required, 6 digits) | `item` Transaction Object |
| `SETTLEMENT` (card batch) | `data.receiptType` (`1` print summary and detail, `3` background) | `summary.batchNo`, `summary.currencyType`, `summary.noOfTransactions`, `summary.settlementAt`, `summary.totalSalesAmount`; `transactions[]`: `amount`, `currencyType`, `transactionAt`, `transactionId`, `type` (`SALE`, `VOID`) |
| `CANCEL` | none | `code` |

All take `terminalId`. E-wallet terminal transactions are refunded or reversed with the server APIs (`POST /v3/payment/refund`, `POST /v3/payment/reverse`); card transactions must be refunded through the `REFUND` event.

Terminal error codes: `TERMINAL_BUSY`, `TERMINAL_RESTARTED`, `PAYMENT_IN_PROCESS`, `ORDER_CANCELLED_ACTIVITY_TIMEOUT`, `NO_CARD_PAY`, `SETTLEMENT_FAILURE`, `REFUND_FAILURE`, `CANCELLATION_ERROR`, `CANCELLATION_UNSUPPORTED` (see `revenue-monster-errors.md`).

## Deeplink (Android app on RM terminal)

For merchant apps running on RM terminals (Merchant App version above 2.10.0). Send an Android `Intent("REVENUE_MONSTER_PAYMENT")` with extras `packageName`, `receiverName` (your receiver activity class name), and `transactionType`:

| `transactionType` | Action | Extra fields |
| --- | --- | --- |
| `1` | Quick Pay (e-wallet) | `orderId`, `orderTitle`, `amount` (sen), `printReceipt` |
| `2` | Card payment | `orderId`, `orderTitle`, `amount`, `printReceipt` (MF919 ignores receipt control) |
| `3` | Void transaction | `transactionId`, `reason`, `email`, `pin`, `printReceipt` |
| `4` | Wallet settlement | `print` |
| `5` | Card settlement | none |

Your receiver activity (exported, `android.intent.action.SEND`, `text/plain`) reads `intent.getStringExtra("result")`, a JSON string. Types 1 to 3 return `item` Transaction Object plus `code` / `error`. Type 4 returns `totalSalesAmount`, `totalSalesCount`, `totalRefundedAmount`, `totalRefundedCount`, `wallet[].name`, `wallet[].method`, `wallet[].region`, `wallet[].sales.count`, `wallet[].sales.amount`, `wallet[].refunded.count`, `wallet[].refunded.amount`, `range[]` (RFC3339). Type 5 returns the same `summary` and `transactions[]` as the `SETTLEMENT` event.
