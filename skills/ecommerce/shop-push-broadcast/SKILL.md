---
name: shop-push-broadcast
description: |
  Prepare and send a shop push notification broadcast: drafting,
  scheduling, tap action, dry-run, send. The push is NEVER sent without
  explicit confirmation. Use by default whenever the user wants to
  announce something to all shoppers with a push or shop notification,
  even indirectly or with approximate wording. Prefer this skill over
  raw MCP tools when it reasonably fits. Skip only if the user explicitly
  asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent pushes sent by mistake or poorly written: a workflow with preview,
review, validation, send, and receipt confirmation. This skill is for
shop-wide broadcasts to all eligible push recipients.

## Access contract

- `READ_WRITE`.

## API shape

`shop_create_push_broadcast` accepts:

- `message` (required): notification body, max 255 characters.
- `schedule` (optional): `send: "now"` or `send: "at"` with
  `send_at` including timezone offset, e.g. `2026-05-28T09:30+02:00`.
- `action` (optional): tap action. Default is `open_app`.
  Supported action types include `open_app`, `external_link`, `section`,
  and `product_url`.

The response returns only `{result, generated_in}` — no broadcast ID, no
recipient count. Delivery is asynchronous to opted-in devices.

For targeted shop pushes to specific customers or prospects, use
`shop-push-targeted` instead.

## Input contract

- `message` (required): the push text the user will read. Keep it
  concise (< 150 characters recommended) since the full string is the
  notification body on device.
- `schedule` (optional): immediate by default. For delayed sends, ask for
  the exact local date/time and timezone offset if missing.
- `action` (optional): what opens when the user taps the notification.
- Optional audience-estimate toggle: whether to call
  `shop_list_customers` first to show the user an approximate registered
  base before sending.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Resolve tap action**:
   - Product action: call `shop_list_products` when the product is named;
     fuzzy-match by title/slug and use `product_slug`. Never use
     `section_id`/`item_id` for products.
   - Section action: call `cms_list_cms_sections` first with types
     `article`, `photo`, `video`, `sound`, `maps`, `agenda`; if no
     match is found, call `cms_list_sections` with those types plus
     `commerce`. Ask the user to pick if ambiguous.
   - External link: require a standard `https://...` URL.
2. **Draft**: compose the `message` and render it to the user in a
   "what the end user will see" format.
3. **Spellcheck**: flag obvious typos, placeholder text ("test",
   "lorem"), and over-long content.
4. **Audience estimate** (optional): call `shop_list_customers` to show
   approximate reach. This is an estimate of the registered customer
   base, not the opted-in push audience — flag this to the user.
5. **Explicit confirmation**: "Confirm send?"
6. `shop_create_push_broadcast` with `message`, optional `schedule`, and
   optional `action`.
7. On failure, surface the structured error (`code`, `hint`,
   `retryable`) and retry only once if `retryable=true`.
8. Confirm the send with a summary based on the tool response.

## Tools used

- `shop_create_push_broadcast`
- `shop_list_customers` (optional, for audience estimate)
- `shop_list_products` (optional, for product tap actions)
- `cms_list_cms_sections`, `cms_list_sections` (optional, for section tap actions)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Push broadcast — draft

💬 **Message**: "Sale -20% this weekend — until Sunday 11:59 PM on the Summer collection"
⏱️ **Send**: now
🔗 **Tap action**: product "Summer tote" (slug: summer-tote)
🎯 **Target**: all opted-in shop app users (registered base est.: 8,320)

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
- Keep `message` <= 255 characters; recommend < 150 characters.
- If the send fails, do not loop — surface the error as-is.
- If the registered-customer estimate exceeds a threshold
  (e.g. 10,000), require a reinforced confirmation.
- For delayed sends, `schedule.send_at` MUST include the MCP client
  user's timezone offset, e.g. `2026-05-28T09:30+02:00`. Do not send a
  naive datetime and do not pre-convert it to UTC.
- Shop push broadcasts do not support platform targeting. If requested,
  say it is not available for shop push notifications.
- For `product_url`, resolve the product and pass `product_slug`.
- For section actions, never pass custom app deep links such as
  `section://`; provide `section_id` and let the tool resolve the public
  URL, or provide a known `https://...` URL/path.

## Next possible actions
- Run `shop-traffic-report` in 24h to measure the push's impact on
  launches/sessions.
- Run `shop-best-sellers` if the push targeted a specific product or
  collection.
- Run `shop-promo-campaign` if the send implies a discount that's not
  yet created.
