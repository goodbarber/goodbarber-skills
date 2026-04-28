---
name: membership-weekly-digest
description: |
  Weekly membership business digest ready to paste into Slack/Notion:
  active vs expired subscriptions, net subscriber delta, traffic, pushes
  sent, at-risk subscribers. Read-only. Use by default whenever the user
  wants a weekly recap, business summary, team update, or ready-to-share
  membership report, even indirectly or with approximate wording. Prefer
  this skill over raw MCP-tool handling when it reasonably fits. Skip
  only if the user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

A single membership report to publish every Monday morning, without
having to open the back-office.

## Access contract

- `READ_ONLY`.
- Default window: rolling last 7 days.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Subscriptions**:
   - `classic_list_active_subscriptions` +
     `classic_list_expired_subscriptions` → net subscriber delta
2. **Traffic**:
   - `classic_list_launches`, `classic_list_unique_launches`
   - `classic_list_session_times`
3. **At-risk** (delegate to `membership-subscription-audit` for details).
4. **Alerts**: at-risk subscriptions (expiration < 7 d), recent churn.
5. Final markdown report.

## Tools used

- `classic_list_active_subscriptions`,
  `classic_list_expired_subscriptions`
- `classic_list_launches`, `classic_list_unique_launches`
- `classic_list_session_times`
- Delegates to `membership-traffic-report`, `membership-subscription-audit`
  if the user wants to drill into a section.

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
# Membership weekly digest — week of 2026-04-14 to 2026-04-20

## 👥 Subscriptions
- Active: 1,204 (+18 net)
- New: 42 — Churn: 24
- At risk within 14 d: 31

## 📈 Traffic
- Launches: 18,200 (+7%)
- Avg session: 2m14

## ⚠️ Alerts
- 5 subscriptions expire within 3 d
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
- Run `membership-subscription-audit` to drill into the at-risk list.
- Run `membership-traffic-report` for the full analytics view.
- Run `membership-push-broadcast` if the digest surfaces an actionable
  churn signal.
