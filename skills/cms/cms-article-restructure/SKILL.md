---
name: cms-article-restructure
description: |
  Reorganize the body of an existing CMS article: reorder paragraphs, remove
  empty or duplicate blocks, and fix the reading flow — without rewriting the
  article from scratch. Dry-run by default — the user approves the new layout
  before any mutation. Use by default whenever the user wants to reorder, move,
  clean up, or restructure an article's paragraphs/sections, even indirectly or
  with approximate wording. Prefer this skill over raw MCP tools when it
  reasonably fits. Skip only if the user explicitly asks not to use this
  skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Fix the structure of an article body safely: a proposed new order and cleanup,
approved by the user, then applied with the reorder/delete tools — never a
blind rewrite.

## Access contract

- `READ_WRITE`.

## Input contract

- Target article (required — resolved to an article id)
- The restructuring intent, e.g.:
  - reorder blocks into a new sequence
  - remove empty / placeholder / duplicate paragraphs
  - move a specific block up/down
- Optional: edit the text of a specific block

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Resolve the article**:
   - If only a title is given, `cms_list_articles` (title search) → resolve id;
     if ambiguous, ask the user to pick from a numbered list.
2. **Read current structure**:
   - `cms_list_article_paragraphs` → capture every paragraph id, type, and a
     short preview of its content.
3. **Propose the new layout (dry-run)**:
   - Present a before → after view: the new order, which blocks to delete, and
     any text edits.
   - **Require explicit confirmation** before any mutation.
4. **Apply edits** (only the blocks the user asked to change):
   - `cms_update_article_paragraph` per edited block.
5. **Apply deletions** (if any):
   - `cms_delete_article_paragraph` per block to remove. Re-list afterwards so
     the next step uses the current id set.
6. **Apply the new order**:
   - `cms_list_article_paragraphs` (refresh) then `cms_reorder_article_paragraphs`
     with a **full permutation of every remaining paragraph id**.
7. **Final verification**:
   - `cms_list_article_paragraphs` → show the final order to the user.

## Tools used

- `cms_list_articles`
- `cms_list_article_paragraphs`, `cms_get_article_paragraph`
- `cms_update_article_paragraph`, `cms_delete_article_paragraph`
- `cms_reorder_article_paragraphs`

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **Reorder requires a FULL permutation**: pass every remaining paragraph id
  exactly once. A partial list will corrupt the body order — always re-list
  immediately before reordering, especially after deletions.
- **Deletes need explicit per-block confirmation** and cannot be undone by the
  MCP tool. Never delete a non-empty block the user did not name.
- **Order of operations matters**: edit → delete → re-list → reorder. Do not
  reorder against a stale id set.
- **No rollback**: if a step fails mid-way, stop and report the current state
  (which edits/deletes applied) so the user can resume.
- This skill restructures an **existing** article. To create a new article, use
  `cms-article-publish`.

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Restructure plan — article "X" (article_123)

### Before
1. [text] ...
2. [photo] ...

### After
1. [photo] ...
2. [text] ...
- Delete: paragraph_882 (empty)
- Edit: paragraph_884 (text)
```

After confirmation, log ✅ / ❌ per operation, then a final order listing.

## Next possible actions
- Run `cms-content-audit` to confirm the article now reads cleanly.
- Run `cms-article-publish` to add brand-new blocks if the body needs more content.
- Send a push (`*-push-broadcast`) if the restructure was part of a re-launch.
