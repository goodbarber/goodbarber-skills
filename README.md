# GoodBarber Skills

> 44 AI-powered skills to manage your GoodBarber app with any MCP-compatible client — eCommerce, Community, Membership, and CMS.

Listed on [Smithery](https://smithery.ai/servers/goodbarber/goodbarber-public-mcp).

These skills connect to the **GoodBarber MCP server** and give your AI assistant structured workflows to manage your products, orders, customers, promotions, subscriptions, content, analytics, and more — all from a conversational interface.

---

## What are Skills?

Skills are instruction files (`.md`) that teach an AI assistant **how** to perform specific tasks using your GoodBarber app's API via MCP (Model Context Protocol). Instead of manually calling API tools one by one, a skill orchestrates the full workflow — from data retrieval to formatted report — in a single natural-language request.

For example, asking *"Show me my best sellers this month"* triggers the `shop-best-sellers` skill, which automatically fetches your orders and catalog, computes rankings, and returns a formatted report.

---

## Quick Start

### Option A — Install as a Claude Code plugin

The repo is a Claude Code plugin: it ships the GoodBarber MCP server (`.mcp.json`) and the 44 skills in one package.

```bash
git clone https://github.com/goodbarber/goodbarber-skills.git
claude --plugin-dir ./goodbarber-skills
```

Once the plugin is listed in the community marketplace, install it from any session instead:

```
/plugin marketplace add anthropics/claude-plugins-community
/plugin install goodbarber@claude-community
```

On the first GoodBarber tool call, Claude Code opens the authorization page: paste your **Public API key** and validate. Details in [SETUP.md](SETUP.md).

### Option B — Manual setup (any MCP client)

#### 1. Connect the GoodBarber MCP

You need to add the GoodBarber MCP server as a **custom connector** in your client. The server implements the **MCP 2026-07-28 protocol** over Streamable HTTP (earlier handshake-era clients still connect). The `/mcp/sse` path name is historical and kept only for backward compatibility.

**MCP Server URLs:**

| Setup | URL |
|-------|-----|
| Single app | `https://mcp.goodbarber.dev/mcp/sse` |
| Multiple apps | `https://mcp.goodbarber.dev/<app_id>/mcp/sse` |

Replace `<app_id>` with your GoodBarber app ID if you manage multiple apps.

> **Authentication:** the GoodBarber MCP server uses a browser-based OAuth flow.
>
> When your MCP client connects to the MCP endpoint, it may open an authorization page on `mcp.goodbarber.dev`. On that page, you will be asked to paste your **GoodBarber Public API key** and validate the authorization. Generate your Public API key from your app's GoodBarber backoffice, on the **Public API / MCP server** page.
>
> **Important:**
> - Do **not** open the `/authorize` URL manually in your browser.
> - Start from the MCP client by connecting to the MCP endpoint.
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
claude mcp add goodbarber --transport http https://mcp.goodbarber.dev/mcp/sse

# Multiple apps
claude mcp add goodbarber --transport http https://mcp.goodbarber.dev/<app_id>/mcp/sse
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

Point your client to the MCP endpoint above (Streamable HTTP). The GoodBarber MCP server follows the standard MCP protocol and works with any compliant client.

#### 2. Install the skills

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
> **Note:** skill directory names carry their family prefix (`shop-`, `community-`, `membership-`, `cms-`) and are unique, so skills from several families can live in the same destination without collisions.

#### 3. Verify

Ask your AI assistant: *"What skills do you have?"* — it should list the installed GoodBarber skills.

---

## Skills by App Type

### eCommerce (18 skills)

<details>
<summary><strong>Catalog & Products</strong></summary>

| Skill | Description |
|-------|-------------|
| `shop-best-sellers` | Rank products by sales volume and revenue over a given period |
| `shop-catalog-audit` | Detect incomplete product sheets (missing images, descriptions, variants) |
| `shop-low-performers` | Identify products with zero or very low sales |
| `shop-orphan-products` | Find products not assigned to any collection |
| `shop-product-launch` | Guided creation of a full product with variants, images, and SEO |
| `shop-stock-check` | Audit stock levels — flag out-of-stock and low-stock items |
| `shop-reorder-planner` | Generate a supplier-ready replenishment queue based on sales velocity |

</details>

<details>
<summary><strong>Orders & Customers</strong></summary>

| Skill | Description |
|-------|-------------|
| `shop-order-followup` | To-do list of orders to process (pending, to ship, to deliver) |
| `shop-customer-insights` | Segment customers into VIP, loyal, dormant, and one-shot profiles |
| `shop-rfm-segmentation` | Recency / Frequency / Monetary customer segmentation |
| `shop-prospect-nurture` | Prioritize prospects and suggest conversion actions |

</details>

<details>
<summary><strong>Promotions</strong></summary>

| Skill | Description |
|-------|-------------|
| `shop-promo-campaign` | Create a promo code + send an announcement push in one workflow |
| `shop-promo-performance-review` | Analyze promo impact: before, during, and after the campaign |

</details>

<details>
<summary><strong>Analytics & Communication</strong></summary>

| Skill | Description |
|-------|-------------|
| `shop-traffic-report` | App-level analytics: page views, launches, sessions, platforms |
| `shop-kpi-monitor` | Threshold-based daily/weekly KPI health alerts |
| `shop-weekly-digest` | Automated weekly business recap |
| `shop-push-broadcast` | Compose, preview, schedule, and send a push notification to all customers |
| `shop-push-targeted` | Compose, preview, schedule, and send a push to specific customers or prospects |

</details>

---

### Community (5 skills)

| Skill | Description |
|-------|-------------|
| `community-traffic-report` | App analytics: page views, launches, sessions by platform |
| `community-push-broadcast` | Send a push to everyone or to specific community groups |
| `community-push-targeted` | Send a push to specific community users |
| `community-device-landscape` | Platform distribution, top devices, OS versions |
| `community-weekly-digest` | Weekly community activity recap |

---

### Membership (10 skills)

<details>
<summary><strong>Subscriptions</strong></summary>

| Skill | Description |
|-------|-------------|
| `membership-subscription-audit` | Active vs expired, churn rate, at-risk subscribers, winback opportunities |
| `membership-expiration-calendar` | Upcoming subscription expirations timeline |
| `membership-longest-subscribers` | Identify your most loyal long-term subscribers |
| `membership-internal-subscription-grant` | Create, update, or revoke internal subscriptions |

</details>

<details>
<summary><strong>Users & Analytics</strong></summary>

| Skill | Description |
|-------|-------------|
| `membership-prospect-followup` | Prioritize membership prospects for conversion |
| `membership-traffic-report` | App analytics: page views, launches, sessions by platform |
| `membership-push-broadcast` | Send a push notification to all eligible users |
| `membership-push-targeted` | Send a push to specific users or subscription-status audiences |
| `membership-device-landscape` | Platform distribution, top devices, OS versions |
| `membership-weekly-digest` | Weekly membership business recap |

</details>

---

### CMS / Content (11 skills)

Skills for managing editorial content — articles, agenda events, map points of interest, photo galleries, videos, and sounds. Work with any GoodBarber app that has CMS content sections.

<details>
<summary><strong>Authoring & Publishing</strong></summary>

| Skill | Description |
|-------|-------------|
| `cms-article-publish` | Guided creation of a full article: body paragraphs, category, slug, scheduling, paywall |
| `cms-event-publish` | Create an agenda event with start/end datetime, location, and body content |
| `cms-place-publish` | Create a map point of interest (address + coordinates) with description |
| `cms-gallery-builder` | Batch-upload images into a photo gallery and set titles/status |
| `cms-article-restructure` | Reorder, clean, and fix the body paragraphs of an existing article |

</details>

<details>
<summary><strong>Quality & Governance</strong></summary>

| Skill | Description |
|-------|-------------|
| `cms-content-audit` | Detect incomplete content across all types (no cover, empty body, missing dates/coords) |
| `cms-draft-review` | Surface stale drafts and unfinished content with a recommended next step |
| `cms-paywall-audit` | IAP apps: check premium content has a coherent free preview (accessTier, maxFreeParagraphs) |
| `cms-stale-content-refresh` | Rank aging articles as refresh, re-promote, or retire candidates |

</details>

<details>
<summary><strong>Planning & Recap</strong></summary>

| Skill | Description |
|-------|-------------|
| `cms-editorial-calendar` | Forward view: scheduled publications, upcoming events, expiring content |
| `cms-weekly-digest` | What published this week + what's scheduled next week, by content type |

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
├── .claude-plugin/plugin.json
├── .mcp.json
├── README.md
├── SETUP.md
├── server.json        # manifest published to the official MCP registry (dev.goodbarber/goodbarber-public-mcp)
└── skills/
    ├── ecommerce/
    │   ├── shop-best-sellers/SKILL.md
    │   ├── shop-catalog-audit/SKILL.md
    │   ├── shop-customer-insights/SKILL.md
    │   ├── shop-kpi-monitor/SKILL.md
    │   ├── shop-low-performers/SKILL.md
    │   ├── shop-order-followup/SKILL.md
    │   ├── shop-orphan-products/SKILL.md
    │   ├── shop-product-launch/SKILL.md
    │   ├── shop-promo-campaign/SKILL.md
    │   ├── shop-promo-performance-review/SKILL.md
    │   ├── shop-prospect-nurture/SKILL.md
    │   ├── shop-push-broadcast/SKILL.md
    │   ├── shop-push-targeted/SKILL.md
    │   ├── shop-reorder-planner/SKILL.md
    │   ├── shop-rfm-segmentation/SKILL.md
    │   ├── shop-stock-check/SKILL.md
    │   ├── shop-traffic-report/SKILL.md
    │   └── shop-weekly-digest/SKILL.md
    ├── community/
    │   ├── community-device-landscape/SKILL.md
    │   ├── community-push-broadcast/SKILL.md
    │   ├── community-push-targeted/SKILL.md
    │   ├── community-traffic-report/SKILL.md
    │   └── community-weekly-digest/SKILL.md
    ├── membership/
    │   ├── membership-device-landscape/SKILL.md
    │   ├── membership-expiration-calendar/SKILL.md
    │   ├── membership-internal-subscription-grant/SKILL.md
    │   ├── membership-longest-subscribers/SKILL.md
    │   ├── membership-prospect-followup/SKILL.md
    │   ├── membership-push-broadcast/SKILL.md
    │   ├── membership-push-targeted/SKILL.md
    │   ├── membership-subscription-audit/SKILL.md
    │   ├── membership-traffic-report/SKILL.md
    │   └── membership-weekly-digest/SKILL.md
    └── cms/
        ├── cms-article-publish/SKILL.md
        ├── cms-article-restructure/SKILL.md
        ├── cms-content-audit/SKILL.md
        ├── cms-draft-review/SKILL.md
        ├── cms-editorial-calendar/SKILL.md
        ├── cms-event-publish/SKILL.md
        ├── cms-gallery-builder/SKILL.md
        ├── cms-paywall-audit/SKILL.md
        ├── cms-place-publish/SKILL.md
        ├── cms-stale-content-refresh/SKILL.md
        └── cms-weekly-digest/SKILL.md
```

---

## Contributing

Want to add a skill? Create a folder in `skills/<app-type>/` with a `SKILL.md` file. Follow the existing format: YAML frontmatter (`name`, `description`, `compatibility`) + markdown instructions with tool workflow, input/output contracts, and guardrails.

## License

[The Unlicense](LICENSE) — public domain, no restrictions.
