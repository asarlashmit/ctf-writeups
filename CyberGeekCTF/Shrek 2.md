# Shrek 2

## Challenge Overview
- Event: CyberGeek CTF
- Target / Given: Shrek 2
- Final Flag: geek{donkey_fly}

## Executive Summary
The attachment is a PNG with a ZIP archive appended after the `IEND` chunk. The ZIP is encrypted, and the password is hidden in the image's Fourier spectrum. After extracting the ZIP, the payload is whitespace-encoded Morse.

## Solution Walkthrough
### Recon
Download and identify the image:

```bash
curl -L -o donkey.png https://files.ctf7.com/media/challenge_attachments/donkey.png
file donkey.png
exiftool donkey.png
binwalk donkey.png
```

Important findings:

- `donkey.png` is a 1172x1172 RGB PNG.
- `exiftool` reports trailer data after `IEND`.
- `binwalk` finds an appended ZIP archive containing encrypted `vault/hehe.txt`.

Carve the ZIP:

```bash
tail -c +3342471 donkey.png > appended.zip
unzip -l appended.zip
```

### FFT Clue
The visible stripes suggest frequency-domain hiding. Compute an FFT magnitude image:

```bash
python3 - <<'PY'
from PIL import Image, ImageOps
import numpy as np

img = np.array(Image.open('donkey.png').convert('L'), dtype=float)
f = np.fft.fftshift(np.fft.fft2(img))
mag = np.log1p(np.abs(f))
mag = (mag - mag.min()) / (mag.max() - mag.min()) * 255
Image.fromarray(mag.astype('uint8')).save('fft_gray.png')
ImageOps.autocontrast(Image.fromarray(mag.astype('uint8'))).save('fft_gray_autocontrast.png')
PY
```

Inspecting `fft_gray_autocontrast.png` reveals:

```text
villain of movie is key
```

The movie is Shrek 2, whose villain is the Fairy Godmother, so the ZIP password is:

```text
fairygodmother
```

Extract:

```bash
7z x -y -pfairygodmother appended.zip -ozipout
```

### Morse Payload
The extracted `vault/hehe.txt` contains only spaces, tabs, and newlines. Interpret:

- tab = dash
- space = dot
- newline = character separator

```bash
python3 - <<'PY'
M = {
    '.-':'A','-...':'B','-.-.':'C','-..':'D','.':'E','..-.':'F','--.':'G',
    '....':'H','..':'I','.---':'J','-.-':'K','.-..':'L','--':'M','-.':'N',
    '---':'O','.--.':'P','--.-':'Q','.-.':'R','...':'S','-':'T','..-':'U',
    '...-':'V','.--':'W','-..-':'X','-.--':'Y','--..':'Z','-----':'0',
    '.----':'1','..---':'2','...--':'3','....-':'4','.....':'5','-....':'6',
    '--...':'7','---..':'8','----.':'9','-.--.':'(','-.--.-':')','..--.-':'_',
}

data = open('zipout/vault/hehe.txt', 'rb').read()
morse = data.replace(b' ', b'.').replace(b'\t', b'-').decode().splitlines()
print(''.join(M[x] for x in morse if x))
PY
```

Output:

```text
GEEK(DONKEY_FLY)
```

Convert the Morse parentheses to the required flag braces and lowercase the prefix:

```text
geek{donkey_fly}
```

## Flag
```text
geek{donkey_fly}
```
