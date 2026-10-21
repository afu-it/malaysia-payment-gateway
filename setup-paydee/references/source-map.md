# Paydee Source Map

Use this map to pick the right official source before answering or coding. All checked 2026-10-21.

## Coverage

- Index used: `https://developer.paydee.co/sitemap.xml` (4 pages) plus every link on those pages. No `llms.txt` / `llms-full.txt` (404).
- Enumerated 14 official sources: 4 sitemap pages, the docs home, 2 Swagger UI pages and their 2 OpenAPI JSON files, 2 spec PDFs (v5.0.0, v4.0.0-2), 1 XLSX message table, 1 GitHub sample repo, 1 hosted sample app.
- Read 11 in full. Not readable: `/api/mac-validation` (login form only), `https://github.com/fin-project/sample-mpi-py` (404), `https://mpi-sample-py.onrender.com` (503).
- The v5.0.0 PDF is the primary source; v4.0.0-2 differs only by DuitNow QR (`DNQR`), `PAG12`/`PAG13`, and the DuitNow QR void/refund row.
- Left out: Paydee card-issuing, Paydee Visa, Flex/Expenses/Advance and accounts-payable products (not payment acceptance APIs).

## Developer Site

- Getting Started: `https://developer.paydee.co/api/get-started` (same as `https://developer.paydee.co/`)

Use for: hosted versus non-hosted endpoints, locked test/production endpoints, test cards, RSA key exchange and MAC steps, sample code links.

- Use Cases: `https://developer.paydee.co/api/use-cases`

Use for: Sales/Checkout, Channel Inquiry, PreAuth, Split Payment (recurring/installment) control parameters, payment reversal, void/refund table with processing method, "INQ 30 minutes later" guidance, EMV 3DS iframe requirement, transaction ID chaining per message.

- Message Type: `https://developer.paydee.co/api/message-type`

Use for: all `MPI_TRANS_TYPE` values with trigger side, INQ status codes, PRERECURR/INITRECURR settlement, SALESCOMPL 20% rule, message table of field presence per transaction type (incl. `MPI_PAN`, `MPI_CARD_HOLDER_NAME`, recurring fields), Visa 2024 data rules, `application/x-www-form-urlencoded` header.

- MAC Validation: `https://developer.paydee.co/api/mac-validation` (login-gated)

Use for: nothing publicly; MAC detail comes from the spec PDF section 2.13.

## Specification Files

- Spec v5.0.0 PDF: `https://developer.paydee.co/download/Paydee_Payment_Acceptance_Gateway_API_Spec_v5.0.0.pdf`

Use for: `/mkReq`, `/mercReq`, `/mpReq`, `/channels` field tables, void/refund table, all response and referral codes, 3DS statuses, payment status, MAC field orders and samples, enrollment file format, revision history.

- Spec v4.0.0-2 PDF: `https://developer.paydee.co/download/Paydee_Payment_Acceptance_Gateway_API_Spec_v4.0.0-2.pdf`

Use for: comparing changes only.

- Message table XLSX v2.5: `https://developer.paydee.co/download/DevBox.v2.5.xlsx`

Use for: field examples and descriptions (MYR only, UAT ID suffix, `MPI_PAN_EXP` MMYY, recurring field names), `MPI_RESPONSE_TYPE` values.

- OpenAPI 3.0 "3DS Merchant Interface" v2.4: `https://developer.paydee.co/static/sales_v2_4.json`, `https://developer.paydee.co/static/preauth_v2_4.json` (UIs at `https://developer.paydee.co/devbox/sales_v2_4` and `/devbox/preauth_v2_4`)

Use for: test server URL `https://devlink.paydee.co/mpi`, per-session key note, `mpReq` property list (older recurring names).

## Sample Code

- `https://github.com/fin-project/sample-mpi-py` (404 on 2026-10-21)
- `https://mpi-sample-py.onrender.com` (503 on 2026-10-21; described as reference-only hosted checkout sample)

Use for: nothing until reachable.

## Company And Onboarding

- `https://www.paydee.co/` (company, licensing, contact)
- `https://www.paydee.co/paydee-for-business`, `https://www.paydee.co/paydee-for-enterprise` (products)
- `https://www.paydee.co/create-free-account` (apply), `https://www.paydee.co/demo` (sales/demo)

Use for: merchant signup path, company identity, pricing stance ("competitive processing rates", no published fees).
