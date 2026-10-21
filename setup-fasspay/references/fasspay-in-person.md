# Fasspay In-Person (Soft Space Developer Portal)

Use this reference for card-present integrations documented on the Soft Space Developer Portal, `https://developers.softspace.com.my/` (checked 2026-10-21). Field tables are in:

- `fasspay-fasstap-sdk.md`: Fasstap SDK API (SSMPOSSDK, Transaction, builders, TransactionOutcome).
- `fasspay-app-to-app.md`: Fasstap App-to-App request and response objects per action.
- `fasspay-smart-terminal.md`: Smart Terminal SDK, ReaderTag keys, request objects.
- `fasspay-fasstap-codes.md`: Fasstap SDK status and error codes.

These are separate from the online e-commerce API in `fasspay-api.md`. Status codes differ between surfaces.

## Access Model

The portal is public, but building requires material that Soft Space sends privately:

- Fasstap SDK: PGP key exchange, your Application ID and signing keystore public key sent to Soft Space under PGP, SDK configuration (configuration token, MPoC activation token, unique ID, developer ID) returned under PGP, and Maven credentials for `https://nexus.softspace.com.my/nexus/content/repositories/ssmpocsdk` and `.../ssmpossdk`.
- App-to-App: a ZIP package containing `ssmobilethirdpartyintegration.jar`.
- Smart Terminal: the USB Serial SDK AAR/JAR.
- Web API: a Bearer token. The host for `/api/processRequest` is not shown on the portal.

Contact: `https://developers.softspace.com.my/support/contact-us`, `hello@softspace.com.my`, +603 7494 1222.

## Fasstap SDK (tap on phone)

Source: `/guides/fasstap/development/sdk-integration/android/latest/*`, `/reference/fasstap/sdk-integration/android/latest/sdk-api`.

- Accepts contactless cards on an NFC Android phone, no external reader. Kernels: JCB, Visa, Mastercard, UPI, Diners Club, Discover, AMEX, EFTPOS, MADA, NAPAS, MyDebit.
- Supported OS: Android 12 to 16. `minSdkVersion` 31 to 36. JDK 17. `sourceCompatibility`/`targetCompatibility` Java 11. Device needs NFC, online access, Google Mobile Services with CTS.
- Dependency: `my.com.softspace.ssmpossdk:ssmobile-mpos-sdk:4.0.0.0` from the Soft Space Maven repos.
- Manifest: `android:extractNativeLibs="true"`, `android:resizeableActivity="false"`; `tools:overrideLibrary` list if your minSdk is lower (see guide).
- App must block presentation displays (`Display.FLAG_PRESENTATION`) and set `FLAG_SECURE`.
- Register a Google Android Device Verification API key. Distribute only through Google Play. Signing keys must sit in an HSM approved to PCI HSM or FIPS 140-2 Level 3, with dual control on release.
- Do not store end-user phone, email or other correlatable data; clear buffers after use. Use TLS and encrypt payloads; DUKPT for cardholder data and PIN block is advised for third-party payment servers; certificate pinning advised.
- Attestation runs every 5 minutes, on install, and before each transaction.
- Flow: `SSMPOSSDK.init(context, config, InitCallback)`, then `SSMPOSSDK.getInstance().getTransaction().startTransaction(activity, MPOSTransactionParams, TransactionEvents)`. Results arrive in `onTransactionResult` and UI events in `onTransactionUIEvent`. Tap and PIN screens are handled by the MPoC SDK.
- Transaction API: `startTransaction`, `abortTransaction`, `startQRTransaction`, `abortQRTransaction`, `voidTransaction`, `refundTransaction`, `performSettlement`, `refreshToken`, `sendReceipt`, `uploadSignature`, `getTransactionStatus`, `startOfflineTransaction`, `getOfflineTransaction`, `submitOfflineTransaction`.

Transaction result codes (onTransactionResult): `0` successful, `7004` declined, `7005` failed, `7006` no app, `7007` failed allow fallback (treat as failed), `7008` card expired, `7020` online failed, `7024` cancelled, `7028` timeout, `7054` require CDCVM, `7055` end application (treat as decline), `7056` try again, `7059` PIN not supported, `7060` card not supported, `7063` no key, `7064` offline not permitted, `7065` attestation not done, `7066` offline time allowance exceeded, `7067` card not matched, `7068` manual PAN not permitted, `7069` invalid SCA token.

UI event codes: `71` present card, `72` card presented, `74` present card timeout, `23` card read OK, `24` card read error, `25` card read retry, `65` enter PIN, `66` cancel PIN, `67` PIN bypass, `68` PIN timeout, `69` PIN entered, `73` authorising, `70` request signature, `80` required CDCVM, `81` processing.

Transaction status (TransactionOutcome): `100` APPROVED, `101` REVERSED, `102` VOIDED, `103` PENDING_SIGNATURE, `104` SETTLED, `105` PENDING_SETTLEMENT, `106` REFUNDED, `127` PENDING_APPROVAL, `251` DECLINED. Host errors come in `statusMessage`.

Changelog: 14 Aug 2025 (MPoC version, offline transactions, removed `secretKey`/`accessKey`/attestation config fields, `setCardRequiredForRefund` renamed `setCardRequired`, `init` gained callbacks).

## Fasstap App-to-App

Source: `/guides/fasstap/development/app-to-app-integration/*`, `/reference/fasstap/app-to-app-integration/android/latest/sdk-api`.

- Your app calls `ThirdPartyIntegration.requestMPOSIntegration(context, ThirdPartyIntegrationRequestVO)`; the Fasstap app talks to the MPOS payment server and returns `ThirdPartyIntegrationResponseVO` via `processResponseMPOSIntegration`.
- Setup: put `ssmobilethirdpartyintegration.jar` in `libs/`, `implementation files('libs/ssmobilethirdpartyintegration.jar')`, and add a `<queries><package android:name="..."/></queries>` entry for the Fasstap app package.
- Integration types: Pinless SSO (`IntegrationType.IntegrationTypeSSO`, no manual token step) or the deprecated non-pinless SSO (user enters MUID and PIN).
- ActionType values: `Registration`, `GetToken`, `Payment`, `SalesHistory`, `GetTransactionStatus`, `VoidPayment`, `CreditSettlement`, `Refund`, `RecurringPayment`, `QRPayment`, `QRRefund`, `QRCheckStatus`, `InstallmentPayment`, `QuickChip`, `QuickChipCompletion`, `Undefined` (falls back to Payment).
- Required request fields: `mposUrl` (Fasstap app package), `returnUrl` (your package), `developerId`, `thirdPartyUserID`, `returnIntentName`, `integrationType`. `amount` and `referenceNo` are mandatory for payment.
- Success codes in module flows: payment, installment, recurring, quick chip and QR payment return `100`; refund returns `106`; registration, void, QR refund and QR check status return `10` as written on the portal.
- Status codes: `100` APPROVED, `101` REVERSED, `102` VOIDED, `103` PENDING_SIGNATURE, `104` SETTLED, `105` PENDING_SETTLEMENT, `106` REFUNDED, `0` CANCELLED. Error codes `201` to `296` mirror ISO 8583 host responses (for example `205` do not honor, `251` insufficient funds, `254` expired card, `255` incorrect PIN), and `9991` MPOS not activated.
- Changelog: initial release 02 July 2025.

## Smart Terminal SDK

Source: `/guides/smart-terminal/*`, `/reference/smart-terminal/android/latest/sdk-api`.

- Android POS (Android 7.0, API 24 or higher) talks to a Soft Space Android payment terminal over USB OTG. Add the AAR/JAR to `libs/`, `implementation fileTree(dir: 'libs', include: ['*.jar','*.aar'])`, `android.permission.USB_PERMISSION`, `android.hardware.usb.host`.
- Entry: `val sdk = SoftSpaceSDK()`, then `sdk.usbReader` (IReader: `connect`, `sendRequest`, `receiveResponse`, `observeConnection`, `clear`) and `sdk.codec` (ICodecManager: `encodeRequest`, `decodeResponse`, `encodeRequestAsString`, `formatResponseAsString`, `verifyResponse`).
- Flow: build a request object, `encodeRequest`, `sendRequest`, `receiveResponse`, `verifyResponse`, `decodeResponse` into `Map<ReaderTag, String>`.
- Modules: Card Payment (Sale), QR Payment, QR Inquiry, Void, Cancel, Transaction Status, Settlement Info, Settlement, Send E-Receipt, Membership Info and Top Up (Aeon Member Plus card only), Card Refund (card or Aeon QR), QR Refund, Bill Payment (cash or MyDebit for utility bills).
- Response codes: `100` approved; `201`-`299` host declines; `300` processing, `301` insert/swipe/tap, `302` enter PIN, `303` app selection, `304` authorizing, `305` scan QR, `306` finalize required, `307` draw signature, `308` enter phone/email, `309` inquiry required, `310` QR expired, `320` enter bill details, `321` select payment type; `400` processing error, `401` timeout, `500` terminal not ready, `501` ready, `502` maintenance; `600`-`618` coupon (Superks); `900` invalid developer ID, `901` MPOS not activated, `902` login required, `903` action not permitted, `904` service not available, `905` invalid email, `906` invalid phone; `000` cancelled, `001` broken pipe, `002` invalid format, `003` invalid data length, `004` invalid plan code, `18004` VAT not permitted (SCB), `10001` unexpected server exception.
- Changelog: v1.0, 02 December 2020.

## Provisioning Web API (Fasstap)

Source: `/reference/fasstap/sdk-integration/android/latest/web-api` (v4.0.0.0).

All operations: `POST /api/processRequest`, header `Authorization: Bearer <token>`, JSON body with `apiServiceName`. Host: Not documented on the portal. Responses carry `status` (for example `Success`), `message`, and operation fields.

| apiServiceName | Purpose | Request fields (all required) | Response extras |
|---|---|---|---|
| `CREATE_COMPANY` | Create company with MIDs | `companyDetails.businessName` {1,50}, `businessShortName` {1,25}, `businessRegistrationAddress` {1,50}, `postcode` \d{4}, `city`, `region` [A-Za-z0-9 ]{2,50}, `mcc` \d{4}, `lcrEnabled` bool, `lcrMinThreshold` 0-999999998, `lcrMaxThreshold` > min and <= 999999999 (EMV format), `merchantDetails[].mid` \d{15}, `merchantDetails[].dba` {1,25} | `uid` |
| `UPDATE_COMPANY` | Update company | `uid`, same `companyDetails` fields (`businessName` not editable, validation only) without `merchantDetails` | `uid` |
| `VIEW_COMPANY` | Read company | `uid`, `companyDetails.businessName` | `companyDetails` incl. `merchantDetails` |
| `REGISTER_MID` | Add MIDs | `uid`, `companyDetails.businessName`, `companyDetails.merchantDetails[].mid` \d{15}, `dba` {1,25} | |
| `REGISTER_TID` | Register terminals | `terminalDetails[].mid`, `terminalDetails[].tid` \d{8} | `uid` list per TID |
| `VIEW_TID` | Read terminal | `uid`, `terminalDetails.tid` | `terminalDetails.mid`, `tid` |
| `REGISTER_MOBILE_USER` | Create mobile user on a TID | `mobileUserDetails.mobileUserID` \w{3,20}, `terminalDetails.mid`, `terminalDetails.tid` | `uid`, `tempPin`, `mobileUserDetails.userStatus` |
| `VIEW_MOBILE_USER` | Read mobile user | `uid`, `mobileUserDetails.mobileUserID` | `mobileUserDetails`, `terminalDetails` |
| `UPDATE_MOBILE_USER_STATUS` | Activate or suspend | `uid`, `mobileUserDetails.mobileUserID`, `mobileUserDetails.userStatus` (`Active`, `Suspended`) | `tempPin` when moving Suspended to Active |
| `RESET_PIN` | Reset mobile user PIN | `uid`, `mobileUserDetails.mobileUserID` | `tempPin` |

Treat `tempPin` as a secret; deliver it to the user out of band and never log it.
