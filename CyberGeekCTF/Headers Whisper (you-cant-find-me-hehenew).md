# Headers Whisper Writeup

## Challenge

- Name: `Headers Whisper`
- Category: `Steganography / Signal Processing`
- Flag format: `geek{...}`

## Recon

The PCAP contains only ICMP echo requests from `192.168.1.50`.

Key observations:

- No ICMP payload data is present.
- The capture is not stored in strict chronological order.
- After sorting by timestamp, the first 27 packets all go to `192.168.1.100`.
- Those first 27 packets have `icmp.seq = 0..26`.

That first 27-packet block is the real clue. The remaining packets are noise/decoys.

## Useful extraction

Sort packets by timestamp, take the first 27 packets, then decode each packet with:

```text
char = chr(((ip.id XOR icmp.seq) % 95) + 32)
```

Applying that to the first 27 chronological packets yields:

```text
geek{n0t@th4t@34sy@41@2026}
```

## Minimal solve script

```python
from scapy.all import rdpcap, IP, ICMP

pkts = []
for p in rdpcap("capture_XTpM7L8.pcap"):
    if IP in p and ICMP in p:
        pkts.append({
            "time": float(p.time),
            "id": p[IP].id,
            "seq": p[ICMP].seq,
        })

pkts.sort(key=lambda x: x["time"])
flag = "".join(chr((((p["id"] ^ p["seq"]) & 0xff) % 95) + 32) for p in pkts[:27])
print(flag)
```

## Flag

```text
geek{n0t@th4t@34sy@41@2026}
```
