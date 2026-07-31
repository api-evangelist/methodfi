---
name: Monitor accounts with subscriptions and webhooks
description: Keep liability data fresh with Updates/Connect subscriptions and receive change notifications over webhooks.
api: openapi/methodfi-openapi-original.yml
operations: [createEntitySubscription, createAccountSubscription, createAccountUpdate, createWebhook, listEvents, retrieveEvent]
---

# Monitor accounts with subscriptions and webhooks

Set-and-forget monitoring so balances, credit health, and new liabilities stay current.

## Auth & conventions
- `Authorization: Bearer sk_...` + `Method-Version: 2026-03-30`; `Idempotency-Key` on POSTs.

## Steps
1. **Register a webhook** — `createWebhook` with your HTTPS endpoint. Method posts events named `<resource>.<action>` (literal) and `<resource>.<signal>.<direction>` (computed). Handle unfamiliar event types gracefully — new types are additive.
2. **Subscribe for entity-level monitoring** — `createEntitySubscription` to watch for new liabilities and credit-health attribute changes (e.g. `credit_score.increased`, `attribute.credit_health_credit_card_usage.increased`).
3. **Subscribe / refresh at the account level** — `createAccountSubscription` for near-real-time account changes, or trigger an on-demand `createAccountUpdate` to refresh a single account's data.
4. **Reconcile from the Events API** — on webhook receipt (or on a schedule), call `retrieveEvent` for the `evt_...` id, or `listEvents` with `from_date`/`to_date` to backfill anything missed.

## Notes
- Events cover accounts, balances, credit scores, connects, and entity/account attributes — see asyncapi/methodfi-webhooks.yml for the catalog.
- Webhook processing errors use the `25XXX` resource error code range.
- Paginate `listEvents` with `page`/`page_cursor`/`page_limit` (max 100) and read `Pagination-*` response headers.
