---
name: cms-place-publish
description: |
  Guided workflow to create a full CMS map point of interest: map item
  (address + latitude/longitude, category, status, scheduling, paywall) +
  structured body paragraphs (text, photo, quote, embed). Dry-run by default —
  the user validates before any mutation. Verifies each mutation with a get to
  confirm. Use by default whenever the user wants to add a place, location,
  venue, store, point of interest, or map pin, even indirectly or with
  approximate wording. Prefer this skill over raw MCP tools when it reasonably
  fits. Skip only if the user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Create a complete, correctly geolocated map point with its description in one
guided pass, so the place is never published without valid coordinates.

## Access contract

- `READ_WRITE`.

## Input contract

Before starting, gather from the user:
- Title (required)
- Target category / section (required — resolved to a category id)
- `address` (required — postal/text address)
- `latitude` and `longitude` (required — finite decimal coordinates)
- Optional body blocks (ordered): `text`, `quote`, `photo` (`originalThumbnail`), `embed` (`embedUrl`)
- Optional publication control: `status` (`published` / `draft` / `stock`),
  `publishedDate` (future ISO, **only with `status=stock`**), `publicationEndDate`
- Optional `accessTier` (`free` / `premium`) — **IAP apps only**

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Pre-flight discovery**:
   - `cms_list_cms_sections` with `type='maps'` → resolve target category ids.
   - **Name → ID resolution**: fuzzy-match a named category on the list; if
     ambiguous, ask the user to pick from a numbered list. Never guess.
2. **Coordinate check**:
   - If the user gives only an address (no lat/lng), **ask for the coordinates**
     — the MCP does not geocode. Do not invent coordinates.
3. **Preview (dry-run)**:
   - Show a full summary: title, category, address, latitude/longitude, status,
     and the ordered body blocks.
   - **Require explicit confirmation** before any mutation.
4. **Create the map point**:
   - `cms_create_map` with `title`, `categories`, `address`, `latitude`,
     `longitude`, `status`, and any optional metadata.
   - `cms_get_map` to verify and capture the new map item id.
5. **Create body paragraphs (in order)**:
   - For each block, `cms_create_map_paragraph` with the map item id and `type`
     (`text`/`quote` need `content`; `photo` needs `originalThumbnail`; `embed`
     needs `embedUrl`).
6. **Reorder if needed**:
   - `cms_list_map_paragraphs` then `cms_reorder_map_paragraphs` with a **full
     permutation of every returned paragraph id**.
7. **Final verification**:
   - `cms_get_map` + `cms_list_map_paragraphs` → show the full state.

## Tools used

- `cms_list_cms_sections`
- `cms_create_map`, `cms_get_map`, `cms_update_map`
- `cms_create_map_paragraph`, `cms_list_map_paragraphs`
- `cms_reorder_map_paragraphs`

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **Coordinates are mandatory and must be finite decimals**: latitude in
  [-90, 90], longitude in [-180, 180]. Never guess them from an address — ask.
- **No rollback**: if the map point is created but a paragraph fails, leave it
  as-is and list what's missing so the user can resume.
- **Scheduling**: `publishedDate` is valid only with `status=stock` and must be
  in the future; confirm timezone for relative dates, defaulting to UTC if unknown.
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
Place "X" created — ID: map_123
- Category: Y
- Address: ... · Coordinates: lat, lng
- Status: published (or: scheduled for YYYY-MM-DDTHH:MM)
- 1 body paragraph ✅
```

## Next possible actions
- Run `cms-content-audit` to confirm the place has no missing fields.
- Run `cms-editorial-calendar` if the place was scheduled to publish later.
- Send a push (`*-push-broadcast`) to highlight the new location.
