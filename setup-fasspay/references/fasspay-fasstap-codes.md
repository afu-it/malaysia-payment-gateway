# Fasstap SDK Status And Error Codes

Every code block on the Fasstap SDK status code page, as code, source (SDK or Backend) and root cause. Summary tables (transaction results, UI events, transaction status) are in `fasspay-in-person.md`.

Source: `https://developers.softspace.com.my/guides/fasstap/development/sdk-integration/android/latest/status-code` (checked 2026-10-21). Generated from the page data and condensed; descriptions over about 110 characters are cut with "...". Req: Y = required, C = conditional, blank = optional or not stated.
| Code | Source | Description / root cause |
|---|---|---|
| 9984 | SDK | Missing required parameters from login. |
| 1000 | SDK | Internet is not detected. |
| 1001 | SDK | Service is currently unavailable. |
| -1004 | SDK | Service is currently unavailable. |
| 9010 | Backend | Invalid service name/ version |
| 9011 | Backend | Method invocation error |
| 10001 | Backend | Service is currently unavailable. |
| 2002 | Backend | Session is expired or kicked out from others. |
| 3011 | Backend | Merchant has exceeded a maximum number of attempts. |
| 5010 | Backend | Mobile user account login is invalid or the SSO token is invalidated. |
| 14015 | Backend | Mobile user's sso token has been revoked or mobile user is different from the previous. |
| 100 | Backend |  |
| 7056 | SDK | Transaction failed due to “Bad Tap” in the midst of processing. |
| 7059 | SDK | Transaction failed due to transaction amount above CVM limit and card type is not supported for PIN. |
| 9012 | SDK | Transaction has been declined, card is declined by kernel. |
| 7501 | SDK | Exception during GPO command (Timeout/Tag lost). |
| 7020 | SDK | Transaction trying to send online is not successful, possible missing key or didn't perform login. |
| 7502 | SDK | Card locked response during GPO command (6F00). |
| 2** | Backend | - |
| 5111 | Backend | Total monthly transaction limit has exceeded. |
| 5112 | Backend | Total per transaction limit is exceeded. |
| 5120 | Backend | Batch is not ready. There is one or more processing / unsuccessful settlement. |
| 8090 | Backend | Host timeout or return an empty response. |
| 8091 | Backend | Unable to establish connection to host, host is possibly down. |
| 8092 | Backend | Connection to host has timeout from payment server side. |
| 8093 | Backend | Batch upload is not successfully completed based on host's response. |
| 9002 | Backend | Customer's card is expired. |
| 9003 | Backend | Card used is not effective yet, not activated or suspended. |
| 9004 | SDK | - |
| 3012 | Backend | Void / Settle buttons are disabled for users which do not have permission to void or settle transactions. |
| 9022 | Backend | This transaction type is not allowed to void. |
| 19000 | Backend | This transaction type is not allowed to void. |
| 5113 | Backend | Customer entered an invalid phone number for e-receipt. |
| 5114 | Backend | Customer entered an invalid email address format for e-receipt. |
| 5115 | Backend | Email / SMS e-receipt sending service is currently unavailable. |
| 10x | Backend |  |
| 100 | Backend |  |
| 9500 | SDK | QR Payment is not enabled for the account. |
| 9501 | SDK | QR Payment is not enabled for the account. |
| 19000 | SDK | QR transaction request failed. |
| 0 | SDK | Operation carried out successfully |
| -1 | SDK | It usually occurs during development phase, when attestation library isn't being initialized |
| -2 | SDK | The permisison(s) required by the attestation library to carry out operations is(are) not granted by the user |
| 1 | SDK | Unexpected error that happens within attestation library |
| 2 | SDK | Error occurs while carrying out attestation operations from attestation backend |
| 3 | SDK | Data needed are not valid, usually occurs during development |
| 4 | SDK | Usually it occurs when reader is malfunctioned |
| 5 | SDK | Error when processing data in attestation library. Usually occurs during development phase |
| 6 | SDK | Error when processing data in attestation library. Usually occurs during development phase |
| 7 | SDK | Error when processing data in attestation library. Usually occurs during development phase |
| 8 | SDK | Execution of a certain operation that is not permitted |
| 9 | SDK | When device's memory is less than the attestation's library needs |
| 10 | SDK | This error will not be raised |
| 11 | SDK | Google Play Service is not available |
| 12 | SDK | Google Safetynet is not available |
| 256 | SDK | This error will not be raised |
| 257 | SDK | Needed file is missing or corrupted |
| 258 | SDK | Network down |
| 259 | SDK | Attestation server returning unexpected response |
| 513 | SDK | Security key is missing |
| 514 | SDK | Security key is corrupted |
| 515 | SDK | Security key is corrupted |
| 516 | SDK | Cryptographic operation carried out unsuccessfully |
| 517 | SDK | Certificate of the attestation server in the device has expired |
| 518 | SDK | Certificate file is missing. Probably due to data file corrupted |
| 519 | SDK | Certificate isn't recognized. This usually happens when the app isn't installed from a legit app store |
| 520 | SDK | The device has failed attestation. Usually happens when device is compromised/ jailbroken/ rooted |
| 521 | SDK | The reader has failed attestation. Usually occurs when the reader is not recognized, or malfunctioned |
| 522 | SDK | The reader has failed to activate. Usually occurs when the reader is not recognized, or malfunctioned |
| 523 | SDK | Provisioning operation has failed. Usually occurs when the data is corrupted while communicating with backend |
| 524 | SDK | When the key exchange operation carried out unsuccessfully. Usually occurs when the data is corrupted while co |
| 769 | SDK | Trying to login as a different user than previously logged in user |
| 770 | SDK | Trying to login as a different reader than previously logged in reader |
| 771 | SDK | Trying to use a reader which firmware is incorrect |
| 772 | SDK | Trying to use a reader which key is incorrect |
| 520001 | SDK | COTS's time is not synchronized with server during attestation |
| 520002 | SDK | Security patch version is outdated/non-compliance |
| 520003 | SDK | Trying to use an application that downloaded from an unknown source |
| 520004 | SDK | Trying to use an application or device that's fails play integrity verification |
| 520101 | SDK | The device developer mode option is enabled |
| 520102 | SDK | The device ADB is enabled |
| 520103 | SDK | Rooted device detected |
| 520104 | SDK | The device USB debugging is enabled |
| 520105 | SDK | The app run on a emulator |
| 520106 | SDK | Malicious hooking activity, or hooking app in the background |
| -1001 | SDK | Connection request timeout |
| -1003 | SDK | Connection error to payment server - cannot find from path link. |
| -1004 | SDK | Service is currently unavailable |
| -1005 | SDK | Connection network to payment server is dropped off |
| -1006 | SDK | Certificate pinning |
| -1202 | SDK | Secure certification of payment server is untrusted |
| 1001 | SDK | This error code is only happened during payment process when payment timeout of 70s is triggered |
| 1002 | SDK | Unrecoverable error occur from payment server side. Need payment log for further troubleshoot |
| 1006 | SDK | Certificate of SSL has expired or got issue in cert pinning |
| 2002 | SDK | Device's session has expired, possible cause include server restart (manually triggered by dev) |
| 3032 | SDK | Faulty reader due to key injection issue, i.e. reader didn't perform/pass key injection test |
| 6000 | SDK | The reader has been tampered and will need to be returned and replaced with a new reader |
| 6001 | SDK | (Not specified) |
| 7004 | SDK | The bank has returned the message 'Transaction Declined' when processing the transaction |
| 7005 | SDK | The bank has returned the message 'Transaction Failed' when processing the transaction |
| 7028 | SDK | The transaction has timed out. The customer is advised to remove their card and try again |
| 7024 | SDK | Cancel transaction while entering the Cardholder PIN |
| 7041 | SDK | Happen during contactless transaction: when the card tapped and removed immediately from the FLite reader; Pay |
| 7054 | SDK | FLite reader activation processes failed during first time connecting / new device (TBC) |
| 7056 | SDK | The transaction has failed for an unknown reason |
| 7060 | SDK | Card scheme not supported or min amount is not met for the campaign |
| 9803 | SDK | The POS application has tried to initiate a payment with an invalid payment amount. Ensure the value is not 0 |
| 9804 | SDK | The POS application has tried to void the payment |
| 9805 | SDK | The POS application has tried to initiate an action that is not supported or not enabled on this account |
| 9812 | SDK | An error occurred when trying to complete a quickchip payment |
| 9987 | SDK | The POS application has attempted to initiate a quickchip payment but quickchip is disabled for this merchant |
| 9990 | SDK | Your device is detected to be rooted or jailbroken |
| 9991 | SDK | When route from 3rd party app to MPOS app and detects MPOS app does not have any user activated before |
| 9992 | SDK | SDK unexcepted exception |
| 9993 | SDK | Get KSN Fail, this device is malfunction or is not yet injected |
| 9994 | SDK | Unexpected error occur from mobile app side |
| 9996 | SDK | Unexpected error occur from payment server side. Possible that payment server does not response data to mobile |
| 9997 | SDK | Either reader or card is accidentally or purposely being loose connection/disconnected from phone |
| 9998 | SDK | Invalid hash value field return from payment server. Need to debug via payment log |
| 9999 | SDK | Invalid session value field return from payment server. Need to debug via payment log |
| 2001 | SDK | Initialization fault |
| 2002 | SDK | Device's session has expired, possible cause include server restart (manually triggered by dev) |
| 2012 | SDK | Mobile user is trying to access in invalid page in sales history |
| 3011 | SDK | Mobile user account is blocked/ suspended. User needs to reinstate the account via bank portal or merchant por |
| 3012 | SDK | Rare scenario: VOID or Settle buttons are disabled if user doesn't have permission but just in case, we prepar |
| 3013 | SDK | Merchant account has been suspended. User need to reactivate account via bank portal to accept transaction |
| 3014 | SDK | Merchant account has been terminated |
| 3020 | SDK | Merchant's phone/device does not match |
| 3021 | SDK | Reader not belongs to Bank |
| 3022 | SDK | This error code is obsoleted, if indeed receive this error code, please consult IT |
| 3023 | SDK | Phone ID sent from mobile apps is not the same as per registered |
| 3029 | SDK | - |
| 3030 | SDK | Mobile user attempts to login using a reader that doesn't belong to the company/merchant that the current mobi |
| 3031 | SDK | Reader suspended from portal |
| 3032 | SDK | Key injection is not performed for the current reader that mobile user is using. Suggest to redo key injection |
| 3033 | SDK | Key injection failed |
| 5009 | SDK | User tries to login into key injection app using an invalid user id. The login information associated with thi |
| 5010 | SDK | Invalid user login or mobile user, or mobile user or company has been suspended |
| 5011 | SDK | Mobile user enters an invalid PIN |
| 5012 | SDK | User cannot re-use any of the last 5 PIN that was used before |
| 5013 | SDK | Activation fails. Please check activation code |
| 5014 | SDK | Last attempt for user to login before mPOS suspends the mobile user |
| 5015 | SDK | New PIN is vulnerable. It is sequential, repetitive or commonly-used |
| 5016 | SDK | First Time Activation not performed |
| 5017 | SDK | Entered invalid business registration number for first time registration |
| 5018 | SDK | Entered invalid email address for first time activation |
| 5019 | SDK | User ID is activated |
| 5020 | SDK | There is a newer version of the mpos app in appstore or Google Play. User will be redirected to the correct ap |
| 5021 | SDK | (Not specified) |
| 5023 | SDK | MCCS Blocked |
| 5024 | SDK | User account is currently logged in on another device. User will hit session expire on the other device once c |
| 5025 | SDK | Login expired by new device |
| 5026 | SDK | (Not specified) |
| 5110 | SDK | This is a general error indicating the transaction cannot continue and the user should be instructed to try ag |
| 5111 | SDK | Total monthly transaction limit is exceeded. Bank may either increase the limit or disable the checking (defau |
| 5112 | SDK | Total per transaction limit is exceeded. Bank may either increase the per transaction limit or disable the che |
| 5113 | SDK | User entered an invalid phone number. Phone number is not between 10 to 11 digit |
| 5114 | SDK | Invalid email address format |
| 5115 | SDK | Email / SMS service is currently down |
| 5116 | SDK | Total daily transaction limit is exceeded. Bank may either increase the limit or disable the checking (default |
| 5117 | SDK | Daily transaction count is exceeded. Bank may either increase the limit or disable the checking (Default to ma |
| 5118 | SDK | Total monthly transaction count is exceeded. Bank may either increase the count or disable the checking (defau |
| 5120 | SDK | Batch is not ready. There is either one or more processing/ unsuccessful settlement. Please either settle it m |
| 5555 | SDK | The system is currently down |
| 7056 | SDK | Transaction amount exceeded limit |
| 8090 | SDK | Host timeout or return empty response |
| 8091 | SDK | Unable to establish connection to host. Possibly host is down |
| 8092 | SDK | Connection to host has timeout from payment server side |
| 8093 | SDK | Batch upload is not successfully completed based on host's response |
| 8094 | SDK | Unable to decrypt host response |
| 8095 | SDK | Unable to receive any response from host (Host communication using API) |
| 8096 | SDK | Host returns 404 not found (Host communication using API) |
| 8101 | SDK | Amount is not populated/null in data sent by mobile (SDK only) |
| 8102 | SDK | Reader unable to read magstripe track2 data. Magstripe data was invalid. Either the card or the reader was fau |
| 8103 | SDK | Magstripe is not allowed for this card. Please use chip transaction instead |
| 8104 | SDK | KSN is null in data sent by mobile |
| 14111 | SDK | Exceeded void window of 25 mins |
| 8201 | SDK | App/terminal when ask blocked bin from server, but didnt provide revision field |
| 8202 | SDK | Revision received is less than 0 or higher than latest file revision |
| 8203 | SDK | No bin file found in system |
| 8204 | SDK | Transaction amount exceed |
| 8205 | SDK | The transaction cannot be found, for example when performing void |
| 8206 | SDK | Duplicate transaction reference no, All reference no should be unique |
| 8207 | SDK | Error invalid merchant data |
| 14010 | SDK | Your SSO token has been revoked |
| 14011 | SDK | You are not permitted to perform SSO operations |
| 14012 | SDK | Invalid SSO token |
| 14013 | SDK | You are not permitted to perform any operations |
| 14014 | SDK | Please perform activation on PlugNPay |
| 14015 | SDK | Invalid Company SSO Identifier |
| 14017 | SDK | Void denied. This transaction has been voided or pending void |
| 14100 | SDK | User does not have reset pin session |
| 14101 | SDK | Company country and Mobile User Country not match |
| 14110 | SDK | The activation code is expired. Please request for a new activation code. |
| 14111 | SDK | Invalid OTP. Please fill in with correct Authentication Code. |
| 14112 | SDK | OTP has expired. Please click on "Resend OTP" for your new Authentication Code. |
| 15001 | SDK | Upload ID exist with a CONFIRMED status |
| 15002 | SDK | Upload ID exist with an unknown status |
| 15003 | SDK | Error total comparison |
| 15004 | SDK | Offline unknown status |
| 15005 | SDK | Transaction auth type unknown |
| 15006 | SDK | Transaction status unknown |
| 15007 | SDK | Error processing contactless transaction's transaction date |
| 15008 | SDK | Error processing contactless transaction's transaction time |
| 15009 | SDK | Transaction transaction null |
| 15010 | SDK | Reader serial no., total transaction count/amount cannot be null |
| 15011 | SDK | MID does not exist |
| 15012 | SDK | No TID available assigned to the MID |
| 15013 | SDK | Track2 cannot be null |
| 16001 | SDK | Detected location of transaction is out of allowed boundaries. |
| 16002 | SDK | Transaction not exist |
| 16003 | SDK | Receipt not exist |
| 16004 | SDK | QR token expired |
| 17001 | SDK | (Not specified) |
| 17002 | SDK | (Not specified) |
| 17003 | SDK | (Not specified) |
| 17004 | SDK | (Not specified) |
| 17005 | SDK | Error during initialization |
| 17006 | SDK | There are still pending unresolved transaction |
| 17007 | SDK | Country Code passed in the API is not in uppercase |
| 18000 | SDK | Inactive service code |
| 18001 | SDK | Unathorized service access |
| 18002 | SDK | (Not specified) |
| 18003 | SDK | Unified payment not supported |
| 18004 | SDK | Fasstap reader acceptance not enabled. |
| 18005 | SDK | Lighthouse reader acceptance not enabled |
| 18006 | SDK | K9 reader acceptance not enabled |
| 19001 | SDK | Please contact SOFT SPACE if this problem persists (SPoC caPEM validation error) |
