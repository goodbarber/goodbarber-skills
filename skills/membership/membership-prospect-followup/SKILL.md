---
name: membership-prospect-followup
description: |
  Prioritize membership prospects and update their sales notes after
  interaction. Read for analysis, write only note-by-note. Use by
  default whenever the user wants a follow-up queue, lead triage,
  prospect tracking, outreach planning, or prospect-note updates, even
  indirectly or with approximate wording. Prefer this skill over raw
  MCP-tool handling when it reasonably fits. Skip only if the user
  explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Give the sales team a prioritized call queue and an easy way to enrich
notes as interactions happen.

## Access contract

- `READ_ONLY` for analysis.
- `READ_WRITE` for `classic_update_prospect_note`.

## Prioritization rules

- **Hot**: signed up < 7 d, never contacted
- **Warm**: last note > 7 d ago and no follow-up
- **Follow-up due**: note exists with a reminder date now past
- **Cold**: > 30 d without interaction
- **Archive**: > 90 d without interaction

## API shape — what the tools return

- `classic_list_prospects` accepts **only** `page` (no server-side
  filtering by email, name, signup date, or status).
- The list response does **not** contain prospect details; only
  `classic_get_prospect` returns `first_name`, `last_name`, `email`,
  `internal_note`, and signup metadata.
- Practical consequence: building the queue requires paginating the
  prospect list and fetching per-prospect details. **Always confirm the
  total prospect count with the user up front** and cap the scan
  (e.g. last N pages) rather than iterating the whole base silently.

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

### "Produce the queue" mode
1. Ask the user for a scan budget: how many pages / prospects to cover.
2. `classic_list_prospects` page-by-page up to that budget.
3. For each prospect ID returned: `classic_get_prospect` to read notes
   and signup date (batch in parallel).
4. Bucket client-side into Hot / Warm / Follow-up / Cold / Archive.
5. Parse notes to detect dated reminders (user convention).
6. Render the sorted list.

### "Update a note" mode
1. Resolve the prospect ID.
   - If the user gives an ID, use it directly.
   - If the user gives a name or email, warn that resolution requires
     scanning (no server-side filter). Scan within the agreed budget and
     fuzzy-match on the returned `first_name` / `last_name` / `email`.
     Ask the user to pick from a numbered shortlist if ambiguous.
2. `classic_get_prospect` to show the current note.
3. Propose the amended note (timestamped append, don't overwrite history).
4. Confirm.
5. `classic_update_prospect_note`.
6. Re-get to verify.

## Tools used

- `classic_list_prospects`
- `classic_get_prospect`
- `classic_update_prospect_note`

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Prospects queue — sorted by priority

### 🔥 Hot (< 7 d, never contacted)
| Name | Email | Signed up | Source |

### 🔁 Follow-up due
| Name | Last note | Reminder date | Days overdue |

### 🌡️ Warm
...

### ❄️ Cold — re-engage via campaign
N prospects → candidates for "we haven't forgotten you" push
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Notes: timestamped append, format `[YYYY-MM-DD HH:MM] content`, never
  overwrite.
- No bulk edits — one note = one confirmation.
- GDPR: do not export emails without an explicit request.
- **Name → ID resolution**: server-side filtering is not available on
  `classic_list_prospects`. Fuzzy-match only within the pages scanned;
  warn the user when the match surface is partial and ask them to pick
  from a numbered shortlist if ambiguous.

## Next possible actions
- Run `membership-push-broadcast` to re-engage the Cold bucket.
- Run `membership-internal-subscription-grant` if a hot prospect
  converts (manual gift flow).
- Run `membership-subscription-audit` to see whether prospects who
  converted are still active.
