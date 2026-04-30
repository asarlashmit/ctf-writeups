# GhostLoadnew Writeup

Challenge: `GhostLoadnew`  
Type: `android / reverse`

## Flag

`Kaal{gh0st_1n_th3_m3m0ry}`

## Summary

The APK uses a two-stage login flow:

1. A visible first-stage user initializes a native payload.
2. A hidden second-stage user triggers a fake memory fault.
3. The native loader logs a pointer to a decrypted in-memory buffer.
4. That buffer is the real flag.

## Recon

The APK contains:

- `assets/core.bin`
- `lib/x86_64/libloader.so`
- app code in `classes3.dex`

Decompiling showed these important classes:

- `MainActivity`
- `LoaderBridge`
- `IntegrityCheck`

`MainActivity.handleLogin()` reveals the visible credentials:

- username: `analyst01`
- password: `Inv3stigate!`

Those credentials do not give the flag directly. They only call:

```java
LoaderBridge.initPayload(context)
```

## Loader Analysis

`libloader.so` reads `assets/core.bin`, XORs every byte with `0x47`, treats the first 4 bytes as the output size, then `zlib`-decompresses the rest into `payload.so`.

Extraction logic:

```python
from pathlib import Path
import struct, zlib

data = bytearray(Path("jadx_out/resources/assets/core.bin").read_bytes())
for i in range(len(data)):
    data[i] ^= 0x47

size = struct.unpack("<I", data[:4])[0]
payload = zlib.decompress(data[4:])
Path("payload.so").write_bytes(payload)
```

## Hidden Credentials

Inside `payload.so`, `handle_login()` stores obfuscated username and password in `.rodata`.

- `ENC_USER = b"ri\`env"` and each byte is XORed with `0x01`
- `ENC_PASS = b"Uj3qr1p"` and each byte is XORed with `0x02`

This decodes to:

- username: `shadow`
- password: `Wh1sp3r`

When those are supplied, the payload returns:

```text
MEMORY_FAULT: Native heap state corrupted.Inspect Memory/logs see if you see some ghost.
```

## Real Flag Path

Back in `libloader.so`, a helper routine decrypts a 25-byte buffer before the payload is loaded.

The encrypted bytes are:

```text
f1 e8 f9 d6 f2 ff d2 b9 eb ce d6 a9 d4 d6 ec d2 ba c7 d7 ba f5 8a fb e1 c7
```

The XOR key repeats every 3 bytes:

- index `% 3 == 0` -> `0xBA`
- index `% 3 == 1` -> `0x89`
- index `% 3 == 2` -> `0x98`

Recovery:

```python
enc = bytes.fromhex("f1e8f9d6f2ffd2b9ebced6a9d4d6ecd2bac7d7baf58afbe1c7")
keys = [0xBA, 0x89, 0x98]
flag = bytes(b ^ keys[i % 3] for i, b in enumerate(enc)).decode()
print(flag)
```

Output:

```text
Kaal{gh0st_1n_th3_m3m0ry}
```

## Final Answer

`Kaal{gh0st_1n_th3_m3m0ry}`
