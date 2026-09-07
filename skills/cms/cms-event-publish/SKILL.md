---
name: cms-event-publish
description: |
  Guided workflow to create a full CMS agenda event: event item (start/end
  datetime, category, status, scheduling, paywall) + structured body
  paragraphs (text, photo, quote, embed). Dry-run by default — the user
  validates before any mutation. Verifies each mutation with a get to confirm.
  Use by default whenever the user wants to create, add, schedule, or publish
  an event, agenda item, or calendar entry, even indirectly or with
  approximate wording. Prefer this skill over raw MCP tools when it reasonably
  fits. Skip only if the user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Create a complete, correctly-dated agenda event with its body content in one
guided pass, validating each mutation so the published event is never missing
its start/end time or description.

## Access contract

- `READ_WRITE`.

## Input contract

Before starting, gather from the user:
- Title (required)
- Target category / section (required — resolved to a category id)
- `sortDate`: event **start** instant — required, ISO datetime **with timezone offset**
- `endDate`: event **end** instant — required, must be after `sortDate`
- Optional `date`: editorial listing date (display metadata only)
- Optional `allDay` flag (when true, `endDate` may equal `sortDate`)
- Optional body blocks (ordered): `text`, `quote`, `photo` (`originalThumbnail`), `embed` (`embedUrl`)
- Optional publication control: `status` (`published` / `draft` / `stock`),
  `publishedDate` (future ISO, **only with `status=stock`**), `publicationEndDate`
- Optional `accessTier` (`free` / `premium`) — **IAP apps only**

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Pre-flight discovery**:
   - `cms_list_cms_sections` with `type='agenda'` → resolve target category ids.
   - **Name → ID resolution**: fuzzy-match a named category on the list; if
     ambiguous, ask the user to pick from a numbered list. Never guess.
2. **Preview (dry-run)**:
   - Show a full summary: title, category, start/end datetimes (with timezone),
     status, and the ordered body blocks.
   - **Require explicit confirmation** before any mutation.
3. **Create the event**:
   - `cms_create_event` with `title`, `categories`, `sortDate`, `endDate`,
     `status`, and any optional metadata.
   - `cms_get_event` to verify and capture the new event id.
4. **Create body paragraphs (in order)**:
   - For each block, `cms_create_event_paragraph` with the event id and `type`
     (`text`/`quote` need `content`; `photo` needs `originalThumbnail`; `embed`
     needs `embedUrl`).
5. **Reorder if needed**:
   - `cms_list_event_paragraphs` then `cms_reorder_event_paragraphs` with a
     **full permutation of every returned paragraph id**.
6. **Final verification**:
   - `cms_get_event` + `cms_list_event_paragraphs` → show the full state.

## Tools used

- `cms_list_cms_sections`
- `cms_create_event`, `cms_get_event`, `cms_update_event`
- `cms_create_event_paragraph`, `cms_list_event_paragraphs`
- `cms_reorder_event_paragraphs`

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **Dates are the most error-prone part**: `sortDate` and `endDate` must be ISO
  datetimes **with a timezone offset**; `endDate` must be after `sortDate`
  (or on/after it when `allDay` is true). Do not submit naive datetimes.
- **Timezone**: for relative dates ("Friday 7pm", "next week") confirm the
  intended timezone before building the offset. Never assume.
- **Scheduling**: `publishedDate` is valid only with `status=stock` and must be
  in the future. Do not confuse it with `sortDate` (the event time).
- **No rollback**: if the event is created but a paragraph fails, leave it as-is
  and list what's missing so the user can resume.
- **Image inputs**: photo paragraphs accept a public http(s) URL or base64 for
  `originalThumbnail`; local paths are not readable by the server.
- **Paywall is IAP-only**: only set `accessTier` (`free`/`premium`) when the app
  sells in-app purchases.
- **`meta_get_tool_plan` discipline**: only on a parameter-shape failure, never preventively.

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

At each step, log ✅ / ❌, the created id, and the verification result.

Final summary:
```
Event "X" created — ID: event_123
- Category: Y
- Start: YYYY-MM-DDTHH:MM±HH:MM · End: YYYY-MM-DDTHH:MM±HH:MM
- Status: published (or: scheduled for YYYY-MM-DDTHH:MM)
- 2 body paragraphs ✅
```

## Next possible actions
- Run `cms-editorial-calendar` to see the event alongside other upcoming items.
- Run `cms-content-audit` to confirm the event has no missing fields.
- Send a push (`*-push-broadcast`) to announce the event to your audience.
