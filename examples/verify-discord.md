# Example: Verify a Discord account

End-to-end recipe an AI agent can follow to receive a Discord SMS verification code.

## Goal

Sign up for Discord with a phone number that passes Discord's carrier-lookup checks, capture the OTP, and confirm the order.

## Prerequisites

- VirtualSMS API key (`VIRTUALSMS_API_KEY` env var)
- MCP client connected to `virtualsms` server (or REST access)
- Account balance ≥ the price returned by `find_cheapest`

## Steps

### 1. Find the cheapest available country for Discord

```
find_cheapest(service="discord")
```

Returns something like:

```json
{
  "service": "discord",
  "country": "in",
  "price": 0.18,
  "stock": 412
}
```

### 2. Buy a number

```
create_order(service="discord", country="in")
```

Returns:

```json
{
  "order_id": "ord_a1b2c3d4",
  "phone_number": "+919876543210",
  "expires_at": "2026-04-30T01:30:00Z",
  "price": 0.18
}
```

### 3. Trigger Discord's verification

Submit `+919876543210` on Discord's signup form (browser automation or manual handoff).

### 4. Wait for the SMS

```
wait_for_sms(order_id="ord_a1b2c3d4", timeout=180)
```

Returns:

```json
{
  "status": "received",
  "code": "847291",
  "full_text": "Your Discord verification code is: 847291"
}
```

### 5. Submit the code on Discord

Type `847291` into the verification form.

### 6. Confirm the order

```
confirm_order(order_id="ord_a1b2c3d4")
```

This finalizes the charge. If you skip this, the order auto-confirms after the rental window expires.

## What to do if the SMS never arrives

```
swap_order(order_id="ord_a1b2c3d4")
```

Returns a new phone number for the same order, no extra charge. Repeat steps 3-5 with the new number.

If two consecutive swaps fail, cancel:

```
cancel_order(order_id="ord_a1b2c3d4")
```

You get a full refund.

## Common pitfalls

- **Discord rate-limits country `us`** — `in` / `ru` / `id` typically convert better
- **Don't reuse a single number across multiple Discord signups** — Discord clusters numbers
- **The number you receive may already have been used for OTHER services in the past** — that's expected and fine for new Discord signups, just don't reuse it for the same service repeatedly
