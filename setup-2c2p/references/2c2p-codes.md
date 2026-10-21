# 2C2P Codes, Malaysian Channels And Test Data

Enums for PGW v4.x and Payment Maintenance. Entry point: `2c2p-api.md`. Checked against the official docs on 2026-10-21.

## Payment Flow Codes (frontend return, Do Payment, Transaction Status)

| Code | Meaning | Next step |
| --- | --- | --- |
| `1000` | Load URL in iframe / WebView | Show the 3DS or bank page inside the frame. |
| `1001` | Full redirect | Redirect the browser to the returned URL. |
| `1002` | Deep link or web URL, then poll | Open the app / tab, then poll Transaction Status. |
| `1003` | Payment slip | Show the slip, mark the order pending. |
| `1004` | App scheme with callback | Open the wallet app; use fallback URL if not installed. |
| `1005` | QR generated | Show QR, poll Transaction Status until it changes. |
| `2000` | Transaction completed | Check your backend notification status or call Payment Inquiry. |

Source: https://developer.2c2p.com/docs/response-code-payment-flow

## Payment Response Codes (Payment Token, Inquiry, notification)

State codes:

| Code | Meaning |
| --- | --- |
| `0000` | Successful |
| `0001` | Transaction is pending |
| `0003` | Transaction is cancelled |
| `0004` | Soft-declined; resubmit after 3DS authentication |
| `0999` | System error |
| `2001` | Transaction in progress |
| `2002` | Transaction not found |
| `2003` | Payment / Inquiry failed |

Card and issuer codes (`4xxx`):
`4000` card verification successful; `4001` refer to issuer; `4002` refer to issuer special conditions; `4003` invalid merchant ID; `4004` pick up card; `4005` do not honor; `4006` error; `4007` pick up card, special condition; `4008` honor with ID; `4009` request in progress; `4010` partial amount approved; `4011` approved VIP; `4012` invalid transaction; `4013` invalid amount; `4014` invalid card number; `4015` no such issuer; `4016` approved, update track 3; `4018` customer dispute; `4019` re-enter transaction; `4020` invalid response; `4021` no action taken; `4022` suspected malfunction; `4023` unacceptable transaction fee; `4024` file update not supported; `4025` unable to locate record; `4026` duplicate file update record; `4027` file update field edit error; `4028` file update file locked out; `4029` file update not successful; `4030` format error; `4031` bank not supported by switch; `4032` completed partially; `4033` expired card, pick up; `4034` suspected fraud, pick up; `4035` restricted card, pick up; `4036` allowable PIN tries exceeded; `4037` no credit account; `4038` allowable PIN tries exceeded; `4039` no credit account; `4040` requested function not supported; `4041` lost card, pick up; `4042` no universal amount; `4043` stolen card, pick up; `4044` no investment account; `4045` settlement success; `4046` settlement fail; `4047` cancel success; `4048` cancel fail; `4049` no transaction reference number; `4050` host down; `4051` insufficient funds; `4052` no cheque account; `4053` no savings account; `4054` expired card; `4055` incorrect PIN; `4056` no card record; `4057` not permitted to cardholder; `4058` not permitted to terminal; `4059` suspected fraud; `4060` card acceptor contact acquirer; `4061` exceeds withdrawal amount limit; `4062` restricted card; `4063` security violation; `4064` original amount incorrect; `4065` exceeds withdrawal frequency; `4066` card acceptor call acquirer security; `4067` hard capture; `4068` response received too late; `4070` settle amount cannot exceed authorized amount; `4071` inquiry record not exist; `4072` promotion not allowed for this method; `4073` promotion limit reached; `4075` allowable PIN tries exceeded; `4076` invalid credit card format; `4077` invalid expiry date format; `4078` invalid three digits format; `4080` user cancelled by closing browser; `4081` unable to authenticate cardholder; `4086` ATM malfunction; `4087` no envelope inserted; `4088` unable to dispense; `4089` administration error; `4090` cut-off in progress; `4091` issuer or switch inoperative; `4092` financial institution not found; `4093` transaction cannot be completed; `4094` duplicate transmission; `4095` reconcile error; `4096` system malfunction; `4097` reconciliation totals reset; `4098` MAC error; `4099` unable to complete payment; `4110` settled; `4120` refunded; `4121` refund rejected; `4122` refund failed; `4130` chargeback; `4131` chargeback rejected; `4132` chargeback failed; `4140` transaction does not exist; `4200` tokenization successful; `4201` tokenization failed; `4202` invalid card / customer token; `4203` no response from account issuer; `4204` tokenization action failed (token); `4205` tokenization action failed (limit exceeded); `4208` tokenization action failed (client); `4209` tokenization action failed (issuer). `4069`, `4074`, `4079`, `4082` to `4085` are reserved.

APM and gateway codes (`5xxx`, `6xxx`, `7xxx`):
`5002` timeout; `5003` invalid message; `5004` invalid profile (merchant) ID; `5005` duplicated invoice; `5006` invalid amount; `5007` insufficient balance; `5008` invalid currency code; `5009` payment expired; `5011` invalid payee ID; `5012` invalid customer ID; `5013` account does not exist; `5014` authentication failed; `5015` customer paid more than amount; `5016` customer paid less than amount; `5017` paid expired; `5018` reserved; `5019` no action from WebPay; `5998` internal error; `6012` invalid transaction; `6101` invalid request message; `6102` payload required; `6103` invalid JWT data; `6104` merchantId required; `6105` paymentChannel required; `6106` authCode required; `6107` invalid merchantId; `6108` invalid paymentChannel; `6109` paymentChannel not configured; `6110` unable to retrieve usertoken; `7012` invalid transaction.

Validation codes (`9xxx`):
`9004` parameter value not valid; `9005` mandatory fields missing; `9006` field exceeds length; `9007` invalid merchant; `9008` invalid payment expiry; `9009` amount invalid; `9010` invalid currency code; `9012` paymentItem name required; `9013` paymentItem quantity required; `9014` paymentItem amount required; `9015` existing invoice number; `9016` failed to retrieve PaymentInstruction; `9017` PaymentInstruction not available; `9020` payment expired (V4 API); `9035` payment failed; `9037` merchant configuration missing; `9038` failed to generate token; `9039` merchant frontend URL missing; `9040` token invalid; `9041` payment token already used; `9042` hash value mismatch; `9043` SCA exemption indicator invalid; `9044` previous payment ID required; `9057` payment options invalid; `9058` payment channel invalid; `9059` payment channel unauthorized; `9060` payment channel unconfigured; `9078` promotion code does not exist; `9080` tokenization not allowed; `9088` SubMerchant required; `9089` duplicated SubMerchant; `9090` SubMerchant not found; `9091` invalid sub merchant ID; `9092` invalid sub merchant invoiceNo; `9093` existing sub merchant invoice number; `9094` invalid sub merchant amount; `9095` sub merchant amount mismatch; `9100` FxRateId and OriginalAmount required; `9101` FX payment not allowed; `9102` FxRate not available; `9103` invalid amount for FxRateId; `9104` invalid country code (airline); `9105` invalid currency code (airline); `9106` invalid loyalty redeem amount; `9107` invalid loyalty provider; `9108` duplicated loyalty reward ID; `9109` loyalty external merchant ID required; `9110` failed to inquire loyalty rewards; `9202` invalid CustomerToken; `9900` unable to decrypt payload; `9901` invalid invoicePrefix; `9902` allowAccumulate required; `9903` maxAccumulateAmount required; `9904` recurringInterval or ChargeOnDate required; `9905` recurringCount required; `9906` recurringInterval or ChargeOnDate required; `9907` invalid ChargeNextDate; `9908` invalid ChargeOnDate; `9909` chargeNextDate required; `9990` request to merchant front end failed; `9991` merchant secure request failed; `9992` payment secure request failed; `9993` unknown error; `9994` DB service request failed; `9995` payment service request failed; `9996` Qwik service request failed; `9997` user preferences request failed; `9998` store card request failed; `9999` request to merchant backend failed.

Source: https://developer.2c2p.com/docs/response-code-payment

## Payment Maintenance Result Codes (`respCode`)

`00` success; `01` stored card ID not found; `02` invalid request; `03` invalid merchant ID; `04` invalid stored card unique ID; `05` invalid customer email; `10` missing compulsory values; `11` request validation failed; `12` transaction status not valid for this action; `13` invalid hash value; `14` invalid merchant ID; `15` invalid invoice number; `16` transaction does not exist; `17` request type invalid; `18` invalid action amount; `21` void not allowed; `22` sub transaction cannot be voided individually; `25` void failed; `29` already voided; `30` unable to refund / settle more than transaction amount; `31` settlement not allowed; `32` settlement not required; `33` partial settlement not allowed; `34` settlement rejected (duplicate capture detected; inquire before a new capture); `35` settlement failed; `36` settlement not allowed beyond deadline; `37` sub merchant settlement amount too high; `38` sub transaction cannot be captured individually; `39` already settled; `40` refund amount more than transaction amount; `41` refund not allowed; `42` refund pending; `43` partial refund not allowed; `44` refund rejected; `45` refund failed; `46` insufficient funds to refund; `47` sub merchant refund amount too high; `48` sub merchant insufficient funds; `49` already fully refunded; `50` same idempotent key exists with different type; `51` same idempotent key exists with status pending; `52` cannot retrieve main transaction, refund failed; `53` no transaction reference number; `54` refund exceeded allowable timeframe; `61` cancel not allowed; `65` cancel failed; `95` request timed out; `96` unable to decrypt; `97` process not supported; `98` request not available; `99` unable to complete the request.

Source: https://developer.2c2p.com/docs/response-code-payment-maintenance-result-code

## Payment Maintenance Status Codes (`status`)

| Code | Meaning | Code | Meaning |
| --- | --- | --- | --- |
| `A` | Approved | `RR` | Refund rejected |
| `AP` | Approval pending | `RR1` | Refund rejected, insufficient balance |
| `AE` | Approved after expired (APM) | `RR2` | Refund rejected, invalid bank information |
| `AL` | Approved with less amount (APM) | `RR3` | Refund rejected, bank account mismatch |
| `AM` | Approved with more amount (APM) | `RS` | Ready for settlement |
| `PF` | Payment failed | `S` | Settled |
| `AR` | Authentication rejected (MPI) | `T` | Credit adjustment |
| `FF` | Fraud rule rejected | `V` | Voided / cancelled |
| `IP` | Rejected, invalid promotion | `VP` | Void pending |
| `ROE` | Rejected, routing | `EX` | Payment expired |
| `RP` | Refund pending | `CTS` | Tokenization success |
| `RF` | Refund confirmed | `CTF` | Tokenization failed |
| `RFF` | Refund failed | `PPC` / `PFC` | Partially / fully captured |

Source: https://developer.2c2p.com/docs/response-code-payment-maintenance-status-code

## Malaysian Payment Channels

Columns: category / group / channel code. `paymentChannel` on Payment Token takes the group code; Do Payment takes the channel code (and agent codes for 123). Exact Do Payment shapes are in `2c2p-direct-api.md`.

| Method | Category | Group | Channel | Agent / notes |
| --- | --- | --- | --- | --- |
| Cards incl. MyDebit | `GCARD` | `CC` | `CC` | Visa, Mastercard, Amex, JCB, UnionPay, MyDebit |
| Card instalment | `GCARD` | `IPP` | `IPP` | HSBC, Maybank, Public Bank, CIMB, Hong Leong, RHB, OCBC, Alliance, Amex, AmBank, Affin, Standard Chartered, UOB |
| Apple Pay / Google Pay | `DPAY` | `CWALLET` | `APPLEPAY` / `GOOGLEPAY` | Scheme tokens may route as `CSTOKEN` |
| DuitNow QR | `QR` | `QRC` | `DUITNOW` | MY |
| Customer-presented QR | `QR` | `QRC` | `SHPQR`, `GPQR`, `GIPQR` | ShopeePay, GrabPay, GIP |
| Touch 'n Go eWallet | `DPAY` | `EWALLET` | `TNG` | Tokenisable |
| Boost | `DPAY` | `EWALLET` | `BOOST` | |
| GrabPay | `DPAY` | `EWALLET` | `GRAB` | Tokenisable in SG and MY |
| ShopeePay | `DPAY` | `EWALLET` | `SHPPAY` | Tokenisable |
| ShopBack Pay | `DPAY` | `EWALLET` | `SHOPBACK` | |
| WeChat Pay | `DPAY` | `EWALLET` | `WECHAT`, `WCMINI`, `WCAPP` | |
| Alipay | `DPAY` | `EWALLET` | `ALIPAY` | |
| Atome, Grab PayLater, SPayLater | `GBNPL` | `BNPL` | `ATOME`, `GRABBNPL`, `SPAYLATER` | |
| FPX online banking | `WEBPAY` | `WEBPAY` | `123` | agent channel `WEBPAY`, agent codes below |
| Over the counter (EPAY) | `COUNTER` | `OTCTR` | `123` | agent `MYEPAY`, agent channel `OVERTHECOUNTER` |

FPX agent codes (channel `123`, agent channel `WEBPAY`): `MYABB` Affin, `MYAGB` Agro, `MYABMB` Alliance (personal), `MYAMB` AmBank, `MYBISM` Bank Islam, `MYBKRM` Bank Rakyat, `MYBMMB` Bank Muamalat, `MYBSN` BSN, `MYBOC` Bank of China, `MYCIMBCLICKS` CIMB Clicks, `MYCITI` Citibank, `MYDBB` Deutsche Bank, `MYHLB` Hong Leong, `MYHSBC` HSBC, `MYKFH` KFH, `MYM2U` Maybank2U, `MYOCBC` OCBC, `MYPBB` Public Bank, `MYPBBE` PB Enterprise, `MYRHB` RHB, `MYSCB` Standard Chartered, `MYUOB` UOB, `MYUOBE` UOB Regional, `MYM2E` Maybank2E, `M2UPAY` Maybank2U. Sandbox test banks: `MYSBIA`, `MYSBIB`, `MYSBIC` (UAT / sandbox only). The agent code list page names Maybank2U `MYMBB` while the channel page and migration table use `MYM2U`; use what Payment Option Details returns for your account.

FPX code migration (legacy to new, required for Do Payment, SDK and legacy APIs): `FPX_ABB` to `MYABB`, `FPX_ABMB` to `MYABMB`, `FPX_AGB` to `MYAGB`, `FPX_AMB` to `MYAMB`, `FPX_BIMB` to `MYBISM`, `FPX_BKRM` to `MYBKRM`, `FPX_BMMB` to `MYBMMB`, `FPX_BOC` to `MYBOC`, `FPX_BSN` to `MYBSN`, `FPX_CIMB` to `MYCIMBCLICKS`, `FPX_DBB` to `MYDBB`, `FPX_HLB` to `MYHLB`, `FPX_HSBC` to `MYHSBC`, `FPX_KFH` to `MYKFH`, `FPX_M2U` to `MYM2U`, `FPX_M2E` to `MYM2E`, `FPX_OCBC` to `MYOCBC`, `FPX_PBB` to `MYPBB`, `FPX_PBBE` to `MYPBBE`, `FPX_RHB` to `MYRHB`, `FPX_SCB` to `MYSCB`, `FPX_UOB` to `MYUOB`, `FPX_UOBE` to `MYUOBE`.

Source: https://developer.2c2p.com/docs/reference-payment-channels, https://developer.2c2p.com/docs/reference-agent-code-update-for-fpx-payment

## Malaysian Agent Codes (`agentCode` / `paidAgent`)

`123MY` 123 Malaysia, `ALB` Alliance Bank, `ALBMPGS` Alliance Bank MPGS, `AMBMY` AmBank Malaysia, `AMEXM` Amex Malaysia, `BOOST` Boost, `CIMBMY` CIMB, `MYEPAY` EPAY, `GrabMY` Grab MY, `HLB` Hong Leong, `MBBM` Maybank Malaysia, `M2UPAY` Maybank2U, `PBB` Public Bank, `TNG` Touch 'n Go, plus the FPX `MY...` bank codes above.

Source: https://developer.2c2p.com/docs/reference-codes-agent-code

## Payment Scheme Codes (`paymentScheme`, `processBy`), Malaysian Relevant

`VI` Visa, `MA` Mastercard, `AM` American Express, `JC` JCB, `UP` UnionPay, `DN` Diners, `DU` DuitNow (QR), `TG` Touch 'n Go, `BO` Boost, `GP` GrabPay, `GPI` GrabPay instalment 4 months, `G08` / `G12` GrabPay instalment 8 / 12 months, `GPP` GrabPay postpaid, `SH` ShopeePay, `S01` to `S24` Shopee PayLater by tenor, `SB` ShopBack, `AO` Atome, `A01` / `A03` / `A06` / `A12` Atome PayLater by months, `AL` Alipay / Alipay+, `WC` WeChat. The full list (over 200 codes, mostly non-Malaysian wallets) is on the source page. Note `MY` is MPay Vietnam, not Malaysia.

Source: https://developer.2c2p.com/docs/reference-codes-payment-scheme

## Void Cut-Off Times (Asia/Kuala_Lumpur)

| Bank code | Name | Cut-off |
| --- | --- | --- |
| `ALB` | Alliance Bank | 2200 |
| `AMBMY` | AmBank Malaysia | 2115 |
| `AMEXM` | Amex Malaysia | 0000 |
| `BOOST` | Boost | 2359 |
| `CIMBMY` | CIMB | 2200 |
| `GrabMY` | Grab MY | 2359 |
| `HLB` | Hong Leong | 2300 |
| `MBBM` | Maybank Malaysia | 2359 |
| `OCBC` | OCBC | 2030 |
| `PBB` | Public Bank | 1800 |
| `TNG` | Touch 'n Go | 2359 |
| `123MY` | 123 Malaysia | 0000 |

Source: https://developer.2c2p.com/docs/reference-payment-channels-cut-off-time

## Currency

MYR: alpha `MYR`, numeric `458`, 2 decimals. PGW JSON APIs use the alpha code; maintenance, recurring and card token XML use the numeric code.

Source: https://developer.2c2p.com/docs/reference-codes-currency, https://developer.2c2p.com/docs/sandbox

## Test Cards And Accounts (sandbox only)

Malaysia page:

| Card | Number | Details |
| --- | --- | --- |
| Visa | `4111111111111111` | Any future expiry (for example 12/30), CVV 123, OTP 123456 |
| Mastercard | `5555555555554444`, `5395970000000006` | CVV 123, OTP 123456 |
| UnionPay | `6250947000000014` | Expiry 12/33, CVN2 123, phone 11112222, verification code 111111 |

Malaysian IPP test cards: PBB Visa `4485844044748462`, MC `5375742626694494`; MBB Visa `4842810230406585`, MC `5426340900064101`, Amex `379186031000003`; RHB Visa `4532883278094055`, MC `5428512442627946`; CIMB Visa `4929471020049931`, MC `5480425997008633`; OCBC Visa `4485191611156806`, MC `5152744093793782`; HLB Visa `4485977682582270`, MC `5547760456744232`; HSBC Visa `4916880688316587`, MC `5445050787267761`; Affin Visa `4539176588327544`, MC `5551311052552562`; SCB Visa `4716255265631868`, MC `5377056725975870`; UOB Visa `4840844271563543`, MC `5148151207906101`.

1-2-3 (online banking) test account: User ID `1234`, Password `1234`. FPX test banks: `MYSBIA`, `MYSBIB`, `MYSBIC`.

Generic pages: rejected Visa `4444333322221111`; Amex `374340095639074` (testing page) and `378282246310005` CVV 1234 (sandbox setup page); JCB `3562808775869340` CVV 495, OTP 123456.

Source: https://developer.2c2p.com/docs/reference-test-information-my, https://developer.2c2p.com/docs/reference-testing-information, https://developer.2c2p.com/docs/sandbox-setup
