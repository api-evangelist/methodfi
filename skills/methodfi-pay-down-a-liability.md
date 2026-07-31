---
name: Pay down a liability
description: Move money from a funding source account to a discovered liability account, idempotently, and track settlement.
api: openapi/methodfi-openapi-original.yml
operations: [createAccount, createAccountPayoff, createPayment, retrievePayment, listPayments]
---

# Pay down a liability

Move funds to a connected liability (a "paydown") over ACH.

## Auth & conventions
- `Authorization: Bearer sk_...` + `Method-Version: 2026-03-30`.
- **Always** send `Idempotency-Key: <uuid>` on `createPayment` — Method replays the stored result for a repeated key, guaranteeing the payment is created once. On `503 IDEMPOTENCY_UNAVAILABLE`, retry later.
- Amounts are integer cents. Minimum `100` ($1.00), maximum `100000000` ($1,000,000.00).

## Steps
1. **Ensure a source** — the funding/source `acc_...` must be an ACH account with send capability (create via `createAccount` if needed). The destination is the liability `acc_...` discovered via Connect.
2. **(Optional) Quote a payoff** — `createAccountPayoff` on the liability account to get an exact payoff amount before paying.
3. **Create the payment** — `createPayment` with `amount`, `source`, `destination`, and a `description` (ACH description max 10 chars). Store the `pmt_...` id.
4. **Track settlement** — poll `retrievePayment` or subscribe to payment webhooks. Status moves through `pending` → settled in 1–3 business days; `estimated_completion_date` and settlement dates are returned.

## Error handling
- Request rejects surface `sub_type`s like `INVALID_AMOUNT`, `INVALID_SOURCE_LIABILITY` (source cannot be a liability), `INSUFFICIENT_FUNDS`, `MAX_DAILY_VOLUME_EXCEEDED`.
- Post-submit failures set the payment's `error` property with decline codes `10001`–`10011` (e.g. `PAYMENT_INSUFFICIENT_FUNDS`, `PAYMENT_UNAUTHORIZED`) — see errors/methodfi-decline-codes.yml. Do not retry `PAYMENT_UNAUTHORIZED*`.
- Respect the payment-creation rate tier (120 rpm); back off on `429 TOO_MANY_REQUESTS`.
