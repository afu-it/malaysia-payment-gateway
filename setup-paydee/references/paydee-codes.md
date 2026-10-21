# Paydee Code Tables

Source: Paydee Payment Acceptance Gateway API Spec v5.0.0, sections 2.4 to 2.11 (`https://developer.paydee.co/download/Paydee_Payment_Acceptance_Gateway_API_Spec_v5.0.0.pdf`). Checked 2026-10-21. Payment status (`MPI_STATUS`) is in `paydee-api.md`.

## API Response Codes (`errorCode` / `MPI_ERROR_CODE`)

| Code | Description |
|---|---|
| `000` | Success Transaction |
| `001` | Transaction failed |
| `002` | Transaction have been void/reversal |
| `003` | Transaction timeout |
| `004` | Transaction in processing |
| `101` | Message not recognised (not AReq, ARes, CReq, CRes, PReq, PRes, RReq, RRes) |
| `102` | Message version number not valid for the receiving component |
| `103` | Exceeded maximum number of PReq messages sent to the DS |
| `201` | Required message element missing (element named in `MPI_ERROR_DESC`) |
| `202` | Critical message extension not recognised |
| `203` | Data element not in required format or value invalid (element named in `MPI_ERROR_DESC`) |
| `204` | Valid data element present more than once |
| `301` | Transaction ID not valid for the receiving component |
| `302` | Data could not be decrypted |
| `303` | Access denied, invalid endpoint |
| `304` | ISO code not valid (country or currency) |
| `305` | Cardholder account number not in a range belonging to issuer |
| `306` | Merchant Category Code not valid for payment system |
| `307` | Serial number not valid |
| `402` | Transaction timed-out |
| `403` | Transient system failure |
| `404` | Permanent system failure |
| `405` | System connection failure |
| `503` | Invalid Merchant |
| `512` | Invalid Transaction (see referral codes 512) |
| `554` | Expired PAN expiry entered |
| `557` | Transaction not permitted to card |
| `579` | Acquirer Host Declined (ISO 8583 DE39 in `MPI_REFERRAL_CODE`) |
| `589` | No Active Terminal |
| `594` | Duplicate Transaction |
| `596` | System malfunction |
| `5A0` | MAC verification failed |
| `5A4` | Acquirer Timeout |
| `999` | Challenge Failed |
| `PAG01` | Payment method is not available for merchant |
| `PAG02` | The chosen e-wallet channel is not supported |
| `PAG03` | Invalid security hash string from gateway |
| `PAG04` | The chosen FPX bank is not supported |
| `PAG05` | Exceeded transaction authentication max failed attempt |
| `PAG06` | Recurring should start on [date] |
| `PAG07` | Recurring has expired. Last cycle was [date] |
| `PAG08` | Recurring for billing cycle [date] has been paid |
| `PAG09` | Recurring not initialized |
| `PAG10` | Original recurring transaction is not success |
| `PAG11` | Recurring has been terminated |
| `PAG12` | Recurring has been cancelled |
| `PAG13` | Recurring has been expired |
| `PAG100` | Invalid transaction (e-wallets) |
| `PAG200` | Invalid transaction (FPX) |
| `PAG300` | Invalid transaction (Trade) |

INQ reuses `000`..`004` as the original transaction's state: successful, failed, cancelled, expired (timeout), in progress.

## Referral Codes For `512`

`01` transaction not exist, `02` failed transaction, `03` already voided, `04` already reversed, `05` already settled, `06` not allowed to void.

## Referral Codes For `579` (ISO 8583 DE39, cards)

`00` approved, `01` refer to card issuer, `02` refer to issuer special condition, `03` invalid merchant, `04` pick-up card, `05` do not honour, `06` check value error, `07` pick-up special condition, `08` honour with identification, `09` request in progress, `10` approved partial amount, `11` approved VIP, `12` invalid transaction, `13` invalid amount, `14` invalid card number, `15` no such issuer, `16` approved update track 3, `17` customer cancellation, `18` customer dispute, `19` re-enter transaction, `20` invalid response, `21` no action taken, `25` unable to locate record, `30` format error, `31` bank not supported by switch, `32` completed partially, `33` expired card, `34` fraud card, `35` card acceptor contact acquirer, `36` restricted card, `37` call acquirer security, `38` PIN tries exceeded, `39` no credit account, `40` function not supported, `41` lost card, `42` no universal account, `43` stolen card pick-up, `45` new inactivated card, `46` closed card account, `51` not sufficient funds, `52` no cheque account, `53` no savings account, `54` expired card, `55` incorrect PIN, `56` no card record, `57` not permitted to cardholder, `58` not permitted to terminal, `59` suspected fraud, `61` exceeds withdrawal amount limit, `62` restricted card, `63` security violation, `64` original amount incorrect, `65` exceeds withdrawal frequency, `68` response received too late, `75` PIN tries exceeded, `76` invalid prod code, `77` reconcile error, `78` inactivate block card, `79` declined, `80` batch number not found, `84` validate ARQC error, `86` PIN reuse, `87` PIN required, `89` bad terminal ID, `91` issuer or switch inoperative, `92` routing not found, `94` duplicate transaction, `95` reconcile error, `96` system malfunction, `97` account currency error, `99` PIN block error.

## Referral Codes For `PAG100` (e-wallets)

- Generic: `P14` amount must be more than CUR XXX.XX; `P15` amount must be less than CUR XXX.XX.
- Boost: `E0005` mandatory parameter missing, `E0006` message too long, `E0007` message too short, `E0008` invalid email, `E0009` invalid date format, `E0010` invalid double format, `E0013` invalid msisdn, `E0016` error occurred, `E0017` invalid path variable, `E0020` merchant not found, `E0021` merchant integration info not found, `E0022` payment token expired, `E0023` merchant transaction not found, `E0024` already paid for token, `E0027` insufficient wallet balance, `E0032` invalid merchant category, `E0038` minimum bio data not found, `BOOST_11` buyer cancelled.
- GrabPay: `99/ Invalid transaction_id` customer did not pay, `99/Record not found`, `currency_mismatch`, `init_record_not_exist`, `payment_not_found`, `partial_refund_not_allowed`, `merchant_insufficient_balance`, `exceed_payment_amount`, `no_record_found`, `charging`, `Refunding`, plus OAuth/OIDC errors `invalid_request`, `unauthorized_client`, `Access_denied`, `unsupported_response_type`, `invalid_scope`, `server_error`, `temporarily_unavailable`, `interaction_required`, `login_required`, `account_selection_required`, `consent_required`, `invalid_request_uri`, `invalid_request_object`, `request_not_supported`, `request_uri_not_supported`, `registration_not_supported`.
- Maybank QR Push: `QR098` try later, `QR090` mandatory fields missing, `QR091` invalid QR ID, `QR092` invalid terminal ID, `QR122` account locked or inactive, `QR155` invalid transaction, `401` OAuth verification failed, `MB2U_11` failed, `MB2U_01` failed.
- Touch 'n Go: `00000000/INIT` not paid or unfinished, `00000000/CLOSED` order closed, `PAYING` paid but not finished, `MERCHANT_ACCEPT` accepted by merchant, `CANCELLED`.
- Alipay: `ILLEGAL_SIGN`, `ILLEGAL_ARGUMENT`, `HASH_NO_PRIVILEGE`, `ILLEGAL_SERVICE`, `ILLEGAL_PARTNER`, `ILLEGAL_SIGN_TYPE`, `FOREX_MERCHANT_NOT_SUPPORT_THIS_CURRENCY`, `ILLEGAL_SECURITY_PROFILE`, `REPEAT_OUT_TRADE_NO`, `ILLEGAL_CURRENCY`, `ILLEGAL_PARTNER_EXTERFACE`, `SYSTEM_EXCEPTION`, `ILLEGAL_TIMEOUT_RULE`, `ILLEGAL_CHARSET`, `99/TRADE_NOT_EXIST` (no payment or onboarding failed).
- UnionPay: `01` refer to issuer, `03` invalid merchant, `05` account closed, `06` card expired, `09` in progress, `12` invalid transaction, `13` invalid amount, `14` invalid buyer account, `17` buyer cancel, `22` suspected malfunction, `25` original not found, `30` format or MAC error, `31` invalid bank, `39` no credit account, `51` insufficient fund, `53` no saving account, `57` not permitted, `61` withdrawal limit, `65` withdrawal frequency, `76` not found, `77` invalid buyer name/ID, `79` host decline when down, `85` bank internal error, `87` debit failed, `88` credit failed, `89` not received, `92` routing not found, `93` cannot be completed, `FE` internal error, `OE` outside operating hours, `OF` transaction timeout, `OA` session timeout, `1A`..`1L` buyer timeout/missing info/cancel at login, account selection, TAC and confirmation pages (same meanings as PAG200), `2A` below minimum amount.
- ShopeePay: `SHOPEEPAY__7` payment invalidated.

## Referral Codes For `PAG200` (FPX)

`05` invalid seller or acquiring bank code, `39` no credit account, `48` max transaction limit exceeded, `49` merchant specific limit exceeded, `51` insufficient funds, `53` no buyer account number, `57` not permitted, `58` transaction to merchant not permitted, `76` not found, `77` invalid buyer name, `79` host decline when down, `80` buyer cancel, `85` bank internal error, `87` debit failed, `88` credit failed, `89` not received, `90` internet banking unavailable, `92` invalid buyer bank, `99` pending authorization (B2B), `BB` blocked bank, `BC` cancelled by customer, `DB` invalid email format, `OE` outside FPX operating hours, `SB` invalid acquiring bank code, `XA` invalid source IP (B2B2), `XE` invalid message, `XF` invalid number of orders, `XW` seller exchange date difference exceeded, `1A` login page timeout, `1B` login info missing, `1C` cancel at login, `1D` account selection timeout, `1E` account selection info missing, `1F` cancel at account selection, `1G` TAC page timeout, `1H` TAC info missing, `1I` cancel at TAC, `1J` confirmation timeout, `1K` confirmation info missing, `1L` cancel at confirmation, `2A` below minimum limit, `2X` cancelled by merchant, `BO` order list format error, `B1` invalid seller ID, `B2` seller not allowed to refund, `B3` multiple refund not allowed, `B4` refund exceeds maximum, `B5` original transaction not found, `B6` original pending debit/credit, `B7` original not successful, `B8` previous refund pending, `B9` refund below minimum, `C1` invalid refund model, `C2` invalid refund buyer bank, `C3` invalid refund seller bank, `C4` no valid order list, `C5` duplicate seller order number.

## Referral Codes For `PAG300` (Trade Credit)

`200003` Insufficient Credit Limit.

## 3DS Authentication Status (`MPI_AUTH_STATUS`, GPayments)

| Value | Meaning |
|---|---|
| `Y` | Authenticated, liability shift, proceed |
| `N` | Not authenticated, denied |
| `U` | Could not be performed; may proceed, no liability shift |
| `A` | Attempts processing; liability shift, may proceed |
| `C` | Challenge required |
| `R` | Rejected by issuer; do not authorise |
| `D` | Decoupled authentication confirmed (3DS 2.2.0) |
| `I` | Informational only (3DS 2.2.0) |

## 3DS Status Reason (`MPI_STATUS_REASON`)

`01` card authentication failed, `02` unknown device, `03` unsupported device, `04` exceeds authentication frequency, `05` expired card, `06` invalid card number, `07` invalid transaction, `08` no card record, `09` security failure, `10` stolen card, `11` suspected fraud, `12` not permitted to cardholder, `13` cardholder not enrolled, `14` timed out at ACS, `15` low confidence, `16` medium confidence, `17` high confidence, `18` very high confidence, `19` exceeds ACS max challenges, `20` non-payment not supported, `21` 3RI not supported, `22` ACS technical issue, `23` decoupled required but not requested, `24` decoupled max expiry exceeded, `25` decoupled insufficient time, `26` attempted but not performed by cardholder, `80`..`99` reserved for DS.
