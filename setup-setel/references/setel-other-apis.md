# Setel Other APIs

Non-checkout APIs on `https://developer.setel.com` (checked 2026-10-21): code scan wallet transactions, SSO/OAuth, user accounts, user payment methods, loyalty, retail external orders, automated parking and EV charging. Base URLs and auth headers are in `setel-api.md`. Unless noted, calls use `x-api-key` + `x-api-secret`.

## Pay With Setel Code Scan (Wallet Transactions)

Source: `https://developer.setel.com/docs/pay-with-setel-code-scan`, `https://developer.setel.com/reference/wallets`.

Flow: customer shows the Setel code (QR and barcode) in the Setel App; the merchant POS/terminal scans it and calls Create a wallet transaction; a payment intent is returned and goes straight to `processing` then `succeeded` or `failed`.

- Direct capture only (no hold).
- No webhook. Use the returned status, then Retrieve a wallet transaction if unsure.
- Payment method: Setel Wallet only; Setel CardTerus "Not supported yet, coming soon".
- Sandbox test accounts for restricted account, exceeded limits, expired card and insufficient balance are listed as "To be created".

### Create a wallet transaction

`POST /v1/wallet/transactions`. Optional headers `x-interface-id`, `x-interface-type`. Alert: "Please provide a unique `referenceId` to avoid multiple payments and for a successful reconciliation."

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `code` | string | Yes | Access code from the Setel App (the scanned code). |
| `amount` | string | Yes | Ringgit. |
| `currency` | string | Yes | `MYR` only (default). |
| `subMerchant` | object | Yes in schema | `name`, `reference`, `merchantCategoryCode`, `state` required; `outletName` optional. Same `state` list as checkout. |
| `referenceId` | string | No (recommended unique) | Merchant reference. |
| `description` | string | No | |
| `referenceMeta.locationName` | string | No | Parking payments. |
| `referenceMeta.locationAddress` | string | No | |
| `referenceMeta.gps.lat` / `.lng` | string | No | |
| `referenceMeta.duration.startTime` / `.endTime` | string | No | UTC ISO dates. |

Response `201`: `id`, `paymentIntentId`, `paymentIntentStatus`, `code`, `referenceId`, `amount`, `currency`, `description`, `createdAt`, `updatedAt`.

Errors: `400 INVALID_OTAC` ("Invalid otac"), `502 SERVICE_UNAVAILABLE 9300202`.

### Retrieve wallet transactions

- `GET /v1/wallet/transactions/{id}`.
- `GET /v1/wallet/transactions?referenceId=<referenceId>`.

Response `200`: `id`, `paymentIntentId`, `paymentIntentStatus`, `code`, `referenceId`, `amount`, `currency`, `description`, `subMerchant.{name,reference}`, `createdAt`, `updatedAt`.

Refunds and cancels for wallet transactions go through the payment intent endpoints in `setel-api.md`.

### Retrieve a payment code

`GET /v1/code/{codeString}`, headers `x-interface-id`, `x-interface-type` (required). Response: `code` (OTAC, example `SET8293032632522750`), `timeout` (integer, example 180), `createdAt`, `expireAt`, `userHashCode`.

## Single Sign-On (Setel Connect)

Sources: `https://developer.setel.com/docs/introduction-to-single-sign-on`, `.../docs/sso-with-client-secret`, `.../docs/sso-with-pkce`, `.../docs/available-scopes`, `.../docs/error-handling`, `.../reference/issue-oauth-token`, `.../reference/single-sign-on`.

Two OAuth 2.0 authorisation-code flows:

- Client secret: partner has a backend; tokens must never reach the frontend.
- PKCE: no dedicated backend. `code_verifier` is 43 to 128 chars from `A-Z a-z 0-9 - . _ ~`; `code_challenge = base64url(SHA-256(code_verifier))`; `code_challenge_method=S256`.

Authorisation URL (host shown only as `[setel-connect-url]`; Setel provides it):

```text
https://[setel-connect-url]/authorize?response_type=code&client_id=<CLIENT_ID>&redirect_uri=<REDIRECT_URI>&scope=user_info_read_scope&state=<STATE>[&code_challenge=<CHALLENGE>&code_challenge_method=S256]
```

- `client_id` "is not the same as your merchant ID".
- Scope: `user_info_read_scope` (user name and email).
- Verify `state` on return: `<redirect_uri>?code=<CODE>&state=<STATE>`.
- On failure the redirect carries `error` and maybe `error_description` (for your logs only; show your own message). Values: `access_denied`, `invalid_request`, `unauthorized_client`, `unsupported_response_type`, `invalid_scope`, `server_error`, `temporarily_unavailable`. Unknown `client_id` means no redirect.

### Issue OAuth token

`POST /v1/oauth/token`, `application/x-www-form-urlencoded` (OpenAPI server `https://api-sandbox.setel.my`, no API-key security).

| Field | Required | Notes |
| --- | --- | --- |
| `grant_type` | Yes | `authorization_code` (default) or `refresh_token`. |
| `client_id` | Yes | |
| `client_secret` | Yes in schema | PKCE guide sends `code_verifier` instead. |
| `code` | For `authorization_code` | |
| `redirect_uri` | For `authorization_code` | |
| `refresh_token` | For `refresh_token` | |

Response `200` (reference): `access_token`, `refresh_token`, `expires_in`, `token_type: "Bearer"`. The guides instead show camelCase JSON (`grantType`, `clientId`, `clientSecret`, `redirectUri`) and responses `accessToken`, `refreshToken`, `expiresIn: 1800`, `refreshExpiresIn: 336000`. Confirm the accepted naming in sandbox.

The access token is then sent as the `access-token` header on user-scoped endpoints.

## Accounts

Source: `https://developer.setel.com/reference/accounts` and child pages. Headers `x-interface-id`, `x-interface-type` are required on most.

| Method | Path | Body / headers | Response |
| --- | --- | --- | --- |
| POST | `/v1/account/unverified` | `phoneNumber` (e.g. `+60123456789`) | `200` unverified `{}`; `207` existing: `userId`, `accountVerified`, `firstTimeRegistration`, `hasPasscode` |
| POST | `/v1/account/passcode` | `access-token`; `passcode` | `201` `userId` |
| PATCH | `/v1/account/passcode` | `access-token`, `x-source`; `passcode` | `200` `userId` |
| POST | `/v1/account/sessions/passcode` | `phoneNumber`, `passcode` | `201` `access_token`, `expire_in`, `distinctId` |
| POST | `/v1/account/sessions/otp/verification` | query `voice`, `resend` (bool); `phoneNumber` | `201` `{}` (sends SMS or voice OTP) |
| PUT | `/v1/account/sessions/otp/verification` | `phoneNumber`, `otp` | `200` `accountState`, `otpToken`, `phoneToken` |
| POST | `/v1/account/sessions/otp` | headers `otp-token`, `phone-token`; `phoneNumber` | `200` `accessToken` (+ more) |
| DELETE | `/v1/account/sessions/current` | `access-token` | `200` `{ "message": "Logout successfully" }` |

## User Payment Methods

- `GET /v1/payments/payment-methods` (`access-token`). Array items: `id`, `isLastUsed`, `userId`, `family` (`wallet`, `card`, `smartpay`), `type` (`wallet_setel`, `smartpay`, `card_visa`, ...), `card`.
  - Visa/Mastercard `card`: `lastFourDigits`, `brand`, `issuing`, `cardId`.
  - SmartPay `card`: `cardNumber`, `cardType`, `expiryDate`, `mesraPoints`.
  - Mesra card: `cardNumber`, `pointBalance`, `type` (company or consumer), `status` (`issued`, `active`, `frozen`, `temporarilyFrozen`), `issuedBy` (`setel`, `petronas`), `isActive`.
- `GET /v1/payments/payment-methods/{id}/balance` (`access-token`). Response `id`, `balance` (integer, example 300; unit not stated).

These feed the Finish checkout session call (`paymentMethodId`, `userId`).

## Loyalty (Mesra)

| Method | Path | Inputs | Notes |
| --- | --- | --- | --- |
| PUT | `/v1/loyalty/cards` | `access-token`, interface headers; query `idType` (`ic number`, `old ic number`, `passport number`), `idNumber` | Activate Mesra card |
| POST | `/v1/loyalty/cards/switch-to-physical-card` | `access-token`, interface headers; body `idType`, `idNumber`, `cardNumber` | Link physical card |
| GET | `/v1/loyalty/cards` | `access-token` | `cardNumber`, `pointBalance`, `userId`, `type`, `status`, `issuedBy`, `isActive`, `isPointEarningEnabled`, `isPointRedemptionEnabled` |
| GET | `/v1/loyalty/members/by-card/{cardNumber}` | `x-api-key`, `x-api-secret`, interface headers | Card details by number |

## Retail

- `GET /v1/retail/fuel-order-options` (`access-token`): `presetAmounts[]`, `presetVolumes[]`, `fullTankAmount`, `fullTankVolume`, `minAmount`, `maxAmount`, `minVolume` (each `{ quantity }`).
- `POST /v1/retail/external-orders`: lets users earn Mesra points without paying through Setel. Required `userId`, `referenceId`, `amount` (string), `currency` (`MYR`), `taxInvoiceNumber`, `paymentMethod` (`family`, `type` required, `brand`; values provided by Setel), `products[]` (`unitPrice`, `quantity`, `loyaltyCategoryCode` 4 digits). Optional `isEligibleForPoint`, `referenceMeta`. Physical-card redemptions and PETRONAS non-PDRM SmartPay payments are not eligible (`isEligibleForPoint: false`).
- `GET /v1/retail/external-orders?referenceId={referenceId}`: returns the order with `id`.

## Automated Parking

Sources: `https://developer.setel.com/docs/how-automated-parking-works`, `.../docs/automated-parking-webhooks`, `.../docs/parking-location-deployment`, `/reference/parking-sessions` pages. For License Plate Reader sites whose locations are imported into Setel; users need LPN set up in the Setel app. Check season-parking customers before creating a session.

| Method | Path | Body | Response |
| --- | --- | --- | --- |
| POST | `/b2c/api/parking/sessions` | `plateNumber`, `entryLane`, `locationCode` (req); `metadata.requestTimestamp` (ISO 8601 entry time) | `sessionId`, `locationCode`, `metadata.responseTimestamp` |
| GET | `/b2c/api/parking/sessions/{sessionId}` | | `session.{id,status,plateNumber,entryLane,exitLane,checkinAt,checkoutAt,finalFees,locationName,locationAddress,locationGpsCoordinates}` |
| PUT | `/b2c/api/parking/sessions/{sessionId}/close` | `finalFees` (number, max 2 decimals), `exitLane` (req); `metadata.requestTimestamp` | `sessionId`, `locationCode`, `metadata` |
| PUT | `/b2c/api/parking/sessions/{sessionId}/charge` | `metadata.requestTimestamp` | adds `metadata.approvalCode` |
| PUT | `/b2c/api/parking/sessions/{sessionId}/end` | `finalFees`, `exitLane` | Deprecated v1.0 ("remain operational until further notice") |
| PUT | `/b2c/api/parking/sessions/{sessionId}` | `remarks` enum: "Session is voided because the customer didn't pay for parking during exit using Setel", "Payment in advance: Season pass purchased", "Payment in advance: Kiosk/Device handheld/Web portal", "Payment in advance: Validation at counter" | `session` with `status: "VOIDED"` |
| POST | `/api/parking/events` | interface headers; `type` (`STARTED`/`ENDED`), `locationName`, `code` (user QR id) req; `requiredBalance` (2 dp) | `{}` |

APIs the parking operator hosts for Setel (headers defined by the vendor; query `session_id`):

- Fee query (required): returns `data.sessionId`, `data.licensePlate`, `data.amount` (MYR, rounded up to 2 dp).
- Session status check (optional): `data.entryTime`, `exitTime`, `voidTime`, `sessionStatus` (`active`, `exited`, `voided`, `unknown`).
- Void parking session (optional): `body.responseInfo.{code (0 = success), status, codeDescription}`, `body.response.{sessionId, licensePlate, entryTime, exitTime}`.

Deployment: integration, location onboarding (location codes; flag multi-building sites), UAT in staging then production smoke test.

## EV Charging

Sources: `https://developer.setel.com/docs/how-ev-charging-works` and child pages.

Setel endpoints (query `vendorName`):

- `POST /v1/api/ev-charging/webhook/{vendorName}/events`: `sessionId`, `payload.{ocppStatus, locationId, chargingPointId, connectorId}` (all required).
- `POST /v1/api/ev-charging/webhook/{vendorName}/payment`: `sessionId`, `payload.{completedUsage (kWh), completedDuration (s), completedAmount (0.00), billingRate}` (all required).

Vendor-hosted APIs: get charging location (station, charging points, connectors with status and price), create charging session (`chargingPointId`, `connectorId` -> `sessionId`), retrieve charging session (`status`, `completedAmount` RM, `completedUsage` kWh, `completedDuration` s, `billingRate`, timestamps, ids), end charging session (`sessionId`).
