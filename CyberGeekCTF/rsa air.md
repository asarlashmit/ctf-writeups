# RSA Air

## Challenge Overview
- Event: CyberGeek CTF
- Category: Crypto / RSA
- Prompt: Decrypt the tiny RSA instance `c = 0b1100`, `e = 5`, `n = 15`.
- Final Flag: geek{12}

## Executive Summary
This RSA instance is intentionally tiny. Factoring `n = 15`, computing `phi(n)`, and deriving the private exponent is enough to decrypt the ciphertext immediately.

## Solution Walkthrough
### Solution
RSA is only secure when `n` is hard to factor. Here, `n` is tiny:

```text
n = 15 = 3 * 5
```

After factoring `n`, compute Euler's totient:

```text
phi(n) = (3 - 1) * (5 - 1) = 8
```

The private exponent is the modular inverse of `e` modulo `phi(n)`:

```text
d = 5^-1 mod 8 = 5
```

Convert the ciphertext from binary:

```text
c = 0b1100 = 12
```

Decrypt:

```text
m = c^d mod n
m = 12^5 mod 15
m = 12
```

So the plaintext integer is:

```text
m = 12
```

### Verification
The included solver reproduces the result:

```bash
python3 solve.py
```

Output:

```text
n = 15 = 3 * 5
e = 5
c = 0b1100 = 12
phi(n) = 8
d = 5
m = 12
brute force matches = [12]
flag = geek{12}
```

The brute-force check confirms that among all valid RSA plaintext residues `0..14`, only `12` encrypts to `12` with `(e, n) = (5, 15)`.

This also makes sense from modular arithmetic: Carmichael's function for `15` is `lambda(15) = lcm(2, 4) = 4`, and `5 = 1 mod 4`, so `m^5 = m mod 15` for every residue modulo `15`.

## Flag
```text
geek{12}
```
