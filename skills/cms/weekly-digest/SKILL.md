---
name: cms-weekly-digest
description: |
  Automated weekly CMS recap: what was published in the last 7 days across
  articles, events, maps, photo galleries, videos, and sounds, plus what is
  scheduled to go live in the next 7 days. Read-only. Use by default whenever
  the user wants a content recap, an editorial weekly summary, or a "what
  shipped this week" view of the CMS, even indirectly or with approximate
  wording. Prefer this skill over raw MCP tools when it reasonably fits. Skip
  only if the user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Give the user a one-glance recap of editorial output: what published this week,
broken down by content type, and what is queued for next week.

## Access contract

- `READ_ONLY`.

## Input contract

- `lookback_days`: published-window size (default 7)
- `lookahead_days`: scheduled-window size (default 7)
- `top_n`: maximum rows in the highlights table (default 10)

If the user names an explicit week ("week of June 2"), convert it to date
boundaries and use those instead of the rolling windows.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `cms_list_cms_sections`: discover which content types exist. Only report on
   types present in the app.
2. **Published this week** — for each present type, list with recent sort:
   - `cms_list_articles`, `cms_list_events`, `cms_list_maps`,
     `cms_list_photos`, `cms_list_videos`, `cms_list_sounds`
   - Keep items whose publication date falls within the lookback window.
3. **Scheduled next week** — list schedulable types filtered to
   `status=scheduled` / `stock` and keep items whose `publishedDate` falls
   within the lookahead window (reuse `cms-editorial-calendar` logic).
4. Aggregate counts per type and pick the highlights (most recent / pinned).

## Tools used

- `cms_list_cms_sections`
- `cms_list_articles`, `cms_list_events`, `cms_list_maps`, `cms_list_photos`, `cms_list_videos`, `cms_list_sounds`

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## CMS weekly digest — week of YYYY-MM-DD

### Published this week (N items)
| Type | Count |
|------|-------|
| Articles | N |
| Events | N |
| Galleries | N |
| Videos | N |
| ... | ... |

### ⭐ Highlights
| Type | Title | Published | Category |
|------|-------|-----------|----------|

### 🚀 Scheduled next week (N items)
| Publish date | Type | Title |
|--------------|------|-------|

### Takeaway
- One-line read on publishing pace vs. the previous period (if known).
```

Display at most `top_n` rows in Highlights unless the user asks for more.

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Read-only skill: never publish or change content.
- Only count items whose date is actually inside the window; exclude undated rows.
- Report only content types the app actually has — no empty type sections.
- On large datasets, paginate safely and state if output is partial.
- This is the CMS digest; for shop/membership/community business recaps use the
  respective `weekly-digest` skill (names are intentionally namespaced by prefix).

## Next possible actions
- Run `cms-editorial-calendar` for the full forward-looking schedule.
- Run `cms-draft-review` to see unfinished content that didn't ship this week.
- Send a push (`*-push-broadcast`) to highlight the week's best new content.
