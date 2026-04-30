# Headers Whisper

## Challenge

We are given a classic PCAP containing an unusual burst of ICMP traffic. The flag
format is `geek{...}`.

## Reconnaissance

`capinfos` shows 274 packets, raw IPv4 encapsulation, and every packet is exactly
28 bytes. That is only a 20-byte IPv4 header plus an 8-byte ICMP echo header, so
there is no payload to carve.

Useful checks:

```bash
capinfos capture_XTpM7L8.pcap
tshark -r capture_XTpM7L8.pcap -q -z io,phs
tshark -r capture_XTpM7L8.pcap -T fields \
  -e frame.number -e frame.time_epoch -e ip.src -e ip.dst -e ip.id -e icmp.seq
```

Most packets are noisy pings from `192.168.1.50` to many hosts in
`192.168.1.0/24`. The standout conversation is `192.168.1.50 ->
192.168.1.100`: it has exactly 27 packets, its ICMP sequence numbers are
`0..26`, and its IPv4 ID values are small while the cover traffic uses random
larger IDs.

## Exploit

For the packets sent to `192.168.1.100`, sort by `icmp.seq`. The byte is hidden
across the IPv4 ID and ICMP sequence header fields:

```text
decoded_byte = (ip.id & 0xff) XOR icmp.seq XOR 0x20
```

That produces a leetspeak hint:

```text
geek{n0t_th4t_34sy_41_2026}
```

The accepted submission is the straightforward leetspeak normalization:

```text
geek{not_that_easy_al_2026}
```

## Solver

```bash
python3 solve.py
```

Output:

```text
raw decoded hint: geek{n0t_th4t_34sy_41_2026}
Headers Whisper: geek{not_that_easy_al_2026}
```
