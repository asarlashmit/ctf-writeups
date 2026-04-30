# myDefinitive Encryption Standard

## Flag

`jctf{f3ist&l_fun_w1thout_sb0x}`

## Solution

The provided `jersey_chall_file.py` implements a 4-round Feistel-style block cipher over 8-byte blocks. Encryption parses each block as two 32-bit halves, applies:

```python
l, r = r, l ^ bbb(r, k)
```

and then outputs `r || l` as a final swap-back.

To decrypt a ciphertext block, start from `l = ciphertext[4:8]` and `r = ciphertext[0:4]`, then apply the round keys in reverse:

```python
l, r = r ^ bbb(l, k), l
```

The key hint is `0xD4D4A1A0`. The challenge hint mentions IBM systems, pointing to EBCDIC. Replacing one missing EBCDIC byte from the hint gave candidate key `0xD4D4A1A5`. To confirm it was not a false positive, I brute-forced the full 32-bit keyspace against the known first flag block `jctf{f3i`; the only hit was:

```text
key = 0xd4d4a1a5
decrypted = jctf{f3ist&l_fun_w1thout_sb0\x00\x00x}
```

The two zero bytes are not part of the original flag. The encryptor processes the final short block without padding, but `to_bytes(4, "big")` expands the short right half back to four bytes during decryption. Removing those inserted zeros gives:

```text
jctf{f3ist&l_fun_w1thout_sb0x}
```

Re-encrypting that 30-byte plaintext with key `0xd4d4a1a5` reproduces the given ciphertext exactly.
