# GoMobi Codes and Error Lists

Checked 2026-10-21 against `https://gomobi.io/documents`. Bank lists change; always load the live FPX bank list.

## Generic

| Code | Meaning |
|---|---|
| `0000` | Success |
| `0001` | Failure |
| `0002` | Returned by DuitNow QR and EZYCASH status; meaning Not documented in official docs (checked 2026-10-21) |

Standard HTTP status codes also apply (`error-codes`).

## FPX Bank Codes (from the docs' sample response)

B2C (bankType `01`):

| BankCode | Display name |
|---|---|
| `ABB0233` | Affin Bank |
| `ABMB0212` | Alliance Bank (Personal) |
| `AMBB0209` | AmBank |
| `BIMB0340` | Bank Islam |
| `BMMB0341` | Bank Muamalat |
| `BKRM0602` | Bank Rakyat |
| `BSN0601` | BSN |
| `BCBB0235` | CIMB Clicks |
| `HLB0224` | Hong Leong Bank |
| `HSBC0223` | HSBC Bank |
| `KFH0346` | KFH |
| `MB2U0227` | Maybank2U |
| `OCBC0229` | OCBC Bank |
| `PBB0233` | Public Bank |
| `RHB0218` | RHB Bank |
| `SCB0216` | Standard Chartered |
| `UOB0226` | UOB Bank |
| `MBB0228` | Maybank2E |
| `AGRO01` | Agrobank |
| `BOCM01` | Bank of China |
| `MBSB001` | MBSB Bank Berhad |

B2B (bankType `02`):

| BankCode | Display name |
|---|---|
| `ABB0235` | AFFINMAX |
| `ABMB0213` | Alliance Bank (Business) |
| `AMBB0208` | AmBank |
| `BIMB0340` | Bank Islam |
| `BKRM0602` | i-bizRAKYAT |
| `BMMB0342` | Bank Muamalat |
| `BCBB0235` | CIMB Bank |
| `DBB0199` | Deutsche Bank |
| `HLB0224` | Hong Leong Bank |
| `HSBC0223` | HSBC Bank |
| `KFH0346` | KFH |
| `MBB0228` | Maybank2E |
| `OCBC0229` | OCBC Bank |
| `PBB0233` | Public Bank |
| `PBB0234` | PB Enterprise |
| `RHB0218` | RHB Bank |
| `SCB0215` | Standard Chartered |
| `UOB0228` | UOB Regional |
| `BNP003` | BNP Paribas |
| `AGRO02` | Agrobank |

The sandbox test bank is "SBI Bank A" (`sandbox-environments`).

## FPX Debit Auth Codes (`fpx_debitAuthCode`)

Source: FAQ status code table and `payment-response`. `00` = approved; `09` and `99` = pending; all others = not paid.

| Code | Description |
|---|---|
| `00` | Approved |
| `03` | Invalid Merchant |
| `05` | Invalid Seller Or Acquiring Bank Code |
| `09` | Transaction Pending |
| `12` | Invalid Transaction |
| `13` | Invalid Amount |
| `14` | Invalid Buyer Account |
| `20` | Invalid Response |
| `30` | Format Error |
| `31` | Invalid Bank |
| `39` | No Credit Account |
| `45` | Duplicate Seller Order Number |
| `46` | Invalid Seller Exchange Or Seller |
| `47` | Invalid Currency |
| `48` | Maximum Transaction Limit Exceeded |
| `49` | Merchant Specific Limit Exceeded |
| `50` | Invalid Seller for Merchant Specific Limit |
| `51` | Insufficient Funds |
| `53` | No Buyer Account Number |
| `57` | Transaction Not Permitted |
| `58` | Transaction To Merchant Not Permitted |
| `70` | Invalid Serial Number |
| `76` | Transaction Not Found |
| `77` | Invalid Buyer Name Or Buyer ID |
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
| `99` | Pending Authorization (Applicable for B2B model) |
| `BB` | Blocked Bank |
| `BC` | Transaction Cancelled By Customer |
| `DA` | Invalid Application Type |
| `DB` | Invalid Email Format |
| `DC` | Invalid Maximum Frequency |
| `DD` | Invalid Frequency Mode |
| `DE` | Invalid Expiry Date |
| `DF` | Invalid e-Mandate Buyer Bank ID |
| `FE` | Internal Error |
| `OE` | Transaction Rejected As Not In FPX Operating Hours |
| `OF` | Transaction Timeout |
| `SB` | Invalid Acquiring Bank Code |
| `XA` | Invalid Source IP Address (Applicable for B2B2 model) |
| `XB` | Invalid Seller Exchange IP |
| `XC` | Seller Exchange Encryption Error |
| `XE` | Invalid Message |
| `XF` | Invalid Number Of Orders |
| `XI` | Invalid Seller Exchange |
| `XM` | Invalid FPX Transaction Model |
| `XN` | Transaction Rejected Due To Duplicate Seller Exchange Order Number |
| `XO` | Duplicate Exchange Order Number |
| `XS` | Seller Does Not Belong To Exchange |
| `XT` | Invalid Transaction Type |
| `XW` | Seller Exchange Date Difference Exceeded |
| `1A` | Buyer Session Timeout At Internet Banking Login Page |
| `1B` | Buyer Failed To Provide The Necessary Info To Login To Internet Banking Login Page |
| `1C` | Buyer Choose Cancel At Login Page |
| `1D` | Buyer Session Timeout At Account Selection Page |
| `1E` | Buyer Failed To Provide The Necessary Info At Account Selection Page |
| `1F` | Buyer Choose Cancel At Account Selection Page |
| `1G` | Buyer Session Timeout At TAC Request Page |
| `1H` | Buyer Failed To Provide The Necessary Info At TAC Request Page |
| `1I` | Buyer Choose Cancel At TAC Request Page |
| `1J` | Buyer Session Timeout At Confirmation Page |
| `1K` | Buyer Failed To Provide The Necessary Info At Confirmation Page |
| `1L` | Buyer Choose Cancel At Confirmation Page |
| `1M` | Internet Banking Session Timeout |
| `2A` | Transaction Amount Is Lower Than Minimum Limit |
| `2X` | Transaction Is Canceled By Merchant |

Additional internal errors on the payment response: `MC` Invalid Checksum, `MS` Invalid SubMerchant or Invalid SellerOrderNo.

## Card Response Codes

Source: FAQ "Card Status Codes" table. Success = `00` at the processor; the API layer returns `responseCode` `0000`.

| Code | Description |
|---|---|
| `00` | SUCCESS |
| `05` | Do Not Honour |
| `51` | Insufficient Fund |
| `57` | Transaction Not Permitted to Cardholder |
| `12` | INVALID_PAN - 8006, Invalid PAN, f004_PAN null or empty or length not between 13 and 19 |
| `93` | TPCC_FAILED_3D - 8041 Failed 3D Authentication |
| `14` | Invalid Card No |
| `94` | DUPL_TXN - 5025, Duplicate Transaction |
| `54` | Expired Card |
| `96` | System Malfunction/Error |
| `61` | Over Limit |
| `04` | Pick Up Card |
| `58` | Transaction Not Permitted to Terminal |
| `03` | Invalid Merchant ID |
| `91` | Issuer/Switch Error |
| `41` | Card Reported Lost |
| `13` | Invalid Amount |
| `63` | Security Violation |
| `P1` | Failed Authentication |
| `15` | Invalid Issuer |
| `30` | Message Format Error |
| `56` | Invalid Card |
| `92` | Error in merchant profile configuration |
| `07` | Pick Up Card |
| `75` | PIN Tries Exceeded |
| `99` | Declined |
| `19` | Re-enter Transaction |
| `43` | Stolen Card |
| `01` | Refer To Issuer |
| `52` | No Checking Account |
| `CC_3001` | Card Authentication Failed |
| `CC_05` | do not honor |
| `P15` | Transaction amount must be less than MYR 5.00. |
| `P10` | Sorry, Your Credit Card  Number or CVV or expiration date is not valid |
| `P02` | Your transaction has been denied due to merchant account issue (P02) |
| `CC_46` | Closed account |
| `CC_82` | Cannot approve at this time, try again later. |
| `CC_03` | Invalid merchant |
| `CC_83` | Authentication may improve the likelihood of an approval. Retry using authentication (such as EMV3DS). |
| `CC_51` | Insufficient funds / over credit limit |
| `CC_62` | restricted card |
| `CC_N7` | decline for CVV2 failure |
| `CC_E0000` | Internal resource not found or validation failed, trace ID: ... |
| `CC_63` | Security violation |
| `CC_78` | Blocked, first used Transaction from new cardholder, and card not properly unblocked |
| `CC_54` | Expired card |
| `CC_57` | Transaction not permitted to cardholder |
| `CC_41` | Pick up card (lost card) |
| `P03` | Your payment info format not correct. |
| `CC_59` | Suspected fraud |
| `CC_5C` | Transaction not supported, blocked by issuer |
| `CC_3011` | Suspected fraud |
| `CC_3000` | Transaction has been completed with not-successful status |
| `CC_61` | Activity amount limit exceeded |
| `CC_3008` | No Card record |
| `CC_3026` | Authentication attempted but not performed by the cardholder. |
| `CC_99` | Unknown Error |
| `CC_3019` | Exceeds ACS maximum challenges |
| `CC_3014` | Transaction timed out at the ACS |
| `CC_3012` | Transaction not permitted to cardholder |
| `06` | Error |
| `CC_9G` | Blocked by cardholder, contact cardholder |
| `CC_93` | Transaction cannot be completed; violation of law |
| `68` | Expired Payment Status |
| `CC_6000` | Transaction ended with ErrorMessage is raised from one of the 3D participants |
| `CC_04` | Pick up card |
| `CC_55` | Incorrect PIN |
| `CC_96` | System malfunction or certain field error conditions |
| `CC_12` | invalid transaction |
| `CC_77` | Invalid/nonexistent From Account specified |
| `CC_5014` | Transaction timed out at the ACS |
| `P59` | Country issued card is rejected by merchant |
| `CC_65` | Activity count limit exceeded |
| `CC_V0000` | Field body.paymentMethod.card.cardInfo.cvc invalid format |
| `CC_1000` | Transaction is in progress |
| `CC_14` | Invalid account number |
| `CC_58` | Transaction not permitted to acquirer |
| `CC_79` | Updated information was found in the Mastercard ABU database. Check for new information before reattempting. |

## Payout BIC Codes

Source: `payout/malaysia/api` BIC table.

| Bank | Bank code | BIC |
|---|---|---|
| Affin Bank Berhad | ABB | PHBMMYKL |
| Alliance Bank Malaysia Berhad | ABMB | MFBBMYKL |
| Al-Rajhi | ARM | RJHIMYKL |
| Ambank Malaysia Berhad | AMFB | ARBKMYKL |
| Bank Islam Malaysia Berhad | BIMB | BIMBMYKL |
| Bank Kerjasama Rakyat Malaysia Berhad | BKRB | BKRMMYKL |
| Bank Muamalat Malaysia Bhd | BMMB | BMMBMYKL |
| Bank of America (M) Berhad | BOA | BOFAMY2X |
| Bank of China (M) Berhad | BOCM | BKCHMYKL |
| Agrobank | BPM | AGOBMYKL |
| Bank Simpanan Nasional Berhad | BSNB | BSNAMYK1 |
| Bank of Tokyo-Mitsubishi UFJ (M) Berhad | BOTM | BOTKMYKX |
| BNP Paribas Malaysia Berhad | BNP | BNPAMYKL |
| CIMB Bank Berhad | CIMB | CIBBMYKL |
| Deutsche Bank (Malaysia) Berhad | DB | DEUTMYKL |
| Hong Leong Bank Berhad | HLB | HLBBMYKL |
| HSBC Bank Malaysia Berhad | HSBC | HBMBMYKL |
| Industrial and Commercial Bank of China (M) Berhad | ICB | ICBKMYKL |
| JP Morgan Chase Bank Berhad | JPMC | CHASMYKX |
| Kuwait Finance House | KFH | KFHOMYKL |
| Maybank Berhad | MBB | MBBEMYKL |
| Mizuho Bank (Malaysia) Berhad | MCBM | MHCBMYKA |
| OCBC Bank Berhad | OCBC | OCBCMYKL |
| Public Bank Berhad | PBB | PBBEMYKL |
| RHB Bank Berhad | RHB | RHBBMYKL |
| Standard Chartered Bank Malaysia Berhad | SCB | SCBLMYKX |
| Sumitomo Mitsui Banking Corporation (M) Berhad | SMBC | SMBCMYKL |
| United Overseas Bank Berhad (UOB) | UOB | UOVBMYKL |
| MBSB Bank Berhad | MBSB | AFBQMYKL |
| Finexus Cards Sdn. Bhd. | N/A | FNXSMYNB |
| Touch 'n Go eWallet | TNGD | TNGDMYNB |
| China Construction Bank (M) Berhad | PCBC | PCBCMYKL |
| BigPay Malaysia Sdn Bhd | - | BGPYMYNB |
| Axiata Digital Ecode Sdn Bhd - Boost | - | BOSTMYNB |
| GX Bank Berhad | - | GXSPMYKL |
| Merchantrade Asia Sdn Bhd | - | MASBMYNB |
| Shopee | - | ARPYMYNB |

## E-wallet Error Descriptions (Boost, GrabPay, ShopeePay, TNG)

`responseCode` `0001`, `responseMessage` `FAILURE`, with `responseDescription`:

| responseDescription | Meaning (inferred from the message text) |
|---|---|
| Invalid Merchant Details | wrong mobiApiKey or merchant |
| amount: must not be null or empty | missing amount |
| amount: Invalid format | amount not in expected format |
| invoiceId: must not be null or empty | missing invoiceId |
| No service founded | wrong `service` value |
| checksum: must not be null or empty | missing checkSum |
| Invalid checksum | checksum mismatch (not minified, extra/empty fields, wrong key/salt) |
| subMid: must not be null or empty | sub-merchant required |
| Access Denied: Sub-merchant can't do transaction | sub-merchant blocked |
| Access Denied: Payment not enabled | method not enabled |
| Access Denied: The merchant is not active. | merchant inactive |
| Access Denied: The sub-merchant is not active. | sub-merchant inactive |
| Access Denied: This merchant is enabled for trusted access only. | trusted-module merchant |
| Invalid key from trusted module | trusted key mismatch |
| InvalidPayment Type | method not valid for merchant |
| Invalid MDR contact merchant support | MDR not configured |

Success descriptions: Boost/Shopee/TNG "QRCode hit success", Grab "One Time Init Success".

## DuitNow QR Errors

| Code | Messages |
|---|---|
| `E001` | MobiApiKey cannot be null or empty; Invalid Merchant |
| `E002` | Duitnow QR not enabled (merchant, or for this subMerchant) |
| `E003` | Invalid mobiApiKey Signature; Invalid Request Signature |
| `E006` | motoApiKey cannot be null or empty; Payment Info cannot be null; Country can't be null or empty; Invalid country; Payment Method can't be null or empty; Invalid payment Method; Invoice ID already exist; Amount cannot be null or empty; Invalid amount format; Amount must be greater than 1 RM; Only two decimals allowed; Remark cannot be null or empty; InvoiceId cannot be null or empty; InvoiceId maximum length should be 30 |
| `E007` | Invalid Request Signature |
| `E011` | Generating token failed |
| `E016`, `E017` | Transaction not found |
| `E020` | Payment Details not found; Merchant Details not found |
| `E021` | Payment Failed |
| `E022` | Merchant Not Found |

## EZYCASH (Cash) Errors

| Code | Messages |
|---|---|
| `E001` | MobiApiKey cannot be null or empty; Invalid Merchant |
| `E002` | EZYCash not enabled for this subMerchant |
| `E003` | Invalid mobiApiKey Signature; Invalid Request Signature |
| `E004` | Invalid SubMerchant |
| `E006` | Payment Info cannot be null; Invoice ID already exist; InvoiceId cannot be null or empty; InvoiceId maximum length should be 30; Remark cannot be null or empty; Remark maximum length should be 20; Amount cannot be null or empty; Invalid amount format; Country can't be null or empty; Invalid country; Payment Method can't be null or empty; Invalid payment Method; RedirectURL can't be null or empty |
| `E007` | Invalid Request Signature |
| `E011` | Payment processing failed due to a technical issue |
| `E016`, `E017` | Transaction not found |
| `E020` | Merchant details not found |
| `E021` | Payment Details not found; RedirectURL must start with http:// or https://; Invalid RedirectURL format |
| `E022` | Payment Failed |

## Payout Validation Errors (payout IPN page)

| Message | When it occurs |
|---|---|
| Missing APIKEY | mobiApiKey not passed |
| Bank Account is missing | bankAccNo missing |
| Bank Name is missing | bankName missing |
| Business Registered No / IC Missing | businessRegNo missing |
| Customer Name is Missing | customerName missing |
| Sub-Merchant MID Missing | subMID not included |
| Bank Bic Code Missing | bicCode not provided |
| Remarks length shouldn't exceed 20 | remarks too long |
| Amount Field is Missing/Invalid | amount empty or wrong format |
| Invalid Payout Type | payoutType set incorrectly |
| Minimum settlement payout is 10.00 RM | amount below threshold |
| Settlement via payout is not enabled | merchant not approved for settlement payout |
| Merchant is not active | account inactive |
| No Available Balance Found | payout wallet zero or negative |
| In-Sufficient Fund | balance lower than payout amount |
| Mdr Rates Not Registered | MDR not configured |
| Sub-Merchant MID Invalid | wrong or unlinked sub-MID |
| PayoutId Already Exists | duplicate payoutid |
| APIKEY is Invalid | key mismatch or not registered |

Payout request errors: InvoiceId Exists Already, Sub Merchant not found, Invalid payment type, Transaction not found, Invalid mid, Invalid master mid, MobiKeyDetails not found, Merchant not found.
