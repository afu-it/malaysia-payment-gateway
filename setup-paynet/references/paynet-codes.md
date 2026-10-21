# PayNet Codes Reference

All code lists published on the PayNet docs, checked 2026-10-21. Sources: `/docs/fpx/response-codes`, `/docs/fpx/mapping-table`, `/docs/duitnow-pay/response-codes`, `/docs/duitnow-pay/mapping-tables`, `/docs/duitnow-pay/transaction-codes`, `/docs/online-banking-wallet/response-codes`, `/docs/duitnow-qr/response-codes`, `/docs/operations/financial-reports/financial-reports-api/responses`, `/docs/API/http-status-codes`. The DuitNow Pay response-code page is a superset of the Online Banking/Wallet and DuitNow QR pages (it adds D316, E001 to E004, U240, U241 and others).

## FPX

### FPX services response codes (`fpx_debitAuthCode`)

| Code | Description |
|---|---|
| 00 | Approved |
| 05 | Invalid Seller Or Acquiring Bank Code |
| 09 | Transaction Pending |
| 12 | Invalid Transaction |
| 13 | Invalid Amount |
| 20 | Invalid Response |
| 30 | Format Error |
| 31 | Invalid Bank |
| 45 | Duplicate Seller Order Number |
| 46 | Invalid Seller Exchange Or Seller |
| 48 | Maximum Transaction Limit Exceeded |
| 50 | Invalid Seller for Merchant Specific Limit |
| 51 | Insufficient Funds |
| 53 | No Buyer Account Number |
| 57 | Transaction Not Permitted |
| 58 | Transaction To Merchant Not Permitted |
| 70 | Invalid Serial Number |
| 76 | Transaction Not Found |
| 77 | Invalid Buyer Name Or Buyer ID |
| 78 | Decryption Failed |
| 79 | Host Decline When Down |
| 80 | Buyer Cancel Transaction |
| 83 | Invalid Transaction Model |
| 84 | Invalid Transaction Type |
| 85 | Internal Error At Bank System |
| 88 | Credit Failed Exception Handling |
| 89 | Transaction Not Received Exception Handling |
| 90 | Bank Internet Banking Unavailable |
| 92 | Invalid Buyer Bank |
| 96 | System Malfunction |
| 98 | MAC Error |
| 99 | Pending Authorization (Applicable for B2B model) |
| BC | Transaction Cancelled By Customer |
| DA | Invalid Application Type |
| DB | Invalid Email Format |
| DC | Invalid Maximum Frequency |
| DD | Invalid Frequency Mode |
| DE | Invalid Expiry Date |
| DF | Invalid e-Mandate Buyer Bank ID |
| FE | Internal Error |
| OE | Transaction Rejected As Not In FPX Operating Hours |
| SB | Invalid Acquiring Bank Code |
| XA | Invalid Source IP Address (Applicable for B2B2 model) |
| XE | Invalid Message |
| XF | Invalid Number Of Orders |
| XM | Invalid FPX Transaction Model |
| XN | Transaction Rejected Due To Duplicate Seller Exchange Order Number |
| XO | Duplicate Exchange Order Number |
| XS | Seller Does Not Belong To Exchange |
| XT | Invalid Transaction Type |
| XW | Seller Exchange Date Difference Exceeded |
| 1B | Buyer Failed To Provide The Necessary Info To Login To Internet Banking Login Page |
| 1C | Buyer Choose Cancel At Login Page |
| 1D | Buyer Session Timeout At Account Selection Page |
| 1E | Buyer Failed To Provide The Necessary Info At Account Selection Page |
| 1F | Buyer Choose Cancel At Account Selection Page |
| 1G | Buyer Session Timeout At TAC Request Page |
| 1H | Buyer Failed To Provide The Necessary Info At TAC Request Page |
| 1I | Buyer Choose Cancel At TAC Request Page |
| 1J | Buyer Session Timeout At Confirmation Page |
| 1K | Buyer Failed To Provide The Necessary Info At Confirmation Page |
| 1L | Buyer Choose Cancel At Confirmation Page |
| 1M | Internet Banking Session Timeout. |
| 2A | Transaction Amount Is Lower Than Minimum Limit |
| 2X | Transaction Is Canceled By Merchant |

The source page also shows codes `03`, `14`, `39`, `47`, `49`, `87`, `BB`, `OF`, `XB`, `XI` and `1A` glued to the end of the previous row with no description (both docs sites, checked 2026-10-21). Their meanings are not documented; treat them as failed unless PayNet says otherwise.

Receipt mapping (browser-redirection page): `00` successful, `99` pending for authorisation, `09` pending, anything else unsuccessful.

### FPX bulk refund extended codes

| Code | Description |
|---|---|
| B0 | Order list format error |
| B1 | Invalid seller ID |
| B2 | Seller is not allow to refund |
| B3 | Seller is not allow to do multiple refund |
| B4 | Requested refund amount exceed maximum allowable |
| B5 | Original transaction ID is not found |
| B6 | Original transaction ID status is still pending debit/credit |
| B7 | Original transaction ID status was not successful |
| B8 | Previous refund request still pending debit/credit |
| B9 | Requested refund amount below minimum allowable |
| C1 | Invalid refund transaction model |
| C2 | Invalid refund buyer bank |
| C3 | Invalid refund seller bank |
| C4 | Refund request fail due to no valid order list |
| C5 | Order list contain duplicate seller order number |
| 1S | Bulk Refund Successful Submitted (from the bulk refund flow page; glued without description in the code table) |
| 00 | Bulk Refund Debit Approved |

### FPX Webview services and API general codes

| Code | Description |
|---|---|
| FPX-200 | Success |
| FPX-204 | Invalid URL |
| FPX-400 | Invalid format |
| FPX-404 | Item not found |
| FPX-401 | Unauthorized |
| FPX-500 | Internal server error |
| API.001 | Invalid URL |
| API.002 | Invalid format |
| API.003 | Invalid length |
| API.004 | Invalid value |
| API.005 | Missing mandatory field |
| API.007 | Unauthorized request (DuitNow Pay list) |
| API.008 | Request timeout |

FPX sample certificate check codes (PHP sample): `00` verified, `09` data cannot be verified against the signature, `06` invalid certificates, `07` one certificate found and expired, `08` both certificates expired.

## DuitNow (RPP, DuitNow Pay, v3)

### Status codes

| Code | Name | Meaning |
|---|---|---|
| ACSP | AcceptedSettlementInProcess | Checks passed; accepted for execution |
| ACTC | AcceptedTechnicalValidation | Authentication and syntax/semantic validation passed |
| RJCT | Rejected | Rejected |

Other ISO status codes in the mapping tables, not used by RPP: ACCP, ACSC, ACWC, PNDG.

### DuitNow Pay substates

Payment: `RECEIVED` (pending), `CLEARED` (successful credit), `REJECTED` (rejected by debiting agent), `PENDAUTH` (pending authorisation). Mapping-table names also list `PENDING AUTHORISATION`/`ACCEPTED` and `CANCELLED` (cancelled by merchant). AutoDebit response: `CLEARED`, `REJECTED`. Refund: `REFUND_PENDING`, `REFUND_ACCEPTED`, `REFUND_REJECTED`, `REFUND_EXCEPTION`. Consent request status: `PDNG`, `ACTV`, `CANC`, `PDAU`, `EXPI`.

### Consent status

| Code | Meaning |
|---|---|
| ACTV | Active |
| ICTV | Inactive |
| SUSP | Suspended by customer |
| SUSB | Suspended by bank |
| ACTB | Activated by bank |
| PNDG | Pending |
| PDAU | Pending authorisation |
| CANC | Cancelled |
| EXPI | Pending consent passed its expiry period |

Consent termination reason codes: `01` not using service anymore, `02` dispute on payment details, `03` not interested, `04` fraud, `05` early settlement of contract, `99` other.

### Mapping tables (DuitNow Pay)

- Identification type: `01` New IC, `02` Army, `03` Passport, `04` Registration number, `05` Mobile.
- Identity validation: `00` none, `01` debtor name, `02` debtor ID, `03` name and ID.
- Source of funds: `01` CASA, `02` credit card, `03` eWallet. v3 enums: `CASA`, `CREDIT_CARD`, `WALLET`.
- Frequency: `01` unlimited, `02` daily, `03` weekly, `04` monthly, `05` quarterly, `06` yearly.
- Account type: `CACC` current, `SVGS` savings, `CCRD` credit card, `WALL` eWallet. Cash account: `CACC`, `LOAN`, `SVGS`, `CCRD`, `DFLT`.
- Resident status `01`/`02`/`03`; product type `I`/`C`; sharia `Y`/`N`/`U`; account owner `1` single, `2` joint, `3` government, `4` corporate, `5` NGO.
- Proxy type: `NRIC`, `PSPT`, `BREG`, `MBNO`, `ARMN`. Proxy registration: `ACTV`, `AMND`, `DEAC`, `NEWR`, `PORT`, `SUSP`. Proxy status: `ACTV`, `SUSP`, `ICTV`, `SUSB`.
- Channel: `RB`, `CB`, `RM`, `CM`, `OT`, `BW`, `BA` (and `QR` in v3). Originator: `O`, `H`, `R`.
- Charge bearer: `DEBT`, `CRED`, `SHAR`, `SLEV`. Settlement method: `CLRG`. Category purpose codes: BONU, CASH, CBLK, CCRD, CORT, DCRD, DIVI, DVPM, EPAY, FCOL, GOVT, HEDG, ICCP, IDCP, INTC, INTE, LOAN, OTHR, PENS, RVPM, SALA, SECU, SSBE, SUPP, TAXS, TRAD, TREA, VATX, WHLD.
- Event codes: `2001` bank availability status, `2003` liquidity position limit breached, `2004` settlement notification, `CMN001` consent update, `CMN002` consent termination, `PMT001` payment status, `CRN001` consent update status, `RED001` RTP redirect, `RED002` consent registration redirect. Function codes: `1001` sign on, `1002` sign off, `1003` echo.
- Allowed characters: ID `+'-/A-Za-z0-9*`; account ID `A-Za-z0-9*+/.=`; IP `.0-9:A-Za-z`; name `[A-Za-z0-9 @'/&()-_,\'*]:#+!";$%^{}~`; proxy `+*A-Za-z0-9/#`; reference `+'/_-.,():?A-Za-z0-9@!$\`; URL `-/:+-?=&A-Za-z0-9@`.

### RPP transaction codes (selection relevant to acquiring)

| Code | Meaning |
|---|---|
| 010 | Credit transfer (DuitNow instant transfer) |
| 110 | Credit transfer pay-to-proxy |
| 510 | Account enquiry for credit transfer |
| 030 / 031 / 040 | QR POS / cross-border QR POS / QR P2P payment |
| 520 / 521 / 522 | QR account enquiry domestic / cross-border / cash-out |
| 032 | QR cash-out payment |
| 240 / 880 / 250 / 872 | CPM QR debit / pre-auth / post-auth debit / cancellation |
| 630 | Transaction status enquiry by Business Message Identifier |
| 650 | Request list of banks |
| 851 / 853 / 852 | Request-to-pay by proxy / by account / reject |
| 855 / 856 | Request-to-pay forward by proxy / by account |
| 060 | Credit transfer (RTP decoupled) |
| 861 / 862 / 863 / 864 / 865 | RTP redirect / retrieve info / update status / status enquiry / cancellation |
| 070 | Credit transfer (RTP redirect) |
| 854 | Request-to-pay refund (refund intent) |
| 801 / 802 / 803 / 804 | Consent registration via proxy / via account / register / reject (decoupled) |
| 811 / 812 / 813 / 814 / 815 | Consent registration redirect / retrieve detail / update status / status enquiry / cancellation |
| 821 / 822 / 823 | Consent via debiting agent / update status / acceptance coupled with RTP |
| 751 / 752 | Consent enquiry by secondary ID / by consent ID |
| 763 / 764 | Consent termination |
| 660 / 670 | List of merchants / merchant products |
| 871 | AutoDebit pre-authorisation |
| 210 / 220 / 230 | Real-time debit from bank / from merchant / pre-auth |
| 080 | Direct debit credit to acquirer |
| 011 / 012 | Credit transfer reversal clear / masked |
| 610 / 620 / 710 / 720 / 901 | Proxy resolution / enquiry / registration / maintenance / bank transfer notice |
| 611 / 111 | Cross-border pay-to-proxy enquiry / credit transfer |
| 000 | Network administration (sign on, sign off, echo) |

### End-to-end return codes (highlights from the DuitNow Pay page)

| Situation | Code to OFI |
|---|---|
| Successful | `ACSP` `U000` |
| Account enquiry timed out | `RJCT` `U172` |
| Credit transfer timed out, no response from participant | `ACTC` `U002` (stored in SAF) |
| Invalid signature OFI to RPP | `RJCT` `U999` |
| Successful but signature error in response | `ACTC` `U998` |
| Invalid signature RPP to RFI | `RJCT` `25` |
| Unable to credit (dormant, invalid, closed, blacklisted, held, deceased) | `RJCT` `45` |
| Account does not exist | `RJCT` `52`, `53` or `N3` (mapped to `53`) |

Cross-border foreign participant reason codes (Singapore and others) are also listed on that page; they apply only to cross-border P2P.

### Reason codes (RPP, DuitNow Pay, v3 `resp.reason.code`)

| Code | Description | Status |
|---|---|---|
| U000 | Success/ Transaction Accepted | ACSP |
| U002 | Success/ Transaction Accepted Stored in SAF | ACTC |
| U100 | Product Not Found | RJCT |
| U101 | Tenant Not Found | RJCT |
| U102 | Tenant Not Active | RJCT |
| U103 | Tenant Undefined | RJCT |
| U104 | Inbound Scheme Not Found | RJCT |
| U105 | Inbound Scheme Not Active | RJCT |
| U106 | Original Payment Not Found | RJCT |
| U107 | Original Payment Rejected | RJCT |
| U108 | Original Payment Not Responded | RJCT |
| U110 | Payment Not Accepted | RJCT |
| U111 | Minimum Amount Check Failed | RJCT |
| U112 | Maximum Amount Check Failed | RJCT |
| U114 | Mod 10 check failed | RJCT |
| U115 | Date Sent Tolerance Check failed | RJCT |
| U116 | Date Settlement Tolerance Check failed | RJCT |
| U117 | Original Date Tolerance configuration error | RJCT |
| U118 | Original Date Tolerance check failed | RJCT |
| U119 | Session Validation Failed | RJCT |
| U121 | Inbound Bank Not Found | RJCT |
| U122 | Inbound Bank Not Active | RJCT |
| U123 | Product Undefined | RJCT |
| U124 | Bank Code Not found in message | RJCT |
| U125 | Inbound Bank Id Cannot be Determined (Bank Classifier not found) | RJCT |
| U126 | Outbound Bank Cannot be Determined | RJCT |
| U128 | Outbound Bank Not Active | RJCT |
| U129 | Payee Bank Unavailable | RJCT |
| U130 | Inbound Settlement Bank not found | RJCT |
| U131 | Outbound Settlement Bank not found | RJCT |
| U132 | Inbound Settlement Bank Inactive | RJCT |
| U134 | Outbound Settlement Bank Inactive | RJCT |
| U135 | Pre-Authorization Match Not Found | RJCT |
| U136 | Pre-Authorization Bank Account Check Failed | RJCT |
| U137 | Pre-Authorization Timeout | RJCT |
| U138 | Pre-Authorization Config Not Found | RJCT |
| U139 | Pre-Authorization Endpoint or Channel Not Configured | RJCT |
| U140 | Pre-Authorization Exception No Response Returned | RJCT |
| U141 | Pre-Authorization Exception | RJCT |
| U142 | Pre-Authorization Stand-In Limit Exceeded | RJCT |
| U143 | Pre-Authorization Stand-In Account Not Found | RJCT |
| U144 | Pre-Authorization Stand-In Account Inactive | RJCT |
| U145 | Pre-Authorization Stand-In Insufficient Funds | RJCT |
| U146 | Pre-Authorization Stand-In Not Permitted | RJCT |
| U147 | Pre-Authorization No Stand-In Response | RJCT |
| U149 | Duplicate Transaction | RJCT |
| U150 | Notification Configuration Not Found | RJCT |
| U151 | Notification Endpoint Or Channel Not Configured | RJCT |
| U155 | Fraud Check Failed | RJCT |
| U156 | Sanction Check Failed | RJCT |
| U157 | AML Check Failed | RJCT |
| U159 | Pre-Authorization Stand-In Rejected with Default Status | RJCT |
| U160 | Settlement Rules Not Found | RJCT |
| U161 | Settlement Endpoint or Channel Not Configured | RJCT |
| U162 | Settlement Confirmation Not Received | RJCT |
| U163 | Forwarded Settlement Confirmation Response Not Received | RJCT |
| U164 | Settlement Total Management Not Configured | RJCT |
| U170 | Online Route Not Found | RJCT |
| U171 | Online Authorization Exception | RJCT |
| U172 | Online Authorization Exception No Response returned | RJCT |
| U173 | Online Authorization Exception Timeout | RJCT |
| U180 | Authorization Service is Not Configured | RJCT |
| U181 | Stand-In Limit Exceeded | RJCT |
| U182 | Stand-In Account Not Found | RJCT |
| U183 | Stand-In Account Inactive | RJCT |
| U184 | Stand-In Insufficient Funds | RJCT |
| U185 | Stand-In Not Permitted | RJCT |
| U186 | Stand-In No Response | RJCT |
| U187 | Stand-In forwarding Advice cannot be delivered | RJCT |
| U189 | Stand-In Rejected With Default Status | RJCT |
| U190 | LiquidityManagementNotConfigured | RJCT |
| U191 | Liquidity Rules Not Found | RJCT |
| U192 | Liquidity Position Validation Rule not found (Threshold limits) | RJCT |
| U193 | Liquidity Position Validation Failure | RJCT |
| U194 | Insufficient Liquidity to process payment | RJCT |
| U195 | Liquidity Forced Payment Rejection | RJCT |
| U196 | Liquidity Parameter Invalid Or Null | RJCT |
| U197 | Liquidity Parameters Not Found | RJCT |
| U198 | Liquidity Validation Not Found | RJCT |
| U199 | Liquidity Position Definition Not Found | RJCT |
| U200 | LQM Maximum Amount Not Found | RJCT |
| U201 | LQM Maximum Amount Exceeded | RJCT |
| U202 | LQM Net Sender Threshold Limit Not Found | RJCT |
| U203 | LQM Insufficient Net Sender Funds | RJCT |
| U205 | Scheme Settlement Body Unavailable | RJCT |
| U206 | Settlement Rejection | RJCT |
| U207 | Default Settlement Body Not Defined | RJCT |
| U208 | Settlement Forward Confirmation Send Failed | RJCT |
| U209 | Settlement Forward Confirmation Receive Failed | RJCT |
| U210 | Outbound Scheme Not Found | RJCT |
| U211 | Outbound Scheme Not Active | RJCT |
| U212 | Connector Not Found | RJCT |
| U213 | Connector Configuration Error | RJCT |
| U214 | Clearing Rejection | RJCT |
| U215 | Message Rejected with admi.002 | RJCT |
| U221 | MandateRulesNotFound | RJCT |
| U222 | MandateMasterUpdateError | RJCT |
| U223 | MandateRecordNotFound | RJCT |
| U224 | MandateStartDateLessThanCurrentDate | RJCT |
| U225 | MandateStartDateFormatError | RJCT |
| U226 | MandateEndDateLessThanCurrentDate | RJCT |
| U227 | MandateEndDateFormatError | RJCT |
| U228 | MandateFrequencyTypeValidationError | RJCT |
| U229 | MandateUpdaterNotKnown | RJCT |
| U230 | MandateSenderNotAllowedToUpdate | RJCT |
| U231 | MandateStatusValidationError | RJCT |
| U232 | AmountGreaterThanMaximumLimit | RJCT |
| U233 | MadateAmountMissing | RJCT |
| U234 | RequestBeforeMandateStartDate | RJCT |
| U235 | FutureMandateCannotBeUpdated | RJCT |
| U236 | ExpiredMandateCannotBeUpdated | RJCT |
| U237 | CancelledMandateCannotBeUpdated | RJCT |
| U238 | RequestAfterMandateEndDate | RJCT |
| U239 | HoldDateExceedsExipryDate | RJCT |
| U240 | AdjustmentTypeError | RJCT |
| U241 | AdjustmentActionError | RJCT |
| U242 | AdjustmentMissingMandatoryField | RJCT |
| U243 | BalanceTransferBalanceNotEnough | RJCT |
| U244 | SponsorAdjustmentProcessingRuleNotFound | RJCT |
| U245 | SponsorAdjustmentAuthTimeOut | RJCT |
| U246 | SponsorAdjustmentAuthReject | RJCT |
| U247 | AdjustmentNotPerformed | RJCT |
| U248 | AdjustmentPerformedButFail | RJCT |
| U249 | AutoAdjustmentBalanceConfirmFail | RJCT |
| U250 | AliasTypeNotSupported | RJCT |
| U251 | SponsorAdjustmentAuthCommunicationError | RJCT |
| U252 | SponsorAdjustmentBalanceNotEnough | RJCT |
| U253 | InboundBankSuspended | RJCT |
| U254 | OutboundBankSuspended | RJCT |
| U255 | InboundSettlementBankSuspended | RJCT |
| U256 | OutboundSettlementBankSuspended | RJCT |
| U257 | AdjustmentAtNoneBusinessDay | RJCT |
| U258 | PRMCheckFailed | RJCT |
| U259 | PRMInvalidStatus | RJCT |
| U260 | PRMCheckDeny | RJCT |
| U261 | PRMCheckRefer | RJCT |
| U262 | AccountNotMatchAlias | RJCT |
| U263 | SenderNotEligibleForRequest | RJCT |
| U264 | MandateIsNotPendingAuthorization | RJCT |
| U265 | Acceptable Source of Fund Not Allowed | RJCT |
| U266 | CryptoAESAliasNotFound | RJCT |
| U267 | CryptoAESKeyFormatError | RJCT |
| U268 | MerchantIdNotAllowed | RJCT |
| U269 | RTPTxnNotMatched | RJCT |
| U270 | SubstateNotMatched | RJCT |
| U271 | MandateInitTxnNotMatched | RJCT |
| U272 | MandateEndDtLessEqualsThanCurrentDt | RJCT |
| U273 | RtpRecordNotFound | RJCT |
| U274 | CANotMatched | RJCT |
| U275 | DANotMatched | RJCT |
| U276 | ConsentTypeNotAllowed | RJCT |
| U277 | CryptoDecFailed | RJCT |
| U278 | CryptoEncFailed | RJCT |
| U279 | CryptoSignFailed | RJCT |
| U280 | CertPathNotConfig | RJCT |
| U281 | CryptoVerifyFailed | RJCT |
| U282 | CryptoRuleNotFound | RJCT |
| U283 | MandateBRNValueCheckFailed | RJCT |
| U284 | MandateUpdateNotAllowed | RJCT |
| U285 | MandateExpired | RJCT |
| U286 | MandateNotActive | RJCT |
| U287 | ExceedMandateMaxAmt | RJCT |
| U288 | MandateStatusNotAllowed | RJCT |
| U289 | MandateExpiryDateExceedMax | RJCT |
| U290 | MandateInitRequestExpired | RJCT |
| U300 | No QR Enquiry prior to QR Payment | RJCT |
| U301 | Invalid Source of Fund (QR) | RJCT |
| U304 | Invalid QR Response | RJCT |
| U305 | Date in BizMsgId does not match CreDtTm (QR) | RJCT |
| U306 | Invalid Secondary ID (QR) | RJCT |
| U307 | QR Category does not match transaction type (QR) | RJCT |
| U310 | RequestForPayExpired | RJCT |
| U311 | RequestForPayAlreadyTransferred | RJCT |
| U312 | RequestForPayNotFound | RJCT |
| U313 | RequestForPayAlreadyProcessed | RJCT |
| U314 | RequestForPayAlreadyRejected | RJCT |
| U315 | RequestForPayInvalidState | RJCT |
| U316 | RequestForPayAlreadyReceived | RJCT |
| U317 | RequestForPayInvalidDbtrAgt | RJCT |
| U318 | ExpiryDateLessThanCurrentDate | RJCT |
| U319 | ExpiryDateFormatError | RJCT |
| U320 | RequestedExecutionDateLessThanCurrentDate | RJCT |
| U321 | RequestedExecutionDateFormatError | RJCT |
| U322 | MandateIsNotActive | RJCT |
| U323 | InvalidDebtorAgent | RJCT |
| U324 | InvalidOriginalDebtorAgent | RJCT |
| U325 | MandateAlreadyTransferred | RJCT |
| U326 | RequestForPayAlreadyAccepted | RJCT |
| U327 | MerchantNotActive | RJCT |
| U328 | MerchantProductNotFound (Biller Code Not Found) | RJCT |
| U329 | MerchantNotFound | RJCT |
| U330 | MerchantParameterNotFound | RJCT |
| U331 | MerchantMinAmountCheckFailed | RJCT |
| U332 | MerchantMaxAmountCheckFailed | RJCT |
| U333 | DateXpryCheckFailed | RJCT |
| U334 | MandateIsNotPending | RJCT |
| U335 | RPFLookupRulesNotFound | RJCT |
| U336 | PreAuthExpiryDateFormatError | RJCT |
| U337 | PreAuthExpiryDateExceedsMandateExipryDate | RJCT |
| U338 | PreAuthExpiryDateLessThanCreDtTm | RJCT |
| U339 | PreAuthExpiryDateLessThanCurrentDate | RJCT |
| U340 | MechantProductNotActive | RJCT |
| U341 | MerchantValidationRuleNotFound | RJCT |
| U342 | MerchantProductBankCodeNotFound | RJCT |
| U343 | MandateUpdateAlreadyInProgress | RJCT |
| U344 | DDParticipantsChecksFail | RJCT |
| U345 | RejectedMandateCannotBeUpdated | RJCT |
| U346 | RejectedMandateCannotBeAuthorized | RJCT |
| U347 | RequestForPayAlreadyPendingAuthorization | RJCT |
| U348 | CreditorBankAccountDetailsMissing | RJCT |
| U800 | Alias Management action not configured | RJCT |
| U801 | Alias rules not found | RJCT |
| U802 | Addressing Privilege not defined | RJCT |
| U803 | Not sufficient privilege to perform addressing action | RJCT |
| U804 | Alias Not Found | RJCT |
| U805 | Alias is suspended | RJCT |
| U806 | Alias already registered with the same FI but different account | RJCT |
| U807 | Alias already registered with another FI | RJCT |
| U808 | Alias already registered with the same account | RJCT |
| U809 | Not sufficient privilege to perform addressing action on an alias | RJCT |
| U810 | Alias Request failed | RJCT |
| U811 | Alias suspended by Administrator | RJCT |
| U812 | Alias Destination Not Configured | RJCT |
| U813 | Cache Alias Maintenance Failed | RJCT |
| U814 | Alias already registered with same FI | RJCT |
| U815 | Duplicate Alias Service Request | RJCT |
| U816 | DuplicateAliasServiceRequestDetected | RJCT |
| U850 | System Notification Event Code Or Rules Not Found | RJCT |
| U851 | General Purpose Rules Not Found | RJCT |
| U890 | Invalid Business Hierarchy | RJCT |
| U900 | Internal Timeout | RJCT |
| U901 | System Malfunction | RJCT |
| U902 | Connection Or Communication Error | RJCT |
| U903 | Scripting Error | RJCT |
| U904 | Endpoint Error | RJCT |
| U905 | Configuration Error | RJCT |
| U906 | Logging or Database Error | RJCT |
| U907 | Session Error - No Plan Executed | RJCT |
| U908 | Validation Error | RJCT |
| U909 | Available Response Code | RJCT |
| U910 | No Session Response Within Timeout | RJCT |
| U911 | No Such Session | RJCT |
| U912 | Session Validation Error | RJCT |
| U913 | No Session Request Variable | RJCT |
| U990 | Session Error Timeout | RJCT |
| U992 | Response Not Delivered | RJCT |
| U993 | Reason Code not determined in response | RJCT |
| U999 | Signature Validation Failed on Request | RJCT |
| P001 | Invalid Forex Rate | RJCT |
| P002 | Settlement Amount is not matched with Field3 | RJCT |
| P003 | Non Bank Participant Not Allow | RJCT |
| P004 | Duplicate Forex Record | RJCT |
| P005 | Invalid Forex Record | RJCT |
| P006 | Forex Rate Expired | RJCT |
| P007 | Field 3 and IntrBkSttlmAmt not match | RJCT |
| P008 | Preauth Amount exceed limit | RJCT |
| P009 | Preauth Limit not configured | RJCT |
| P010 | Invalid Ref-1 | RJCT |
| P990 | Session Error Timeout | RJCT |
| E001 | No matching merchant product found | RJCT |
| E002 | No transaction log data found | RJCT |
| E003 | Refund amount exceed refund percentage allowed | RJCT |
| E004 | No matching merchant found | RJCT |

### Reject reasons (message validation, admi.002)

| Code | Description |
|---|---|
| 0001 | Missing mandatory Field |
| 0002 | Invalid attribute type |
| 0003 | Endpoint has no definition for given attribute type |
| 0004 | Invalid character detected during decoding of BCD encoded length |
| 0005 | Unsupported length encoding |
| 0006 | Sign byte for a BCD number was not 'C' or 'D' |
| 0007 | Invalid Sign Nibble |
| 0008 | Extra content for array field while decoding value |
| 0009 | Not enough elements for array field of fixed number of elements while decoding value |
| 0010 | Array field too long while encoding |
| 0011 | Array field too short while encoding |
| 0012 | Length value too long to encode for specified length type |
| 0013 | Decoding of BER TLV field reached end prematurely |
| 0014 | Encoding content too long for size of field |
| 0015 | Inbound field validation failed |
| 0016 | Outbound field validation failed |
| 0017 | Inbound validation rule failed |
| 0018 | Outbound validation rule failed |
| 0019 | Script execution error |
| 0020 | Inbound Assignment Error |
| 0021 | Outbound Assignment Error |
| 0022 | Configuration Error |
| 0023 | Unknown Field |
| 0024 | No EftContexts found |
| 0025 | No EftContext found |
| 0026 | No Card Type found |
| 0027 | No Card Service Type found |
| 0028 | Duplicate Message |
| 0029 | Invalid data found for decode |
| 0030 | Invalid data found for encode |
| 0031 | Encoding content too short for size of field |
| 0032 | Invalid tag delimiter |
| 0033 | Invalid attribute delimiter |
| 0034 | Not matching end tag |
| 0035 | Not well formed XML data |
| 0036 | Decoded length is invalid |
| 0037 | Unsupported Message Composition |
| 0038 | Unsupported Message Type |
| 0039 | Field Not Enabled |
| 0040 | No Matching Session Found |
| 0041 | No Message type |
| 0042 | Runtime Error |
| 0043 | No Session Definition |
| 0044 | Message Correlation Error |
| 0045 | NO_ALIAS_PRODUCED |
| 0046 | DECODE_VALUE_TOO_SHORT |
| 0047 | DECODE_VALUE_TOO_LONG |
| 0048 | INVALID_XML_PCDATA |
| 0049 | Session Not Active |
| 0050 | NO Outbound Handler |
| 0051 | File load session processing error |
| 0052 | Terminal/Merchant missing or inactive |
| 0053 | Invalid enum value found during BSI marshalling/ unmarshalling |
| 0054 | Encoded payload is too large for configured maximum message size |
| 0055 | Encoded header is too large for configured maximum message size |
| 0056 | Encoded trailer is too large for configured maximum message size |
| 0057 | MAC validation failed |
| 0058 | MAC generation failed |
| 0059 | Data encryption failed |
| 0060 | Data decryption failed |
| 0061 | PIN translation failed |
| 0062 | PIN verification failed |

### Issuer rejection codes (D codes)

| Code | Name | Description | Legacy code |
|---|---|---|---|
| D101 | FAILED_BY_CUSTOMER | User Cancel During Login |  |
| D102 | FAILED_BY_CUSTOMER | User Cancel During Account Selection |  |
| D103 | FAILED_BY_CUSTOMER | User Cancel During TAC Request |  |
| D104 | FAILED_BY_CUSTOMER | User Cancel During Confirmation |  |
| D105 | FAILED_BY_CUSTOMER | User Fail to Login |  |
| D106 | FAILED_BY_CUSTOMER | Fail to Authenticate MFA |  |
| D107 | FAILED_BY_CUSTOMER | Session Timeout |  |
| D301 | INSUFFICIENT_FUNDS | Insufficient Funds | 51 |
| D302 | TRANSACTION_NOT_ALLOWED | Transaction Not Permitted | 12 / 40 / 57 |
| D303 | AMOUNT_EXCEEDS_MAXIMUM | Maximum Transaction Limit Exceeded |  |
| D304 | INVALID_ACCOUNT | Invalid Payer Identity | 77 |
| D305 | INVALID_ACCOUNT | Invalid Payer Secondary Identity | 78 |
| D306 | INVALID_SOURCE | Source of Fund Not Supported | 47 |
| D307 | INVALID_ACCOUNT | Account is Dormant / Inactive / Invalid | 45 |
| D308 | INVALID_ACCOUNT | Account is Restricted to be Debiting |  |
| D309 | INVALID_ACCOUNT | Account is Mule |  |
| D310 | FREQ_EXCEEDS_MAXIMUM | Consent Exceeded Max Debit Frequency | 66 |
| D311 | REQUEST_REJECTED | MSIC / MCC Not Accepted By Participant | 79 |
| D312 | INVALID_DESTINATION | Account Does Not Exist | 52 / 53 / N3 / 39 |
| D313 | QR_EXPIRED | QR Expired | 48 |
| D314 | MERCHANT_NOT_FOUND | Merchant Inactive / Suspended / Terminated | 49 |
| D315 | INVALID_ACCOUNT | Account is Restricted to be Crediting |  |
| D316 | INVALID_ACCOUNT | Account is flagged for First Ride Risk (FRR) |  |
| D401 | REQUEST_TIMEOUT | Transaction Enquiry Timeout |  |
| D402 | REQUEST_TIMEOUT | Consent Enquiry Timeout |  |
| D403 | SYSTEM_ERROR | Internal Error at Bank System | 29 / 85 |
| D404 | INVALID_ACCOUNT | Invalid Amount | 13 |
| D405 | SIGNATURE_ERROR | Invalid Signature from RPP to RFI | 25 |
| D406 | MESSAGE_VALIDATION_ERROR | Invalid Format from PayNet | 30 |
| D407 | DUPLICATE_REQUEST | Duplicate Transmission Detected at Responding Party | 05 / 94 |
| D408 | QR_VALIDATION_ERROR | QR Validation Failed | 46 |
| D409 | TRANSACTION_NOT_FOUND | Transaction Cannot be Found | 76 |
| D410 | SYSTEM_ERROR | Response Received Too Late | 68 |
| D411 | SYSTEM_ERROR | Unsupported Browser / Operating System |  |
| D501 | INVALID_MERCHANT | Merchant is Prohibited by Bank |  |
| D502 | TRANSACTION_NOT_ALLOWED | Suspicious Transaction |  |
| D503 | INVALID_ACCOUNT | Account is Prohibited by Bank |  |


## Financial Reports API errors

| Code | Message |
|---|---|
| E0001 | The query exceeds the retention period |
| E0003 | Data is not ready |
| E0004 | The given value for $param is invalid: $value |
| E0005 | Invalid date: $value, format should be `yyyy-MM-dd` with leading zero |
| E0006 | $field is required |
| E0007 | Invalid month format: $value, format should be `yyyy-MM` |
| E0009 | This report is unavailable for $fi_id |
| E0010 | Duplicate field or alias is not allowed: %field |
| E0011 | The query exceeds the month retention period |
| E0012 | Unprocessable entity |
| E0014 | Rate limit exceeded |
| E0015 | Query not allowed |
| E0016 | This download link has expired |
| E9999 | Internal server error |

I01 file request statuses: `new`, `processing`, `ready`, `error`. HTTP: 200 OK; 400 missing `X-Fiid`, timestamp or signature, timestamp out of range, invalid signature; 401 missing authorization header; 403 forbidden or wrong FI ID, or E0016 expired link; 500 internal error.

## Common HTTP codes (`/docs/API/http-status-codes`)

200 OK, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not found, 429 Too Many Requests (`{"error": "Quota has been exceeded"}`), 500 Internal server error, 503 Service unavailable.
