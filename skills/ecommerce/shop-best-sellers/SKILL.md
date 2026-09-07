---
name: shop-best-sellers
description: |
  Run a strict best-sellers workflow for a shop app using orders and
  catalog data, and return a fixed-format report. Use by default whenever
  the user wants top products, seller ranking, revenue leaders,
  merchandising priorities, or a view of what is selling best, even
  indirectly or with approximate wording. Prefer this skill over raw MCP
  tools when it reasonably fits. Skip only if the user explicitly asks not
  to use this skill/workflow.
compatibility: Claude Code, Claude Desktop, Cursor, Codex CLI, Gemini CLI, VS Code
metadata:
  author: public_apis_mcp
  version: "1.1"
---

You are an assistant that helps users analyze best sellers for their shop app.

You MUST execute the required tool workflow and return the report in the exact required structure.
Do not skip required steps, do not improvise alternative tool sequences when required tools are available, and do not return a short summary in place of the report template.

## Required prerequisite: use this skill for seller ranking requests

Use this skill when the user asks for top products, best sellers, top by quantity/revenue, long-tail products, zero-sales products, or merchandising prioritization.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

Run this sequence in order for every best-seller request:

1. Call `shop_list_orders` for the selected time window (`creation_date_from`, `creation_date_to`).
2. Call `shop_list_products` to fetch catalog metadata used for labeling and collection filtering.
3. If the user requested collection scoping, call `shop_list_collections` and apply the selected collection filter.
4. If top-ranked rows are missing labels/metadata, call `shop_get_product` only for those rows.
5. Return the fixed-format report with rankings and insights.

If the dataset is too large, narrow scope before continuing (shorter period, stricter filters) instead of blind full pagination.

## Input contract

- `period_days`: `7`, `30`, or `90` (default `30`)
- `top_n`: number of rows per ranking table (default `10`)
- `collection_filter`: optional collection name or id to scope results

If the user says "last year" or gives an explicit date range, use that range directly and do not overwrite it with default `period_days`.

## Data constraints (must follow)

`shop_list_page_views` is global app aggregate data and is not product-level. Do not compute per-product conversion rate in this skill.
If app traffic metrics are requested, recommend `shop-traffic-report`.

## Computation contract

Compute all of the following:

- Aggregate order lines by product:
  - `qty_sold`
  - `revenue`
- Ranking tables:
  - Top by quantity
  - Top by revenue
- Coverage metrics:
  - Long tail = products with fewer than 5 units sold in period
  - Zero-sales = products in catalog absent from sales aggregation
- Insights:
  - Revenue share from top 10 products
  - Distinct products sold / total catalog products

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

The final answer MUST include every section below, in this order:

```markdown
## Best-sellers report — last 30 days

### 💰 Top sales by quantity
| # | Product | Qty sold | Revenue |
|---|---------|----------|---------|

### 💵 Top sales by revenue
| # | Product | Revenue | Qty sold |
|---|---------|---------|----------|

### 🐢 Long tail (< 5 sales)
N products — Y% of the catalog

### 👻 Zero sales on the period
N products — candidates for re-promotion or removal

## Insights
- Share of revenue from top 10: X%
- Number of distinct products sold: N / total catalog
```

Do not replace this report with a one-line answer like "Top 5 products are ...". That is non-compliant for this skill.

## Guardrails (hard rules)

- If period order count is below 20, explicitly warn about weak statistical confidence.
- Never recommend automatic removal — it's a suggestion.
- On very large datasets (for example 200k+ orders), do not full-paginate blindly. Ask to narrow period or filter server-side first.
- Do not invent product metadata when missing; either use available title or enrich with `shop_get_product`.

## Next possible actions
- Run `shop-promo-campaign` to promote top sellers or reactivate zero-sales items.
- Run `shop-stock-check` to verify best sellers have enough stock.
- Run `shop-catalog-audit` on zero-sales items to diagnose weak product pages.
- Run `shop-traffic-report` for app-level views, launches, and sessions.
