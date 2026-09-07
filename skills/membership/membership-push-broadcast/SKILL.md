---
name: membership-push-broadcast
description: |
  Prepare and send a membership push notification broadcast: drafting,
  scheduling, tap action, filters, dry-run, send. The push is NEVER sent
  without explicit confirmation. Membership broadcasts do not support
  group targeting. Use by default whenever the user wants to notify all
  subscribers or app users, announce something in the app, or send a
  push, even indirectly or with approximate wording. Prefer this skill
  over raw MCP-tool handling when it reasonably fits. Skip only if the
  user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent pushes sent by mistake or poorly written: a workflow with preview,
review, validation, send, and receipt confirmation. This skill is for
membership-wide broadcasts.

## Access contract

- `READ_WRITE`.

## API shape

`classic_create_push_broadcast` accepts `message`, optional `platform`,
optional `schedule`, optional `action`, and optional `filters`. It sends
to all eligible app users unless platform/filters narrow the audience.
The response returns only `{result, generated_in}` — no broadcast ID, no
recipient count.

For pushes to specific users or subscription-status audiences, use
`membership-push-targeted` instead.

## Input contract

- `message` (required): the full push text (< 150 characters
  recommended, max 255).
- `platform` (optional): `all`, `ios`, `android`, `pwa`, or an array
  such as `["ios", "pwa"]`. Use one push call for multiple platforms.
- `schedule` (optional): immediate by default. For delayed sends,
  require a timezone offset in `send_at`.
- `action` (optional): `open_app`, `external_link`, or `section`.
- `filters` (optional): `filters.native` for iOS/Android and
  `filters.pwa` for PWA. Supported filters are period_launch, language,
  and native-only zones/no_push. Group filters are not available on
  membership apps.
- Optional audience-estimate toggle: whether to call
  `classic_list_active_subscriptions` first to show approximate reach.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Resolve tap action**:
   - Section action: call `cms_list_cms_sections` first with types
     `article`, `photo`, `video`, `sound`, `maps`, `agenda`; if no
     match is found, call `cms_list_sections` with the same types. Ask
     the user to pick if ambiguous.
   - External link: require a standard `https://...` URL.
2. **Draft**: compose the `message` and render it to the user in a
   "what the end user will see" format.
3. **Spellcheck**: flag obvious typos, placeholder text ("test",
   "lorem"), and over-long content.
4. **Audience estimate** (optional): call
   `classic_list_active_subscriptions` to show approximate reach. This
   is an estimate of active subscribers, not the opted-in push audience
   — flag this to the user.
5. **Explicit confirmation**: "Confirm send?"
6. `classic_create_push_broadcast` with `message`, optional `platform`,
   optional `schedule`, optional `action`, and optional `filters`.
7. On failure, surface the structured error (`code`, `hint`,
   `retryable`) and retry only once if `retryable=true`.
8. Confirm the send with a summary based on the tool response.

## Tools used

- `classic_create_push_broadcast`
- `classic_list_active_subscriptions` (optional, for audience estimate)
- `cms_list_cms_sections`, `cms_list_sections` (optional, for section tap actions)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Push broadcast — draft

💬 **Message**: "Season 3 is out — 8 new episodes now in the app"
📱 **Platform**: all
⏱️ **Send**: now
🔗 **Tap action**: open app
🎯 **Target**: all eligible users (active-subs est.: 12,430)

→ Confirm send? (yes/no)

---

## Push sent ✅
- Accepted by API (result: ok, generated_in: 142 ms)
- Note: the API does not return a broadcast ID or a delivery count;
  delivery is asynchronous to opted-in devices (a subset of active
  subscribers).
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **No automatic sends** without user validation.
- Reject empty `message` or placeholder content ("test", "lorem").
- Keep `message` <= 255 characters; recommend < 150 characters.
- If the send fails, do not loop — surface the error as-is.
- If the active-sub estimate exceeds a threshold (e.g. 10,000), require
  a reinforced confirmation.
- For delayed sends, `schedule.send_at` MUST include the MCP client
  user's timezone offset, e.g. `2026-05-28T09:30+02:00`. Do not send a
  naive datetime and do not pre-convert it to UTC.
- For platform targeting, use one push call with `platform` as an array
  when multiple platforms are requested.
- PWA filters do not support `zones` or `no_push`.
- **No group targeting on membership apps**: if the user asks to target
  a group, say it is not supported.

## Next possible actions
- Run `membership-traffic-report` in 24h to measure the push's impact on
  launches/sessions.
- Run `membership-subscription-audit` if the push targeted at-risk subs —
  to see if churn reduced.
