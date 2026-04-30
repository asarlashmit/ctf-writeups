# Baldi Writeup

Challenge: `Baldi`

Category: `Pwn`

Flag: `sillyctf{b@LdI_iNt3G3R_0v3RFL0Wzz}`

## Summary

The service presents three math questions. The first two are normal:

- `2 + 2 = 4`
- `99 * 99 = 9801`

The third question is:

`What is x + |y| < 0, where x is > 0?`

At first glance this is impossible, because `|y|` should never be negative and `x` must be positive.

## Bug

The program uses signed 32-bit integers and computes `abs(y)` without handling the edge case `INT_MIN`.

For a 32-bit signed integer:

- `INT_MIN = -2147483648`
- `abs(INT_MIN)` overflows and stays negative on typical two's-complement systems

So if we choose:

- `x = 1`
- `y = -2147483648`

Then the program effectively evaluates:

`1 + abs(-2147483648) = 1 + (-2147483648) = -2147483647`

That is negative, so the impossible condition becomes true.

## Exploit

Answer the prompts with:

1. `4`
2. `9801`
3. `1`
4. `-2147483648`

The service responds with the flag.

## Reproduction

Run:

```bash
python3 solve_baldi.py
```
