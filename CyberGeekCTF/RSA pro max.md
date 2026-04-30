# RSA Pro Max

## Challenge Overview
- Event: CyberGeek CTF
- Category: Crypto / RSA
- Prompt: Recover the plaintext from two RSA ciphertexts that share the same modulus.
- Final Flag: geek{maiti}

## Executive Summary
Both ciphertexts use the same modulus with different coprime exponents. That makes the challenge a textbook common-modulus attack recoverable with Bezout coefficients.

## Solution Walkthrough
### Vulnerability
This is the RSA common modulus attack.

If the same message `m` is encrypted with the same modulus `n` but two coprime
public exponents, then:

```text
c1 = m^e1 mod n
c2 = m^e2 mod n
```

Because `gcd(17, 13) = 1`, Bezout's identity gives integers `a` and `b` such
that:

```text
17a + 13b = 1
```

One valid pair is:

```text
a = -3
b = 4
```

So:

```text
m = c1^-3 * c2^4 mod n
```

The negative exponent means we use a modular inverse.

### Reproduction Script
```python
from math import gcd

n = 3233
e1 = 17
e2 = 13

c1 = [2923, 1313, 1313, 690, 855, 2271, 1632, 3179, 884, 3179, 1516]
c2 = [500, 772, 772, 1326, 794, 719, 2810, 2649, 2015, 2649, 454]

a, b = -3, 4

blocks = []
for x, y in zip(c1, c2):
    assert gcd(x, n) == 1
    m = (pow(pow(x, -1, n), -a, n) * pow(y, b, n)) % n
    blocks.append(m)

flag = ''.join(chr(m) for m in blocks)
print(blocks)
print(flag)
```

Output:

```text
[103, 101, 101, 107, 123, 109, 97, 105, 116, 105, 125]
geek{maiti}
```

## Flag
```text
geek{maiti}
```
