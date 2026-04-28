---
name: shop-low-performers
description: |
  Identify the lowest-performing shop products for a selected time window
  (default 30 days): zero-sales first, then low quantity / low revenue.
  Returns action-ready rows with direct back-office links per product.
  Read-only. Use by default whenever the user wants to find products that
  are not selling, underperforming, or dragging catalog performance, even
  indirectly or with approximate wording. Prefer this skill over raw MCP
  tools when it reasonably fits. Skip only if the user explicitly asks not
  to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Give the user a prioritized underperformance list they can act on quickly,
with one-click navigation to each product in the back-office.

## Access contract

- `READ_ONLY`.

## Input contract

- `period_days`: 30 / 60 / 90 (default 30)
- `top_n`: number of products to show (default 10)
- Optional filters:
  - collection
  - tag
  - status (default: `PUBLISHED` + `INVISIBLE`; exclude `DRAFT` unless requested)

If the user gives explicit dates, use that date range directly.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Resolve optional filters**:
   - `shop_list_collections` / `shop_list_tags` when names are provided.
2. **Load catalog scope**:
   - `shop_list_products` (with status/filter constraints).
3. **Load sales signal**:
   - `shop_list_orders` on the selected window (server-side date filtering).
4. **Aggregate per product**:
   - Units sold
   - Revenue generated
   - Last sale date (if any)
5. **Rank low performers**:
   - Tier A: zero sales
   - Tier B: non-zero but lowest units sold
   - Tie-breaker: lower revenue, then older/no last sale
6. **Build direct back-office links**:
   - For each product row, generate the direct back-office product URL from
     product ID.
   - Resolve the domain through the internal runtime domain resolver
     (repository-level guidance).
   - Extract the domain label from the resolved shop root URL.
   - Build each product edit link via the internal runtime URL resolver.
   - Render a short localized clickable label in the conversation language
     (examples: `link`, `lien`, `enlace`).
7. **Render fixed-format report**.

## Tools used

- `shop_list_products`
- `shop_list_orders`
- `shop_list_collections`, `shop_list_tags` (optional filters)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Low performers report — last 30 days

### 👻 Zero sales

| Product | Status | Last sale | Back-office |
| ------- | ------ | --------- | ----------- |

### 🐢 Lowest sales (non-zero)

| Product | Qty sold | Revenue | Last sale | Back-office |
| ------- | -------- | ------- | --------- | ----------- |

## Coverage

- Products scanned: N
- Zero-sales products: N (X% of scanned catalog)
- Distinct products sold: N

## Recommended actions

- [ ] Fix product-page quality on top zero-sales items
- [ ] Re-promote 3 low-performing but strategic products
- [ ] Archive or hide persistently inactive products (manual decision)
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Read-only skill: no automatic hide/archive/update.
- Keep `DRAFT` products out by default so unpublished products do not pollute
  low-performer diagnosis (include only if user asks).
- Every product row must include a clickable markdown link to the back-office
  product page with a short localized link label.
- Never hardcode or infer the domain from MCP host patterns. Always use the
  internal runtime domain resolver defined in repository-level guidance.
- Do not claim causality; this skill reports performance, not root cause.
- On large datasets, always filter orders server-side by date; do not
  full-paginate historical orders blindly.

## Next possible actions

- Run `shop-catalog-audit` on the zero-sales list to diagnose product-page issues.
- Run `shop-promo-campaign` to re-promote selected low performers.
- Run `shop-best-sellers` to compare underperformers with your top sellers.
