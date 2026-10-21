# Omise Refunds, Voids and Disputes

Checked 2026-10-21.

## Source URLs

- Refund API: `https://docs.omise.co/en/refunds-api/malaysia`
- How do I create a refund: `https://docs.omise.co/en/how-do-i-create-a-refund/malaysia`
- Refunds for installments: `https://docs.omise.co/en/how-do-refunds-work-for-installment-payments/malaysia`
- Refunds for internet banking: `https://docs.omise.co/en/how-does-the-refund-process-work-for-internet-banking/malaysia`
- Dispute API: `https://docs.omise.co/en/disputes-api/malaysia`
- Document API: `https://docs.omise.co/en/documents-api/malaysia`
- Disputes overview: `https://docs.omise.co/en/disputes-overview/malaysia`
- Disputes best practices: `https://docs.omise.co/en/disputes-best-practices/malaysia`
- "How do I manage a dispute?" support article (served at this slug): `https://docs.omise.co/en/how-are-omise-fees-displayed/malaysia`
- Withdrawn disputes evidence: `https://docs.omise.co/en/evidence-for-withdrawn-disputes/malaysia`
- Disputes in test mode: `https://docs.omise.co/en/how-to-open-and-manage-disputes-in-test-mode/malaysia`
- Method pages (refund windows): see `omise-malaysia-methods.md`

## Refund rules

- A charge is refundable when it is captured, not disputed, not fully refunded, has fewer than 15 refunds, and was captured no more than 365 days ago.
- Card brand note: JCB, AMEX and Diners can be refunded within 160 days.
- Method windows are shorter: FPX six months; DuitNow OBW, DuitNow QR, Touch 'n Go, Boost, ShopeePay, Maybank QR 180 days; GrabPay three months; Atome and Atome QR 60 days.
- A full refund before settlement is processed as a void (`voided=true`). `void=true` forces void-only where possible.
- Check `refundable` and `partially_refundable` on the charge before offering partial refunds.
- Installment charges: full refund only. Interest absorbed by the merchant is not returned.
- ShopeePay off-us transactions (paid through a mobile banking app) cannot be refunded or voided through Omise.
- Internet banking support article: contact the customer and pay the refund yourself in advance. The FPX page says FPX can be refunded by API within six months. Confirm for your account.
- Timing (support article): Omise processes immediately; credit cards see it on the next billing cycle; debit cards in 30 to 60 days.
- Refunds can also be made from the dashboard (charge > `...` > refund).
- Cancelled cards can still be refunded; the banks route the funds.

## Create a refund

`POST https://api.omise.co/charges/{id}/refunds` (secret key).

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `amount` | integer | yes | Smallest unit of charge currency |
| `metadata` | object | no | Max field length 15,000 characters |
| `reason` | string | no | Free text up to 255 characters; longer values are truncated silently |
| `ref_id` | string | no | Documented as "TBD" |
| `void` | boolean | no | Only process as a void if possible. Default: void before settlement, refund after |

```bash
curl https://api.omise.co/charges/chrg_test_xxx/refunds -u "$OMISE_SECRET_KEY:" \
  -d "amount=10000" -d "metadata[order_id]=1001" -d "reason=Customer request"
```

Errors: `failed_refund`, `failed_void`, `not_found`.

## Refund object

| Field | Type | Notes |
| --- | --- | --- |
| `object` | string | `refund` |
| `id` | string | `/rfnd(_test)?_[0-9a-z]+/` |
| `livemode`, `location` | boolean, string | |
| `amount`, `currency` | integer, string | Charge currency |
| `funding_amount`, `funding_currency` | integer, string | Settlement currency amount |
| `charge` | id/expandable | Charge id |
| `transaction` | id/expandable | Balance transaction |
| `status` | string | Refund status (values not enumerated in docs) |
| `voided` | boolean | Processed as void |
| `reason`, `metadata`, `ref_id` | | Yours |
| `acquirer_reference_number`, `approval_code` | string | Network and issuer references |
| `terminal`, `capture`, `merchant_info`, `merchant_name`, `merchant_uid` | | POS and PayFac fields |
| `created_at` | string | ISO 8601 |

## Other refund endpoints

| Endpoint | Purpose |
| --- | --- |
| `GET /charges/{id}/refunds` | List refunds for a charge (list params) |
| `GET /charges/{id}/refunds/{refund_id}` | Retrieve a refund |
| `GET /refunds` | List all refunds on the account |

Webhook: `refund.create` (sent to the charge's `webhook_endpoints` if it had them).

## Disputes

- A dispute (chargeback) is filed by the cardholder or issuer. Commercial disputes (goods or service) and fraud disputes (not authorized).
- Merchant options: contact the customer, respond with evidence, or accept (returns the full amount). Action is required within 7 days.
- Omise may suspend or terminate accounts whose dispute level exceeds 0.5% or 75 chargebacks in a month.
- If the customer says they withdrew the dispute, still submit evidence; a signed cardholder letter (amount, date, issuing bank, last 4 digits) is recommended.
- Test mode: open any captured charge in the dashboard and create a dispute, then choose the outcome.

### Dispute object

`object=dispute`, `id` (`/dspt(_test)?_[0-9a-z]+/`), `livemode`, `location`, `amount`, `currency`, `funding_amount`, `funding_currency`, `charge`, `status` (`open`, `pending`, `won`, `lost`), `reason_code`, `reason_message`, `message`, `admin_message`, `documents` (list), `transactions`, `metadata`, `closed_at`, `created_at`, `merchant_info`, `merchant_name`, `merchant_uid`.

### Dispute endpoints

| Endpoint | Params | Purpose |
| --- | --- | --- |
| `GET /disputes` | list params | All disputes |
| `GET /disputes/open`, `/pending`, `/closed` | list params | By state (`closed` holds `won` and `lost`) |
| `GET /disputes/{id}` | | Retrieve |
| `PATCH /disputes/{id}` | `message`, `metadata` | Add your explanation |
| `PATCH /disputes/{id}/accept` | | Accept |
| `PATCH /disputes/{id}/close` | `status` (`won` or `lost`) | Close an open or pending dispute |
| `POST /charges/{id}/disputes` | | Create a dispute for a charge |

Webhooks: `dispute.create`, `dispute.update`, `dispute.accept`, `dispute.close`.

### Evidence documents

`POST /disputes/{id}/documents` (multipart): `file` (PNG, JPG or PDF), `kind` (`cardholder_details`, `details_of_purchase`, `proof_of_receipt`, `proof_of_acceptance`). Uploading to a dispute that is no longer open returns `documents_locked`; wrong type returns `invalid_file_type`; no file returns `missing_file`.

Document object: `object=document`, `id` (`docu_...`), `livemode`, `location`, `filename`, `kind`, `download_uri`, `deleted`, `created_at`. Also `GET /disputes/{id}/documents`, `GET /disputes/{id}/documents/{document_id}`, `DELETE /disputes/{id}/documents/{document_id}`.

## Fraud signals relevant to refunds and disputes

- Charges blocked by Omise risk analysis fail with `failed_fraud_check`.
- Higher-risk business types are put on mandatory 3DS. Frictionless and challenge 3DS both shift liability to the issuer.
- AVS applies to US, UK and Canadian cards only.
