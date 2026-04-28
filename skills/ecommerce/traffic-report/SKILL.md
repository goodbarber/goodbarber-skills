---
name: shop-traffic-report
description: |
  Consolidated shop analytics report: page views, launches, sessions,
  weekday patterns, downloads. Read-only. Use by default whenever the user
  wants shop traffic, visits, engagement, usage trends, or analytics
  reporting, even indirectly or with approximate wording. Prefer this skill
  over raw MCP tools when it reasonably fits. Skip only if the user
  explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

One command → a full traffic overview of the shop, ready to paste into a
report or share in a meeting.

## Access contract

- `READ_ONLY`.

## Input contract

- `period_days`: 7 / 30 / 90 (default 30)
- `compare_previous`: true to show the delta vs previous period
  (default true)

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Parallel analytics calls**:
   - `shop_list_page_views` (current + previous period for delta)
   - `shop_list_launches`, `shop_list_unique_launches`
   - `shop_list_session_times`
   - `shop_list_downloads_global`, `shop_list_downloads`
   - `shop_list_page_views_per_weekday`
2. **Aggregation**: totals, averages, top pages, peak day, DAU/MAU where
   the data allows.
3. **Delta vs previous period** if requested.
4. **Markdown report**.

## Tools used

- `shop_list_page_views`, `shop_list_launches`, `shop_list_unique_launches`
- `shop_list_session_times`, `shop_list_downloads`,
  `shop_list_downloads_global`
- `shop_list_page_views_per_weekday`

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Shop traffic report — last 30 days (vs previous 30 d)

### 📊 Volumes
| Metric | Period | Prev. | Delta |
|--------|--------|-------|-------|
| Page views | 124,320 | 108,445 | +14.6% |
| Launches | 18,200 | 17,002 | +7.0% |
| Unique launches | 4,210 | 3,988 | +5.6% |
| Avg session | 2m14 | 2m08 | +4.7% |

### 📅 Weekday patterns
- Busiest day: Tuesday (21% of traffic)
- Quietest day: Sunday

### 📥 Downloads
- New downloads in period: N (+X% vs prev.)
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **No per-page breakdown**: `shop_list_page_views` returns
  `{total_page_views, history}` — a global daily aggregate only. Do
  not invent a "most viewed pages" table; report totals and trends,
  not per-page numbers.
- If a metric is unavailable (API returns nothing), render it as "n/a"
  instead of inventing a number.
- Respect pagination on large analytics lists.
- Never compare windows of different sizes without warning.

## Next possible actions
- Run `shop-best-sellers` to link the most-viewed pages back to sales.
- Run `shop-weekly-digest` to publish this as part of the weekly recap.
- Run `shop-push-broadcast` if a traffic drop calls for a re-engagement.
