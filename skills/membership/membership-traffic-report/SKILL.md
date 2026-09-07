---
name: membership-traffic-report
description: |
  Consolidated membership app analytics: page views, launches, sessions,
  devices, OS distribution, weekday patterns, downloads. Read-only. Use
  by default whenever the user wants traffic, analytics, usage, visits,
  engagement, or an app-performance snapshot, even indirectly or with
  approximate wording. Prefer this skill over raw MCP-tool handling when
  it reasonably fits. Skip only if the user explicitly asks not to use
  this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

One command → a full traffic overview of the membership app, ready to
paste into a report or share in a meeting.

## Access contract

- `READ_ONLY`.

## Input contract

- `period_days`: 7 / 30 / 90 (default 30)
- `compare_previous`: true to show the delta vs previous period
  (default true)

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Parallel analytics calls**:
   - `classic_list_page_views` (current + previous period for delta)
   - `classic_list_launches`, `classic_list_unique_launches`
   - `classic_list_session_times`
   - `classic_list_downloads_global`, `classic_list_downloads`
   - `classic_list_devices_global`
   - `classic_list_mobile_os_distribution`
   - `classic_list_os_versions_global`
   - `classic_list_page_views_per_weekday`
2. **Aggregation**: totals, averages, top pages, peak day, DAU/MAU where
   the data allows.
3. **Delta vs previous period** if requested.
4. **Markdown report**.

## Tools used

- `classic_list_page_views`, `classic_list_launches`,
  `classic_list_unique_launches`
- `classic_list_session_times`, `classic_list_downloads`,
  `classic_list_downloads_global`
- `classic_list_devices_global`, `classic_list_mobile_os_distribution`,
  `classic_list_os_versions_global`
- `classic_list_page_views_per_weekday`

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Traffic report — last 30 days (vs previous 30 d)

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

### 📱 Platforms
- iOS: 62% — Android: 38%
- Most common OS: iOS 17 (44%)

### 📥 Downloads
- New downloads in period: N (+X% vs prev.)
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **No per-page breakdown**: `classic_list_page_views` returns
  `{total_page_views, history}` — a global daily aggregate only. Do
  not invent a "most viewed pages" table; report totals and trends,
  not per-page numbers.
- If a metric is unavailable (API returns nothing), render it as "n/a"
  instead of inventing a number.
- Respect pagination on large analytics lists.
- Never compare windows of different sizes without warning.

## Next possible actions
- Run `membership-weekly-digest` to publish the numbers as part of the
  weekly recap.
- Run `membership-subscription-audit` if a session-time drop correlates
  with churn.
- Run `membership-push-broadcast` to re-engage on a traffic dip.
