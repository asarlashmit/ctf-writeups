# Recovered Signal File

## Challenge

- Name: Recovered Signal File
- Type: Forensics / Crypto
- Flag format: `flag{}`

## Summary

The recovered log contained a Base64-encoded transmission. Decoding the Base64 produced text that was still unreadable but matched the shape of a Caesar-shifted flag. A Caesar shift of 3 recovered the plaintext flag.

## Reconnaissance

The downloaded file was small ASCII text:

```bash
file signal.log
wc -c signal.log
sed -n '1,120p' signal.log
```

Important content:

```text
Encoded Transmission:
aW9kant2ZHdob29sd2hfdmxqcWRvX2doZnJnaGd9
```

## Decode

First, decode the transmission from Base64:

```bash
python3 - <<'PY'
import base64

encoded = "aW9kant2ZHdob29sd2hfdmxqcWRvX2doZnJnaGd9"
print(base64.b64decode(encoded).decode())
PY
```

Output:

```text
iodj{vdwhoolwh_vljqdo_ghfrghg}
```

This resembles the target flag format, but each alphabetic character is shifted forward. Applying a Caesar shift of 3 backward gives:

```bash
python3 - <<'PY'
import string

ciphertext = "iodj{vdwhoolwh_vljqdo_ghfrghg}"
alphabet = string.ascii_lowercase
plaintext = ''.join(
    alphabet[(alphabet.index(c) - 3) % 26] if c in alphabet else c
    for c in ciphertext
)
print(plaintext)
PY
```

## Flag

```text
flag{satellite_signal_decoded}
```
