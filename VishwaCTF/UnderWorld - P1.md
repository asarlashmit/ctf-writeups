# UnderWorld - P1 Writeup

Challenge type: forensics / Minecraft world analysis

## Summary

The provided ZIP extracts to a Minecraft world save named `UnderWorld`. Most of the world is a flat custom map, but only a small area in `extract/UnderWorld/region/r.0.-1.mca` contains non-default blocks and block entities. By parsing the modern MCA chunk format directly and enumerating block entities, the flag appears in a chest whose diamonds are individually renamed to one character each.

## Recon

1. Download and extract the challenge archive.
2. Inspect `level.dat` and player data.
3. Scan the world for interesting chunks by checking each chunk palette and `block_entities`.

The custom structure is concentrated around chunk coordinates:

- `x = 1..9`
- `z = -12..-8`

Interesting findings inside that area:

- A decoy barrel spelling `No Emeralds here`
- A hanging sign with `Chall 2: Emreald Blocks are Beautiful`
- A chest at `(87, 41, -116)` containing renamed diamonds

## Extraction

The chest at `(87, 41, -116)` stores 27 diamonds with custom names. Reading them in slot order yields:

`VishwaCTF{m1n3cr4f7_15_fun}`

## Reproduction

Create a virtualenv and install the parser:

```bash
python3 -m venv .venv
.venv/bin/pip install nbtlib
```

Run the extractor:

```bash
.venv/bin/python solve.py
```

Expected output:

```text
VishwaCTF{m1n3cr4f7_15_fun} @ (87, 41, -116)
```

## Flag

`VishwaCTF{m1n3cr4f7_15_fun}`
