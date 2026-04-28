---
name: shop-rfm-segmentation
description: |
  Segment customers with an RFM model (Recency, Frequency, Monetary) and
  produce action-ready buckets for growth and retention. Read-only by
  default; execution is delegated to promo/push skills. Use by default
  whenever the user wants customer targeting, retention segments, scoring,
  or high-value/risk buckets, even indirectly or with approximate wording.
  Prefer this skill over raw MCP tools when it reasonably fits. Skip only
  if the user explicitly asks not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Provide a practical, explainable customer segmentation that directly maps to
marketing actions without spreadsheet work.

## Access contract

- `READ_ONLY` for analysis.
- `READ_WRITE` only if the user explicitly asks to execute follow-up actions
  in another skill.

## Input contract

- Analysis window (default: last 12 months)
- Scoring strategy:
  - quantiles (default) or fixed thresholds
- Optional exclusion: customers with no valid order in window

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. `shop_list_customers` (scope to relevant population when large).
2. `shop_list_orders` for the analysis window (server-side date filters).
3. Aggregate per customer:
   - Recency (days since last order)
   - Frequency (order count)
   - Monetary (total revenue)
4. Score each dimension (1-5) and build RFM score triplets.
5. Map score patterns to business segments:
   - Champions
   - Loyal
   - Potential loyalists
   - At risk
   - Hibernating
6. Produce action recommendations per segment, with delegated execution
   pointers.

## Tools used

- `shop_list_customers`, `shop_get_customer` (optional enrichment)
- `shop_list_orders`

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## RFM segmentation — last 12 months

### Segment overview
| Segment | Customers | Revenue share | Typical profile |
|---------|-----------|---------------|-----------------|
| Champions | 124 | 38% | Recent, frequent, high spend |
| Loyal | 402 | 34% | Frequent repeat buyers |
| Potential loyalists | 290 | 14% | New-ish repeat potential |
| At risk | 210 | 10% | Historically valuable, not recent |
| Hibernating | 980 | 4% | Old/low activity |

### Priority targets
- Segment: At risk
  - Size: N
  - Revenue at risk: $X
  - Suggested play: targeted comeback incentive

## Recommended actions
- [ ] Champions: reward-only campaign (no heavy discount)
- [ ] At risk: winback code with 7-day expiry
- [ ] Hibernating: low-cost reactivation push
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Do not export personal data (emails/phones) unless explicitly requested.
- If sample quality is weak (too few orders or very new shop), state the
  limitation before recommendations.
- Keep scoring explainable: show how bins/quantiles were built.
- For very large customer bases, avoid blind full pagination; narrow to
  active window and document exclusions.

## Next possible actions
- Run `shop-promo-campaign` to create a segment-specific offer.
- Run `shop-push-broadcast` to execute a reactivation message.
- Run `shop-customer-insights` for loyalty-focused follow-up.
