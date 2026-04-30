# pwn/vkexchange

## Flag

`UMDCTF{yeah_im_sorry_for_making_this_i_know_its_really_annoying_but_at_least_maybe_you_learned_vulkan}`

## Summary

The challenge exposes a Vulkan descriptor-set OOB write through:

```c
update_storage_desc(app, app->quote_book, 0, (uint32_t)idx, b->buf, off, range);
```

`quote_book` binding `0` only has one storage-buffer descriptor, but `price_index` is allowed up to `300000`.

On the challenge runtime (Mesa lavapipe 22.3.6), `lvp_UpdateDescriptorSets()` does not bounds-check `dstArrayElement`; it just does:

```c
desc += write->dstArrayElement;
```

That gives an out-of-bounds descriptor write into later descriptor-set allocations.

## Exploit idea

1. Create one account buffer we can later audit.
2. Create one market with `32768` outcome slots and `memo_bytes = 8`.
3. Open the exchange, which allocates `quote_book`, the market set, and `settlement_book` consecutively.
4. Use `quote_position(price_index=32778, account=0, offset=0, range=256)`.

With this geometry, the OOB write lands on `settlement_book` binding `1`, replacing the clearing buffer descriptor with our account buffer.

The settlement shader does:

```glsl
clearing_words[push.index] = oracle_words[push.index];
```

`oracle_buf` contains the flag, so each `settle_market(round)` copies one 32-bit word of the flag into our audited account buffer.

5. Call `settle_market(round)` for enough rounds.
6. `audit_account()` the account buffer and decode the hex string.

## Minimal solve sequence

```text
1
256
4
32768
8
5
6
32778
0
0
256
7
0
7
1
...
3
0
0
256
```

## Notes

- `memo_bytes = 8` is the alignment fix that makes the overwrite land on the right settlement descriptor.
- `solve.py` automates the full exploit against the remote service.
