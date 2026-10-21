# Gkash API Reference

Use this reference for factual Gkash API guidance. Facts below come from the official docs at `https://doc.gkash.my/` (checked 2026-10-21) unless marked otherwise. The docs site is a Blazor app; page text only appears after JavaScript runs, so plain HTTP fetches of the pages return an empty shell.

## Source URLs

- `https://doc.gkash.my/getting-started`
- `https://doc.gkash.my/v2/web-payments`
- `https://doc.gkash.my/web-payments` (legacy)
- `https://doc.gkash.my/v2/preauth-and-capture`
- `https://doc.gkash.my/v2/recurring-payments`
- `https://doc.gkash.my/query-transaction-status`
- `https://doc.gkash.my/refund`
- `https://doc.gkash.my/google-pay`
- `https://doc.gkash.my/ewallet-payments`
- `https://doc.gkash.my/terminal-payments`
- `https://doc.gkash.my/soundbox-payments`
- `https://doc.gkash.my/preauth-and-capture` (legacy)
- `https://doc.gkash.my/recurring-payments` (legacy)
- `https://doc.gkash.my/woocommerce`, `https://doc.gkash.my/opencart`
- `https://wordpress.org/plugins/woo-gkash/` (official Gkash plugin, author Gkash Sdn. Bhd.)
- `https://github.com/gkashmy/gkash-ios-sdk`

## Integration Surfaces

- Host-to-host checkout (current): server posts JSON to Gkash, gets a redirect URL or auto-submit HTML form back. Docs say: "For new integrations, use the host-to-host checkout flow."
- Legacy checkout: the browser posts an HTML form directly to Gkash's hosted payment page.
- Pre-auth and capture (cards).
- Recurring payments (card tokenisation via `epkey`).
- Google Pay on the hosted page (selected merchants, enabled in merchant portal).
- Plugins: WooCommerce, OpenCart.
- Mobile SDKs: Android, iOS, React Native (`https://github.com/gkashmy`).
- Offline: e-wallet QR (customer or merchant presented), static DuitNow QR, SoftPOS terminal, Soundbox, e-Invoice QR. Full field tables and their different signature rules are in `references/gkash-offline.md`.

Hosted page tabs (from `preselection` / `paymentmethod` values): `ECOMM` (credit/debit card), `EBANKING` (internet banking), `EWALLET`.

## Endpoint Catalogue

Every operation on the public docs site (15 pages, enumerated 2026-10-21). Paths are shown on the staging host `https://api-staging.pay.asia`. Offline operations are detailed in `references/gkash-offline.md`.

| # | Method | Path | Body | Purpose | Doc page |
| --- | --- | --- | --- | --- | --- |
| 1 | POST | `/api/payment/form` | JSON | Host-to-host checkout; returns `redirect.url` or `redirect.html`. Also used for pre-auth (`v_preauth=PREAUTH`) and first recurring charge (`recurringtype`). | `/v2/web-payments`, `/v2/preauth-and-capture`, `/v2/recurring-payments` |
| 2 | POST | `/api/paymentform.aspx` | Form (browser) | Legacy hosted payment form; same fields, pre-auth and recurring variants. | `/web-payments`, `/preauth-and-capture`, `/recurring-payments` |
| 3 | GET (302) | `/api/v2/GUnifiedSelect.aspx?i=...` | n/a | Hosted payment page that `redirect.url` points to. Do not build this URL yourself. | `/v2/web-payments` |
| 4 | POST | `/API/PaymentPage.aspx` | Form (auto-submit) | Target of the `redirect.html` form (`POID`, `CartID`, `Token`). Render the HTML as returned. | `/v2/web-payments` |
| 5 | POST | `<callbackurl>` (yours) | Form | Server-to-server result. Verify signature, reply `OK`. | all payment pages |
| 6 | POST | `<returnurl>` (yours) | Form (browser) | Browser return with signed result. UX only. | web payment pages |
| 7 | POST | `/api/payment/submit` | Form | Server-to-server: pre-auth capture (`v_preauth=<POID>`), recurring charge (`epkey`), e-wallet QR scan (`QRCode`), QR display (`paymentid`), Soundbox (`isSoundBox`). | `/v2/preauth-and-capture`, `/v2/recurring-payments`, `/ewallet-payments`, `/soundbox-payments` |
| 8 | POST | `/api/payment/query` | JSON | Transaction status, including refund status. | `/query-transaction-status` |
| 9 | POST | `/api/payment/refund` | JSON | Refund a transaction. | `/refund` |
| 10 | POST | `/apim/merchant/GetDuitNowStaticQR` | JSON | Fetch the merchant static DuitNow QR (needs `token`). | `/ewallet-payments` |
| 11 | POST | `/apim/merchant/RemotePay` | JSON | Push a payment to the Gkash SoftPOS app. | `/terminal-payments` |
| 12 | POST | `/apim/merchant/RemoteCancel` | JSON | Cancel an ongoing SoftPOS payment. | `/terminal-payments` |
| 13 | POST | `/apim/merchant/SoundboxCancel` | JSON | Cancel an ongoing Soundbox payment. | `/soundbox-payments` |
| 14 | POST | `/apim/merchant/PublishInvoice` | JSON | Show an e-Invoice QR on a Soundbox. | `/soundbox-payments` |

Merchant portal only (no API): Google Pay enablement under Payment Settings > Customize Payment Page (`/google-pay`). Plugin pages (`/woocommerce`, `/opencart`) describe configuration only.

Not offered as a documented API (checked 2026-10-21): void or cancel of an online card payment, pre-auth release, payout or disbursement, settlement report download, webhook registration (the callback URL is sent per request), customer or token management beyond `epkey`, the "Userlogin" call that issues the static QR `token`.

## Enums

| Enum | Values | Source |
| --- | --- | --- |
| Payment `status` | `88 - Transferred`, `66 - Failed`, `11 - Pending` | all payment pages |
| Refund `status` | `00 - Refund Successful`, `01 - Refund Failed`, `02 - Partial Refund Successful` | `/refund` |
| Query `refundstatus` | `00 - Refund Successful`, `01 - Refund Failed`, or empty | `/query-transaction-status` |
| `preselection`, `paymentmethod` | `ECOMM`, `EBANKING`, `EWALLET` | `/v2/web-payments` |
| `installment` | `true`, `false` | `/v2/web-payments` |
| `installmentbank` | `CIMB`, `PBB`, `HLB`, `RHB` | `/v2/web-payments` |
| `installmentduration` | `6`, `12`, `24`, `36` | `/v2/web-payments` |
| `recurringtype` | `MONTHLY`, `ANNUAL` | `/v2/recurring-payments` |
| `v_preauth` | `PREAUTH`, or a `POID` to capture | `/v2/preauth-and-capture` |
| `v_currency` | `MYR` only | all pages |
| QR `paymentid`, SoftPOS `PaymentType` | See `references/gkash-offline.md` | `/ewallet-payments`, `/terminal-payments` |
| `PaymentType` (result, free text examples) | `Visa Credit 0011`, `Visa Debit`, `Master Credit`, `Master Credit 0008`, `GrabPay ECOMM`, `TNG QR SCAN`, `GrabPay QR SCAN`, `TNG QR DISPLAY`, `DUITNOW DYNAMIC QR DISPLAY` | various |
| `description` (examples) | `00 - Approved`, `SUCCESS`, `DecodedQRText:...` | various; informational only |

FPX bank codes, error codes, and HTTP error responses: not published on the docs site (checked 2026-10-21). Bank selection happens on the Gkash hosted page.

## Environments

| Environment | Host | Source |
| --- | --- | --- |
| Staging (sandbox) | `https://api-staging.pay.asia` | All doc pages |
| Production | `https://api.gkash.my` | Official WooCommerce plugin (`https://api.gkash.my/api/PaymentForm.aspx`) and iOS SDK `HOST_URL` |

The doc pages only show staging URLs. The production host is confirmed from two official Gkash code sources for the legacy payment form. That `/api/payment/form`, `/api/payment/query`, `/api/payment/refund`, and `/api/payment/submit` live at the same paths on `https://api.gkash.my` is a reasonable inference but is not stated on a doc page (checked 2026-10-21). Confirm with Gkash before go-live.

Sandbox merchant portal: `https://demo.pay.asia`.

## Authentication

There is no API key header or bearer token. Every request carries a `signature` field computed with the merchant's signature key. The merchant is identified by `CID` (Gkash "User ID", for example `M161-U-999`).

## Signature Algorithm (all online payment calls)

General rule from the docs:

1. Concatenate the listed fields in the exact order, separated by `;`.
2. Amount fields are formatted to 2 decimal places, then commas and the decimal point are removed. `100.00` -> `10000`, `1` -> `100`, `0.1` -> `010`.
3. Convert the whole string to uppercase (this also uppercases the signature key, cart ID, and status text).
4. Hash with SHA512. Send and compare as lowercase hex ("converted to lower case").

| Use | String to sign (before uppercasing) |
| --- | --- |
| Checkout request (v2 and legacy), recurring first charge, recurring repeat charge | `signatureKey;CID;v_cartid;v_amount;v_currency` |
| Pre-auth request (`v_preauth=PREAUTH`) | `signatureKey;CID;v_cartid;v_amount;v_currency;v_preauth` |
| Pre-auth capture (`v_preauth=<POID>`) | `signatureKey;CID;v_cartid;v_amount;v_currency;v_preauth` |
| Callback and return URL verification | `signatureKey;CID;POID;cartid;amount;currency;status` |
| Query request | `signatureKey;CID;cartid;amount;currency` |
| Refund request | `signatureKey;CID;cartid;amount;currency` |
| E-wallet QR scan, QR display, Soundbox (`/api/payment/submit`) | `signatureKey;CID;v_cartid;v_amount;v_currency` (same as checkout) |

The `/apim/merchant/*` endpoints (SoftPOS, Soundbox cancel, e-Invoice) use a different convention: input not uppercased, output UPPERCASE hex. See `references/gkash-offline.md`.

The callback doc text names the fields `v_cartid`, `v_amount`, `v_currency`, but the callback actually posts them as `cartid`, `amount`, `currency`. Use the posted values. Gkash's own WooCommerce plugin builds the callback string from posted `CID`, `POID`, `cartid`, `amount`, `currency`, `status`.

Documented test vectors (verified locally by recomputing SHA512, 2026-10-21):

```text
SHA512("SIGNATUREKEY9999;M161-U-999;MERCHANT-REFERENCE-712893;10000;MYR")
= 52d08ae99588d9ac00dbd59ae1ec6d3bf44657feeebaa4758517aa79aa6e791315a82055c5e498c780e2f977575c755d5623cd5ad198d780cde2742a7b92041c

SHA512("SIGNATUREKEY9999;M161-U-999;M161-PO-999999;MERCHANT-REFERENCE-712893;10000;MYR;88 - TRANSFERRED")
= 7a93e3420fef64dc7ac439cde5bcb8f6daaeff4336e1cc533dc70da64056a369a88ed3550d7b344a2feba0666413a676ec00fb4a93647f9dd207fe1e56e83fce

SHA512("SIGNATUREKEY9999;M161-U-999;MERCHANT-REFERENCE-712893;10000;MYR;PREAUTH")
= d214ad338b9c461b91820a21510a0f79edc2205a75b1b6d81fb1fd17fb7f366338c4a44a9bd865df3c0e31fa04bd6690149e2936ac90e04d4f7b3d2399d56025

SHA512("SIGNATUREKEY9999;M161-U-999;MERCHANT-REFERENCE-712894;10000;MYR;M161-PO-999999")
= 93d99585989aa4062f0b6b307166b85c232ebf2f4dab1eb1c1765b6dc32aa57e6375da29bd09b12bbf83f9b5de98d81c22d28d252f1c7f6d62ae4cd0f5cc77e7
```

Pseudo-code:

```ts
import { createHash, timingSafeEqual } from "node:crypto";

// "100.00" -> "10000", "1" -> "100", "0.1" -> "010". Use a decimal type, not float math, in production.
function sigAmount(amount: string): string {
  const [i, f = ""] = amount.replace(/,/g, "").split(".");
  return `${i}${(f + "00").slice(0, 2)}`;
}

function sign(parts: string[]): string {
  return createHash("sha512").update(parts.join(";").toUpperCase(), "utf8").digest("hex"); // lowercase hex
}

const requestSig = sign([KEY, CID, cartId, sigAmount(amount), "MYR"]);

function verifyCallback(p: Record<string, string>): boolean {
  const expected = sign([KEY, p.CID, p.POID, p.cartid, sigAmount(p.amount), p.currency, p.status]);
  const got = (p.signature ?? "").trim().toLowerCase();
  return got.length === expected.length && timingSafeEqual(Buffer.from(got), Buffer.from(expected));
}
```

`sigAmount` above assumes the input already has at most 2 decimals, as Gkash sends. Round your own amounts to 2 decimals first. Test it against `1` -> `100` and `0.1` -> `010`.

## Host-to-Host Checkout (current)

```text
POST https://api-staging.pay.asia/api/payment/form
Content-Type: application/json
```

Request fields (`https://doc.gkash.my/v2/web-payments`):

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `version` | String | Yes | Constant `1.5.5`. |
| `CID` | String | Yes | Merchant ID, for example `M161-U-999`. |
| `v_cartid` | String | Yes | Unique merchant reference. Special characters allowed: `-`, `_`, `.`. |
| `v_currency` | String | Yes | ISO 4217. Only `MYR` supported. |
| `v_amount` | String | Yes | Decimal string, for example `100.00`. |
| `signature` | String | Yes | Lowercase SHA512, see above. |
| `v_preauth` | String | Conditional | `PREAUTH` for pre-auth, or a `POID` to capture. |
| `recurringtype` | String | Conditional | `MONTHLY` or `ANNUAL`. |
| `subscriptioncount` | String | Conditional | Number of charges within a subscription. |
| `returnurl` | String | No | Browser is redirected here after payment. |
| `callbackurl` | String | No | Server-to-server result is posted here. Always send it. |
| `v_firstname`, `v_lastname` | String | No | Buyer name. |
| `v_billemail` | String | No | Buyer email. |
| `v_billstreet`, `v_billpost`, `v_billcity`, `v_billstate` | String | No | Billing address. |
| `v_billcountry` | String | No | ISO 3166-1 alpha-2, for example `MY`. |
| `v_billphone` | String | No | With country code, for example `60124567890`. |
| `v_shipstreet`, `v_shippost`, `v_shipcity`, `v_shipstate`, `v_shipcountry` | String | No | Shipping address. |
| `clientip` | String | No | Buyer IP. |
| `v_productdesc` | String | No | Product description. |
| `preselection` | String | No | Preselect tab: `ECOMM`, `EBANKING`, `EWALLET`. |
| `paymentmethod` | String | No | Comma-separated tabs to show, for example `EWALLET,ECOMM`. |
| `installment` | String | No | `true` or `false`: show installment option. |
| `installmentbank` | String | No | `CIMB`, `PBB`, `HLB`, `RHB`. |
| `installmentduration` | String | No | Months: `6`, `12`, `24`, `36`. |

Example:

```bash
curl -L 'https://api-staging.pay.asia/api/payment/form' \
  -H 'Content-Type: application/json' \
  -d '{"version":"1.5.5","CID":"<CID>","v_currency":"MYR","v_amount":"100.00",
       "v_cartid":"order-123-a1","signature":"<sha512>",
       "returnurl":"https://example.com/pay/return","callbackurl":"https://example.com/pay/callback"}'
```

Response fields (note the different key casing):

| Field | Notes |
| --- | --- |
| `status` | `11 - Pending` at creation. |
| `cid` | Merchant ID. |
| `poId` | Gkash transaction reference, for example `M161-PO-999999`. Store it. |
| `cartId` | Your `v_cartid`. |
| `amount`, `currency` | Echoed. |
| `signature` | SHA512 returned by Gkash. The docs do not state its formula, but the documented example (`13498ca0...d7a4`) equals the callback formula with status `11 - PENDING`, verified locally. |
| `redirect.url` | If present, HTTP 302 the browser to it (for example `.../api/v2/GUnifiedSelect.aspx?i=...`). |
| `redirect.html` | If present, render the full HTML; its script auto-submits a form to the Gkash payment page. |

Exactly one of `redirect.url` or `redirect.html` is returned. The checkout response is not a payment result.

## Legacy Checkout (browser form)

```text
POST https://api-staging.pay.asia/api/paymentform.aspx
Content-Type: application/x-www-form-urlencoded
```

Same request fields and signature as host-to-host (minus the JSON response). The browser submits an HTML form with hidden inputs directly to Gkash. Gkash's WooCommerce plugin 1.3.0 still uses this flow with `version` `1.5.0`. Compute the signature on your server and render the form; never put the signature key in the page.

## Callback (server-to-server)

```text
POST <callbackurl>
Content-Type: application/x-www-form-urlencoded
```

| Field | Required | Notes |
| --- | --- | --- |
| `CID` | Yes | Merchant ID. |
| `POID` | Yes | Gkash transaction reference. |
| `status` | Yes | `88 - Transferred`, `66 - Failed`, `11 - Pending`. |
| `cartid` | Yes | Your `v_cartid`. |
| `currency` | Yes | `MYR`. |
| `amount` | Yes | Decimal string, for example `100.00`. |
| `signature` | Yes | Lowercase SHA512 over `key;CID;POID;cartid;amount;currency;status`. |
| `epkey` | Conditional | Recurring token (recurring flow only). |
| `SubscriptionRemID` | Conditional | Subscription ID, for example `M161-SC-9999`. |
| `SubscriptionCount` | Conditional | Successful charges so far in the subscription. |
| `description` | No | For example `00 - Approved`. Informational only; varies by acquirer. |
| `PaymentType` | No | For example `Visa Credit 0011`, `GrabPay ECOMM`, `Master Credit`. |

Rules from the docs:

- "It is mandatory to verify the signature sent by Gkash."
- "Merchants are required to print the text "OK" in the response body upon receiving the callback." Respond HTTP 200 with body `OK`.

Handler order:

1. Read form fields as raw strings (do not trim or reformat before signing, other than the amount rule).
2. Verify signature in constant time. Reject on mismatch (do not reply `OK`).
3. Check posted `CID` equals your CID, and `cartid`, `amount`, `currency` match the stored pending payment.
4. Map status by leading code: `88` paid, `66` failed, `11` pending (leave open, requery later).
5. Persist idempotently keyed on `POID` / `cartid`, then reply `OK`. Reply `OK` to verified duplicates without re-fulfilling.

Not documented in official docs (checked 2026-10-21): callback retry schedule, what happens if `OK` is not returned, callback source IPs.

## Return URL (browser)

```text
POST <returnurl>
Content-Type: application/x-www-form-urlencoded
```

Fields: `CID`, `POID`, `status`, `cartid`, `currency`, `amount`, `signature`, `description`, `PaymentType`. Docs: "It is also mandatory to verify the signature sent by Gkash using the same calculation as the server-side callback signature calculation."

Your return route must accept POST (not only GET). Use it to show a result or "confirming payment" page. Fulfil from the verified callback or a query result, not from the return alone. Gkash's own WooCommerce plugin changes order state only from the callback.

## Status Codes

| Value | Meaning |
| --- | --- |
| `88 - Transferred` | Successful payment. |
| `66 - Failed` | Failed. |
| `11 - Pending` | Pending (also the checkout response status). |
| `00 - Refund Successful` | Refund response / `refundstatus`. |
| `01 - Refund Failed` | Refund response / `refundstatus`. |
| `02 - Partial Refund Successful` | Refund response. |

## Query Transaction Status

```text
POST https://api-staging.pay.asia/api/payment/query
Content-Type: application/json
```

| Field | Required | Notes |
| --- | --- | --- |
| `version` | Yes | Constant `1.3.0`. |
| `CID` | Yes | Merchant ID. |
| `cartid` | Yes | Your cart ID. |
| `currency` | Yes | `MYR`. |
| `amount` | Yes | Original amount, for example `100.00`. |
| `signature` | Yes | SHA512 over `key;CID;cartid;amount;currency`. |

Response fields: `CID`, `POID`, `status`, `cartid`, `currency`, `amount`, `description`, `PaymentType`, `refundstatus`, `refundamount`, `refunddate` (for example `2020-10-22 00:00:00`). The documented response has no `signature` field, so trust it only because you called Gkash over TLS yourself. Still match `cartid`, `amount`, `currency` before settling.

Use it for pending callbacks, missing callbacks, and reconciliation. Rate limits for query: not documented (checked 2026-10-21).

## Refund

```text
POST https://api-staging.pay.asia/api/payment/refund
Content-Type: application/json
```

| Field | Required | Notes |
| --- | --- | --- |
| `version` | Yes | Constant `1.3.1`. |
| `CID` | Yes | Merchant ID. |
| `cartid` | Yes | Cart ID of the original payment. |
| `currency` | Yes | `MYR`. |
| `amount` | Yes | Doc description says "Transaction Amount". |
| `signature` | Yes | SHA512 over `key;CID;cartid;amount;currency`. |

Response (HTTP 200 JSON):

| Field | Required | Notes |
| --- | --- | --- |
| `CID` | Yes | Merchant ID. |
| `POID` | Yes | Gkash reference of the transaction. |
| `status` | Yes | `00 - Refund Successful`, `01 - Refund Failed`, `02 - Partial Refund Successful`. |
| `cartid` | Yes | Cart ID. |
| `currency` | Yes | `MYR`. |
| `amount` | Yes | Amount. |
| `description` | No | Informational. |

The refund response has no `signature` field.

Uncertain: the docs list a partial refund status, and the example request sends `100.00` while the example response shows `1.00`. Whether `amount` is the original transaction amount or the amount to refund is not stated (checked 2026-10-21). Confirm with Gkash before building partial refunds. Store a local refund record before calling and make retries idempotent.

## Pre-auth and Capture

1. Checkout as above with `v_preauth=PREAUTH`. Signature: `key;CID;v_cartid;v_amount;v_currency;PREAUTH`.
2. Callback arrives as normal; save `POID`.
3. Capture server-to-server:

```text
POST https://api-staging.pay.asia/api/payment/submit
Content-Type: application/x-www-form-urlencoded
```

Same fields as checkout with `v_preauth=<POID>` and a new `v_cartid` (docs example uses a different cart ID). Signature: `key;CID;v_cartid;v_amount;v_currency;<POID>`.

Capture request fields used in the docs example: `version` (`1.5.5`), `CID`, `v_currency`, `v_amount`, `v_cartid`, `signature`, `v_preauth`. Docs: "the same parameters as Web Payments", so the optional buyer fields are accepted too.

Synchronous JSON response (HTTP 200):

| Field | Notes |
| --- | --- |
| `status` | For example `88 - Transferred`. |
| `description` | For example `00 - Approved`. Informational. |
| `CID` | Merchant ID. |
| `POID` | Gkash reference. |
| `cartid` | Capture cart ID. |
| `amount`, `currency` | Captured amount, `MYR`. |
| `PaymentType` | For example `Master Credit 0008`. |
| `signature` | See the sync response signature note under Recurring Payments. |

Pre-auth expiry, partial capture, and void/release: not documented (checked 2026-10-21).

## Recurring Payments

1. Checkout with `recurringtype` (`MONTHLY` or `ANNUAL`), optional `subscriptioncount`. Signature is the normal checkout signature (recurring fields are not signed).
2. Callback includes `epkey` (and may include `SubscriptionRemID`, `SubscriptionCount`). Store `epkey` encrypted at rest.
3. Charge again server-to-server: `POST /api/payment/submit` (`application/x-www-form-urlencoded`) with checkout fields, a new `v_cartid`, `recurringtype`, and `epkey`. Signature: `key;CID;v_cartid;v_amount;v_currency`.
4. The JSON response contains a new `epkey`: "Use it to initiate the next recurring payment."

Repeat charge request fields used in the docs example: `version` (`1.5.5`), `CID`, `v_currency`, `v_amount`, `v_cartid` (new), `signature`, `recurringtype`, `epkey`. Docs: "the same parameters as a Web Payments checkout request".

Repeat charge JSON response (HTTP 200): `status`, `description`, `CID`, `POID`, `cartid`, `amount`, `currency`, `PaymentType`, `epkey` (new token, replaces the old one), `signature`.

Recurring callback adds `epkey`, and conditionally `SubscriptionRemID` (for example `M161-SC-9999`) and `SubscriptionCount` (successful charges so far). The recurring fields `recurringtype` and `subscriptioncount` are not part of any signature.

Synchronous response signature (capture, recurring, e-wallet submit): the formula is not stated, and the example value in the docs does not match the callback formula for the example data, so it looks like a placeholder (checked 2026-10-21). Confirm the final state with the query API or the callback, not the sync response signature alone.

## Amount and Currency Rules

- Amounts are decimal strings in ringgit with 2 decimals (`100.00`), not sen.
- Only `MYR` is supported.
- In signatures only, the amount becomes digits with no dot (`10000`).
- Minimum and maximum amount: not documented (checked 2026-10-21).

## Sandbox

- Create a sandbox account with the form on `https://doc.gkash.my/getting-started` (company name, email, contact number). Gkash emails a verification link and the integration details.
- Sandbox merchant portal: `https://demo.pay.asia` (account management, reports, real-time transactions).
- Staging API host: `https://api-staging.pay.asia`.
- Test cards, test banks, and simulated outcomes: not documented publicly (checked 2026-10-21).

## Not Documented (checked 2026-10-21)

- Rate limits.
- IP allowlist for callbacks or API calls.
- Error response format and error codes for rejected requests (for example bad signature).
- Callback retry policy.
- Production paths for the JSON endpoints (only the host is confirmed).

## Common Errors

- Signature mismatch from not uppercasing the whole string (the key often has lowercase letters).
- Amount formatted as `100` or `100.0` instead of `10000` in the signature.
- Using `v_cartid` from your DB instead of the posted `cartid` (or vice versa) when they differ; sign posted values, then compare to DB.
- Status uppercased in the payload instead of only in the signature input.
- Return route only accepts GET; Gkash posts a form.
- Callback handler returns JSON or empty body instead of `OK`.
- Reusing a `v_cartid` on retry.
- Staging CID or key used against production.
