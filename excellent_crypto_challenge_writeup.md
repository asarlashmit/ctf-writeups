# Excellent Crypto Challenge Writeup

## Summary

The base64 blob decodes to:

```text
p = 2305843009213693951
a = 2
b = 3
G = (5, 1)
Q = (1896662826602979990, 1425560687073180440)
R = (320761691192926716, 300507216662097367)
cipher_1 = dcb9e7887358f88e5ba5c366b459f93889ed77a6
cipher_2 = eea0fb96656dc9ac0092d069fd5ec71998db4aad60764fedf3b75511ea34e151
```

At first glance this looks like ECC over:

```text
y^2 = x^3 + 2x + 3 mod p
```

That interpretation is wrong, because `G`, `Q`, and `R` are not on that curve.

## Step 1: Recover the real curve

The point addition and doubling formulas for short Weierstrass curves never use `b`, so if the implementation started from `G` anyway, the real curve is determined by `G`:

```text
b' = y^2 - x^3 - ax mod p
   = 1^2 - 5^3 - 2*5 mod p
   = -134 mod p
```

So the actual curve is:

```text
y^2 = x^3 + 2x - 134 mod p
```

`G`, `Q`, and `R` all lie on this curve.

## Step 2: Recover the nonce scalar

The point `R` is an ephemeral public point, so we look for a small discrete log:

```text
R = kG
```

A short baby-step/giant-step search finds:

```text
k = 247439
```

That is tiny for an ECC nonce and immediately breaks the shared secret.

## Step 3: Compute the shared secret

Now compute:

```text
S = kQ = (2199468938808738143, 791263125865558851)
```

The encryption key is:

```python
sha256(str(S.x).encode()).digest()
```

Using that digest as the XOR keystream gives:

```text
cipher_1 -> Approved transaction
cipher_2 -> sillyCTF{Can't_Be_Reusing_NONCE}
```

## Flag

```text
sillyCTF{Can't_Be_Reusing_NONCE}
```

## Reproduction

Run:

```bash
python3 solve_excellent_crypto_challenge.py
```
