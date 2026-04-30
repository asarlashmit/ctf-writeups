# Sexy Srijjy

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics / Image Reconstruction
- Prompt: Broke my heart now keep arranging shards
- Final Flag: geek{🍰💜}

## Executive Summary
Only sixteen image tiles contain real data. Reassembling those non-white shards reveals the flag directly as a mosaic.

## Solution Walkthrough
### Recon
The fresh archive contains 256 files under `tiles_16x16/`:

- all files are BMP images
- each tile is `80x73`
- filenames are emoji strings with 10 unique symbols and lengths `1..3`
- almost every tile is pure white

Useful quick checks:

```bash
unzip -l latest_lol.zip
python3 - <<'PY'
import zipfile, io, collections
from PIL import Image
import numpy as np

zf = zipfile.ZipFile("latest_lol.zip")
names = [n for n in zf.namelist() if not n.endswith("/")]
print("tile count:", len(names))

nonwhite = []
for name in names:
    img = Image.open(io.BytesIO(zf.read(name))).convert("RGB")
    arr = np.array(img)
    count = ((arr < 250).any(axis=2)).sum()
    if count:
        nonwhite.append((name.split("/")[-1], int(count)))

print("non-white tiles:", len(nonwhite))
print(nonwhite)
PY
```

That shows only 16 tiles contain any visible data. The rest are solid white.

### Solving
The intended task is to reassemble only those 16 informative shards. Since the
background is plain white, the easiest route is:

1. Parse the zip directly with Python instead of extracting emoji filenames.
2. Split out the 16 non-white tiles.
3. Match top and bottom fragments by horizontal ink profile.
4. Order the resulting columns by left/right edge similarity.

The active shards reconstruct into a stylized rendering of the flag itself:

```text
geek{🍰💜}
```

The two emojis are literal:

- `🍰` = slice of cake
- `💜` = purple heart

## Flag
```text
geek{🍰💜}
```
