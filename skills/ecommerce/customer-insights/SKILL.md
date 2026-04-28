---
name: shop-customer-insights
description: |
  Analyze the customer base: top customers by revenue, purchase frequency,
  loyalty status, segments to re-engage. Recommend loyalty adjustments or
  targeted pushes. Read-only by default; the user validates each loyalty
  update. Use by default whenever the user wants customer analysis,
  segmentation, loyalty review, re-engagement targets, or who to reward,
  even indirectly or with approximate wording. Prefer this skill over raw
  MCP tools when it reasonably fits. Skip only if the user explicitly asks
  not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Turn the raw customer base into actionable segments: who to reward, who to
re-engage, who to ignore.

## Access contract

- `READ_ONLY` for analysis.
- `READ_WRITE` if the user validates a loyalty update at the end.

## Segments produced by the skill

- **VIP**: top 5% by cumulative revenue
- **Loyal**: ≥ 3 orders in the period
- **Dormant**: 1+ order, no purchase for > 90 d
- **New**: first order < 30 d
- **One-shot**: exactly 1 order, > 90 d ago

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `shop_list_customers`: customer base. On large shops (>10k customers),
   **do not** full-paginate blindly — filter server-side when possible
   (e.g. `updated_at` for recently active customers) or sample.
2. `shop_list_orders`: 12-month order history (tunable). Filter
   server-side on the date window; do not fetch all orders to filter
   client-side.
3. Aggregate per customer: order count, total revenue, last purchase.
4. For VIP and Loyal: `shop_get_loyalty` to see their current status.
5. Propose an action list:
   - Upgrade loyalty tier for X customers
   - Targeted "come back" push for Dormants
   - Reward code for VIPs

## Tools used

- `shop_list_customers`, `shop_get_customer`
- `shop_list_orders`
- `shop_get_loyalty`, `shop_update_loyalty` (only on validated action)
- `shop_create_push_broadcast` (optional, for Dormant re-engagement)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Customer insights — period: last 12 months

### 👑 VIP (N customers = X% of revenue)
| Customer | Orders | Revenue | Last purchase | Current loyalty |
|----------|--------|---------|---------------|------------------|

### 🔄 Loyal
...

### 😴 Dormant (re-engage)
N customers inactive for > 90 d → estimated lost revenue: $X

### 🆕 New
...

## Recommended actions
- [ ] Upgrade loyalty for 12 VIPs → Gold
- [ ] "We miss you" push to 84 dormants
- [ ] Welcome-back promo -15% expiring 2026-05-15
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Never bulk-update loyalty without line-by-line validation (or explicit
  "yes, apply to all VIPs" confirmation).
- Respect GDPR: never export customer emails to an unrequested output.
- If the report implies a push, hand off to `shop-promo-campaign` or
  `shop-push-broadcast` for the send — no shortcut.
- **Large customer base**: if fetching every customer would time out,
  sample or narrow to a segment (e.g. last 12 months active).

## Next possible actions
- Run `shop-promo-campaign` to generate VIP/dormant-targeted codes.
- Run `shop-push-broadcast` to send the "we miss you" push to dormants.
- Run `shop-best-sellers` to pick the product to anchor the winback
  campaign on.
