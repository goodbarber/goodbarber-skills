---
name: membership-push-targeted
description: |
  Prepare and send a targeted membership push notification to specific
  users or subscription-status audiences: recipient resolution, drafting,
  scheduling, platform targeting, filters, tap action, dry-run, send. The
  push is NEVER sent without explicit confirmation. Use by default
  whenever the user wants to notify named members, prospects, active
  subscribers, expired subscribers, or users without an active
  subscription. Prefer this skill over raw MCP tools when it reasonably
  fits. Skip only if the user explicitly asks not to use this
  skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent membership pushes sent to the wrong users or audience status: a
workflow with user/status resolution, preview, validation, send, and
receipt confirmation.

## Access contract

- `READ_WRITE`.

## API shape

`classic_v3_create_push_notification` accepts:

- `message` (required): notification body, max 255 characters.
- `targeting` (optional): `user_id`/`user_ids` OR
  `subscription_status` (`ACTIVE`, `EXPIRED`, `NOT_ACTIVE`). If omitted,
  the tool sends to all eligible recipients; use
  `membership-push-broadcast` for that intent.
- `platform` (optional): `all`, `ios`, `android`, `pwa`, or an array
  such as `["ios", "pwa"]`.
- `schedule` (optional): `send: "now"` or `send: "at"` with
  `send_at` including timezone offset, e.g. `2026-05-28T09:30+02:00`.
- `action` (optional): `open_app`, `external_link`, or `section`.
- `filters` (optional): available only with platform targeting.

Platform targeting, user IDs, and subscription-status targeting are
exclusive targeting modes. Do not combine any two of them. Filters are
available only with platform targeting.

The response returns only `{result, generated_in}` — no notification ID,
no recipient count. Delivery is asynchronous to opted-in devices.

## Input contract

- Targeting (required for this skill):
  - Named users/emails or explicit user IDs; OR
  - subscription audience: active subscribers, expired subscribers, or
    users without an active subscription.
- `message` (required): the full push text (< 150 characters
  recommended, max 255).
- `platform` (optional): default `all`; use only when not targeting
  specific users or subscription status.
- `schedule` (optional): immediate by default. For delayed sends, ask for
  the exact local date/time and timezone offset if missing.
- `action` (optional): what opens when the user taps the notification.
- `filters` (optional): only with platform targeting. Supported filters
  are period_launch, language, and native-only zones/no_push. Group
  filters are not available on membership apps.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Targeting resolution**:
   - If the user asks for active subscribers, set
     `targeting.subscription_status` to `ACTIVE`.
   - If the user explicitly asks for expired subscribers, set
     `targeting.subscription_status` to `EXPIRED`.
   - If the user asks for users without a subscription or without an
     active subscription, set `targeting.subscription_status` to
     `NOT_ACTIVE`.
   - If the user provides names or emails, search
     `classic_list_prospects`, `classic_list_active_subscriptions`, and
     `classic_list_expired_subscriptions`; do not stop after only one
     list. Fuzzy-match case-insensitively and ask the user to choose if
     ambiguous. Never guess a user ID.
2. **Validate targeting mode**:
   - Do not combine `targeting.user_ids`, `targeting.subscription_status`,
     platform targeting, or filters.
   - If the user requests an invalid combination, explain the conflict
     and ask them to choose one targeting mode.
3. **Resolve tap action**:
   - Section action: call `cms_list_cms_sections` first with types
     `article`, `photo`, `video`, `sound`, `maps`, `agenda`; if no
     match is found, call `cms_list_sections` with the same types. Ask
     the user to pick if ambiguous.
   - External link: require a standard `https://...` URL.
4. **Draft**: compose the `message` and render it to the user in a
   "what the end user will see" format, including targeting, platform,
   schedule, action, and filters.
5. **Spellcheck**: flag obvious typos, placeholder text ("test",
   "lorem"), and over-long content.
6. **Explicit confirmation**: "Confirm send?"
7. `classic_v3_create_push_notification` with `message`, selected
   targeting mode, optional `schedule`, and optional `action`.
8. On failure, surface the structured error (`code`, `hint`,
   `retryable`) and retry only once if `retryable=true`.
9. Confirm the send with a summary based on the tool response.

## Tools used

- `classic_v3_create_push_notification`
- `classic_list_prospects`, `classic_list_active_subscriptions`,
  `classic_list_expired_subscriptions` (recipient resolution)
- `cms_list_cms_sections`, `cms_list_sections` (optional, for section tap actions)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Targeted push — draft

💬 **Message**: "Your subscription expires soon — renew today to keep access"
📱 **Platform**: not used with subscription-status targeting
⏱️ **Send**: 2026-05-28T09:30+02:00
🔗 **Tap action**: open app
🎯 **Target**: active subscribers (subscription_status: ACTIVE)

→ Confirm send? (yes/no)

---

## Push sent ✅
- Accepted by API (result: ok, generated_in: 142 ms)
- Note: the API does not return a notification ID or delivery count;
  delivery is asynchronous to opted-in devices in the targeted audience.
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **No automatic sends** without user validation.
- Reject empty `message` or placeholder content ("test", "lorem").
- Keep `message` <= 255 characters; recommend < 150 characters.
- If the send fails, do not loop — surface the error as-is.
- For delayed sends, `schedule.send_at` MUST include the MCP client
  user's timezone offset, e.g. `2026-05-28T09:30+02:00`. Do not send a
  naive datetime and do not pre-convert it to UTC.
- Platform targeting, user IDs, and subscription-status targeting are
  mutually exclusive.
- Filters are available only with platform targeting, not with user IDs
  or subscription-status targeting.
- Interpret unqualified "subscribers" as `ACTIVE`; only use `EXPIRED`
  when the user explicitly says expired subscribers.
- Group filters and group targeting are not available on membership
  apps.

## Next possible actions
- Run `membership-subscription-audit` to review churn and winback
  opportunities.
- Run `membership-push-broadcast` if the message should go to everyone.
- Run `membership-traffic-report` in 24h to measure the push's impact on
  launches/sessions.
