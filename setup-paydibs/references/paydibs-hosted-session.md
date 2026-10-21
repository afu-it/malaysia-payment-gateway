# Paydibs Hosted Payment Session (Card SDK)

Accept cards on your own checkout page without PAN or CVV touching your servers. Added in docs version 3.8. Checked 2026-10-21. Sources:

- Initiate session: `https://v3api-docs.paydibs.com/docs/hosted-session/initiate`
- Integration guide: `https://v3api-docs.paydibs.com/docs/hosted-session/integration-guide`
- SDK API reference: `https://v3api-docs.paydibs.com/docs/hosted-session/api-reference`
- SDK file: `https://v3api-docs.paydibs.com/HOSTED_SESSION_SDK/sdk/v1/sdk.js`
- Sign: `https://v3api-docs.paydibs.com/docs/sign-hashing/merchant-request-sign`, `.../merchant-response-sign`

## Flow

1. Server: `POST <payment URL>/PPG/create-pay-session` (JSON) to get `PaySessID`. Values must equal the later PAY request.
2. Browser: load `sdk.js`, `HostedSession.configure({ environment, merId, paySessId })`, `mount()`, await `ready`, then `updateSession()` when the customer clicks pay.
3. On `status === 'ok'`, your backend sends the normal PAY request (`paydibs-api.md`) with `PaySessId` set to the same session ID, then handles RURL, callback and QUERY as usual.

## Create Session

URL: `<payment URL>/PPG/create-pay-session`, POST. TLS 1.2 only in both environments.

Request, mandatory:

| Field | Type | Max | Notes |
| --- | --- | --- | --- |
| `MerchantID` | AN | 6 | |
| `MerchantPymtID` | ANS (space) | 20 | Unique; duplicates not allowed once processed to PAY |
| `MerchantTxnAmt` | N | 12(2) | `1.00` format |
| `MerchantCurrCode` | A | 3 | |
| `Sign` | ANS | 128 | Hosted session request Sign |

Optional:

| Field | Type | Max | Notes |
| --- | --- | --- | --- |
| `RcurIdc` | B | - | `true` for recurring; then `CustReferenceId` is required |
| `CustReferenceId` | ANS (space) | 100 | Customer reference for saved cards |
| `PageTimeout` | N | 4 | Controls `PaySessExpTm`. Default 600 seconds |

`CustReferenceId` and `RcurIdc` are stored in the session; do not send them again in PAY.

Response (`suc`, `code`, `msg`, `data`, `traceId`, `timeStamp`), `data` fields:

| Field | Type | Mandatory | Notes |
| --- | --- | --- | --- |
| `MerchantID` | AN 6 | Yes | |
| `MerchantPymtID` | ANS (space) 20 | Yes | |
| `MerchantTxnAmt` | N 12(2) | Yes | |
| `MerchantCurrCode` | A 3 | Yes | |
| `PaySessID` | ANS 36 | Yes | UUID |
| `PaySessExpTm` | DateTime | Yes | ISO 8601, e.g. `2026-05-25T17:36:50` |
| `Sign` | ANS 128 | Yes | Hosted session response Sign |
| `RcurIdc` | B | No | |
| `PaySessFailMsg` | ANS | No | Error message when session creation fails |

Sample response also echoes `CustReferenceId`.

Sign:

- Request: `MerchantPassword + MerchantID + MerchantPymtID + MerchantTxnAmt + MerchantCurrCode + CustReferenceId` (empty if not sent). Doc example `C7(7B3N8MIMEONERtest0525051.00MYRTEST0524CCE` gives `b497585e9652dcb1...6078f28`.
- Response: `MerchantPassword + MerchantID + MerchantPymtID + MerchantTxnAmt + MerchantCurrCode + PaySessID + PaySessExpTm`. Doc example `C7(7B3N8MIMEONERtest0525051.00MYR95f1334f-89dd-4fee-9d60-b88f641ddff32026-05-25T17:36:50` gives `24f6ac0ec0afeccc...218ac422`.

## Loading The SDK

- Download `sdk.js` from the integration guide and serve it from your own public folder over HTTPS. `mount()` throws `insecure_context` on plain HTTP (localhost counts as secure).
- `environment` selects the iframe origin hardcoded in `sdk.js`: `stg` = `https://stgapi.paydibs.com`, `prod` = `https://v3api.paydibs.com` (iframe path `/PPG/hosted-payment-iframe`). Allow these in your CSP `frame-src`.

```html
<div id="card-fields"></div>
<button id="pay-btn">Pay</button>
<script src="/sdk.js"></script>
<script>
  const payment = HostedSession.configure({
    environment: 'stg',            // 'prod' | 'stg'
    merId: '<MERCHANT_ID>',
    paySessId: '<PAY_SESSION_FROM_BACKEND>',
  });
  payment.mount('#card-fields');
  payment.ready.then(() => {
    payment.onError((err) => console.error(err.code, err.message, err.recoverable));
  });
  document.getElementById('pay-btn').addEventListener('click', async () => {
    const result = await payment.updateSession();
    if (result.status === 'ok') {
      await fetch('/api/checkout/authorize', { method: 'POST' }); // backend sends PAY with PaySessId
    }
  });
</script>
```

## Methods

| Signature | Returns | Notes |
| --- | --- | --- |
| `HostedSession.configure(options)` | `HostedSession` | Static factory; throws if `environment` or `merId` missing |
| `.mount(target)` | void | CSS selector or Element; throws if not configured, target missing, or insecure context |
| `.unmount()` | void | Removes iframe, fails in-flight calls, clears subscribers; instance unusable afterwards |
| `.updateSession()` | `Promise<UpdateSessionResult>` | Validates and writes card details to the session |
| `.validate()` | `Promise<StateSnapshot>` | Validation only, no write |
| `.focus(field)` | void | Focus a `FieldName` |
| `.getState()` | `StateSnapshot` | Last known state, synchronous |
| `.ready` | Promise (property) | Resolves after handshake; attach `on*` after it |

## configure() Options

| Name | Type | Required | Default |
| --- | --- | --- | --- |
| `environment` | `'prod' \| 'stg'` | Yes | `'prod'` |
| `merId` | string | Yes | - |
| `paySessId` | string | No (needed for `updateSession()`) | undefined |
| `debug` | boolean | No | false |
| `handshakeTimeoutMs` | number | No | 60000 |
| `interaction.autoAdvance` | boolean | No | false |
| `interaction.validateOn` | `'change' \| 'blur' \| 'submit'` | No | `'blur'` |
| `interaction.clearValueOnModeChanged` | boolean | No | false |
| `theme` | `ThemeOption` | No | - |
| `layout` | `'stacked' \| 'compact'` | No | `'compact'` |
| `fieldOrder` | `FieldName[]` (all four, no duplicates) | No | `['cardholderName','cardNumber','expiry','cvc']` |
| `labelPosition` | `'top' \| 'inline'` | No | `'top'` |
| `modeSelector` | `'toggle' \| 'tabs'` | No | `'toggle'` |
| `modeLabels` | `{ saved?, newCard? }` | No | `{ saved: 'Saved cards', newCard: 'New card' }` |

Invalid optional values are dropped silently (warning in debug mode) and the default is used.

## Events (call after `ready`; each returns an unsubscribe function)

| Method | Payload | Fires when |
| --- | --- | --- |
| `onChange(fields, cb)` | `{ complete, empty, error? }` | Value or validation of a watched field changes |
| `onFocus(fields, cb)` / `onBlur(fields, cb)` | `{}` | Field gains or loses focus |
| `onValidityChange(fields, cb)` | boolean | Validity flips |
| `onEmptinessChange(fields, cb)` | boolean | Empty state flips |
| `onCardBrandChange(cb)` | `CardBrand` | Detected brand changes |
| `onError(cb)` | `{ code, message, recoverable }` | Iframe reports an error |
| `onSavedCardSelect(cb)` | `{ tokenId }` | User picks a saved card |
| `onSavedCardDelete(cb)` | `{ tokenId, ok, reason? }` | User deletes a saved card (server-side unbind result) |
| `onSaveCardChange(cb)` | `{ saveCard }` | "Save this card" toggled |
| `onModeChange(cb)` | `{ mode, selectedSavedCardId, savedCardCvvComplete }` | Saved/new card view switch; also once after handshake |

`fields` is one `FieldName` or an array; the callback runs per field.

## Types

- `FieldName`: `'cardNumber' | 'expiry' | 'cvc' | 'cardholderName'`
- `CardBrand`: `'visa' | 'mastercard' | 'unknown' | null`
- `ModeEnum`: `'new' | 'saved'`
- `UpdateSessionResult`: `{ status, errors?: Record<string,string>, sourceOfFunds?: unknown }`
- `UpdateSessionStatus`: `ok`, `fields_in_error`, `system_error`, `session_invalid`, `session_expired`, `unsupported`
- `StateSnapshot`: `{ mode, brand, isValid, newCard: { complete, errors }, savedCard: { selectedSavedCardId, cvvComplete, cvvError } }`
- `ThemeOption`: `colors { primary, background, text, border, error, success }`, `typography { fontFamily: 'Lato'|'Poppins'|'system', fontSize 12 to 18 px }`, `borderRadius` 0 to 24 px

Error codes: see `paydibs-codes.md` (Hosted Session SDK Codes). Non-recoverable errors (e.g. `handshake_timeout`, `iframe_destroyed`) leave the iframe unusable: create a new session and remount.

Card brands in the type list are Visa and Mastercard only; other schemes are not documented for the SDK (checked 2026-10-21).
