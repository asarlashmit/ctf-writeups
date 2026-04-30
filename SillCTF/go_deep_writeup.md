# Go Deep Writeup

Challenge: `Go Deep`

Flag: `KCTF{g0_d33p_d0wn}`

## Notes

The provided short URL was protected behind Cloudflare/Turnstile from the terminal environment, so I matched the challenge to the public `KnightCTF 2023 / Go Deep!` artifact set and reproduced the solve locally from the sample image.

## Solve Path

1. Start from the challenge image `sea.jpg`.
2. Run `stegseek` on the JPEG with `rockyou`:

```bash
stegseek sea.jpg rockyou.txt
```

This finds the passphrase `iloveu` and extracts `flag.txt`, which only says:

```text
What you are looking for is not here. Go deeper!
```

3. Run `binwalk` on the JPEG:

```bash
binwalk sea.jpg
```

This reveals an embedded ZIP at offset `0x8D19BF`.

4. Carve the ZIP, unzip it, and inspect the extracted file `deep`.

`deep` claims to be a JPEG, but the bytes immediately after the fake JPEG header contain PNG chunks like `IHDR`, `sRGB`, `gAMA`, and `IDAT`. So this is really a PNG with a forged header.

5. Fix the PNG signature.

The start of the file becomes:

```text
89 50 4E 47 0D 0A 1A 0A 00 00 00 0D 49 48 44 52
```

6. The file still does not render correctly because the `IHDR` width/height are wrong for the stored CRC.

Stored `IHDR` CRC:

```text
0x874442b8
```

Bruteforcing width/height until the `IHDR` CRC matches gives:

```text
width  = 599
height = 416
```

7. Patch those dimensions into the `IHDR` chunk and open the PNG. The bottom-left of the recovered image contains the flag:

```text
KCTF{g0_d33p_d0wn}
```

## Minimal Python For The CRC Fix

```python
from pathlib import Path
from struct import pack
from zlib import crc32

b = bytearray(Path("deep").read_bytes())

# Replace fake JPEG header with a real PNG signature prefix.
b[:10] = b"\x89PNG\r\n\x1a\n\x00\x00"

ihdr = bytearray(b[12:29])
stored = int.from_bytes(b[29:33], "big")

for w in range(1, 2000):
    ihdr[4:8] = pack(">I", w)
    for h in range(1, 2000):
        ihdr[8:12] = pack(">I", h)
        if crc32(ihdr) & 0xffffffff == stored:
            b[16:20] = pack(">I", w)
            b[20:24] = pack(">I", h)
            Path("deep_fixed.png").write_bytes(b)
            print(w, h)
            raise SystemExit
```
