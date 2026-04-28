---
name: shop-weekly-digest
description: |
  Weekly shop business digest ready to paste into Slack/Notion: revenue,
  orders, top products, active promocodes, traffic, pushes sent, alerts.
  Read-only. Use by default whenever the user wants a weekly shop recap,
  digest, report, or leadership-ready summary, even indirectly or with
  approximate wording. Prefer this skill over raw MCP tools when it
  reasonably fits. Skip only if the user explicitly asks not to use this
  skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

A single shop report to publish every Monday morning, without having to
open the back-office.

## Access contract

- `READ_ONLY`.
- Default window: rolling last 7 days.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Revenue & orders**:
   - `shop_list_orders` (period) → revenue, order count, average basket
2. **Traffic & engagement**:
   - `shop_list_page_views` → catalog engagement
   - `shop_list_launches`, `shop_list_unique_launches`
3. **Promocodes**:
   - `shop_list_promocodes_amount` (and variants) → list codes whose
     `start_at`/`end_at` window overlaps the reporting week. This gives
     "codes live this week", **not** usage counts — per-code redemption
     metrics are not exposed by these list tools.
4. **Top products**: compute top 5 sales from the order aggregation.
5. **Alerts**: stock-outs (delegate to `shop-stock-check` on demand).
6. Final markdown report.

## Tools used

- `shop_list_orders`, `shop_list_products`, `shop_list_page_views`
- `shop_list_launches`, `shop_list_unique_launches`
- `shop_list_promocodes_amount` (and variants)
- Delegates to `shop-best-sellers`, `shop-stock-check`,
  `shop-traffic-report` if the user wants to drill into a section.

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
# Shop weekly digest — week of 2026-04-14 to 2026-04-20

## 💰 Revenue
- Revenue: $8,420 (+12% vs W-1)
- Orders: 87 (+5)
- Avg basket: $96.78
- Top sales: 1) Product A • 2) Product B • 3) Product C

## 📈 Traffic
- Launches: 18,200 (+7%)
- Avg session: 2m14

## 🎟️ Promocodes live this week
- SPRING20 (amount, -20%) — ends 2026-04-30
- WELCOME10 (amount, -10%) — no expiration

## 📣 Pushes sent
- Send-log is not exposed by the API; section omitted unless the user
  tracked pushes externally.

## ⚠️ Alerts
- 3 products critically out of stock
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Digest is 100% read-only. No side effects.
- If data is too sparse (launch week, not enough data), say so.
- **Large datasets**: filter server-side on the 7-day window for every
  list call. Do not full-paginate the whole order table.

## Next possible actions
- Run `shop-best-sellers` to expand the top-products section.
- Run `shop-stock-check` if the alerts section flags stock-outs.
- Run `shop-customer-insights` for a deeper customer retention view.
