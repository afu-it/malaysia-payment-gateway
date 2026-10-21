# Setel Settlement And Reconciliation

Sources (checked 2026-10-21):

- `https://developer.setel.com/docs/after-the-payment`
- `https://developer.setel.com/docs/reconciling-payments-with-settlements`
- `https://developer.setel.com/docs/accessing-settlement-reports`
- `https://developer.setel.com/docs/accessing-settlement-reports-sftp-connection`
- `https://developer.setel.com/docs/enabling-manual-settlement-for-virtualcardless-payments`
- `https://developer.setel.com/reference/create-a-settlement`

## How Settlement Works

- Partners give Setel a designated bank account during onboarding.
- Default is automatic settlement. The report compiles successful transactions from the previous day, 00:00 to 23:59 GMT+8.
- One settlement report file is generated daily at 01:00 GMT+8 if there is transaction data and the total payout is greater than zero. If the total is below zero, no file is generated and the day's data rolls over until both conditions are met.
- Bank processing time after the file is sent varies by bank.

## Getting Reports

Setel Dashboard (`https://dashboard.setel.my/login`, credentials provided by Setel):

1. Log in, select the merchant.
2. Settlement listing: filter by Status or Created; **Download CSV** emails the file (one or several addresses); click a settlement for details (bank info, merchant info, payments by category such as Payout and Charges, JSON tab).
3. The email contains a download link.

SFTP (one-time setup with the Setel Merchant Ops team):

- Report generated 01:00, connection made, file sent 06:00 GMT+8.
- Provide: host address (IP, string), port (integer), username, password, destination folder.
- Each connection is tied to one merchant ID and gets a connection ID. Multiple connections allowed if each points to a different IP. Edit anything except the connection ID; deleting requires a new connection.
- You may need to whitelist Setel's IP (provided by Setel).
- No file is sent on days without successful transactions.

## MPF File Format

File name: `MPF_<PayeeID>_<filedate>.csv` (MPF = Merchant Payment File).

Header (SFTP only, not in Dashboard downloads):

| Field | Format |
| --- | --- |
| Company name | string, max 100 |
| Company ID | string, max 100 |
| Merchant name | string, max 100 |
| Merchant ID | string, max 100 |
| File generation date and time | `yyyy-mm-dd`, `hh:mm:ss` (24h, GMT+8) |

Body (always present):

| Field | Description | Format |
| --- | --- | --- |
| Created at | Transaction time GMT+8 | `yyyy-mm-dd`, `hh:mm:ss` |
| Sub merchant reference | Merchant's own reference | string, max 100 |
| Sub merchant name | | string, max 100 |
| Terminal ID | | string, max 32 |
| Batch number | Terminal transactions | string, max 32 |
| STAN | System Trace Audit Number (terminal transactions) | string, max 32 |
| Transaction type | | "To be updated" in docs |
| Payment method | | string, max 32 |
| Reference type | Service type | "To be updated" in docs |
| ID | Setel transaction ID (`transactionID` in Dashboard) | string, max 32 |
| Original transaction ID | For refunds | string, max 32 |
| Settlement ID | Groups a settlement | string, max 32 |
| Reference ID | Integrator's `referenceId` | string, max 32 |
| POS Transaction ID | POS transactions | string, max 32 |
| Gross amount | Charge positive, refund negative, MYR | float, 2 dp |
| Total fee | Per MDR | float, 2 dp |
| Net amount | Merchant receives | float, 2 dp |
| Reserved | Future use | N/A |

Note: Reference ID is limited to 32 characters in the file. Keep `referenceId` at 32 characters or fewer if you reconcile on it (the API itself does not document a limit).

Brief (SFTP only): `Setel Ventures` constant, total number of records (integer), total gross, total fee (MDR), total net.

Reconciliation tips: match `Reference ID` and `ID` to stored `referenceId` and payment intent / transaction ids; refunds link through `Original transaction ID`; fees are deducted per transaction.

## Manual Settlement

Applies only to Setel-managed virtual transactions (type `prime` for Setel Wallet, `virtual loyalty` for loyalty points).

Enable: Dashboard > Settings > Payments > Edit > untick **Enable** for Automated Settlements.

### Create a settlement

`POST /v1/settlements/non-terminal-settlements`, partner auth.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `merchantId` | string | Yes | Merchant generating the settlement. |
| `idempotentId` | string | Yes | Unique per request; duplicates are rejected. |
| `enterpriseId` | string | Yes | Enterprise you are registered with. |

Response `200`: `id`, `merchantId`, `idempotentId`, `type` (example `PRIME`), `totalTransactions`, `status` (example `CREATED`), `amount` (RM).

Errors: `400` (empty `merchantId`, empty or duplicate `idempotentId`, invalid `enterpriseId`), `401`, `404` merchantId not found, `422` bank information not found, `500`, `503`.

Notes: you may settle more than once a day, but the bank compiles them into one daily settlement. A settlement equals the POS end-of-day till summary, not a cashier till summary. Bank processing timing does not change.
