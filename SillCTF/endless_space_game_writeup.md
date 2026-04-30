# Endless Space Game Writeup

Challenge: `Endless Space Game`
Category: `Reverse Engineering`

Flag: `sillyCTF{retroremastered}`

## Summary

The provided link was a `shorturl.at` shortlink protected by Cloudflare. Instead of fighting the redirect directly, I used ShortURL's own unshorten feature to recover the real destination, which was a public SharePoint folder containing a Unity game ZIP.

After downloading the ZIP and unpacking the Unity build, I reversed `Assembly-CSharp.dll`. The game includes a `PrintFlag` script and a `FlagCrypto` helper that decrypts a base64 ciphertext with AES using:

- password: `dEi0245RHYB12ic`
- salt: `sillysalting`
- PBKDF2-HMAC-SHA1, `100000` iterations

At first, decrypting the hardcoded ciphertext from the DLL only produced a placeholder:

`change this flag before production!`

That was not the real flag.

## The Important Detail

`PrintFlag.flag` is a public Unity field, so its value can be overridden in the serialized scene data even if the constructor contains a default.

Using `UnityPy`, I inspected the `level2` scene and found `WinManager` had a `PrintFlag` component with a different serialized ciphertext:

`SB1WutP8DlpgdkPnQPf7Jre3aL8UfKVcOIvokdpWCbs=`

That is the value actually used by the win scene.

## Decryption

```python
import base64
import hashlib
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

cipher_b64 = "SB1WutP8DlpgdkPnQPf7Jre3aL8UfKVcOIvokdpWCbs="
password = b"dEi0245RHYB12ic"
salt = b"sillysalting"

ct = base64.b64decode(cipher_b64)
dk = hashlib.pbkdf2_hmac("sha1", password, salt, 100000, 48)
key, iv = dk[:32], dk[32:48]

pt = unpad(AES.new(key, AES.MODE_CBC, iv).decrypt(ct), 16)
print(pt.decode())
```

Output:

```text
sillyCTF{retroremastered}
```

## Notes

- DLL default ciphertext was a decoy/placeholder.
- The real flag lived in the Unity scene serialization, not only in code.
- The winning scene was `level2`, where the serialized `PrintFlag` component overrides the public `flag` field.
