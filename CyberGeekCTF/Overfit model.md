# Overfit Model

## Challenge Overview
- Event: CyberGeek CTF
- Category: Misc / Machine Learning
- Final Flag: geek{youthecybergeek}

## Executive Summary
The model is a tiny CNN:

- `Conv2d(3, 8, kernel_size=5, stride=2)`
- `ReLU`
- `AdaptiveAvgPool2d((10, 10))`
- `Linear(800, 15)`

After parsing `vault.pth` directly, the important observation was that the final linear layer had highly correlated rows and the last conv filter was mostly a positive blur-like kernel. In practice, the network had collapsed into a coarse brightness mapper instead of a true "exact-image lock".

That means the intended training image is not actually needed. A simple forged input works.

## Solution Walkthrough
### What worked
I generated a uniform gray JPEG with pixel value `208` on all channels:

```python
from PIL import Image
Image.new("RGB", (640, 640), (208, 208, 208)).save("constant_208.jpg", quality=100)
```

Running the model logic on that image produced:

```text
zpvuifdzcfshffl
```

This is a Caesar-shifted string. Shifting each lowercase letter back by `1` gives:

```text
youthecybergeek
```

So the flag is:

```text
geek{youthecybergeek}
```

## Flag
```text
geek{youthecybergeek}
```

## References and Notes
### Notes
- The challenge hint about "ASCII values" was the useful one.
- The model is overfit in a very weak way: because the learned filters are mostly positive and heavily pooled, many smooth bright images give similar outputs.
- A forged constant image is enough to recover the hidden phrase.
