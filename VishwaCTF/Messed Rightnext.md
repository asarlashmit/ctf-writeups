# Messed Rightnext

## Summary

The binary is a decoy verifier. Any positive-length input goes straight to the `invalid` path, and EOF goes to `Byy!`. There is no live comparison stage at all.

The useful part is the stack setup:

- It writes the SHA-256 IV / round constants onto the stack.
- It writes the hex alphabet `0123456789abcdef`.
- After `read`, it zeroes an 8-qword block, copies 6 fixed bytes into it, and writes `0x80` immediately after them.

That is exactly the start of a SHA-256 padded message block. The copied bytes are:

```text
81 62 43 34 63 77
```

So the missing “next stage” is the hash stage. The intended secret is the SHA-256 of those 6 bytes.

## Recovering The Bytes

I used a Unicorn-based emulator to run the x86-64 ELF on the ARM host and trace stack writes. The relevant writes were:

```text
off 0xd8..0xdd = 81 62 43 34 63 77
off 0xde       = 80
```

The binary also materializes the standard SHA-256 constants, which confirms the intended operation.

## Solve

```python
import hashlib
payload = bytes.fromhex("816243346377")
digest = hashlib.sha256(payload).hexdigest()
print(f"VishwaCTF{{{digest}}}")
```

## Flag

```text
VishwaCTF{21f4ed9eb0b3db54a0398742eabbb28f439bfde724c26e4c157899e3cf83ce04}
```
