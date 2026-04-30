# Modulo Equation Writeup

## Challenge

- Name: `Modulo Equation`
- Type: `PPC`
- Source: https://yukicoder.me/problems/13060

## Solve

We need the smallest positive integer `x` such that:

- `x mod A == B mod x`
- with `1 <= B < A <= 300`

The key observation is that any `x < A` cannot work:

- If `x < A`, then `x mod A = x`.
- But `B mod x < x`, so equality is impossible.

Also `x = A` does not work:

- `A mod A = 0`
- Since `A > B`, `B mod A = B > 0`

So every valid solution must satisfy `x > A`.  
For any such `x`, we have `B mod x = B`, so the condition becomes:

- `x mod A = B`

Hence every solution has the form:

- `x = kA + B` for some `k >= 1`

The minimum is therefore:

- `x = A + B`

That yields the full accepted solver:

File: [solution.py](/home/kali/ctf-orc/ctf/modulo-equation/solution.py)

```python
import sys


def main() -> None:
    a, b = map(int, sys.stdin.readline().split())
    print(a + b)


if __name__ == "__main__":
    main()
```

## Verification

I submitted the Python solution to yukicoder and got:

- Submission: `#1158190`
- Result: `AC`

After AC, the previously hidden editorial became visible. The editorial contains the CPCTF flag.

## Flag

`CPCTF{D1d_y0u_8ru73_f0rc3_17!?}`
