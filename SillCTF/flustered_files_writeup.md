# Flustered Files

Type: Chaos Theory / Forensics

## Flag

`sillyCTF{w@$h_my_b3ll@y}`

## Summary

The short URL served a small archive, `files.zip`, containing a single file named `kittycat.jpg`.

That JPEG was not just an image. It was a JPEG/ZIP polyglot: opening the picture showed a cat plus the message:

```text
IT'S DANGEROUS TO GO ALONE! TAKE THIS.
```

Extracting the hidden archive from the JPEG produced two more files:

- `beautiful_sunset.png`
- `print_flag`

`print_flag` looked like an ELF, but it was intentionally broken. Its tail was actually raw image data. Interpreting the trailing `30000` bytes as a `100 x 100 x 3` RGB image produced a clean sunset, while `beautiful_sunset.png` was the noisy/glitched version of the same scene. Working from those paired images and isolating the overlaid text revealed the flag.

## Recon

Initial extraction:

```bash
unzip files.zip -d extracted_flustered
file extracted_flustered/kittycat.jpg
```

Output:

```text
extracted_flustered/kittycat.jpg: JPEG image data, JFIF standard 1.01, ...
```

Viewing the image showed the cat and the hint to inspect it further.

## Hidden Archive

The `.jpg` had appended ZIP data, so it could be carved directly:

```bash
binwalk -e extracted_flustered/kittycat.jpg
```

That yielded the nested files:

```text
beautiful_sunset.png
print_flag
```

Basic identification:

```bash
file nested/beautiful_sunset.png nested/print_flag
```

Output:

```text
nested/beautiful_sunset.png: PNG image data, 100 x 100, 8-bit/color RGB
nested/print_flag: ELF 64-bit LSB shared object, x86-64, ...
```

## Fake ELF, Real Image

`print_flag` is not meant to be executed. The interesting part is the corrupted tail after the real startup code. That tail is exactly `30000` bytes, which matches a raw `100 * 100 * 3` RGB image.

This snippet turns the tail into an image:

```bash
python - <<'PY'
from pathlib import Path
from PIL import Image

blob = Path('nested/print_flag').read_bytes()
tail = blob[0x1129:0x1129 + 30000]
Image.frombytes('RGB', (100, 100), tail).save('print_flag_tail_clean.png')
PY
```

The result is a clean cartoon sunset. Compared with `beautiful_sunset.png`, it is clear that the two files are paired and the PNG is the mixed-up/noisy version of that same base image.

## Recovering The Flag

Once the paired images are treated as graphics instead of normal file types, the hidden text can be isolated from the noisy overlay. In the local solve artifacts, the final readable crop was saved as `flag_crop.png`, which shows the flag directly:

```text
sillyCTF{w@$h_my_b3ll@y}
```

## Notes

The main trick is that every file lies about what it is:

- `files.zip` only contains a JPEG
- `kittycat.jpg` is secretly also a ZIP
- `print_flag` looks executable, but its payload is image data

Once that theme is recognized, the challenge collapses quickly.
