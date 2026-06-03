---
name: shop-push-targeted
description: |
  Prepare and send a targeted shop push notification to specific
  customers or prospects: recipient resolution, drafting, scheduling, tap
  action, dry-run, send. The push is NEVER sent without explicit
  confirmation. Use by default whenever the user wants to notify named
  shoppers, customers, prospects, or a small explicit recipient list.
  Prefer this skill over raw MCP tools when it reasonably fits. Skip only
  if the user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent targeted pushes sent to the wrong people or with poor wording: a
workflow with recipient resolution, preview, validation, send, and receipt
confirmation.

## Access contract

- `READ_WRITE`.

## API shape

`shop_create_push_notification` accepts:

- `message` (required): notification body, max 255 characters.
- `targeting` (optional): `user_id` or `user_ids`. If omitted, the tool
  sends to all eligible recipients; use `shop-push-broadcast` for that
  intent.
- `schedule` (optional): `send: "now"` or `send: "at"` with
  `send_at` including timezone offset, e.g. `2026-05-28T09:30+02:00`.
- `action` (optional): tap action. Default is `open_app`.
  Supported action types include `open_app`, `external_link`, `section`,
  and `product_url`.

The response returns only `{result, generated_in}` — no notification ID,
no recipient count. Delivery is asynchronous to opted-in devices.

## Input contract

- Recipients (required for this skill): customer/prospect names, emails,
  or explicit user IDs.
- `message` (required): the push text the user will read. Keep it
  concise (< 150 characters recommended, max 255).
- `schedule` (optional): immediate by default. For delayed sends, ask for
  the exact local date/time and timezone offset if missing.
- `action` (optional): what opens when the user taps the notification.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Recipient resolution**:
   - If the user provides names or emails, search both
     `shop_list_customers` and `shop_list_prospects`; do not stop after
     only one list.
   - Fuzzy-match names/emails case-insensitively. Ask the user to choose
     from a numbered list if ambiguous. Never guess a user ID.
   - If no recipient can be resolved, stop and ask for a valid user ID,
     name, or email.
2. **Resolve tap action**:
   - Product action: call `shop_list_products` when the product is named;
     fuzzy-match by title/slug and use `product_slug`. Never use
     `section_id`/`item_id` for products.
   - Section action: call `cms_list_cms_sections` first with types
     `article`, `photo`, `video`, `sound`, `maps`, `agenda`; if no
     match is found, call `cms_list_sections` with those types plus
     `commerce`. Ask the user to pick if ambiguous.
   - External link: require a standard `https://...` URL.
3. **Draft**: compose the `message` and render it to the user in a
   "what the end user will see" format, including resolved recipients.
4. **Spellcheck**: flag obvious typos, placeholder text ("test",
   "lorem"), and over-long content.
5. **Explicit confirmation**: "Confirm send?"
6. `shop_create_push_notification` with `message`, `targeting`, optional
   `schedule`, and optional `action`.
7. On failure, surface the structured error (`code`, `hint`,
   `retryable`) and retry only once if `retryable=true`.
8. Confirm the send with a summary based on the tool response.

## Tools used

- `shop_create_push_notification`
- `shop_list_customers`, `shop_list_prospects` (recipient resolution)
- `shop_list_products` (optional, for product tap actions)
- `cms_list_cms_sections`, `cms_list_sections` (optional, for section tap actions)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Targeted push — draft

💬 **Message**: "Your reserved item is back in stock — open the app to order"
⏱️ **Send**: now
🔗 **Tap action**: product "Summer tote" (slug: summer-tote)
🎯 **Recipients**: Marie Martin (user_id: 193763), Alex Rossi (user_id: 204118)

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
- Platform targeting is not available for shop push notifications.
- If the user intent is app-wide, stop and hand off to
  `shop-push-broadcast`.
- For named user targeting, search both customers and prospects before
  concluding the user cannot be found.
- For `product_url`, resolve the product and pass `product_slug`.

## Next possible actions
- Run `shop-customer-insights` to find broader segments for a future
  campaign.
- Run `shop-push-broadcast` if the message should go to all shoppers.
- Run `shop-promo-campaign` if this targeted message needs a discount.
