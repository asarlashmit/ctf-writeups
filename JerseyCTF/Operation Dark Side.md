# Operation Dark Side Writeup

## Challenge

- Category: Crypto
- Artifact: `intercepted_transmission.txt`
- Flag: `jctf{relay_safe_mode_activated_2026}`

## Solution

The intercepted file contains three useful fields:

- A long hex-encoded ciphertext.
- `Secret Key: ecf2584110823e9d31c049207f8034b923f213b52c1ea9a2a18661dc6063a400`
- `IV: 91e94784686346258e6acc81009edd65`

The description points to Rijndael, which is AES. The key is 32 bytes, so the correct OpenSSL primitive is AES-256. Since an IV is supplied and the ciphertext length is block-aligned, AES-256-CBC is the natural first mode to test.

```bash
python3 - <<'PY'
from pathlib import Path
import re, subprocess

text = Path("intercepted_transmission.txt").read_text()
ciphertext = re.match(r"([0-9a-fA-F]+)", text).group(1)
key = re.search(r"Secret Key:\s*([0-9a-fA-F]+)", text).group(1)
iv = re.search(r"IV:\s*([0-9a-fA-F]+)", text).group(1)

Path("/tmp/ods_ct.bin").write_bytes(bytes.fromhex(ciphertext))
plaintext = subprocess.check_output([
    "openssl", "enc", "-d", "-aes-256-cbc",
    "-K", key,
    "-iv", iv,
    "-in", "/tmp/ods_ct.bin",
])

print(plaintext.decode())
PY
```

The decrypted telemetry includes this line:

```text
[SEC_LOG]: NV_MEM_DUMP: amN0ZntyZWxheV9zYWZlX21vZGVfYWN0aXZhdGVkXzIwMjZ9
```

That value is base64:

```bash
printf '%s' 'amN0ZntyZWxheV9zYWZlX21vZGVfYWN0aXZhdGVkXzIwMjZ9' | base64 -d
```

Decoded flag:

```text
jctf{relay_safe_mode_activated_2026}
```
