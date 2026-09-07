---
name: cms-paywall-audit
description: |
  In-app-purchase paywall consistency audit for CMS content: review accessTier
  (free/premium) and, for articles, maxFreeParagraphs and displaySummaryInList,
  flagging premium items with no free preview, invalid field combinations, and
  previews longer than the body. Read-only — recommends but does not mutate.
  Only meaningful for apps that sell in-app purchases. Use by default whenever
  the user wants to audit paywalls, premium content, free previews, or IAP
  gating, even indirectly or with approximate wording. Prefer this skill over
  raw MCP tools when it reasonably fits. Skip only if the user explicitly asks
  not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Ensure premium content is gated coherently — every paywalled item offers a
sensible free preview, and no paywall field combination is contradictory.

## Access contract

- `READ_ONLY`.

## Applicability check (run first)

- This skill only matters for apps that **sell in-app purchases**. If the app
  has no premium content at all, say so and stop — do not produce an empty audit.
- `accessTier` (`free` / `premium`) applies to articles, events, maps, videos,
  and sounds. `maxFreeParagraphs` and `displaySummaryInList` apply to
  **articles only**.

## Default paywall rules

An item is flagged if:
- `accessTier=premium` **and** `maxFreeParagraphs=0` **and** `displaySummaryInList`
  is off → **no preview at all** (reader sees nothing before paying)
- `displaySummaryInList` is on **while** `maxFreeParagraphs > 0` → invalid combo
  (`displaySummaryInList` is only writable when `maxFreeParagraphs` is 0)
- `maxFreeParagraphs` greater than the article's actual paragraph count → the
  "preview" exposes the entire body for free (paywall is ineffective)
- `accessTier=premium` on an item with an empty body → nothing to sell
- Inconsistent tiers within a category the user expects to be uniformly premium

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `cms_list_cms_sections`: discover content types/sections present.
2. List items per type and read `accessTier` from the list rows:
   - `cms_list_articles`, `cms_list_events`, `cms_list_maps`, `cms_list_videos`,
     `cms_list_sounds`.
3. For premium articles, confirm preview config:
   - `cms_get_article` for `maxFreeParagraphs` / `displaySummaryInList`
   - `cms_list_article_paragraphs` to compare `maxFreeParagraphs` against the
     real paragraph count.
4. Apply the paywall rules and group findings by severity.

## Tools used

- `cms_list_cms_sections`
- `cms_list_articles`, `cms_list_events`, `cms_list_maps`, `cms_list_videos`, `cms_list_sounds`
- `cms_get_article`, `cms_list_article_paragraphs` (targeted confirmation)

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## Paywall audit (N premium items, M flagged)

### 🔴 Broken paywalls (fix first)
- Premium with no preview: N [IDs...]
- Preview exposes full body: N [IDs...]

### 🟡 Invalid configuration
- displaySummaryInList set while maxFreeParagraphs > 0: N [IDs...]
- Premium item with empty body: N [IDs...]

### 🟢 Review
- Mixed tiers in a "premium" category: N

## Items to fix first
| Type | Item | Issue | Recommended fix |
|------|------|-------|------------------|
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Read-only skill: never change `accessTier`, `maxFreeParagraphs`, or
  `displaySummaryInList`. Recommend only.
- If the app sells no IAP, do not invent findings — report "no premium content".
- `maxFreeParagraphs` is an integer 0–5; treat values outside that as anomalies.
- On large catalogs, audit premium items by category in chunks rather than
  fetching everything.

## Next possible actions
- Use `cms_update_article` to fix preview length / tier on a flagged article.
- Run `cms-content-audit` for non-paywall quality issues on the same items.
- Run `cms-article-publish` to recreate a premium article with correct preview settings.
