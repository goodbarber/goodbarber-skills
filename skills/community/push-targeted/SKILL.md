---
name: community-push-targeted
description: |
  Prepare and send a targeted community push notification to specific
  users: user resolution, drafting, scheduling, platform targeting,
  filters, tap action, dry-run, send. The push is NEVER sent without
  explicit confirmation. Use by default whenever the user wants to notify
  named community users or an explicit user ID list. Prefer this skill
  over raw MCP tools when it reasonably fits. Skip only if the user
  explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent targeted pushes sent to the wrong community users or with poor
wording: a workflow with user resolution, preview, validation, send, and
receipt confirmation.

## Access contract

- `READ_WRITE`.

## API shape

`classic_v1_create_push_notification` accepts:

- `message` (required): notification body, max 255 characters.
- `targeting` (optional): `user_id` or `user_ids`. If omitted, the tool
  sends to all eligible recipients; use `community-push-broadcast` for
  app-wide or group broadcast intents.
- `platform` (optional): `all`, `ios`, `android`, `pwa`, or an array
  such as `["ios", "pwa"]`.
- `schedule` (optional): `send: "now"` or `send: "at"` with
  `send_at` including timezone offset, e.g. `2026-05-28T09:30+02:00`.
- `action` (optional): `open_app`, `external_link`, or `section`.
- `filters` (optional): available with platform targeting.

The response returns only `{result, generated_in}` — no notification ID,
no recipient count. Delivery is asynchronous to opted-in devices.

## Input contract

- Recipients (required for this skill): community user names, emails, or
  explicit user IDs.
- `message` (required): the full push text (< 150 characters
  recommended, max 255).
- `platform` (optional): default `all`; use one push call for multiple
  platforms.
- `schedule` (optional): immediate by default. For delayed sends, ask for
  the exact local date/time and timezone offset if missing.
- `action` (optional): what opens when the user taps the notification.
- `filters` (optional): `filters.native` for iOS/Android and
  `filters.pwa` for PWA. Supported filters are groups, period_launch,
  language, and native-only zones/no_push.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Recipient resolution**:
   - If the user provides names or emails, call `classic_v1_list_user`.
   - Fuzzy-match names/emails case-insensitively. Ask the user to choose
     from a numbered list if ambiguous. Never guess a user ID.
   - If no recipient can be resolved, stop and ask for a valid user ID,
     name, or email.
2. **Resolve tap action**:
   - Section action: call `cms_list_cms_sections` first with types
     `article`, `photo`, `video`, `sound`, `maps`, `agenda`; if no
     match is found, call `cms_list_sections` with the same types. Ask
     the user to pick if ambiguous.
   - External link: require a standard `https://...` URL.
3. **Draft**: compose the `message` and render it to the user in a
   "what the end user will see" format, including resolved recipients,
   platform, schedule, action, and filters.
4. **Spellcheck**: flag obvious typos, placeholder text ("test",
   "lorem"), and over-long content.
5. **Explicit confirmation**: "Confirm send?"
6. `classic_v1_create_push_notification` with `message`, `targeting`,
   optional `platform`, optional `schedule`, optional `action`, and
   optional `filters`.
7. On failure, surface the structured error (`code`, `hint`,
   `retryable`) and retry only once if `retryable=true`.
8. Confirm the send with a summary based on the tool response.

## Tools used

- `classic_v1_create_push_notification`
- `classic_v1_list_user` (recipient resolution)
- `cms_list_cms_sections`, `cms_list_sections` (optional, for section tap actions)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Targeted push — draft

💬 **Message**: "Your organizer badge is ready — open the app to view it"
📱 **Platform**: all
⏱️ **Send**: now
🔗 **Tap action**: open app
🎯 **Recipients**: Sam Lee (user_id: 817), Nora Bell (user_id: 8249)

→ Confirm send? (yes/no)

---

## Push sent ✅
- Accepted by API (result: ok, generated_in: 142 ms)
- Note: the API does not return a notification ID or delivery count;
  delivery is asynchronous to opted-in devices for the targeted users.
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
- For platform targeting, use one push call with `platform` as an array
  when multiple platforms are requested.
- PWA filters do not support `zones` or `no_push`.
- If the user wants a group broadcast, stop and hand off to
  `community-push-broadcast`.
- **`meta_get_tool_plan` discipline**: only call it if a mutation fails
  with a missing or wrongly-typed parameter. Do not call it preventively.

## Next possible actions
- Run `community-push-broadcast` if the message should go to a group or
  everyone.
- Run `community-traffic-report` in 24h to measure the push's impact on
  launches/sessions.
- Run `community-weekly-digest` to include this targeted outreach in the
  weekly recap.
