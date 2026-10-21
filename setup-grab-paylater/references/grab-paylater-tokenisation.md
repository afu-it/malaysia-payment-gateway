# GrabPay Tokenised Payments v4 (recurring, saved wallet)

Use for repeat purchases and subscriptions: the shopper binds their Grab account once, then the merchant charges the stored token. Source: `https://developer.grab.com/docs/tokenisation-v4` and its 29 sub-pages (checked 2026-10-21). Account enablement is manual: "Talk to your Account Manager to enable this."

## Facts

- Environment listed: Production `https://partner-api.grab.com` only ("test your setup in the production environment"). Some samples still show the staging host.
- Scopes: `openid` + `payment.recurring_charge`.
- Countries for bind: `SG`, `MY`, `PH`. Charge currency: `SGD` and `MYR` only.
- Token (access_token from bind) validity: 10 years.
- Same partner HMAC and `X-GID-AUX-POP` algorithms as OTC (see `grab-paylater-api.md`). v4 docs JS sample hashes an empty string for GET bodies; confirm with Grab if a GET returns 401.
- Payment types: `GPWALLET`, `POSTPAID`, `INSTALMENT_4`, `INSTALMENT_8`, `INSTALMENT_12`.

## Endpoint Catalogue

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | `/grabid/v1/oauth2/.well-known/openid-configuration` | none | Service discovery |
| POST | `/grabpay/partner/v4/bind` | Partner HMAC | Initiate bind, returns `request` |
| GET | `/grabid/v1/oauth2/authorize` | browser (PKCE) | Bind consent page (scope `payment.recurring_charge`) |
| POST | `/grabid/v1/oauth2/token` | client_secret | Exchange code for token (store 10 years) |
| GET | `/grabpay/partner/v4/bind/{partnerTxID}/status` | Partner HMAC | Bind status; returns `oAuthCode` if redirect was lost |
| GET | `/grabpay/partner/v4/bind/paymentMethods?currency=&amount=` | Bearer + POP | Available methods, balances, limits, instalment schedule |
| GET | `/grabpay/partner/v4/promo?merchantID=&currency=&amount=&paymentType=&msgID=&promoCode=` | Bearer + POP | List promos |
| POST | `/grabpay/partner/v4/promo/validate` | Bearer + POP | Validate promo token, get discounted amount |
| POST | `/grabpay/partner/v4/charge` | Bearer + POP | Charge the bound user |
| POST | `/grabpay/partner/v4/charge/continue` | Bearer + POP | Continue a charge after wallet top-up redirect |
| GET | `/grabpay/partner/v4/charge/{partnerTxID}/status?currency=` | Bearer + POP | Charge status |
| POST | `/grabpay/partner/v4/direct/refund` | Partner HMAC | Refund (sample curl shows `/v4/refund`; table shows `/v4/direct/refund`) |
| GET | `/grabpay/partner/v4/direct/refund/{partnerTxID}/status?currency=` | Partner HMAC | Refund status |
| DELETE | `/grabpay/partner/v4/bind` | Bearer + POP | Unbind (deactivate token) |
| POST | merchant webhook URL | Grab partner HMAC | Bind, Charge, Refund, Unbind notifications |

## Initiate Bind: `POST /grabpay/partner/v4/bind`

Headers: `Authorization: {partner_id}:{hmac}`, `Content-Type: application/json`, `Date` (current, same as signed).

| Field | Type | Req | Notes |
|---|---|---|---|
| `partnerTxID` | string <=32 `[a-zA-Z0-9-_]` | Yes | Idempotency key for the bind |
| `countryCode` | string | Yes | `MY` |
| `merchantIDs` | string[] | Yes | Merchant IDs to link |
| `bindType` | string | Yes | `TOKENIZE` (only value) |

Response: `request` (JWT for the authorize URL), `partnerTxID`. Errors: 400 Bad Input `{"devMessage":"Bad Input.","arg":"..."}`; 401 Unauthorized (bad HMAC); 409 `no_record_found` (`code` 4090, merchant unknown), 409 `currency_mismatch`; 500 `service_error` (`code` 5003, retry).

Authorize URL params are the same as OTC except `scope=openid payment.recurring_charge` and `request` from bind. Redirect `error` values: `BIND_FAILED` (do not retry, ask Grab), `PAYMENT_METHOD_NOT_AVAILABLE` (do not retry). Error redirect also carries `error_description` and `state`.

## Get OAuth Token

Same as OTC: `POST /grabid/v1/oauth2/token` with `code`, `client_id`, `grant_type=authorization_code`, `redirect_uri`, `code_verifier`, `client_secret`. Response `access_token`, `token_type`, `expires_in`, `id_token`. Errors: 400 `unexpected_error` (missing params), 429 `unexpected_error`, 404 `invalid_grant` (code not found), 400 `invalid_request` (bad code challenge), 400/404 `invalid_token`, 400 `unauthorized_client` (scopes not allowed).

## Bind Status: `GET /grabpay/partner/v4/bind/{partnerTxID}/status`

Partner HMAC. Response: `transactionID`, `txStatus` (`success`, `failed`, `processing`), `oAuthCode` (sample key spelled `OAuthCode`).

## Check Payment Methods: `GET /grabpay/partner/v4/bind/paymentMethods?currency=MYR&amount=<int>`

Bearer + POP. `amount` recommended, `currency` required. Response:
- `userDetail.phoneNumber` (masked).
- `paymentMethod[]`: `paymentType`, `balance` (float), `currency`, `subtitle`, `isAvailable` (bool), `reason`, `totalAmount`, `amountPerPayment`, `totalProcessingFee`, `processingFeePerPayment`, `processingFeePercentage`, `repaymentDetail[]` {`dueDate` (ms), `amountDue`}.

Note: balances and instalment amounts here are floats in major units, unlike request amounts. Errors: 400 `bad_request` (4003), 401, 409 `no_record_found`, 409 `invalid_bind_metadata` (user must re-bind), 409 `currency_mismatch`, 500 (5003, retry).

## Create Charge: `POST /grabpay/partner/v4/charge`

Headers: Bearer, `X-GID-AUX-POP`, `Date`, `Content-Type`.

| Field | Type | Req | Notes |
|---|---|---|---|
| `partnerGroupTxID` | string <=32 | Yes | Order ID |
| `partnerTxID` | string <=32 | Yes | Unique per charge, idempotency key |
| `currency` | string | Yes | `SGD` or `MYR` |
| `amount` | int64 | Yes | Smallest unit |
| `description` | string <=255 | Yes | |
| `merchantID` | string <=255 | Yes | |
| `paymentType` | string | Yes | `GPWALLET`, `POSTPAID`, `INSTALMENT_4/8/12` |
| `promoToken` | string | No | From promo endpoints |
| `metaInfo` | object | No | `brandName`, `location` {ipAddress, latitude, longitude, accuracy}, `device` {deviceID, deviceModel, deviceBrand, iosUDID, imei}, `subMerchant` {name, category}, `partnerUserInfo` {id, phone, email} |
| `items` | object[] | No (recommended) | `itemName`, `quantity`, `price` (int), `category`, `supplierName`; if sent, `itemName`, `quantity`, `price` are mandatory |
| `shippingDetails` | object | No (recommended) | `firstName`, `lastName`, `address`, `city`, `postalCode`, `phone`, `email`, `countryCode` |
| `redirectDetail` | object | No | Registered partners only: `enableRedirect` (bool), `merchantRedirectURL` (required if enabled), `state` |

Response: `txID`, `description`, `txStatus` (`success`, `failed`, `processing`, `transaction_already_exist`), `reason`, `paymentMethod`, `additionalInfo.amountBreakdown` {`grossAmount`, `paidAmount`, `merchantAcquisitionPromoAmount`, `merchantRetentionPromoAmount`, `grabPromoAmount`}, `redirectionDetail` {`status`, `redirectURL`, `txStatus`} when `enableRedirect=true` (then `txStatus: processing`, `reason: redirect_pending`).

Errors: 400 Bad Input; 401 (bad token or POP); 409 `token_mismatch` (4095, token not for this merchant), `token_deactivated` (4095, re-bind), `wallet_not_bound` (4095, re-bind), `invalid_payment_method` (4090), `external_service_failed` (4091, retry); 500 `service_error`/`transaction_unknown_status` (5001, retry); 200 with `txStatus: failed` and `description` `err_charge_failed`, `currency_mismatch`, `user_compliance_check_failed`, `merchant_fraud_check_failed` (mark failed), or `insufficient_fund` with `reason: insufficient_balance` (user must top up).

## Top-up and Continue: `POST /grabpay/partner/v4/charge/continue`

If `enableRedirect` was true and the wallet was short, redirect the user to `redirectURL`; after top-up Grab redirects to `merchantRedirectURL`. Then call continue with Bearer + POP and body `partnerTxID` (new unique ID for this continue request) and `txID` (from `/charge`). Response mirrors charge: `txID`, `description`, `txStatus`, `reason`, `additionalInfo`, `paymentMethod`.

## Charge Status: `GET /grabpay/partner/v4/charge/{partnerTxID}/status?currency=MYR`

Bearer + POP. Response: `txID`, `description`, `txStatus`, `reason`, `paymentMethod`, `additionalInfo.amountBreakdown`. Unknown or `processing`: wait for webhook or poll at most every 2 minutes.

## Promotions

- Get Promo `GET /grabpay/partner/v4/promo`: query `merchantID`, `currency`, `amount`, `paymentType`, `msgID` (unique request ID), `promoCode` (for non-discoverable promos). Response `promoList[]` {`token`, `name`, `description` (may contain HTML), `icon`, `headerImage`, `currency`, `isAvailable`}. Errors: 409 `token_mismatch`, 409 `promo_not_found`.
- Validate Promo `POST /grabpay/partner/v4/promo/validate`: body `merchantID`, `currency`, `amount`, `paymentType`, `msgID`, `token`. Response promo fields plus `amountMeta` {`discountedAmount`, `additionalPaymentMethodDetail` {paymentType, totalAmount, amountPerPayment, totalProcessingFee, processingFeePerPayment, processingFeePercentage, repaymentDetail[]}}. Error 404 `promo_not_applicable`.
- Partial refunds are not allowed when a promo was applied.

## Refund: `POST /grabpay/partner/v4/direct/refund`

Partner HMAC (not Bearer). Body: `partnerGroupTxID`, `partnerTxID` (new, unique), `currency`, `amount`, `description` (optional), `merchantID`, `originTxID` (charge `txID`). Response: `txID`, `description`, `txStatus`, `reason`, `paymentMethod`, `additionalInfo.amountBreakdown` (adds `refundedAmount`). Refunds still work after unbind within the refund window.

Errors: 400 Bad Input; 401 `refund_window_exceeded` (4010); 409 `invalid_merchant` (40953, partner differs from charge), `concurrent_refunds_not_supported` (4090), `payment_not_found` (4090); 500 `service_error` (5001, retry). HTTP 200 outcomes: `txStatus: processing` + `reason: unknown` or `pending_approval` (call refund status), `failed` with `description` `transaction_decline`, `partner_mismatch`, `currency_mismatch`, `user_mismatch`, `partial_refund_not_allowed`, `refund_exceeds_charge_amount` (`reason: exceed_payment_amount`), `concurrent_transfers_not_supported`, `insufficient_fund` (`reason: merchant_insufficient_balance`).

Refund status `GET /grabpay/partner/v4/direct/refund/{partnerTxID}/status?currency=MYR`, Partner HMAC; same response shape.

## Unbind: `DELETE /grabpay/partner/v4/bind`

Bearer + POP. Body `merchantIDs` (string[]). Response: `txStatus`, `partnerTxID` (of the bind), `statusDetails` {`status` `UNBINDED`, `statusCode` `BUS-TXN-UNB`, `statusReason`}. After unbind, charge status and payment methods calls error. Users can also unbind in the Grab app; Grab then sends an Unbind webhook. Errors: 401, 400 `bad_request` (4003, missing params or bind not success), 500 (5003, retry).

## Webhooks (v4)

Sent for Bind, Charge, Refund, Unbind. Respond HTTP 200; otherwise Grab makes 2 more attempts. Headers as OTC: `Authorization: {partner_id}:{hmac}`, `Date`, Content-Type `application/json` (use `application/json; charset=utf-8` when recomputing the HMAC).

| Field | Notes |
|---|---|
| `txType`, `txCategory` | `Bind`, `Charge`, `Refund`, `Unbind` |
| `partnerID` | Your partner ID |
| `partnerTxID` | Your ID |
| `txID` | GrabPay ID |
| `OrigTxID` | Refund only: original charge `txID`; otherwise same as `txID` |
| `txStatus` | `success` or `failed` |
| `amount`, `currency` | Charge and refund only |
| `createdAt`, `completedAt` | Unix timestamps |
| `Payload` | Unbind only: unlinked `merchantIDs` |

Ignore fields not in this table (deprecated).

## v4 Reason codes

`insufficient_balance`, `kyc_check_failed`, `kyb_check_failed`, `merchant_insufficient_balance`, `currency_mismatch`, `payment_not_found`, `exceed_payment_amount`, `partial_refund_not_allowed` (promo applied), `wrong_transaction_state`, `authorising`, `capturing`, `cancelling`, `charging`, `refunding`, `pending_approval`, `redirect_pending`, `timeout`, `unknown`.

## v4 HTTP codes

200 OK; 201 redirect resource created (proceed to the redirect step); 400 bad request; 401 unauthorized; 403 forbidden; 404 not found; 409 conflict (see reason); 500 internal error (retry); 503 maintenance (retry later). OAuth extra errors: `server_error`, `temporarily_unavailable`; OpenID errors include `account_selection_required`, `invalid_request_uri`, `request_not_supported`, `registration_not_supported`.
