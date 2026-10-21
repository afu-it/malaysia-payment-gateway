# GrabPay & PayLater Codes (One Time Charge)

Status values, reason codes, and errors for the OTC Partner API. Source: `https://developer.grab.com/docs/payment-otc/api/v2` (sections "Status codes and reasons", "Error handling", "Initiate payments", "Refund payments") and the older pages `charge-status-reasons`, `refund-status-reasons`, `appendix-reason-params`, `grabpay-redirect`, `appendix-troubleshooting`, `init-payments` under `https://developer.grab.com/docs/paysi-agent-partner-v2-otc/`. Checked 2026-10-21. Codes tagged LEGACY will be deprecated. Grab says the list can grow, so treat unknown codes as non-final and re-query.

## Charge txStatus

| txStatus | Meaning | Final? |
|---|---|---|
| `success` | Charge succeeded | Yes |
| `failed` | Charge failed | Yes |
| `processing` | Still processing; wait for webhook or poll at most every 2 minutes | No |
| `cancelled` | User did not click Pay, or `/charge/complete` was not called before the authorisation expired | Yes (no money moved) |
| `authorised` | Authorised but not completed; call `/charge/complete` | No |
| `authorisation_declined` | Authorisation declined | Yes |
| `transaction_already_exist` | (Legacy) `partnerTxID` already used; call status. Newer merchants get the latest status instead | Re-query |

Deprecated field `status` (`success`, `failed`, `unknown`) is still returned; use `txStatus`.

## Charge reason codes

| reason | Meaning |
|---|---|
| `currency_mismatch` | Currency does not match the merchant configuration |
| `no_record_found` | No record for this transaction |
| `authorising` | Authorisation in progress |
| `capturing` | Capture in progress |
| `user_fail_consent` | User did not approve in time |
| `pending_user_consent` | Waiting for user on Grab checkout page |
| `pending_capture` | Waiting for `/charge/complete` |
| `cancelling` | Cancellation in progress |
| `auth_expired` | Cancelled because complete was not called in time |
| `unknown` | Reason cannot be determined |
| `kyc_compliance_decline` | Regulatory compliance decline |
| `transaction_decline` | Declined |
| `insufficient_balance` | User balance or limit insufficient |

## Charge txStatus + reason combinations (HTTP 200)

| txStatus | reason | Action |
|---|---|---|
| `failed` | (empty, LEGACY) | Unknown failure |
| `failed` | `err_capture_failed` | Internal capture error; do not retry this one; start a new payment |
| `cancelled` | (empty, LEGACY) or `auth_expired` | Start a new payment; complete within 10 minutes of auth webhook or redirect |
| `cancelled` | `user_fail_consent` | Start a new payment |
| `processing` | (empty), `pending_user_consent`, `authorising`, `capturing` | Re-query later |
| `authorised` | (empty, LEGACY) or `pending_capture` | Call `/charge/complete` before expiry |
| `authorisation_declined` | (empty, LEGACY), `kyc_compliance_decline`, `transaction_decline`, `insufficient_balance`, `kyc_check_failed` (LEGACY), `kyb_check_failed` (LEGACY), `user_compliance_check_failed` (LEGACY) | Final decline |
| `processing` / `failed` / `authorisation_declined` | `unknown` | Interpret by txStatus |

## Refund txStatus

| txStatus | Meaning |
|---|---|
| `success` | Refund succeeded |
| `failed` | Refund failed |
| `processing` | Processing; wait for webhook or poll at most every 2 minutes |
| `transaction_already_exist` | (Legacy) refund `partnerTxID` already used; call refund status |

## Refund reason codes and actions

| txStatus | reason | Action |
|---|---|---|
| `failed` | `partial_refund_not_allowed` | Only full refund allowed for this transaction |
| `failed` | `payment_not_found` | Check `originTxID` and amount |
| `failed` | `transaction_decline` | Declined by the system |
| `failed` | `merchant_insufficient_balance` | Top up merchant wallet (it is cashed out daily) |
| `failed` | `exceed_payment_amount` | Total refunds must not exceed the payment |
| `processing` | `refunding` or `pending_approval` | Check refund status later |
| `failed` / `processing` | `unknown` | Failed: mark failed. Processing: re-query |
| `failed` | (empty) | Mark failed; may re-initiate or contact support |

## Init 409 reasons

Body sample: `{"code": 4090, "reason": "invalid_merchant"}`.

| reason | Action |
|---|---|
| `currency_mismatch` | New `partnerTxID`, correct currency |
| `invalid_merchant` | Use your assigned `merchantID` |
| `transaction_already_exists` | Generate a new unique `partnerTxID` |
| `client_error` | Fix abnormal parameters |
| `user_does_not_exist` (LEGACY) | Merchant not onboarded for OTC |
| `no_record_found` (LEGACY) | Invalid `merchantID` |

## Refund 409 reasons

| reason | Action |
|---|---|
| `concurrent_refunds_not_supported` | A refund for this `partnerTxID` is in progress; do not call again, use status |
| `invalid_merchant` | Use your assigned `merchantID` |
| `payment_not_found` | `originTxID` does not match a refundable payment |
| `client_error` | Fix parameters |
| `unknown` (LEGACY) | Fix parameters and retry |
| `no_record_found` (LEGACY) | Invalid `merchantID` |

## HTTP codes (all OTC endpoints)

| HTTP | Meaning |
|---|---|
| 200 | Processed; read `txStatus` |
| 400 | Missing or malformed parameters |
| 401 | Bad HMAC, bad Bearer token, bad POP, or token not tied to the `partnerTxID` |
| 403 | Merchant lacks permission |
| 404 | Not found (wrong `partnerTxID` or currency on status endpoints) |
| 409 | Conflict, see reason |
| 429 | Rate limited; exponential backoff with jitter |
| 5xx | Grab internal error; retry later with backoff |

## Redirect `error` values (from the authorize page)

The redirect always carries `state`. On success it carries `code`; on failure `error`.

| error | Meaning | Action |
|---|---|---|
| `user_canceled` | User cancelled | - |
| `session_expired` | Session expired | New init with new `partnerTxID` |
| `invalid_acr_values` | Bad `acr_values` | Fix format, new init |
| `invalid_token` | Bad `request` | Use the `request` from a successful init |
| `invalid_argument` | Bad URL parameters | Fix, new init |
| `invalid_scope` | Bad `scope` | Use the scope given at onboarding |
| `mfa_not_completed` | MFA failed | New init |
| `transaction_not_found` | No transaction for the request | Fix parameters, new init |
| `kyc_compliance_decline` | Compliance decline | - |
| `transaction_declined` | Declined | - |
| `server_error` | Grab internal issue | New init later |
| `client_error` | Unspecified user error | New init later |
| `insufficient_balance` | Not enough balance on selected method | - |
| `unknown` | Not processed in time; cancelled; a temporary hold may show | New init later |
| `confirm_failed` (LEGACY) | User or server error | New init later |
| `invalid_request` (LEGACY) | Could not be processed | New init later |

Authorize endpoint 400 body sample: `{"code": 400, "reason": "bad_request"}`.

## OAuth 2.0 errors (token and authorize)

`invalid_request`, `unauthorized_client`, `access_denied`, `unsupported_response_type`, `invalid_scope`, `request_uri_not_supported`, `consent_required`, `login_required`, `interaction_required`, `invalid_token`, `invalid_request_object` (for example empty scope), `invalid_grant` (malformed or unknown grant/code), `invalid_client` (client ID or partner ID mismatch), `unexpected_error` (authorisation server unavailable; retry later).

## Payment methods

| Value | Where | Meaning |
|---|---|---|
| `GPWALLET` | responses, webhook | GrabPay wallet |
| `POSTPAID` | responses, webhook, `hidePaymentMethods` | PayLater, pay next month |
| `INSTALMENT_4`, `INSTALMENT_8`, `INSTALMENT_12` | responses, webhook | PayLater instalments |
| `INSTALMENT` | `hidePaymentMethods` only | Hides all instalment plans |
| `CARD` | Thailand only | Card |

## Webhook txType

`Init`, `Auth`, `Charge`, `Capture`, `Refund`. `Capture` = Grab captured the auth amount and moved it to the merchant. `Refund` = payment refunded. `Cancel` was removed on 6 Aug 2021.

## items[].itemCategory enum

1 Men's Wear, 2 Women's Apparel, 3 Men's Bag, 4 Women's Bag, 5 Men's Shoes, 6 Women's Shoes, 7 Mobile & Gadgets, 8 Beauty & Personal Care, 9 Home Appliances, 10 Home & Living, 11 Kids Fashion, 12 Toys, Kids & Babies, 13 Video Games, 14 Food & Beverages, 15 Computers & Peripherals, 16 Collectibles, Hobbies & Books, 17 Health & Wellness, 18 Travel & Luggage, 19 Tickets & Vouchers, 20 Watches, 21 Jewellery & Accessories, 22 Sports & Outdoors, 23 Automotives, 24 Cameras & Drones, 25 Pet Food & Accessories, 26 Airtime, 27 Credit Bill, 28 Other Bill, 29 Miscellaneous.

## Currency decimals

SGD 2, MYR 2, PHP 2, IDR 2, THB 2, VND 0.
