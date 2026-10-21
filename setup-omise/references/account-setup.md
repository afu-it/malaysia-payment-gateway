# Omise Account Setup (Malaysia)

Checked 2026-10-21. Fees: see the official pricing page only, `https://www.omise.co/en/pricing/malaysia`. Fee numbers are not copied into this skill.

## Source URLs

- Enable live account: `https://docs.omise.co/en/how-do-i-enable-live-account/malaysia`
- Access API keys: `https://docs.omise.co/en/how-to-access-omise-api-keys/malaysia`
- Test keys and live keys: `https://docs.omise.co/en/what-are-test-keys-and-live-keys/malaysia`
- Leaked secret key: `https://docs.omise.co/en/my-api-secret-key-has-been-leaked-what-to-do/malaysia`
- Authentication: `https://docs.omise.co/en/api-authentication/malaysia`
- Versioning: `https://docs.omise.co/en/api-versioning/malaysia`
- Update API version: `https://docs.omise.co/en/how-can-i-update-my-api-version/malaysia`
- Contact: `https://docs.omise.co/en/how-to-contact-omise/malaysia`
- Prohibited businesses: `https://docs.omise.co/en/prohibited-businesses/malaysia`
- Malaysia terms: `https://docs.omise.co/en/terms-malaysia/malaysia`
- Security best practices: `https://docs.omise.co/en/security-best-practices/malaysia`
- Pricing: `https://www.omise.co/en/pricing/malaysia`

## Steps

1. Sign up at `https://dashboard.omise.co` and verify the email. Test mode is available immediately.
2. Dashboard > Keys (test mode): copy `pkey_test_...` and `skey_test_...` into your secret store as `OMISE_PUBLIC_KEY` and `OMISE_SECRET_KEY`.
3. In the dashboard API version setting, set test mode to `2019-05-29` (or pin `Omise-Version` per request).
4. Dashboard > Webhooks (test): set the HTTPS endpoint, then create a webhook secret in Webhooks Settings and store it as `OMISE_WEBHOOK_SECRET`.
5. Email support@omise.co to enable each non-card method you need (FPX, DuitNow OBW, DuitNow QR, Touch 'n Go, Boost, GrabPay, ShopeePay, Maybank QR, Atome, installments) and 3-D Secure. Confirm with `GET /capability`.
6. Build and test (see `omise-testing-errors.md`).
7. Apply for a live account from the dashboard (fully online).
8. After approval: repeat steps 2 to 5 for live mode. Live and test have separate keys, API versions, webhook endpoints and webhook secrets.

## Live account prerequisites

- An Omise account with a verified email.
- A live website.
- The refund policy shown to the customer before final checkout with an accept action, or on the checkout screen near the submit button. A link alone to a separate page is not enough (Visa card acceptance guidelines).
- Documents in PDF, JPG or PNG.

Required documents (business):

- Latest full company registration documents.
- Front page of the bank statement.
- Additional permits or licences for the business category.
- Directors: proof of identity (NRIC, or passport for foreigners) and proof of address.
- Ultimate beneficial owners above 25%: proof of identity and address.
- Retail photos (3 interior with products or services, 3 exterior with company logo).
- Bank form (sent after the application).

Review time: the KYC team contacts you within 30 business days (article wording). Check the prohibited businesses list before applying.

## Keys

- Two sets: test (public plus secret) and live (public plus secret). The key set decides whether real money moves.
- Live secret key is shown only for 15 minutes on the first live sign-in. After that, roll it to see a new one.
- Rolling: the old key is deprecated after 60 minutes, or click "Revoke key now". Omise emails the IP and date on every roll.
- If a secret key leaks: roll it immediately in the dashboard.
- Never put the secret key in browser or mobile code. Public key only on clients.
- Enable 2FA on the dashboard; review sessions at `https://dashboard.omise.co/sessions`.

## Environment variables (placeholders)

```bash
OMISE_PUBLIC_KEY=pkey_test_<placeholder>
OMISE_SECRET_KEY=skey_test_<placeholder>
OMISE_WEBHOOK_SECRET=<base64-webhook-secret-placeholder>
OMISE_API_VERSION=2019-05-29
```

## Support

- Email support@omise.co. Business hours UTC+07:00, Monday to Sunday and public holidays 09:00 to 18:00.
- Phone support UTC+07:00: Monday to Friday 06:00 to 22:00; weekends and holidays 09:00 to 18:00. The phone number is on the contact page.
- Status page: `https://status.omise.co`.
- Contracting entity: not named on the fetched Malaysia terms page (checked 2026-10-21). MPG's own data lists Omise Payment Malaysia Sdn Bhd; confirm on your merchant agreement.
