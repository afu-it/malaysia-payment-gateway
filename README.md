# Malaysia Payment Gateway Skills

![Made in Malaysia](https://img.shields.io/badge/Made%20in-Malaysia-red)
![Agent Skills](https://img.shields.io/badge/Agent-Skills-blue)
![Payment Gateway](https://img.shields.io/badge/Payment-Gateway-green)

Agent skills for implementing Malaysian payment gateway integrations with server-side checkout, verified webhooks, and idempotent settlement.

Made from Malaysia for developers building FPX, DuitNow QR, card, e-wallet, and local payment flows.

## Skills

| Skill | Use for |
| --- | --- |
| `malaysia-payment-gateway` | Gateway-agnostic planning, review, and security checklist |
| `setup-chip` | CHIP Collect checkout, purchases, callbacks, refunds, payment methods |
| `setup-curlec` | Curlec/Razorpay Orders API, Checkout.js, callbacks, webhooks |
| `setup-xendit` | Xendit Payment Sessions, Payment Requests, webhooks, xenPlatform |
| `setup-bayarcash` | Bayarcash API v2/v3, payment intents, callbacks, checksums |
| `setup-bcl` | BCL Pay, Payment Link, QR Terminal, Forms, Bayarcash-linked setup |
| `setup-toyyibpay` | toyyibPay categories, bills, callbacks, DuitNow QR, transaction checks |
| `setup-billplz` | Billplz collections, bills, X Signature callbacks, Payment Order payouts |
| `setup-fiuu` | Fiuu hosted payment, Seamless Integration, Inpage Checkout, XDK, callbacks |
| `setup-senangpay` | senangPay hosted/manual checkout, Direct API, callbacks, query status, refunds, plugins, recurring, payout, tokenisation, split settlement |
| `setup-hitpay` | HitPay Payment Request API, hosted checkout, Drop-In UI, webhooks, refunds, plugins, recurring billing, payouts |
| `setup-stripe-malaysia` | Stripe Malaysia Checkout, Payment Element, webhooks, Stripe CLI, MYR, cards, FPX |

## Install

Before installing:

1. Check current installed version.
2. Verify your agent supports Skills.
3. Update npm if needed.

Install from this repository:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway
```

The CLI can show available skills and let you select them interactively.

List available skills:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --list
```

Install all skills:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --all
```

Install one skill:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --skill setup-curlec
```

Install globally:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway -g
```

Verify CLI version:

```bash
npx skills@latest --version
```

## Recommended Setup

1. Install the overview skill first:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --skill malaysia-payment-gateway
```

2. Install the provider you need:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --skill setup-curlec
```

3. Ask your agent to inspect your project before coding.

4. Ask your agent to implement checkout, webhook verification, settlement, tests, and docs together.

## How To Ask Your Agent

Use the skill name in your prompt:

```text
Use $malaysia-payment-gateway to review my app payment flow and tell me what is missing before production.
```

```text
Use $setup-curlec to implement Curlec checkout, callback verification, webhook settlement, and tests in this Next.js project.
```

```text
Use $setup-chip to add CHIP Collect checkout and webhook handling. Do not unlock access from redirect alone.
```

```text
Use $setup-xendit to implement hosted checkout with Xendit Payment Sessions and idempotent webhook settlement.
```

```text
Use $setup-bayarcash to integrate Bayarcash payments with checksum/callback verification.
```

```text
Use $setup-bcl to set up BCL Pay and explain which parts are BCL-owned versus Bayarcash-linked.
```

```text
Use $setup-toyyibpay to integrate toyyibPay bills, callback hash verification, and idempotent settlement.
```

```text
Use $setup-billplz to integrate Billplz bills, X Signature verification, and idempotent settlement.
```

```text
Use $setup-fiuu to integrate Fiuu hosted checkout, Notify URL verification, Callback URL handling, and idempotent settlement.
```

```text
Use $setup-senangpay to integrate senangPay hosted checkout, Callback URL verification, query status reconciliation, and idempotent settlement.
```

```text
Use $setup-hitpay to integrate HitPay Payment Request checkout, webhook signature verification, status fallback, and idempotent settlement.
```

```text
Use $setup-stripe-malaysia to integrate Stripe Malaysia checkout, webhook verification, and idempotent settlement.
```

## Agent Rules

Ask the agent to:

- Read existing code before adding new files.
- Read provider `references/account-setup.md` before explaining registration, credentials, dashboard setup, or required API keys.
- Use provider `references/account-setup.md` for framework setup notes, required env vars, and deployment secrets where documented.
- When referral or invite guidance exists, present it as optional and transparent; never imply it is required unless provider registration actually requires an invite.
- Keep gateway secrets server-side.
- Create a local pending payment before provider checkout.
- Verify webhook/callback signatures.
- Re-fetch payment state from the provider when possible.
- Match provider, gateway reference, amount, currency, and local payment status before marking paid.
- Make settlement idempotent.
- Add tests for invalid signatures, duplicate events, wrong amount/currency, and successful settlement.

## Production Checklist

- Required environment variables are documented.
- Database migrations are included.
- Webhook URLs are configured in sandbox and live dashboards.
- Sandbox payment succeeds.
- Duplicate webhook replay is harmless.
- Fake success redirect cannot unlock paid access.
- Live and sandbox keys are separated.

## Contribute A New Gateway

Contributions are welcome. Use one folder per skill.

### Step 1: Pick a skill name

Use action-style names:

```text
setup-toyyibpay
setup-senangpay
setup-stripe-malaysia
```

Rules:

- Lowercase only.
- Use hyphens.
- Keep it short.
- Folder name must match `name:` in `SKILL.md`.

### Step 2: Create the folder

```bash
mkdir -p setup-yourgateway/references setup-yourgateway/agents
```

### Step 3: Add `SKILL.md`

Minimum shape:

```markdown
---
name: setup-yourgateway
description: Set up, build, debug, review, and explain YourGateway payment integrations. Use when working with checkout, payment links, callbacks, webhooks, signatures, refunds, sandbox/live setup, and payment settlement for YourGateway.
---

# Setup YourGateway

Use this skill for YourGateway integration work.

## Source

Read `references/yourgateway.md` before giving factual API guidance or writing integration code.

## Core Workflow

1. Identify environment: sandbox or production.
2. Create local pending payment first.
3. Create checkout/session server-side.
4. Verify callback/webhook signature.
5. Re-fetch payment state from provider when possible.
6. Settle idempotently after amount, currency, provider, and reference match.
```

### Step 4: Add `agents/openai.yaml`

```yaml
interface:
  display_name: "Setup YourGateway"
  short_description: "Set up YourGateway payments"
  default_prompt: "Use $setup-yourgateway to integrate YourGateway payments."
```

### Step 5: Add references

Create:

```text
setup-yourgateway/references/yourgateway.md
```

Include:

- Official docs URLs.
- Environment base URLs.
- Auth method.
- Checkout/session endpoint.
- Webhook/callback signature rules.
- Amount unit rules.
- Sandbox test notes.
- Common failure fixes.

Do not include real API keys, tokens, merchant secrets, or private customer data.

### Step 6: Validate

Run validator:

```bash
python3 /home/user/.codex/skills/.system/skill-creator/scripts/quick_validate.py setup-yourgateway
```

If you do not have that validator, install the latest Skills CLI and at minimum verify:

```bash
npx skills@latest --version
```

### Step 7: Test install locally

From a separate test directory:

```bash
npx skills@latest add /path/to/malaysia-payment-gateway --skill setup-yourgateway
```

### Step 8: Open pull request

```bash
git checkout -b add-setup-yourgateway
git add setup-yourgateway
git commit -m "Add setup-yourgateway skill"
git push origin add-setup-yourgateway
```

Then open a pull request with:

- Gateway name.
- What products/features are covered.
- Official docs source links.
- Validation result.
- Any known gaps.
