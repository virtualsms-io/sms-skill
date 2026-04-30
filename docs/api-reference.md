# VirtualSMS API Reference

Three transports, identical data plane. Pick the one that matches your stack.

## Transports

| Transport | Auth | Best for |
|---|---|---|
| **MCP** (`virtualsms-mcp`) | API key in env | AI agents in Claude Code / Cursor / Codex / Windsurf |
| **REST** (`https://virtualsms.io/api/v2`) | Bearer token | Backend services, scripts, cURL |
| **x402** (`https://x402.virtualsms.io/api/v2`) | Wallet signature on Base | Pay-per-call, no account |

All three resolve to the same orders, the same balance, and the same SMS storage.

---

## REST endpoints

Base URL: `https://virtualsms.io/api/v2`
Auth header: `Authorization: Bearer sk_live_...`

### Discovery

| Method | Path | Returns |
|---|---|---|
| `GET` | `/services` | `[{id, name, category}, ...]` — 2,500+ services |
| `GET` | `/countries` | `[{iso2, name, in_stock}, ...]` — 145+ countries |
| `GET` | `/services/:service/cheapest` | `{country, price, stock}` |
| `GET` | `/countries/:iso2/pricing` | `[{service, price, stock}, ...]` |
| `GET` | `/services/:service/availability` | `[{country, price, stock}, ...]` |

### Account

| Method | Path | Returns |
|---|---|---|
| `GET` | `/me` | `{email, balance, plan, ...}` |
| `GET` | `/me/balance` | `{balance, currency}` |
| `GET` | `/me/stats` | `{lifetime_orders, lifetime_spend, ...}` |
| `GET` | `/me/transactions` | paginated ledger |

### Orders

| Method | Path | Body / Returns |
|---|---|---|
| `POST` | `/orders` | `{service, country, rental_hours?}` → `{order_id, phone_number, expires_at, price}` |
| `GET` | `/orders` | paginated active + recent |
| `GET` | `/orders/:id` | full order state |
| `GET` | `/orders/:id/sms` | poll for SMS — long-poll up to 180 sec |
| `POST` | `/orders/:id/swap` | swap to a new number, free |
| `POST` | `/orders/:id/cancel` | cancel + full refund (only before SMS arrives) |
| `POST` | `/orders/:id/confirm` | finalize charge |
| `POST` | `/orders/:id/extend` | `{hours}` → extend rental |
| `POST` | `/orders/:id/release` | end rental early |

### Order state machine

```
created → waiting → received → confirmed
               ↓
            swapped (free, returns to waiting)
               ↓
            cancelled (full refund) | expired
```

---

## MCP tools

The MCP server registers 18 tools, one per REST endpoint above plus aggregations. All tools return JSON-serializable values.

| Tool name | Equivalent REST |
|---|---|
| `list_services` | `GET /services` |
| `list_countries` | `GET /countries` |
| `find_cheapest` | `GET /services/:s/cheapest` |
| `country_pricing` | `GET /countries/:c/pricing` |
| `service_availability` | `GET /services/:s/availability` |
| `get_balance` | `GET /me/balance` |
| `get_profile` | `GET /me` |
| `get_stats` | `GET /me/stats` |
| `list_transactions` | `GET /me/transactions` |
| `create_order` | `POST /orders` |
| `get_order` | `GET /orders/:id` |
| `list_orders` | `GET /orders` |
| `wait_for_sms` | `GET /orders/:id/sms` (long-polled) |
| `swap_order` | `POST /orders/:id/swap` |
| `cancel_order` | `POST /orders/:id/cancel` |
| `confirm_order` | `POST /orders/:id/confirm` |
| `extend_rental` | `POST /orders/:id/extend` |
| `release_rental` | `POST /orders/:id/release` |

---

## x402 endpoints

Base URL: `https://x402.virtualsms.io/api/v2`. Same paths as REST — auth replaced by HTTP 402 handshake.

See [`examples/x402-payment.md`](../examples/x402-payment.md) for the wallet-signing flow.

---

## Errors

All transports return RFC 7807 problem-details JSON on errors:

```json
{
  "type": "https://virtualsms.io/errors/insufficient-balance",
  "title": "Insufficient balance",
  "status": 402,
  "detail": "Balance $0.05 < required $0.18",
  "instance": "/api/v2/orders"
}
```

Common error types:

| HTTP | Type | Cause |
|---|---|---|
| 400 | `validation` | bad request body |
| 401 | `unauthorized` | missing / wrong API key |
| 402 | `insufficient-balance` | top up at https://virtualsms.io/dashboard |
| 404 | `service-not-found` / `country-not-found` / `order-not-found` | bad slug |
| 409 | `order-already-confirmed` | trying to swap/cancel a confirmed order |
| 410 | `order-expired` | order window passed |
| 429 | `rate-limited` | back off per `Retry-After` header |
| 503 | `no-stock` | retry with a different country |

---

## Rate limits

- 60 req/min per API key on most endpoints
- `wait_for_sms` is long-polled — count as 1 req per call regardless of duration
- 429 responses include `Retry-After` (seconds)

---

## Webhooks (REST + x402 only)

Configure at https://virtualsms.io/dashboard → Developer → Webhooks. Events:

- `order.sms_received`
- `order.confirmed`
- `order.cancelled`
- `order.swapped`
- `balance.deposited`

Signed with HMAC-SHA256 in the `X-VirtualSMS-Signature` header.

---

## More

- Live API explorer: https://virtualsms.io/api
- Code examples (Python / Node / cURL): https://github.com/virtualsms-io/examples
- MCP server source: https://github.com/virtualsms-io/mcp-server
- Status: https://status.virtualsms.io
