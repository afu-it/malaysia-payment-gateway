# SecurePay Codes and Enums

Bank lists change. Fetch them live (`GET /public/v1/banks/b2c` or `/b2b` on v1, `GET /v1/paynet/:gateway_name/banks` on v2) instead of hard-coding. The tables below are for reference and tests. Checked 2026-10-21.

## FPX B2C (retail) bank codes

Production public list `https://securepay.my/api/public/v1/banks/b2c`, fetched 2026-10-21:

| Code | Name |
| --- | --- |
| `ABB0233` | Affin Bank |
| `AGRO01` | Agrobank |
| `ABMB0212` | Alliance Bank (Personal) |
| `AMBB0209` | AmBank |
| `BIMB0340` | Bank Islam |
| `BMMB0341` | Bank Muamalat |
| `BOCM01` | Bank of China |
| `BKRM0602` | Bank Rakyat |
| `BSN0601` | BSN |
| `BCBB0235` | CIMB Bank |
| `HLB0224` | Hong Leong Bank |
| `HSBC0223` | HSBC Bank |
| `KFH0346` | KFH |
| `MBB0228` | Maybank2E |
| `MB2U0227` | Maybank2U |
| `MBSB001` | MBSB Bank Berhad |
| `OCBC0229` | OCBC Bank |
| `PBB0233` | Public Bank |
| `RHB0218` | RHB Bank |
| `SCB0216` | Standard Chartered |
| `UOB0226` | UOB Bank |

Docs examples also list `CIT0219` Citibank (shown offline in v2 sample).

## FPX B2B1 (corporate) bank codes

Production public list `https://securepay.my/api/public/v1/banks/b2b`, fetched 2026-10-21 (`model` returned `B2B1`):

| Code | Name |
| --- | --- |
| `ABB0235` | AFFINMAX |
| `AGRO02` | Agrobank |
| `ABMB0213` | Alliance Bank (Business) |
| `AMBB0208` | AmBank |
| `BIMB0340` | Bank Islam |
| `BMMB0342` | Bank Muamalat |
| `BNP003` | BNP Paribas |
| `BCBB0235` | CIMB Bank |
| `CIT0218` | Citibank Corporate Banking |
| `DBB0199` | Deutsche Bank |
| `HLB0224` | Hong Leong Bank |
| `HSBC0223` | HSBC Bank |
| `BKRM0602` | i-bizRAKYAT |
| `KFH0346` | KFH |
| `MBB0228` | Maybank2E |
| `MBSB001` | MBSB |
| `OCBC0229` | OCBC Bank |
| `PBB0234` | PB Enterprise |
| `PBB0233` | Public Bank |
| `RHB0218` | RHB Bank |
| `SCB0215` | Standard Chartered |
| `UOB0228` | UOB Regional |

Send `model=B2B1` with a B2B1 bank code. Some codes (e.g. `BIMB0340`, `MBB0228`) appear in both lists.

## Sandbox test banks (FPX)

| Code | Name |
| --- | --- |
| `TEST0021` | SBI Bank A |
| `TEST0022` | SBI Bank B |
| `TEST0023` | SBI Bank C |

Docs sandbox samples also include `ABB0234` (Affin B2C - Test ID) and `UOB0229` (UOB Bank - Test ID). Sandbox only.

## Payment status values

| Field | Values | Source |
| --- | --- | --- |
| `payment_status` (v1) | `true` success, `false` failed | v1 status, callback |
| `payment_stage` (v1) | `payment_successful`, `payment_unsuccessful` (observed in samples) | v1 list/search, callback example |
| `fpx_status` (v1) | `true`/`false` | v1 callback example, list |
| `status` (v2 record) | `successful`, `failed` observed; `pending`, `succeeded`, `failed` listed | v2 status, Payment Records |
| `payment.status` (v2 callback) | `successful` observed | v2 callback |
| Mandate `status` (v2) | `pending`, `active`, `failed`, `revoked` | v2 Initiate eMandate |

A complete status enum is not documented.

## FPX debit auth code (`fpx_debit_auth_code`)

| Code | Meaning (SecurePay docs) |
| --- | --- |
| `00` | Success |
| `99` | Pending approval (B2B1 corporate). A later callback sends `00` after approval |
| other | Unsuccessful. Example in docs: `76` with `fpx_status_message` "Transaction Not Found" |

SecurePay does not publish the full FPX code list; `fpx_status_message` carries the text.

## Other enums

| Field | Values |
| --- | --- |
| `model` / `fpx_model` | `B2C`, `B2B1` |
| `fpx_bank_selection` | `dropdown`, `grid` |
| `payment_mode` | `merchant`, `customer` (also `buyer` in list docs), `credit` / `merchant_credit` |
| `payment_method` (v1) | `fpx_online_banking`, FPX credit card |
| `source` (v1) | `FPX`, `Credit Card`, other |
| `search_reference` (v1 search) | `fpx_ref_number`, `order_number`, `exchange_number` |
| `payment_status` filter (v1 list) | `true`, `false`, `all` |
| Payment Session `currency` (v1) | `USD` only (converted to MYR) |
| v2 `gateway_name` | `fpx`, `direct_debit`, `duitnow` |
| v2 `bank_type` | `b2c`, `b2b1` (fpx, direct_debit); `retail`, `corporate` (duitnow) |
| v2 `gateway_type` | `Fpx`, `Duitnow` (from `/v1/interface/gateways`) |
| v2 `payment_method` | `Fpx`, `DirectDebit`, `ClosedWallet` |
| v2 `id_type` | `1` new IC, `2` old IC, `3` passport, `4` business registration, `5` others |
| v2 `mode_frequency` | `DL`, `WK`, `MT`, `YR` |
| v2 `application_type` | `01` enrollment |
| v2 split `mode` | `fixed`, `percentage` |
| v2 Charge Mode | `Merchant`, `Credit`, `Customer`, `Inherit` (gateway level) |
