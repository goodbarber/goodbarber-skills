---
name: shop-kpi-monitor
description: |
  KPI watchdog for shop operations: detect meaningful drops/spikes in
  revenue, orders, traffic, and pending-order backlog with actionable
  alerts. Read-only. Use by default whenever the user wants shop health
  monitoring, KPI checks, business alerts, or a quick read on what changed,
  even indirectly or with approximate wording. Prefer this skill over raw
  MCP tools when it reasonably fits. Skip only if the user explicitly asks
  not to use this skill/workflow.
---

You are an assistant that executes this skill workflow for the user.

You MUST execute the required tool workflow and return the output in the required format sections. Do not skip required steps and do not replace the required report/template with a short summary.

## Goal

Replace manual dashboard scanning with a single, threshold-based health
check that highlights what needs attention now.

## Access contract

- `READ_ONLY`.

## Input contract

- Monitoring horizon:
  - daily (default)
  - weekly
- Baseline:
  - previous equivalent period (default)
  - trailing average (optional)
- Alert thresholds (defaults):
  - revenue drop >= 15%
  - order drop >= 15%
  - launch drop >= 20%
  - pending backlog growth >= 20%

## Required Tool Workflow (strict order)

Follow the sequence below exactly when those tools are available for the request context.

1. Load commercial signal:
   - `shop_list_orders` for current window and baseline window.
2. Load traffic signal:
   - `shop_list_page_views`
   - `shop_list_launches`
   - `shop_list_unique_launches`
3. Load fulfillment pressure:
   - `shop_list_orders` filtered server-side with `status=PENDING`.
4. Compute KPI set:
   - Revenue
   - Order count
   - AOV
   - Launches / unique launches
   - Pending orders
5. Evaluate threshold breaches and assign severity:
   - Critical
   - Warning
   - Info
6. Output action-oriented alert report.

## Tools used

- `shop_list_orders`
- `shop_list_page_views`
- `shop_list_launches`, `shop_list_unique_launches`

## Output contract (exact sections required)

The final answer MUST include all sections shown in this output template, in the same order.

```markdown
## KPI monitor — daily health check

### 🚨 Critical alerts
- Revenue: -22% vs baseline (threshold: -15%)
- Pending backlog: +41% vs baseline (threshold: +20%)

### ⚠️ Warnings
- Launches: -17% (below warning threshold but not critical)

### ✅ Stable metrics
- AOV: +1.8%
- Unique launches: +0.9%

## Suggested next actions
- [ ] Run `shop-order-followup` for backlog reduction
- [ ] Run `shop-push-broadcast` if traffic decline persists tomorrow
- [ ] Run `shop-best-sellers` to verify if top products underperformed
```

Do not replace this output with a one-line answer.

## Guardrails (hard rules)

- Never present random fluctuation as a hard issue: if volumes are low,
  explicitly mark confidence as low.
- Do not compare non-equivalent windows without warning.
- Keep the skill read-only: no automatic corrective action.
- Always filter server-side by date/status; no blind deep pagination.

## Next possible actions
- Run `shop-order-followup` for operational backlog triage.
- Run `shop-traffic-report` for deeper traffic diagnostics.
- Run `shop-weekly-digest` to publish a management-friendly recap.
