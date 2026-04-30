# UMDCTF 2026 - `web/umdmarket`

Flag: `UMDCTF{fu7ur3_j4n3_s7r33t_h1r3}`

## Summary

The frontend bundle exposes the entire binary protocol for a WebTransport market server running on `https://umdmarket.challs.umdctf.io:4443/wt`.

The intended balance progression is:

- start with `1,000,000`
- trade up to `10,000,000`
- call `BUY_FLAG (0x50)`

The bug is in `RESEND (0x26)`.

Instead of returning the original quote tuple `(requested_seq, ticker_id, yes_price, hmac)`, the server returns:

- `seq = current server seq`
- `yes_price = historical price from requested_seq`
- `hmac = valid signature over (current_seq, ticker_id, historical_price)`

That makes any old price in the resend window immediately tradable as a fresh quote.

So if a market hit:

- a high YES price at some older seq
- a low YES price at some other older seq

you can:

1. `BUY_NO` at the old high YES price, which means a very cheap NO entry.
2. Wait out the 50-tick cooldown.
3. `SELL_NO` using the old low YES price, which means an expensive NO exit.

This is a guaranteed profit using only stale prices.

## Recon

The local challenge directory was empty, so the solve came from the live target.

The homepage was a Vite SPA. Pulling `assets/index-SX7k_t3W.js` exposed:

- the WebTransport endpoint: `https://umdmarket.challs.umdctf.io:4443/wt`
- the pinned cert hash
- all message types and field layouts
- client-side parsing logic for:
  - `REGISTER (0x20)`
  - `LOGIN (0x21)`
  - `SUBSCRIBE (0x22)`
  - `FETCH_TICKERS (0x24)`
  - `FETCH_HISTORY (0x25)`
  - `RESEND (0x26)`
  - `TRADE (0x30)`
  - `PORTFOLIO (0x40)`
  - `BUY_FLAG (0x50)`

Important constants from the docs:

- `TickInterval = 100ms`
- `MaxAge = 5`
- `MinCooldown = 50`
- `ResendWindow = 500`

## Transport

I used `aioquic` instead of a browser.

The working flow was:

1. Open QUIC with HTTP/3 ALPN.
2. Send a WebTransport `CONNECT` request on the session stream.
3. Use one WebTransport bidirectional stream per RPC.
4. Read quote datagrams from the session stream.

One implementation detail: `aioquic` did not automatically decode responses on client-opened WebTransport request streams the way I wanted, so the solver treats those per-request streams as raw QUIC stream payloads after creation.

## Bug

`RESEND` is documented as returning a freshly signed quote, and that is exactly the problem.

Example behavior observed live:

- requesting an older seq like `56`
- receiving a response whose `seq` was the current value, e.g. `62`
- but whose `yes_price` was still the old historical price

That means the quote passes freshness checks in `TRADE`, while still filling at a stale price.

For a market with historical YES prices:

- high YES = `9908`
- low YES = `9801`

the NO prices are:

- low NO = `92`
- high NO = `199`

So a cycle is:

1. `RESEND(high_yes_seq, ticker)` -> fresh quote for YES=`9908`
2. `BUY_NO` at `92`
3. wait 50 ticks
4. `RESEND(low_yes_seq, ticker)` -> fresh quote for YES=`9801`
5. `SELL_NO` at `199`

That more than doubles the bankroll in one round trip.

## Exploit

The solver in [solve.py](/home/kali/ctf-orc/ctf/web-umdmarket/solve.py) does this:

1. Register a fresh user.
2. Fetch tickers and histories.
3. Subscribe to all tickers and collect initial quote history.
4. For each ticker, compute the best stale round-trip:
   - `BUY_YES` at old minimum YES and `SELL_YES` at old maximum YES, or
   - `BUY_NO` at old maximum YES and `SELL_NO` at old minimum YES.
5. Choose the ticker with the largest multiplicative return.
6. Repeat cycles until balance exceeds `10,000,000`.
7. Call `BUY_FLAG`.

The winning market during the solve was `RAIN_SEA`, using stale NO prices.

Observed bankroll progression:

- `1,000,000`
- `1,303,792`
- `2,462,656`
- `5,628,784`
- `12,865,648`

Then `BUY_FLAG` returned:

`UMDCTF{fu7ur3_j4n3_s7r33t_h1r3}`
