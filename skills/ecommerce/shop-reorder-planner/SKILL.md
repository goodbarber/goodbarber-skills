---
name: shop-reorder-planner
description: |
  Build a supplier-ready reorder queue from current stock and recent sell
  velocity. Prioritizes what to reorder now, soon, or monitor. Read-only.
  Use by default whenever the user wants reorder, restock, purchasing,
  stock-cover, or supplier-planning help, even indirectly or with
  approximate wording. Prefer this skill over raw MCP tools when it
  reasonably fits. Skip only if the user explicitly asks not to use this
  skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Turn inventory snapshots into a practical replenishment plan that saves the
user from manual SKU-by-SKU planning.

## Access contract

- `READ_ONLY`.

## Input contract

- `velocity_period_days`: 30 / 60 / 90 (default 30)
- `target_cover_days`: desired inventory cover (default 21)
- Optional filters:
  - collection
  - tag
- Optional exclusions:
  - unlimited stock (`-1`) excluded by default

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Resolve optional filters**:
   - `shop_list_collections` / `shop_list_tags` if names are provided.
2. **Load catalog and stock state**:
   - `shop_list_products` (with filters if requested).
   - `shop_get_product` for products with variants to retrieve precise
     per-variant stock.
3. **Load demand signal**:
   - `shop_list_orders` on the velocity window (server-side date filters).
4. **Compute per SKU (product or variant)**:
   - Current stock
   - Units sold in period
   - Daily velocity = units_sold / period_days
   - Days of cover = stock / daily_velocity (if velocity > 0)
   - Suggested reorder qty = max(0, target_cover_days \* daily_velocity - stock)
5. **Prioritize**:
   - Reorder now
   - Reorder soon
   - Monitor
6. **Build product back-office links**:
   - For each row, build a direct product URL to the back-office product page.
   - Use the shop product ID returned by `shop_list_products` / `shop_get_product`.
   - Render a short clickable label translated to the conversation language
     (examples: `link`, `lien`, `enlace`, `link` in German).
7. **Render buyer-ready queue**.

## Tools used

- `shop_list_products`, `shop_get_product`
- `shop_list_orders`
- `shop_list_collections`, `shop_list_tags` (optional filters)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Reorder planner — target cover: 21 days

### 🔴 Reorder now

| Product / Variant | Stock | 30d sold | Daily velocity | Days of cover | Suggested reorder | Back-office |
| ----------------- | ----- | -------- | -------------- | ------------- | ----------------- | ----------- |

### 🟠 Reorder soon

| Product / Variant | Stock | 30d sold | Daily velocity | Days of cover | Suggested reorder | Back-office |
| ----------------- | ----- | -------- | -------------- | ------------- | ----------------- | ----------- |

### 🟢 Monitor

N SKUs with sufficient cover

## Purchasing summary

- SKUs to reorder now: N
- Total suggested units: N
- Highest urgency SKU: ...
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- `-1` stock means unlimited: exclude from shortage and reorder math.
- If a SKU has zero sales in the selected period, do not force reorder;
  classify as monitor unless user asks for minimum stock policy.
- Keep calculations explicit and rounded consistently (state the rounding).
- Every product row in Reorder now / Reorder soon must include a clickable
  markdown link to the back-office product page.
- The clickable text must be short and localized to the conversation
  language (for example: `link`, `lien`, `enlace`).
- On large order tables, narrow by date and filters server-side; do not
  full-paginate historical data.
- Read-only skill: no catalog or order mutation.

## Next possible actions

- Run `shop-stock-check` for a deeper shortage-focused audit.
- Run `shop-best-sellers` to validate demand assumptions.
- Run `shop-promo-performance-review` before increasing stock for promo-driven SKUs.
