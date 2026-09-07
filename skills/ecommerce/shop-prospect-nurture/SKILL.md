---
name: shop-prospect-nurture
description: |
  Analyze shop prospects (non-converted visitors), prioritize follow-ups,
  update sales notes. Produces a call queue sorted by potential. Modifies
  prospect notes only on request. Use by default whenever the user wants to
  review shop leads, prospects, follow-ups, or who to contact next, even
  indirectly or with approximate wording. Prefer this skill over raw MCP
  tools when it reasonably fits. Skip only if the user explicitly asks not
  to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Get prospects out of the silo: know who to call first and enrich notes as
interactions happen.

## Access contract

- `READ_ONLY` for analysis and prospect reads.
- `READ_WRITE` if the user wants to update a note (tool
  `shop_update_prospect_note` if available — otherwise flag the
  limitation).

## Default prioritization rules

- **Hot**: signed up < 7 d, has not purchased
- **Warm**: signed up between 7 and 30 d
- **Cold**: > 30 d without action
- **Lost**: > 90 d without interaction, archive

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

### "Produce the queue" mode
1. `shop_list_prospects` filtered server-side when possible
   (`updated_at`, status). Do not full-paginate on large bases; narrow
   to the recent window first.
2. For each hot / follow-up prospect: `shop_get_prospect` to read
   existing notes.
3. Parse notes to detect dated reminders (user convention).
4. Render the sorted list.

### "Update a note" mode
1. `shop_get_prospect` to show the current note.
2. Propose the amended note (timestamped append, don't overwrite history).
3. Confirm.
4. Update the note via the available tool.
5. Re-get to verify.

## Tools used

- `shop_list_prospects`
- `shop_get_prospect`
- (Optional per API) prospect note update — verify via
  `meta_get_tool_plan` whether the tool exists shop-side

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Shop prospects — follow-up queue

### 🔥 Hot (call within 48h) — N prospects
| Name | Signed up | Source | Existing notes |
|------|-----------|--------|-----------------|

### 🌡️ Warm — N prospects
...

### ❄️ Cold — re-engage via push/email
N prospects → suggestion: "discover offer -10%" push

### 🗑️ Archive
N prospects → > 90 d without action
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- No automatic note modifications. Every update is explicit.
- Do not export emails to an unrequested output (GDPR).
- If the shop API does not expose a note update tool, flag it and
  recommend handling the note outside the app.
- **Name → ID resolution**: if the user refers to a prospect by name,
  fuzzy-match on the list and ask them to pick if ambiguous.

## Next possible actions
- Run `shop-push-broadcast` to send the "discover us" push to the Cold
  bucket.
- Run `shop-promo-campaign` to generate a welcome code for the Hot
  bucket.
- Run `shop-customer-insights` to see whether any prospects have in fact
  converted.
