# Share Commerce API Reference

Distilled from the full public developer docs at `https://devdocs.share-commerce.com/` (29 pages, all pages show "Last updated on 22nd Sep 2026") and the official plugin source at `https://github.com/share-commerce`. Checked 2026-10-21. Every sample signature below was recomputed; results are marked **verified** or **does not reproduce**.

## Source URLs

- `https://devdocs.share-commerce.com/Overview`
- `https://devdocs.share-commerce.com/PreRequisite`
- `https://devdocs.share-commerce.com/Payment/PaymentRequest`
- `https://devdocs.share-commerce.com/Payment/RedirectCallback`
- `https://devdocs.share-commerce.com/Payment/SeemlessPaymentRequest`
- `https://devdocs.share-commerce.com/Payment/Enquiry`
- `https://devdocs.share-commerce.com/Payment/PaymentVoid`
- `https://devdocs.share-commerce.com/PreAuth/PreAuth`, `/PreAuth/CreateCheckout`, `/PreAuth/PreAuthVoid`, `/PreAuth/CaptureSale`, `/PreAuth/CaptureSaleVoid`, `/PreAuth/Enquiry`, `/PreAuth/Response`
- `https://devdocs.share-commerce.com/Recurring/CreateRequest`, `/Recurring/ResponseRedirect`, `/Recurring/Enquiry`, `/Recurring/CallbackEnquiryResponse`
- `https://devdocs.share-commerce.com/Appendix1` to `/Appendix5`
- `https://github.com/share-commerce/Plugin.WooCommerce-8.x` (`includes/class-wc-gateway-sharecommerce.php`)
- `https://github.com/share-commerce/Plugin.Opencart3`, `Plugin.Opencart4`, `Plugin.Magento.2.3.6`, `Plugin.Prestashop.1.7.-`, `Plugin.WooCommerce-5.9.0`

## Environments

| Environment | Base URL |
| --- | --- |
| Staging (UAT) | `https://stagingpayment.share-commerce.com` |
| Production | `https://payment.share-commerce.com` |

- Staging seemless checkout pages are served from `https://sandbox-checkout.share-commerce.com/...` (seen in the Seemless response sample).
- Merchant ID, Secret Key and testing credentials are issued separately for UAT and production by email after onboarding (PreRequisite page).
- Share Commerce whitelists the merchant server IP range "for enquiry purpose" (PreRequisite). Recurring Enquiry also says the merchant must provide IPs for API access whitelist.
- HTTPS only for all gateway calls.
- Public sandbox test cards/banks: not documented in official docs (checked 2026-10-21). Testing credentials come by email.
- Rate limits: not documented in official docs (checked 2026-10-21).
- Share Commerce outbound IPs (for allowlisting callbacks): not documented in official docs (checked 2026-10-21).

## Authentication

No API key header or OAuth. Every call is authenticated by `MerchantID` plus `SCSign`, an HMAC-SHA256 hex digest (lowercase in all samples) keyed with the merchant Secret Key. Where `SCSign` goes depends on the endpoint (see Signature section).

Docs example key used in all samples: `mSuE3Ttn5B8vJhe5ncMutMLV` (Seemless page uses `o4q1VRXDfBok71119uQjtS1f`). These are published example keys, useful only to unit-test your signer against the verified vectors below. Never use them as real credentials.

## Endpoint Catalogue

Paths are relative to the environment base URL unless noted.

| # | Method | Path | Purpose | Body | SCSign location |
| --- | --- | --- | --- | --- | --- |
| 1 | GET or POST (browser) | `/payment` | Hosted Payment Request; shows payment options or jumps to one method | form fields / query string | field |
| 2 | (gateway to merchant) GET | merchant `RedirectUrl` | Browser return with result | query string | field |
| 3 | (gateway to merchant) POST | merchant callback URL (registered at onboarding) | Server-to-server result | JSON | field |
| 4 | POST | `/seemlesspayment` | Create Seemless payment; returns `CheckoutUrl` for iframe | `application/x-www-form-urlencoded` | field |
| 5 | POST | `/enquiry` | Payment status requery | JSON | HTTP header |
| 6 | POST | `/Payment/Void` | Void a same-day payment | JSON | HTTP header |
| 7 | GET or POST (browser) | `/PreAuth` | Pre-Auth, legacy (do not use for new integrations) | form fields / query string | field |
| 8 | POST | `/CreateCheckout/PreAuth` | Pre-Auth, recommended; returns `CheckoutUrl` | JSON | field in JSON body |
| 9 | POST | `/PreAuth/Void` | Void a Pre-Auth | JSON | HTTP header |
| 10 | POST | `/PreAuth/Capture` | Capture / Sale from a Pre-Auth | JSON | HTTP header |
| 11 | POST | `/PreAuth/Capture/Void` | Void a Capture / Sale | JSON | HTTP header |
| 12 | POST | `/PreAuth/Enquiry` | Pre-Auth status requery | JSON | HTTP header |
| 13 | POST | `/CreateCheckout/Recurring` | Create a recurring plan; returns `CheckoutUrl` | JSON | HTTP header |
| 14 | (gateway to merchant) GET | merchant `RedirectUrl` | Recurring browser return | query string | field |
| 15 | POST | `/Recurring/Enquiry` | Recurring plan status and cycles | JSON | HTTP header |
| 16 | (gateway to merchant) POST | merchant callback URL | Recurring callback | JSON | HTTP header |
| 17 | POST | `http://[terminal_ip_address]:8888` | In-store card terminal local API (not online) | JSON | none |

Refund API, payout/disbursement API, settlement report API, card tokenisation API for one-click checkout: not documented in official docs (checked 2026-10-21). The Payment Void response message is `FULL_REFUNDED`, and Appendix 3 lists a `REFUND` payment type, but no refund endpoint is published.

## Field Conventions

- Legend from the docs: `M` mandatory; `O` optional: "Value can be empty, but parameter must exist." Send every optional parameter, empty if unused.
- Amount fields are `Decimal (18,2)` ringgit with two decimals, e.g. `101.50`. Not sen.
- `CurrencyCode` is a String(5); samples use `MYR` (a Seemless sample shows `USD`).
- `MerchantOrderNo` must be unique per order. Official plugins use `<orderId>_<unixTime>` so retries get new numbers, and put the store order id in `MerchantRef1`.

## 1. Payment Request (hosted checkout)

`GET` or `POST https://stagingpayment.share-commerce.com/payment` (production `https://payment.share-commerce.com/payment`). The browser must submit it (auto-submit HTML form, or redirect to the GET URL). Official plugins use both: WooCommerce redirects to a GET URL with `urlencode`d values; OpenCart/PrestaShop/Magento render a POST form.

Request fields, in sign order:

| # | Field | Type (Len) | M/O | Notes |
| --- | --- | --- | --- | --- |
| 1 | `MerchantID` | String (50) | M | Assigned by Share Commerce |
| 2 | `CurrencyCode` | String (5) | M | e.g. `MYR` |
| 3 | `TxnAmount` | Decimal (18,2) | M | e.g. `101.50` |
| 4 | `MerchantOrderNo` | String (50) | M | Unique per transaction |
| 5 | `MerchantOrderDesc` | String (1000) | M | Order description |
| 6 | `MerchantRef1` | String (100) | O | Merchant reference |
| 7 | `MerchantRef2` | String (100) | O | Merchant reference |
| 8 | `MerchantRef3` | String (100) | O | Merchant reference |
| 9 | `CustReference` | String (50) | O | Customer reference |
| 10 | `CustName` | String (150) | O | |
| 11 | `CustEmail` | String (150) | O | |
| 12 | `CustPhoneNo` | String (50) | O | Plugins strip `+` and `-` |
| 13 | `CustAddress1` | String (100) | O | |
| 14 | `CustAddress2` | String (100) | O | |
| 15 | `CustCountryCode` | String (10) | O | e.g. `MY` |
| 16 | `CustAddressState` | String (150) | O | |
| 17 | `CustAddressCity` | String (150) | O | |
| 18 | `RedirectUrl` | String (1000) | M | Browser return URL after completed/failed |
| 19 | `PaymentMethod` | String (100) | O | Appendix 2 code to skip the options page and go straight to that bank/wallet; empty shows the Share Commerce options page |
| 20 | `Versioning` | Integer | M | Table says "Hard code 7"; page samples use `6`; current plugins send `7` |
| 21 | `SCSign` | | M | See Signature section |

Response: the customer finishes on Share Commerce, then you receive the redirect (section 2) and the callback (section 3).

## 2 and 3. Redirect and Callback (payment result)

Share Commerce redirects the browser to `RedirectUrl` with a query string, and separately POSTs a callback "to merchant's callback URL ... This ensures that orders can be completed even in cases where the customer's connection is terminated prematurely. Callback will be in HTTP POST method." (Overview). The callback body is JSON (samples, and the official WooCommerce/OpenCart 4 handlers read `php://input` and `json_decode` it).

Fields (documented order):

| # | Field | Type (Len) | M/O | Notes |
| --- | --- | --- | --- | --- |
| 1 | `RespCode` | String (30) | M | Appendix 1 |
| 2 | `RespDesc` | String (1000) | O | e.g. `Success` |
| 3 | `MerchantID` | String (50) | M | |
| 4 | `TxnRefNo` | String (50) | M | Share Commerce transaction reference |
| 5 | `CurrencyCode` | String (5) | M | |
| 6 | `TxnAmount` | Decimal (18,2) | M | String in redirect, JSON number in callback sample (`101.50`) |
| 7 | `MerchantOrderNo` | String (50) | M | |
| 8 | `MerchantOrderDesc` | String (1000) | O | Callback sample spells it `MerchantOderDesc` (typo in sample) |
| 9 | `MerchantRef1` | String (100) | O | |
| 10 | `MerchantRef2` | String (100) | O | |
| 11 | `MerchantRef3` | String (100) | O | |
| 12 | `CustReference` | String (50) | O | |
| 13 | `CustName` | String (150) | O | |
| 14 | `CustEmail` | String (150) | O | Callback sample shows it still URL-encoded (`tomt%40email.com`) |
| 15 | `CustPhoneNo` | String (50) | O | |
| 16 | `CustAddress1` | String (100) | O | |
| 17 | `CustAddress2` | String (100) | O | |
| 18 | `CustCountryCode` | String (10) | O | |
| 19 | `CustAddressState` | String (150) | O | |
| 20 | `CustAddressCity` | String (150) | O | |
| 21 | `CustPostcode` | String (100) | O | |
| 22 | `PaymentMethod` | String (100) | M | Appendix 2 code chosen by the customer (Seemless shows `HOSTED_CARD`) |
| 23 | `MaskedPAN` | String (20) | O | e.g. `545301XXXXXX1234` for cards |
| 24 | `Versioning` | Integer | M | Table says "Hard code value 6" |
| 25 | `SCSign` | | M | See Signature section |

The Seemless postMessage sample shows a redirect carrying extra keys not in the table (`CustPhoneCountryCode`, `CustAddressPostcode`) and no `CustPostcode`. So the field set can differ by flow and version. Verify over what you actually receive (see below).

Callback acknowledgement: not documented in official docs (checked 2026-10-21). Official WooCommerce and OpenCart 4 handlers respond with body `OK`. Do the same with HTTP 200. Retry policy: not documented.

## 4. Seemless Payment Request (iframe)

`POST https://stagingpayment.share-commerce.com/seemlesspayment` (production `https://payment.share-commerce.com/seemlesspayment`), `Content-Type: application/x-www-form-urlencoded`, server-to-server. Then load the returned `CheckoutUrl` in an iframe.

Request fields: the same 21 fields and order as the Payment Request table above (here `Versioning` is marked O, "Hard code 7").

Request sample (from docs):

```text
MerchantID=BRYAN57328&CurrencyCode=MYR&TxnAmount=1.50&MerchantOrderNo=M1734657848&MerchantOrderDesc=Perfume&MerchantRef1=mref1&MerchantRef2=mref2&MerchantRef3=mref3&CustReference=ctRefIC111&CustName=Bryan NG&CustEmail=&CustPhoneNo=&CustAddress1=&CustAddress2=&CustCountryCode=&CustAddressState=&CustAddressCity=&RedirectUrl=https://httpbin.org/anything&PaymentMethod=&Versioning=7&SCSign=a82ebd04...
```

Response (JSON):

| Field | Type | M/O | Notes |
| --- | --- | --- | --- |
| `RespCode` | String (50) | M | Appendix 1 |
| `RespMessage` | String (2000) | M | |
| `MerchantID` | String | M | |
| `MerchantOrderNo` | String | M | |
| `GatewayTxnRefNo` | String | M | Gateway reference |
| `CheckoutUrl` | String | M | Load in iframe |
| `Signature` | String | (sample only) | Present in sample, not in table; its algorithm is not documented |

postMessage from the iframe: `{"key": "<event key>", "status": "SUCCESS" | "FAILED", "RedirectUrl": "<RedirectUrl with signed result query>"}`. The docs show event key `seemlesspayment86DNZK`; whether it is fixed or per merchant is not documented. Note both samples carry `RespCode=00` even when `status` is `FAILED`. Never settle from postMessage; settle from the verified callback.

## 5. Payment Enquiry (requery)

`POST https://stagingpayment.share-commerce.com/enquiry` (production `https://payment.share-commerce.com/enquiry`). JSON body, `SCSign` in HTTP header.

| Field | Type (Len) | M/O | Notes |
| --- | --- | --- | --- |
| `MerchantID` | String (50) | M | |
| `MerchantOrderNo` | String (50) | M | As submitted on the Payment Request |
| `Versioning` | Integer | M | Table "Hard code 7"; sample uses `6` |
| Header `SCSign` | | M | |

Response: same fields as the Redirect (section 2 table). The page says "Refer to below SCSign Calculation section" but has none. Assume (unconfirmed) the same raw-JSON-body HMAC used by every other JSON POST, and verify the response with the section 2/3 rule. Confirm with Share Commerce.

## 6. Payment Void

`POST https://stagingpayment.share-commerce.com/Payment/Void` (production `https://payment.share-commerce.com/Payment/Void`). "Only allow void for same day transaction." JSON body, `SCSign` header = HMAC of the raw body.

| Field | Type (Len) | M/O | Notes |
| --- | --- | --- | --- |
| `MerchantID` | String (50) | M | |
| `MerchantOrderNo` | String (50) | M | From the payment |
| `TxnRefNo` | String (50) | M | From the payment result |
| `VoidRemarks` | String (150) | M | |
| Header `SCSign` | | M | |

Response (JSON): `RespCode` String(50) M (Appendix 1), `RespMessage` String(2000) M (sample `FULL_REFUNDED`), `MerchantOrderNo` M, `TxnRefNo` M, `VoidAmount` Decimal(18,2) M, `VoidRemarks` String(150) M. Whether the response is signed is not documented.

## 7 and 8. Pre-Auth (card authorisation)

Recommended for new work: `POST /CreateCheckout/PreAuth` (JSON). Legacy: `GET`/`POST /PreAuth` (form, "do not use for new integrations"). Same fields and same signing for both; `SCSign` is a field (in the JSON body for CreateCheckout), not a header.

| # | Field | Type (Len) | M/O | Notes |
| --- | --- | --- | --- | --- |
| 1 | `MerchantID` | String (50) | M | |
| 2 | `CurrencyCode` | String (5) | M | |
| 3 | `PreAuthAmount` | Double (18,2) | M | Sample sends string `"101.50"` |
| 4 | `MerchantOrderNo` | String (50) | M | Unique per Pre-Auth |
| 5 | `MerchantOrderDesc` | String (1000) | M | |
| 6 | `MerchantPaymentNo` | String (50) | M | Unique per request |
| 7 | `MerchantRef1` | String (100) | O | |
| 8 | `MerchantRef2` | String (100) | O | |
| 9 | `MerchantRef3` | String (100) | O | |
| 10 | `CustReference` | String (50) | O | "used for customer tokenization if value provided" |
| 11 | `CustName` | String (150) | O | |
| 12 | `CustEmail` | String (150) | O | |
| 13 | `CustPhoneNo` | String (50) | O | |
| 14 | `CustAddress1` | String (100) | O | |
| 15 | `CustAddress2` | String (100) | O | |
| 16 | `CustCountryCode` | String (10) | O | |
| 17 | `CustAddressState` | String (150) | O | |
| 18 | `CustAddressCity` | String (150) | O | |
| 19 | `CustPostcode` | String (10) | O | |
| 20 | `RedirectUrl` | String (1000) | M | |
| 21 | `PaymentMethod` | String (100) | O | Blank shows landing page |
| 22 | `Versioning` | Integer | O | "Hard code 2" |
| 23 | `SCSign` | | M | |

CreateCheckout/PreAuth response (JSON, NOT signed). `RespCode` here is an INTEGER `1` success / `9` failed, not Appendix 1:

| Field | Type | M/O | Notes |
| --- | --- | --- | --- |
| `RespCode` | Integer | M | `1` or `9` |
| `RespMessage` | String | O | Only when not success |
| `TxnRefNo` | String (50) | M | |
| `CheckoutCode` | String | M | |
| `CheckoutUrl` | String | M | Redirect the customer here |

## 9 to 12. Pre-Auth Void, Capture, Capture Void, Enquiry

All JSON POST, `SCSign` header = HMAC of the raw request body.

| Endpoint | Fields (all M) |
| --- | --- |
| `/PreAuth/Void` | `MerchantID`, `MerchantOrderNo`, `MerchantPaymentNo` (unique per request), `PreAuthTxnRefNo` (TxnRefNo of the successful Pre-Auth), `VoidAmount` Decimal(18,2) |
| `/PreAuth/Capture` | `MerchantID`, `MerchantOrderNo`, `MerchantPaymentNo`, `PreAuthTxnRefNo`, `CaptureAmount` Decimal(18,2) (cannot exceed the Pre-Auth amount) |
| `/PreAuth/Capture/Void` | `MerchantID`, `MerchantOrderNo`, `MerchantPaymentNo`, `PreAuthTxnRefNo`, `VoidAmount` Decimal(18,2) |
| `/PreAuth/Enquiry` | `MerchantID`, `MerchantOrderNo`, `Versioning` (`2`) |

### Pre-Auth responses (two shapes)

Shape A, Create redirect (GET query) and Enquiry (JSON): `SCSign` is a field. Shape B, Capture / Void / Capture-Void (JSON): no `MaskedPAN`, `SCSign` in the HTTP response header only. Enquiry responses also carry a second `SCSign` response header = HMAC of the entire JSON body.

Fields, in order (Shape B omits `MaskedPAN`):

`RespCode` String(30) M (Appendix 1); `RespMessage` String(2000) M (samples use key `RespDesc` in shape A); `PaymentType` String(30) M (Appendix 3); `MerchantID` M; `TxnRefNo` M; `CurrencyCode` M; `TxnAmount` Decimal(18,2) M; `MerchantOrderNo` M; `MerchantOrderDesc` String(2000) M; `MerchantPaymentNo` M; `MerchantRef1..3` O; `CustReference` O; `CustName` O; `CustEmail` O; `CustPhoneNo` O; `CustAddress1` O; `CustAddress2` O; `CustCountryCode` O; `CustAddressState` O; `CustAddressCity` O; `CustPostcode` O; `PaymentMethod` M; `MaskedPAN` (shape A only); `Versioning` M; `TxnDateTime` Datetime M (sample `2022-06-13 15:49.12`); `SCSign`.

## 13 to 16. Recurring (tokenised subscriptions)

### Create: `POST /CreateCheckout/Recurring`

JSON body, `SCSign` header = HMAC of the raw JSON body. Then redirect the customer to `Result.CheckoutUrl`.

| Field | Type (Len) | M/O | Notes |
| --- | --- | --- | --- |
| `MerchantID` | String (50) | M | |
| `Frequency` | Integer | M | Number of cycles; `0` = until terminated or auto-terminated by max retry |
| `FrequencyPeriod` | String (15) | M | `DAILY`, `WEEKLY`, `MONTHLY`, `YEARLY` |
| `FrequencyInterval` | Integer | M | With MONTHLY: `1` monthly, `2` every 2 months, `3` quarterly |
| `RetryDays` | Integer | O | Reserved, must be `0` |
| `MaxContinuouslyFailedCycles` | Integer | O | Reserved, must be `0` |
| `EndDate` | String | O | `yyyy-MM-dd`, blank for none; must be on/after start date |
| `ChargeDayOfMonth` | Integer (1-31) | O | MONTHLY only |
| `AllowRecharge` | Boolean | O | Default true |
| `RechargeWindowDays` | Integer | O | Default 3 |
| `MaxRechargePerCycle` | Integer | O | Default 3 |
| `SkipFailedCycle` | Boolean | O | Default true |
| `SuspendOnHardDecline` | Boolean | O | Default true |
| `NotifyMerchantOnFailure` | Boolean | O | Default true |
| `AcqMaxChargeAmount` | Decimal (18,2) | O | Acquirer mode; not below `RecurringAmount`; 0 = none |
| `AcqMaxTotalAmount` | Decimal (18,2) | O | Acquirer mode; not below `RecurringAmount x Frequency`; 0 = none |
| `CurrencyCode` | String (5) | M | |
| `RecurringAmount` | Decimal (18,2) | M | |
| `RecurringStartDate` | String | M | `yyyy-MM-dd`, today or later |
| `MerchantOrderNo` | String (50) | M | Tracks the plan and its cycles |
| `MerchantOrderDesc` | String (1000) | M | |
| `MerchantRef1..3` | String (100) | O | |
| `CustName` | String (150) | M | |
| `CustEmail` | String (150) | M | |
| `CustPhoneCountryCode` | String (5) | O | e.g. `60` |
| `CustPhoneNo` | String (50) | O | |
| `CustAddress1`, `CustAddress2` | String (100) | O | |
| `CustAddressPostcode` | String (15) | O | |
| `CustCountryCode` | String (10) | O | |
| `CustAddressState`, `CustAddressCity` | String (150) | O | |
| `RedirectUrl` | String (1000) | M | |
| `IsSendReceiptToMerchant` | Boolean | O | Default false |
| `IsSendReceiptToCustomer` | Boolean | O | Default false |
| `Versioning` | Integer | O | `2` |

Response (JSON, NOT signed): `RespCode` String(50) M (Appendix 1), `RespMessage` String(2000) M, `Result` Object C (only on success) with `TxnRefNo`, `CheckoutUrl`, `RecurringGuid`, `RecurringMode` (sample `GATEWAY`).

### Redirect: GET to `RedirectUrl`

Fields in sign order: `RecurringStatus` Integer (Appendix 4), `MerchantID`, `MerchantOrderNo`, `Currency` (note: not `CurrencyCode`), `RecurringAmount`, `TxnRefNo`, `Versioning` (`2`), then `SCSign` in the query string. Docs advise calling Enquiry after the redirect.

### Enquiry: `POST /Recurring/Enquiry`

JSON `{ MerchantID, MerchantOrderNo, Versioning: 2 }`, all M, `SCSign` header = HMAC of the raw body. IP whitelist required.

### Callback and Enquiry response

Server-to-server HTTP POST JSON to the callback URL kept by Share Commerce; `SCSign` in the HTTP header = HMAC of the whole JSON body.

| Field | Type (Len) | M/O/C | Notes |
| --- | --- | --- | --- |
| `RecurringStatus` | Integer | M | Appendix 4 |
| `RecurringMessage` | String (500) | C | Termination details |
| `MerchantID` | String (50) | M | |
| `MerchantOrderNo` | String (50) | M | |
| `RecurringAmount` | Decimal (18,2) | M | Sign sample serialises `15.00` as `15` |
| `NextPaymentDate` | String | C | `yyyy-MM-dd`, while ongoing |
| `MaskedPAN` | String (20) | M | |
| `PaymentTransactionList[]` | Array | | Executed cycles |
| `.RecurringDate` | String (15) | M | `yyyy-MM-dd` |
| `.TxnStatus` | Integer | M | Appendix 5 |
| `.TxnRefNo` | String (50) | C | |
| `.Cycle` | Integer | M | Cycle number |

## Signature (SCSign)

Algorithm everywhere: `hex(HMAC_SHA256(key = SecretKey, message = signString))`, lowercase hex.

### Rule A: concatenated values (form-style requests and query/JSON results)

Used by: `/payment`, `/seemlesspayment`, `/PreAuth`, `/CreateCheckout/PreAuth` requests; payment redirect and callback; Pre-Auth responses (both shapes); Recurring redirect.

Concatenate the field values in table order with no separator and no keys. Empty optional fields contribute an empty string. Exclude `SCSign`.

`PaymentMethod` and `Versioning` are included only if the request carries them. The Payment Request page states: Versioning omitted/0/1 means neither is in the string; `2` means only `Versioning`; `3` or higher means both. Official plugins match: PrestaShop and Magento send neither field and sign without them; WooCommerce 8.x and OpenCart 3/4 send both with `Versioning=7` and sign both. (WooCommerce 8.x builds the array as `..., RedirectUrl, Versioning, PaymentMethod` with `PaymentMethod` empty, so the order makes no difference there. If you send a non-empty `PaymentMethod`, put it before `Versioning` as documented.)

Verification of incoming results, as every official plugin does it: iterate the received parameters in received order, skip `SCSign` (and your own routing params), concatenate the values, HMAC, compare.

```text
// request
fields = [MerchantID, CurrencyCode, TxnAmount, MerchantOrderNo, MerchantOrderDesc,
          MerchantRef1, MerchantRef2, MerchantRef3, CustReference, CustName, CustEmail,
          CustPhoneNo, CustAddress1, CustAddress2, CustCountryCode, CustAddressState,
          CustAddressCity, RedirectUrl, PaymentMethod, Versioning]
signString = join("", raw string value of each field)      // "" for empty optionals
SCSign = lowercase_hex(HMAC_SHA256(SECRET_KEY, signString))

// redirect (query) or callback (JSON)
signString = join("", value for (key, value) in received_params_in_order if key != "SCSign")
ok = constant_time_equal(lowercase_hex(HMAC_SHA256(SECRET_KEY, signString)), received.SCSign)
```

Gotchas:

- Values are signed raw (unencoded): the email is `tomt@email.com` and the URL `https://...` in the sign string even though the GET URL encodes them.
- Keep amount text exactly as sent/received (`101.50`). For JSON callbacks, read the raw body and extract values without float conversion (PHP `json_decode` turns `101.50` into `101.5`; decide with staging traffic which form Share Commerce signs).
- The docs page's sample strings contain a hidden U+200B between `https` and `://`. Copying them breaks your hash.

### Rule B: raw JSON body (JSON server calls)

Used by: `/enquiry` (assumed, see section 5), `/Payment/Void`, `/PreAuth/Void`, `/PreAuth/Capture`, `/PreAuth/Capture/Void`, `/PreAuth/Enquiry`, `/CreateCheckout/Recurring`, `/Recurring/Enquiry` requests; Recurring callback and Enquiry response; the extra header on Pre-Auth Enquiry responses.

`SCSign = hex(HMAC_SHA256(SECRET_KEY, exact_body_bytes))`, sent as HTTP header `SCSign`. Serialise once, sign those bytes, send those bytes. For incoming JSON, HMAC the raw body before parsing.

Exception: `/CreateCheckout/PreAuth` is JSON but uses Rule A with `SCSign` inside the body.

### Verified test vectors (docs example key `mSuE3Ttn5B8vJhe5ncMutMLV`)

| Case | Sign string | SCSign | Status |
| --- | --- | --- | --- |
| Payment request, no `PaymentMethod`/`Versioning` | `MID00001MYR101.50POST0001perfumemref1mref2mref3ctRefIC111TomTomtomt@email.com01938668396add1add2MYsemenyihselangorhttps://www.MerchantRedirectURL.com` | `7cc5dd56354aeddb89a4123b2f163d87bbade4f79f59b36105475fdc218f4930` | verified |
| Payment request, `Versioning=2` | same + `2` | `b8feaa620ce67fa176ccf87e8b483f1ce8af02d51d131bce77f6b4f84431bdf2` | verified |
| Payment request, `CARD` + `6` | same + `CARD6` | docs say `9a07b085...a59a7bc4`; recomputes to `ad3c62b6bd7bcf35fc87b9e27485fff8c2e088d2c6d383beeeb08a3086cd9d20` | does not reproduce |
| Payment callback/redirect sample | `00SuccessMID000013262873184MYR101.50POST0001perfumemref1mref2mref3ctRefIC111TomTomtomt@email.com01938668396add1add2MYsemenyihselangorCARD545301XXXXXX12346` (no `CustPostcode`) | `3bc0f71376f33e9f7439df71b6ff0f8a2a26eb589442302368ee6c90c22f23c8` | verified (matches the sample URL/JSON SCSign) |
| Payment callback, docs string with `57000` | as above with `57000` before `CARD` | docs say `f5e3093a...c3c8755e` | does not reproduce |
| Payment Void | `{"MerchantID":"MID0001","MerchantOrderNo":"POST0001","TxnRefNo":"8712893729013","VoidRemarks":"Void Transaction"}` | `db4ae5622955a907f3ed8e44949c668becf4a4c79f335a6849082bbe75ca62a0` | verified |
| Pre-Auth Capture | `{"MerchantID":"MID0001","MerchantOrderNo":"POST0001","MerchantPaymentNo":"AHS92738","PreAuthTxnRefNo": "8712893729013","CaptureAmount":25.00}` (note the space after `PreAuthTxnRefNo":`) | `fdcc76af29a9468b7ff696b8d7b5992a460548d407ffa97443875855325dc574` | verified |
| Pre-Auth Enquiry | `{"MerchantID":"MID0001","MerchantOrderNo":"POST0001","Versioning":2}` | `d275f570cbf67de8766e113e1988c4ceefa98999ddc460c2b51c30d909048b59` | verified |
| Pre-Auth request (`/PreAuth`, CreateCheckout) | `MID00001MYR101.50POST0001perfumeAHS917123mref1mref2mref3ctRefIC111TomTomtomt@email.com01938668396add1add2MYsemenyihselangor12345https://www.shop.com2` | docs say `f4f69767...47511da8` | does not reproduce |
| Pre-Auth response shape A | `00SuccessPREAUTHMID000013262873184MYR101.50POST0001perfumeAKSP172389123mref1mref2mref3ctRefIC111TomTomtomt@email.com01938668396add1add2MYsemenyihselangor12345CARD545301XXXXXX123422022-06-13 15:49.12` | `ec0a47f774a003842042d8be1a448ce58f026cdd6c9bb087960f400a7a65ff18` | verified |
| Pre-Auth response shape B | `00SuccessPREAUTHMID00013262873184MYR100.00POST0001PerfumeAKSP172389123mref1mref2mref3ctRefIC111TomTomtomt@email.com01938668396add1add2MYsemenyihselangor12345CARD22022-06-13 15:49.12` | `05401db0cb5848b8bf2cf0d66481a02a7585e36c081c47c4683ca63ed1e5f967` | verified |
| Recurring Enquiry | `{"MerchantID":"MID0001","MerchantOrderNo":"RecurringPayment_001","Versioning":2}` | `56f3eb548ac6859f0fd05f5c9a020781377289dc133a7ce2777311c5739c5396` | verified |
| Recurring redirect | `1MID001RecurringPayment_001MYR15.008F4870839A761AB84B2` | `00bd4764959a35123e15487b27a85492d67f057e6afba2bb13a1ff83ccb697ed` | verified |
| Recurring callback | the compact JSON on the page (`"RecurringAmount":15`) | `8828efaa7921e2f08c624f05d07470b8c6b5be641add9774b85f5b21452e9e31` | verified |
| Recurring create | the compact JSON on the page (email `hello@aa.com`, normal spaces) | `f5b893c4ce4c3ca7fdc14343889b1815231dec59579c24d41ae9b80736463537` | verified |
| Seemless (key `o4q1VRXDfBok71119uQjtS1f`) | `BRYAN57328MYR1.50M1734657848Perfumemref1mref2mref3ctRefIC111Bryan NGhttps://httpbin.org/anything7` | `a82ebd04a085c92473abfb46887f12e3ad19d6289be6deb1243f3b0e1f5d2dfd` | verified |

Doc inconsistencies an integrator must know:

- The Seemless and Recurring pages say "sign with whole json string", but the Seemless worked example is Rule A (concatenated values). Follow the example (verified).
- The callback table lists `CustPostcode`, but the only reproducible callback sample hash excludes it. Verify over the fields actually received.
- `Versioning` value: tables say 7 (request) and 6 (response); samples use 6; plugins use 7. Use what Share Commerce confirms at onboarding; sign whatever you send.

## Status and Code Enums

Appendix 1, `RespCode`: `00` Success; `09` Pending; `99` Failed; `IE` Application error; `10` Voided. (CreateCheckout/PreAuth only: integer `1` success, `9` failed.)

Appendix 2, `PaymentMethod`:

| Category | Code | Description |
| --- | --- | --- |
| Credit / Debit Card | `CARD` | Card |
| Online Banking | `FPX` | FPX |
| E-Wallets | `ALIPAY` | Alipay |
| E-Wallets | `BOOST` | Boost |
| E-Wallets | `GRAB` | Grab Pay |
| E-Wallets | `MAYBANKQR` | Maybank QR |
| E-Wallets | `MCASH` | MCash |
| E-Wallets | `PRESTO` | Presto |
| E-Wallets | `RAZER` | Razer |
| E-Wallets | `SHOPEE` | Shopee |
| E-Wallets | `TNG` | Touch N' Go |
| E-Wallets | `WECHAT_CN` | WeChat (China) |
| E-Wallets | `WECHAT_MY` | WeChat (Malaysia) |
| Buy Now Pay Later | `BNPL_MYIOU` | MyIOU |

`HOSTED_CARD` also appears in the Seemless result sample. DuitNow QR and FPX bank codes: not listed in official docs (checked 2026-10-21).

Appendix 3, `PaymentType`: `PAY`, `REFUND`, `VOID`, `PREAUTH`, `PREAUTH_VOID`, `CAPTURE`, `CAPTURE_VOID`.

Appendix 4, `RecurringStatus`: `0` Pending for tokenize payment; `1` Recurring ongoing; `2` Failed; `8` Recurring completed; `9` Recurring terminated.

Appendix 5, `TxnStatus` (recurring cycles): `0` Pending; `1` Success; `2` Voided; `3` Processing; `9` Failed.

Error codes beyond Appendix 1: not documented. One failure message is shown: `Invalid SCSign - expected HMAC-SHA256 of the signing string for this Versioning`.

## Settlement Rules

1. Insert local order `pending` with the `MerchantOrderNo`, amount, currency, environment.
2. On callback: HMAC check first (constant time), then require `MerchantID` = yours, `MerchantOrderNo` known, `TxnAmount` and `CurrencyCode` equal to stored, `RespCode == "00"`.
3. Mark paid once; store `TxnRefNo`, `PaymentMethod`, `MaskedPAN`. Unique key on `MerchantOrderNo`.
4. `09` stays pending; requery with `/enquiry` later. `99`/`IE` mark failed only if not already paid.
5. Redirect: verify the signature, show status, but only settle via the same idempotent function, never on redirect alone.
6. Reply `OK` with HTTP 200 to the callback after durable processing.

## Terminal Payment API (in-store, out of online scope)

`POST http://[terminal_ip_address]:8888`, `Content-Type: application/json`, LAN only, listener active only on the terminal main page. `TransactionType` values: `1` Sale, `2` Void, `3` Settlement, `4` Pre-Authorization, `5` Sale Complete, `6` Pre-Authorization Cancel, `7` E-Wallet Sale, `8` E-Wallet Void, `9` E-Wallet Settlement, `10` Void Sale Complete. Example request `{"TransactionType":"1","TransactionAmount":"1.00"}`; response includes `ResponseCode` (`00` Approved), `TransactionMID`, `TransactionTID`, `TransactionRRN`, `TransactionApprovalCode`, `TransactionInvoice`, `TransactionCardNo` (masked), `TransactionDateTime` (`yyyyMMddHHmmss`). Request keys per type: Sale/PreAuth/E-Wallet Sale: `TransactionType`, `TransactionAmount`; Void/Void Sale Complete/PreAuth Cancel: `TransactionType`, `TransactionInvoice`; Sale Complete: adds `TransactionRRN`, `TransactionApprovalCode`, `TransactionInvoice`; E-Wallet Void: `TransactionType`, `TransactionRefId`; Settlements: `TransactionType`. A separate terminal docs site exists at `https://devdocsterminal.share-commerce.com/` (not covered here).
