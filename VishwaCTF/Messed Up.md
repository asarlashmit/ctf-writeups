# Messed Up Writeup

Challenge: `Messed Up`  
Category: `rev`

## Summary

The handout contains a static x86-64 ELF called `MessedUp`. The binary is heavily obfuscated and implements a state-machine style control flow with many arithmetic blocks and computed jumps.

The program reads input, echoes it back, and always prints that the secret is invalid. The input is not part of the real validation logic. The flag is hidden in constants that are XOR-decoded on the stack.

## Recon

Basic inspection:

```bash
file MessedUp
readelf -h -l -s MessedUp
objdump -d -M intel MessedUp | less
```

Key observations:

- Static 64-bit ELF
- Not stripped enough to be pleasant, but still highly obfuscated
- Runs under `qemu-x86_64` on an aarch64 host
- The visible runtime behavior is:

```text
Enter Secret:
Your Secret "<input>" is invalid!
Byy!
```

Tracing with:

```bash
printf 'test\n' | qemu-x86_64 -strace ./MessedUp
```

showed that the program:

- writes the prompt in pieces
- reads up to 127 bytes into a stack buffer
- writes back the user input
- always prints the invalid message

So the input path is a decoy.

## Dynamic Analysis

Because the control flow is intentionally messy, I switched from static reversing to emulation and trace collection.

I wrote a small Unicorn-based helper, [`emu.py`](/home/kali/ctf-orc/ctf/messed-up/emu.py), to:

- map the ELF load segments
- emulate the stack
- hook `read`, `write`, and `exit`
- log stack writes near `rsp`
- dump the final stack state

Running:

```bash
./.venv/bin/python emu.py > emu.out
```

made the hidden pattern obvious: the binary stores ciphertext qwords at negative stack offsets and matching key qwords exactly `0x40` bytes later. XORing each pair reveals plaintext.

Recovered pairs:

```text
-0x48 ^ -0x08 -> "Enter Se"
-0x40 ^ +0x00 -> "cret: "
-0x38 ^ +0x08 -> "VishwaCT"
-0x30 ^ +0x10 -> "F{50rry_"
-0x28 ^ +0x18 -> "17_w4s_r"
-0x20 ^ +0x20 -> "3411y_m3"
-0x18 ^ +0x28 -> "s53d_UP!"
-0x10 ^ +0x30 -> "!!}"
```

The first two chunks build the prompt. The remaining six chunks build the flag.

## Flag

```text
VishwaCTF{50rry_17_w4s_r3411y_m3s53d_UP!!!}
```
