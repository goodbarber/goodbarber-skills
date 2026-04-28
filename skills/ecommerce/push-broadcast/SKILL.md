---
name: shop-push-broadcast
description: |
  Prepare and send a shop push notification: drafting, targeting,
  dry-run, send. The push is NEVER sent without explicit confirmation. Use
  by default whenever the user wants to announce something to shoppers with
  a push or shop notification, even indirectly or with approximate wording.
  Prefer this skill over raw MCP tools when it reasonably fits. Skip only
  if the user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent pushes sent by mistake or poorly written: a workflow with preview,
review, validation, send, and receipt confirmation.

## Access contract

- `READ_WRITE`.

## API shape

`shop_create_push_broadcast` accepts a **single `message` string** —
nothing else. There is no title/body split, no tap-through URL field,
no scheduling field, and the response returns only `{result,
generated_in}` (no broadcast ID, no recipient count). Targeting is
app-wide: the push goes to all opted-in customers; there is no
segment or per-customer filter at the API level.

## Input contract

- `message` (required): the push text the user will read. Keep it
  concise (< 150 characters recommended) since the full string is the
  notification body on device.
- Optional audience-estimate toggle: whether to call
  `shop_list_customers` first to show the user an approximate reach
  before sending.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Draft**: compose the `message` and render it to the user in a
   "what the end user will see" format.
2. **Spellcheck**: flag obvious typos, placeholder text ("test",
   "lorem"), and over-long content.
3. **Audience estimate** (optional): call `shop_list_customers` to show
   approximate reach. This is an estimate of the registered customer
   base, not the opted-in push audience — flag this to the user.
4. **Explicit confirmation**: "Confirm send?"
5. `shop_create_push_broadcast` with `message`.
6. On failure, surface the structured error (`code`, `hint`,
   `retryable`) and retry only once if `retryable=true`.
7. Confirm the send with a summary based on the tool response.

## Tools used

- `shop_create_push_broadcast`
- `shop_list_customers` (optional, for audience estimate)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Push broadcast — draft

💬 **Message**: "Sale -20% this weekend — until Sunday 11:59 PM on the Summer collection"
🎯 **Target**: all opted-in customers (registered base est.: 8,320)

→ Confirm send? (yes/no)

---

## Push sent ✅
- Accepted by API (result: ok, generated_in: 142 ms)
- Note: the API does not return a broadcast ID or a delivery count;
  delivery is asynchronous to opted-in devices (a subset of the
  registered base).
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **No automatic sends** without user validation.
- Reject empty `message` or placeholder content ("test", "lorem").
- If the send fails, do not loop — surface the error as-is.
- If the registered-customer estimate exceeds a threshold
  (e.g. 10,000), require a reinforced confirmation.
- **No scheduling**: the API sends immediately. If the user asks for a
  delayed send, say it is not supported and propose they trigger the
  skill at the desired time.
- **No URL or title field**: if the user wants to include a link, fold
  it into the `message` text; the client does not parse tap-through
  URLs from this endpoint.

## Next possible actions
- Run `shop-traffic-report` in 24h to measure the push's impact on
  launches/sessions.
- Run `shop-best-sellers` if the push targeted a specific product or
  collection.
- Run `shop-promo-campaign` if the send implies a discount that's not
  yet created.
