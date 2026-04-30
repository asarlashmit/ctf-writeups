# closing-bellxs

## Recon

The provided `nc challs.umdctf.io 30310` service did not expose a useful text protocol. Searching the local archive for the challenge theme led to the UMDCTF 2026 `rev/rainbet` files, which match the betting/mines/chicken mechanics exactly.

The live challenge service is:

- `https://rainbet.challs.umdctf.io/`
- `wss://rainbet.challs.umdctf.io/ws`

## Vulnerability

Two critical leaks make the game fully predictable:

1. The backend RNG generator is shipped locally as `rainbet_gen.wasm`.
2. The live endpoint `/api/sessioninfo` returns both the `session_id` and the HMAC `secret` used to sign actions.

That means each round can be reproduced exactly from `(session_id, round_idx)`, and each move can be signed legitimately.

## Exploit

For each round:

1. Fetch `/api/sessioninfo` to get a fresh `session_id` and `secret`.
2. Run the leaked WASM locally to generate the exact round for the current streak.
3. If the round is `mines`, reveal every non-mine tile.
4. If the round is `chicken`, cross until the first car and then cash out.
5. Sign each action as `HMAC-SHA256(secret, canon_view)` and send it over the websocket.

The existing solver at `../rev-rainbet/solve.js` already implements this flow.

## Result

Running:

```bash
node ../rev-rainbet/solve.js
```

returned:

```text
UMDCTF{one_might_argue_that_gambling_is_the_best_vice_but_they_would_be_wrong}
```
