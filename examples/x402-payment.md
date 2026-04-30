# Example: Pay-per-call with x402 (no account)

VirtualSMS exposes its core endpoints via the [x402](https://www.x402.org) protocol — agents pay with USDC on Base mainnet on a per-request basis. No signup, no API key, no balance management.

## When to use

- Your agent has a Base-mainnet wallet but no VirtualSMS account
- You want pay-as-you-go pricing without pre-funding a balance
- You're building agent-native workflows where wallet-as-identity is the norm

## Setup

You need:

- A wallet with USDC on Base mainnet
- An x402-aware HTTP client (e.g., [`x402-fetch`](https://www.npmjs.com/package/x402-fetch), [`x402-axios`](https://www.npmjs.com/package/x402-axios), or any client that handles HTTP 402 responses)

## Endpoints

x402-gated mirrors of the REST API live at:

```
https://x402.virtualsms.io/api/v2/services
https://x402.virtualsms.io/api/v2/countries
https://x402.virtualsms.io/api/v2/orders        (POST to buy)
https://x402.virtualsms.io/api/v2/orders/{id}
https://x402.virtualsms.io/api/v2/orders/{id}/sms
```

## Flow

### 1. Make the request — server returns 402 with payment requirements

```bash
curl -i https://x402.virtualsms.io/api/v2/services
```

```http
HTTP/1.1 402 Payment Required
X-Payment-Required: { "amount": "0.01", "asset": "USDC", "chain": "base", "recipient": "0x..." }
```

### 2. Sign + retry with `X-Payment` header

The x402 client library handles this automatically. With `x402-fetch`:

```js
import { wrapFetchWithPayment } from "x402-fetch";
import { createWalletClient, http } from "viem";
import { base } from "viem/chains";

const wallet = createWalletClient({ chain: base, transport: http(), account });
const fetchPaid = wrapFetchWithPayment(fetch, wallet);

const res = await fetchPaid("https://x402.virtualsms.io/api/v2/services");
const services = await res.json();
```

### 3. Buy a number (also pay-per-call)

```js
const order = await fetchPaid("https://x402.virtualsms.io/api/v2/orders", {
  method: "POST",
  body: JSON.stringify({ service: "discord", country: "in" }),
  headers: { "Content-Type": "application/json" },
});
```

### 4. Poll for SMS (also pay-per-call)

```js
const sms = await fetchPaid(`https://x402.virtualsms.io/api/v2/orders/${order.order_id}/sms`);
```

## Pricing

Each call charges a small USDC amount returned by the 402 handshake. Order creation includes the number cost; SMS poll calls cost a fraction of a cent. Exact prices are returned by the server in the `X-Payment-Required` response — never hardcode.

## Common pitfalls

- **The x402 client must be on the same chain (Base mainnet)** — Sepolia testnet endpoints exist at `x402-test.virtualsms.io` for development
- **402 responses are stateless** — your client must support the retry-with-payment pattern
- **Bridge USDC to Base before first use** — Coinbase, Across, or any Base-supporting bridge work
