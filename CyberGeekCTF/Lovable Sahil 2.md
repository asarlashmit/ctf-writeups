# Lovable Sahil 2

## Challenge Overview
- Event: CyberGeek CTF
- Category: Reverse Engineering / VM
- Final Flag: geek{n0_m0r3_AI}

## Executive Summary
The stripped ELF hides its flag check inside a tiny bytecode VM. Decoding that VM program exposes the byte-wise constraints needed to reconstruct the accepted flag.

## Solution Walkthrough
### VM Decode
Each opcode byte is decoded with:

```text
decoded = bytecode[pc] ^ pc ^ key
```

The useful instructions are:

```text
0x10 imm  set input index
0x11      load input[index] into acc
0x12      load input[index + 1] into tmp
0x20      acc ^= tmp
0x21 imm  acc += imm
0x30 imm  compare acc == imm
0x40      key = ((key * 2) ^ 0x1b) & 0xff
0x50      assert previous compare succeeded
0xff      success/end
```

Decoding the bytecode gives repeated checks of this form:

```text
(input[i] ^ input[i + 1]) + 0x0a == target[i]
```

The decoded compare targets are:

```text
0c 0a 18 1a 1f 68 79 3c 67 4c 4b 76 28 12 3e
```

So the adjacent XOR values are:

```text
02 00 0e 10 15 5e 6f 32 5d 42 41 6c 1e 08 34
```

Using the known flag prefix `geek{` fixes the only free byte and reconstructs:

```text
geek{n0_m0r3_AI}
```

### Verification
```bash
printf '%s\n' 'geek{n0_m0r3_AI}' | ./chall_aLyMUHu
```

Output:

```text
Enter Flag: Correct! You survived the VM.
```

## Flag
```text
geek{n0_m0r3_AI}
```
