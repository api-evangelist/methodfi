---
name: Onboard an entity and discover liabilities
description: Create a Method entity, verify identity, and run Connect to discover the user's complete liability picture.
api: openapi/methodfi-openapi-original.yml
operations: [createEntity, createEntityVerificationSession, updateEntityVerificationSession, createEntityConnect, retrieveEntityConnect, listAccounts]
---

# Onboard an entity and discover liabilities

Foundational flow for lending, PFM, and commerce integrations on Method.

## Auth & conventions
- Send `Authorization: Bearer sk_...` (secret key) and `Method-Version: 2026-03-30` on every request.
- Add `Idempotency-Key: <uuid>` to every POST so retries never double-create.
- Base host: `https://production.methodfi.com` (use `dev.` while testing).

## Steps
1. **Create the entity** — `createEntity` with `type: individual` and the user's `first_name`, `last_name`, `phone`, `email`, `dob`. Store the returned `ent_...` id. Attach your own `metadata` (< 1KB) to cross-reference your user.
2. **Verify identity** — `createEntityVerificationSession` (choose a method: SMS, KBA, SNA, or KYC), then `updateEntityVerificationSession` with the user's response to complete it. An entity must reach a verified/matched state before sensitive products run.
3. **Discover liabilities** — `createEntityConnect` for the entity to soft-pull and enumerate all connected liability accounts. Poll `retrieveEntityConnect` (or subscribe to the `connect.update` webhook) until it completes; handle partial results.
4. **Read the accounts** — `listAccounts` filtered to the entity to get the discovered `acc_...` liability accounts.

## Error handling
- Entity request errors (validation, `DUPLICATE_ENTITY_DETAILS`, `ENTITY_INVALID_SSN`) return the `{ type, code, sub_type, message }` envelope — see errors/methodfi-problem-types.yml.
- Resource-level `ENTITY_DISABLED` codes (12001–12005) land on the entity's `error` property (e.g. `ENTITY_PENDING_KYC_REVIEW`, `ENTITY_SSN_MISMATCH`); do not retry blindly.
- In `dev`, queue discovery with `simulateEntityConnect` behaviors (`new_credit_card_account`, `new_auto_loan_account`, …).
