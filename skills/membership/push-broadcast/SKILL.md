---
name: membership-push-broadcast
description: |
  Prepare and send a membership push notification (broadcast to all):
  drafting, dry-run, send. The push is NEVER sent without explicit
  confirmation. Membership apps do not support group targeting. Use by
  default whenever the user wants to notify subscribers, announce
  something in the app, or send a push, even indirectly or with
  approximate wording. Prefer this skill over raw MCP-tool handling when
  it reasonably fits. Skip only if the user explicitly asks not to use
  this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent pushes sent by mistake or poorly written: a workflow with preview,
review, validation, send, and receipt confirmation.

## Access contract

- `READ_WRITE`.

## API shape

`classic_create_push_broadcast` accepts `message` (required) and
`platform` (optional: `all` / `pwa` / `ios` / `android`). There is no
title/body split, no tap-through URL field, no scheduling, and no
group targeting on membership apps. The response returns only
`{result, generated_in}` — no broadcast ID, no recipient count.

## Input contract

- `message` (required): the full push text (< 150 characters
  recommended — it becomes the notification body on device).
- `platform` (optional): restrict delivery to a platform (`all` by
  default).
- Optional audience-estimate toggle: whether to call
  `classic_list_active_subscriptions` first to show approximate reach.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Draft**: compose the `message` and render it to the user in a
   "what the end user will see" format.
2. **Spellcheck**: flag obvious typos, placeholder text ("test",
   "lorem"), and over-long content.
3. **Audience estimate** (optional): call
   `classic_list_active_subscriptions` to show approximate reach. This
   is an estimate of active subscribers, not the opted-in push
   audience — flag this to the user.
4. **Explicit confirmation**: "Confirm send?"
5. `classic_create_push_broadcast` with `message` (and optional
   `platform`).
6. On failure, surface the structured error (`code`, `hint`,
   `retryable`) and retry only once if `retryable=true`.
7. Confirm the send with a summary based on the tool response.

## Tools used

- `classic_create_push_broadcast`
- `classic_list_active_subscriptions` (optional, for audience estimate)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Push broadcast — draft

💬 **Message**: "Season 3 is out — 8 new episodes now in the app"
📱 **Platform**: all
🎯 **Target**: all opted-in subscribers (active-subs est.: 12,430)

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
- If the send fails, do not loop — surface the error as-is.
- If the active-sub estimate exceeds a threshold (e.g. 10,000), require
  a reinforced confirmation.
- **No scheduling**: the API sends immediately. If the user asks for a
  delayed send, say it is not supported and propose they trigger the
  skill at the desired time.
- **No URL or title field**: if the user wants to include a link, fold
  it into the `message` text.
- **No group targeting on membership apps**: if the user asks to
  target a group, say it is not supported.

## Next possible actions
- Run `membership-traffic-report` in 24h to measure the push's impact on
  launches/sessions.
- Run `membership-subscription-audit` if the push targeted at-risk subs —
  to see if churn reduced.
