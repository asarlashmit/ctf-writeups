# TinyLang Writeup

## Flag

`V15hw4CTF{cu570m_14ngu4g3_f4113d_52482b70}`

## Summary

The interpreter stores variables in a global table in `.bss`, but every `let`
copies 64 bytes into a slot while advancing the index by only 20 bytes. The
variable count is also unchecked. After a few inserts this overwrites adjacent
globals, including the function pointer used when `print` fails to find a
variable.

That gives an arbitrary indirect call with `rdi` pointing at attacker-controlled
input from `print <...>`.

## Reversing Notes

From the provided ELF:

- `main` leaks the PIE base with `dladdr`.
- Variables start at `.bss + 0x20` (`0x40a0` in the local binary).
- Each entry advances by `0x14` bytes, but the code copies four 16-byte chunks
  for a total of 64 bytes.
- The variable count lives at `0x4140`.
- Two function pointers live at `0x4148` and `0x4150`.
- On `print <name>`, if the variable is missing, the binary jumps through the
  pointer at `0x4150`.

Locally, the useful wrappers were:

- `base + 0x12a0`: `printf("Error: %s\n", input)`
- `base + 0x12c0`: `jmp system@plt`

The remote deployment was offset-shifted by `+0x10`, so the live service used:

- `base + 0x12b0`: error wrapper
- `base + 0x12d0`: `jmp system@plt`

I identified that shift by blackbox-testing nearby offsets against the remote
service after the local exploit stopped matching exactly.

## Exploit

The first 5 `let` statements are fillers.

The 6th `let` reaches the count field at `0x4140`. Its payload sets the
overwritten count back to `5`, so after the function increments it, the next
index stays at `6`.

The 7th `let` reaches both function pointers at `0x4148` and `0x4150`. I write
the remote `system` thunk (`base + 0x12d0`) into both locations.

After that, any missing variable triggers:

`print <shell command>`

which becomes:

`system("<shell command>")`

## Flag Extraction

Direct filesystem enumeration did not show a normal flag file. A readable core
dump at `/app/core` did contain the flag, so the reproducible solve flow is:

1. Use the `system` thunk to remove any stale `/app/core`.
2. Start a new session and point the handler at the crashing thunk
   (`base + 0x12c0`) to regenerate a fresh core dump.
3. Start another `system` session and run:

```sh
grep -a -o 'V15hw4CTF{[^}]*}' /app/core 2>/dev/null | head -n1
```

That returns:

`V15hw4CTF{cu570m_14ngu4g3_f4113d_52482b70}`

## Files

- `solve.py`: automated remote solve script
- `writeup.md`: this writeup
