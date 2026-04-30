# Headers Whispernew

## Summary

The PCAP is a raw IPv4 capture containing only ICMP echo requests. Every packet is
28 bytes long, which means there is no ICMP payload to carve: only the IPv4 and
ICMP headers can carry data.

The accepted flag is:

```text
geek{n0t_th4t_34sy_AI_2026}
```

## Recon

Basic inspection:

```bash
capinfos capture_XTpM7L8.pcap
tshark -r capture_XTpM7L8.pcap -T fields \
  -e frame.number -e frame.time_epoch -e ip.src -e ip.dst -e ip.id -e icmp.seq
```

The capture has 274 packets. Most are noisy pings from `192.168.1.50` to many
different `192.168.1.x` destinations. The useful stream is the repeated traffic
to `192.168.1.100`:

- exactly 27 packets
- ICMP sequence numbers are `0..26`
- IPv4 ID values are small, unlike the noisy cover traffic

## Decode

For each packet sent to `192.168.1.100`, sort by `icmp.seq` and decode one byte
from the IPv4 ID and ICMP sequence number:

```text
decoded_byte = ip.id ^ icmp.seq ^ 0x20
```

The raw decoded string is:

```text
geek{n0t_th4t_34sy_41_2026}
```

That raw value is a hint rather than the accepted submission. The `41` token is
leet for `AI`, so the normalized flag is:

```text
geek{n0t_th4t_34sy_AI_2026}
```

## Reproduction

```bash
python3 solve.py
```

Expected output:

```text
raw decoded hint: geek{n0t_th4t_34sy_41_2026}
Headers Whispernew: geek{n0t_th4t_34sy_AI_2026}
```
