# Takeshi's Castle

## Challenge Overview
- Event: CyberGeek CTF
- Category: Reverse Engineering / VM
- Final Flag: geek{f1n@l_5h0d0wn}

## Executive Summary
The attachment is a stripped 64-bit ELF. It would not execute in this environment because the host only had an AArch64 `libstdc++.so.6`, so I solved it statically.

## Solution Walkthrough
### Recon
- `file takeshisCastle` shows a stripped PIE C++ binary.
- The interesting strings are:
  - `Wrong flag.`
  - `Correct!! My job ends here!`
  - `Nopenope!!`
  - `Got it?:`

Disassembly of `main` shows:

- the binary allocates `0x198` bytes and copies a blob from `.rodata` at `0x2140`
- user input is copied into a zeroed buffer
- a verifier at `0x1470` interprets the blob as bytecode

### VM
The verifier is a tiny VM with these opcodes after decoding each byte as:

`decoded = bytecode[ip] ^ ip ^ key`

Initial `key = 0x42`.

Meaningful opcodes:

- `0x10`: set index register `A` from the next decoded byte
- `0x11`: `B = input[A]`
- `0x12`: `C = input[A + 1]`
- `0x20`: `B ^= C`
- `0x21`: add the next decoded byte to `B`
- `0x30`: compare `B` with the next decoded byte
- `0x40`: update key with `key = ((key * 2) ^ 0x1b) & 0xff`
- `0x50`: fail if the previous compare was false
- `0xff`: success

After decoding the program, the repeated block is:

```text
SET_A i
LOAD input[i]
LOAD input[i + 1]
XOR
ADD 10
CMP target[i]
ASSERT
KEY
```

So every block enforces:

`input[i] ^ input[i + 1] = target[i] - 10`

The compare bytes decode to:

```text
12 12 12 12 12 12 12 13 90 91 11 14 13 16 17 15 17 15 21
18 90 96 92 97 94 91 12 17 13 16 95 91 13 12 90 91 21
```

That gives 37 XOR equations over 38 bytes. The first byte is never fixed, so there are 256 valid inner strings.

### Picking the Intended Body
Enumerating the whole family shows only one candidate that is entirely lowercase hexadecimal:

`202020203c23742507291a7e2f75217b302b38`

That hex string is not the final submission yet. Hex-decoding it gives 19 bytes:

```text
20 20 20 20 3c 23 74 25 07 29 1a 7e 2f 75 21 7b 30 2b 38
```

The challenge tells us the real flag format starts with `geek{`, so XORing the first bytes of the ciphertext with `g e e k {` yields:

```text
20 ^ 67 = 47  -> 'G'
20 ^ 65 = 45  -> 'E'
20 ^ 65 = 45  -> 'E'
20 ^ 6b = 4b  -> 'K'
3c ^ 7b = 47  -> 'G'
```

That immediately exposes the repeating XOR key: `GEEK`.

Applying repeating-key XOR with `GEEK` to the full ciphertext reveals:

`geek{f1n@l_5h0d0wn}`

### Reproduction
Run:

```bash
python3 solve.py
```

The solver extracts the bytecode from the binary, rebuilds the constraint system, finds the unique lowercase-hex accepted input, hex-decodes it, recovers the repeating XOR key from the known `geek{` prefix, and prints the final flag.

## Flag
```text
geek{f1n@l_5h0d0wn}
```
