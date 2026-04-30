# Example: Rent a number long-term

For agents that need to receive multiple SMS over hours, days, or weeks — e.g., a long-running Telegram userbot, a recurring 2FA channel, or a notification sink.

## When to use

- You need the same number to receive SMS multiple times
- The number must persist across sessions
- One-shot orders aren't enough (they release the number after the first code)

## Steps

### 1. Check rental availability + pricing

```
country_pricing(country="us", rental=true)
```

Returns rental tiers:

```json
{
  "country": "us",
  "rental_options": [
    { "hours": 4,   "price": 1.50 },
    { "hours": 24,  "price": 4.00 },
    { "hours": 168, "price": 18.00 },
    { "hours": 720, "price": 60.00 }
  ]
}
```

### 2. Create a rental

```
create_order(service="any", country="us", rental_hours=168)
```

Returns:

```json
{
  "order_id": "ord_rental_xyz",
  "phone_number": "+15551234567",
  "rental_until": "2026-05-07T00:00:00Z",
  "price": 18.00
}
```

### 3. Receive SMS during the rental window

```
wait_for_sms(order_id="ord_rental_xyz", timeout=600)
```

Call this as many times as needed — each call returns the latest unread SMS.

### 4. Optionally extend

```
extend_rental(order_id="ord_rental_xyz", hours=72)
```

### 5. Release early (optional refund logic varies)

```
release_rental(order_id="ord_rental_xyz")
```

## Pricing note

Rental prices are returned by the live API — examples above are illustrative. Always quote `country_pricing` results to the user, never the numbers in this doc.

## Common pitfalls

- **Rentals don't auto-renew** — set a reminder to call `extend_rental` before `rental_until`
- **Some services flag long-rental numbers** — for high-trust signups, prefer one-shot orders
- **Rental numbers are sometimes shared across accounts after release** — privacy-sensitive use cases should release manually rather than wait for expiry
