# Headers Whisper - Easy Version

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics / network steganography
- Final Flag: geek{n0t_th4t_34sy_41_2026}

## Executive Summary
The PCAP contains 274 ICMP echo requests with no payload. That means the hidden data must be stored in packet metadata rather than ICMP body bytes.

The important clues were:

- Every packet is a 28-byte ICMP echo request, so there is no normal data section.
- The capture is not in strict time order.
- When sorted by packet timestamp, the first 27 packets all go to `192.168.1.100`.
- Those 27 packets have ICMP sequence numbers `0` through `26`, which makes them look intentionally arranged.

## Solution Walkthrough
### Decoding
For the first 27 time-ordered packets:

1. Take the low byte of the IPv4 identification field.
2. XOR it with the low byte of the ICMP sequence number.
3. XOR the result with `0x20`.
4. Convert the byte to ASCII.

That yields:

`geek{n0t_th4t_34sy_41_2026}`

### Reproduction Script
`solve.py` reproduces the extraction:

```python
packets = sorted(rdpcap("capture.pcap"), key=lambda pkt: float(pkt.time))
flag = "".join(
    chr(((pkt[IP].id & 0xFF) ^ (pkt[ICMP].seq & 0xFF) ^ 0x20))
    for pkt in packets[:27]
)
print(flag)
```

## Flag
```text
geek{n0t_th4t_34sy_41_2026}
```
