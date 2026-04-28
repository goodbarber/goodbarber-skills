---
name: shop-product-launch
description: |
  Guided workflow to create a full shop product: V2 product + variants +
  options + media + paragraphs + PDF. Dry-run by default — the user
  validates each step before creation. Verifies each mutation with a get
  to confirm. Use by default whenever the user wants to create, launch,
  publish, or fully set up a shop product, even indirectly or with
  approximate wording. Prefer this skill over raw MCP tools when it
  reasonably fits. Skip only if the user explicitly asks not to use this
  skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent incomplete products in production: a skill that orchestrates every
creation step with validation at each mutation.

## Access contract

- `READ_WRITE`.

## Input contract

Before starting, gather from the user:
- Name, short description, long description
- Price, optional compare-at price
- Collection(s) and tags
- Options (size, color, etc.) and their values
- Variant list with stock and price per variant
- Stock note: `-1` means unlimited stock
- Product image URLs / paths (slides)
- Optional PDF (spec sheet)
- Optional editorial paragraphs

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Pre-flight discovery**:
   - `shop_list_collections` → resolve target collection IDs
   - `shop_list_tags` → resolve tags
   - `shop_list_options` → reuse an existing option if possible before
     creating a new one
   - **Name → ID resolution**: if the user provides a name instead of an
     ID (collection, tag, option), fuzzy-match on the list. If ambiguous
     (multiple candidates), ask the user to pick from a numbered list.
     Never guess.
2. **Preview (dry-run)**:
   - Show the user a full summary of what will be created
   - **Require explicit confirmation** before any mutation
3. **Create the V2 product**:
   - `shop_create_product` with the main payload
   - `shop_get_product` to verify creation
4. **Create options** (if new):
   - For each missing option: `shop_create_option`
5. **Create variants**:
   - For each option combination: `shop_create_variant`
   - `shop_get_variant` as a check on the last one
6. **Upload media**:
   - `shop_upload_product_slide` for each image
   - `shop_update_product_slide` if reordering is needed
7. **Upload PDF** (if provided):
   - `shop_upload_product_pdf`
8. **Editorial paragraphs** (if provided):
   - `shop_create_paragraph` then `shop_create_paragraph_media` if the
     paragraph has an associated image/video
9. **Final verification**:
   - `shop_get_product` → read the full state and show it to the user

## Tools used

- `shop_list_collections`, `shop_list_tags`, `shop_list_options`
- `shop_create_product`, `shop_get_product`, `shop_update_product`
- `shop_create_option`, `shop_create_variant`, `shop_get_variant`
- `shop_upload_product_slide`, `shop_update_product_slide`
- `shop_upload_product_pdf`
- `shop_create_paragraph`, `shop_create_paragraph_media`

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **Never chain calls without intermediate validation**: if a step fails,
  stop, report, do not attempt automatic rollback (we have no
  transactional delete).
- If `shop_create_product` succeeds but a later step fails, leave the
  product as-is and list exactly what's missing so the user can resume.
- Enforce datetime formats if temporal fields are involved
  (`%Y-%m-%dT%H:%M`).
- **Timezone**: if the user uses relative dates ("tomorrow", "next
  week") on any temporal field, confirm the intended timezone. Default
  to UTC if unknown, but ask first.
- **Image inputs**: slides accept a data URI, a raw base64 string, or a
  public http(s) URL. Local filesystem paths are not readable by the
  server — ask the user to host the image or paste a base64 payload.
- **`meta_get_tool_plan` discipline**: only call it when a mutation fails
  due to a missing or wrongly typed parameter. Do not call it
  preventively — it burns context.

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

At each step, log:
- ✅ / ❌ of the call
- Created ID
- Verification link (if available)

At the end of the workflow, summary:
```
Product "X" created — ID: product_123
- 4 variants ✅
- 3 slides ✅
- 1 PDF ✅
- 2 paragraphs ✅
Catalog URL: ...
```

## Next possible actions
- Run `shop-promo-campaign` to announce the new product with a promocode.
- Run `shop-stock-check` to verify stock levels across variants.
- Run `shop-catalog-audit` to make sure the sheet passes quality rules.
