# Its not easy being green

## Challenge Type

Steganography / image encoding

## Recon

The provided file was a PNG:

```bash
file challenge.png
```

The image looked like a 28x16 grid of colored squares separated by black lines. Counting the separators showed a regular layout:

- cell size: 65x65
- separator size: 2 pixels
- grid size: 28 columns x 16 rows

The title clue, `It's not easy being green`, strongly suggested checking the green channel first.

## Solve

Sampling the center of each cell and reading only the green byte in row-major order produced ASCII text directly.

Example extraction logic:

```python
from PIL import Image

img = Image.open("challenge.png").convert("RGB")
pixels = img.load()

rows, cols = 16, 28
cell, gap = 65, 2
step = cell + gap

text = []
for row in range(rows):
    y = row * step + cell // 2
    for col in range(cols):
        x = col * step + cell // 2
        text.append(chr(pixels[x, y][1]))

print("".join(text).rstrip("\x00"))
```

That decoded the lyrics to `Rainbow Connection`, and embedded in the text was the flag:

`sillyCTF{d@rk_k3rm1t_s@ys_d0_1t}`

## Flag

`sillyCTF{d@rk_k3rm1t_s@ys_d0_1t}`
