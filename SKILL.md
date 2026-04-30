---
name: sms-skill
description: Receive SMS verification codes via real-SIM phone numbers across 145+ countries and 2,500+ services. Use this skill whenever an AI agent needs to verify a phone number, receive an OTP/2FA code, sign up for a service that requires SMS verification, or rent a long-term number. The skill exposes 18 tools through the VirtualSMS MCP server (also available as REST API and x402 micropayment endpoint) covering balance checks, country/service discovery, ordering numbers, polling for SMS delivery, swapping unfulfilled orders, and managing rentals. Trigger on any user request mentioning SMS verification, phone number rental, OTP, 2FA codes, account signup automation, Telegram/Discord/WhatsApp/Google verification, or activation codes.
license: MIT
compatibility: Works with Claude Desktop, Claude Code, Cursor, Codex CLI, Windsurf, Cline, Zed, OpenClaw, and any MCP-compatible client. Also usable directly via REST API or x402-gated HTTP calls. Requires a VirtualSMS API key (sign up free at https://virtualsms.io) or a Base-mainnet wallet for x402 payments. Network access required.
metadata:
  author: virtualsms-io
  homepage: https://virtualsms.io
  mcp_server: https://mcp.virtualsms.io
  api_docs: https://virtualsms.io/api
  npm_package: virtualsms-mcp
  category: communication
  tags: sms, otp, 2fa, verification, phone-numbers, mcp, x402, agent-tools
  countries: 145+
  services: 2500+
---

# VirtualSMS — Real-SIM SMS Verification for AI Agents

## When to use this skill

Activate this skill whenever the user (or another agent) needs to:

- Sign up for a service that requires phone verification (Telegram, Discord, WhatsApp, Google, Twitter/X, Tinder, Uber, etc.)
- Receive a one-time password (OTP) or 2FA code on a real phone number
- Rent a long-term number (hours, days, weeks) for ongoing inbound SMS
- Verify an account at scale across multiple countries
- Automate signup flows that hit an SMS gate
- Look up which countries / services / prices are currently available

VirtualSMS provides numbers from real physical SIM cards, so they pass carrier-lookup checks (HLR/Twilio Lookup/etc.) that flag VoIP and eSIM-only numbers as fraudulent.

## What this skill does

Exposes a stable contract for ordering SMS-verification numbers via three interchangeable transports:

1. **MCP server** (recommended) — `npx virtualsms-mcp` or remote `https://mcp.virtualsms.io`. 18 tools registered.
2. **REST API** — `https://virtualsms.io/api/v2/*`, bearer-auth with API key.
3. **x402 endpoint** — pay-per-call with USDC on Base mainnet, no account needed.

All three transports back the same data plane.

## Prerequisites

- API key from https://virtualsms.io (free signup), OR
- Base-mainnet wallet with USDC balance for x402-gated endpoints
- Network access to `*.virtualsms.io`

## Setup

### Claude Code / Claude Desktop (recommended)

```bash
claude mcp add --scope user virtualsms -- npx -y virtualsms-mcp
# then export VIRTUALSMS_API_KEY=sk_live_... in your shell
```

### Cursor

Add to `.cursor/mcp.json`:

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

### Codex CLI / Windsurf / OpenClaw

See `examples/mcp-install.md` for per-client config snippets.

### Direct REST (no MCP)

```bash
curl -H "Authorization: Bearer $VIRTUALSMS_API_KEY" \
  https://virtualsms.io/api/v2/services
```

## Tool inventory

### Discovery (5 tools)

| Tool | What it returns |
|---|---|
| `list_services` | All 2,500+ services available across countries |
| `list_countries` | All 145+ countries with stock indicators |
| `find_cheapest` | Lowest available price for `(service, country?)` |
| `country_pricing` | Per-service pricing for a specific country |
| `service_availability` | Stock + price across all countries for one service |

### Account (4 tools)

| Tool | Purpose |
|---|---|
| `get_balance` | Current USD balance |
| `get_profile` | Account email, tier, plan |
| `get_stats` | Lifetime activations + spend |
| `list_transactions` | Deposit + spend ledger |

### Order management (9 tools)

| Tool | Purpose |
|---|---|
| `create_order` | Buy a number for `(service, country)` |
| `get_order` | Read current state of an order |
| `list_orders` | Paginated active + recent orders |
| `wait_for_sms` | Poll until SMS arrives or timeout |
| `swap_order` | Replace a number that didn't receive SMS (free) |
| `cancel_order` | Cancel before SMS arrives (full refund) |
| `confirm_order` | Mark received SMS as accepted |
| `extend_rental` | Add time to a long-term rental |
| `release_rental` | End a long-term rental early |

## Recommended flow

```
1. find_cheapest(service="telegram", country=null)
2. create_order(service="telegram", country="ru")  → returns order_id + phone_number
3. Trigger the verification on the target site using phone_number
4. wait_for_sms(order_id, timeout=180)            → returns code
5. If no code: swap_order(order_id) (free) and retry, OR cancel_order(order_id) for full refund
6. Once code accepted: confirm_order(order_id)
```

## Pricing

Pricing is pulled from the live API — exact prices are returned by `find_cheapest` and `country_pricing` at request time. Indicative range starts from $0.05/code for high-volume services and scales by country and service. Always quote real prices from the API, never hardcode.

x402 endpoints are settled per-call in USDC on Base mainnet — no account, no minimum.

## Why real SIMs

Most "virtual number" services route through VoIP/eSIM ranges that fail carrier-lookup checks (Twilio Lookup, HLR queries, NumVerify, etc.). Services that care about verification quality (banks, KYC platforms, large social networks) reject those ranges. VirtualSMS sources from physical SIMs in 145+ countries so the numbers survive deep-verification flows.

## Safety notes

- **Don't rent numbers for fraud, account takeover, harassment, or any prohibited use.** VirtualSMS retains the right to refuse service and ban accounts that violate the terms at https://virtualsms.io/terms.
- **One number ≠ one user.** A number may be reused for different services across renters; always poll only your own `order_id` to read the SMS.
- **Codes expire.** If `wait_for_sms` times out, prefer `swap_order` (free) over creating a new order.
- **API keys are bearer credentials.** Treat like passwords; rotate from https://virtualsms.io/dashboard if leaked.

## Examples

See `examples/` for end-to-end recipes:

- `examples/verify-discord.md` — sign up for Discord with a Russian number
- `examples/verify-telegram.md` — receive a Telegram login code
- `examples/rent-long-term.md` — rent a number for a week of inbound SMS
- `examples/x402-payment.md` — pay-per-call without an account
- `examples/mcp-install.md` — install snippets for 6+ MCP-compatible clients

## API reference

See `docs/api-reference.md` for full REST + MCP endpoint documentation.

## Links

- Website: https://virtualsms.io
- MCP server (open source): https://github.com/virtualsms-io/mcp-server
- npm: https://www.npmjs.com/package/virtualsms-mcp
- API docs: https://virtualsms.io/api
- Status: https://status.virtualsms.io
- Support: https://virtualsms.io/contact

## License

MIT — see [LICENSE](LICENSE).
