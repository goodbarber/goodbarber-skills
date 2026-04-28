# GoodBarber Skills

> 30 AI-powered skills to manage your GoodBarber app with any MCP-compatible client — eCommerce, Community, and Membership.

These skills connect to the **GoodBarber MCP server** and give your AI assistant structured workflows to manage your products, orders, customers, promotions, subscriptions, analytics, and more — all from a conversational interface.

---

## What are Skills?

Skills are instruction files (`.md`) that teach an AI assistant **how** to perform specific tasks using your GoodBarber app's API via MCP (Model Context Protocol). Instead of manually calling API tools one by one, a skill orchestrates the full workflow — from data retrieval to formatted report — in a single natural-language request.

For example, asking *"Show me my best sellers this month"* triggers the `best-sellers` skill, which automatically fetches your orders and catalog, computes rankings, and returns a formatted report.

---

## Quick Start

### 1. Connect the GoodBarber MCP

You need to add the GoodBarber MCP server as a **custom connector** in your client. The MCP server uses Server-Sent Events (SSE).

**MCP Server URLs:**

| Setup | URL |
|-------|-----|
| Single app | `https://mcp.goodbarber.dev/mcp/sse` |
| Multiple apps | `https://mcp.goodbarber.dev/<app_id>/mcp/sse` |

Replace `<app_id>` with your GoodBarber app ID if you manage multiple apps.

**Claude Desktop (Cowork):**

1. Open **Settings > Connectors**
2. Click **Add custom connector**
3. Enter the MCP server URL above
4. Authenticate with your GoodBarber credentials

**Claude Code (CLI):**

```bash
# Single app
claude mcp add goodbarber --transport sse https://mcp.goodbarber.dev/mcp/sse

# Multiple apps
claude mcp add goodbarber --transport sse https://mcp.goodbarber.dev/<app_id>/mcp/sse
```

**Cursor / VS Code / Windsurf:**

Add to your MCP configuration file (`.cursor/mcp.json`, `.vscode/mcp.json`, etc.):

```json
{
  "mcpServers": {
    "goodbarber": {
      "url": "https://mcp.goodbarber.dev/mcp/sse"
    }
  }
}
```

**Any other MCP-compatible client:**

Point your client to the SSE endpoint above. The GoodBarber MCP server follows the standard MCP protocol and works with any compliant client.

### 2. Install the skills

Clone this repo and copy only the skills for your app type:

```bash
git clone https://github.com/sintineddi/goodbarber-skills.git
```

**eCommerce app:**
```bash
cp -r goodbarber-skills/skills/ecommerce/* ~/.claude/skills/
```

**Community app:**
```bash
cp -r goodbarber-skills/skills/community/* ~/.claude/skills/
```

**Membership app:**
```bash
cp -r goodbarber-skills/skills/membership/* ~/.claude/skills/
```

> **Tip:** If your GoodBarber app combines multiple types (e.g. eCommerce + Membership), install skills from both directories.

### 3. Verify

Ask your AI assistant: *"What skills do you have?"* — it should list the installed GoodBarber skills.

---

## Skills by App Type

### eCommerce (17 skills)

<details>
<summary><strong>Catalog & Products</strong></summary>

| Skill | Description |
|-------|-------------|
| `best-sellers` | Rank products by sales volume and revenue over a given period |
| `catalog-audit` | Detect incomplete product sheets (missing images, descriptions, variants) |
| `low-performers` | Identify products with zero or very low sales |
| `orphan-products` | Find products not assigned to any collection |
| `product-launch` | Guided creation of a full product with variants, images, and SEO |
| `stock-check` | Audit stock levels — flag out-of-stock and low-stock items |
| `reorder-planner` | Generate a supplier-ready replenishment queue based on sales velocity |

</details>

<details>
<summary><strong>Orders & Customers</strong></summary>

| Skill | Description |
|-------|-------------|
| `order-followup` | To-do list of orders to process (pending, to ship, to deliver) |
| `customer-insights` | Segment customers into VIP, loyal, dormant, and one-shot profiles |
| `rfm-segmentation` | Recency / Frequency / Monetary customer segmentation |
| `prospect-nurture` | Prioritize prospects and suggest conversion actions |

</details>

<details>
<summary><strong>Promotions</strong></summary>

| Skill | Description |
|-------|-------------|
| `promo-campaign` | Create a promo code + send an announcement push in one workflow |
| `promo-performance-review` | Analyze promo impact: before, during, and after the campaign |

</details>

<details>
<summary><strong>Analytics & Communication</strong></summary>

| Skill | Description |
|-------|-------------|
| `traffic-report` | App-level analytics: page views, launches, sessions, platforms |
| `kpi-monitor` | Threshold-based daily/weekly KPI health alerts |
| `weekly-digest` | Automated weekly business recap |
| `push-broadcast` | Compose, preview, and send a push notification to all customers |

</details>

---

### Community (4 skills)

| Skill | Description |
|-------|-------------|
| `traffic-report` | App analytics: page views, launches, sessions by platform |
| `push-broadcast` | Send a push to everyone or to a specific community group |
| `device-landscape` | Platform distribution, top devices, OS versions |
| `weekly-digest` | Weekly community activity recap |

---

### Membership (9 skills)

<details>
<summary><strong>Subscriptions</strong></summary>

| Skill | Description |
|-------|-------------|
| `subscription-audit` | Active vs expired, churn rate, at-risk subscribers, winback opportunities |
| `expiration-calendar` | Upcoming subscription expirations timeline |
| `longest-subscribers` | Identify your most loyal long-term subscribers |
| `internal-subscription-grant` | Create, update, or revoke internal subscriptions |

</details>

<details>
<summary><strong>Users & Analytics</strong></summary>

| Skill | Description |
|-------|-------------|
| `prospect-followup` | Prioritize membership prospects for conversion |
| `traffic-report` | App analytics: page views, launches, sessions by platform |
| `push-broadcast` | Send a push notification to all subscribers |
| `device-landscape` | Platform distribution, top devices, OS versions |
| `weekly-digest` | Weekly membership business recap |

</details>

---

## Usage Examples

```
"Show me my best sellers this month"
→ best-sellers

"Any products running low on stock?"
→ stock-check

"Create a 20% promo code for summer and notify everyone"
→ promo-campaign

"How's my churn rate looking?"
→ subscription-audit

"Send a push to the Paris group: meetup Friday at 7pm"
→ push-broadcast (community)

"Give me my weekly digest"
→ weekly-digest
```

---

## Design Principles

All skills follow these rules:

- **Pagination-aware** — Large datasets are filtered server-side, not blindly paginated
- **Fuzzy matching** — Mention a product or collection by name; the skill resolves it
- **Workflow chaining** — Every skill suggests 2–3 related skills to run next
- **No destructive actions without confirmation** — Mutations always require explicit user validation
- **Structured output** — Every skill returns a formatted report, not a one-liner

---

## Compatibility

These skills work with any MCP-compatible AI client, including:

- Claude Desktop (Cowork)
- Claude Code (CLI)
- Cursor
- VS Code (Copilot, Claude, Cline, etc.)
- Windsurf
- Codex CLI
- Gemini CLI
- Any client supporting the Model Context Protocol

---

## Repo Structure

```
goodbarber-skills/
├── README.md
└── skills/
    ├── ecommerce/
    │   ├── best-sellers/SKILL.md
    │   ├── catalog-audit/SKILL.md
    │   ├── customer-insights/SKILL.md
    │   ├── ...
    │   └── weekly-digest/SKILL.md
    ├── community/
    │   ├── device-landscape/SKILL.md
    │   ├── push-broadcast/SKILL.md
    │   ├── traffic-report/SKILL.md
    │   └── weekly-digest/SKILL.md
    └── membership/
        ├── subscription-audit/SKILL.md
        ├── expiration-calendar/SKILL.md
        ├── ...
        └── weekly-digest/SKILL.md
```

---

## Contributing

Want to add a skill? Create a folder in `skills/<app-type>/` with a `SKILL.md` file. Follow the existing format: YAML frontmatter (`name`, `description`, `compatibility`) + markdown instructions with tool workflow, input/output contracts, and guardrails.

## License

MIT
