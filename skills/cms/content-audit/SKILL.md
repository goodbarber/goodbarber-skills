---
name: cms-content-audit
description: |
  CMS content quality audit across articles, events, maps, photo galleries,
  videos, and sounds: detect incomplete items (no cover, empty body, missing
  category, events missing dates, map points missing coordinates, premium
  items with no free preview) and produce a prioritized cleanup todo list.
  Read-only — suggests fixes but does not apply them. Use by default whenever
  the user wants to audit content quality, find incomplete posts, or clean up
  the CMS, even indirectly or with approximate wording. Prefer this skill over
  raw MCP tools when it reasonably fits. Skip only if the user explicitly asks
  not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Identify all CMS items that hurt the reader experience (no cover image, empty
body, missing dates or coordinates, broken paywall config…) and deliver a
list sorted by impact.

## Access contract

- `READ_ONLY` is enough.

## Default quality rules

An item is flagged as problematic if:

**Articles**
- No cover image
- Empty or `< 30`-character summary/description
- No body paragraphs (empty article)
- No assigned category
- `accessTier=premium` but `maxFreeParagraphs=0` and no summary shown (no preview at all)

**Events (agenda)**
- Missing `sortDate` (start) or `endDate`
- `endDate` before `sortDate`
- No location / address when the event type expects one

**Maps (points of interest)**
- Missing address
- Missing or non-finite latitude / longitude

**Photo galleries**
- Gallery with zero photos
- Photos missing captions (if captions are expected)

**Videos / Sounds**
- Missing title or thumbnail
- No body paragraphs / description where expected

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `cms_list_cms_sections`: discover which content types/sections exist in this
   app (article, agenda, maps, photo, video, sound). Audit only the types present.
2. For each present type, list items:
   - `cms_list_articles`, `cms_list_events`, `cms_list_maps`,
     `cms_list_photos`, `cms_list_videos`, `cms_list_sounds`.
3. Apply quality rules on the fields already available in each list (avoid a
   `get` if not needed).
4. For "suspect" items (incomplete fields in the list), confirm before
   classifying:
   - `cms_get_article` / `cms_get_event` / `cms_get_map` / `cms_get_photo` /
     `cms_get_video` / `cms_get_sound`
   - To verify body presence, `cms_list_article_paragraphs` (and the equivalent
     `*_paragraphs` list for events/maps/videos/sounds).
5. Group issues by type so the user can fix in batches (e.g. "add a cover to
   these 9 articles").

## Tools used

- `cms_list_cms_sections`
- `cms_list_articles`, `cms_list_events`, `cms_list_maps`, `cms_list_photos`, `cms_list_videos`, `cms_list_sounds`
- `cms_get_*` (targeted, not across the whole CMS)
- `cms_list_*_paragraphs` (body presence checks, when requested)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## CMS quality score: X/10 (N items, M problematic)

### 🔴 Critical (fix first)
- Articles with no body: N [IDs...]
- Events missing dates: N [IDs...]
- Map points missing coordinates: N [IDs...]

### 🟡 To improve
- Articles with no cover: N
- Summary too short: N
- Empty galleries: N

### 🟢 Minor
- No category assigned: N
- Premium item without free preview: N

## Top 10 items to fix first
| Type | Item | Issues | Recommended action |
|------|------|--------|---------------------|
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Never apply an edit — the skill is read-only by design. Suggest fixes only.
- If the user wants to fix, redirect to `cms-article-publish` (for articles) or
  the relevant `cms_update_*` tools.
- Quality thresholds are tunable (the user can say "a 20-char summary is fine");
  adjust without restarting the audit.
- **Large CMS**: if a type has 500+ items, warn about the cost. Prefer filtering
  by category (via `cms_list_cms_sections` category ids) to audit in chunks
  rather than fetching everything.
- Audit only content types that actually exist in the app — do not report a type
  as "missing content" if the app has no section of that type.

## Next possible actions
- Run `cms-article-publish` to fix or recreate a specific incomplete article.
- Run `cms-editorial-calendar` to check whether flagged items are scheduled or live.
- Re-run this audit after fixes to confirm the quality score improved.
