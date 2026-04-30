# Peeedeeeffffff

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics / PDF Steganography
- Final Flag: geek{I_l0ve_PDF}

## Executive Summary
The PDF contains several decoys:

- A JavaScript object with fake credentials and two fake flags.
- A fake embedded SQLite file `data.db`.
- A compressed object that expands to `geek{wr0ng_l4y3r_wr0ng_4nsw3r}`.

The real path is hinted by the PDF metadata and embedded PNG note:

- `Subject: ghost objects present`
- `key.png` contains `real_key=XR3_m9!`

The real flag is split across three orphan objects:

1. Object `21`: Flate-compressed, then XORed with `XR3_m9!`
2. Object `34`: base64
3. Object `55`: hex, then reversed

Combining those three decoded chunks reconstructs the hidden JSON:

```json
{"users":[{"u":"admin","p":"geek{I_l0ve_PDF}"},{"u":"analyst","p":"rotating-secret"}],"version":7,"engine":"pdfbks"}
```

## Solution Walkthrough
### Recon
Useful commands:

```bash
file Archive_me_in_your_Heart.pdf
exiftool Archive_me_in_your_Heart.pdf
strings -n 6 Archive_me_in_your_Heart.pdf | head -n 200
mutool extract Archive_me_in_your_Heart.pdf
mutool show Archive_me_in_your_Heart.pdf xref
mutool show -b Archive_me_in_your_Heart.pdf 21 | xxd -g 1
mutool show -b Archive_me_in_your_Heart.pdf 34 | xxd -g 1
mutool show -b Archive_me_in_your_Heart.pdf 55 | xxd -g 1
```

Key findings:

- `Producer: pdfbks-insane-builder`
- `Subject: ghost objects present`
- Embedded files extracted by `mutool`:
  - `data.db` fake
  - `key.png` with `real_key=XR3_m9!`

### Reproduction Script
```python
import re, base64, binascii, zlib
from itertools import cycle

pdf = open("Archive_me_in_your_Heart.pdf", "rb").read()

def stream(obj_num):
    m = re.search(
        rb"\n%d 0 obj\s*<<.*?>>\s*stream\n(.*?)\nendstream" % obj_num,
        pdf,
        re.S,
    )
    return m.group(1)

key = b"XR3_m9!"

chunk1 = zlib.decompress(stream(21))
chunk1 = bytes(a ^ b for a, b in zip(chunk1, cycle(key)))

chunk2 = base64.b64decode(stream(34) + b"=" * ((4 - len(stream(34)) % 4) % 4))
chunk3 = binascii.unhexlify(stream(55))[::-1]

blob = (chunk1 + chunk2 + chunk3).decode()
print(blob)
```

Running it prints:

```text
{"users":[{"u":"admin","p":"geek{I_l0ve_PDF}"},{"u":"analyst","p":"rotating-secret"}],"version":7,"engine":"pdfbks"}
```

The admin password is the real flag.

## Flag
```text
geek{I_l0ve_PDF}
```
