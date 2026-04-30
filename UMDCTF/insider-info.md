# insider-info

Flag: `UMDCTF{5Ur31Y_N0_0N3_W111_N071C3_MY_1N51D3r_7r4D1N6}`

## Summary

The service accepts exactly two raw DNS packets on a TCP socket. Each packet is length-prefixed with a 2-byte big-endian size, then forwarded to a local `dnslib` UDP resolver.

The bug is that the resolver limits the number of packets, not the number of DNS questions inside one packet:

- `*.inside.info` returns `secret[index]`
- `subdomain + ".inside.info"` returns the flag

`subdomain` is the full 819-character secret split into 63-byte labels. That domain is too long for normal DNS helpers, but `dnslib` will still parse a raw overlong QNAME if we encode it ourselves.

## Exploit

1. Send one DNS packet containing 819 TXT questions:
   - `0.inside.info`
   - `1.inside.info`
   - ...
   - `818.inside.info`
2. Read all 819 TXT answers and concatenate them to recover `secret`.
3. Build a second raw DNS packet whose QNAME is:

```
<secret[0:63]>.<secret[63:126]>....<secret[756:819]>.inside.info
```

4. Mark that question as TXT and send it as the second and final packet.
5. The server responds with a TXT record for `flag.inside.info`.

## Notes

- The service framing is big-endian.
- The first query is only about 8 KB, so fitting 819 questions in one packet is easy.
- The second query must be built manually because `dnslib` refuses to construct such a long domain through its normal `DNSLabel` path.

## Solver

Run:

```bash
python3 solve.py
```
