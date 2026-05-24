---
name: setup-bcl
description: Set up, build, debug, review, and explain BCL payment gateway, BCL Pay, BCL Payment Link, BCL QR Terminal, BCL Forms, Bayarcash-linked setup, portal keys, transactions, webhooks, direct debit forms, client info, payment redirects, and BCL operating procedures using BCL docs.
---

# BCL Payment Gateway

Use this skill for BCL payment gateway and BCL Pay work.

Read `references/bcl-docs.md` before giving factual BCL guidance or writing integration code. It contains source URLs, product boundaries, setup flow, portal key guidance, transaction/webhook notes, and known links from `docs.bcl.my`.

Read `references/account-setup.md` when user asks how to register, which keys are needed, where credentials come from, or what dashboard setup is required.

## Core Workflow

1. Identify surface: BCL Pay setup, Payment Link, QR Terminal, Forms, client info, transactions, webhook, or direct debit.
2. Check whether task is BCL-owned, Bayarcash-linked, or pure Bayarcash API work.
3. Use BCL docs for BCL UI/workflow details; use Bayarcash docs only for raw Bayarcash API fields and checksum/callback specifics.
4. Keep API keys, portal keys, and token values server-side unless BCL docs explicitly show public use.
5. Treat payment status, callbacks, webhooks, and direct debit as high impact. Verify identity, amount, order/reference uniqueness, and idempotency before fulfilment.

## Boundary Rules

- BCL is not the same documentation set as Bayarcash. BCL docs reference connecting BCL Pay to Bayarcash and exposing a Bayarcash portal key.
- Do not merge BCL procedures into the Bayarcash skill unless user asks for Bayarcash API details.
- Do not invent BCL endpoint schemas. If exact fields are missing from local reference, use live docs at `https://docs.bcl.my/`.
- For latest fees, supported channels, account-specific settings, dashboard UI changes, or webhook payload shape, verify live docs or account console first.
