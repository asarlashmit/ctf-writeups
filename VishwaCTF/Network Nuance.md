# Network Nuance Writeup

## Challenge

- Name: `Network Nuance`
- Category: `Forensics / Network`

## Observation

The provided file `network_log.txt` is not a PCAP. It is a single-line text log containing entries like:

```text
Packet 0: SeqNum=186\nPacket 1: SeqNum=205\n...
```

The hint says:

> The sequence numbers in the ICMP echo requests don't match the replies.

That suggests the logged sequence numbers are shifted from their original values.

## Extraction

I parsed every `SeqNum` value and tested constant offsets until the output matched the required flag format `VishwaCTF{...}`.

The correct offset is `100`.

So for each packet:

```text
original_char = chr(SeqNum - 100)
```

## Solve Script

```python
import re

s = open("network_log.txt").read()
nums = list(map(int, re.findall(r"SeqNum=(\d+)", s)))
flag = "".join(chr(n - 100) for n in nums)
print(flag)
```

## Flag

```text
VishwaCTF{N3tw0rk_P4ck3t_H1dd3n_VH}
```
