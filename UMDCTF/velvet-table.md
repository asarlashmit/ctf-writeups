# velvet-table

Flag: `UMDCTF{smallbins_still_love_the_stack_when_the_house_sets_the_table}`

## Summary

This binary is a heap challenge with a protected payout function on the stack.

The important pieces are:

- `cashout` frees a chunk but leaves stale pointers around in several cases.
- `settle-ledger` changes `update` from a partial every-other-byte write into a full unchecked `memcpy`.
- The stack already contains a fake `0x110` chunk header at `rsp+0x90`.
- `payout` calls a function pointer stored on the stack after checking a simple XOR-based integrity value.

The exploit is:

1. Build enough ledger score to unlock `settle-ledger`.
2. Allocate three `0x100` chunks and one filler chunk.
3. Free two `0x100` chunks so the tcache count for that size is `2`.
4. Overflow from the first live `0x100` chunk into the freed head chunk and poison its `next` pointer to the fake stack chunk.
5. Allocate twice to get a seat whose pointer is the fake stack chunk on `main`'s stack.
6. Use `inspect` to leak the current payout function pointer and its checksum.
7. Patch the function pointer from the local reject stub to the nearby `system("/bin/sh")` stub and update the checksum to match.
8. Trigger `payout` and read the flag from the spawned shell.

## Reversing Notes

The menu handlers are all in `main`.

### Seat table

There are 16 global seat entries at `.bss:0x50a0`, each laid out as:

- `ptr`
- `size`
- `active`

User seat numbers are remapped through:

```c
idx = ((seat ^ marker_nibble) + 3) & 0xf;
```

The `table marker` printed at startup leaks enough stack information to recover `marker_nibble` and the low 32 bits of `((rsp + 0x90) >> 4)`.

### `cashout`

`cashout` does:

```c
free(ptr);
active = 0;
```

For `0x100` and `0x180` requests it sometimes clears the pointer too, but only to mirror an internal count. This matters because we can still work with freed `0x100` chunks by overflowing into them from an adjacent live chunk after settlement.

### `update`

Before settlement, `update` only copies every other byte.

After `settle-ledger`, it becomes:

```c
read(tmp, len);
memcpy(chunk, tmp, len);
```

with `len` allowed up to `0x400`, regardless of the chunk size. That gives a direct heap overflow.

### Fake stack chunk

`main` initializes this structure on the stack:

```c
fake.prev_size = 0;
fake.size      = 0x111;
fake.user[0]   = 0;
fake.user[1]   = 0;
```

This is exactly what is needed for a fake `0x110` tcache entry.

The full fake user pointer on a real x86_64 Linux target is:

```text
fake_user = ((((0x7ff << 32) | rbx_low32) << 4) + 0x10)
```

where:

```text
rbx_low32 = table_marker ^ 0x9ac90307
```

The upper `0x7ff` works because the lost top three hex digits of a normal x86_64 userspace stack address are constant.

### `inspect` leak

`inspect` does not print raw bytes. It XORs each byte with a reversible per-byte mask derived from the startup seed and the mapped seat index. Once that is reversed, the fake stack chunk leaks:

- offset `0x10`: current payout function pointer
- offset `0x18`: current checksum

## Exploit Details

Use this heap layout:

- `A = malloc(0x100)`
- `B = malloc(0x100)`
- `C = malloc(0x100)`
- `D = malloc(0x80)`

Then:

1. `settle-ledger`
2. `free(C)`
3. `free(B)`

Now the `0x110` tcache bin has count `2` and head `B`.

Overflow `A` into `B` and write:

- preserved `B` chunk header
- poisoned `B->next = fake_user ^ (B_user >> 12)`

Allocating `0x100` twice gives:

- first allocation: returns `B`
- second allocation: returns the fake stack chunk

From there:

1. `inspect` the fake chunk seat
2. decrypt the leak
3. compute:

```text
new_func = old_func + 0x10
new_chk  = old_chk ^ old_func ^ new_func
```

`old_func + 0x10` changes the pointer from the reject stub at `0x1b50` to the shell stub at `0x1b60`.

Finally call `payout`, get `yay.`, and use the spawned shell to read `/app/flag.txt`.

## Result

`UMDCTF{smallbins_still_love_the_stack_when_the_house_sets_the_table}`
