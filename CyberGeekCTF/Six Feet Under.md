# Six Feet Under

## Challenge Overview
- Event: CyberGeek CTF
- Final Flag: geek{smart_iceberg}

## Executive Summary
The visible caption says:

> There are no secrets in life. Just hidden truths that lie beneath the surface.

Taking that literally, the hidden data was below the visible surface of the
image. The JPEG contained more compressed image rows than the displayed height
allowed.

## Solution Walkthrough
### Recon
```bash
file i_love_this_show.jpeg
exiftool i_love_this_show.jpeg
identify -verbose i_love_this_show.jpeg
```

Important findings:

- The real JPEG/SOF dimensions were `1138x938`.
- EXIF claimed the image should be `1138x1102`.
- Image decoders warned about extraneous bytes before JPEG restart markers.

The EXIF height was larger than the visible JPEG height, which suggested that
additional rows were present but hidden by a smaller SOF height field.

### Exploit
The JPEG SOF0 marker stored height `938` (`0x03aa`). Changing it to the EXIF
height `1102` (`0x044e`) revealed the hidden lower part of the image.

```bash
python3 - <<'PY'
from pathlib import Path

b = bytearray(Path("i_love_this_show.jpeg").read_bytes())
sof = b.index(b"\xff\xc0")
height_off = sof + 5

print("old height:", int.from_bytes(b[height_off:height_off+2], "big"))
b[height_off:height_off+2] = (1102).to_bytes(2, "big")

Path("revealed_height_1102.jpeg").write_bytes(b)
PY
```

Opening `revealed_height_1102.jpeg` shows the buried bottom caption:

```text
You're close! keep going
geek{smart_iceberg}
```

## Flag
```text
geek{smart_iceberg}
```
