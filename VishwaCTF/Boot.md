# Boot Writeup

Challenge: `Boot`

Flag: `VishwaCTF{iM4G35_4Re_n0t_th4T_c00L}`

## Summary

The challenge file is a bootable ISO. The main trick is that you do **not** need to boot it.

Static analysis shows the filesystem contents are identical to the official Tiny Core 17.0 ISO. The only interesting difference is in the bootloader blob `boot/isolinux/isolinux.bin`.

That file has an extra 147-byte tail containing the same 48-character string repeated 3 times:

```text
9d5zYNcj9f1S46vC74aruPgE9b6T3ceX4Qsa2VQAbVnDFfTr
```

This string is valid Base58. Decoding it gives the flag.

## Recon

Identify the file type:

```bash
file challenge.bin
```

Output:

```text
ISO 9660 CD-ROM filesystem data 'ISOIMAGE' (bootable)
```

Extract and inspect:

```bash
7z x -y challenge.bin -oiso
isoinfo -i challenge.bin -R -f
```

The visible ISO contents look like a normal Tiny Core Linux boot ISO.

## Compare Against Official Tiny Core

Download the official Tiny Core 17.0 ISO and compare:

```bash
curl -sL http://tinycorelinux.net/17.x/x86/release/TinyCore-17.0.iso -o official.iso
7z x -y official.iso -oofficial-iso
```

Then compare extracted files:

```bash
python3 - <<'PY'
import hashlib
from pathlib import Path

def md5tree(root):
    out = {}
    for p in Path(root).rglob('*'):
        if p.is_file():
            out[str(p.relative_to(root))] = hashlib.md5(p.read_bytes()).hexdigest()
    return out

A = md5tree('iso')
B = md5tree('official-iso')

for k in sorted(set(A) & set(B)):
    if A[k] != B[k]:
        print(k, A[k], B[k])
PY
```

Only these differed:

```text
[BOOT]/Boot-NoEmul.img
boot/isolinux/boot.cat
boot/isolinux/isolinux.bin
```

That points directly at the El Torito boot structures, not the ISO filesystem.

## Extract the Hidden Data

Compare the challenge `isolinux.bin` with the official one:

```bash
python3 - <<'PY'
from pathlib import Path

p = Path('iso/boot/isolinux/isolinux.bin').read_bytes()
q = Path('official-iso/boot/isolinux/isolinux.bin').read_bytes()

print('challenge size:', len(p))
print('official size :', len(q))
print('extra tail    :', len(p) - len(q))
print((p[len(q):]).decode())
PY
```

This reveals the appended tail:

```text
9d5zYNcj9f1S46vC74aruPgE9b6T3ceX4Qsa2VQAbVnDFfTr
9d5zYNcj9f1S46vC74aruPgE9b6T3ceX4Qsa2VQAbVnDFfTr
9d5zYNcj9f1S46vC74aruPgE9b6T3ceX4Qsa2VQAbVnDFfTr
```

Take one copy and decode it as Base58:

```bash
python3 - <<'PY'
s = '9d5zYNcj9f1S46vC74aruPgE9b6T3ceX4Qsa2VQAbVnDFfTr'
alphabet = '123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz'
num = 0
for c in s:
    num = num * 58 + alphabet.index(c)
out = num.to_bytes((num.bit_length() + 7) // 8, 'big')
print(out.decode())
PY
```

Output:

```text
VishwaCTF{iM4G35_4Re_n0t_th4T_c00L}
```

## Final Flag

```text
VishwaCTF{iM4G35_4Re_n0t_th4T_c00L}
```
