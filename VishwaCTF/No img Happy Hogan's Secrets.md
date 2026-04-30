# No img Happy Hogan's Secrets

## Challenge Type

Stego / forensics

## Files

- `Recognize.jpg`
- `Protected_audio.zip`

## Solve Path

### 1. Recon the image

`binwalk Recognize.jpg` showed an appended ZIP:

```bash
binwalk Recognize.jpg
```

Important hit:

```text
67542         0x107D6         Zip archive data, name: note.txt
```

Extracting the hidden file from the JPEG:

```bash
unzip -p Recognize.jpg note.txt
```

Output:

```text
Happy Hogan: 555449316256707463476c6157455539
"It's not that complicated."
```

### 2. Decode the password clue

The string is hex, then base64:

```bash
python3 - <<'PY'
import base64
s='555449316256707463476c6157455539'
print(bytes.fromhex(s).decode())
print(base64.b64decode(bytes.fromhex(s)).decode())
PY
```

This gives:

```text
UTI1bVptcGlaWEU9
Q25mZmpiZXE=
```

Decode base64 again:

```bash
python3 - <<'PY'
import base64
print(base64.b64decode('Q25mZmpiZXE=').decode())
PY
```

Output:

```text
Cnffjbeq
```

`Cnffjbeq` is ROT13 for `Password`.

So the ZIP password is:

```text
Password
```

### 3. Extract the protected archive

```bash
7z x -pPassword Protected_audio.zip -oextracted
```

This yields:

- `extracted/output2.mp3`
- `extracted/decoy.txt`

`decoy.txt` contains a fake flag, so ignore it.

### 4. Inspect the audio metadata

```bash
ffprobe -hide_banner extracted/output2.mp3
```

Relevant metadata:

```text
description : 53585a6d645770755545645465325a6e4d33517758334130595639764d31387a626a567366513d3d
```

Decode it:

```bash
python3 - <<'PY'
import base64
hexs='53585a6d645770755545645465325a6e4d33517758334130595639764d31387a626a567366513d3d'
s=bytes.fromhex(hexs).decode()
print(s)
print(base64.b64decode(s).decode())
PY
```

Output:

```text
SXZmdWpuUEdTe2ZnM3QwX3A0YV9vM18zbjVsfQ==
IvfujnPGS{fg3t0_p4a_o3_3n5l}
```

Apply ROT13:

```bash
python3 - <<'PY'
import codecs
print(codecs.decode('IvfujnPGS{fg3t0_p4a_o3_3n5l}', 'rot_13'))
PY
```

Final flag:

```text
VishwaCTF{st3g0_c4n_b3_3a5y}
```

## Notes

- The MP3 also has appended junk/ZIP data near the tail.
- That tail contains a near-duplicate `SECRET_FLAG` string that decodes to a typo-like variant.
- The cleanest and most consistent flag comes from the ID3 `description` field and matches the intended stego theme.

## Flag

`VishwaCTF{st3g0_c4n_b3_3a5y}`
