---
name: membership-longest-subscribers
description: |
  Build a membership leaderboard with two ranked tables:
  (1) currently active users by longest subscription duration,
  (2) all users (active or not) by longest subscription duration.
  Default output size is 10 rows per table and can be overridden. Use by
  default whenever the user wants the longest-tenured subscribers, top
  subscribers, a loyalty leaderboard, or duration-based rankings, even
  indirectly or with approximate wording. Prefer this skill over raw
  MCP-tool handling when it reasonably fits. Skip only if the user
  explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Return two action-ready ranking tables to identify:

- users with the longest subscription activity,
- users with the longest subscription activity even if they are no longer active.

## Access contract

- `READ_ONLY`.

## Input contract

- `top_n`: number of users to show in each table (default 10)
- Optional filters when explicitly requested by the user:
  - subscription type / plan name
  - language / locale
  - date window override (default behavior is full history)

Unless the user asks otherwise, use full subscription history.

## Ranking definitions (hard requirement)

Use these exact definitions:

1. **Committed subscription duration (per subscription line)**:
   - `subscription_end_at - subscription_start_at`
   - Include the whole committed period even if the subscription started recently.
   - Example: if a user subscribed today for a 1-year plan, count 1 full year.
2. **Total subscription duration (per user)**:
   - Sum committed subscription durations across all subscription lines.
3. **Table 1 ranking population**:
   - Only users currently active now.
   - Sort by total subscription duration (descending).
4. **Table 2 ranking population**:
   - All users, active and inactive.
   - Sort by total subscription duration (descending).
     If two users tie, use alphabetical order on email as tie-breaker.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. Load active subscriptions:
   - `classic_list_active_subscriptions`
2. Load expired subscriptions:
   - `classic_list_expired_subscriptions`
3. Build unified user-level aggregates from both datasets:
   - identity fields: email, first name, last name
   - latest/current subscription type name
   - total subscription duration
   - current active status
4. Produce the 2 rankings from the same aggregate:
   - active-only by duration
   - all-users by duration
5. Render the fixed-format report.

## Tools used

- `classic_list_active_subscriptions`
- `classic_list_expired_subscriptions`

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Membership longevity leaderboard

### 1) Active users — longest subscription duration

| Email | First name | Last name | Subscription type | Total subscribed duration |
| ----- | ---------- | --------- | ----------------- | ------------------------- |

### 2) All users — longest subscription duration

| Email | First name | Last name | Subscription type | Total subscribed duration | Active now |
| ----- | ---------- | --------- | ----------------- | ------------------------- | ---------- |
```

Each table must contain `top_n` rows by default (10 if not provided), unless there are fewer users available.

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Read-only skill: do not update any subscription or notes.
- Never compute duration using "time elapsed so far" for active subscriptions when commitment dates exist.
- Prefer committed duration from start/end dates so annual plans count as one full year immediately.
- Always include: email, first name, last name, and subscription type in all tables.
- If subscription type is missing, render `Unknown`.
- On large datasets, use server-side filtering/pagination safely and mention if results are partial.

## Next possible actions

- Run `membership-subscription-audit` to inspect churn/at-risk users among the top lists.
- Run `membership-push-broadcast` to send a targeted campaign to high-value subscribers.
- Run `membership-weekly-digest` to publish leaderboard highlights in your weekly summary.
