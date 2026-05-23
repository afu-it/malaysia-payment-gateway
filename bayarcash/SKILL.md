---
name: bayarcash
description: Build, debug, and explain Bayarcash payment integrations using Bayarcash API docs, Web Impian API docs, Bayarcash PHP SDK, WordPress plugins, Boost.space integration links, and Bayarcash MCP server references. Use when working with Bayarcash API v2/v3, payment intents, FPX banks, DuitNow banks or wallets, portals, payment channels, callbacks/webhooks, checksums, transaction lookups, direct debit/e-Mandate flows, enterprise partner merchant/payout APIs, Laravel/PHP SDK usage, WordPress Bayarcash integrations, or Bayarcash MCP tooling.
---

# Bayarcash

Use this skill for Bayarcash payment gateway integration work.

## Source

Read `references/bayarcash-api.md` before giving factual API guidance or writing integration code. It contains source URLs, environment bases, endpoint map, checksum/callback rules, SDK notes, and live-doc lookup instructions.

Read `references/bayarcash-api-documentation.md` when exact fields, examples, direct debit, enterprise partner, portal, transaction, or checksum details are needed from the GitHub documentation repo snapshot.

## Core Workflow

1. Identify environment: sandbox or production, API v2 or v3.
2. Choose integration surface: raw HTTP API, PHP SDK, WordPress plugin, MCP tool, or Boost.space workflow.
3. Look up exact endpoint, request fields, callback shape, checksum rule, and version support in the reference files.
4. Keep API tokens and secret keys server-side. Never expose secrets in browser code, WordPress shortcodes, logs, screenshots, or client bundles.
5. Treat payment, callback, direct debit, and payout actions as high impact. Validate amount units, order uniqueness, callback idempotency, status mapping, environment, and channel enablement.

## Working Rules

- Prefer API v3 for new work when available, especially where v3-only fields such as `callback_url` or payment intent IDs are needed.
- Do not invent bank codes, channel IDs, status values, checksum formulas, or callback payload fields. Use source docs or query live docs.
- Store `order_number`, Bayarcash IDs, transaction IDs, callback checksum result, and processed status to prevent duplicate fulfillment.
- Return HTTP `200` for accepted server callbacks after verification and durable recording.
- For WordPress tasks, distinguish Bayarcash payment plugins from unrelated MyInvois/e-invoice plugins unless user explicitly connects both.
- If user asks for latest fees, supported channels, bank availability, plugin compatibility, MCP tools, or portal/account-specific data, verify live docs or account console first.
