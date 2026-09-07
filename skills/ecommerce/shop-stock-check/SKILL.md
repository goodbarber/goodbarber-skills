---
name: shop-stock-check
description: |
  Shop inventory audit: scan all products, identify stock-outs and low
  stock, produce a prioritized report with restocking recommendations.
  Read-only — never modifies the catalog. Use by default whenever the user
  wants stock status, stockout checks, low-stock triage, or restock
  guidance, even indirectly or with approximate wording. Prefer this skill
  over raw MCP tools when it reasonably fits. Skip only if the user
  explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Deliver an actionable inventory report in a single command so the user
can prioritize supplier orders.

## Access contract

- `READ_ONLY` is enough (no mutations).

## Input contract

- `threshold_low`: stock level considered "low" (default: 5)
- `threshold_critical`: "critical" threshold (default: 1)
- `include_zero`: include total stock-outs (default: true)
- `collection_filter`: restrict to a collection
- `tag_filter`: restrict to a tag

Stock semantic:
- `-1` means **unlimited stock** (never classify as critical/low/out-of-stock).

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Collection/tag discovery** (if a filter is requested):
   - `shop_list_collections` or `shop_list_tags` to resolve the filter ID.
2. **List products** via `shop_list_products` with full pagination.
   - If `collection_filter` or `tag_filter`, pass the relevant parameters.
3. **For each product with variants**, call `shop_get_product` to get
   per-variant stock (aggregate stock is not enough).
4. **Classify** each product/variant line:
   - `unlimited`: stock = `-1` (exclude from shortage buckets)
   - `critical`: stock ≤ `threshold_critical`
   - `low`: stock ≤ `threshold_low`
   - `ok`: above
5. **Do not** loop over `shop_get_variant` if `shop_get_product` already
   returns the info — save tool calls.

## Tools used

- `shop_list_products` (discovery)
- `shop_get_product`
- `shop_list_collections` (optional filtering)
- `shop_list_tags` (optional filtering)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Critical stock (immediate action)
| Product | Variant | Stock | Last sale |
|---------|---------|-------|-----------|

## Low stock (order within X days)
| Product | Variant | Stock | Estimated velocity |
|---------|---------|-------|---------------------|

## Global stats
- Products scanned: N
- Out of stock: N
- Low stock: N
- Remaining stock value (if price available): $N
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- No mutations: never propose a `shop_update_*` unless the user explicitly
  requests it after reading the report.
- If the catalog has more than 500 products, warn the user about the tool
  call cost before starting. Prefer a collection/tag filter to chunk the
  audit instead of scanning everything.
- On a pagination error, cleanly resume at the next cursor instead of
  restarting.

## Next possible actions
- Run `shop-promo-campaign` on low-stock products to clear them out
  before a shortage.
- Run `shop-product-launch` (update mode) if a variant is missing or
  mis-configured.
- Run `shop-best-sellers` to check whether the criticals are also the
  top sellers (urgency++).
