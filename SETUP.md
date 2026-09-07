# Setup

The GoodBarber plugin bundles two things:

- the **GoodBarber MCP server** (`https://mcp.goodbarber.dev/mcp/sse`, Streamable HTTP, OAuth 2.1), declared in `.mcp.json`;
- **44 skills** (eCommerce, CMS, Community, Membership) that orchestrate the MCP tools into complete workflows.

## 1. Get your Public API key

1. Open your app's GoodBarber back office.
2. Go to the **Public API / MCP server** page.
3. Generate (or copy) your **Public API key**.

## 2. Authorize the MCP server

Nothing to configure by hand. On the first GoodBarber tool call, Claude Code opens the GoodBarber authorization page in your browser:

1. Paste your **Public API key**.
2. Validate.

The OAuth flow completes and the token is stored by Claude Code. Do **not** open `https://mcp.goodbarber.dev/authorize` manually: the client generates the required parameters (`redirect_uri`, `client_id`, `state`, `code_challenge`) and a manual visit fails.

## 3. Check the connection

Ask Claude: *"What GoodBarber tools do you have?"* or run `/mcp` and confirm `goodbarber` is connected. Then try a skill, for example: *"Show me my best sellers this month"*.

## Several apps on the same account

The bundled server URL binds one app per authorization (the app whose Public API key you paste). To manage several apps, add one server per app with the per-app URL and its own key:

```bash
claude mcp add goodbarber-<app_id> --transport http https://mcp.goodbarber.dev/<app_id>/mcp/sse
```

Replace `<app_id>` with the GoodBarber app ID shown in your back office.

## Which skills apply to my app

Skills are grouped by product family and load all together; each one states the app type it targets.

| Family | Skills | For |
|--------|--------|-----|
| `shop-*` | 18 | eCommerce apps |
| `cms-*` | 11 | any app with content sections (articles, agenda, maps, galleries, videos, sounds) |
| `membership-*` | 10 | Membership apps |
| `community-*` | 5 | Community apps |

A GoodBarber app combines at most two product families, so skills for the other families simply have no matching tools and stay unused.

## Troubleshooting

- **401 / "authorization required" loops**: revoke the connection with `/mcp` and reconnect; paste a freshly generated Public API key.
- **A skill says a tool is missing**: the skill targets another product family than your app (see the table above).
- **Multiple apps**: make sure each per-app server was authorized with that app's key.
