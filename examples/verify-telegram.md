# Example: Receive a Telegram login code

Telegram is the highest-volume verification target on VirtualSMS — recipe is essentially the same as Discord but with service-specific tips.

## Steps

```
1. find_cheapest(service="telegram")
   → returns lowest-price country (often ru, kz, or id)
2. create_order(service="telegram", country="<from step 1>")
   → returns order_id + phone_number
3. Trigger the Telegram login flow with phone_number
4. wait_for_sms(order_id, timeout=180)
   → returns the 5-digit code Telegram sent
5. confirm_order(order_id)
```

## Service-specific tips

- **Telegram codes are 5 digits** — agents should expect that exact format
- **Telegram delivers via in-app push first if the number is on Telegram already** — VirtualSMS still captures the SMS fallback within ~60 sec
- **Don't poll faster than 5 sec** — `wait_for_sms` already implements optimal polling
- **If the same number was on Telegram recently, you may receive a "this number is already in use" UI flow on the client side** — request a new number via `swap_order`

## Long-term Telegram bot rentals

For inbound SMS over hours/days (e.g., a Telegram bot account that needs to receive ongoing 2FA challenges), use a long-term rental instead:

```
create_order(service="telegram", country="ru", rental_hours=168)
```

This holds the number for 7 days. Use `wait_for_sms` repeatedly during that window. Extend with `extend_rental(order_id, hours=N)` or release early with `release_rental(order_id)`.

See `rent-long-term.md` for details on rentals.
