# Shrek 1

## Challenge Overview
- Event: CyberGeek CTF
- Target / Given: Shrek 1
- Final Flag: geek{far_far_away}

## Executive Summary
The hint said the photo felt "noisy" and the description emphasized "layers". That suggested image/stego analysis rather than metadata or file carving.

## Solution Walkthrough
### Recon
- Downloaded the provided file and confirmed it was a plain RGB PNG:
  - `file shrek.png`
  - `exiftool shrek.png`
  - `binwalk shrek.png`
- No useful metadata, no appended payload, and no obvious embedded file in PNG chunks.
- LSB and transform checks produced noise and false positives, but nothing directly recoverable.

### Solve Path
The useful step was moving to the frequency domain.

1. Compute the FFT magnitude of the grayscale image:

```bash
python3 - <<'PY'
from PIL import Image
import numpy as np

img = np.array(Image.open('shrek.png').convert('L'), dtype=float)
f = np.fft.fftshift(np.fft.fft2(img))
mag = np.log1p(np.abs(f))
mag = (mag - mag.min()) / (mag.max() - mag.min()) * 255
Image.fromarray(mag.astype('uint8')).save('fft_gray.png')
PY
```

2. Inspect `fft_gray.png`.

Visible text appears in the corners of the spectrum:

`WjJWbGEzdG1ZWEpmWm1GeVgyRjNZWGw5`

3. Decode it once:

```bash
python3 - <<'PY'
import base64
print(base64.b64decode('WjJWbGEzdG1ZWEpmWm1GeVgyRjNZWGw5'))
PY
```

Output:

`Z2Vla3tmYXJfZmFyX2F3YXl9`

4. Decode that result again:

```bash
python3 - <<'PY'
import base64
print(base64.b64decode('Z2Vla3tmYXJfZmFyX2F3YXl9').decode())
PY
```

Output:

`geek{far_far_away}`

## Flag
```text
geek{far_far_away}
```
