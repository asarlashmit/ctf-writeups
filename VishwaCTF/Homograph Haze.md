# Homograph Haze

## Type

Misc / stego using Unicode homoglyphs

## Recon

The attachment looks like one long poem line, but some instances of `and` are actually written as `аnd` where the first character is Cyrillic small `а` (`U+0430`) instead of Latin `a` (`U+0061`).

There are 50 repeated sentence blocks, so each block can be treated as one bit:

- Latin `a` in `and` -> `0`
- Cyrillic `а` in `аnd` -> `1`

That produces this bitstream:

```text
01010110011010010111001101101000011101110110000101
```

## Decode

The important detail is that the stream length is exactly 50 bits, which fits 5-bit groups perfectly. So the clean, lossless decode is to split into groups of 5 and map them with the RFC 4648 base32 alphabet:

```text
KZUXG2DXMF
```

An earlier byte-oriented interpretation gives `Vishwa` only by discarding 2 bits at the end of the stream, so it is not the exact hidden text. The exact extracted payload is the 10-character 5-bit string above.

Wrapping that payload in the provided flag format yields the flag:

```text
VishwaCTF{KZUXG2DXMF}
```

## Solver

Run:

```bash
python3 solve.py
```
