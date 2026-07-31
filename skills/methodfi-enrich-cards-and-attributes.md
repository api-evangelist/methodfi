---
name: Enrich cards and compute financial attributes
description: Pull card-brand details and credit scores, and compute Entity/Account financial attributes for PFM and lending insights.
api: openapi/methodfi-openapi-original.yml
operations: [createEntityCreditScore, retrieveEntityCreditScore, createAccountCardBrand, retrieveAccountCardBrand, createEntityAttributes, retrieveEntityAttribute, createAccountAttribute]
---

# Enrich cards and compute financial attributes

Turn discovered liabilities into insight: card brand art/terms, credit scores, and computed attributes.

## Auth & conventions
- `Authorization: Bearer sk_...` + `Method-Version: 2026-03-30`; `Idempotency-Key` on POSTs.

## Steps
1. **Credit scores** — `createEntityCreditScore` for the entity, then `retrieveEntityCreditScore`. Subscribe for `credit_score.increased` / `credit_score.decreased` events to monitor over time.
2. **Card brand details** — `createAccountCardBrand` then `retrieveAccountCardBrand` on a credit-card account to get the brand `details` object: purchase/cash-advance APR ranges, `annual_fee` / `late_payment_fee`, a rewards object (type + per-category earn rates), promotions, and `card_category`. (Available on `Method-Version` 2025-07-04+.)
3. **Compute attributes** — `createEntityAttributes` to compute aggregated financial metrics across the entity's accounts (revolving balances, utilization, delinquency, HELUC/available-credit signals). On version `2026-03-30` attributes return a per-attribute `{ value, error }` shape and compute asynchronously — poll `retrieveEntityAttribute` or wait for `attribute.update` / `entity_attribute.*` webhooks. Use `createAccountAttribute` for account-scoped attributes.

## Error handling
- Per-attribute failures carry an `error` object in the `{ value, error }` shape using the standard `{ type, sub_type, code, message }` format.
- `ENTITY_CREDIT_SCORE_NOT_FOUND` means no score is available for the entity yet.
- Card Brands details require the account to be enabled for the product — contact Method CSM if the `details` object is absent.
