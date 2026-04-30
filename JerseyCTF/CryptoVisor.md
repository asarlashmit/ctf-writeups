# CryptoVisor

## Challenge

- Name: CryptoVisor
- Category: Crypto
- Goal: Decode the intercepted TradeVisor transmission and recover the access key.

## Solution

The provided download initially saved as an HTML page, but the real challenge text was present in `CryptoVisor_real.txt`.

The ciphertext used printable ASCII characters and started with:

```text
*@F 92G6 7@F?5 :E]
```

This is a strong indicator for ROT47, since `*@F` decodes to `You`.

Decode ROT47 across printable ASCII:

```bash
python3 - <<'PY'
from pathlib import Path
s = Path('CryptoVisor_real.txt').read_text()
out = ''.join(chr(33 + ((ord(c) - 33 + 47) % 94)) if 33 <= ord(c) <= 126 else c for c in s)
print(out)
PY
```

The decoded message contained a second encrypted sentence:

```text
Lbh unir oebxra vagb gur GenqrIvfbe argjbex naq unir erprvirq gur synt. Gur synt vf wpgs{4poqpP0Qp=SRoq0W_S}
```

The hint mentioned another ROT decryption, and this text is ROT13. Decoding it:

```bash
python3 - <<'PY'
import codecs
msg = 'Lbh unir oebxra vagb gur GenqrIvfbe argjbex naq unir erprvirq gur synt. Gur synt vf wpgs{4poqpP0Qp=SRoq0W_S}'
print(codecs.decode(msg, 'rot_13'))
PY
```

Output:

```text
You have broken into the TradeVisor network and have received the flag. The flag is jctf{4cbdcC0Dc=FEbd0J_F}
```

## Flag

```text
jctf{4cbdcC0Dc=FEbd0J_F}
```
