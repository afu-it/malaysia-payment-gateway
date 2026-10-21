# bayarlah Account Setup

Use this reference when explaining how to become a bayarlah merchant or developer, which credentials exist, where they come from, env var names, WeChat refund authorization, settlement timing and pricing. Checked 2026-10-21.

## Source URLs

- `https://www.bayarlah.my/` (official site, Beez Fintech Sdn Bhd)
- `https://www.bayarlah.my/merchant`
- `https://www.bayarlah.my/faqs`
- `https://www.bayarlah.my/contact`
- `https://doc.bayarlah.my/overview`
- `https://doc.bayarlah.my/preparation-before-integration`
- `https://doc.bayarlah.my/introduction-to-terms`
- `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/activation-interface`
- `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/refund-wechat-and-card`
- `https://doc.bayarlah.my/upay-web-api/list-of-integration-interfaces/redirect-payment-interface-wechat-only`
- `https://doc.bayarlah.my/event-order-notification`

## Product positioning

bayarlah is operated by Beez Fintech Sdn Bhd. The FAQ describes it as "a merchant acquiring platform which allows merchants to accept various e-wallet on a single QR Code Payment System", accepting "more than 30 banks and e-wallets". The developer docs describe B-to-C and C-to-B mobile payments over DuitNow and cross-border WeChat, with Alipay+ marked coming soon. Products on the site include Smart Store, Static QR Stand, Smart QR and Smart Terminal. The API is aimed at POS systems, kiosks, websites and WeChat Mini Programs.

## Becoming a merchant

- Any merchant running a legal business can apply; contact the sales team (FAQ).
- Sales contact on the official site: `sales@beezfin.com`, phone `+60 10 227 8488`, Mon to Fri 8.00 AM to 7.00 PM.
- Technical or business cooperation contact named in the docs overview: Mr. Chu, `hmchu@beezfin.com`.
- Required merchant documents: Not documented in official docs (checked 2026-10-21). Ask sales.
- Merchant web portal: "Merchant Login" on `https://www.bayarlah.my/merchant`; the docs name the merchant platform `https://s.bayarlah.net` for generating store activation codes.

## Getting API parameters (developer)

Send the bayarlah integration technical support team:

- Developer company name
- Developer company address
- Developer contact person
- Developer contact phone number
- Developer contact email (corporate email)
- Developer programming language

bayarlah then issues `vendor_sn`, `vendor_key` and `app_id`.

## Credentials and where they come from

| Credential | Source | Lifetime |
| --- | --- | --- |
| `vendor_sn`, `vendor_key` | bayarlah technical support | Fixed; signs activation only |
| `app_id` | bayarlah technical support | Fixed per application type |
| Activation `code` | Merchant platform `https://s.bayarlah.net`, per store | 7 days, limited uses; do not hard-code |
| `terminal_sn`, `terminal_key` | Returned by `/terminal/activate` | `terminal_key` rotates on each check-in (recommended daily) |
| bayarlah RSA public key | Your bayarlah technical liaison | For `notify_url` callback verification |
| Event push `app_id`, `app_secret` | bayarlah, with your push URL configured by bayarlah | For Event Order Notification verification |

## Suggested env vars

These names are suggestions, not bayarlah-defined:

```text
BAYARLAH_API_BASE=https://api.bayarlah.net
BAYARLAH_VENDOR_SN=<VENDOR_SN>
BAYARLAH_VENDOR_KEY=<VENDOR_KEY>
BAYARLAH_APP_ID=<APP_ID>
BAYARLAH_DEVICE_ID=<BRAND_STORE_POS_ID>
BAYARLAH_RSA_PUBLIC_KEY=<PEM_FROM_BAYARLAH>
BAYARLAH_EVENT_APP_ID=<EVENT_APP_ID>
BAYARLAH_EVENT_APP_SECRET=<EVENT_APP_SECRET>
BAYARLAH_NOTIFY_URL=https://example.com/payments/bayarlah/notify
```

Store `terminal_sn` and `terminal_key` in the database (or a secrets store you can update at runtime), not in static env vars, because check-in rotates the key. Pass the activation code once, at activation time, as an admin input.

## Environments

Only the production API domain `https://api.bayarlah.net` is published. A sandbox or test environment is Not documented in official docs (checked 2026-10-21). Ask bayarlah support for test merchant parameters. For WeChat WAP, bayarlah sales file a service-provider network application and a test-merchant network application for you.

## Payment method activation

- DuitNow QR (`payway "1000"`) and WeChat (`payway "3"`) are the documented API channels.
- WeChat WAP (redirect gateway) and Mini Program require bayarlah sales to file WeChat network applications; WeChat parameters must match the developer's.
- Mini Program plugin: apply for plugin permission on the WeChat plugin docs page linked from `https://doc.bayarlah.my/more-payment-function-interfaces/mini-program-plugin-api-wechat-only`.
- Event Order Notification: give bayarlah an HTTPS URL to configure.

## WeChat refund authorization

New WeChat official (designated) merchants cannot refund by default. To allow API refunds:

1. The merchant super administrator logs into `https://pay.weixin.qq.com`.
2. Product Center, My Authorized Products, find service provider "1238313502 Beez Fintech SDN BHD", Service Provider API Refund Authorization, click to authorize.

## Settlement

From the official FAQ ("Will the money go directly to the bank account?"):

- Monday to Thursday: within 24 hours (next day).
- Friday to Sunday: next Monday.
- Public holidays: next working day.
- The FAQ also states "Transaction hours are from 4 pm to 6 pm."

Confirm the schedule in your merchant agreement. A settlement report API is Not documented in official docs (checked 2026-10-21); the site says reports can be downloaded from the merchant web portal.

## Pricing

No public pricing page exists on `https://www.bayarlah.my` (checked 2026-10-21). Pricing is quote-only; contact `sales@beezfin.com`.
