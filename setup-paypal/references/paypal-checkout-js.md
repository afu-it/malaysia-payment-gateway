# PayPal Checkout: JavaScript SDK v6 And v5

Sources: `/checkout`, `/checkout/integrate`, `/sdk/js/set-up`, `/sdk/js/reference`, `/v5-v6`, `/sdk/js/v5/configuration`, `/sdk/js/v5/reference`, `/subscriptions/integrate`, `/expanded/eligibility`. Checked 2026-10-21.

The browser SDK only renders buttons and collects approval. Orders are created and captured on your server with the secret. PayPal: "Never create orders in the browser", and the client should send a SKU, not an amount ("Your API should exchange a SKU for amount values").

## Which SDK

- v6 (current): `https://www.paypal.com/web-sdk/v6/core` (sandbox `https://www.sandbox.paypal.com/web-sdk/v6/core`). Use for new builds. Sample repo: `https://github.com/paypal-examples/v6-web-sdk-sample-integration`.
- v5 (still documented, widely deployed): `https://www.paypal.com/sdk/js?client-id=<CLIENT_ID>&currency=MYR`. Load it from PayPal; do not bundle or self-host.
- Do not load both on one page.

## v6 Initialize

```javascript
const sdkInstance = await window.paypal.createInstance({
  clientId: "<PAYPAL_CLIENT_ID>",        // safe in front-end code
  components: ["paypal-payments"],      // default
  pageType: "checkout",                 // checkout | product-details | cart | mini-cart | home
  locale: "en-MY",                      // optional BCP-47; auto-detected if omitted
  clientMetadataId: crypto.randomUUID() // optional
});
```

Components: `paypal-payments` (PayPal and Pay Later), `venmo-payments` (US only), `googlepay-payments`, `applepay-payments`, `fastlane`, `paypal-messages`, `paypal-guest-payments` (PayPal-hosted guest card form). `clientToken` replaces `clientId` only for Fastlane.

## v6 Eligibility

`await sdkInstance.findEligibleMethods({ currencyCode: "MYR", countryCode: "MY", amount: "49.90" })` then `.isEligible("paypal")`, `.getDetails(method)`. `currencyCode` defaults to `"USD"`, so pass `MYR`. Always check before showing a button.

## v6 One-Time Payment Session

```javascript
const session = await sdkInstance.createPayPalOneTimePaymentSession({
  onApprove: async ({ orderId, payerId }) => {
    const r = await fetch(`/api/paypal/orders/${orderId}/capture`, { method: "POST" });
    const result = await r.json();           // your server decides paid/not paid
  },
  onCancel: ({ orderId }) => { /* re-enable button */ },
  onError: ({ code, message }) => { /* show friendly message */ },
  // optional: onShippingAddressChange, onShippingOptionsChange, orderId (eager create), commit (default true), savePayment
});

button.addEventListener("click", async () => {
  const orderPromise = createOrderOnServer();   // resolves { orderId }; do not await before start()
  await session.start({ presentationMode: "auto" }, orderPromise);
});
```

`presentationMode`: `auto` (recommended; popup then modal fallback), `popup`, `modal` (WebView only), `redirect` (mobile; needs return and cancel URLs; with `autoRedirect: { enabled: false }` `start` returns `{ redirectURL }`), `payment-handler` (experimental). For redirect flows call `session.hasReturned()` on page load and `await session.resume()` to fire the original callbacks.

`onApprove` data: `orderId`, `payerId`, `paymentId` (legacy), `billingToken` (save flows). `commit: true` shows "Pay Now", `false` shows "Continue".

Web component: `<paypal-button type="pay|checkout|buynow|subscribe" class="paypal-gold|paypal-blue|paypal-white">`, CSS vars `--paypal-button-border-radius`, `--paypal-mark-border-radius`.

v6 `onError` codes: `ERR_INVALID_CLIENT_TOKEN`, `ERR_DOMAIN_MISMATCH`, `ERR_DEV_UNABLE_TO_OPEN_POPUP`, `ERR_FLOW_PAYMENT_HANDLER_BROWSER_INCOMPATIBLE`, `INSTRUMENT_DECLINED`, `NETWORK_ERROR`.

Other v6 sessions: `createPayPalSavePaymentSession`, `createPayPalSubscriptionSession` (`onApprove({ subscriptionId, payerId })`, `start(options, subscriptionPromise)` resolving `{ subscriptionId }`), `createPayLaterOneTimePaymentSession`, `createPayPalCreditOneTimePaymentSession`, `createVenmoOneTimePaymentSession`, `createGooglePayOneTimePaymentSession`, `createApplePayOneTimePaymentSession`, `createFastlane`, `createPayPalMessages`, `createCardFieldsOneTimePaymentSession`, `createCardFieldsSavePaymentSession`, `createPayPalGuestOneTimePaymentSession` (`onApprove`, `onComplete` required; `onWarn` for recoverable card errors).

Malaysia note: card fields (`createCardFields...`) belong to Expanded Checkout, which lists 37 countries without Malaysia. Pay Later, Credit and Venmo are not offered in Malaysia per the eligibility page. For Malaysia rely on `paypal-payments` and let `findEligibleMethods` decide what else appears.

## v5 Script Query Parameters

| Parameter | Values and notes |
| --- | --- |
| `client-id` | Required. `sb` works as a sandbox shortcut. |
| `currency` | Default `USD`. Set `MYR` for ringgit orders; must match the order `currency_code`. |
| `intent` | `capture` (default), `authorize`, `subscription` (with `vault=true`), `tokenize`. Must match the API `intent`. |
| `commit` | `true` (default, "Pay Now") or `false` ("Continue"). Must match `user_action`. |
| `vault` | `true` for subscriptions and billing agreements. |
| `components` | `buttons` (default), `marks`, `messages`, `funding-eligibility`, `hosted-fields`, `applepay`, `card-fields`, `hosted-buttons`. |
| `enable-funding` / `disable-funding` | `card`, `credit`, `paylater`, `venmo`, and EU methods. Do not use `disable-funding` to turn off advanced card fields. |
| `buyer-country` | Sandbox only. Do not send in production. |
| `locale` | Example values use underscore (`en_US`, `fr_FR`); the supported-locale table lists `ms-MY`, `en-MY` for Malaysia. PayPal recommends not setting it (auto-detected). |
| `merchant-id` | Partners only. Email addresses are deprecated as merchant IDs. |
| `integration-date`, `debug` | Backward compatibility date; debug only for testing. |
| `disable-card` | Deprecated. |

Script attributes: `data-csp-nonce`, `data-client-token`, `data-page-type`, `data-partner-attribution-id`, `data-user-id-token`.

## v5 Buttons

```html
<script src="https://www.paypal.com/sdk/js?client-id=<CLIENT_ID>&currency=MYR"></script>
<div id="paypal-button-container"></div>
<script>
paypal.Buttons({
  async createOrder() {
    const r = await fetch("/api/paypal/orders", { method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ cart: [{ sku: "<SKU>", quantity: "1" }] }) });
    return (await r.json()).id;           // PayPal order ID from your server
  },
  async onApprove(data) {                 // data.orderID
    const r = await fetch(`/api/paypal/orders/${data.orderID}/capture`, { method: "POST" });
    const result = await r.json();
  },
  onCancel(data) {},                      // data.orderID
  onError(err) {}
}).render("#paypal-button-container");
</script>
```

Other v5 options: `style` (`layout`, `color`, `shape`, `borderRadius`, `height` 25 to 55, `disableMaxHeight`, `label`, `tagline`), `message`, `displayOnly`, `onInit`/`onClick` (`fundingSource`), `onShippingChange`, `createSubscription` (`actions.subscription.create({ plan_id })`, needs `vault=true&intent=subscription`), `paypal.Buttons().isEligible()`, `paypal.Marks()`.

Note the case difference: v5 passes `data.orderID`; v6 passes `orderId`.

## Server Routes You Need

1. `POST /api/paypal/orders`: price the cart server-side, create local pending payment, call `POST /v2/checkout/orders` with `MYR`, return `{ id }`.
2. `POST /api/paypal/orders/:id/capture`: check the order belongs to the session and is still pending, call capture with `PayPal-Request-Id`, apply settlement checks (`paypal-orders.md`), return a minimal status to the browser.
3. `POST /api/paypal/webhook`: raw-body signature verification and idempotent settlement (`paypal-webhooks.md`).

Browser results (`onApprove`, redirect return) are UX only. The server capture response or a verified webhook decides payment state.

## Low-Code Options

- Payment Links and Buttons (dashboard or `/v1/checkout/payment-resources` API) and hosted buy buttons (`components=hosted-buttons`) need no server; reconcile with webhooks or Transaction Search. Sources: `/payment-links-buttons/overview`, `/guides/low-code-buy-button`.
