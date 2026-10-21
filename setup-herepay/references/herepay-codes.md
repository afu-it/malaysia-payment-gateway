# Herepay Codes and Enums

Enum and code tables from the official Herepay docs. Checked 2026-10-21. Use live `GET /api/v1/herepay/payment/channels` for the bank list at runtime; these tables are for reference, tests, and validation only.

Sources:

- Mapping Table: `https://herepay.readme.io/reference/banks`
- Get Payment Channels example: `https://herepay.readme.io/reference/get_api-v1-herepay-payment-channels`
- Status Codes: `https://herepay.readme.io/reference/error-codes`
- Callback: `https://herepay.readme.io/reference/post_callback`
- HTTP Status Codes: `https://herepay.readme.io/reference/success`, `/client-error`, `/server-error`
- Get Transactions examples: `https://herepay.readme.io/reference/get_api-v1-herepay-transactions`

## Payment Methods (`payment_method`)

| Value sent in initiate | Meaning | Notes |
| --- | --- | --- |
| `Online Banking` | FPX B2C (personal internet banking) | Single-step approval. |
| `Corporate Banking` | FPX B2B (corporate internet banking) | Maker then Authorizer; can stay pending until approved. |

Values returned elsewhere: transaction list examples show `payment_method` `FPX`; callback `fpx_type` examples are `FPX` and `CCA`. No card or e-wallet `payment_method` value is documented for initiate.

The Mapping Table page describes charges per method; the channels example returns a different fixed number. Charges change, so read `data[].charges` live and link to `https://www.herepay.org/harga/` instead of copying numbers.

## Bank Codes: Online Banking (B2C), production list

From the Mapping Table page.

| Bank ID (`bank_prefix`) | Member bank |
| --- | --- |
| `ABB0233` | Affin Bank |
| `AGRO01` | AGRONet |
| `ABMB0212` | Alliance Bank (Personal) |
| `AMBB0209` | AmBank |
| `BIMB0340` | Bank Islam |
| `BMMB0341` | Bank Muamalat |
| `BOCM01` | Bank of China |
| `BKRM0602` | Bank Rakyat |
| `BSN0601` | BSN |
| `BCBB0235` | CIMB Clicks |
| `CIT0219` | Citibank |
| `HLB0224` | Hong Leong Bank |
| `HSBC0223` | HSBC Bank |
| `KFH0346` | KFH |
| `MBB0228` | Maybank2E |
| `MB2U0227` | Maybank2U |
| `OCBC0229` | OCBC Bank |
| `PBB0233` | Public Bank |
| `RHB0218` | RHB Bank |
| `SCB0216` | Standard Chartered |
| `UOB0226` | UOB Bank |

## Bank Codes: Corporate Banking (B2B), production list

From the Mapping Table page.

| Bank ID (`bank_prefix`) | Member bank |
| --- | --- |
| `ABB0235` | AFFINMAX |
| `AGRO02` | Agrobank |
| `ABMB0213` | Alliance Bank (Business) |
| `AMBB0208` | AmBank |
| `BIMB0340` | Bank Islam |
| `BMMB0342` | Bank Muamalat |
| `BKRM0602` | Bank Rakyat |
| `BNP003` | BNP Paribas |
| `BCBB0235` | CIMB Clicks |
| `CIT0218` | Citibank Corporate Banking |
| `DBB0199` | Deutsche Bank |
| `HLB0224` | Hong Leong Bank |
| `HSBC0223` | HSBC Bank |
| `KFH0346` | KFH |
| `MBB0228` | Maybank2E |
| `OCBC0229` | OCBC Bank |
| `PBB0234` | PB Enterprise |
| `PBB0233` | Public Bank |
| `RHB0218` | RHB Bank |
| `SCB0215` | Standard Chartered |
| `UOB0229` | UOB Bank |
| `UOB0228` | UOB Regional |

Some codes appear in both lists (for example `BIMB0340`, `HLB0224`, `PBB0233`). Always send the `bank_prefix` together with the `payment_method` of the channel group it came from in the live channels response.

## Test Banks (UAT only)

| Bank ID | Name |
| --- | --- |
| `ABB0234` | Affin B2C - Test ID |
| `LOAD001` | LOAD001 |
| `TEST0021` | SBI Bank A |
| `TEST0022` | SBI Bank B |
| `TEST0023` | SBI Bank C |

The UAT channels example lists `LOAD001`, `TEST0021`, `TEST0022`, `TEST0023` under both methods, and `ABB0234` under Online Banking only. It also includes a row with prefix `MBBM2U2` and empty `name`/`img_url`, and marks `CIT0219` (Citibank) `Inactive`. Simulator logins (B2C, B2B Maker `1234`/`1234`, B2B Authorizer portal) are in `references/herepay-api.md` under Sandbox Test Data.

## Transaction Status Codes (Status Codes page)

| Code | Name | Message |
| --- | --- | --- |
| `00` | `Success` | The operation was completed successfully. |
| `29` | `Pending` | The operation is in progress and awaiting confirmation. |
| `30` | `Failed` | The operation could not be completed successfully. |
| `41` | `Unauthorized` | You are unauthorized to perform this action. |

## Callback Value Sets (Callback OpenAPI)

| Field | Documented values |
| --- | --- |
| `status` | `Failed`, `Completed`, `Pending` |
| `status_code` | `0` Failed, `1` Completed, `2` Pending |
| `message` | `Approved`, `Failed`, `Pending` |
| `fpx_type` | Examples `FPX`, `CCA` |

Other values seen in official examples: `status` `Success`; `message` `Transaction Pending`, `Pending Authorization (Applicable for B2B model)`. The official PHP samples and WooCommerce plugin treat `00` as success, `29` pending, `30` failed, `41` unauthorized (failed).

Defensive mapping (apply only after checksum passes):

| Outcome | Match any of |
| --- | --- |
| Paid | `status_code` `00` or `1`; `status` `Success` or `Completed` |
| Pending | `status_code` `29` or `2`; `status` `Pending` |
| Failed | `status_code` `30`, `0`, or `41`; `status` `Failed` |
| Unknown | Anything else: keep pending, log, requery |

## HTTP Status Codes

| Code | Text | Meaning |
| --- | --- | --- |
| `200` | OK | `POST` processed without errors. |
| `201` | Created | `POST` processed and a resource was created. |
| `204` | No Content | Processed, no response body. |
| `400` | Bad Request | Invalid input or missing required parameters. |
| `401` | Unauthorized | Invalid Access ID or user token; may include an internal error code. |
| `403` | Forbidden | Denied: invalid API key, incorrect request signature, or no permission. |
| `404` | Not Found | Session or transaction not found. |
| `500` | Internal Server Error | Unexpected server problem; may include an internal error code. |
| `502` | Bad Gateway | Invalid response from an upstream server. |

No list of internal error codes is published (checked 2026-10-21).
