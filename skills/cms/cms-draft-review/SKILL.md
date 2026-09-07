---
name: cms-draft-review
description: |
  Surface unfinished CMS content: drafts and never-published items across
  articles, events, maps, and videos, grouped by type and age, with a
  recommended action (publish, finish, or delete) for each. Read-only —
  recommends but does not mutate. Use by default whenever the user wants to
  review drafts, find unpublished content, clean up work-in-progress, or see
  what is sitting unfinished, even indirectly or with approximate wording.
  Prefer this skill over raw MCP tools when it reasonably fits. Skip only if
  the user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Give the user a clear backlog of unfinished CMS content so nothing stays stuck
in draft forever — each item paired with a concrete next step.

## Access contract

- `READ_ONLY`.

## Input contract

- `stale_after_days`: age threshold above which a draft is "stale" (default 30)
- `top_n`: maximum rows per table (default 10)
- Optional filters: content type, category / section

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `cms_list_cms_sections`: discover which content types exist. Review only the
   types present (articles, agenda, maps, video). **Photos have no `draft`
   status — exclude them.**
2. For each present type, list drafts:
   - `cms_list_articles`, `cms_list_events`, `cms_list_maps`, `cms_list_videos`
     filtered to `status=draft`.
3. For each draft, assess completeness against minimum-publish rules:
   - Article/video: has a body / embed and a category
   - Event: has valid `sortDate` / `endDate`
   - Map: has address + coordinates
   - Use `cms_get_*` / `cms_list_*_paragraphs` only to confirm suspect items.
4. Compute age from the item date and bucket:
   - `Fresh` (≤ `stale_after_days`)
   - `Stale` (> `stale_after_days`)
   - `Abandoned` (> 90 days)
5. Recommend an action per item:
   - **Publish** — complete and ready
   - **Finish** — incomplete, needs missing fields first
   - **Delete** — abandoned and empty

## Tools used

- `cms_list_cms_sections`
- `cms_list_articles`, `cms_list_events`, `cms_list_maps`, `cms_list_videos`
- `cms_get_*`, `cms_list_*_paragraphs` (targeted confirmation only)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## CMS draft review (N drafts found)

### Summary
- Ready to publish: N · Need finishing: N · Abandoned: N

### 🟢 Ready to publish
| Type | Title | Age (days) | Category |
|------|-------|-----------|----------|

### 🟡 Need finishing
| Type | Title | Age (days) | Missing |
|------|-------|-----------|---------|

### 🔴 Abandoned (> 90 days, empty)
| Type | Title | Age (days) | Recommended |
|------|-------|-----------|-------------|
```

Display at most `top_n` rows per table unless the user asks for more.

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Read-only skill: never publish, update, or delete. Recommend only.
- Photos are excluded (no `draft` status). Do not report them.
- "Abandoned" must be both old **and** effectively empty — do not recommend
  deleting an old-but-complete draft.
- On large datasets, paginate safely and state if output is partial.
- Thresholds are tunable (`stale_after_days`); adjust without restarting.

## Next possible actions
- Run `cms-article-publish` to finish and publish a specific article draft.
- Run `cms-content-audit` to see exactly what each "need finishing" draft lacks.
- Use the relevant `cms_update_*` tool to publish a "ready" draft after review.
