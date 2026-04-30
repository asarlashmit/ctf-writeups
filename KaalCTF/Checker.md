# Checker Writeup

## Challenge

- Name: `Checker`
- Category: `rev`
- Flag format: `Kaal{}`

## Summary

The challenge ships as a Windows `checker.exe`, but it is only a PyInstaller wrapper around a compiled Cython module, `source.pyd`.

The real flag check is inside `source.pyd`. The checker does not derive the flag from user input. Instead, it builds the expected flag by concatenating the return values of hidden helpers `_f0` through `_f12` and compares the result to the submitted input.

The solve path was:

1. Extract the PyInstaller payload.
2. Reverse `source.pyd`.
3. Recover the hidden helper logic.
4. Rebuild Cython's compressed string table and integer table.
5. Concatenate the helper outputs.

## 1. Extracting the Payload

`checker.exe` is a 64-bit Windows PyInstaller binary. After extraction, the important files were:

- `loader.bin`
- `source.pyd`

`loader.bin` only does:

```python
import source

if __name__ == "__main__":
    source.main()
```

So the real logic is in `source.pyd`.

## 2. Locating the Flag Check

Inside `source.pyd`, the `check_flag` routine calls hidden helpers:

- `_f0`
- `_f1`
- `_f2`
- `_f3`
- `_f4`
- `_f5`
- `_f6`
- `_f7`
- `_f8`
- `_f9`
- `_f10`
- `_f11`
- `_f12`

It concatenates their outputs and compares the final string against the provided input. That means the flag is:

```text
_f0 + _f1 + _f2 + _f3 + _f4 + _f5 + _f6 + _f7 + _f8 + _f9 + _f10 + _f11 + _f12
```

## 3. Recovering the Easy Helpers

Some helpers return fixed characters directly:

- `_f0 = "Kaal{"`
- `_f4 = "1"`
- `_f6 = "a"`
- `_f8 = "o"`
- `_f12 = "}"`

## 4. Recovering the Cython String Table

The binary uses Cython's compressed string initialization. During module startup it:

1. Imports `zlib.decompress`
2. Decompresses a blob at `0x180016d50`
3. Rebuilds a string table starting at `0x18001d180`
4. Rebuilds an integer table starting at `0x18001d790`

Decompressing the blob yields the string pool, and the length table at `0x180018590` splits it into 193 entries.

Relevant recovered entries:

- `0x18001d180 -> "0n"`
- `0x18001d1c0 -> ""`
- `0x18001d218 -> "L"`
- `0x18001d220 -> "N"`
- `0x18001d260 -> "also_fake"`
- `0x18001d2b0 -> "c0"`
- `0x18001d4c8 -> "mp"`
- `0x18001d4d8 -> "never_reached"`
- `0x18001d548 -> "pyth"`
- `0x18001d768 -> b"_"` which decodes to `"_"`

Relevant recovered integer entries:

- `0x18001d790 -> 0`
- `0x18001d798 -> -1`
- `0x18001d7a0 -> 1`
- `0x18001d810 -> 105`
- `0x18001d838 -> 116`
- `0x18001d900 -> 999`
- `0x18001d908 -> 1337`

## 5. Solving the Remaining Helpers

Using those reconstructed table entries:

- `_f2 = "_"`
- `_f3 = "" + "c0" + "mp" = "c0mp"`
- `_f5 = "" + "L" = "L"`
- `_f7 = chr(116) + "" + chr(105) = "ti"`
- `_f9 = "N"[::-1] = "N"`
- `_f10 = ""`
- `_f11 = "" + "" = ""`

`_f1` is a fake branch gadget. It computes `1337 + 1 - 1`, then compares the result against:

- `0`
- `999`

Both comparisons fail, so it returns:

- `"pyth" + "0n" = "pyth0n"`

## 6. Final Concatenation

```text
Kaal{ + pyth0n + _ + c0mp + 1 + L + a + ti + o + N + "" + "" + }
```

Which gives:

```text
Kaal{pyth0n_c0mp1LatioN}
```

## Flag

```text
Kaal{pyth0n_c0mp1LatioN}
```
