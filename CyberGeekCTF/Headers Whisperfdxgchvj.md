# Headers Whisperfdxgchvj

## Flag

`geek{n0t_th4t_34sy_41_2026}`

## Recon

The PCAP contains 274 ICMP echo requests from `192.168.1.50` with no payload data.
That immediately rules out the usual "data hidden in ICMP payload" path and pushes the analysis toward header fields and timing.

One destination stood out during sorting by timestamp:

- `192.168.1.100` receives 27 ICMP echo requests
- Those 27 packets are the earliest packets in the capture
- Their `icmp.seq` values are exactly `0` through `26`
- Their `icmp.ident` is constant `0`

That 27-packet block looked like a deliberate encoded message rather than background traffic.

## Extraction

For the `192.168.1.100` packets, the low byte of `ip.id` decodes cleanly when XORed with `0x20 + icmp.seq`.

Formula:

```text
decoded_char = low_byte(ip.id) XOR (0x20 + icmp.seq)
```

Example from the first few packets:

- `ip.id = 0x0047`, `icmp.seq = 0` -> `0x47 ^ 0x20 = 0x67` -> `g`
- `ip.id = 0x0044`, `icmp.seq = 1` -> `0x44 ^ 0x21 = 0x65` -> `e`
- `ip.id = 0x0047`, `icmp.seq = 2` -> `0x47 ^ 0x22 = 0x65` -> `e`
- `ip.id = 0x0048`, `icmp.seq = 3` -> `0x48 ^ 0x23 = 0x6b` -> `k`
- `ip.id = 0x005f`, `icmp.seq = 4` -> `0x5f ^ 0x24 = 0x7b` -> `{`

This immediately reveals the prefix `geek{`.

Decoding all 27 packets yields:

```text
geek{n0t_th4t_34sy_41_2026}
```

## Reproduction

Run:

```bash
python3 solve.py
```

It prints:

```text
geek{n0t_th4t_34sy_41_2026}
```
