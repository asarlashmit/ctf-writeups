# Fractured Frame

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics
- Final Flag: geek{h34d3r_r3p41r_m4st3r_2026}

## Executive Summary
The provided `asset.png` was a valid PNG with a corrupted file signature. After repairing the PNG header, the image metadata exposed a base64-encoded hint. The file also had an encrypted ZIP archive appended after the PNG data. The decoded metadata hint was the ZIP password, which extracted `flag.txt`.

## Solution Walkthrough
### Recon
`file` showed the asset was not recognized as a PNG:

```bash
file asset.png
```

Output:

```text
asset.png: Zip archive, with extra data prepended
```

The first bytes showed a damaged PNG signature, while the PNG chunk structure was still present:

```bash
xxd -g1 -l 128 asset.png
```

Relevant bytes:

```text
00000000: 49 49 49 54 41 0a 1a 0a 00 00 00 0d 49 48 44 52
```

A normal PNG starts with:

```text
89 50 4e 47 0d 0a 1a 0a
```

`binwalk` also found an appended encrypted ZIP archive:

```bash
binwalk asset.png
```

Relevant output:

```text
1401854  0x1563FE  Zip archive data, encrypted ..., name: flag.txt
1402058  0x1564CA  End of Zip archive
```

### Header Repair
The first 8 bytes were replaced with the correct PNG signature:

```bash
python3 - <<'PY'
from pathlib import Path

data = bytearray(Path("asset.png").read_bytes())
data[:8] = b"\x89PNG\r\n\x1a\n"
Path("asset_fixed.png").write_bytes(data)
PY
```

After repair:

```bash
file asset_fixed.png
```

Output:

```text
asset_fixed.png: PNG image data, 1263 x 697, 8-bit/color RGB, non-interlaced
```

### Password Hint
`exiftool` showed a PNG comment:

```bash
exiftool asset_fixed.png
```

Relevant output:

```text
Comment : Aurora_System_ID: VEhFX05vcnRo
```

The value was base64:

```bash
python3 - <<'PY'
import base64
print(base64.b64decode("VEhFX05vcnRo").decode())
PY
```

Output:

```text
THE_North
```

This matched the aurora/northern-lights theme in the repaired image and was used as the ZIP password.

### Extraction
```bash
mkdir -p extract_try
unzip -P THE_North asset.png -d extract_try
cat extract_try/flag.txt
```

Output:

```text
geek{h34d3r_r3p41r_m4st3r_2026}
```

## Flag
```text
geek{h34d3r_r3p41r_m4st3r_2026}
```
