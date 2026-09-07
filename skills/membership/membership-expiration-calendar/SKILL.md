---
name: membership-expiration-calendar
description: |
  Build a day-by-day membership expiration calendar for the upcoming
  period (default 30 days), with urgency buckets to prioritize retention
  actions. Read-only. Use by default whenever the user wants to know who
  expires soon, what renewals are coming up, or which subscribers need
  retention attention next, even indirectly or with approximate wording.
  Prefer this skill over raw MCP-tool handling when it reasonably fits.
  Skip only if the user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Provide an operational calendar view of upcoming subscription expirations so
the user can act before churn happens.

## Access contract

- `READ_ONLY`.

## Input contract

- `window_days`: how many days ahead to scan (default 30)
- `top_n`: maximum rows to display (default 10)
- Optional filters when explicitly requested:
  - subscription type / plan name
  - language / locale

If the user asks for "this week" or "this month", convert that to explicit
date boundaries and use those boundaries instead of `window_days`.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `classic_list_active_subscriptions`
   - Load active subscriptions only.
2. Filter to expiring subscriptions:
   - Keep rows where `expiration_date` exists and is within the selected window.
   - Exclude rows with no `expiration_date` (for example unlimited/internal plans).
3. Enrich each retained row:
   - Email, first name, last name
   - Subscription type name
   - Expiration date (UTC date)
   - Days remaining
   - Urgency bucket:
     - `Today` (0 days)
     - `Critical` (1-3 days)
     - `High` (4-7 days)
     - `Medium` (8-14 days)
     - `Low` (15+ days)
4. Sort and render:
   - Primary sort: expiration date ascending
   - Secondary sort: email alphabetical
5. Render fixed-format report.

## Tools used

- `classic_list_active_subscriptions`

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Membership expiration calendar

### Window

- Start: YYYY-MM-DD
- End: YYYY-MM-DD
- Active subscriptions scanned: N
- Expiring in window: N

### 📅 Upcoming expirations

| Expiration date | Days left | Urgency | Email | First name | Last name | Subscription type |
| --------------- | --------- | ------- | ----- | ---------- | --------- | ----------------- |

### ⚠️ Priority breakdown

- Today: N
- Critical (1-3d): N
- High (4-7d): N
- Medium (8-14d): N
- Low (15+d): N
```

Display at most `top_n` rows (default 10) in the table unless the user asks for more.

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Read-only skill: never update subscriptions or notes.
- Do not include expired subscriptions in the upcoming expiration table.
- Rows without `expiration_date` must not be treated as expiring.
- Always include email, first name, last name, and subscription type columns.
- If user identity fields are missing, render `Unknown`.
- On large datasets, paginate active subscriptions safely and state if output is partial.
- For short prompts like "who expires next?" or "next to expire", do not return
  a plain list: always render the full sections (`Window`, `Upcoming
expirations`, `Priority breakdown`).

## Next possible actions

- Run `membership-subscription-audit` to review churn and at-risk context.
- Run `membership-push-broadcast` to target users in Critical/High buckets.
- Run `membership-weekly-digest` to include expiration pressure in the weekly recap.
