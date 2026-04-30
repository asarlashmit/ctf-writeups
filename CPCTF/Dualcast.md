# Dualcast Writeup

## Challenge

- Name: `Dualcast`
- Type: `rev / crypto (very easy encoding)`

## Recon

The provided Python file was:

```python
from Crypto.Util.number import bytes_to_long
flag = "CPCTF{REDACTED}"
flag_bytes = flag.encode()
print(f"c = {bytes_to_long(flag_bytes)}")
```

The output file contained:

```text
c = 510812092313572375684202062709941424740135938555245927502061365582594139087652994941
```

## Analysis

`bytes_to_long()` does not encrypt anything. It only interprets a byte string as a big-endian integer.

So the given value `c` is just the flag bytes converted into an integer. To recover the flag, we reverse the process with `long_to_bytes()` or `int.to_bytes()`.

## Solve

```python
from Crypto.Util.number import long_to_bytes

c = 510812092313572375684202062709941424740135938555245927502061365582594139087652994941
print(long_to_bytes(c).decode())
```

## Flag

```text
CPCTF{wh47_7yp3_15_y0ur_477r1bu73?}
```
