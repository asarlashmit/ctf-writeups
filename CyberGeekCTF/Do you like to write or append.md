# Do you like to write or append?

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics / Steganography
- Artifact(s): strawberry_ice_cream.jpg
- Prompt: Recover the hidden image appended to the provided JPEG.
- Final Flag: geek{deja-vu!}

## Executive Summary
The provided JPEG contains another complete JPEG appended after the first image. Normal image viewers display only the first JPEG, so the hidden appended image must be carved out and viewed separately.

## Solution Walkthrough
### Recon
I first checked the files and metadata:

```bash
file strawberry_ice_cream.jpg part1.jpg part2.jpg
exiftool strawberry_ice_cream.jpg part1.jpg part2.jpg
binwalk strawberry_ice_cream.jpg
```

`binwalk` showed two JPEG signatures inside `strawberry_ice_cream.jpg`:

```text
DECIMAL       HEXADECIMAL     DESCRIPTION
0             0x0             JPEG image data, JFIF standard 1.01
182202        0x2C7BA         JPEG image data, JFIF standard 1.01
```

The file sizes also confirmed the append trick:

```bash
wc -c strawberry_ice_cream.jpg part1.jpg part2.jpg
cmp -s strawberry_ice_cream.jpg <(cat part1.jpg part2.jpg); echo $?
```

The result was `0`, meaning:

```text
strawberry_ice_cream.jpg == part1.jpg + part2.jpg
```

### Exploit
The second JPEG starts at byte offset `182202`, so it can be extracted with:

```bash
dd if=strawberry_ice_cream.jpg of=hidden.jpg bs=1 skip=182202 status=none
```

Opening the extracted image reveals the flag as visible text.

## Flag
```text
geek{deja-vu!}
```
