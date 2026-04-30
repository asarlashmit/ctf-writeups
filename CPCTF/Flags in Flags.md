# Flags in Flags

## Challenge

- Name: `Flags in Flags`
- Type: `forensics / image stego`
- File: `Flaggggg_2348u932f4728nv.png`

## Recon

The PNG was a normal `1553x1536` RGBA image with no appended files or useful metadata beyond `Software: Figma`.

Visual inspection showed a repeated pattern of colored flags, which matched the hint about hiding a flag among many flags.

## Solve

The useful path was low-bit visualization instead of metadata extraction.

I generated derived images from the PNG:

- RGB channel splits
- contrast/auto-level views
- HSV channels
- low-bit visualizations

The hidden text appeared in the `lsb_4` image, where the lower 4 bits of each RGB channel were scaled up for visibility.

Example command used:

```bash
python3 - <<'PY'
from PIL import Image
img = Image.open('Flaggggg_2348u932f4728nv.png').convert('RGB')
w, h = img.size
pix = img.load()
out = Image.new('RGB', (w, h))
for y in range(h):
    for x in range(w):
        r, g, b = pix[x, y]
        mask = 0x0f
        out.putpixel((x, y), tuple((v & mask) * 17 for v in (r, g, b)))
out.save('lsb_4.png')
PY
```

After inspecting `lsb_4.png`, one flag near the center-left contained the hidden string:

`CPCTF{FLAG_MANY_FLAGS_FLAG}`

## Flag

`CPCTF{FLAG_MANY_FLAGS_FLAG}`
