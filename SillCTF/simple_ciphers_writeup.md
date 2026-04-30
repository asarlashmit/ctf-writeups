# Simple Ciphers Writeup

Type: Crypto

## Summary

The provided string:

```text
c2lsbHlDVEZ7d293X3RoYXRfd2FzX2Ffc2ltcGxlX3dheV90b19lbmNvZGVfdGV4dH0=
```

is valid Base64. Decoding it directly reveals the flag. The attached image was not needed for the solve.

## Solve

Decode the ciphertext:

```bash
printf '%s' 'c2lsbHlDVEZ7d293X3RoYXRfd2FzX2Ffc2ltcGxlX3dheV90b19lbmNvZGVfdGV4dH0=' | base64 -d
```

Output:

```text
sillyCTF{wow_that_was_a_simple_way_to_encode_text}
```

## Flag

```text
sillyCTF{wow_that_was_a_simple_way_to_encode_text}
```
