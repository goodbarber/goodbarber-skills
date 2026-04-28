---
name: community-weekly-digest
description: |
  Weekly community business digest ready to paste into Slack/Notion:
  traffic, launches, sessions, pushes sent (global + per group),
  engagement alerts. Read-only. Use by default whenever the user wants
  this outcome, even indirectly or with approximate wording: weekly
  digest, weekly report, recap, summary, Monday update, team update, or
  internal newsletter. Prefer this skill over raw MCP tools when it
  reasonably fits. Skip only if the user explicitly asks not to use this
  skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

A single community report to publish every Monday morning, without
having to open the back-office.

## Access contract

- `READ_ONLY`.
- Default window: rolling last 7 days.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Traffic**:
   - `classic_list_launches`, `classic_list_unique_launches`
   - `classic_list_session_times`
   - `classic_list_page_views`
2. **Engagement by weekday**:
   - `classic_list_page_views_per_weekday`
3. **Alerts**: launch drop vs W-1, session time drop.
4. Final markdown report.

## Tools used

- `classic_list_launches`, `classic_list_unique_launches`
- `classic_list_session_times`
- `classic_list_page_views`, `classic_list_page_views_per_weekday`
- Delegates to `community-traffic-report` if the user wants more detail.

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
# Community weekly digest — week of 2026-04-14 to 2026-04-20

## 📈 Traffic
- Launches: 18,200 (+7%)
- Unique launches: 4,210 (+5.6%)
- Avg session: 2m14

## 📅 Peak day
- Tuesday: 21% of the week's traffic

## ⚠️ Alerts
- Session time dropped -12% vs W-1 — worth investigating
```

> Note: the API does not expose a broadcast list, so a "pushes sent
> this week" section cannot be produced from the tool catalog alone.
> Omit it, or ask the user to track sends externally if they want it.

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Digest is 100% read-only. No side effects.
- If data is too sparse (launch week, not enough data), say so.
- **Large datasets**: filter server-side on the 7-day window for every
  list call.

## Next possible actions
- Run `community-traffic-report` for the full analytics drill-down.
- Run `community-push-broadcast` if the digest flags a significant
  engagement drop.
