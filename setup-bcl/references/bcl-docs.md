# BCL Documentation Reference

Source checked on 2026-05-18:

- Docs home: `https://docs.bcl.my/?origin=QXBwXEZpbGFtZW50XFBhZ2VzXFRlYW1cVGVhbVBhbmVs`
- How to connect BCL Pay to Bayarcash: `https://docs.bcl.my/727`
- How to set up Payment Link: `https://docs.bcl.my/732`
- How to create a QR Terminal: `https://docs.bcl.my/358`
- How to retrieve Bayarcash Portal Key from BCL: `https://docs.bcl.my/779`
- Manual PDF: `https://bcl-media.s3.ap-southeast-1.amazonaws.com/BCL+Manual.pdf`

## Product Boundary

BCL docs describe BCL Pay and BCL dashboard workflows. BCL can connect to Bayarcash and expose Bayarcash-related configuration such as portal key, but BCL docs are a separate documentation set from Bayarcash API docs.

Use BCL docs for:

- BCL Pay setup and operation.
- BCL Payment Link setup.
- BCL QR Terminal setup.
- BCL Forms and direct debit form workflows.
- BCL transaction pages and payment detail lookup.
- BCL client info pages.
- BCL webhook configuration pages.
- Finding Bayarcash Portal Key from inside BCL.

Use Bayarcash docs for:

- Raw Bayarcash API v2/v3 endpoints.
- Payment intent request fields.
- Bayarcash checksum calculation and callback verification.
- Bayarcash transaction callback status and retry rules.

## BCL + Bayarcash Setup

The BCL guide "How to Connect BCL Pay to Bayarcash" covers linking BCL Pay with Bayarcash. It is a BCL-side setup guide, not a raw Bayarcash API integration guide.

Relevant BCL setup pages listed in docs navigation include:

- How to connect BCL Pay to Bayarcash.
- How to setup Payment Link.
- How to create QR Terminal.
- How to setup Payment Link using QR Terminal.
- How to share BCL Pay payment link.
- How to set a minimum amount.
- How to remove other payment methods.
- How to disable payment link.
- How to view payment details.
- How to edit client information.
- How to retrieve Bayarcash Portal Key.

## Payment Link

BCL docs provide a dashboard workflow for setting up Payment Link. Treat Payment Link as a BCL-managed payment collection surface unless user supplies raw Bayarcash API code.

Implementation notes:

- A BCL Payment Link may ultimately route through connected payment providers.
- Store the BCL link/reference and any returned provider transaction ID when integrating with backend fulfilment.
- Do not fulfil from browser redirect alone; confirm payment state through BCL/Bayarcash transaction detail, callback, webhook, or trusted server flow.

## QR Terminal

BCL docs provide a workflow for creating a QR Terminal and for using QR Terminal with Payment Link.

Implementation notes:

- Treat QR Terminal as another BCL-managed entry point to payment.
- Keep QR/link ownership clear by terminal, branch, user, or campaign where relevant.
- Reconcile QR payments against transaction pages or provider callbacks before marking internal orders paid.

## Portal Key

BCL docs include "How to retrieve Bayarcash Portal Key from BCL" at `https://docs.bcl.my/779`.

Use this when user asks where to find the portal key inside BCL. For raw API calls, validate exact Bayarcash API requirements in the Bayarcash skill docs.

## Webhook And Callbacks

BCL navigation lists "Webhook" under forms/direct debit areas. Local BCL docs snapshot does not include exact webhook payload schema.

Rules:

- Prefer server-to-server confirmation over client redirect.
- Verify any signature/checksum if BCL or Bayarcash provides one.
- Persist raw callback/webhook body, headers, verification result, transaction/reference IDs, amount, and status.
- Make fulfilment idempotent by BCL transaction ID, Bayarcash transaction ID, `order_number`, or another unique reference available in the payload.
- If using Bayarcash callback payloads, use Bayarcash checksum and callback rules from `bayarcash` skill.

## Direct Debit / Forms

BCL docs navigation contains:

- Direct Debit.
- View Form.
- Subscription.
- Webhook.
- How to configure payment form.
- How to create new payment form.
- How to set up On-Demand Payment.
- How to view and edit payment form.
- How to get Direct Debit URL.

Treat direct debit as high impact:

- Verify mandate/subscription status before charging or provisioning.
- Store mandate/subscription IDs and payment IDs.
- Support cancellation, failed payment states, and webhook retries.

## Known Documentation Gaps

These details should be checked live before implementation:

- Exact BCL webhook payload and signature scheme.
- Current BCL dashboard labels or menu locations.
- Current payment provider/channel availability.
- Account-specific Bayarcash portal key and permissions.
- Fees, settlement timing, and production go-live requirements.

Live lookup start point:

```text
https://docs.bcl.my/
```
