# Final Message

## Challenge

- Category: audio forensics / steganography
- File: `Final_Message.flac`

## Recon

The file is a stereo FLAC with a noisy AM/numbers-station style voice track. Metadata did not contain the flag, and normal string extraction only showed ordinary FLAC metadata.

The spoken Russian phonetic alphabet text points at a hidden coded message, but the useful artifact appears after the speech in a high-frequency band that is not obvious by listening.

## Solution

I generated a spectrogram for the high-frequency part of the audio, focusing on the end of the track:

```python
import soundfile as sf
import numpy as np
import matplotlib.pyplot as plt
from scipy.signal import stft

x, sr = sf.read("Final_Message.flac")
if x.ndim > 1:
    x = x.mean(axis=1)

y = x[int(49.0 * sr):int(56.0 * sr)]
f, t, Z = stft(y, fs=sr, nperseg=4096, noverlap=3980, window="hann")
S = 20 * np.log10(np.abs(Z) + 1e-10)

mask = (f >= 13500) & (f <= 19000)
Sb = S[mask]
fb = f[mask]

plt.figure(figsize=(12, 4), dpi=120)
plt.imshow(
    Sb,
    origin="lower",
    aspect="auto",
    extent=[49.0, 56.0, fb[0], fb[-1]],
    cmap="gray",
    vmin=np.percentile(Sb, 45),
    vmax=np.percentile(Sb, 99.7),
)
plt.axis("off")
plt.tight_layout(pad=0)
plt.savefig("hf_hidden.jpg", bbox_inches="tight", pad_inches=0)
```

The rendered spectrogram clearly spells the Cyrillic word:

```text
ЛАЙКА
```

This matches the challenge hint about the Soviet space program: Laika was the Soviet space dog. The challenge also states that the plaintext is fully capitalized and uses Cyrillic characters, so the recovered plaintext is wrapped directly in the required flag format.

## Flag

```text
jctf{ЛАЙКА}
```
