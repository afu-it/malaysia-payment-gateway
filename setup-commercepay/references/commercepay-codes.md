# CommercePay Codes Reference

Channel IDs, FPX/DuitNow bank codes (`providerChannelId`), payout bank and DuitNow Proxy codes, and staging payout test accounts. Copied from the official docs, checked 2026-10-21.

Sources: `https://docs.commercepay.asia/channel-list`, `https://docs.commercepay.asia/payout-provider-channel`, `https://docs.commercepay.asia/payout-test-account`.

Channel availability changes. Confirm with a live `GET /api/services/app/Channel/GetTenantChannelsByCountry?countryCode=MY` and `GetProviderChannels` before go-live. Channel IDs differ between staging and production; keep them in per-environment config.

When `isProviderHostChannel` is yes, send the bank code as `providerChannelId`; otherwise omit `providerChannelId`.

## Staging Channel IDs

| Channel Name | channelId | isProviderHostChannel |
| --- | --- | --- |
| FPX B2C (Provider A) | 2 | yes |
| GrabPay | 7 | no |
| Credit Card | 9 | no |
| TNG eWallet | 10 | no |
| ShopeePay | 11 | no |
| Atome | 14 | no |
| Payout | 15 | no |
| Duitnow Online Banking B2C | 28 | yes |
| Debit/Credit Cards | 45 | no |
| Apple Pay | 58 | no |
| Google Pay | 61 | no |
| FPX B2C (Provider B) | 64 | yes |
| FPX B2B (Provider B) | 65 | yes |
| DuitNow QR (Provider B) | 66 | no |
| KK Mart (Cash over Counter) | 67 | no |

### FPX B2C (Provider A) (channelId 2) - Available Banks (Staging)

| Bank Name | providerChannelId |
| --- | --- |
| Affin B2C - Test ID | `ABB0234` |
| Affin Bank | `ABB0233` |
| AGRONet | `AGRO01` |
| Alliance Bank (Personal) | `ABMB0212` |
| AmBank | `AMBB0209` |
| Bank Islam | `BIMB0340` |
| Bank Muamalat | `BMMB0341` |
| Bank Of China (M) Berhad | `BOCM01` |
| Bank Rakyat | `BKRM0602` |
| BSN | `BSN0601` |
| CIMB Clicks | `BCBB0235` |
| Citibank | `CIT0219` |
| Hong Leong Bank | `HLB0224` |
| HSBC Bank | `HSBC0223` |
| KFH | `KFH0346` |
| Maybank2E | `MBB0228` |
| Maybank2U | `MB2U0227` |
| OCBC Bank | `OCBC0229` |
| Public Bank | `PBB0233` |
| RHB Bank | `RHB0218` |
| SBI Bank A | `TEST0021` |
| SBI Bank B | `TEST0022` |
| SBI Bank C | `TEST0023` |
| Standard Chartered | `SCB0216` |
| UOB Bank | `UOB0226` |

### FPX B2C (Provider B) (channelId 64) - Available Banks (Staging)

| Bank Name | providerChannelId |
| --- | --- |
| Affin Bank | `ABB0233` |
| Affin B2C - Test ID | `ABB0234` |
| Alliance Bank (Personal) | `ABMB0212` |
| AGRONet | `AGRO01` |
| AmBank (Offline) | `AMBB0209` |
| CIMB Clicks | `BCBB0235` |
| Bank Islam | `BIMB0340` |
| Bank Rakyat | `BKRM0602` |
| Bank Muamalat | `BMMB0341` |
| Bank Of China | `BOCM01` |
| BSN | `BSN0601` |
| Citibank (Offline) | `CIT0219` |
| GXBANK01 Bank (Offline) | `GXBANK01` |
| Hong Leong Bank | `HLB0224` |
| HSBC Bank | `HSBC0223` |
| KAFB01 Bank | `KAFB01` |
| KFH (Offline) | `KFH0346` |
| Load Bank (Offline) | `LOAD001` |
| Maybank2U (Offline) | `MB2U0227` |
| Maybank2E (Offline) | `MBB0228` |
| M2U Test | `MBBM2U2` |
| MBSB Bank | `MBSB001` |
| OCBC Bank | `OCBC0229` |
| Public Bank | `PBB0233` |
| RHB Bank | `RHB0218` |
| Standard Chartered (Offline) | `SCB0216` |
| SBI Bank A | `TEST0021` |
| SBI Bank B | `TEST0022` |
| SBI Bank C | `TEST0023` |
| UOB Bank | `UOB0226` |

### FPX B2B (Provider B) (channelId 65) - Available Banks (Staging)

| Bank Name | providerChannelId |
| --- | --- |
| AFFINMAX | `ABB0235` |
| Alliance Bank (Business) | `ABMB0213` |
| AGRONetBIZ | `AGRO02` |
| AmBank | `AMBB0208` |
| CIMB Bank | `BCBB0235` |
| Bank Islam | `BIMB0340` |
| i-bizRAKYAT | `BKRM0602` |
| Bank Muamalat | `BMMB0342` |
| BNP Paribas | `BNP003` |
| Citibank Corporate Banking | `CIT0218` |
| Deutsche Bank | `DBB0199` |
| Hong Leong Bank | `HLB0224` |
| HSBC Bank | `HSBC0223` |
| KFH (Offline) | `KFH0346` |
| Load Bank (Offline) | `LOAD001` |
| Maybank2E (Offline) | `MBB0228` |
| MBSB Bank | `MBSB001` |
| OCBC Bank | `OCBC0229` |
| Public Bank PBe | `PBB0233` |
| Public Bank PB enterprise | `PBB0234` |
| RHB Bank | `RHB0218` |
| Standard Chartered (Offline) | `SCB0215` |
| SBI Bank A | `TEST0021` |
| SBI Bank B | `TEST0022` |
| SBI Bank C | `TEST0023` |
| UOB Regional | `UOB0228` |
| UOB Bank - Test ID | `UOB0229` |

### Duitnow Online Banking B2C (channelId 28) - Available Banks (Staging)

| Bank Name | providerChannelId |
| --- | --- |
| Aeon Digital Bank (RPP) | `ACDBMYK2` |
| PYN Bank A (RPP) | `ACFBMYK1` |
| MBSB Bank (RPP) | `AFBQMYKL` |
| AgroBank (RPP) | `AGOBMYKL` |
| Ambank (RPP) | `ARBKMYKL` |
| CIMB Bank (FPX) | `BCBB0235` |
| Bank Islam Malaysia Berhad (FPX) | `BIMB0340` |
| Bank Islam Malaysia Berhad (RPP) | `BIMBMYKL` |
| Bank Of China (RPP) | `BKCHMYKL` |
| Bank Kerjasama Rakyat (RPP) | `BKRMMYKL` |
| Bank Muamalat Malaysia Berhad (RPP) | `BMMBMYKL` |
| Boost Digital Bank (RPP) | `BOBEMYK2` |
| Bank Simpanan Nasional (RPP) | `BSNAMYK1` |
| CIMB Bank (RPP) | `CIBBMYKL` |
| Paynet Bank Simulator 2 (RPP) | `DENGMYK1` |
| PayNet IPOJ Bank (RPP) | `DEVPMYK1` |
| DASH Bank (RPP) | `DEVPMYK2` |
| Dummy Bank 1 (RPP) | `DMM1MYKL` |
| Dummy Bank 4 (RPP) | `DMM4MYKL` |
| Finexus (RPP) | `FNXSMYNB` |
| Fasspay (RPP) | `FSPYMYNB` |
| HSBC (RPP) | `HBMBMYKL` |
| Hong Leong Bank Berhad (RPP) | `HLBBMYKL` |
| PYN Bank B (RPP) | `HLFBMYK1` |
| HSBC (FPX) | `HSBC0223` |
| Industrial and Comercial Bank Of China (RPP) | `ICBKMYKL` |
| iPay88 (FPX) | `IPAYMYNB` |
| MAYBANK (FPX) | `MB2U0227` |
| MAYBANK (RPP) | `MBBEMYKL` |
| MobilityOne (RPP) | `MBLOMYNB` |
| Alliance Bank (RPP) | `MFBBMYKL` |
| Picasso Next Switch API Simulator (RPP) | `NXTAMYK1` |
| Picasso Next Switch API Simulator 2 (RPP) | `NXTAMYK2` |
| Picasso Next Switch TCPIP Simulator (RPP) | `NXTTMYK1` |
| Picasso Next Switch TCPIP Simulator 2 (RPP) | `NXTTMYK2` |
| OCBC Berhad (FPX) | `OCBC0229` |
| OCBC Berhad (RPP) | `OCBCMYKL` |
| Public Bank (RPP) | `PBBEMYKL` |
| CCBM (RPP) | `PCBCMYKL` |
| AFFIN BANK BERHAD (RPP) | `PHBMMYKL` |
| PayNet Simulator Bank (RPP) | `PICAMYK1` |
| PayNet Simulator Bank 2 (RPP) | `PICAMYK2` |
| RHB Bank (RPP) | `RHBBMYKL` |
| Al Rajhi (RPP) | `RJHIMYKL` |
| Standard Chartered Bank (RPP) | `SCBLMYKX` |
| Sea Capital Services Berhad (RPP) | `SCCHMYKL` |
| Setel (RPP) | `SVSBMYNB` |
| SBI BANK A (FPX) | `TEST0021` |
| SBI BANK B (FPX) | `TEST0022` |
| SBI BANK B (FPX) | `TEST0023` |
| Paynet Bank Simulator 1 (RPP) | `TESTMYK1` |
| United Overseas Bank (RPP) | `UOVBMYKL` |
| VAC Bank 1 (RPP) | `VACMYKL1` |
| VAC Bank 2 (RPP) | `VACMYKL2` |
| VAC Bank 3 (RPP) | `VACMYKL3` |

## Production Channel IDs

| Channel Name | channelId | isProviderHostChannel |
| --- | --- | --- |
| FPX B2C (Provider A) | 2 | yes |
| GrabPay | 4 | no |
| ShopeePay | 6 | no |
| TNG e-wallet | 7 | no |
| FPX B2B (Provider A) | 8 | yes |
| Payout | 9 | no |
| Atome | 10 | no |
| Duitnow Online Banking B2C | 18 | yes |
| Duitnow Online Banking B2B | 19 | yes |
| Boost | 20 | no |
| DuitNow QR (Provider A) | 21 | no |
| Alipay | 22 | no |
| Debit/Credit Cards | 23 | no |
| Debit/Credit Cards Recurring | 26 | no |
| Apple Pay | 30 | no |
| Google Pay | 32 | no |
| FPX B2C (Provider B) | 35 | yes |
| FPX B2B (Provider B) | 36 | yes |
| DuitNow QR (Provider B) | 37 | no |

### FPX B2C (Provider A) (channelId 2) - Available Banks (Production)

| Bank Name | providerChannelId |
| --- | --- |
| Affin Bank | `ABB0233` |
| AGRONet | `AGRO01` |
| Alliance Bank (Personal) | `ABMB0212` |
| AmBank | `AMBB0209` |
| Bank Islam | `BIMB0340` |
| Bank Muamalat | `BMMB0341` |
| Bank Of China (M) Berhad | `BOCM01` |
| Bank Rakyat | `BKRM0602` |
| BSN | `BSN0601` |
| CIMB Clicks | `BCBB0235` |
| Hong Leong Bank | `HLB0224` |
| HSBC Bank | `HSBC0223` |
| KFH | `KFH0346` |
| Maybank2E | `MBB0228` |
| Maybank2U | `MB2U0227` |
| OCBC Bank | `OCBC0229` |
| Public Bank | `PBB0233` |
| RHB Bank | `RHB0218` |
| Standard Chartered | `SCB0216` |
| UOB Bank | `UOB0226` |

### FPX B2B (Provider A) (channelId 8) - Available Banks (Production)

| Bank Name | providerChannelId |
| --- | --- |
| AFFINMAX | `ABB0235` |
| AGRONetBIZ | `AGRO02` |
| Alliance Bank (Business) | `ABMB0213` |
| AmBank | `AMBB0208` |
| Bank Islam | `BIMB0340` |
| Bank Muamalat | `BMMB0342` |
| Bank Rakyat | `BKRM0602` |
| BNP Paribas | `BNP003` |
| CIMB Clicks | `BCBB0235` |
| Citibank Corporate Banking | `CIT0218` |
| Deutsche Bank | `DBB0199` |
| Hong Leong Bank | `HLB0224` |
| HSBC Bank | `HSBC0223` |
| KFH | `KFH0346` |
| Maybank2E | `MBB0228` |
| OCBC Bank | `OCBC0229` |
| Public Bank | `PBB0233` |
| Public Bank PB enterprise | `PBB0234` |
| RHB Bank | `RHB0218` |
| Standard Chartered | `SCB0215` |
| UOB Regional | `UOB0228` |

### Duitnow Online Banking B2C (channelId 18) - Available Banks (Production)

| Bank Name | providerChannelId |
| --- | --- |
| Affin Bank (FPX) | `ABB0233` |
| Alliance Bank Malaysia Berhad (FPX) | `ABMB0212` |
| AEON Bank (RPP) | `ACDBMYK2` |
| MBSB Bank Berhad (RPP) | `AFBQMYKL` |
| Agro Bank (RPP) | `AGOBMYKL` |
| Agro Bank (FPX) | `AGRO01` |
| Ambank Berhad (FPX) | `AMBB0209` |
| Ambank Berhad (RPP) | `ARBKMYKL` |
| CIMB Bank (FPX) | `BCBB0235` |
| Bank Islam (FPX) | `BIMB0340` |
| Bank Islam (RPP) | `BIMBMYKL` |
| Bank of China (M) Berhad (RPP) | `BKCHMYKL` |
| Bank Kerjasama Rakyat Malaysia Berhad (FPX) | `BKRM0602` |
| Bank Kerjasama Rakyat Malaysia Berhad (RPP) | `BKRMMYKL` |
| Bank Muamalat (FPX) | `BMMB0341` |
| Bank Muamalat (RPP) | `BMMBMYKL` |
| Boost Bank (RPP) | `BOBEMYK2` |
| Bank Simpanan Nasional Berhad (FPX) | `BSN0601` |
| Bank Simpanan Nasional Berhad (RPP) | `BSNAMYK1` |
| CIMB Bank (RPP) | `CIBBMYKL` |
| Finexus (RPP) | `FNXSMYNB` |
| HSBC Bank (RPP) | `HBMBMYKL` |
| Hong Leong Bank Berhad (FPX) | `HLB0224` |
| HSBC Bank (FPX) | `HSBC0223` |
| Kuwait Finance House (FPX) | `KFH0346` |
| Maybank (FPX) | `MB2U0227` |
| MobilityOne (RPP) | `MBLOMYNB` |
| Alliance Bank Malaysia Berhad (RPP) | `MFBBMYKL` |
| OCBC Bank (Malaysia) Berhad (FPX) | `OCBC0229` |
| OCBC Bank (Malaysia) Berhad (RPP) | `OCBCMYKL` |
| Public Bank Berhad (FPX) | `PBB0233` |
| Public Bank Berhad (RPP) | `PBBEMYKL` |
| Affin Bank (RPP) | `PHBMMYKL` |
| RHB Banking Group (FPX) | `RHB0218` |
| Al-Rajhi Bank (RPP) | `RJHIMYKL` |
| Standard Chartered Bank Malaysia Berhad (FPX) | `SCB0216` |
| United Overseas Bank Berhad (UOB) (FPX) | `UOB0226` |
| United Overseas Bank Berhad (UOB) (RPP) | `UOVBMYKL` |

### Duitnow Online Banking B2B (channelId 19) - Available Banks (Production)

| Bank Name | providerChannelId |
| --- | --- |
| MBSB Bank Berhad (RPP) | `AFBQMYKL` |
| Agro Bank (RPP) | `AGOBMYKL` |
| Ambank Berhad (RPP) | `ARBKMYKL` |
| Bank Islam (RPP) | `BIMBMYKL` |
| Bank Kerjasama Rakyat Malaysia Berhad (RPP) | `BKRMMYKL` |
| Bank Muamalat (RPP) | `BMMBMYKL` |
| BNP Paribas Malaysia Berhad (RPP) | `BNPAMYKL` |
| JP Morgan Chase Bank Berhad (RPP) | `CHASMYKX` |
| CIMB Bank (RPP) | `CIBBMYKL` |
| Deutsche Bank (Malaysia) Berhad (RPP) | `DEUTMYKL` |
| Maybank (RPP) | `MBBEMYKL` |
| Alliance Bank Malaysia Berhad (RPP) | `MFBBMYKL` |
| Public Bank Berhad (RPP) | `PBBEMYKL` |
| Affin Bank (RPP) | `PHBMMYKL` |
| RHB Banking Group (RPP) | `RHBBMYKL` |
| Al-Rajhi Bank (RPP) | `RJHIMYKL` |
| Sumitomo Mitsui Banking Corporation (M) Berhad (RPP) | `SMBCMYKL` |
| United Overseas Bank Berhad (UOB) (RPP) | `UOVBMYKL` |

### FPX B2C (Provider B) (channelId 35) - Available Banks (Production)

| Bank Name | providerChannelId |
| --- | --- |
| Affin Bank | `ABB0233` |
| Alliance Bank (Personal) | `ABMB0212` |
| AGRONet | `AGRO01` |
| AmBank | `AMBB0209` |
| CIMB Clicks | `BCBB0235` |
| Bank Islam | `BIMB0340` |
| Bank Rakyat | `BKRM0602` |
| Bank Muamalat | `BMMB0341` |
| Bank Of China | `BOCM01` |
| BSN | `BSN0601` |
| Hong Leong Bank | `HLB0224` |
| HSBC Bank | `HSBC0223` |
| KFH | `KFH0346` |
| Maybank2U | `MB2U0227` |
| Maybank2E | `MBB0228` |
| MBSB Bank | `MBSB001` |
| OCBC Bank | `OCBC0229` |
| Public Bank | `PBB0233` |
| RHB Bank | `RHB0218` |
| Standard Chartered | `SCB0216` |
| UOB Bank | `UOB0226` |

### FPX B2B (Provider B) (channelId 36) - Available Banks (Production)

| Bank Name | providerChannelId |
| --- | --- |
| AFFINMAX | `ABB0235` |
| Alliance Bank (Business) | `ABMB0213` |
| AGRONetBIZ | `AGRO02` |
| AmBank | `AMBB0208` |
| CIMB Bank | `BCBB0235` |
| Bank Islam | `BIMB0340` |
| i-bizRAKYAT | `BKRM0602` |
| Bank Muamalat | `BMMB0342` |
| BNP Paribas | `BNP003` |
| Citibank Corporate Banking | `CIT0218` |
| Deutsche Bank | `DBB0199` |
| Hong Leong Bank | `HLB0224` |
| HSBC Bank | `HSBC0223` |
| KFH | `KFH0346` |
| Maybank2E | `MBB0228` |
| MBSB Bank | `MBSB001` |
| OCBC Bank | `OCBC0229` |
| Public Bank PBe | `PBB0233` |
| Public Bank PB enterprise | `PBB0234` |
| RHB Bank | `RHB0218` |
| Standard Chartered | `SCB0215` |
| UOB Regional | `UOB0228` |

## Payout Bank Codes (providerChannelId for bank payouts)

| Bank Name | Bank Code |
| --- | --- |
| UOB Bank | UOVBMYKL |
| Affin Bank | PHBMMYKL |
| Agrobank | AGOBMYKL |
| Al Rajhi Bank | RJHIMYKL |
| Alliance Bank | MFBBMYKL |
| Ambank | ARBKMYKL |
| Bank Islam | BIMBMYKL |
| Bank Kerjasama Rakyat | BKRMMYKL |
| Bank Muamalat | BMMBMYKL |
| Bank of America | BOFAMY2X |
| Bank of China | BKCHMYKL |
| Bank Simpanan Nasional | BSNAMYK1 |
| BNP Paribas | BNPAMYKL |
| China Construction Bank | PCBCMYKL |
| CIMB Bank | CIBBMYKL |
| Citibank | CITIMYKL |
| Deutsche Bank | DEUTMYKL |
| Hong Leong Bank | HLBBMYKL |
| HSBC Bank | HBMBMYKL |
| Industrial & Commercial Bank of China | ICBKMYKL |
| JP Morgan Chase | CHASMYKX |
| Kuwait Finance House | KFHOMYKL |
| Maybank | MBBEMYKL |
| MBSB Bank Berhad | AFBQMYKL |
| Mizuho Bank | MHCBMYKA |
| MUFG bank | BOTKMYKX |
| OCBC Bank | OCBCMYKL |
| Public Bank | PBBEMYKL |
| RHB Bank | RHBBMYKL |
| Standard Chartered Bank | SCBLMYKX |
| Sumitomo Mitsui Banking Corporation | SMBCMYKL |
| GX Bank Berhad | GXSPMYKL |
| Merchantrade Asia Sdn Bhd | MASBMYNB |
| Boost Bank Berhad | BOBEMYK2 |
| AEON Bank (M) Berhad | ACDBMYK2 |
| TNG eWallet\* | TNGDMYNB |

For TNG eWallet (`TNGDMYNB`), put the recipient DuitNow ID in `customer.username`.

## Payout DuitNow Proxy Codes (providerChannelId for proxy payouts)

| Proxy Name | Proxy Code |
| --- | --- |
| Mobile Number | MBNO |
| National ID (NRIC) | NRIC |
| Business Registration Number | BREG |

Put the NRIC or BRN in `customer.username` (docs sample). For a mobile-number proxy, `customer.mobileNo` is required; the docs do not say whether the number must also be in `customer.username`. The proxy must be registered with DuitNow and linked to a valid account.

## Staging Payout Test Accounts (staging only)

Bank account:

| Bank Code | Account Number | Account Name |
| --- | --- | --- |
| UOVBMYKL | 2093011793 | ACCOUNT DUITNOW - HYPHEN ACCOUNT DUITNOW - HYPHEN |

DuitNow Proxy:

| Proxy Type | Proxy Number |
| --- | --- |
| Business Registration Number | 111111M |
| Mobile Number | 60172223334 |
| Mobile Number | 60162500001 |
| NRIC | 660124020789 |
| NRIC | 801010100124 |

## Payin Test Data

Staging FPX lists simulator banks `TEST0021` (SBI Bank A), `TEST0022` (SBI Bank B), `TEST0023` (SBI Bank C) and other test IDs above. Published test card numbers for payins: not documented in official docs (checked 2026-10-21). The `channelDetails` example in openapi.json (`5123450000000008`, `01`/`39`, `100`) is a schema example, not a declared test card.
