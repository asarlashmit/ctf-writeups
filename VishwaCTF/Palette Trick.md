# Palette Trick Writeup

## Challenge

- Name: Palette Trick
- Category: Forensics / Stego
- File: `wheel.png`

## Observation

The PNG is a `31x1` indexed-color image (`mode=P`).

That matters because indexed PNGs store:

1. A pixel index table
2. A palette that maps each index to an RGB color

The challenge hint says the flag is hidden in the order of colors in the index table. In this file, the pixel data is simply:

`[0, 1, 2, ..., 30]`

So each pixel points to the matching palette entry in order.

## Extraction

Reading the first 31 palette entries shows that each entry's red-channel value is a printable ASCII byte:

```python
from PIL import Image

img = Image.open("wheel.png")
pal = img.getpalette()
vals = [pal[i * 3] for i in range(31)]   # red value from each RGB triplet
print(vals)
print("".join(chr(v) for v in vals))
```

Output:

```text
[86, 105, 115, 104, 119, 97, 67, 84, 70, 123, 80, 52, 108, 51, 116, 116, 51, 95, 49, 110, 100, 51, 120, 95, 83, 51, 99, 114, 51, 116, 125]
VishwaCTF{P4l3tt3_1nd3x_S3cr3t}
```

## Flag

`VishwaCTF{P4l3tt3_1nd3x_S3cr3t}`
