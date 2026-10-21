# Betterpay Codes And Enums

Checked: 2026-10-21.

Sources:

- Docs `https://www.betterpay.me/docs/#standard_payment_gateway1` (Response Code and Description)
- Docs `#bplite_status_code`, `#requery1`, `#bank_list1`, `#batchrecurring8`, `#bplite_channels1`, `#direct_payment1`

## LITE transaction status (`txn_status`)

| Code | Meaning |
|---|---|
| `00` | Transaction Successful |
| `11` | Transaction Failed |
| `22` | Transaction Pending |
| `failed` | Customer cancelled payment |

LITE `pay_method`: `online_banking` (FPX), `card`, `ewallet`, `bnpl`.

## PRO transaction status (`txn_status`) and FPX response codes

Settle only on `00`. Treat `09` and `99` as pending and requery later. Everything else is a failure or cancel.

| Code | Description |
|---|---|
| `00` | Transaction Approved |
| `03` | Invalid Merchant |
| `05` | Invalid Seller or Acquiring Bank Code |
| `09` | Transaction Pending |
| `12` | Invalid Transaction |
| `13` | Invalid Amount |
| `14` | Invalid Buyer Account |
| `20` | Invalid Response |
| `31` | Invalid Bank |
| `39` | No Credit Account |
| `45` | Duplicate Seller Order Number |
| `46` | Invalid Seller Exchange or Seller |
| `47` | Invalid Currency |
| `48` | Maximum Transaction Limit Exceeded RM30,000.00 for B2C |
| `49` | Merchant Specific Limit Exceeded |
| `50` | Invalid Seller for Merchant Specific Limit |
| `51` | Insufficient Funds |
| `53` | No Buyer Account Number |
| `57` | Transaction Not Permitted |
| `58` | Transaction To Merchant Not Permitted |
| `70` | Invalid Serial Number |
| `76` | Transaction Not Found |
| `77` | Invalid Buyer Name or Buyer ID |
| `78` | Decryption Failed |
| `79` | Host Decline When Down |
| `80` | Buyer Cancel Transaction |
| `83` | Invalid Transaction Model |
| `84` | Invalid Transaction Type |
| `85` | Internal Error At Bank System |
| `87` | Debit Failed Exception Handling |
| `88` | Credit Failed Exception Handling |
| `89` | Transaction Not Received Exception Handling |
| `90` | Bank Internet Banking Unavailable |
| `92` | Invalid Buyer Bank |
| `96` | System Malfunction |
| `98` | MAC Error |
| `99` | Pending Authorization (B2B model) |
| `BC` | Transaction Cancelled By Customer |
| `DA` | Invalid Application Type |
| `DB` | Invalid Email Format |
| `DC` | Invalid Maximum Frequency |
| `DD` | Invalid Frequency Mode |
| `DE` | Invalid Expiry Date |
| `DF` | Invalid e-Mandate |
| `FE` | Internal Error |
| `OE` | Transaction Rejected As Not In FPX Operating Hours |
| `OF` | Transaction Timeout |
| `SB` | Invalid Acquiring Bank Code |
| `XA` | Invalid Source IP Address (B2B2 model) |
| `XB` | Invalid Seller Exchange IP |
| `XC` | Seller Exchange Encryption Error |
| `XE` | Invalid Message |
| `XF` | Invalid Number of Orders |
| `XI` | Invalid Seller Exchange |
| `XM` | Invalid FPX Transaction Model |
| `XN` | Transaction Rejected Due To Duplicate Seller Exchange Order Number |
| `XO` | Duplicate Exchange Order Number |
| `XS` | Seller Does Not Belong To Exchange |
| `XT` | Invalid Transaction Type |
| `XW` | Seller Exchange Date Difference Exceeded |
| `1A` | Buyer Session Timeout At Internet Banking Login Page |
| `1B` | Buyer Failed To Provide The Necessary Info To Login To Internet Banking Login Page |
| `1C` | Buyer Choose Cancel At Login Page (also "Void" in the Card Pre-authorization callback) |
| `1D` | Buyer Session Timeout At Account Selection Page |
| `1E` | Buyer Failed To Provide The Necessary Info To Login To Internet Banking Login Page |
| `1F` | Buyer Choose Cancel At Account Selection Page |
| `1G` | Buyer Session Timeout At TAC Request Page |
| `1H` | Buyer Failed To Provide Necessary Info At TAC Request Page |
| `1I` | Buyer Choose Cancel At TAC Request Page |
| `1J` | Buyer Session Timeout At Confirmation Page |
| `1K` | Buyer Failed To Provide Necessary Info At Confirmation Page |
| `1L` | Buyer Choose Cancel At Confirmation Page |
| `1M` | Internet Banking Session Timeout |
| `2A` | Transaction Amount Is Lower Than Minimum Limit RM1.00 for B2C |
| others | Unable To Trace An Error |

PRO `pay_method` values in callbacks: `fpx`, `paypal`, `mastercard`, `others` (the channel name is in `trx_txt`).

## FPX refund status codes (refund_list, Get Refund Status)

| Code | Description |
|---|---|
| `B0` | Order list format error |
| `B1` | Invalid seller ID |
| `B2` | Seller is not allowed to refund |
| `B3` | Seller is not allowed to do multiple refund |
| `B4` | Requested refund amount exceeds maximum allowable |
| `B5` | Original transaction ID is not found |
| `B6` | Original transaction ID status is still pending debit/credit |
| `B7` | Original transaction ID status was not successful |
| `B8` | Previous refund request still pending debit/credit |
| `B9` | Requested refund amount below minimum allowable |
| `C1` | Invalid refund transaction model |
| `C2` | Invalid refund buyer bank |
| `C3` | Invalid refund seller bank |
| `C4` | Refund request fail due to no valid order list |
| `C5` | Order list contains duplicate seller order number |
| `1S` | Refund Successfully Submitted |
| `00` | Transaction Approved |

## API response codes

| API | Codes |
|---|---|
| Get Payment Status (PRO) | `00` success; `97` transaction pending; `97` invalid verification/hash (the docs reuse `97`); `98` transaction failed; `99` invalid invoice / invalid data |
| Get Bank List | `98` no bank list type (send `b2c` or `b2b`); `99` verification fail / invalid data |
| Reports (transactions, settlement, disburse) | `97` invalid date_from format; `99` verification fail |
| Create Bill | `00` success; `99` fail to create a bill |
| Payment Link | `00` success; `99` unsuccessful |
| Refund APIs | `00` submitted; `99` account not valid for refund API / invalid data |
| OCBC collection | `99` batch ID already exists; `99` authentication error (hash) |
| Rate limit | HTTP 429, `response` `99`, `msg` "Rate limit exceeded ..." |
| Token charge / capture | `txn_status` `00` success, `12` failed, `09` pending (details only) |
| Card Pre-authorization callback | `00` authorized, `76` cancel, `1C` void |
| OCBC collection callback `data[].status` | `00` success, `99` fail |

For every API, `response` is the API call result, not the payment result. Check `txn_status` / `status_code` for the payment.

## OCBC collection status values

PENDING, SUCCESSFUL, INSUFFICIENT FUNDS, ACCOUNT CLOSED, INVALID BUYER ACCOUNT, WITHDRAWAL FREQ EXCEEDED, AUTHORIZATION REVOKED, WITHDRAWAL LIMIT EXCEEDED, NOT AUTHORIZED, INVALID BUYER BANK, TRANSACTION NOT PERMITTED, INVALID AMOUNT, INVALID SELLER ID, INVALID BUYER NAME/ID, INVALID RESPONSE, INVALID SERIAL NUMBER, NO REASON TO DECLINE, PENDING DEBIT RETRY, PENDING BUYER BK RESPONSE, BUYER BANK NOT RESPONDED, CANCELLED BY BANK, CANCELLED BY CUSTOMER.

## Payment method codes

Standard Payment `payment_method` (filter list): `b2c` (online banking MYR/SGD/THB/PHP), `b2b` (FPX business), `cc` (card), `ewallet` (e-wallet / DuitNow QR), `indo_bank` (Indonesia bank portal), `bnpl`, `ipp` (instalment), `otc` (over the counter), `dnqr` (DuitNow QR).

Direct Payment `payment_method`: `b2c`, `b2b`, `cc`, `ewallet`, `indo_bank`, `bnpl`, `ipp`.

Get Bank List `type`: `b2b`, `b2c`, `indo_bank`, `ewallet`. Optional `currency` (default MYR). Response rows: `bank_id`, `bank_name`, `image`, `status` (`Online`/`Offline`).

## Bank and channel codes (PRO Direct Payment `bank_code`)

Always fetch live codes with Get Bank List (PRO) or Get Payment Channels (LITE). The codes below are the docs examples.

FPX B2C banks (Get Bank List sample): `ABB0233` Affin Bank, `ABMB0212` Alliance Bank (Personal), `AMBB0209` AmBank, `BIMB0340` Bank Islam, `BKRM0602` Bank Rakyat, `BMMB0341` Bank Muamalat, `BSN0601` BSN, `BCBB0235` CIMB Clicks, `HLB0224` Hong Leong Bank, `HSBC0223` HSBC Bank, `KFH0346` KFH, `MB2U0227` Maybank2U, `MBB0228` Maybank2E, `OCBC0229` OCBC Bank, `PBB0233` Public Bank, `RHB0218` RHB Bank, `SCB0216` Standard Chartered, `UOB0226` UOB Bank. The sample PHP form also lists `CIT0219` Citibank.

E-wallet codes in the docs sample form: `BOOST` Boost, `TNG` Touch 'n Go, `GRAB` Grab, `SHPPAY` ShopeePay, `OVO` OVO, `LINKAJA` LinkAja.

Fixed codes:

| Code | Channel |
|---|---|
| `CC00001` | Credit/debit card |
| `BNPL00001` | PaySlowSlow / Riipay |
| `IOUPay-PW` | IOUPay |
| `Atome` | Atome BNPL |
| `IPP00001` | Default instalment plan |
| `RPP_DUITNOWQR` | DuitNow QR (with `payment_method=ewallet`) |

LITE Get Payment Channels examples: `FPX_ABB` (Affin Bank FPX, type `IB`), `CREDIT` (Visa/Mastercard local, type `CC`), `IOUPay-PW` (type `BNPL`). Channel `type`: `IB`, `CC`, `EW`, `BNPL`.

## Recurring enums

- `subscription_type`: `fixed`, `flexi`, `calculated`.
- `terms` / `mandate_frequency`: `Daily`, `Weekly`, `Monthly`, `Yearly`. `repayment_periods.terms`: `Week`, `Month`, `Year`.
- `id_type`: `1` new IC, `2` old IC, `3` passport, `4` business registration, `5` others.
- `subscription_status` (callback): `new`, `maintenance`, `terminated`.
- e-Mandate `payment_method`: `b2c`, `b2b`.

## Currencies

PRO: `MYR`, `SGD`, `IDR`, `THB`, `PHP`. LITE adds `USD`. Payment Link (PRO): `MYR`, `IDR`, `SGD`. Availability depends on the account plan.
