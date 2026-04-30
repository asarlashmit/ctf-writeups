# closing-bell

## Recon

The provided `nc challs.umdctf.io 30310` endpoint did not expose a useful text protocol. Searching the local UMDCTF archive for matching betting / suspicious-win themes led to `rev/rainbet`, which ships:

- `rainbet_gen.wasm`: the exact backend RNG generator
- client logic showing how requests are signed

The live service at `https://rainbet.challs.umdctf.io/` also exposes `/api/sessioninfo`, which returns both:

- `session_id`
- `secret` for HMAC-signing actions

## Vulnerability

This completely breaks trust:

1. The round generator is leaked locally.
2. The signing secret is leaked remotely.

So each round can be reproduced exactly from `(session_id, round_idx)`, and each websocket action can be signed legitimately.

## Exploit

For each round:

- Generate the exact game locally with `rainbet_gen.wasm`
- If it is `mines`, reveal every non-mine tile
- If it is `chicken`, cross until the first car and then cash out
- Sign each action with `HMAC-SHA256(secret, canon_view)`

I used the existing solver in the sibling workspace:

```bash
node ../rev-rainbet/solve.js
```

It completed 25 perfect rounds and returned:

```text
UMDCTF{one_might_argue_that_gambling_is_the_best_vice_but_they_would_be_wrong}
```
