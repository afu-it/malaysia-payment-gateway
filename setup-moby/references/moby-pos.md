# MOBY POS, Kiosk And Installment Plans (IPP)

In-store integration. Online checkout lives in `moby-api.md`. Checked 2026-10-21.

## Source URLs

- `https://docs.mobypay.my/docs/instore-overview`
- `https://docs.mobypay.my/docs/instore-tcp`
- `https://docs.mobypay.my/docs/instore-messages`
- `https://docs.mobypay.my/docs/instore-ipp`
- `https://docs.mobypay.my/docs/instore-platforms` (Kotlin, Swift, C#, Python examples)
- `https://github.com/MobyPayTech/moby-plugins/tree/main/pos-kiosk-integration` (`kiosk.py`, `TESTING.md`, PDF)
- `https://dev.pay.mobycheckout.com/docs?api-docs.json` (tags `Installment Plans`, `Visa Installment`)

## Model

The Kiosk app runs a TCP server on port 8080 (configurable). The POS terminal connects as a TCP client. Same protocol on one device (`127.0.0.1`), LAN (local IP) or internet (public IP/hostname; wrap in TLS).

Payment modes: `card`, `bnpl`, `duitnow_qr`, `ipp`.

## Framing And Envelope

Newline-delimited JSON, one object per line:

```json
{ "payload": { "type": "...", "txn_id": "TXN1705123456789", "timestamp": "1705123456789",
               "nonce": "<uuid-v4>", "pos_id": "KIOSK001" },
  "signature": "<hex hmac>" }
```

## Signing (HMAC-SHA256 over the payload object)

1. Sort payload keys alphabetically.
2. Serialise as compact JSON (separators `,` and `:`, no spaces), UTF-8.
3. `hex(HMAC_SHA256(shared_secret, json))`.

```python
data = json.dumps(dict(sorted(payload.items())), separators=(',', ':'))
sig  = hmac.new(secret.encode(), data.encode(), hashlib.sha256).hexdigest()
```

Python `json.dumps` escapes non-ASCII by default; the docs' Node example uses `JSON.stringify`, which does not. Keep payload values ASCII or agree on one serialiser on both ends.

Receive validation order: structure (`payload` and `signature` present), HMAC match (constant time), nonce never seen before (UUID v4), timestamp within plus or minus 60 seconds (clocks NTP-synced).

The default test secret `POS-KIOSK-SECRET-KEY-2024` is public. Production must use a random secret of at least 32 characters, distributed out of band.

## Messages

Kiosk to POS:

| type | Extra fields |
|---|---|
| `transaction_request` | `txn_id` (`TXN` + epoch ms), `amount` (number, MYR), `payment_mode`, `pos_id`, `timestamp` (epoch ms string), `nonce` |
| `cancel_transaction` | `txn_id`, `pos_id`, `timestamp`, `nonce` |
| `ipp_plan_selection` | `txn_id`, `plan_id` (e.g. `IPP_3M`), `pos_id`, `timestamp`, `nonce` |

POS to Kiosk:

| type | Fields | status values |
|---|---|---|
| `ack` | `txn_id`, `status`, `timestamp`, `nonce` | `received`, `processing` |
| `transaction_result` | `txn_id`, `status`, `authorization_code`, `card_last4`, `timestamp`, `nonce` | `success`, `failed`, `ipp_plans` |

When `status` is `ipp_plans`, the result carries `amount` and `plans[]`: `planId`, `frequency` (e.g. `Monthly`), `totalInstallments`, `installmentDetails[]{installmentNumber, date, amount, installmentFee, installmentFeePercentage}`. Show them, then send `ipp_plan_selection`.

## IPP Flow

1. POS starts payment with `payment_mode: "ipp"`.
2. POS hardware returns `status: "ipp_plans"`.
3. Kiosk shows plans; customer picks one; Kiosk sends `ipp_plan_selection`.
4. POS confirms and processes.

## IPP REST Endpoints

Docs curl examples send no auth header; the OpenAPI spec marks both with the `x-token` header scheme. How to obtain an `x-token` is not documented (checked 2026-10-21).

`GET /api/omnipanel/terminal/installment-plans` query: `amount` (MYR, req), `reference_no` (req), `enc_account_number` (encrypted card account number from the terminal, req), `client_id` (req).

- Docs response: array of `{ plan_id, tenure, monthly_payment, interest_rate, total_payment, currency }`.
- OpenAPI response: `{ amount, currency, plans[]{ vPlanID, installment_frequency, total_installments, installment_details[]{installment_number, date, amount, installment_fee, installment_fee_percentage} } }`.
- Shapes differ; parse both. 400 bad request, 404 not found.

`POST /api/omnipanel/terminal/installment-plans` body: `plan_id`, `auth_code` (from terminal), `client_id`, `reference_no` (all required). 201 returns `vPlanAcceptanceID`, `vPlanID`, `paymentAccountReference`, `transactionInfo{ vTransactionID, externalTransactionRef, authCode, currency, category, acquirerReferenceNumber, transactionAmount, dateTime, installmentTransactions[]{ vInstallmentTransactionID, status, clearingAmount, settlementCurrency, acquirerReferenceNumber, clearingDateTime, clearingSequenceNumber } }`, `acceptedDateTime`.

`DELETE /api/visa/cancel-plan?clientId=<CLIENT_ID>`, Bearer. Body: `clientId`, `referenceNo`, `refundAmount` (MYR). Cancels an active Visa installment plan and refunds that amount. 200 `{ "data": {} }`; 422 `{ "message": "Plan not found or already cancelled" }`; 500 `{ "message": "Something went wrong", "response": "" }`.
