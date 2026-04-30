# The Last Fragment

## Challenge Overview
- Event: CyberGeek CTF
- Final Flag: geek{vvirtual_m4ch1n3_m4st3r}

## Executive Summary
- Challenge type: reverse engineering
- Files: `Task 6/emulator`, `Task 6/fragment.bin`
- Flag: `geek{vvirtual_m4ch1n3_m4st3r}`

## Solution Walkthrough
### Recon
`emulator` is a stripped 64-bit x86_64 PIE ELF. `fragment.bin` is only 117 bytes and has a very regular pattern:

```text
71 1A 82 42 71 1A 82 42 ... 71 1A 82 42 FF
```

That is 29 repetitions of `71 1A 82 42`, followed by `FF`.

### Reversing the Emulator
Disassembling the binary with `objdump -d -Mintel` shows the main opcode loop around `0x15b0`.

The relevant behavior is:

- `0x71`: append the next input byte to an output deque
- `0x1A`: rotate the last output byte left by 3 bits
- `0x82 <imm>`: XOR the last output byte with the immediate byte
- `0xFF`: stop

Since the fragment is `71 1A 82 42` repeated, each input byte is transformed as:

```text
T(x) = ROL8(x, 3) ^ 0x42
```

The binary then copies 29 bytes from `.rodata` at `0x3123`:

```text
a9 d1 db e1 d9 e3 29 b8 db 31 cb 01 59 e3 29 b8
21 49 e9 e1 d1 09 f1 f1 99 19 69 69 79
```

It compares those bytes against the transformed deque in reverse order. So if `target` is the byte string above:

```text
T(flag[i]) = target[28 - i]
```

Invert the transform:

```text
flag[i] = ROR8(target[28 - i] ^ 0x42, 3)
```

### Reproduction Script
`solution.py` reproduces that inversion and also checks that `fragment.bin` matches the repeated opcode pattern.

Run:

```bash
python3 solution.py
```

Output:

```text
geek{vvirtual_m4ch1n3_m4st3r}
```

## Flag
```text
geek{vvirtual_m4ch1n3_m4st3r}
```
