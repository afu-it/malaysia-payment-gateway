# Tulus Codes and Enums

Source: `https://developers.tulus.my/sdk/callback.md`, `https://developers.tulus.my/zakat/`, `https://developers.tulus.my/schema/`, `https://developers.tulus.my/adb/` (checked 2026-10-21).

## Callback `status`

| Value | Meaning | Action |
|---|---|---|
| `success` | Approved (`fpx_debit_auth_code` = `00`) | Safe to fulfil |
| `pending` | Outcome unknown (auth code empty, `09`, `76`, `99`) | Wait for a follow-up callback or requery |
| `failed` | Declined or cancelled | Do not fulfil |

Indirect browser return only uses `success` and `failed`. Report API `fpxrequest.payment_status` maps the same way.

## `fpx_debit_auth_code`

| Code | Description | `status` |
|---|---|---|
| `00` | Approved | success |
| `09` | Transaction Pending | pending |
| `76` | Transaction Not Found | pending |
| `99` | Pending Authorization (B2B) | pending |
| (empty) | Not yet received | pending |
| `03` | Invalid Merchant | failed |
| `05` | Invalid Seller Or Acquiring Bank Code | failed |
| `12` | Invalid Transaction | failed |
| `13` | Invalid Amount | failed |
| `14` | Invalid Buyer Account | failed |
| `20` | Invalid Response | failed |
| `30` | Format Error | failed |
| `31` | Invalid Bank | failed |
| `39` | No Credit Account | failed |
| `45` | Duplicate Seller Order Number | failed |
| `46` | Invalid Seller Exchange Or Seller | failed |
| `47` | Invalid Currency | failed |
| `48` | Maximum Transaction Limit Exceeded | failed |
| `49` | Merchant Specific Limit Exceeded | failed |
| `50` | Invalid Seller for Merchant Specific Limit | failed |
| `51` | Insufficient Funds | failed |
| `53` | No Buyer Account Number | failed |
| `57` | Transaction Not Permitted | failed |
| `58` | Transaction To Merchant Not Permitted | failed |
| `70` | Invalid Serial Number | failed |
| `77` | Invalid Buyer Name Or Buyer ID | failed |
| `78` | Decryption Failed | failed |
| `79` | Host Decline When Down | failed |
| `80` | Buyer Cancel Transaction | failed |
| `83` | Invalid Transaction Model | failed |
| `84` | Invalid Transaction Type | failed |
| `85` | Internal Error At Bank System | failed |
| `87` | Debit Failed Exception Handling | failed |
| `88` | Credit Failed Exception Handling | failed |
| `89` | Transaction Not Received Exception | failed |
| `90` | Bank Internet Banking Unavailable | failed |
| `92` | Invalid Buyer Bank | failed |
| `96` | System Malfunction | failed |
| `98` | MAC Error | failed |
| `BB` | Blocked Bank | failed |
| `BC` | Transaction Cancelled By Customer | failed |
| `FE` | Internal Error | failed |
| `OE` | Transaction Rejected, Outside FPX Operating Hours | failed |
| `OF` | Transaction Timeout | failed |
| `SB` | Invalid Acquiring Bank Code | failed |
| `1A` | Buyer Session Timeout At Login Page | failed |
| `1B` | Buyer Failed To Login | failed |
| `1C` | Buyer Cancelled At Login Page | failed |
| `1D` | Buyer Session Timeout At Account Selection | failed |
| `1E` | Buyer Failed At Account Selection | failed |
| `1F` | Buyer Cancelled At Account Selection | failed |
| `1G` | Buyer Session Timeout At TAC Request | failed |
| `1H` | Buyer Failed At TAC Request | failed |
| `1I` | Buyer Cancelled At TAC Request | failed |
| `1J` | Buyer Session Timeout At Confirmation | failed |
| `1K` | Buyer Failed At Confirmation | failed |
| `1L` | Buyer Cancelled At Confirmation | failed |
| `1M` | Internet Banking Session Timeout | failed |
| `2A` | Transaction Amount Below Minimum Limit | failed |

Rule: `00` success; empty/`09`/`76`/`99` pending; any other code failed. The same `fpx_*` names are used for card (MPGS), Touch 'n Go and SPayLater payments.

## Payment Method Identifiers

Seen in official examples for `fpx.payment_method`: `fpxpay`, `mpgs` (credit card), `fpx.b2c`, `fpx.b2b1`. `pref_method` example: `fpx/b2b1`. `pref_bank` example: `ABB0235` (Affin Bank B2B). Buyer bank id examples: `BSN0601`, `MB2U0227`. A full list of method or bank codes is not published (checked 2026-10-21).

## Zakat Fitrah Agency Codes

| State | Zakat agency | Agency code |
|---|---|---|
| Johor | Majlis Agama Islam Negeri Johor (MAIJ) | `JHRFITRAH` |
| Kelantan | MAIK | `MAKFITRAH` |
| Negeri Sembilan | MAINS | `NSNFITRAH` |
| Pulau Pinang | MAINPP | `ZKPNG` |
| Perlis | MAIPs | `ZKPLS` |
| Sabah | Majlis Ugama Islam Sabah (MUIS) | `ZKSBH` |
| Sarawak | Tabung Baitulmal Sarawak (TBS) | `ZKBTL` |
| Selangor | Lembaga Zakat Selangor (LZS) | `ZKLZS` |
| Terengganu | MAIDAM | `ZKTRG` |
| Wilayah Persekutuan | PPZ-MAIWP | `ZKPPZ` |

Each code links to `https://developers.tulus.my/schema/<CODE>.json`, a form config with `dictionary.zakat_type`, `dictionary.negeri` (state ids `1` Johor to `16` Putrajaya), `data.zakat_rate` (fitrah rice-price options, for example RM 8 / 10 / 20), `data.ramadan.start` / `end`, `form.zakat.fields`, and a maintenance message. Rates are precalculated per state. On 2026-10-21 the JSON files for `JHRFITRAH`, `MAKFITRAH`, `NSNFITRAH`, `ZKLZS`, `ZKTRG` (and `LZSFITRAH`, named on the zakat page) returned 200; `ZKPNG`, `ZKPLS`, `ZKSBH`, `ZKBTL`, `ZKPPZ` returned 404. The PPZ form schema example uses agency `ZKPPZMAIWP`, not `ZKPPZ`. Confirm the exact agency code for your collection with Tulus.

## Zakat Form Schemas (PPZ-MAIWP, LZS)

`/schema/PPZMAIWP.schema.json` and `/schema/ZKLZS.schema.json`. Required: `email`, `zakat_name`, `zakat_ic_type`, `zakat_ic_no`, `zakat_phone_no`, `zakat_address`, `zakat_postcode`, `zakat_state`, `zakat_payer_type`, `zakat_type`, `zakat_haul`, `amount`, `agency`. Optional: `zakat_address_2`, `zakat_address_3`. Examples: `/schema/PPZMAIWP.example.json`, `/schema/ZKLZS.example.json` (`zakat_ic_type` `Awam`, `zakat_payer_type` `individu`, `zakat_type` `zakat_harta`, `amount` `"102.10"`). Upstream forms change roughly annually; re-fetch before building.

Report schema `/schema/report.schema.json` adds: `transaction_id`, `transaction_timestamp`, `zakat_rate`, `zakat_dependant`, `amountFitrah`, `amountHarta`, `amount`, `agency`.

## `zakat_type` values (ADB schema)

`fitrah`, `zakat_emas`, `zakat_fitrah`, `zakat_kwsp`, `zakat_padi`, `zakat_pendapatan`, `zakat_perak`, `zakat_perniagaan`, `zakat_pertanian`, `zakat_qada`, `zakat_rikaz`, `zakat_saham`, `zakat_simpanan`, `zakat_ternakan`.

## `zakat_state` values (ADB schema)

`johor`, `kedah`, `kelantan`, `melaka`, `negeri_sembilan`, `pahang`, `pulau_pinang`, `perak`, `perlis`, `sabah`, `sarawak`, `selangor`, `terengganu`, `kuala_lumpur`, `labuan`, `putrajaya`.
