# Fortnite Dumpy Writeup

Challenge type: `forensics`

## Summary

The short URL led to a SharePoint folder containing a single artifact: `memdump.1960`.

That file was not a full RAM image. It was an ELF core dump from a crashed or dumped `python3` process:

```text
memdump.1960: ELF 64-bit LSB core file, x86-64, version 1 (SYSV), SVR4-style, from 'python3'
```

Inspecting the Python heap in the core revealed a live `bytearray` named `flag_enc` with the following bytes:

```python
b'@\x11OPAkOgufy\x14ykuW@e\x1a\x13@MuIB\x12\x1aUBdOUEr\x1e\x1e'
```

The challenge hint said:

```text
CaseOh is BASEd in Ohio. He is very XORdinary.
```

That points to a two-step decode:

1. XOR the bytes
2. Base-decode the result

XORing every byte with `0x23` produces this Base64 string:

```text
c2lsbHlDVEZ7ZHVtcF90cnVja19vaGlvfQ==
```

Base64-decoding that string yields the flag:

```text
sillyCTF{dump_truck_ohio}
```

## Useful commands

Confirm the artifact type:

```bash
file memdump.1960
```

Check that the core came from Python:

```bash
gdb-multiarch -q -c memdump.1960 -ex 'set pagination off' -ex 'bt' -ex 'quit'
```

Recover and decode the encrypted bytes:

```bash
python3 - <<'PY'
import base64

flag_enc = b'@\x11OPAkOgufy\x14ykuW@e\x1a\x13@MuIB\x12\x1aUBdOUEr\x1e\x1e'
xored = bytes(b ^ 0x23 for b in flag_enc)

print(xored.decode())
print(base64.b64decode(xored).decode())
PY
```

Expected output:

```text
c2lsbHlDVEZ7ZHVtcF90cnVja19vaGlvfQ==
sillyCTF{dump_truck_ohio}
```

## Flag

```text
sillyCTF{dump_truck_ohio}
```
