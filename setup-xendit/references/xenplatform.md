# Xendit xenPlatform

Sources read:

- https://docs.xendit.co/apidocs#xenplatform
- https://docs.xendit.co/docs/xenplatform-overview

## Model

xenPlatform is for platform businesses managing payments and funds for multiple merchants.

Key terms:

- Master Account: platform's main Xendit account with xenPlatform activated. Can create/manage sub-accounts.
- Sub-Account: Xendit account where transactions are processed for merchants, partners, sellers, branches, drivers, or similar entities.
- Account ID: identifier used when making requests for a specific account. Xendit docs may also call this `user-id`, `owner_id`, or `business_id` depending on endpoint/product.

## Use Cases

xenPlatform supports:

- creating/managing sub-accounts
- accepting payments for merchants
- splitting payments between accounts
- monitoring transactions centrally
- payouts to merchant bank accounts
- moving balances between platform and sub-accounts

Common platforms:

- marketplace sellers
- SaaS vendors
- POS merchant branches
- brand subsidiaries

## Routing Requests

When acting on behalf of sub-accounts, confirm exact API mechanism from current endpoint docs. Xendit commonly uses account identifiers such as:

- `for-user-id` header for acting on behalf of a sub-account in many API flows
- `business_id`, `owner_id`, or similar fields in object payloads/responses

Do not mix master-account and sub-account state. Persist both platform account ID and sub-account ID per transaction.

## Payment Patterns

Accept payments for sub-accounts:

1. Platform authenticates with master account secret key.
2. Request targets merchant/sub-account using documented routing mechanism.
3. Payment is processed under sub-account context.
4. Webhooks and reports are correlated back to platform order and merchant account.

Split/fee patterns:

- Use split fees/routing rules only after confirming supported channel and endpoint.
- Store split rule/fee identifiers separately from payment identifiers.
- Reconcile fees, settlements, and transfers as separate ledger movements.

Transfers and payouts:

- Treat transfer balances and payouts as separate money-moving workflows with their own idempotency and approval controls.
- Never infer merchant balance solely from successful payment; account for settlement timing, fees, refunds, chargebacks, and pending states.

## Design Checklist

- Decide sub-account type and onboarding flow before integration.
- Confirm who controls merchant dashboard, payouts, refunds, and disputes.
- Confirm webhook destination strategy: one platform endpoint vs per-sub-account endpoint.
- Confirm reporting: consolidated platform reporting plus per-merchant transaction views.
- Keep an internal ledger for platform fees, merchant payable, refunds, chargebacks, payouts, and balance transfers.

