# Penetrable Tanish's Corrupted Keys

## Challenge Overview
- Event: CyberGeek CTF
- Category: steganography / Unicode zero-width characters
- Prompt: Investigate the localized density anomaly in `corrupted_keys.txt`.
- Final Flag: geek{sup3rman_is_just_a_guy}

## Executive Summary
The text file is mostly decoy flags. The only meaningful anomaly is one line polluted with zero-width Unicode characters, which encodes the real secret.

## Solution Walkthrough
### Recon
The attachment is a text file containing 10,000 apparent flag candidates:

```bash
file corrupted_keys.txt
wc -l -c corrupted_keys.txt
sed -n '1,5p' corrupted_keys.txt
```

Most lines match the same format:

```text
geek{12_hex_chars}
```

Checking for entries that did not match that pattern revealed the anomaly:

```bash
rg -n -v '^geek\{[0-9a-f]{12}\}$' corrupted_keys.txt
```

Only line 7332 was malformed. It looked like a normal flag candidate, but had many invisible Unicode characters inserted between `geek` and `{127b77c08b3c}`.

### Extraction
Viewing the line as bytes showed repeated UTF-8 sequences:

```bash
sed -n '7332p' corrupted_keys.txt | xxd -g1 -c32
```

The hidden characters were:

- `e2 80 8b`: U+200B zero-width space
- `e2 80 8c`: U+200C zero-width non-joiner

There were 224 zero-width characters, which is exactly 28 bytes of binary data. Mapping U+200B to `0` and U+200C to `1` decoded cleanly as ASCII:

```bash
python3 - <<'PY'
from pathlib import Path

line = Path('corrupted_keys.txt').read_text(encoding='utf-8').splitlines()[7331]
bits = ''.join(
    '0' if ch == '\u200b' else
    '1' if ch == '\u200c' else
    ''
    for ch in line
)
flag = ''.join(chr(int(bits[i:i+8], 2)) for i in range(0, len(bits), 8))
print(flag)
PY
```

Output:

```text
geek{sup3rman_is_just_a_guy}
```

## Flag
```text
geek{sup3rman_is_just_a_guy}
```
