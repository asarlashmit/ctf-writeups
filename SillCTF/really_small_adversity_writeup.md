# Really Small Adversity

Category: crypto

## Challenge

Ciphertext:

```text
16180705626253079377831812490700
```

Artifact:

`challenge.pub` was behind the provided short URL. The short URL resolved to a SharePoint folder, and the file was accessible after keeping the guest `FedAuth` cookie from the shared link.

## Recon

The shared folder contained a single file:

```text
challenge.pub
```

Its contents:

```pem
-----BEGIN PUBLIC KEY-----
MCkwDQYJKoZIhvcNAQEBBQADGAAwFQIOAP///////l///////CUCAwEAAQ==
-----END PUBLIC KEY-----
```

OpenSSL shows this is a tiny RSA public key:

```text
Public-Key: (104 bit)
Modulus: FFFFFFFFFFFE5FFFFFFFFFFC25
Exponent: 65537
```

This is immediately vulnerable because a 104-bit RSA modulus is trivial to factor.

## Solve

Convert the modulus and factor it:

```python
from sympy import factorint

n = int("FFFFFFFFFFFE5FFFFFFFFFFC25", 16)
print(factorint(n))
```

Result:

```python
{
    4503599627370449: 1,
    4503599627370517: 1,
}
```

So:

```text
p = 4503599627370449
q = 4503599627370517
e = 65537
c = 16180705626253079377831812490700
```

Recover the private exponent and decrypt:

```python
from Crypto.Util.number import inverse, long_to_bytes

p = 4503599627370449
q = 4503599627370517
n = p * q
e = 65537
c = 16180705626253079377831812490700

phi = (p - 1) * (q - 1)
d = inverse(e, phi)
m = pow(c, d, n)

print(long_to_bytes(m))
```

Output:

```text
b'R$A_FunNy_HA!'
```

## Flag

```text
sillyCTF{R$A_FunNy_HA!}
```
