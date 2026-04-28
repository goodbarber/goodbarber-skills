---
name: shop-promo-campaign
description: |
  End-to-end shop promo campaign design: pick the promocode type (amount,
  product, collection, tag), create the code, and send the announcement
  push broadcast. Dry-run required before creation — the user sees exactly
  what will be created. Use by default whenever the user wants to plan or
  launch a promo, discount, sale, offer, or code-based campaign, even
  indirectly or with approximate wording. Prefer this skill over raw MCP
  tools when it reasonably fits. Skip only if the user explicitly asks not
  to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Deliver a coherent campaign: the right promocode type + well-targeted
announcement push, without date format errors or missing targets.

## Access contract

- `READ_WRITE`.

## Input contract

- Promo type: `amount` | `product` | `collection` | `tag`
- Discount value (amount or percentage)
- Target per type:
  - `product` → product ID(s)
  - `collection` → collection ID(s)
  - `tag` → tag ID(s)
  - `amount` → none (global)
- `start_at`, `end_at` (or `end_date: "none"`)
- Max uses, per customer
- Send a push? (yes/no + title + body + optional web URL — standard
  `https://...`, the app uses universal links, never a custom scheme)

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Discovery** (required before creation):
   - If `type=product`: `shop_list_products` to confirm IDs
   - If `type=collection`: `shop_list_collections`
   - If `type=tag`: `shop_list_tags`
   - **Name → ID resolution**: if the user gives a product/collection/tag
     name instead of an ID, fuzzy-match on the list. If ambiguous, ask
     them to pick from a numbered list. Never guess.
2. **Dry-run**: show the full payload to the user, request confirmation.
3. **Create the code** by type:
   - `shop_create_promocode_amount`
   - `shop_create_promocode_product`
   - `shop_create_promocode_collections`
   - `shop_create_promocode_tags`
4. **Verify**: `shop_get_promocode` to confirm.
5. **Push broadcast** (if requested):
   - `shop_create_push_broadcast` with message + code in the body
   - Draft first, user confirms, then send
6. **Summary**: link to the created code, push send time.

## Tools used

- Discovery: `shop_list_products`, `shop_list_collections`, `shop_list_tags`
- Creation: `shop_create_promocode_amount`, `shop_create_promocode_product`,
  `shop_create_promocode_collections`, `shop_create_promocode_tags`
- Verification: `shop_get_promocode`, `shop_list_promocodes_*`
- Communication: `shop_create_push_broadcast`
- `meta_get_tool_plan` if unsure about a format

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Promo campaign — SPRING_SALE_2026

- Type: collection
- Discount: -20%
- Target: collection "Summer" (ID: col_42)
- Dates: 2026-04-20T09:00 → 2026-04-27T23:59
- Max uses: 500 (1 per customer)
- Code created: SPRING20 ✅
- Push sent: "Summer sale starts now!" → 12,430 recipients ✅
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **Never** send the push before the promocode is created and verified
  (otherwise we announce a nonexistent code).
- **Date format**: `start_at` and `end_at` must be `%Y-%m-%dT%H:%M`.
  Reject invalid formats before hitting the API. Use `end_date: "none"`
  for a promo with no expiry.
- **Timezone**: if the user gives relative dates ("this weekend", "next
  Friday"), confirm the intended timezone. Default to UTC if unknown,
  but ask first.
- If `end_at` is in the past, block and ask for confirmation.
- Dry-run every mutation.
- **`meta_get_tool_plan` discipline**: only call it when a mutation fails
  due to a missing or wrongly typed parameter. Do not call it
  preventively — it burns context.

## Next possible actions
- Run `shop-push-broadcast` for a standalone follow-up reminder before the
  promo expires.
- Run `shop-customer-insights` to target dormants with the new code.
- Run `shop-best-sellers` at the end of the promo to measure impact.
