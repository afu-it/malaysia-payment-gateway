# Adyen Checkout Flows (Sessions, Hosted Checkout, Advanced)

Use this file to build the payment-creation side. Checked 2026-10-21. Field tables: `adyen-fields.md`. Method specifics: `adyen-payment-methods-my.md`.

## Source URLs

- `https://docs.adyen.com/online-payments/build-your-integration`
- `https://docs.adyen.com/online-payments/build-your-integration/sessions-flow` (Web, Drop-in and Components tabs)
- `https://docs.adyen.com/online-payments/build-your-integration/advanced-flow` (Web tabs)
- `https://docs.adyen.com/online-payments/build-your-integration/hosted-checkout` (page content read from the live HTML; it is not in `llms-full.txt`)
- `https://docs.adyen.com/online-payments/payment-result-codes`
- `https://docs.adyen.com/online-payments/build-your-integration/payment-result-codes` and `https://docs.adyen.com/online-payments/response-handling`
- `https://docs.adyen.com/online-payments/upgrade-your-integration/upgrade-to-checkout-api-v72`
- `https://docs.adyen.com/online-payments/web-best-practices`
- `https://docs.adyen.com/online-payments/release-notes` (Web environments, SRI hashes)
- Checkout OpenAPI v72: `https://github.com/Adyen/adyen-openapi/blob/main/json/CheckoutService-v72.json`

## Pick A Flow

| Flow | Server calls | Frontend | When |
| --- | --- | --- | --- |
| Hosted Checkout | `POST /sessions` with `mode: "hosted"` | none, redirect to `url` | Fastest. Adyen hosts the page. |
| Sessions flow | `POST /sessions` | Drop-in or Components with `session` | Default embedded integration. One server call. |
| Advanced flow | `POST /paymentMethods`, `POST /payments`, `POST /payments/details` | Drop-in or Components with callbacks | You need control over each step (custom logic between steps, own risk data, partial payments). |

In all flows the payment outcome you act on is the `AUTHORISATION` webhook (`adyen-webhooks.md`), not the browser result.

## Common Request Rules

- Base URL test: `https://checkout-test.adyen.com/v72`. Live: `https://<PREFIX>-checkout-live.adyenpayments.com/checkout/v72`. Source: `https://docs.adyen.com/development-resources/live-endpoints`
- Headers: `x-API-key: <API_KEY>`, `content-type: application/json`, `idempotency-key: <UUID_V4>` on every POST.
- `amount.value` is in minor units. MYR: RM 49.90 = `4990`. Source: `https://docs.adyen.com/development-resources/currency-codes`
- `reference`: your order id, required, max 80 characters (v72). GrabPay: max 32, `[A-Za-z0-9_-]`.
- `returnUrl`: max 1024 characters; Adyen appends query parameters; do not put personal data in it.
- `countryCode`: `MY` for Malaysian shoppers so local methods appear.
- `shopperReference`: required for tokenization; min 3 characters; no personal data.
- Recommended for risk and 3DS: `shopperEmail`, `shopperIP`, `billingAddress`, `browserInfo` (advanced flow), `channel: "Web"`, `shopperLocale` (Drop-in `locale` values: `https://github.com/Adyen/adyen-web/tree/main/packages/server/translations`), `lineItems`.
- `shopperStatement`: text on the shopper's bank statement (go-live checklist item).

## Hosted Checkout

Source: `https://docs.adyen.com/online-payments/build-your-integration/hosted-checkout`

1. `POST /sessions` with `merchantAccount`, `amount`, `reference` (min 3), `returnUrl`, `countryCode`, `mode: "hosted"`. Optional: `themeId`, `expiresAt` (default 1 hour, max 24 hours), `shopperLocale`, `lineItems`.
2. Redirect the shopper to response `url`.
3. Shopper returns to `returnUrl` with `sessionId` and `sessionResult`.
4. Optional UX check: `GET /sessions/{sessionId}?sessionResult=<value>` returns `status`: `completed`, `paymentPending`, `expired` (and `canceled` in the Sessions flow docs).
5. Fulfil from the `AUTHORISATION` webhook. The shopper can retry on the hosted page, so expect several `AUTHORISATION` webhooks per session (`additionalData.checkoutSessionId`). Fulfil once per order and check amounts.

Rules: the page expires at `expiresAt` or after too many attempts. From 2025-11-01 Hosted Checkout and payment links cannot be embedded in an iframe. Themes: Customer Area > **Pay by Link** > **Themes** (role "Hosted Checkout and Pay by Link Settings").

## Sessions Flow (Web Drop-in / Components v6)

Server:

```bash
curl https://checkout-test.adyen.com/v72/sessions \
  -H "x-API-key: <API_KEY>" -H "content-type: application/json" \
  -H "idempotency-key: <UUID_V4>" \
  -d '{
    "merchantAccount": "<MERCHANT_ACCOUNT>",
    "amount": { "value": 4990, "currency": "MYR" },
    "reference": "ORDER-10001",
    "returnUrl": "https://shop.example.my/checkout/return?orderId=ORDER-10001",
    "countryCode": "MY",
    "channel": "Web",
    "shopperLocale": "en-US"
  }'
```

Response: `id`, `sessionData`, `expiresAt` (default 1 hour after creation, max 24 hours), `amount`, `reference`, `returnUrl`, `mode`. Send only `id` and `sessionData` to the browser.

Change the amount before payment (v72): create the session with `payable: false`, then `PATCH /sessions/{sessionId}` with `sessionData` and the new `amount`.

Client:

```js
import { AdyenCheckout, Dropin } from '@adyen/adyen-web/auto';
import '@adyen/adyen-web/styles/adyen.css';

const checkout = await AdyenCheckout({
  session: { id: session.id, sessionData: session.sessionData },
  environment: 'test',            // live: 'live', 'live-us', 'live-au', 'live-nea', 'live-in', 'live-apse'
  amount: { value: 4990, currency: 'MYR' },
  countryCode: 'MY',
  locale: 'en-US',
  clientKey: '<CLIENT_KEY>',      // test_... or live_..., not secret
  onPaymentCompleted: (result) => showResult(result.resultCode),
  onPaymentFailed: (result) => showResult(result.resultCode),
  onError: (error) => console.error(error.name, error.message),
});
new Dropin(checkout, {}).mount('#dropin-container');
```

- CDN alternative: `https://checkoutshopper-test.cdn.adyen.com/checkoutshopper/sdk/<VERSION>/adyen.js` and `.../adyen.css`, with the `integrity` hash from the release notes. Live CDN host follows the environment (for example `checkoutshopper-live-apse.cdn.adyen.com` for `live-apse`, Web v5.13.0 or later; APSE location-based endpoints are enabled through Support).
- `onError` `error.name`: `NETWORK_ERROR`, `CANCEL`, `IMPLEMENTATION_ERROR`, `ERROR`.
- Redirect methods (FPX, TnG, GrabPay, Boost, Atome, Alipay): shopper returns to `returnUrl?sessionId=...&redirectResult=...`. Create `AdyenCheckout` with `session: { id: sessionId }` and call `checkout.submitDetails({ details: { redirectResult } })`. If the shopper never comes back, the webhook still arrives.
- Server-side check: `GET /sessions/{sessionId}?sessionResult=...` gives `completed`, `paymentPending`, `canceled`, `expired`. This status is not updated after the fact; do not poll it.
- Tokenization in sessions: `shopperReference`, `storePaymentMethodMode` (`disabled` default, `askForConsent`, `enabled`), `recurringProcessingModel`. See `adyen-tokenization-links.md`.

## Advanced Flow

Source: `https://docs.adyen.com/online-payments/build-your-integration/advanced-flow`

1. `POST /paymentMethods` with `merchantAccount`, `countryCode: "MY"`, `amount`, `channel: "Web"`, `shopperLocale`. Optional `allowedPaymentMethods` / `blockedPaymentMethods` (types), `shopperReference` (returns `storedPaymentMethods`). Response `paymentMethods[]` (`type`, `name`, `brands`, `issuers` for FPX) and `storedPaymentMethods[]`. Pass the whole response to the client as `paymentMethodsResponse`.
2. Client: `AdyenCheckout({ paymentMethodsResponse, clientKey, environment, amount, countryCode, locale, onSubmit, onAdditionalDetails, onPaymentCompleted, onPaymentFailed, onError })`.
3. `onSubmit(state, component, actions)`: send `state.data` to your server, which calls `POST /payments`:

```json
{
  "merchantAccount": "<MERCHANT_ACCOUNT>",
  "amount": { "value": 4990, "currency": "MYR" },
  "reference": "ORDER-10001",
  "paymentMethod": { "type": "molpay_ebanking_fpx_MY", "issuer": "fpx_mb2u" },
  "returnUrl": "https://shop.example.my/checkout/return?orderId=ORDER-10001",
  "countryCode": "MY",
  "channel": "Web",
  "shopperIP": "<SHOPPER_IP>",
  "shopperEmail": "<SHOPPER_EMAIL>",
  "browserInfo": "<state.data.browserInfo>",
  "origin": "https://shop.example.my"
}
```

   Pass through `state.data` fields as they are (`paymentMethod`, `browserInfo`, `riskData`, `billingAddress`, `storePaymentMethod`, `checkoutAttemptId`). Then call `actions.resolve({ resultCode, action, order, donationToken })` with the response, or `actions.reject()` on failure.
4. If the response has `action`, Drop-in handles it: `redirect` (FPX and wallets), `threeDS2` (native 3DS2 challenge or fingerprint), `qrCode` (DuitNow), `await`, `sdk`, `voucher`.
5. `onAdditionalDetails(state, component, actions)`: send `state.data` to your server, which calls `POST /payments/details`. For a full-page redirect return, read `redirectResult` from the URL and call `POST /payments/details` with `{ "details": { "redirectResult": "<URL_DECODED_VALUE>" } }`.
6. Show the result from `resultCode`; fulfil from the webhook.

3DS2 (native, Advanced flow): send `authenticationData.threeDSRequestData.nativeThreeDS: "preferred"` (v69+), `channel: "Web"`, `browserInfo` (all sub-fields required for Web, from `state.data.browserInfo`), `origin` (`window.location.origin` of the Drop-in page; if missing or wrong the 3DS2 action cannot be handled), `returnUrl`, and `shopperIP` (required for Visa and JCB on web). A strict Content Security Policy can block the native challenge; without `nativeThreeDS` the 3DS flow uses a redirect. Source: `https://docs.adyen.com/online-payments/3d-secure/native-3ds2`

## Result Codes

Source: `https://docs.adyen.com/online-payments/payment-result-codes`

| `resultCode` | Meaning | Do |
| --- | --- | --- |
| `Authorised` | Approved. | Show success, wait for `AUTHORISATION` webhook to fulfil. |
| `Pending` | Final result not known yet (common for bank and wallet methods). | Show "processing", wait for webhook. |
| `Received` | Request received, final result later. | Same as Pending. |
| `Refused` | Declined. `refusalReason` gives the reason. | Ask for another method. Do not show `refusalReason` to the shopper. |
| `Cancelled` | Cancelled by shopper or before completion. | Ask to retry. |
| `Error` | Error during processing. | Show error, log `refusalReason`. |
| `RedirectShopper` | Shopper must be redirected (`action`). | Handle `action`. |
| `IdentifyShopper`, `ChallengeShopper` | 3DS2 fingerprint or challenge (`action`). | Handle `action`. |
| `PresentToShopper` | Show voucher or QR. | Handle `action`. |
| `AuthenticationFinished`, `AuthenticationNotRequired` | Standalone 3DS outcomes. | Continue per authentication-only flow. |
| `PartiallyAuthorised` | Partial payments order; part authorised. | Collect the rest. |

Do not update order state from `resultCode` alone.

## Payment Status Requery

Adyen Checkout has no "get payment by reference" endpoint. Options (checked 2026-10-21):

- Webhooks are the documented source of truth. Missed webhooks are retried for 30 days and can be re-sent from Customer Area > Webhooks > Troubleshoot.
- `GET /sessions/{sessionId}?sessionResult=` for session-based payments, at return time only.
- `GET /paymentLinks/{linkId}` for payment links (`active`, `completed`, `expired`, `paymentPending`).
- Customer Area > Transactions > Payments, and the Payment accounting / Settlement details reports for reconciliation.

## Version Notes (v72)

Source: `https://docs.adyen.com/online-payments/upgrade-your-integration/upgrade-to-checkout-api-v72`

- New stricter validation: `reference` max 80, `metadata` key max 20 and value max 80, `returnUrl` RFC 2396 and max 1024, `captureDelayHours` max 672, `shopperEmail` format checks.
- Enhanced scheme data moves from `additionalData` to the `enhancedSchemeData` object.
- `payable` and `PATCH /sessions/{sessionId}` are new.

## Browser And Platform Rules

Source: `https://docs.adyen.com/online-payments/web-best-practices`

- Do not render Drop-in in a cross-domain iframe (redirect methods break); a same-domain iframe is the minimum. Avoid WebViews in apps; use the system browser tabs.
- With server-side rendering, initialise `AdyenCheckout` only on the client.
- Local development: `http://localhost` is a secure context; client key allowed origins in test can be `http://localhost:<port>`; live requires HTTPS.
- Test on real phones (touch events, app switching for TnG, GrabPay, Atome).
