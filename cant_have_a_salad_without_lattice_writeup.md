# Can't Have A Salad Without Lattice

Type: `crypto`

## Flag

`sillyCTF{P0st_Qu@ntum_FunTime}`

## Summary

The short URL resolves to a public SharePoint folder containing a single file, [`lattice.cdf`](/home/kali/ctf-haven/ctf/lattice.cdf). The Wolfram notebook itself is mostly a 3D lattice plot, but the important data is appended at the end as comments:

- `flag_enc = eVxqWrw1CbpuaDrQbREuBH3wWB8rgsNBDaPccxa4DvInlG5TfCYf5oyKhFPOLOtz`
- 43 ciphertext blocks, each with a 3-coefficient `u` and 3-coefficient `v`

The challenge is a toy post-quantum / lattice encryption puzzle. The key observation is that the 43 blocks only use **5 unique `u` values** in a repeating cycle.

## Recon

Looking at the appended block data shows:

- only 5 distinct `u` vectors
- for any fixed `u`, each coordinate of `v` only takes 2 possible values
- those two values differ by about `q/2` with `q = 3329`

That is exactly what you expect from a toy RLWE / Kyber-style bit embedding: each `v` coefficient stores one bit by toggling between two values separated by roughly `q/2`.

## Bit Recovery

For each of the 5 repeating `u` classes, every `v` coefficient has two values:

- one “low” value
- one “high” value

If you label `low -> 0` and `high -> 1`, each block becomes 3 bits. The subtlety is that the baseline can wrap modulo `q`, so some class/coordinate pairs need that labeling flipped. There are only:

- 5 `u` classes
- 3 coefficients each

So there are only `2^15 = 32768` possible low/high flip patterns.

Brute-forcing those 15 local flip bits and then interpreting the recovered 129 bits as a 128-bit AES key plus 1 padding bit quickly produces a clean ASCII key:

```text
encryptionkey123
```

The recovered bitstream is:

```text
011001010110111001100011011100100111100101110000011101000110100101101111011011100110101101100101011110010011000100110010001100110
```

Dropping the final padding bit and splitting into bytes gives:

```text
65 6e 63 72 79 70 74 69 6f 6e 6b 65 79 31 32 33
```

which is:

```text
encryptionkey123
```

## Flag Decryption

Base64-decoding `flag_enc` gives 48 bytes, which cleanly splits into:

- first 16 bytes: IV
- last 32 bytes: AES-CBC ciphertext

Using:

- key: `encryptionkey123`
- mode: AES-CBC
- IV: first 16 decoded bytes

decrypts to:

```text
sillyCTF{P0st_Qu@ntum_FunTime}
```

## Reproduction

Run:

```bash
python3 solve_cant_have_a_salad_without_lattice.py
```

The solver parses [`lattice.cdf`](/home/kali/ctf-haven/ctf/lattice.cdf), brute-forces the 15 local low/high ambiguities, recovers the AES key, and decrypts the flag.
