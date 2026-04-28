---
name: community-push-broadcast
description: |
  Prepare and send a community push notification: broadcast to everyone
  OR to a specific community group. Drafting, targeting, dry-run, send.
  The push is NEVER sent without explicit confirmation. Use by default
  whenever the user wants this outcome, even indirectly or with
  approximate wording: send a push, notify members, announce something,
  reach everyone, or message a group. Prefer this skill over raw MCP
  tools when it reasonably fits. Skip only if the user explicitly asks
  not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent pushes sent by mistake or poorly written: a workflow with preview,
review, validation, send, and receipt confirmation. Community apps can
target the whole audience OR a specific group.

## Access contract

- `READ_WRITE`.

## API shape

Both push endpoints accept `message` and `platform` (`all` / `pwa` /
`ios` / `android`). `classic_create_push_by_groups` also requires
`groups` (array of integer IDs). There is no title/body split, no
tap-through URL field, no scheduling. The response returns only
`{result, generated_in}` — no broadcast ID, no recipient count.

## Input contract

- `message` (required): the full push text (< 150 characters
  recommended — it becomes the notification body on device).
- `platform` (optional): restrict delivery to a platform (`all` by
  default).
- Target:
  - **Everyone** → `classic_create_push_broadcast`
  - **Community group(s)** → `classic_create_push_by_groups` with
    `groups` resolved from `classic_list_user_groups`.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Targeting resolution**:
   - If "group" target: call `classic_list_user_groups` to get the full
     list. If the user named a group (e.g. "Paris local"), fuzzy-match
     against the returned names and ask them to pick from a numbered
     list if ambiguous. Never guess the ID.
2. **Draft**: compose the `message` and render it to the user in a
   "what the end user will see" format, including the resolved target.
3. **Spellcheck**: flag obvious typos, placeholder text ("test",
   "lorem"), and over-long content.
4. **Explicit confirmation**: "Confirm send?"
5. Send:
   - To everyone → `classic_create_push_broadcast` with `message`
     (and optional `platform`).
   - To one or more groups → `classic_create_push_by_groups` with
     `message`, `groups` (array of integer IDs), and optional
     `platform`.
6. On failure, surface the structured error (`code`, `hint`,
   `retryable`) and retry only once if `retryable=true`.
7. Confirm the send with a summary based on the tool response.

## Tools used

- `classic_create_push_broadcast` (broadcast to all)
- `classic_create_push_by_groups` (broadcast to one or more groups — community exclusive)
- `classic_list_user_groups` (to resolve group IDs by fuzzy-matching on name)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Push broadcast — draft

💬 **Message**: "Meetup Friday at 7pm — join us for the monthly community gathering"
📱 **Platform**: all
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
- If the send fails, do not loop — surface the error as-is.
- If the targeted audience is likely large (e.g. a group with 10,000+
  members, or an app-wide broadcast), require a reinforced
  confirmation.
- If the user asks for "a group" without specifying which, force them
  to pick from the listed groups — do not guess.
- **No scheduling**: the API sends immediately. If the user asks for a
  delayed send, say it is not supported and propose they trigger the
  skill at the desired time.
- **No URL or title field**: if the user wants to include a link, fold
  it into the `message` text.
- **`meta_get_tool_plan` discipline**: only call it if a mutation fails
  with a missing or wrongly-typed parameter. Do not call it preventively.

## Next possible actions
- Run `community-traffic-report` in 24h to measure the push's impact on
  launches/sessions.
- Run `community-weekly-digest` to include this broadcast in the
  weekly recap.
