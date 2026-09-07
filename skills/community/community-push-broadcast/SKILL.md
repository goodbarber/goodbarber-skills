---
name: community-push-broadcast
description: |
  Prepare and send a community push notification broadcast: everyone or
  community group(s), with drafting, scheduling, tap action, filters,
  dry-run, and send. The push is NEVER sent without explicit
  confirmation. Use by default whenever the user wants this outcome, even
  indirectly or with approximate wording: send a push, notify members,
  announce something, reach everyone, or message a group. Prefer this
  skill over raw MCP tools when it reasonably fits. Skip only if the user
  explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent pushes sent by mistake or poorly written: a workflow with preview,
review, validation, send, and receipt confirmation. Community broadcasts
can target the whole audience OR one or more community groups.

## Access contract

- `READ_WRITE`.

## API shape

Use:

- `classic_create_push_broadcast` for all eligible users.
- `classic_create_push_by_groups` for community group targeting.

Both endpoints accept `message`, optional `platform`, optional
`schedule`, optional `action`, and optional `filters`.
`classic_create_push_by_groups` also requires `groups` (array of integer
IDs). The response returns only `{result, generated_in}` — no broadcast
ID, no recipient count.

For targeted pushes to specific community users, use
`community-push-targeted` instead.

## Input contract

- `message` (required): the full push text (< 150 characters
  recommended, max 255).
- `platform` (optional): `all`, `ios`, `android`, `pwa`, or an array
  such as `["ios", "pwa"]`. Use one push call for multiple platforms.
- `schedule` (optional): immediate by default. For delayed sends,
  require a timezone offset in `send_at`.
- `action` (optional): `open_app`, `external_link`, or `section`.
- `filters` (optional): `filters.native` for iOS/Android and
  `filters.pwa` for PWA. Supported filters are groups, period_launch,
  language, and native-only zones/no_push.
- Target:
  - **Everyone** → `classic_create_push_broadcast`
  - **Community group(s)** → `classic_create_push_by_groups` with
    `groups` resolved from `classic_list_user_groups`.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Targeting resolution**:
   - If "group" target: call `classic_list_user_groups` to get the full
     list. If the user named a group, fuzzy-match against returned names
     and ask them to pick from a numbered list if ambiguous. Never guess
     the ID.
2. **Resolve tap action**:
   - Section action: call `cms_list_cms_sections` first with types
     `article`, `photo`, `video`, `sound`, `maps`, `agenda`; if no
     match is found, call `cms_list_sections` with the same types. Ask
     the user to pick if ambiguous.
   - External link: require a standard `https://...` URL.
3. **Draft**: compose the `message` and render it to the user in a
   "what the end user will see" format, including platform, schedule,
   action, filters, and resolved target.
4. **Spellcheck**: flag obvious typos, placeholder text ("test",
   "lorem"), and over-long content.
5. **Explicit confirmation**: "Confirm send?"
6. Send:
   - To everyone → `classic_create_push_broadcast`
   - To one or more groups → `classic_create_push_by_groups`
7. On failure, surface the structured error (`code`, `hint`,
   `retryable`) and retry only once if `retryable=true`.
8. Confirm the send with a summary based on the tool response.

## Tools used

- `classic_create_push_broadcast` (broadcast to all)
- `classic_create_push_by_groups` (broadcast to one or more groups)
- `classic_list_user_groups` (to resolve group IDs by fuzzy-matching on name)
- `cms_list_cms_sections`, `cms_list_sections` (optional, for section tap actions)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Push broadcast — draft

💬 **Message**: "Meetup Friday at 7pm — join us for the monthly community gathering"
📱 **Platform**: ios + pwa
⏱️ **Send**: 2026-05-28T19:00+02:00
🔗 **Tap action**: open app
🎯 **Target**: group "Paris local" (id: 42)

→ Confirm send? (yes/no)

---

## Push sent ✅
- Accepted by API (result: ok, generated_in: 142 ms)
- Note: the API does not return a broadcast ID or a delivery count;
  delivery is asynchronous to opted-in devices in the targeted
  audience.
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **No automatic sends** without user validation.
- Reject empty `message` or placeholder content ("test", "lorem").
- Keep `message` <= 255 characters; recommend < 150 characters.
- If the send fails, do not loop — surface the error as-is.
- If the targeted audience is likely large (e.g. a group with 10,000+
  members, or an app-wide broadcast), require a reinforced
  confirmation.
- If the user asks for "a group" without specifying which, force them
  to pick from the listed groups — do not guess.
- For delayed sends, `schedule.send_at` MUST include the MCP client
  user's timezone offset, e.g. `2026-05-28T09:30+02:00`. Do not send a
  naive datetime and do not pre-convert it to UTC.
- For platform targeting, use one push call with `platform` as an array
  when multiple platforms are requested.
- PWA filters do not support `zones` or `no_push`. Group filters are
  available only for Classic v1/community tools.
- **`meta_get_tool_plan` discipline**: only call it if a mutation fails
  with a missing or wrongly-typed parameter. Do not call it preventively.

## Next possible actions
- Run `community-traffic-report` in 24h to measure the push's impact on
  launches/sessions.
- Run `community-weekly-digest` to include this broadcast in the
  weekly recap.
