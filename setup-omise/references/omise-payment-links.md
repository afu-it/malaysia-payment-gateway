# Omise Payment Links+ API

Checked 2026-10-21. Payment Links+ is a separate product with its own dashboard, API host and API key.

## Source URLs

- Payment Links+ API: `https://docs.omise.co/en/payment-links-apis/malaysia`
- Using Payment Links+: `https://docs.omise.co/en/using-payment-links/malaysia`

## Basics

- Base URL: `https://linksplus-api.omise.co`.
- Auth: header `Authorization: <PAYMENT_LINKS_API_KEY>` (from Payment Links+ dashboard > Settings > API keys). Not the Omise secret key.
- Rolling a key hides the old value forever; roll again if you lose it.
- Body: form-encoded or JSON.
- Errors share one shape (`code`, `message`; `401` also has `location`): `400 bad_request`, `401 authentication_failure`, `403 forbidden`, `404 not_found`.

## Endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/external/templates/{template_id}` | Retrieve a template (4-digit id under the template thumbnail) |
| GET | `/external/{team_id}/templates` | List team templates: `{"templates": [...], "total": N}` |
| GET | `/external/links/{link_id}` | Retrieve a link |
| GET | `/external/{team_id}/links` | List team links: `{"links": [...], "total": N}` |
| POST | `/external/links` | Create a link (`201`) |
| PUT | `/external/links/{link_id}` | Update a link |
| POST | `/external/links/{link_id}/disable` | Disable an `active` or `pending` link (idempotent; `400` if expired or paid) |

## Create a link

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `template_id` | integer | yes | Template to use |
| `team_id` | integer | yes | Owning team (Settings > Teams); cannot change later |
| `name` | string | yes | Up to 45 characters |
| `currency` | string | yes | For example `MYR` |
| `amount` | integer | yes | Smallest unit, min `100` |
| `multiple_usage` | boolean | yes | Reusable link or single use |
| `additional` | string | no | Note up to 255 characters, hidden on the form by default |
| `expired_at` | string | no | ISO 8601 with offset, future date, stored in UTC |
| `features` | object | conditional | Required when the template defines product fields; keys must match exactly (else `400 invalid features. "Date" is required`) |
| `custom_webhook` | string | no | HTTPS URL for this link (v3.10.0+) |
| `returnUrl` | string | no | Redirect after payment. Priority: this, template return URL, account Settings > Return URL |

```bash
curl https://linksplus-api.omise.co/external/links -X POST -H "Authorization: $PAYMENT_LINKS_API_KEY" \
  -d "template_id=4108" -d "team_id=5512" -d "name=Order 1001" -d "currency=MYR" -d "amount=15000" \
  -d "multiple_usage=false" -d "custom_webhook=https://shop.example.my/webhooks/links" \
  -d "returnUrl=https://shop.example.my/thanks"
```

Update (`PUT`) accepts `name`, `currency` (one of `THB`, `USD`, `EUR`, `GBP`, `JPY`, `SGD`, `MYR`), `features`, `additional`, `expired_at` (`null` clears), `returnUrl`.

## Objects

Link: `id` (integer), `name`, `currency`, `amount`, `multiple_usage`, `additional`, `expired_at`, `features` (flat object of values), `transaction_id`, `url` (send to the customer), `state` (`active`, `pending`, `disabled`, `expired`, `used`), `template_id`, `kind` (`null` or `quick_link`), `created_at`, `updated_at`.

Template: `id`, `name`, `features` (array of `display_name`, `value` null, `type`), `capabilities` (array of `payment_method`, `currencies`, `card_brands`, `installment_terms`, `banks`, and for cards `manual`, `saveCard`), `zero_interest_installment`, `created_at`, `updated_at`.

## Webhooks and tracking

- Up to three webhooks per payment: default system webhook, the dashboard webhook from the merchant dashboard, and `custom_webhook`. A failing custom webhook does not block the others.
- Webhook payloads use `link_id` for the link `id`.
- The Omise charge appears in the Omise dashboard only after the customer attempts payment (successful, pending if they drop at OTP, failed if they cancel). Find it via Charges > Advanced Search > Metadata with the link `id` or `features` value.
- Settle with the same rules as any charge: verify the signed webhook and re-fetch the charge with `GET /charges/{id}`.
