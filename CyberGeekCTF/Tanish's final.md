# Tanish's final

## Challenge Overview
- Event: CyberGeek CTF
- Category: forensics
- Final Flag: geek{sup3rman_vs_6atman}

## Executive Summary
The provided `server_backup.bin` was not encrypted. It was a shuffled stream of variable-length records:

- `u32_be chunk_index`
- `u32_be chunk_length`
- `chunk_length` bytes of payload

Most records were `512` bytes long, and the final chunk was `195` bytes. Parsing the file sequentially produced exactly `19235` unique chunks with indexes `0..19234`. Sorting by `chunk_index` and concatenating the payloads reconstructed the original file.

## Solution Walkthrough
### Recon
Initial checks showed the blob was generic `data`, but `binwalk`/`strings` hinted at image metadata:

```bash
file server_backup.bin
binwalk server_backup.bin
strings -n 8 server_backup.bin | head
```

Looking at the start of the file showed a repeating structure:

```text
00000000: 00001514 00000200 ...
00000208: 00000a2b 00000200 ...
00000410: 00000da1 00000200 ...
```

That pattern is consistent with `index | length | payload`.

### Recovery
This script rebuilds the original image:

```python
from pathlib import Path

b = Path("server_backup.bin").read_bytes()
off = 0
records = []

while off + 8 <= len(b):
    idx = int.from_bytes(b[off:off+4], "big")
    ln = int.from_bytes(b[off+4:off+8], "big")
    off += 8
    records.append((idx, b[off:off+ln]))
    off += ln

records.sort(key=lambda x: x[0])
Path("recovered.bin").write_bytes(b"".join(chunk for _, chunk in records))
```

After reconstruction:

```bash
exiftool recovered.bin
```

Result:

- File type: `PNG`
- Dimensions: `2816x1536`

Opening the recovered image reveals the flag directly on the screen.

## Flag
```text
geek{sup3rman_vs_6atman}
```
