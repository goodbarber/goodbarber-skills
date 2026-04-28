---
name: shop-orphan-products
description: |
  Find orphan products (products with no assigned collection), return them in
  a table with direct back-office links, and propose a practical collection
  assignment plan based on similar catalog items. Read-only.
  Use by default whenever the user wants to find products missing
  collections or clean up catalog organization, even indirectly or with
  approximate wording. Prefer this skill over raw MCP tools when it
  reasonably fits. Skip only if the user explicitly asks not to use this
  skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Detect every product not attached to a collection and provide actionable
assignment suggestions so the user can clean catalog structure quickly.

## Access contract

- `READ_ONLY`.
- `READ_WRITE` only if the user asks to apply collection assignments.

## Input contract

- Optional `status` filter:
  - default: `PUBLISHED` only
  - supported overrides: `INVISIBLE`, `DRAFT`, `DEMO`, or any explicit
    status combination requested by the user
- Optional `top_n` to limit displayed rows (default: all orphan products)
- Optional "strict mode":
  - suggest only existing collections with strong similarity
  - otherwise allow "create a new collection" recommendations
- Optional apply mode after report:
  - `high_only`: apply only High confidence recommendations
  - `medium_and_above`: apply Medium + High
  - `all`: apply every recommendation
  - `manual`: user provides specific product + one or more collection targets

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Load catalog**:
   - `shop_list_products` with requested status scope.
2. **Load collection reference**:
   - `shop_list_collections` to build candidate targets and known themes.
3. **Detect orphan products**:
   - Product is orphan if `collections` is empty or missing.
4. **Find assignment candidates** for each orphan:
   - Use title, tags, description keywords, and nearby product naming patterns.
   - If obvious match to an existing collection exists, recommend that one.
   - If no reliable match, recommend a new collection theme.
5. **Build direct back-office links**:
   - Resolve the shop root domain through the internal runtime domain resolver
     (repository-level guidance).
   - Extract the domain label from the resolved shop root URL.
   - Build each product edit link via the internal runtime URL resolver.
   - Render a short localized clickable label in the conversation language
     (examples: `link`, `lien`, `enlace`).
6. **Render fixed-format report**:
   - orphan table + assignment plan table.
   - If no explicit status filter was requested, add a short note that this
     run only includes `PUBLISHED` products and mention that non-published
     statuses can be included on request.
7. **Prompt for assignment action**:
   - Ask the user whether to apply:
     - assignment plan as `high_only`, `medium_and_above`, or `all`
     - manual mapping (`product -> one or more collections`)
   - If user confirms an apply mode, switch to apply flow:
     1. Resolve collection names to IDs via `shop_list_collections`.
     2. Resolve product names to IDs from the orphan table.
     3. Show dry-run change list (product, current collections, target collections).
     4. Require explicit confirmation.
     5. Apply product updates with `shop_update_product` one product at a time.
     6. Verify each updated product with `shop_get_product`.

## Tools used

- `shop_list_products`
- `shop_list_collections`
- `shop_update_product` (only in explicit apply flow)
- `shop_get_product` (optional, only when deeper fields are needed)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Orphan products report

### 📦 Products without collection
| Product | Status | Tags | Back-office |
|---------|--------|------|-------------|

## Assignment plan
| Product | Suggested collection | Why this fit | Confidence |
|---------|-----------------------|--------------|------------|

## Coverage
- Products scanned: N
- Orphan products: N (X% of scanned catalog)
- Existing collections reviewed: N

## Apply assignments?
- Choose one:
  - [ ] Apply `high_only`
  - [ ] Apply `medium_and_above`
  - [ ] Apply `all`
  - [ ] Manual mapping (specify product + collection(s))
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Read-only skill: do not assign collections automatically.
- Every orphan row must include a clickable back-office link.
- Never hardcode or infer domain from MCP host patterns. Always use the
  internal runtime domain resolver defined in repository-level guidance.
- If confidence is low, say so explicitly and propose 2 candidate collections
  instead of a single definitive assignment.
- If no suitable existing collection is found, recommend "create a new
  collection" with a concrete suggested name.
- When status is not explicitly provided by the user, default to
  `PUBLISHED` only and state this scope in the final output.
- Never mutate without explicit confirmation on a dry-run plan.
- For `manual` mode, if product or collection names are ambiguous, ask the
  user to choose from numbered candidates; never guess.
- In apply mode, process updates one product at a time and verify each update.

## Next possible actions
- Run `shop-catalog-audit` to fix additional catalog quality issues.
- Run `shop-low-performers` to see whether orphan products are also underperforming.
- Run `shop-product-launch` (update mode) to apply collection assignments manually.
