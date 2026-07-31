---
name: Stream bit.com market data and account events
description: Pull a market snapshot over REST, then subscribe to public and private WebSocket channels on bit.com (Matrixport).
api: openapi/matrixport-bitcom-openapi.yml
operations: [getTickers, getOrderbooks, getWsAuthToken]
---

# Stream market data and account events (bit.com / Matrixport)

## Snapshot over REST (public)
1. Call `getTickers` for a starting price/stats snapshot of your instrument.
2. Call `getOrderbooks` for current depth.

## Subscribe over WebSocket
Connect to `wss://ws.bit.com` (testnet `wss://betaws.bitexch.dev`) and send:
`{ "type": "subscribe", "channels": [ ... ] }`.

- **Public channels** (no auth): `ticker`, `order_book` (e.g. `order_book.10.10`), `trade`, `kline`.
- **Private channels** (auth): `order`, `user_trade`, `um_account`.

## Private channel auth
1. Call `getWsAuthToken` (`GET /v1/ws/auth`, signed) to obtain a WebSocket auth token.
2. Include the token in the subscribe message for the private channels.

## Rules
- Private-channel auth uses the same access-key + HMAC-SHA256 model as REST
  (`authentication/matrixport-authentication.yml`).
- Channel names above are taken verbatim from the official SDK; see
  `asyncapi/matrixport-websocket.yml` for the full channel catalog.
