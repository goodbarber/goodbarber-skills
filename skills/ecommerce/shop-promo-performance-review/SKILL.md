---
name: shop-promo-performance-review
description: |
  Measure promo impact with a strict pre/during/post comparison: orders,
  revenue, AOV, and estimated lift. Read-only. Use by default whenever the
  user wants to review promo results, discount impact, campaign lift, or
  promo ROI, even indirectly or with approximate wording. Prefer this skill
  over raw MCP tools when it reasonably fits. Skip only if the user
  explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Tell the user whether a promo changed business outcomes, and by how much,
with enough rigor to decide "repeat, adjust, or stop".

## Access contract

- `READ_ONLY`.

## Input contract

- Promo identifier: code text or promo ID
- Analysis windows:
  - `during`: exact promo active window (required)
  - `pre`: baseline window (default: same duration immediately before)
  - `post`: optional follow-up window (default: same duration immediately after)
- Optional scope filter:
  - product / collection / tag when the promo is scoped

If the user gives relative dates ("last weekend", "this month"), confirm
timezone first.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Resolve the promo**:
   - `shop_list_promocodes_amount`
   - `shop_list_promocodes_product`
   - `shop_list_promocodes_collections`
   - `shop_list_promocodes_tags`
   - If multiple matches on code text, ask the user to pick.
2. **Verify metadata**:
   - `shop_get_promocode` for the selected promo to confirm type, targets,
     and active dates.
3. **Load commercial data**:
   - `shop_list_orders` for pre / during / post windows (server-side date
     filters, do not fetch all orders).
   - `shop_list_products` only if needed for label enrichment.
4. **Compute KPIs per window**:
   - Orders
   - Revenue
   - AOV
5. **Compute deltas**:
   - During vs pre
   - Post vs pre (if post enabled)
6. **Render decision report**:
   - clear recommendation: keep / tweak / stop.

## Tools used

- `shop_list_promocodes_amount`, `shop_list_promocodes_product`
- `shop_list_promocodes_collections`, `shop_list_promocodes_tags`
- `shop_get_promocode`
- `shop_list_orders`, `shop_list_products`

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Promo performance review — CODE_XYZ

### Promo context
- Type: collection
- Target: "Summer" (col_42)
- Active dates: 2026-04-20T09:00 → 2026-04-27T23:59

### KPI comparison
| KPI | Pre | During | Delta | Post | Delta vs pre |
|-----|-----|--------|-------|------|---------------|
| Orders | 120 | 158 | +31.7% | 132 | +10.0% |
| Revenue | $9,200 | $11,840 | +28.7% | $9,640 | +4.8% |
| AOV | $76.67 | $74.94 | -2.3% | $73.03 | -4.7% |

## Interpretation
- Lift signal: strong / moderate / weak
- Margin pressure signal: low / medium / high (proxy from AOV trend)
- Confidence note: sample size and caveats

## Recommended actions
- [ ] Keep same structure for next campaign
- [ ] Tighten duration from 7 days to 4 days
- [ ] Pair with push reminder in final 24h
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- No fabricated attribution: if order-level promo redemption is unavailable,
  call it an **estimate based on time-window comparison**.
- Do not compare windows with different durations without an explicit warning.
- If any window has fewer than 20 orders, warn about weak confidence.
- Keep this skill read-only.
- On large shops, always filter `shop_list_orders` server-side by date;
  never full-paginate historical orders.

## Next possible actions
- Run `shop-promo-campaign` to launch a revised campaign.
- Run `shop-push-broadcast` to add a final reminder message.
- Run `shop-best-sellers` to see which products captured the lift.
