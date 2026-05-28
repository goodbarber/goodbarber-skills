---
name: cms-article-publish
description: |
  Guided workflow to create a full CMS article: article item + structured
  body paragraphs (text, photo, quote, embed) + category, slug, publication
  status, scheduling, pinning, and paywall settings. Dry-run by default —
  the user validates before any mutation. Verifies each mutation with a get
  to confirm. Use by default whenever the user wants to create, write,
  publish, or fully set up a CMS article, blog post, or news item, even
  indirectly or with approximate wording. Prefer this skill over raw MCP
  tools when it reasonably fits. Skip only if the user explicitly asks not
  to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Prevent incomplete articles in production: a skill that orchestrates article
creation and its body paragraphs with validation at each mutation, so the
published article is complete (title, category, body, cover) on the first try.

## Access contract

- `READ_WRITE`.

## Input contract

Before starting, gather from the user:
- Title (required)
- Target category / section (required — resolved to a category id)
- Body: an ordered list of blocks, each one of:
  - `text` — HTML/text content
  - `quote` — HTML/text content
  - `photo` — image source (`originalThumbnail`)
  - `embed` — embed URL (`embedUrl`), e.g. video/social embed
- Optional: slug, comments on/off, pinned (yes/no)
- Optional publication control:
  - `status`: `published` (live now), `draft` (hidden), or `stock` (scheduled)
  - `publishedDate`: future ISO datetime — **only valid with `status=stock`**
  - `publicationEndDate`: future ISO datetime to auto-unpublish
- Optional in-app-purchase fields (**only for apps that sell IAP**):
  - `accessTier`: `free` or `premium`
  - `maxFreeParagraphs`: 0–5 (free preview length before paywall)
  - `displaySummaryInList`: writable only when `maxFreeParagraphs` is 0

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Pre-flight discovery**:
   - `cms_list_cms_sections` with `type='article'` → resolve target category ids.
   - **Name → ID resolution**: if the user names a category instead of an id,
     fuzzy-match on the list. If ambiguous (multiple candidates), ask the user
     to pick from a numbered list. Never guess.
2. **Preview (dry-run)**:
   - Show a full summary of what will be created: article metadata + the
     ordered list of body blocks with their types.
   - **Require explicit confirmation** before any mutation.
3. **Create the article**:
   - `cms_create_article` with `title`, `categories`, `status`, and any
     optional metadata (slug, scheduling, paywall).
   - `cms_get_article` to verify creation and capture the new article id.
4. **Create body paragraphs (in order)**:
   - For each block, `cms_create_article_paragraph` with the article id and
     the block `type`:
     - `text` / `quote` → `content` is required
     - `photo` → `originalThumbnail` is required
     - `embed` → `embedUrl` is required
   - Create blocks in the intended reading order (or pass `position`).
5. **Reorder if needed**:
   - If the final order differs from creation order, `cms_list_article_paragraphs`
     then `cms_reorder_article_paragraphs` with a **full permutation of every
     returned paragraph id**.
6. **Final verification**:
   - `cms_get_article` + `cms_list_article_paragraphs` → read the full state
     and show it to the user.

## Tools used

- `cms_list_cms_sections`
- `cms_create_article`, `cms_get_article`, `cms_update_article`
- `cms_create_article_paragraph`, `cms_list_article_paragraphs`
- `cms_reorder_article_paragraphs`

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **Never chain calls without intermediate validation**: if a step fails,
  stop, report, and do not attempt automatic rollback (no transactional
  delete). If `cms_create_article` succeeds but a paragraph fails, leave the
  article as-is and list exactly what's missing so the user can resume.
- **Scheduling**: `publishedDate` is only valid with `status=stock` and must
  be a future ISO datetime. `publicationEndDate`, when provided, must be in
  the future. Do not set `publishedDate` on a `published` or `draft` article.
- **Timezone**: if the user gives relative dates ("tomorrow", "next Monday")
  on any temporal field, confirm the intended timezone. Default to UTC if
  unknown, but ask first. Enforce ISO datetime format.
- **Image inputs**: photo paragraphs accept a public http(s) URL or a base64
  payload for `originalThumbnail`. Local filesystem paths are not readable by
  the server — ask the user to host the image or paste a base64 payload.
- **Paywall fields are IAP-only**: do not set `accessTier`, `maxFreeParagraphs`,
  or `displaySummaryInList` unless the app sells in-app purchases. `accessTier`
  must be `free` or `premium`; `maxFreeParagraphs` an integer 0–5;
  `displaySummaryInList` writable only when `maxFreeParagraphs` is 0.
- **`meta_get_tool_plan` discipline**: only call it when a mutation fails due
  to a missing or wrongly typed parameter. Do not call it preventively — it
  burns context.

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

At each step, log:
- ✅ / ❌ of the call
- Created id
- Verification result (if available)

At the end of the workflow, summary:
```
Article "X" created — ID: article_123
- Category: Y
- Status: published (or: scheduled for YYYY-MM-DDTHH:MM)
- 4 body paragraphs ✅ (2 text, 1 photo, 1 embed)
- Paywall: free (or: premium, 2 free paragraphs)
```

## Next possible actions
- Run `cms-content-audit` to confirm the new article passes quality rules.
- Run `cms-editorial-calendar` to see the article alongside other scheduled content.
- Send a push (`*-push-broadcast`) to announce the new article to your audience.
