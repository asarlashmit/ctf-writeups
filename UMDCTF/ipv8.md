# ipv8

## Flag

`UMDCTF{why_was_ipv9_afraid_of_ipv7?ipv789}`

## Summary

The binary is a 64-bit static ELF with a `win()` function at `0x402f45` that calls `system("/bin/sh")`.

The intended checks are split across four packet fields:

- source ASN prefix: discarded with `scanf("%*s")`
- source host address: read with unsafe `scanf("%s")` into `[rbp-0x60]`
- destination ASN prefix: discarded with `scanf("%*s")`
- destination host address: read with `scanf("%48s")` into `[rbp-0xc0]`

Two bugs make the challenge exploitable:

- The source host read is an unrestricted stack overflow.
- The destination host read is a 48-byte token into a 48-byte buffer, so the terminating NUL lands on `[rbp-0x90]`, the field later checked by `check_rine()`.

## Reversing Notes

Useful symbols were present:

- `win`
- `check_valid_address`
- `check_rine`
- `main`

`check_valid_address()` only counts dots, so any string with exactly three `.` characters is accepted.

`check_rine()` compares the string at `[rbp-0x90]` with `strcmp()`:

1. If it equals `0.0.0.0`, the program prints `Sorry, we want devices using ipv8 only...` and exits.
2. If it equals `100.72.7.67`, it prints the success message and calls `win()`.
3. Otherwise it prints `Wrong RINE address!! ...` and returns to `main`.

That means we do not need to satisfy the RINE check. We only need to make sure it does not call `exit()`.

## Exploit

### Step 1

Overflow the source host buffer and overwrite `main`'s saved return address with a small chain:

- `ret` gadget at `0x403244`
- `win` at `0x402f45`

The extra `ret` fixes stack alignment before `win()` calls `system()`.

The source field still has to pass validation, so the payload starts with:

`1.2.3.4\x00`

`scanf("%s")` accepts embedded NUL bytes from the socket, while `check_valid_address()` stops at the first NUL and sees a valid dotted address.

### Step 2

Use the destination host field to force `[rbp-0x90]` away from `0.0.0.0`.

Sending exactly 48 bytes causes `scanf("%48s")` to write the trailing NUL one byte past the buffer, at `[rbp-0x90]`. That turns the checked string into an empty string, so:

- first `strcmp(..., "0.0.0.0")` is nonzero
- second `strcmp(..., "100.72.7.67")` is also nonzero

The program prints the wrong-RINE message and returns normally, which triggers the overwritten return address and drops to a shell.

### Step 3

Run:

```bash
python3 solve.py
```

The exploit sends:

- discarded source ASN
- overflowing source host payload
- discarded destination ASN
- 48-byte destination host payload
- `cat /app/flag.txt`

The remote shell returns:

`UMDCTF{why_was_ipv9_afraid_of_ipv7?ipv789}`
