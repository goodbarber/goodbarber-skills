# GoodBarber Skills

> 44 AI-powered skills to manage your GoodBarber app with any MCP-compatible client — eCommerce, Community, Membership, and CMS.

These skills connect to the **GoodBarber MCP server** and give your AI assistant structured workflows to manage your products, orders, customers, promotions, subscriptions, content, analytics, and more — all from a conversational interface.

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

> **Authentication:** the GoodBarber MCP server uses a browser-based OAuth flow.
>
> When your MCP client connects to the SSE endpoint, it may open an authorization page on `mcp.goodbarber.dev`. On that page, you will be asked to paste your **GoodBarber Public API key** and validate the authorization. Generate your Public API key from your app's GoodBarber backoffice, on the **Public API / MCP server** page.
>
> **Important:**
> - Do **not** open the `/authorize` URL manually in your browser.
> - Start from the MCP client by connecting to the SSE endpoint.
> - The client generates the full authorization request parameters automatically (`redirect_uri`, `client_id`, `state`, `code_challenge`, etc.).
>
> If you open the authorization page manually without those parameters, the authentication will fail.

**Claude Desktop (Cowork):**

1. Open **Settings > Connectors**
2. Click **Add custom connector**
3. Enter the MCP server URL above
4. The connector opens the GoodBarber authorization page — paste your **Public API key** there and validate

**Claude Code (CLI):**

```bash
# Single app
claude mcp add goodbarber --transport sse https://mcp.goodbarber.dev/mcp/sse

# Multiple apps
claude mcp add goodbarber --transport sse https://mcp.goodbarber.dev/<app_id>/mcp/sse
```

Then start a Claude Code session — on the first GoodBarber tool call, the CLI will open the GoodBarber authorization page in your browser. Paste your **Public API key** there and validate to complete the OAuth flow.

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

The first connection from your client triggers the OAuth flow in your browser — paste your **Public API key** on the GoodBarber authorization page and validate.

**Any other MCP-compatible client:**

Point your client to the SSE endpoint above. The GoodBarber MCP server follows the standard MCP protocol and works with any compliant client.

### 2. Install the skills

Clone this repo and copy only the skills for your app type:

```bash
git clone https://github.com/goodbarber/goodbarber-skills.git
```

> **Note:** the examples below use Claude's local skills directory (`~/.claude/skills/`). If you use another client, copy the skill folders to the directory expected by that client.

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

**CMS / Content (any app with content sections):**
```bash
cp -r goodbarber-skills/skills/cms/* ~/.claude/skills/
```

> **Tip:** CMS skills are cross-cutting — install them alongside any app type that has content sections (articles, agenda, maps, galleries, videos, sounds).
>
> **Tip:** If your GoodBarber app combines multiple types (e.g. eCommerce + Membership), install skills from both directories.
>
> **Warning:** some skill names are shared across app types (for example `weekly-digest`, `traffic-report`, `push-broadcast`, or `push-targeted`). If you install skills from multiple directories into the same destination, rename them or place them in separate namespaces to avoid collisions.

### 3. Verify

Ask your AI assistant: *"What skills do you have?"* — it should list the installed GoodBarber skills.

---

## Skills by App Type

### eCommerce (18 skills)

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
| `push-broadcast` | Compose, preview, schedule, and send a push notification to all customers |
| `push-targeted` | Compose, preview, schedule, and send a push to specific customers or prospects |

</details>

---

### Community (5 skills)

| Skill | Description |
|-------|-------------|
| `traffic-report` | App analytics: page views, launches, sessions by platform |
| `push-broadcast` | Send a push to everyone or to specific community groups |
| `push-targeted` | Send a push to specific community users |
| `device-landscape` | Platform distribution, top devices, OS versions |
| `weekly-digest` | Weekly community activity recap |

---

### Membership (10 skills)

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
| `push-broadcast` | Send a push notification to all eligible users |
| `push-targeted` | Send a push to specific users or subscription-status audiences |
| `device-landscape` | Platform distribution, top devices, OS versions |
| `weekly-digest` | Weekly membership business recap |

</details>

---

### CMS / Content (11 skills)

Skills for managing editorial content — articles, agenda events, map points of interest, photo galleries, videos, and sounds. Work with any GoodBarber app that has CMS content sections.

<details>
<summary><strong>Authoring & Publishing</strong></summary>

| Skill | Description |
|-------|-------------|
| `article-publish` | Guided creation of a full article: body paragraphs, category, slug, scheduling, paywall |
| `event-publish` | Create an agenda event with start/end datetime, location, and body content |
| `place-publish` | Create a map point of interest (address + coordinates) with description |
| `gallery-builder` | Batch-upload images into a photo gallery and set titles/status |
| `article-restructure` | Reorder, clean, and fix the body paragraphs of an existing article |

</details>

<details>
<summary><strong>Quality & Governance</strong></summary>

| Skill | Description |
|-------|-------------|
| `content-audit` | Detect incomplete content across all types (no cover, empty body, missing dates/coords) |
| `draft-review` | Surface stale drafts and unfinished content with a recommended next step |
| `paywall-audit` | IAP apps: check premium content has a coherent free preview (accessTier, maxFreeParagraphs) |
| `stale-content-refresh` | Rank aging articles as refresh, re-promote, or retire candidates |

</details>

<details>
<summary><strong>Planning & Recap</strong></summary>

| Skill | Description |
|-------|-------------|
| `editorial-calendar` | Forward view: scheduled publications, upcoming events, expiring content |
| `weekly-digest` | What published this week + what's scheduled next week, by content type |

</details>

> **Note:** `weekly-digest`, `traffic-report`, `push-broadcast`, and `push-targeted` names recur across app types. The CMS skills are namespaced by a `cms-` prefix in their frontmatter (e.g. `cms-weekly-digest`) to avoid collisions — keep them in a separate folder if you install skills from multiple directories.

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

"Send a push to Marie and Alex: your item is back in stock"
→ push-targeted (eCommerce)

"Notify expired subscribers about the renewal offer tomorrow at 9am"
→ push-targeted (membership)

"Give me my weekly digest"
→ weekly-digest

"Write and publish an article about our new opening hours"
→ article-publish (cms)

"What content is scheduled to go live next week?"
→ editorial-calendar (cms)

"Audit my content for missing covers and empty posts"
→ content-audit (cms)
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
    │   ├── push-targeted/SKILL.md
    │   └── weekly-digest/SKILL.md
    ├── community/
    │   ├── device-landscape/SKILL.md
    │   ├── push-broadcast/SKILL.md
    │   ├── push-targeted/SKILL.md
    │   ├── traffic-report/SKILL.md
    │   └── weekly-digest/SKILL.md
    ├── membership/
    │   ├── subscription-audit/SKILL.md
    │   ├── expiration-calendar/SKILL.md
    │   ├── ...
    │   ├── push-broadcast/SKILL.md
    │   ├── push-targeted/SKILL.md
    │   └── weekly-digest/SKILL.md
    └── cms/
        ├── article-publish/SKILL.md
        ├── content-audit/SKILL.md
        ├── editorial-calendar/SKILL.md
        ├── ...
        └── weekly-digest/SKILL.md
```

---

## Contributing

Want to add a skill? Create a folder in `skills/<app-type>/` with a `SKILL.md` file. Follow the existing format: YAML frontmatter (`name`, `description`, `compatibility`) + markdown instructions with tool workflow, input/output contracts, and guardrails.

## License

[The Unlicense](LICENSE) — public domain, no restrictions.
