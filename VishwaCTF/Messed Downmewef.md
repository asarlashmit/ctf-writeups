# Messed Downmewef Writeup

Challenge: `Messed Downmewef`  
Type: `rev`

## Flag

`VishwaCTF{c6f8baf9323f0c542e9595262aba166f7d574d84a249daec61ad33e2d55ab0ee}`

## Summary

The prompt is a decoy. After the `read` syscall, the executed path never reads the user buffer again. The program always prints the input back as invalid and exits.

The real work happens on a fixed internal 6-byte buffer that is copied onto the stack, padded with `0x80`, and then processed with the standard SHA-256 IV and round constants. The important detail is that the binary does not build a normal SHA-256 padded message. It only prepares a single 64-byte block and runs one compression round over it.

## Recon

Basic inspection:

```bash
file MessedDown
readelf -l MessedDown
printf 'ABCDE\n' | qemu-x86_64 -strace ./MessedDown
```

Relevant runtime behavior:

```text
Security is Compromised!
Need to add another stage :)
Enter Secret:
Your Secret "ABCDE" is invalid!
```

The `strace` output shows the binary only:

- writes the intro and prompt
- reads up to 127 bytes into a stack buffer
- echoes the input back as invalid
- exits

Dynamic emulation confirmed that the input buffer is not part of validation.

## Key Findings

The fixed 6-byte value is written here:

```asm
0x6f5de: movabs rax, 0x776334436281
0x6f5e8: mov    qword ptr [rsp+0x110], rax
```

Because the immediate is little-endian in memory, the actual bytes are:

```text
81 62 43 34 63 77
```

The binary then zeroes a 64-byte working block, copies those 6 bytes, and writes `0x80` after them. It never writes the usual final 64-bit message length.

Final block:

```text
81 62 43 34 63 77 80 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

The stack also contains the standard SHA-256 IV and `K` table, which identifies the intended operation.

## Solve

Run one SHA-256 compression round over that exact 64-byte block with the standard IV.

```python
import struct

block = bytes.fromhex("81624334637780" + "00" * 57)

IV = [
    0x6A09E667, 0xBB67AE85, 0x3C6EF372, 0xA54FF53A,
    0x510E527F, 0x9B05688C, 0x1F83D9AB, 0x5BE0CD19,
]

K = [
    0x428A2F98, 0x71374491, 0xB5C0FBCF, 0xE9B5DBA5,
    0x3956C25B, 0x59F111F1, 0x923F82A4, 0xAB1C5ED5,
    0xD807AA98, 0x12835B01, 0x243185BE, 0x550C7DC3,
    0x72BE5D74, 0x80DEB1FE, 0x9BDC06A7, 0xC19BF174,
    0xE49B69C1, 0xEFBE4786, 0x0FC19DC6, 0x240CA1CC,
    0x2DE92C6F, 0x4A7484AA, 0x5CB0A9DC, 0x76F988DA,
    0x983E5152, 0xA831C66D, 0xB00327C8, 0xBF597FC7,
    0xC6E00BF3, 0xD5A79147, 0x06CA6351, 0x14292967,
    0x27B70A85, 0x2E1B2138, 0x4D2C6DFC, 0x53380D13,
    0x650A7354, 0x766A0ABB, 0x81C2C92E, 0x92722C85,
    0xA2BFE8A1, 0xA81A664B, 0xC24B8B70, 0xC76C51A3,
    0xD192E819, 0xD6990624, 0xF40E3585, 0x106AA070,
    0x19A4C116, 0x1E376C08, 0x2748774C, 0x34B0BCB5,
    0x391C0CB3, 0x4ED8AA4A, 0x5B9CCA4F, 0x682E6FF3,
    0x748F82EE, 0x78A5636F, 0x84C87814, 0x8CC70208,
    0x90BEFFFA, 0xA4506CEB, 0xBEF9A3F7, 0xC67178F2,
]

def rotr32(x, n):
    return ((x >> n) | (x << (32 - n))) & 0xFFFFFFFF

w = list(struct.unpack(">16I", block))
for i in range(16, 64):
    s0 = rotr32(w[i - 15], 7) ^ rotr32(w[i - 15], 18) ^ (w[i - 15] >> 3)
    s1 = rotr32(w[i - 2], 17) ^ rotr32(w[i - 2], 19) ^ (w[i - 2] >> 10)
    w.append((w[i - 16] + s0 + w[i - 7] + s1) & 0xFFFFFFFF)

a, b, c, d, e, f, g, h = IV
for i in range(64):
    S1 = rotr32(e, 6) ^ rotr32(e, 11) ^ rotr32(e, 25)
    ch = (e & f) ^ ((~e) & g)
    temp1 = (h + S1 + ch + K[i] + w[i]) & 0xFFFFFFFF
    S0 = rotr32(a, 2) ^ rotr32(a, 13) ^ rotr32(a, 22)
    maj = (a & b) ^ (a & c) ^ (b & c)
    temp2 = (S0 + maj) & 0xFFFFFFFF
    h, g, f, e, d, c, b, a = g, f, e, (d + temp1) & 0xFFFFFFFF, c, b, a, (temp1 + temp2) & 0xFFFFFFFF

out = [(x + y) & 0xFFFFFFFF for x, y in zip([a, b, c, d, e, f, g, h], IV)]
digest = "".join(f"{x:08x}" for x in out)
print(digest)
```

Output:

```text
c6f8baf9323f0c542e9595262aba166f7d574d84a249daec61ad33e2d55ab0ee
```

Wrap it in the challenge format:

`VishwaCTF{c6f8baf9323f0c542e9595262aba166f7d574d84a249daec61ad33e2d55ab0ee}`
