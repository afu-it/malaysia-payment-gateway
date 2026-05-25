# Design: Stripe Malaysia Payment Gateway Skill

**Date:** 2026-05-25
**Status:** Approved

## Summary

Add `setup-stripe-malaysia` to the malaysia-payment-gateway skill collection, following the same provider folder pattern as `setup-xendit`, `setup-hitpay`, etc.

## Approach

Create a single `setup-stripe-malaysia/` folder with:

- `SKILL.md` — skill metadata, quick start, integration rules, core workflow, critical rules, completion criteria, loophole review loop.
- `agents/openai.yaml` — minimal interface definition.
- `references/stripe-api.md` — Stripe API reference distilled for agents: authentication, Checkout Sessions, PaymentIntents, webhooks (Stripe-Signature), CLI local forwarding, idempotency, test mode, environment vars, and relevant endpoint shapes.
- `references/account-setup.md` — registration, API keys (publishable + secret), test/live mode separation, dashboard webhook configuration, MYR/Malaysia-specific caveats.
- `references/source-map.md` — quick-link map to official docs, API reference, SDKs, Stripe.js, CLI docs, and samples repositories.

The skill enforces the same server-verified state machine pattern as other skills in the repo:

1. Create local pending payment before Stripe checkout.
2. Create Checkout Session or PaymentIntent server-side (secret key never exposed).
3. Redirect or render Stripe Elements using only publishable key and server-returned IDs.
4. Never fulfil from client-side redirect alone.
5. Verify webhook `Stripe-Signature` against raw body.
6. Settle idempotently by Stripe event ID and payment object ID.
7. Match amount, currency, and payment status before marking paid.

## Scope

- Malaysia-focused: MYR, cards, FPX and other local payment methods where available in Stripe Malaysia.
- Server-side integration: Checkout Sessions, PaymentIntents, Elements.
- Webhooks: signature verification, idempotent handlers, event filtering.
- CLI: local webhook testing with `stripe listen`.
- SDK: server library usage guidance.
- Test mode vs live mode separation.

## Conventions

- Folder name matches `name:` in SKILL.md: `setup-stripe-malaysia`.
- Lowercase hyphenated naming.
- Same `references/` and `agents/` subfolder pattern as all existing skills.
- Listed in root README skills table and prompt examples.
- Follows the contribution template in README.

## Out of Scope

- Stripe Connect/platform marketplace (can be added later).
- Stripe Billing/Subscriptions deep integration (can be added later).
- Non-MYR currencies or non-Malaysian Stripe accounts (generic guidance only).
