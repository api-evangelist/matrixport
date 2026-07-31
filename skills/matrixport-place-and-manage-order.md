---
name: Place and manage an order on bit.com
description: Authenticate, discover an instrument, place an order, then monitor and cancel it on the bit.com (Matrixport) v1 API.
api: openapi/matrixport-bitcom-openapi.yml
operations: [getInstruments, newOrder, getOpenOrders, getUserTrades, cancelOrders]
---

# Place and manage an order (bit.com / Matrixport)

Use the bit.com v1 REST API (`https://api.bit.com`, testnet `https://betaapi.bitexch.dev`).

## Auth
Every private call needs the account **access key** in the `X-Bit-Access-Key` header plus a
millisecond `timestamp` and an HMAC-SHA256 `signature` computed over the sorted request
parameters using the account **secret key**. See `authentication/matrixport-authentication.yml`.
Test on the testnet host first — never place live orders while developing.

## Steps
1. **Find the instrument.** Call `getInstruments` to resolve the `instrument_id` (e.g. a BTC
   perpetual or option) you want to trade. Public — no signing required.
2. **Place the order.** Call `newOrder` with the `instrument_id`, side, `price`, `qty`, order
   `type` (`limit` / `market` / `stop-limit` / `stop-market`) and `time_in_force`
   (`gtc` / `fok` / `ioc`). Supply an optional `label` as your client-side correlation id.
3. **Confirm it is working.** Call `getOpenOrders` for the instrument and verify the returned
   order matches your `label`.
4. **Watch fills.** Poll `getUserTrades` (or subscribe to the private `order` / `user_trade`
   WebSocket channels) to see executions.
5. **Cancel if needed.** Call `cancelOrders` with the order id (or by instrument) to withdraw the
   working order.

## Rules
- Success is `code == 0`; any non-zero `code` is an error with a descriptive `message`
  (see `errors/matrixport-problem-types.yml`).
- There is no Idempotency-Key; use `label` for correlation and always re-query `getOpenOrders`
  before retrying a placement to avoid duplicate orders.
- Respect per-endpoint rate limits (HTTP 429).
