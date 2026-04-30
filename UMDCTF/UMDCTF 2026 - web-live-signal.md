# UMDCTF 2026 - web/live-signal

Flag: `UMDCTF{z0d_1s_f0r_sc4r3dy_c4ts}`

## Summary

The site is a Next.js app with two relevant server actions:

- `signalsByAnalyst`
- `verifyInsider`

`/insiders` is rate-limited and wants a hidden future signal tuple:

- `ticker`
- `side`
- `contractPrice`
- `signalHmac`

The intended bug is not in `/insiders` directly. The bug is in the public
`signalsByAnalyst` action: it accepts Prisma-style relation filters instead
of validating the input shape.

That gives a boolean oracle over embargoed future rows.

## Recon

The public analyst table uses this server action:

- action id: `60a1ddfeb092ed787fb44362a27b61cff6e945d3d2`

The insider form uses:

- action id: `783c22fa5f432da054e7f973cb1a827a956078d769`

Pulling public signals directly showed that each row includes a `signalHmac`,
but submitting public tuples to `/insiders` fails with:

- `We can't verify your access.`

So the portal wants an embargoed row, not a public one.

## Key clue

`/llms.txt` is exposed in `sitemap.xml` and explicitly warns against:

- relation-filter injection
- nested predicates against the `signals` relation
- boolean-oracle extraction of embargoed rows

That is effectively the vuln description.

## Vulnerability

`signalsByAnalyst` takes a filter object. Public UI only sends:

- `handle`
- `tier`

But the server accepts nested Prisma filters such as:

```json
{
  "handle": "quant_sable",
  "signals": {
    "some": {
      "publishedAt": { "gt": "2026-04-25T00:00:00.000Z" }
    }
  }
}
```

If the filter matches an analyst, the action returns that analyst's public tape.
If not, it returns no rows.

That means we can ask yes/no questions about hidden future signals.

## Hidden signal recovery

The homepage countdown leaks the next drop timestamp:

- `2026-04-26T23:00:00.000Z`

Using relation-filter probes on `signals.some`, I recovered:

- `handle = quant_sable`
- `publishedAt = 2026-04-26T23:00:00.000Z`
- `side = NO`
- `ticker = KXELECTION-28NOV`
- `contractPrice = 41`

Then I recovered the hidden HMAC with a prefix oracle:

```json
{
  "handle": "quant_sable",
  "signals": {
    "some": {
      "publishedAt": "2026-04-26T23:00:00.000Z",
      "side": "NO",
      "ticker": "KXELECTION-28NOV",
      "contractPrice": 41,
      "signalHmac": { "startsWith": "fc434b..." }
    }
  }
}
```

Final hidden tuple:

- `ticker = KXELECTION-28NOV`
- `side = NO`
- `price = 41`
- `signalHmac = fc434bd5283de7356831a82e8838632c`

## Final request

Submitting that exact tuple to the insider action returns the flag:

```json
["KXELECTION-28NOV","NO",41,"fc434bd5283de7356831a82e8838632c"]
```

Server response:

```json
{"ok":true,"message":"Welcome to the inside.","flag":"UMDCTF{z0d_1s_f0r_sc4r3dy_c4ts}"}
```

## Notes

- `/insiders` itself validates input shape, so direct filter-object abuse there
  does not work.
- The solve path is to abuse the public action as an oracle, not to crack the
  HMAC scheme.
