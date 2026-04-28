---
name: membership-subscription-audit
description: |
  Full audit of membership subscriptions: active vs expired, churn
  detection, at-risk customers, winback opportunities. Updates
  subscription notes only on request. Use by default whenever the user
  wants subscription health, churn, retention risk, winback candidates,
  or an overall membership status review, even indirectly or with
  approximate wording. Prefer this skill over raw MCP-tool handling when
  it reasonably fits. Skip only if the user explicitly asks not to use
  this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

One report to know: how many active subscribers, recent churn rate, who
just left, and who is at risk.

## Access contract

- `READ_ONLY` for the report.
- `READ_WRITE` for note updates (line-by-line validation).

## Input contract

- `period_days`: churn analysis window (default 30)
- `at_risk_days`: days-before-expiration considered "at risk" (default 14)

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `classic_list_active_subscriptions`: current state. On large bases
   (>10k active subs), filter server-side (e.g. expiration window) and
   avoid deep full-pagination unless explicitly requested.
2. `classic_list_expired_subscriptions`: churn — filter on the period
   window, not the full history.
3. For each active subscription with near expiration:
   `classic_get_active_subscription` to read notes and details.
4. For recently expired subscriptions (in the period):
   `classic_get_expired_subscription`.
5. Compute:
   - Number of active subscribers
   - Net period churn (expired - renewals)
   - Churn rate %
   - "At risk" list (expiration < X days)

## Tools used

- `classic_list_active_subscriptions`, `classic_get_active_subscription`
- `classic_list_expired_subscriptions`, `classic_get_expired_subscription`
- `classic_update_active_subscription_note` (explicit action)
- `classic_update_expired_subscription_note` (explicit action)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Subscription state

### 📊 Snapshot
- Active: N
- Expired (period): M
- Net churn: -X
- Churn rate: Y%

### ⚠️ At risk (expires < 14 days)
| Subscriber | Plan | Expires | Notes | Suggested action |
|------------|------|---------|-------|-------------------|

### 💔 Recent churn
| Subscriber | Plan | Expired on | Lifetime | Winback possible? |

### ✅ Successful renewals (period)
N renewed subscriptions
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Note updates are unit and validated: never "apply to all".
- No internal subscription creation from this skill — that's the job of
  `membership-internal-subscription-grant`.
- If volumes are huge, sample and warn the user.
- **Large datasets**: prefer server-side filters (expiration window,
  plan) to reduce the fetch cost. Do not paginate the whole archive to
  compute a 30-day churn rate.

## Next possible actions
- Run `membership-push-broadcast` to send a winback push to the recent
  churn bucket.
- Run `membership-internal-subscription-grant` to gift a recovery sub
  to a high-value churn case.
- Run `membership-weekly-digest` to publish the summary numbers.
