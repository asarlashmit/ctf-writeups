# Lovable Sahil 3

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics / Network Steganography
- Artifact(s): description, log.pcap
- Prompt: Our sensors tripped, the alarms screamed, and then... absolute silence...
- Final Flag: geek{s1gn4l_h0pp3r}

## Executive Summary
The noisy PCAP hides a sparse covert channel to a single destination. Concatenating those TCP payloads and XORing them with `0x42` carves an ELF whose checker can be reversed into the flag.

## Solution Walkthrough
### Initial Triage
The pcap was very large (~1.4M packets) and intentionally noisy.
Traffic looked synthetic/random at first glance, but one repeated destination stood out:

    destination IP: 10.13.37.10
    destination port: 1337

No meaningful HTTP/DNS/TLS channel appeared in normal protocol parsing.
That suggested a custom covert channel.

### Isolating Signal from Noise
Filtering TCP packets to 10.13.37.10:1337 gave only 15 packets.
Each had large payloads (mostly length 1000; final packet 560).

Key observation:
Payload bytes were mostly 0x42 ('B') with sparse differing bytes.
This is a common masking/padding technique in CTF stego traffic.

### Payload Decoding Breakthrough
Method:

    Concatenate payloads in timestamp order.
    XOR all bytes with 0x42.

Result:
The transformed stream started with ELF magic:

    7f 45 4c 46  -> "ELF"

So the hidden data was an embedded Linux executable.

Artifacts carved during solve:

    carved_xor42_full.bin   (full XOR output)
    carved_nz.bin           (non-zero bytes only)

The useful binary was carved_xor42_full.bin.

### Reverse Engineering the Hidden ELF
The executable contained strings:

    "Enter the flag:"
    "Access Denied."
    "Access Granted! Flag verified."

Disassembly showed input length check:

    expected length = 0x13 (19 chars)

Validation logic:

    The checker compares in 4 chunks.
    Chunks are transformed using:
    transformed_char = input_char XOR index XOR 0x5a
    Then compared with encoded bytes stored in .data.

Encoded comparison bytes extracted from .data:

    chunk1 (index 0..3):  ">=2"
    chunk2 (index 4..7):  "%,m:"
    chunk3 (index 8..12): "<g<\x0e>"
    chunk4 (index 13..18): "g$%y95"

Invert formula:

    input_char = encoded_char XOR index XOR 0x5a

Recovered candidate input:

    geek{s1gn4l_h0pp3r}

Length check:

    19 chars (passes)

### Minimal Reproduction Script (concept)
Parse pcap packets.
    Keep only TCP packets with dst=10.13.37.10 and dport=1337.
    Concatenate payloads in packet-time order.
    XOR bytes with 0x42 and write carved binary.
    Reverse validation constants from .data:
    flag[i] = enc[i] XOR i XOR 0x5a

This yields:

    geek{s1gn4l_h0pp3r}

## Flag
```text
geek{s1gn4l_h0pp3r}
```
