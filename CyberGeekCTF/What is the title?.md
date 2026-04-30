# What is the title?

## Challenge Overview
- Event: CyberGeek CTF
- Category: Reverse engineering / OSINT
- Final Flag: geek{It's over, isn't it?}

## Executive Summary
The binary’s runtime output is a decoy. Its immediate-byte writes hide a ciphertext that rail-fence decodes into a YouTube link whose real song title becomes the flag.

## Solution Walkthrough
### Solution
The attachment is a small unstripped 64-bit PIE ELF:

```bash
file Problem
```

Output:

```text
Problem: ELF 64-bit LSB pie executable, x86-64, ... not stripped
```

Running it only prints a decoy:

```bash
chmod +x Problem
./Problem
```

```text
nothing here mate ;)
```

Disassembling `main` shows the interesting part. The binary repeatedly writes immediate bytes to the same stack byte:

```asm
mov BYTE PTR [rbp-0x1],0x74
mov BYTE PTR [rbp-0x1],0x2e
mov BYTE PTR [rbp-0x1],0x6f
...
```

Those writes overwrite each other at runtime, but the immediate values form the hidden ciphertext:

```bash
objdump -d -Mintel Problem \
  | sed -n 's/.*\],0x\([0-9a-f][0-9a-f]\)$/\1/p' \
  | xxd -r -p
```

```text
t.o=W-Bs7htwycmvRCkLB8cbK3p4pwo./?o9&P0PbzNt&=swuewhSdl=vxOm5Fix:/tbactSitRn5DQ5ne/utZsufVd
```

The challenge text gives `6 9`, and the hint says to get railed. Using a rail-fence decode with 6 rails and offset 9 gives:

```text
https://www.youtube.com/watch?v=RoStZSd9CWk&list=PL-B0vRunxP8BcbO5fDmzbsKN5QV5Ft37p&index=4
```

The YouTube video title renders as invisible Unicode, so the visible page title is not useful. Pulling comments for the video shows users identifying the human-readable title as a Steven Universe song:

```text
It's over, isn't it?
```

Putting that title in the required flag wrapper gives:

```text
geek{It's over, isn't it?}
```

This exact flag was accepted by the CTF7 submission form.

## Flag
```text
geek{It's over, isn't it?}
```
