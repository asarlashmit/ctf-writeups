# Curve Whisperer

## Challenge Overview
- Event: CyberGeek CTF
- Category: Crypto / Elliptic Curves
- Prompt: Compute R = P + Q on y^2 = x^3 + 2x + 3 mod 97 for P = (3, 6) and Q = (80, 10).
- Final Flag: geek{80, 87}

## Executive Summary
This is a direct elliptic-curve point addition exercise over F97. Compute the slope with a modular inverse, then apply the standard formulas for x3 and y3.

## Solution Walkthrough
### Solution
For two distinct elliptic-curve points, the slope is:

```text
m = (y2 - y1) / (x2 - x1) mod p
```

Here:

```text
m = (10 - 6) / (80 - 3) mod 97
m = 4 / 77 mod 97
```

The modular inverse of `77 mod 97` is `63`, since:

```text
77 * 63 = 4851 = 1 mod 97
```

So:

```text
m = 4 * 63 mod 97
m = 252 mod 97
m = 58
```

Now compute the resulting coordinates:

```text
Rx = m^2 - x1 - x2 mod 97
Rx = 58^2 - 3 - 80 mod 97
Rx = 3281 mod 97
Rx = 80
```

```text
Ry = m * (x1 - Rx) - y1 mod 97
Ry = 58 * (3 - 80) - 6 mod 97
Ry = -4472 mod 97
Ry = 87
```

Therefore:

```text
R = (80, 87)
```

The platform checker expects the same coordinates with a space after the comma inside the flag.

## Flag
```text
geek{80, 87}
```
