# RinggitPay Account Setup

Use this reference when explaining RinggitPay registration, credentials, callback URL registration, UAT vs production, UAT sign-off, payment methods, settlement timing, and pricing.

## Source URLs

- `https://ringgitpay.biz/`
- `https://ringgitpay.biz/sign-up/`
- `https://ringgitpay.biz/support/`
- `https://ringgitpay.biz/instant-collect/`
- `https://ringgitpay.biz/recurring-billing/`
- `https://ringgitpay.biz/pricing/`
- `https://biller.ringgitpay.biz/`
- `https://wordpress.org/plugins/ringgitpay/`
- RinggitPay Integration and Testing spec v1.13 (PDF, see `source-map.md`)
- RinggitPay Direct Debit API guide v1.8 (PDF, see `source-map.md`)

## Product Positioning

RinggitPay is a Malaysian payment platform (company named in the spec as RinggitPay Sdn Bhd; the website names Ascertain Technologies as parent company). Website products: payment gateway, payment links, digital invoices (e-invoice), payment pages (booking, subscription, checkout), recurring billing (auto debit, e-Mandate, subscription plan), and smart recovery (reminders, recovery workflow, customer analytics).

There is no public developer portal. Integration specs are given to merchants by the RinggitPay team after onboarding.

## Registration

1. Fill the "RinggitPay Signup Form" at `https://ringgitpay.biz/sign-up/`: name, company email, contact number, industry, designation, company name, company SSM number, company size, existing payment provider, referral source.
2. Support FAQ: after sign-up "Our account manager will get in touch with you shortly"; a dedicated account manager assists after onboarding; packages are quoted per chosen solutions.
3. Document uploads are not listed on the website (checked 2026-10-21). Expect the account manager to request company documents; do not assume a list.
4. After merchant and application registration, ask the RinggitPay team for: portal username and password, `AppId`, `RequestKey`, `ResponseKey` [spec section 3].
5. Give RinggitPay your `APIURL` (server-to-server) and `RedirectURL` (browser) for each environment. No IP address or port number in either.

Contact (support page): careline@ringgitpay.com, +6017 9898 253. The spec refers technical queries to the "RinggitPay App Team".

## Where Credentials Live

- Portal: `https://biller.ringgitpay.biz/` (the "Login" link on the website).
- RequestKey and ResponseKey: Portal > Merchant Profile, scroll to "RequestKey" / "ResponseKey", or use the keys sent by the RinggitPay App Team. Ask RinggitPay to reset a compromised key.
- Direct Debit API credentials (authorization link, username, password, client id, client secret, scope) and the bank-issued `ddMerchantId` (`DDxxxxxxxx`) come from RinggitPay.

Suggested environment variable names:

```text
RINGGITPAY_ENV=uat                      # or production
RINGGITPAY_APP_ID=<APP_ID>
RINGGITPAY_REQUEST_KEY=<REQUEST_KEY>
RINGGITPAY_RESPONSE_KEY=<RESPONSE_KEY>
RINGGITPAY_PAYMENT_URL=https://ringgitpay.co/payment
RINGGITPAY_ENQUIRY_URL=https://ringgitpay.co/transactionenquiry
# Direct Debit (only if using e-Mandate)
RINGGITPAY_DD_BASE_URL=https://bills2u-api.ringgitpay.co
RINGGITPAY_DD_TOKEN_URL=https://identity.ringgitpay.co/connect/token
RINGGITPAY_DD_CLIENT_ID=<CLIENT_ID>
RINGGITPAY_DD_CLIENT_SECRET=<CLIENT_SECRET>
RINGGITPAY_DD_USERNAME=<USERNAME>
RINGGITPAY_DD_PASSWORD=<PASSWORD>
RINGGITPAY_DD_MERCHANT_ID=<DD_MERCHANT_ID>
```

Production values: `https://ringgitpay.com/payment`, `https://ringgitpay.com/transactionenquiry`, `https://bills2u-api.ringgitpay.biz`, `https://identity.ringgitpay.biz/connect/token`.

## UAT Vs Production

- UAT uses `ringgitpay.co` hosts with FPX and card simulators; production uses `ringgitpay.com` (payments) and `ringgitpay.biz` (Direct Debit API, identity).
- Keys are per application; confirm with RinggitPay whether UAT and production keys differ for your AppId (not stated in the spec).
- WooCommerce plugin: set "UAT/Production" to UAT for testing, then Production to go live; the readme warns to disable UAT mode after testing.

## Go Live (UAT Sign-off)

After UAT, ask RinggitPay for the sign-off checklist. Evidence required [spec section 11]:

1. Merchant checkout page shows currency and amount with two decimals.
2. FPX: screenshot with seller name, currency, amount; success (amount under RM100, Retail > SBI Bank A); failure (under RM100, Retail > SBI Bank B); simulator confirmations; B2B if applicable (2 pending screenshots, email references to RinggitPay for APPROVE/DECLINE, 2 final screenshots).
3. Card: screenshot with seller name, currency, amount; success and failure with the test cards.
4. Receipt screen with order id, amount, status, remarks if failed.
5. Response via RedirectURL/ReturnURL (screenshot) and via APIURL (log).
6. Transaction enquiry log or screenshot.
7. Email received from FPX/card platform.

## Payment Methods

- Plugin readme: FPX internet banking, Visa/Mastercard credit and debit cards; tags also list DuitNow and e-wallets.
- Support FAQ: direct debit, Visa/Mastercard cards, e-wallets GrabPay, Touch 'n Go, Boost.
- The v1.13 spec documents `rp_paymentMode` values `FPX`, `CC`, `DC`, `NA` only. E-wallet and DuitNow modes through the hosted API are not documented; confirm with RinggitPay before relying on them.
- Currency: MYR only (plugin FAQ).

## Settlement And Pricing

- Support FAQ: "You will get your payment credited in 2 days after the payment is initiated by the customer."
- Pricing: see `https://ringgitpay.biz/pricing/` (quote-based via an interest form). Do not hard-code fees.
- Refunds: no refund API or refund procedure is published; handle via the RinggitPay portal or account manager (confirm with RinggitPay).
