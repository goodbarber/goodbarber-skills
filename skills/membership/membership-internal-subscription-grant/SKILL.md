---
name: membership-internal-subscription-grant
description: |
  Safe workflow to create, modify, or revoke an internal membership
  subscription (granted by the team, off-Stripe / off-IAP). Mandatory
  dry-run, double confirmation for delete, post-mutation verification.
  Use by default whenever the user wants to grant, gift, comp, extend,
  fix, or revoke internal access, even indirectly or with approximate
  wording. Prefer this skill over raw MCP-tool handling when it
  reasonably fits. Skip only if the user explicitly asks not to use this
  skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Handle internal subscriptions safely: no double gift, no accidental delete,
mandatory explanatory note.

## Access contract

- `READ_WRITE`.

## Input contract

### For creation

- Target user email or ID
- Plan / duration
- `start_at` (format `%Y-%m-%dT%H:%M`)
- `end_at` (format `%Y-%m-%dT%H:%M`) or duration in months
- Note / reason ("QA test", "Compensation for incident on X",
  "Partner gift")

### For modification

- Internal subscription ID
- Field(s) to change
- New note (append, do not overwrite)

### For deletion

- Internal subscription ID
- Explicit reason

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

### Create

1. **User lookup**:
   - If the user gives a `user_id`, use it directly.
   - If the user gives an email, resolve to a user ID via:
     - `classic_list_prospects` (+ `classic_get_prospect` when needed) for
       users who never had a subscription.
     - `classic_list_expired_subscriptions` (+ `classic_get_expired_subscription`
       when needed) for users who had a subscription in the past and are now
       expired.
   - If multiple or no match, ask the user to confirm (numbered list, or
     paste the exact ID). Never guess.
2. Verify no active subscription already exists for this user via
   `classic_list_active_subscriptions` (filter by user_id/email). This
   step is a duplicate-prevention check, not the primary lookup source.
3. Dry-run: show the full payload.
4. User confirmation.
5. `classic_create_internal_subscription`.
6. `classic_get_internal_subscription` to verify.

### Update

1. `classic_get_internal_subscription` to read current state.
2. Dry-run the diff.
3. Confirm.
4. `classic_update_internal_subscription`.
5. Re-get to verify.

### Delete

1. `classic_get_internal_subscription` to show what will be deleted.
2. **Double confirmation**: ask the user to retype the ID.
3. `classic_delete_internal_subscription`.
4. `classic_list_active_subscriptions` to confirm absence.

## Tools used

- `classic_create_internal_subscription`
- `classic_get_internal_subscription`
- `classic_update_internal_subscription`
- `classic_delete_internal_subscription`
- `classic_list_prospects` / `classic_get_prospect` (prospect lookup)
- `classic_list_expired_subscriptions` / `classic_get_expired_subscription`
  (expired-user lookup)
- `classic_list_active_subscriptions` (pre-check)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Internal subscription creation

- User: pierre@example.com
- Plan: premium_monthly
- Period: 2026-04-20T00:00 → 2026-07-20T00:00
- Note: "Compensation for API incident on 2026-04-15"
- Created ID: is_9f3a ✅
- Verified in active list ✅
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **Double confirmation mandatory for delete** — no shortcut.
- **Reason required**: every create / update / delete must carry a
  descriptive reason, appended to the note (reject "test" or "n/a").
- **Date format**: `%Y-%m-%dT%H:%M` (local ISO).
- **Timezone**: if the user gives relative dates ("tomorrow", "in 3
  months", "end of the year"), confirm the intended timezone. Default
  to UTC if unknown, but ask first — a sub granted in the wrong TZ can
  start or expire a day off.
- On error, never retry more than once — report to the user.
- Eligible targets include:
  - prospects (never subscribed),
  - expired users (previously subscribed, now inactive).
  Always run active-subscriptions check before create to prevent duplicates.
- If an active subscription already exists, propose update instead of
  create.
- **`meta_get_tool_plan` discipline**: only call it if a mutation fails
  due to a missing or wrongly typed parameter. Do not call it
  preventively.

## Next possible actions

- Run `membership-subscription-audit` to verify the new sub appears in
  the active list and to spot any at-risk peers.
- Run `membership-push-broadcast` to notify the user their access is
  granted (only if the user opted in).
