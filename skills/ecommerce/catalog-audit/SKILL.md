---
name: shop-catalog-audit
description: |
  Shop catalog quality audit: detect incomplete products (no image, no
  description, no variant, no tag, no collection, missing price), produce
  a prioritized cleanup todo list. Read-only — suggests fixes but does
  not apply them. Use by default whenever the user wants to audit product
  quality, find incomplete listings, or clean up the catalog, even
  indirectly or with approximate wording. Prefer this skill over raw MCP
  tools when it reasonably fits. Skip only if the user explicitly asks not
  to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Identify all product sheets that hurt conversion (no image, empty
description, no variant…) and deliver a list sorted by impact.

## Access contract

- `READ_ONLY` is enough.

## Default quality rules

A sheet is flagged as problematic if:
- No attached image (slide)
- Empty or < 30-character description
- Price is 0 or missing
- No variant despite the product requiring one (options present)
- No assigned collection
- No assigned tag
- Negative or inconsistent stock (**except `-1`, which means unlimited stock**)

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `shop_list_products`: list all products (full pagination).
2. For each returned product, apply quality rules on the fields already
   available in the list (avoid a `get` if not needed).
3. For "suspect" products (incomplete fields in the list),
   `shop_get_product` to confirm before classifying.
4. Group issues by type so the user can fix in batches (e.g. "add an image
   to these 12 products").

## Tools used

- `shop_list_products`
- `shop_get_product` (targeted, not across the whole catalog)
- `shop_list_tags`, `shop_list_collections` (reference data)
- `shop_list_paragraphs` (if paragraph verification requested)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Catalog quality score: X/10 (N products, M problematic)

### 🔴 Critical (fix first)
- Missing price: N products [IDs...]
- No image: N products [IDs...]

### 🟡 To improve
- Description too short: N products
- No tag: N products

### 🟢 Minor
- No collection: N products

## Top 10 products to fix first
| Product | Issues | Recommended action |
|---------|--------|---------------------|
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Never suggest an edit without explicit permission — the skill is
  read-only by design.
- If the user wants to fix, redirect to `shop-product-launch` or
  individual update tools.
- Quality thresholds are tunable (the user can say "a 10-char description
  is fine"); adjust without restarting the audit.
- **Large catalogs**: if the catalog has 500+ products, warn the user
  about the cost. Prefer filtering by collection/tag to audit in chunks
  rather than fetching everything.

## Next possible actions
- Run `shop-product-launch` to fix a specific product (or recreate a
  missing sheet).
- Run `shop-best-sellers` to check whether the incomplete sheets are
  actually selling (prioritize fixes).
- Run `shop-stock-check` if the audit surfaces many negative-stock
  anomalies.
