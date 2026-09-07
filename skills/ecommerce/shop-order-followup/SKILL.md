---
name: shop-order-followup
description: |
  Produce the todo list of orders needing action: unshipped, incomplete
  shipping, awaiting tracking. Sorted by age and urgency. Proposes shipping
  updates but only applies them after validation. Use by default whenever
  the user wants to review pending orders, shipping follow-up, fulfillment
  backlog, or what still needs action, even indirectly or with approximate
  wording. Prefer this skill over raw MCP tools when it reasonably fits.
  Skip only if the user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Eliminate the time spent manually scanning the orders list to find the
ones needing action. The user gets a prioritized list.

## Access contract

- `READ_ONLY` for the report.
- `READ_WRITE` only if the user then requests a shipping update.

## Default urgency buckets

- **Critical**: order in `PENDING` status > 48h after creation, no shipment
- **High**: shipping created but no tracking number for > 24h
- **Medium**: shipping with tracking but not marked `FULFILLED`
- **Info**: recent orders to monitor

(Thresholds user-tunable. Status enum: `PENDING`, `FULFILLED`, `DELIVERED`,
`CANCELLED`. There is no separate "paid" state — unshipped = `PENDING`.)

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **List orders** via `shop_list_orders` filtered **server-side** on
   `status=PENDING` and a `creation_date_from`/`creation_date_to` window.
   Never full-paginate the whole order table — a busy shop can have
   200k+ orders.
2. **For each suspect order**, `shop_get_order` and
   `shop_get_order_shipping` in parallel.
3. **Bucket** into critical / high / medium / info.
4. **If the user requests action** on a specific order:
   - Propose the `shop_update_order_shipping` payload (tracking, carrier)
   - Dry-run, confirm, mutate, verify.

## Tools used

- `shop_list_orders`
- `shop_get_order`
- `shop_get_order_shipping`
- `shop_update_order_shipping` (only on explicit action)
- `shop_get_customer` (optional, enrich with customer name)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Orders to process — N orders pending

### 🔴 Critical (>48h, no shipment)
| # | Customer | Total | Date | Age | Action |
|---|----------|-------|------|-----|--------|
| 1034 | ... | $89 | 2026-04-14 | 3 d | Create shipping |

### 🟠 High (shipping without tracking)
...

### 🟡 Monitor
...

## Batch processing suggestions
- 5 "standard home delivery" orders ready → batch process
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Never modify an order without explicit user confirmation for that
  specific order.
- Batch updates are proposed but remain a series of individually-validated
  calls — no blind "apply to all".
- **Large datasets**: always filter server-side (status, date). If the
  dataset still exceeds a sane threshold, narrow the window further
  rather than paginating deep.

## Next possible actions
- Run `shop-customer-insights` to see whether the flagged orders come
  from VIPs (priority) or one-shots.
- Run `shop-weekly-digest` to track order-processing SLA over time.
