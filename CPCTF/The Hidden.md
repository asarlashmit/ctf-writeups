# The Hidden Writeup

## Challenge

- Name: `The Hidden`
- Type: `rev`
- Artifact: `hidden` (ELF 64-bit PIE)
- Expected flag format: `CPCTF{}`

## Summary

This challenge is a minimal Linux ELF binary that does not compute or validate anything interesting at runtime. It only prints a hint message, while the actual flag is stored directly in the binary's `.rodata` section.

The shortest solve path was:

1. Download the ZIP.
2. Extract the binary.
3. Run `strings` on the ELF.
4. Confirm the candidate flag in `.rodata`.

## Recon

After extracting the archive:

```bash
file hidden
```

This identifies the file as a stripped 64-bit ELF executable.

Running the binary only prints:

```text
The flag is hidden in this file. Can you find it?
```

That suggests the flag may be embedded statically rather than generated dynamically.

## Extracting the Flag

Using `strings` immediately reveals a candidate:

```bash
strings -a -n 4 hidden
```

Relevant output:

```text
CPCTF{H1dd3n_1n_5tr1ngs}
The flag is hidden in this file. Can you find it?
```

To verify that this was not a decoy, inspecting `.rodata` shows the same flag bytes stored directly in the binary:

```bash
objdump -s -j .rodata hidden
```

Relevant portion:

```text
2010 43504354 467b4831 6464336e 5f316e5f
2020 35747231 6e67737d
```

Decoded ASCII:

```text
CPCTF{H1dd3n_1n_5tr1ngs}
```

## Conclusion

The binary is a simple hiding exercise. The program output is just a hint; the flag is embedded as a plain string inside the executable.

## Flag

`CPCTF{H1dd3n_1n_5tr1ngs}`
