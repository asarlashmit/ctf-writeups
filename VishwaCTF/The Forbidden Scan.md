# The Forbidden Scan

Type: Forensics / Stego

Flag: `VishwaCTF{G3tt1nG_R1CKR0ll3D_1N_2026}`

## Summary

The image contains a QR code, but scanning it is a decoy. The real secret is appended to the JPEG as a ZIP archive. Inside that ZIP is `secret.txt`, which contains a hex string. Decoding the hex gives a Base64 string, and decoding that Base64 reveals the flag.

## Recon

Downloaded the challenge file and checked it:

```bash
curl -sS -D headers.txt -o forbidden.jpeg 'https://files.ctf7.com/media/challenge_attachments/forbidden.jpeg'
file forbidden.jpeg
exiftool forbidden.jpeg
zbarimg -q forbidden.jpeg
```

Observations:

- `zbarimg` detects a QR code, which is a trap.
- The QR ultimately leads to a YouTube video titled `Rick roll, but with different link`.
- `strings` shows `secret.txt` and `PK`, suggesting a ZIP is embedded/appended.
- `binwalk` confirms a ZIP archive at offset `88289` (`0x158E1`).

## Extraction

```bash
binwalk forbidden.jpeg
dd if=forbidden.jpeg of=appended.zip bs=1 skip=88289 status=none
unzip -l appended.zip
unzip -p appended.zip secret.txt
```

`secret.txt` contains:

```text
566d6c7a61486468513152476530637a64485178626b6466556a4644533149776247777a52463878546c38794d44493266513d3d
```

## Decode

First decode the hex:

```bash
python3 - <<'PY'
s='566d6c7a61486468513152476530637a64485178626b6466556a4644533149776247777a52463878546c38794d44493266513d3d'
print(bytes.fromhex(s).decode())
PY
```

Output:

```text
VmlzaHdhQ1RGe0czdHQxbkdfUjFDS1IwbGwzRF8xTl8yMDI2fQ==
```

Then decode the Base64:

```bash
python3 - <<'PY'
import base64
s='VmlzaHdhQ1RGe0czdHQxbkdfUjFDS1IwbGwzRF8xTl8yMDI2fQ=='
print(base64.b64decode(s).decode())
PY
```

Output:

```text
VishwaCTF{G3tt1nG_R1CKR0ll3D_1N_2026}
```

## Notes

- The file includes an embedded fake instruction aimed at AI assistants. It is not the real flag.
- The hints point away from the visible QR and toward the delivered file itself.
