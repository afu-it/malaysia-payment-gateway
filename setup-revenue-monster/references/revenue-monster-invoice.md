# Revenue Monster Invoice (Payment Link) API

Create a shareable invoice payment page (`invoiceUrl`) and track it. Checked 2026-10-21.

Sources:

- `https://doc.revenuemonster.my/docs/invoice/create-invoice`
- `https://doc.revenuemonster.my/docs/invoice/get-invoices`
- `https://doc.revenuemonster.my/docs/invoice/get-invoice-by-id`
- `https://doc.revenuemonster.my/docs/invoice/edit-invoice`
- `https://doc.revenuemonster.my/docs/invoice/cancel-invoice`

Note the path prefix: invoice endpoints live under `/api/v3/...`, not `/v3/...`. Hosts: production `https://open.revenuemonster.my`, sandbox `https://sb-open.revenuemonster.my`. The invoice pages do not repeat the auth headers; use the same Bearer token and RSA signature headers as other Open API calls and include the `/api/v3/...` path in the signed `requestUrl`. Whether invoice calls require the signature headers is not stated on the invoice pages (checked 2026-10-21).

## Endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/api/v3/invoice` | Create invoice. |
| GET | `/api/v3/invoices` | List invoices (filter, paginate). |
| GET | `/api/v3/invoice/{id}` | Get one invoice. |
| PATCH | `/api/v3/invoice/{id}` | Edit a `PENDING` invoice (send all required fields). |
| POST | `/api/v3/invoice/{id}/cancel` | Cancel a `PENDING` invoice (status becomes `EXPIRED`). |

## Create Invoice

`POST /api/v3/invoice`

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `storeId` | String | Yes | |
| `referenceId` | String | Yes | Your unique reference. |
| `merchantEmail` | String | No | Merchant payment notification email. |
| `payerEmail` | String | Yes | |
| `payerName` | String | Yes | |
| `emailSubject` | String | Yes | |
| `shouldSendEmail` | Boolean | No | Email the payer. |
| `expiryDateTime` | String ISO8601 | Yes | For example `2026-05-21T08:02:00.000Z`. |
| `amount` | Number | Yes | Sen (`100` = RM 1.00). |
| `currency` | String | Yes | `MYR`. |
| `description` | String | Yes | |
| `redirectUrl` | String | No | Payer redirect after payment. |
| `remarks` | String | No | |

Response: `item` Invoice object, `code` `SUCCESS`.

## Edit Invoice

`PATCH /api/v3/invoice/{id}`. Only `PENDING` invoices. Partial update is not supported: send every required field even if unchanged.

| Field | Required |
| --- | --- |
| `referenceId`, `storeId`, `payerEmail`, `emailSubject`, `expiryDateTime`, `amount`, `currency`, `description` | Yes |
| `payerName`, `shouldSendEmail`, `redirectUrl`, `remarks` | No |

Note: `payerName` is required on create but optional on edit, per the official tables.

Missing fields return HTTP 422: `{"error":{"code":"VALIDATION_ERROR","message":"Validations error","description":{"amount":"The amount field is required.", ...}}}`.

## List Invoices

`GET /api/v3/invoices` query parameters:

| Param | Type | Notes |
| --- | --- | --- |
| `storeId` | String | Filter. |
| `status` | String[] | Repeatable: `PENDING`, `PAID`, `EXPIRED`, `DELETED`. |
| `limit` | Number | Page size. |
| `cursor` | String | From previous response. |
| `start`, `end` | RFC3339 | Created-at range. |

Response: `cursor` (empty when done), `items[]` Invoice objects.

## Invoice Object

| Field | Type | Notes |
| --- | --- | --- |
| `id` | String | Invoice ID. |
| `referenceId` | String | |
| `recurringInvoiceId`, `invoiceGroupId` | String | Empty if not used. |
| `merchantId`, `merchantName`, `merchantLogoUrl`, `merchantEmail` | String | |
| `storeId` | String | |
| `payerEmail`, `payerName`, `payerPhoneNumber`, `payerCountryCode` | String | |
| `currency` | String | `MYR`. |
| `amount` | Number | Sen. |
| `paidAmount`, `feesPaidAmount`, `netPaidAmount` | Number | Sen. |
| `invoiceStatus` | String | `PENDING`, `PAID`, `EXPIRED`, `DELETED`. |
| `transactionId` | String | Set when paid; query it with `GET /v3/payment/transaction/{transactionId}`. |
| `invoiceUrl` | String | Shareable payment page, for example `https://sb-pg.revenuemonster.my/v1/invoice?invoiceId=...`. |
| `redirectUrl`, `tncUrl`, `remarks` | String | |
| `shouldSendEmail` | Boolean | |
| `emailSubject`, `description` | String | |
| `expiryAt`, `paidAt` | RFC3339 | `paidAt` empty if unpaid. |
| `createdDateTime`, `updatedDateTime` | RFC3339 | |
| `customFields` | Object | `null`; not in use. |

## Settlement Pattern

No invoice webhook or notify URL is documented (checked 2026-10-21). Poll `GET /api/v3/invoice/{id}` (or list with `status=PAID`), then confirm `transactionId` with Query Transaction before fulfilling. Match `amount`, `currency`, `referenceId`, and `storeId`.
