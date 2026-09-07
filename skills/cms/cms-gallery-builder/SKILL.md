---
name: cms-gallery-builder
description: |
  Guided workflow to build a CMS photo gallery: batch-upload images (1–10 per
  batch) into a gallery section, then set each photo's title, description, and
  publication status. Dry-run by default — the user validates before any upload.
  Verifies with a list to confirm. Use by default whenever the user wants to
  create a gallery, add photos, upload images, or build a photo album, even
  indirectly or with approximate wording. Prefer this skill over raw MCP tools
  when it reasonably fits. Skip only if the user explicitly asks not to use
  this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Populate a photo gallery section with images and clean metadata in one guided
pass, batching uploads correctly and verifying the result.

## Access contract

- `READ_WRITE`.

## Input contract

Before starting, gather from the user:
- Target gallery section (required — resolved to a photo section id)
- An ordered list of images, each with:
  - `image` (required): public http(s) URL or base64 payload
  - Optional `title`
  - Optional `description` (set via update after upload)
- Optional `status`: `published` or `stock` (scheduled). **Note: photos do not
  support a `draft` status.**

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. **Pre-flight discovery**:
   - `cms_list_cms_sections` with `type='photo'` → resolve the target gallery
     section id.
   - **Name → ID resolution**: fuzzy-match a named gallery on the list; if
     ambiguous, ask the user to pick from a numbered list. Never guess.
2. **Preview (dry-run)**:
   - Show a full summary: gallery section, number of images, titles, status, and
     how many upload batches will run (10 images max per batch).
   - **Require explicit confirmation** before any upload.
3. **Batch-upload images**:
   - `cms_create_photos` with the `section` and a `photos` array (**1–10 entries
     per call**). Split larger sets into successive batches of ≤10.
4. **Set per-photo metadata** (if titles/descriptions were given but not in upload):
   - `cms_update_photo` per photo to set `title`, `description`, or `status`.
5. **Final verification**:
   - `cms_list_photos` for the same section → confirm the new photos appear and
     show the count.

## Tools used

- `cms_list_cms_sections`
- `cms_create_photos`, `cms_list_photos`, `cms_update_photo`

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- **Batch size**: `cms_create_photos` accepts **at most 10 images per call**.
  Never send more than 10 — split into batches and report each batch result.
- **Image inputs**: each entry needs a non-empty `image` as a public http(s)
  URL or base64 payload. Local filesystem paths are not readable by the server.
- **Status**: photos only support `published` or `stock` (scheduled). Do not pass
  `draft`.
- **No image replacement in place**: image bytes cannot be changed by update —
  to replace an image, delete (`cms_delete_photo`) then re-upload.
- **No gallery reordering tool**: there is no photo-reorder endpoint; upload in
  the desired order. Do not promise reordering after upload.
- **No rollback**: if a later batch fails, leave already-uploaded photos as-is
  and report exactly which batch failed.

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

At each step, log ✅ / ❌, the batch size, and the created photo ids.

Final summary:
```
Gallery "X" (section_123) updated
- Batch 1: 10 photos ✅
- Batch 2: 4 photos ✅
- Metadata set on 14 photos ✅
- Status: published
Total photos now in gallery: N
```

## Next possible actions
- Run `cms-content-audit` to flag photos still missing titles/captions.
- Run `cms-editorial-calendar` if photos were scheduled (`stock`) for later.
- Send a push (`*-push-broadcast`) to announce the new gallery.
