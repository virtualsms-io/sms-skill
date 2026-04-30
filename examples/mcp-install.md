# Install snippets — VirtualSMS MCP across clients

The `virtualsms-mcp` npm package speaks the [Model Context Protocol](https://modelcontextprotocol.io). Drop the snippet for your client below.

> **Get an API key first:** sign up free at https://virtualsms.io and copy your `sk_live_...` key from the dashboard.

---

## Claude Code

```bash
claude mcp add --scope user virtualsms -- npx -y virtualsms-mcp
```

Set the API key as an env var (per-shell or in `~/.bashrc`):

```bash
export VIRTUALSMS_API_KEY=sk_live_...
```

Restart Claude Code. Tools appear as `mcp__virtualsms__*`.

---

## Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "virtualsms": {
      "command": "npx",
      "args": ["-y", "virtualsms-mcp"],
      "env": { "VIRTUALSMS_API_KEY": "sk_live_..." }
    }
  }
}
```

Restart Claude Desktop.

---

## Cursor

Add to `.cursor/mcp.json` (project-scoped) or `~/.cursor/mcp.json` (user-scoped):

```json
{
  "mcpServers": {
    "virtualsms": {
      "command": "npx",
      "args": ["-y", "virtualsms-mcp"],
      "env": { "VIRTUALSMS_API_KEY": "sk_live_..." }
    }
  }
}
```

Restart Cursor.

---

## Codex CLI

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.virtualsms]
command = "npx"
args = ["-y", "virtualsms-mcp"]

[mcp_servers.virtualsms.env]
VIRTUALSMS_API_KEY = "sk_live_..."
```

---

## Windsurf

Add to `~/.windsurfrules` or project `.windsurfrules`:

```
[mcp.virtualsms]
command = "npx -y virtualsms-mcp"
env.VIRTUALSMS_API_KEY = "sk_live_..."
```

---

## Cline / Continue / Zed

These follow the standard MCP `command + args + env` shape. Config file location varies by client; see the client's own docs and use the same JSON-shape as Claude Desktop.

---

## OpenClaw

```bash
openclaw mcp add virtualsms --command "npx -y virtualsms-mcp" --env VIRTUALSMS_API_KEY=sk_live_...
```

---

## Remote (no install)

If you can't run npx locally, point your MCP client at the hosted endpoint:

```
https://mcp.virtualsms.io/sse
```

Pass the API key as a header: `Authorization: Bearer sk_live_...`. Most modern MCP clients support remote SSE transport.

---

## Verify install

After restart, ask the agent:

> Use VirtualSMS to find the cheapest country for Telegram verification.

If the agent can answer with real pricing, you're set.
