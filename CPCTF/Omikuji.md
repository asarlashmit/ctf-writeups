# Omikuji Writeup

Type: `rev`

## Summary

The challenge binary is a 64-bit C++ ELF named `omikuji`. It does not need to be executed to recover the flag.

`main` constructs the full flag one character at a time using repeated calls to `std::string::operator+=` and then passes that string into `omikuji(std::string)`.

## Recon

Downloaded the file and confirmed it is a non-stripped PIE ELF:

```bash
curl -sS https://files.cpctf.space/omikuji/omikuji -o omikuji
file omikuji
nm -C omikuji
objdump -d -Mintel --disassemble=main omikuji
objdump -d -Mintel --disassemble=_Z7omikujiNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE omikuji
```

## Key Observation

In `main`, the program appends these bytes as characters:

```text
C P C T F { D 3 r _ 4 1 7 3 _ w u r f 3 1 7 _ n 1 c h 7 }
```

That directly reconstructs the flag:

```text
CPCTF{D3r_4173_wurf317_n1ch7}
```

## What `omikuji` Does

`omikuji(std::string)` seeds an `mt19937` with `std::random_device`, generates a random integer, and behaves as follows:

- If the random value is exactly `0x7ea` (`2026`), it prints the full flag.
- Otherwise it computes `rand % 100` and only prints a prefix of the flag.
- Depending on the range, it reveals 10, 8, 6, 4, or 2 characters.
- In the worst case it prints no flag data at all.

So the “lucky” path is just a distraction. The complete flag is already embedded in `main`.

## Flag

```text
CPCTF{D3r_4173_wurf317_n1ch7}
```
