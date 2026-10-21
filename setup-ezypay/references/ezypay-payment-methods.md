# Ezypay Payment Methods, Capture Page, Testing and Failure Codes

Sources: `/docs/payment-method-collection`, `/docs/payment-capture-page`, `/docs/payment-capture-page-styling`, `/docs/difference-between-hpp-and-pcp`, `/docs/hosted-payment-page`, `/docs/mobile-app-integration`, `/docs/for-ios`, `/docs/for-android`, `/docs/apple-pay-payment-integration`, `/docs/google-pay-payment-integration`, `/docs/payment-methods-payload`, `/docs/payment-method-management`, `/docs/cards-network-tokenization`, `/docs/vault-environment`, `/docs/test-payment-data`, sandbox pages, `/docs/failed-payment-responses` and the Malaysia card guide. Checked 2026-10-21.

## Malaysia methods

Visa and Mastercard cards; Apple Pay and Google Pay (Visa and Mastercard inside the wallet) are in beta. Network tokenisation is automatic for Visa and Mastercard. No 3DS, no CVV required. Pay Now supports cards, Apple Pay and Google Pay. New payment methods are not enabled by default on merchant accounts (changelog 6/12/2024): ask Ezypay to enable each one.

## Payment Capture Page (PCP)

An iframe hosted in the Ezypay vault environment that captures payment details and returns a token, keeping card data out of your systems (PCI scope reduction). Do not log or store card data yourself. It replaces the legacy Hosted Payment Page. It is not for South Korea (use the Korean Hosted Payment Page). It does not work on `localhost`: use an HTTPS local domain or an HTTPS tunnel.

Prerequisite: create the customer first.

URL:

```text
https://hosted-global-sandbox.ezypay.com/paymentmethod/embed?token=<TOKEN>&customerId=<CUSTOMER_ID>&submitbutton=true
production: https://hosted-global.ezypay.com/paymentmethod/embed?...
```

| Parameter | Req | Notes |
| --- | --- | --- |
| `token` | yes | Ezypay token, valid 60 minutes; regenerate the URL after expiry (see auth notes in `ezypay-api.md`) |
| `customerId` | no | links the new token to this customer automatically |
| `submitbutton` | no | `true` shows Ezypay's submit button; default `false` means you must send the create message yourself |
| `paymentMethods` | no | comma list from `BANK`, `VISA`, `MASTERCARD`, `AMEX`, `PAYTO`, `APPLEPAY_VISA`, `APPLEPAY_MASTERCARD`, `GOOGLEPAY_VISA`, `GOOGLEPAY_MASTERCARD`; default is all enabled for the merchant. For Malaysia use `VISA,MASTERCARD` plus wallet values if enabled |
| `termsandconditions` | no | URL of merchant terms, shown beside Ezypay terms |
| `setasprimary` | no | boolean, default `true` when `customerId` present |
| `paymentMethodManagement` | no | Apple Pay management URL override, full `https://` URL |
| `stylingId` | no | from Create a Styling |
| `feepricing` | no | marked "coming soon"; build your own fee display from the fees API meanwhile |
| `countryCode`, `successUrl`, `cancelUrl` | no | used in the mobile Custom Tabs and SFSafariViewController examples |

Embed:

```html
<iframe id="payment-frame" allow="payment" style="border:0;width:100%;min-height:256px"></iframe>
<script>
  const PCP_ORIGIN = "https://hosted-global-sandbox.ezypay.com"; // production: https://hosted-global.ezypay.com
  document.getElementById("payment-frame").src =
    PCP_ORIGIN + "/paymentmethod/embed?token=" + encodeURIComponent(serverToken) +
    "&customerId=" + encodeURIComponent(customerId) + "&submitbutton=true";

  window.addEventListener("message", (e) => {
    if (e.origin !== PCP_ORIGIN) return;            // origin check: your addition, not in the doc sample
    let r; try { r = typeof e.data === "string" ? JSON.parse(e.data) : e.data; } catch { r = e.data; }
    if (r.type === "success") {
      if (r.data && r.data.paymentMethodToken && r.link_status) { /* POST token to your server */ }
      else if (!r.link_status) { /* server calls Create Payment Method to link it */ }
    }
    if (r.type === "error") { /* show r.data.errorMessage or r.message */ }
    if (r.type === "resize") { /* adjust iframe height */ }
  });
</script>
```

Without `submitbutton=true`, your own button sends `frame.contentWindow.postMessage({ actionType: "create" }, PCP_ORIGIN)`.

Listener result example (field casing as documented):

```json
{ "type": "CARD", "paymentMethodToken": "c50c3335-...", 
  "card": { "accountHolderName": "test", "expiryYear": "31", "expiryMonth": "12", "type": "VISA",
            "last4": "1111", "first6": "411111", "countryCode": "AU", "tokenDetails": null,
            "origin": null, "accountType": null, "tokenized": false },
  "bank": null, "payTo": null, "wallet": null, "purpose": "PAYMENT_METHOD_TOKEN",
  "customerID": "5354a7e1-...", "link_status": true }
```

The doc sample reads `response.type` and `response.data.paymentMethodToken` while the shown payload is flat; mobile samples read `data.paymentMethodToken`, `data.displayName`, `data.errorCode`, `data.errorMessage`. Handle both shapes and always confirm on the server with Retrieve Payment Method.

Other rules:

- `allow="payment"` on the iframe is mandatory for Apple Pay.
- Your site must send the referrer (`Referrer-Policy: origin`) so Ezypay can return the token and verify Apple Pay.
- Validation: card number digits and format, expiry not past. Near-expiry cards are not flagged.
- Recommended iframe: width 100%, height at least 256 px with dynamic sizing.
- Mandatory integration items include showing Ezypay terms and all customer-paid Ezypay fees during payment method creation.

### Styling

Preview at `https://hosted-global-sandbox.ezypay.com/demo`, then `POST /v2/billing/styles` with `name`, `memo`, `cssVariables`, `cssClasses` (all required; send every variable even when changing one). Response `data.id` is the `stylingId`. Retrieve with `GET /v2/billing/styles/{styleId}`. Styles are per environment and can be reused across merchants.

Variables: `--fontFamily` (Google Fonts only), `--fontSizeBase`, `--fontSizeHeading1` to `--fontSizeHeading4`, `--spacingUnit`, `--borderRadius`, `--borderWidth`, `--borderColor`, `--colorPrimary`, `--colorTextTitle`, `--colorTextGeneral`, `--colorTextSubtitle`, `--colorPlaceholder`, `--colorInputValue`, `--colorInputBorder`, `--colorTextHeading1`, `--colorTextHeading2`, `--colorBackgroundPage`, `--colorBackgroundContainerPrimary`, `--colorBackgroundContainerSecondary`, `--colorBackgroundContainerHover`, `--colorDanger`.

Classes: `.InputLabel`, `.InputError`, `.SubmitButton`, `.RadioButton`, `.RadioButtonInner`, `.CheckboxContainer`, `.CheckboxInput`, `.CheckboxLabel`, `.AccordionItem`, `.AccordionContent`, `.AccordionTrigger`, `.AccordionTriggerLabel`, `.MethodAccordionItem`, `.MethodAccordionTrigger`, `.MethodAccordionTriggerLabel`, `.MethodAccordionTriggerDescription`, `.MethodAccordionContent`, `.InformationAccordionItem`, `.InformationAccordionTrigger`, `.InformationAccordionTriggerLabel`, `.InformationAccordionContent`, `.EzypayDetailsLabel`, `.EzypayDetailsContent`, `.EzypayEmail`, `.TermsLabel`, `.TermsLink`, `.SubmitResultsIcon`, `.SubmitResultsIconBackground`, `.SubmitResultsTitle`, `.SubmitResultsContent`. Some CSS properties are blocked.

### PCP versus legacy Hosted Payment Page

| Feature | Legacy HPP | PCP |
| --- | --- | --- |
| Links token to customer automatically | No | Yes |
| Merchant terms | No | Yes |
| Validates method against merchant settings | No | Yes |
| Built-in submit button | No | Yes |
| Styling | No | Yes |
| Google Pay | No | Yes |

Legacy HPP URL (being replaced): `https://vault-sandbox.ezypay.com/paymentmethod/embed?token=...&countryCode=MY&paymentMethods=VISA,MASTERCARD`, listener origin `https://vault-sandbox.ezypay.com`, then link the token with Create Payment Method.

## Mobile apps

| Platform | Option | Notes |
| --- | --- | --- |
| iOS | SFSafariViewController | simplest; Apple Pay, 3DS and external schemes work; requires `successUrl` and `cancelUrl` redirects back to the app |
| iOS | WKWebView | Apple Pay needs iOS 16+; set `WKNavigationDelegate` and `WKUIDelegate`; forward non-HTTP(S) schemes to `UIApplication.shared.open`; Google Pay popups in an overlay WKWebView, other popups in SFSafariViewController; add `" GOOGLE_PAY_SUPPORTED"` to the user agent; install a `ezypay` script message handler bridge (early, late-bind, or poll buffer) to receive `postMessage` |
| Android | Custom Tabs | Google Pay works, Apple Pay no; requires `successUrl` and `cancelUrl` |
| Android | WebView | enable JavaScript, DOM storage, multiple windows, `javaScriptCanOpenWindowsAutomatically`; `WebSettingsCompat.setPaymentRequestEnabled` for Google Pay; handle `onCreateWindow` popups for 3DS or wallets; intercept `intent://` and custom schemes; add `<queries>`; `@JavascriptInterface` bridge re-injected on every `onPageFinished` |

For a native submit button inject `window.postMessage({ actionType: "create" }, "<PCP origin>")`; `origin` is the PCP host only. Google Pay production in an app needs registration in the Google Pay and Wallet Console. Errors `OR_BIBED_15` (WebView setup incomplete) and `OR_BIBED_11` (app not approved, production only).

## Apple Pay and Google Pay

- Only supported in the PCP (not the legacy HPP) and Checkout. Apple Pay only shows in Safari or compatible Apple devices; not on Android.
- Recurring: PCP returns a reusable token for subscriptions or on-demand invoices. One-off: PCP or Checkout (Apple Pay checkout token cannot be reused; Google Pay checkout token can).
- Apple Pay domain verification: host Ezypay's file at `https://<domain>/.well-known/apple-developer-merchantid-domain-association` (download: `https://ezypay-documents.s3.ap-southeast-1.amazonaws.com/platformIntegrator/apple-developer-merchantid-domain-association`), public, no auth, no redirect or proxy, always reachable; register every domain in the flow (iframe host and parent site) with Ezypay as FQDNs. Missing a domain makes the sheet open and close.
- Apple Pay needs a management URL for recurring payments, given at onboarding or via `paymentMethodManagement`.
- Apple Pay payload: `card.first6` is always `999999`, `expiryMonth`/`expiryYear` are `00` (do not show expiry), `origin` `applepay`, `tokenDetails.tokenType` `applepay`, `friendlyDisplay` like `VISA 0121`. Display as `[Apple Pay] | Visa **** 0121`.
- Google Pay payload: `origin` `googlepay` (may be null if the PAN was typed), real expiry. Display as `[Google Pay] | Visa ****1234`. Merchants must accept Google Pay API terms and the acceptable use policy.

## Payment method payload

Generic fields: `paymentMethodToken`, `customerId`, `type` (`CARD`, `BANK`, `PAYTO`, `WALLET`, `QRPAYMENT`), one populated object of `card` / `bank` / `payTo` / `wallet` / `qrPayment`, `invalidReason` (`null` or `{code, description}`), `lastUsedOn`, `replacedPaymentMethodData`, `valid`, `primary`.

Card object: `accountHolderName`, `expiryYear` (2 digits), `expiryMonth`, `type` (`VISA`, `MASTER`, `AMEX`), `last4`, `first6`, `origin` (`null`, `applepay`, `googlepay`), `countryCode`, `tokenDetails` (`tokenType`, `friendlyDisplay`, `originalLast4`, `tokenFirst6`, `tokenLast4`, `tokenExpiry`), `tokenized` (network token in use), `fundingType` (`CREDIT`, `DEBIT`, `PREPAID`, `CHARGE`, `DEFERRED_DEBIT`, `PREPAID_RELOADABLE`, `PREPAID_NONRELOADABLE`, `UNKNOWN`, or `null` before first charge).

Minimum display: card type, last 4, expiry (not for Apple Pay).

## Sandbox test data

Cards (all regions except Korea; sandbox only, publicly documented):

| Case | Result | Card number | Brand | Region |
| --- | --- | --- | --- | --- |
| Paid | success | `4111111111111111` (or any valid 16-digit Luhn number) | Visa | any |
| Insufficient funds | payment failed | `4556175161745153` | Visa | any |
| Expired card | payment failed | `5500000000000004` | Mastercard | all except NZ |
| Invalid payment method | payment failed | `4868470817885831` | Visa | TH, ID, PH only |
| Customer to contact bank | payment failed | `4237990879556339` | Visa | TH, ID, PH only |
| Invalid details | payment failed | `5596398242217872` | Mastercard | all except TH, ID, PH |
| Invalid details | refund failed | `4024007157038612` | Visa | all except TH, ID, PH |

Apple Pay sandbox: needs an Apple sandbox tester account registered by Ezypay (share the email), a real Apple device, region set to Australia; invoice amount `111` simulates `INSUFFICIENT_FUND`. The Checkout page in sandbox offers "Successful" or "Failed" simulation. Real cards may be charged in some Apple Pay setups even in sandbox.

Google Pay sandbox: join the Google test card suite group; in the suite, Mastercard always fails with insufficient funds and Visa always succeeds.

Timing: sandbox cards update within seconds; bank debits update the next day.

Other regions' test data (AU and NZ bank, PayTo, GCash, Korea tokens, PromptPay amounts `1500` expired and `3000` cancel test, Tap to Pay amounts) are on `/docs/test-payment-data` and not needed for Malaysia.

## Failed payments (Malaysia cards)

Ezypay maps processor codes to a generic `failedPaymentReason.code`. Build logic on it. From the official Malaysia Mastercard and Visa guide (`http://www.ezypay.com/failed-payment-response-guide/my/mastercard-visa`, redirects to an ezypay.qwilr.com page):

| `failedPaymentReason.code` | Processor codes | Payment method status | Action |
| --- | --- | --- | --- |
| (success) | `00` | Valid | none |
| `insufficient_funds` | `51` | Valid | retry on a later day, not same day |
| `limit_exceeded` | `61`, `65` | Valid | retry later |
| `lost_or_stolen_card` | `04`, `07`, `41`, `43`, `58` | Invalidated | new payment method before retry |
| `expired_card` | `54` | Invalidated | new payment method |
| `bank_account_closed` | `46` | Invalidated | new payment method |
| `invalid_payment_method` | `12`, `14`, `15`, `57`, `62`, `78`, `92`, Invalid Card Number, PAN length mismatch, ATM only | Invalidated | new payment method |
| `customer_to_contact_bank` | `01`, `03`, `05`, `10`, `13`, `30`, `39`, `55`, `63`, `6P`, `70`, `75`, `79`, `82`, `84`, `85`, `1A`, `Z1`, `Z3`, `N7` | Valid | customer contacts bank or gives another method |
| `declined` | `52`, `53`, amount too low, interregional not allowed | Valid | ask for another method |
| `authority_withdrawn` | `R0`, `R1`, `R3`, recurring revocation | Invalidated | new authority or method |
| `suspected_fraud` | `59`, `83`, `88`, `93`, `FRAUD` | Invalidated | new payment method; no failed payment fee since 04/04/2025 |
| `bank_system_error` | `06`, `19`, `80`, `91`, `94`, `96`, `NO_RESPONSE_FROM_ACM`, `NO_RESPONSE_WITHIN_TIMEOUT`, `NOT_SET` | Valid | Ezypay resends for processing |

Invalidated means hard failure: auto rebill stops, `payment_method_invalid` fires, `paymentMethodInvalid` is true. Replace with `PUT .../paymentmethods/{old}/new`, then retry.

Recovery options: automatic retry (interval and max attempts set at onboarding, counted from the failure notification date, next date in `scheduledPaymentDate`), Ezypay failed payment email from `help@ezypay.com` (optionally with Pay Now), the Retry API, and the hosted Pay Now page (`payNowUrl`; cards, Apple Pay, Google Pay; no extra failed payment fee on Pay Now failures but transaction fees still apply). Resolving (write off or external payment) disables the Pay Now link. Mandatory integration items: manual retry, an automatic retry mechanism, write off, mark externally paid, customer notifications on success and failure.

## Vault and data retention

Card and bank details live only in the PCI DSS vault; Ezypay billing uses tokens with truncated data (name, first 6, last 4). Methods flagged invalid for 13 months are purged quarterly; methods unused for 24 months and not tied to active or billing-issue subscriptions or future invoices are removed; Delete Payment Method removes data permanently. For bulk migration of sensitive data Ezypay publishes a PGP public key (`/docs/public-keys`, key id `0594E635909D30ED`, fingerprint `D12C 67FF 0CD8 5F33 9019 8CD5 0594 E635 909D 30ED`).
