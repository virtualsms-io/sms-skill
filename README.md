# sms-skill

> VirtualSMS Agent Skill — real-SIM SMS verification across 145+ countries and 2,500+ services for Claude, Cursor, Codex, Windsurf, and any MCP-compatible AI agent.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![npm](https://img.shields.io/npm/v/virtualsms-mcp.svg)](https://www.npmjs.com/package/virtualsms-mcp)
[![Website](https://img.shields.io/badge/site-virtualsms.io-blue)](https://virtualsms.io)

<p align="center">
  <a href="https://virtualsms.io">
    <img src="https://virtualsms.io/logo-512.png" alt="VirtualSMS" width="128" height="128" />
  </a>
</p>

## What is this?

This repository is a public **Agent Skill** — an [Agent Skills](https://agentskills.io/specification)-compliant package that lets AI agents (Claude Code, Cursor, Codex, Windsurf, OpenClaw, Cline, Zed, and any MCP-aware client) automatically discover and use **VirtualSMS** for phone-number verification tasks.

VirtualSMS provides physical-SIM phone numbers for receiving SMS and OTP codes — passing carrier-lookup checks that flag VoIP/eSIM numbers as fraudulent — across 145+ countries and 2,500+ services.

## Quick install

### Claude Code / Claude Desktop

```bash
claude mcp add --scope user virtualsms -- npx -y virtualsms-mcp
export VIRTUALSMS_API_KEY=sk_live_...
```

Then drop a copy of [`SKILL.md`](SKILL.md) into your project's `.claude/skills/sms-skill/` directory, or install via your skill marketplace.

### Cursor / Windsurf / Codex / OpenClaw

See [`examples/mcp-install.md`](examples/mcp-install.md).

### Direct REST or x402 (no MCP)

```bash
curl -H "Authorization: Bearer $VIRTUALSMS_API_KEY" \
  https://virtualsms.io/api/v2/services
```

x402 endpoints accept pay-per-call USDC on Base mainnet — no account required. See [`examples/x402-payment.md`](examples/x402-payment.md).

## What's in this repo

| Path | Purpose |
|---|---|
| [`SKILL.md`](SKILL.md) | Canonical Agent Skill manifest (frontmatter + instructions) |
| [`examples/`](examples/) | End-to-end recipes for common verification tasks |
| [`docs/api-reference.md`](docs/api-reference.md) | REST + MCP endpoint reference |
| [`package.json`](package.json) | npm metadata for ecosystem discovery |
| [`LICENSE`](LICENSE) | MIT |

## Coverage

- **Countries:** 145+
- **Services:** 2,500+ (Telegram, Discord, WhatsApp, Google, Twitter/X, Tinder, Uber, banking, gig-economy, niche...)
- **Pricing:** from $0.05/code, real-time pricing returned by API. Never hardcode — call `find_cheapest` or `country_pricing`.
- **Transports:** MCP server, REST API, x402 (USDC on Base)

## Discovery on AI agent directories

This skill is published for indexing by:

- [Skills.sh](https://skills.sh) leaderboard
- [SkillsMP](https://skillsmp.com) marketplace
- [Claude Code Marketplaces](https://claudemarketplaces.com)
- [SkillsLLM](https://skillsllm.com)
- [AgentSkill.sh](https://agentskill.sh)
- [Agent Skill Index](https://agent-skill.co)
- [Skills Directory](https://skillsdirectory.com)
- [SkillsAuth](https://skillsauth.com)

If you're a directory maintainer and want this skill listed manually, open an issue and we'll provide whatever metadata your indexer needs.

## Related repos

- [`virtualsms-io/mcp-server`](https://github.com/virtualsms-io/mcp-server) — the MCP server source
- [`virtualsms-io/api-docs`](https://github.com/virtualsms-io/api-docs) — REST API documentation
- [`virtualsms-io/examples`](https://github.com/virtualsms-io/examples) — Python / Node.js / cURL recipes
- [`virtualsms-io/claude-skill-sms-verification`](https://github.com/virtualsms-io/claude-skill-sms-verification) — Claude-Desktop-specific skill drop-in
- [`virtualsms-io/cursor-rules-sms-verification`](https://github.com/virtualsms-io/cursor-rules-sms-verification) — Cursor `.mdc` rules
- [`virtualsms-io/codex-sms-verification`](https://github.com/virtualsms-io/codex-sms-verification) — Codex CLI config
- [`virtualsms-io/windsurf-workflow-sms`](https://github.com/virtualsms-io/windsurf-workflow-sms) — Windsurf `.windsurfrules`
- [`virtualsms-io/openclaw-skill-sms`](https://github.com/virtualsms-io/openclaw-skill-sms) — OpenClaw skill manifest

## Contributing

PRs welcome for additional examples, client configs, or documentation improvements. For SKILL.md format changes, follow the [Agent Skills specification](https://agentskills.io/specification).

## License

MIT © VirtualSMS — see [LICENSE](LICENSE).

## Links

- [virtualsms.io](https://virtualsms.io) — main site
- [virtualsms.io/api](https://virtualsms.io/api) — REST API docs
- [mcp.virtualsms.io](https://mcp.virtualsms.io) — hosted MCP endpoint
- [status.virtualsms.io](https://status.virtualsms.io) — uptime
- [virtualsms.io/contact](https://virtualsms.io/contact) — support
