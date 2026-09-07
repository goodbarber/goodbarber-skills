---
name: cms-editorial-calendar
description: |
  Build a forward-looking CMS editorial calendar: scheduled publications
  (status=stock with a future publishedDate), upcoming agenda events, and
  content set to auto-unpublish (publicationEndDate), grouped into urgency
  buckets. Read-only. Use by default whenever the user wants to know what is
  scheduled to go live, which events are coming up, or what content expires
  soon, even indirectly or with approximate wording. Prefer this skill over
  raw MCP-tool handling when it reasonably fits. Skip only if the user
  explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Provide an operational calendar view of upcoming CMS activity — what publishes,
what happens, and what expires — so the user can plan editorial work ahead.

## Access contract

- `READ_ONLY`.

## Input contract

- `window_days`: how many days ahead to scan (default 30)
- `top_n`: maximum rows per table (default 10)
- Optional filters when explicitly requested:
  - content type (article / agenda / maps / video / sound)
  - category / section

If the user asks for "this week" or "this month", convert that to explicit date
boundaries and use those boundaries instead of `window_days`.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `cms_list_cms_sections`: discover which content types/sections exist. Scan
   only the types present (and any type/category the user restricted to).
2. **Scheduled publications** (going live in the window):
   - For each schedulable type, list with the scheduled status:
     `cms_list_articles`, `cms_list_events`, `cms_list_maps`,
     `cms_list_videos` filtered to `status=scheduled` (and `stock`).
   - Keep rows whose `publishedDate` falls within the window.
3. **Upcoming events** (happening in the window):
   - `cms_list_events` → keep rows whose `sortDate` (event start) falls within
     the window. This is independent of publication status.
4. **Expiring content** (auto-unpublishing in the window):
   - Across listed items, keep rows with a `publicationEndDate` within the window.
5. **Enrich and bucket** each retained row:
   - Title, type, category
   - Relevant date (publishedDate / sortDate / publicationEndDate)
   - Days remaining
   - Urgency bucket:
     - `Today` (0 days)
     - `Critical` (1–3 days)
     - `High` (4–7 days)
     - `Medium` (8–14 days)
     - `Low` (15+ days)
6. **Sort and render**:
   - Primary sort: date ascending
   - Secondary sort: title alphabetical

## Tools used

- `cms_list_cms_sections`
- `cms_list_articles`, `cms_list_events`, `cms_list_maps`, `cms_list_videos`
- `cms_get_*` (only to confirm a date when a list row is ambiguous)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## CMS editorial calendar

### Window
- Start: YYYY-MM-DD
- End: YYYY-MM-DD
- Content types scanned: ...
- Scheduled to publish: N · Upcoming events: N · Expiring: N

### 🚀 Scheduled publications
| Publish date | Days left | Urgency | Type | Title | Category |
| ------------ | --------- | ------- | ---- | ----- | -------- |

### 📅 Upcoming events
| Event date | Days left | Urgency | Title | Category |
| ---------- | --------- | ------- | ----- | -------- |

### ⏳ Expiring content (auto-unpublish)
| End date | Days left | Urgency | Type | Title |
| -------- | --------- | ------- | ---- | ----- |

### ⚠️ Priority breakdown
- Today: N · Critical (1-3d): N · High (4-7d): N · Medium (8-14d): N · Low (15+d): N
```

Display at most `top_n` rows per table (default 10) unless the user asks for more.

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Read-only skill: never publish, schedule, or update content.
- A `published` item with no future `publishedDate` is already live — do not list
  it under "Scheduled publications".
- Rows without the relevant date must not be treated as scheduled/upcoming/expiring.
- **Timezone**: report dates consistently; if the user gives a relative window,
  confirm the timezone, defaulting to UTC if unknown.
- On large datasets, paginate safely and state explicitly if output is partial.
- For short prompts like "what's coming up?" always render the full sections
  (`Window`, `Scheduled publications`, `Upcoming events`, `Expiring content`,
  `Priority breakdown`), not a plain list.

## Next possible actions
- Run `cms-article-publish` to schedule a new article into an open slot.
- Run `cms-content-audit` to make sure soon-to-publish items are complete.
- Send a push (`*-push-broadcast`) to announce a high-priority upcoming event.
