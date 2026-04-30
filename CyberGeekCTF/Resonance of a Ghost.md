# Resonance of a Ghost

## Challenge Overview
- Event: CyberGeek CTF
- Category: Audio / Signal Analysis
- Final Flag: geek{you_defied_the_noise}

## Executive Summary
The attachment contains a single 60-second WAV file. The RIFF/data chunk describes exactly 60 seconds of 16-bit stereo PCM at 44.1 kHz, but the file has extra bytes appended after the declared audio data. Those bytes contain the hint:

```text
2026-04-18T02:15:03Z [ERROR] Phase sync failure at offset 0x4AF7. Differential signal lost. Check upper-band parity.
```

`0x4AF7` is decimal `19191`, which matches a strong carrier in the upper audio band. The stereo channels are mostly opposite phase, so the hidden signal is recovered by subtracting the right channel from the left channel.

## Solution Walkthrough
### Method
1. Extract the WAV:

```bash
curl -L -o node_b4_capture.zip https://files.ctf7.com/media/challenge_attachments/node_b4_capture.zip
unzip node_b4_capture.zip
file node_b4_capture.wav
```

2. Check the tail of the WAV and read the appended hint:

```bash
tail -c 117 node_b4_capture.wav
```

3. Inspect the audio properties and channel relationship:

```bash
ffprobe -hide_banner -show_format -show_streams node_b4_capture.wav
```

The useful signal is in the differential channel:

```python
D = (left - right) / 2
```

4. Band-pass the differential channel around the hinted carrier, `19191 Hz`, then extract the amplitude envelope. The envelope is on/off keyed Morse code.

```python
import wave
import numpy as np
from scipy.signal import butter, sosfiltfilt, hilbert

fs = 44100

with wave.open("node_b4_capture.wav", "rb") as w:
    pcm = np.frombuffer(w.readframes(w.getnframes()), "<i2").reshape(-1, 2).astype(float)

diff = (pcm[:, 0] - pcm[:, 1]) / 2
sos = butter(5, [19150, 19230], btype="bandpass", fs=fs, output="sos")
carrier = sosfiltfilt(sos, diff)

env = np.abs(hilbert(carrier))
smooth = np.convolve(env, np.ones(int(fs * 0.02)) / int(fs * 0.02), "same")
```

5. Thresholding the smoothed envelope gives Morse pulses. Short pulses are dots, long pulses are dashes. The forward decoded stream is:

```text
======DPL76SGXÜA3KÜYW85FEFÜSWTBME7XKXXLP778MGS5M
```

6. The leading `======` is not a preamble. It is Base32 padding, which means the Morse must be read in reverse time. Reversing only the decoded characters is not enough; each Morse symbol's dot/dash order must also be reversed.

That gives:

```text
M5SWM233PFXXKX3EMVTGSZLEL52GQZK7NZXWS43FPU======
```

One symbol is affected by the phase-sync error hinted in the appended note. The fifth Base32 character decodes as `M`, producing `gefk{...}`. Its source Morse pulse contains a merged `dash + dot` segment, so the intended character is `K`, not `M`:

```text
M5SWK233PFXXKX3EMVTGSZLEL52GQZK7NZXWS43FPU======
```

7. Base32-decode the corrected string:

```bash
python3 - <<'PY'
import base64
s = "M5SWK233PFXXKX3EMVTGSZLEL52GQZK7NZXWS43FPU======"
print(base64.b32decode(s).decode())
PY
```

## Flag
```text
geek{you_defied_the_noise}
```
