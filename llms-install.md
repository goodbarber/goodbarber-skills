# Installing GoodBarber MCP with an AI agent

This file guides an AI coding agent (Cline, Claude Code, Cursor, Codex, Windsurf...) through the setup. Nothing runs locally: the GoodBarber MCP server is hosted at `https://mcp.goodbarber.dev/mcp/sse` (Streamable HTTP, MCP 2026-07-28 protocol; the path name is historical).

## 1. Add the remote server

**Cline** — add to `cline_mcp_settings.json` (or use MCP Servers > Remote server > Streamable HTTP):

```json
{
  "mcpServers": {
    "goodbarber": {
      "type": "streamableHttp",
      "url": "https://mcp.goodbarber.dev/mcp/sse",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

**Claude Code** — `claude mcp add goodbarber --transport http https://mcp.goodbarber.dev/mcp/sse`

**Cursor / VS Code / Windsurf** — add `{"mcpServers": {"goodbarber": {"url": "https://mcp.goodbarber.dev/mcp/sse"}}}` to the MCP configuration file.

**Codex CLI** — `codex mcp add goodbarber --url https://mcp.goodbarber.dev/mcp/sse` then `codex mcp login goodbarber`

**Gemini CLI** — `gemini mcp add --transport http goodbarber https://mcp.goodbarber.dev/mcp/sse` (or `gemini extensions install https://github.com/goodbarber/goodbarber-skills`)

## 2. Authorize

The server answers the first request with `401` and OAuth 2.1 metadata (dynamic client registration, PKCE). The client opens the GoodBarber authorization page in the browser: the user pastes the **Public API token** created in the app's back office (Public API / MCP server page) and validates. Do not put any key or token in the configuration file; there is no environment variable to set.

The tools exposed to the session depend on the rights granted to that token in the back office: a token limited to the shop only exposes the shop, orders, customers and analytics tools.

## 3. Verify

Call a read-only tool, for example `shop_list_collections` (shop apps) or `cms_list_sections` (content apps). Every tool declares `readOnlyHint` / `destructiveHint` annotations, so write tools prompt for confirmation.

## 4. Optional: install the 44 skills

The skills in this repository turn the tools into complete workflows (best sellers, promo campaigns, editorial calendar, subscription audit, push broadcasts). Install them with the skills CLI:

```bash
npx skills add goodbarber/goodbarber-skills -a cline
```

Use `--all` to install every skill for every supported agent.

## Several apps

Each token is tied to one app. Add one server per app with `https://mcp.goodbarber.dev/<app_id>/mcp/sse` and authorize each one with its own token.

## Troubleshooting

- **Authorization page loops or rejects the token**: create a new Public API token in the back office and paste it again.
- **Tools are missing**: the token's rights do not cover that family; edit the rights of the token in the back office.
- **Older client without OAuth support**: use a client that supports OAuth for remote MCP servers (Cline, Claude Code, Cursor, Codex, Windsurf, VS Code do).
