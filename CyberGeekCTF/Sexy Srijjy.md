# Sexy Srijjy

## Challenge Overview
- Event: CyberGeek CTF
- Category: Misc / Graph Reconstruction
- Artifact(s): cubeooter.zip
- Prompt: What a flag is if not product of our routes?
- Final Flag: geek{1848}

## Executive Summary
The bitmap is intentionally almost empty. Once the plotted curve is treated as a graph, its x-intercepts become the only values that matter, and multiplying those roots produces the accepted flag.

## Solution Walkthrough
### Recon
The archive contains a single bitmap:

```bash
unzip -l cubeooter.zip
file graph_challenge.bmp
```

Results:

```text
graph_challenge.bmp: PC bitmap, Windows 3.x format, 500 x 500 x 24
```

The BMP is almost empty. It contains only black and white pixels:

```python
from PIL import Image
import numpy as np

img = Image.open("graph_challenge.bmp").convert("L")
arr = np.array(img)
ys, xs = np.where(arr > 0)
pts = sorted((int(x), int(y)) for x, y in zip(xs, ys))

print(len(pts))
print(pts)
```

There are only 39 white pixels, so the intended data is the plotted graph, not
a normal rendered image or a large hidden payload.

### Solve
The hint says "product of our routes", which points to "roots". The bitmap is
500 pixels high, so the natural x-axis for a centered graph is row `y = 250`.

Extracting the white pixels on that row gives:

```python
roots = [x for x, y in pts if y == 250]
print(roots)
```

Output:

```text
[3, 8, 77]
```

As a sanity check, these roots fit the visible curve as a cubic:

```python
round(250 - 0.01 * (x - 3) * (x - 8) * (x - 77))
```

This model matches every plotted pixel within one pixel of rasterization.

The requested value is the product of the roots:

```text
3 * 8 * 77 = 1848
```

## Flag
```text
geek{1848}
```
