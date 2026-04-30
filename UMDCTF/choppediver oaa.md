# choppediver oaa

Flag: `UMDCTF{cmsc_417_TcP_3nj0y3r}`

## Challenge Type

Web / protocol logic.

## Source Review

`server.py` reveals the real bug:

- Each round sets `deadline = time.monotonic() + budget_ms / 1000.0`.
- The timeout check only happens in the `except TimeoutError` branch.
- Any received heartbeat is accepted and answered with a huge `hb_ack`.
- `keepalive.py` makes every heartbeat ack large by returning a fresh `nonce = secrets.token_hex(2048)`.

That means the round does **not** expire while the server keeps receiving messages quickly enough to avoid `recv(timeout=0.001)` timing out.

## Exploit

The trick is to queue heartbeat messages **for the next round before that round starts**:

1. Send `ready`.
2. Immediately send a burst of heartbeat messages.
3. Receive the current round chart.
4. Classify it.
5. Send the answer.
6. Immediately send another heartbeat burst.

Why this works:

- Messages sent **after** the answer are still unread when the server returns from `_play_round()`.
- The next round starts with those heartbeats already sitting in the socket buffer.
- When `_play_round()` for the next round begins reading, it consumes queued heartbeats instead of hitting a timeout.
- Because the server only checks `deadline` on `TimeoutError`, the answer can be processed far after a nominal `5 ms` budget.

This is why round 14+ becomes solvable over the network.

## Chart Classifier

The charts are simple candlestick trends:

- `buy`: strong uptrend
- `sell`: strong downtrend
- `hold`: sideways movement

I collected labeled round-1 samples by intentionally answering and reading the rejection reason. A lightweight image heuristic was enough:

- Crop the plot area.
- Threshold non-background pixels.
- If foreground density is high, classify as `hold`.
- Otherwise compute the foreground slope:
  - negative slope -> `buy`
  - positive slope -> `sell`

This was perfect on the collected sample set and worked live.

## Solve Script

The full solver is in [solve.py](/home/kali/ctf-orc/ctf/choppediver-oaa/solve.py).

Run it with:

```bash
python solve.py
```
