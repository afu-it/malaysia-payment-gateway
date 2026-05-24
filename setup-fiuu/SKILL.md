---
name: setup-fiuu
description: Set up, build, debug, review, and explain Fiuu payment integrations using official Fiuu API specs, Seamless Integration, Inpage Checkout, Mobile XDK, SDKs, and shopping cart plugin references. Use when working with Fiuu hosted payment pages, payment requests, vcode/skey verification, Return URL, Notify/Notification URL, Callback URL, IPN ACK/retry behavior, payment status requery, refunds, captures, recurring/token payments, channel status, mobile Flutter/Android/iOS/React Native XDK, JavaScript Seamless Integration, Inpage Checkout, WooCommerce, Magento, OpenCart, WHMCS, Shopify, or Fiuu account/dashboard setup.
---

# Fiuu

Use this skill for Fiuu payment gateway integration work.

## Source

Read `references/fiuu-api.md` before giving factual API guidance or writing integration code. It contains source URLs, integration surfaces, environment hosts, endpoint roles, vcode/skey formulas, webhook/IPN guidance, mobile XDK facts, and loophole fixes.

Read `references/account-setup.md` when user asks how to register, which credentials are needed, where keys come from, or what dashboard/domain setup is required.

Read `references/source-map.md` when choosing a Fiuu GitHub repo, SDK, plugin, or mobile platform reference.

## Core Workflow

1. Identify environment: sandbox/UAT/dev/production and web, mobile, plugin, POS, or recurring flow.
2. Choose integration surface: Hosted Payment Page, Seamless Integration, Inpage Checkout, Mobile XDK, SDK, shopping cart plugin, Direct Server, recurring, refund/capture, status requery, or terminal/POS.
3. Create a local pending payment before sending the customer to Fiuu.
4. Generate request hash (`vcode`) server-side when required.
5. Configure and verify backend Notify/Notification URL and Callback URL. Use Return URL only for UX.
6. Verify response hash (`skey` or XDK checksum/VrfKey) server-side with the correct formula and secret.
7. Requery Fiuu status when needed, then settle only after amount, currency, order ID, merchant ID, status, and provider reference match.
8. Make settlement idempotent by storing Fiuu transaction ID, order ID, event/source, verification result, and processed status.

## Core Facts

- Main hosted payment request URL:
  - Production: `https://pay.fiuu.com/RMS/pay/{MerchantID}/{Payment_Method}`
  - Sandbox: `https://sandbox-payment.fiuu.com/RMS/pay/{MerchantID}/{Payment_Method}`
- API hosts in sandbox replace `api.fiuu.com` with `sandbox-api.fiuu.com`.
- Most Fiuu API requests use `application/x-www-form-urlencoded`, not JSON.
- Credentials can include Merchant ID, Verify Key, Secret Key, password/API credentials, and environment-specific sandbox/live values.
- Common web status codes: `00` success, `11` failed, `22` pending.
- Return URL is browser-facing and not reliable for fulfilment.
- Notify/Notification URL is the primary server-to-server webhook for payment status.
- Callback URL handles non-realtime/deferred status changes such as cash channels.
- IPN ACK can trigger retry behavior when enabled; do not acknowledge before durable recording.

## Working Rules

- Do not expose Verify Key, Secret Key, API password, or private signing inputs in browser/mobile code.
- Do not mark paid from Return URL, redirect, mobile result, or frontend event alone.
- Always verify Fiuu hashes on the backend and compare local stored payment data.
- For Seamless Integration, confirm domain registration/whitelisting and use Channel Status API for subscribed channels when channel selection is merchant-side.
- Do not iframe bank login/authorization pages; banks/channels can block cross-origin flows.
- Do not use Direct Server/Card integrations unless PCI-DSS scope and Fiuu approval are explicitly handled.
- For mobile XDK, send returned result/checksum to backend for verification, especially when using private Secret Key.
- If user asks for latest fees, channel availability, dashboard state, onboarding requirements, plugin compatibility, or account-specific enablement, verify live Fiuu docs/dashboard/support first.
