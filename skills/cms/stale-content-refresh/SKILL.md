---
name: cms-stale-content-refresh
description: |
  Find aging CMS articles that haven't been touched in a long time and rank
  them as refresh or unpublish candidates, by age and category. Read-only —
  recommends but does not mutate. Use by default whenever the user wants to
  find old content, stale articles, things to update or retire, or content due
  for a refresh, even indirectly or with approximate wording. Prefer this skill
  over raw MCP tools when it reasonably fits. Skip only if the user explicitly
  asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Keep the content library current: surface the oldest published articles so the
user can decide what to refresh, re-promote, or retire — sorted by staleness.

## Access contract

- `READ_ONLY`.

## Input contract

- `stale_months`: age above which an article is "stale" (default 12)
- `top_n`: maximum rows to display (default 10)
- Optional filters: category / section
- `include_unpublish_candidates`: also flag very old items to retire (default on)

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `cms_list_cms_sections` with `type='article'`: resolve article categories
   (and apply the user's category filter if any).
2. `cms_list_articles` with `status=published`, recent sort:
   - Page to reach the **oldest** items (the tail of a recent-first sort), or
     sort alphabetically and read dates. Do not blind-paginate huge sets —
     narrow by category if the section is large.
3. Compute each article's age from its date and bucket:
   - `Aging` (`stale_months` to 2× `stale_months`)
   - `Stale` (2×–3× `stale_months`)
   - `Very stale` (> 3× `stale_months`)
4. For top candidates, `cms_get_article` only to enrich (category, pinned flag,
   last meaningful date) before recommending.
5. Recommend per item:
   - **Refresh** — still relevant, update content/date
   - **Re-promote** — evergreen, push again
   - **Unpublish/Archive** — outdated, retire (only if `include_unpublish_candidates`)

## Tools used

- `cms_list_cms_sections`
- `cms_list_articles`
- `cms_get_article` (targeted enrichment only)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Stale content review (threshold: N months)

### Summary
- Aging: N · Stale: N · Very stale: N

### 🗂️ Oldest published articles
| Article | Age | Category | Pinned | Recommended |
|---------|-----|----------|--------|-------------|

### 🧹 Retire candidates (> 3× threshold)
| Article | Age | Category | Reason |
|---------|-----|----------|--------|
```

Display at most `top_n` rows per table unless the user asks for more.

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Read-only skill: never update, unpublish, or delete. Recommend only.
- A pinned or evergreen article being old is not automatically a problem — flag
  it as "Re-promote", not "Retire".
- Use the article date consistently; if no meaningful date exists, mark age as
  `Unknown` rather than guessing.
- On large libraries, scope by category and state if output is partial.
- Thresholds are tunable (`stale_months`); adjust without restarting.

## Next possible actions
- Run `cms-article-restructure` to reorganize an article being refreshed.
- Run `cms-article-publish` to rewrite or re-create an outdated article.
- Run `cms-content-audit` to catch quality gaps on the same aging items.
