# Sum of Prod of Root

## Challenge

- Source: `yukicoder No.3505`
- Type: PPC
- Modulus: `998244353`

Given `1 <= N <= 10^18`, compute:

```text
sum_{i=1}^N prod_{k=1}^infinity floor(i^(1/k)) mod 998244353
```

## Observation

The first two factors are:

```text
floor(i^(1/1)) * floor(i^(1/2)) = i * floor(sqrt(i))
```

The remaining part

```text
H(i) = prod_{k=3}^infinity floor(i^(1/k))
```

changes only when `i` reaches a perfect power `a^k` with `k >= 3`.

Up to `10^18`, the number of such events is about:

```text
N^(1/3) + N^(1/4) + ... ~= 10^6
```

So we can sweep those event points and process each interval where `H(i)` is constant.

## Summing The Noisy Part

For an interval `[L, R]` where `H(i) = A`, the contribution is:

```text
A * sum_{i=L}^R i * floor(sqrt(i))
```

Define:

```text
F(n) = sum_{i=1}^n i * floor(sqrt(i))
```

For `m = floor(sqrt(n))`, full square-root blocks are:

```text
i in [q^2, (q+1)^2 - 1], floor(sqrt(i)) = q
```

Their contribution is:

```text
q * sum_{i=q^2}^{(q+1)^2-1} i
= q^2(q+1)(2q+1)
= 2q^4 + 3q^3 + q^2
```

Thus `F(n)` is computed in `O(1)` with standard power sums up to degree 4, plus the final partial block.

## Algorithm

1. Generate all events `(a^k, k, a)` for `k >= 3` and `a^k <= N`.
2. Sort events by value.
3. Sweep from left to right while maintaining:
   - `root[k] = floor(i^(1/k))`
   - `H = product root[k]` for `k >= 3`
4. For each constant interval `[L, R]`, add:

```text
H * (F(R) - F(L-1))
```

All arithmetic is modulo `998244353`.

## Solver

The accepted C++17 solver is in:

```text
solve.cpp
```

Verification:

- Sample `N=6` gives `36`.
- Brute force matched for small and random cases.
- Local maximum test `N=1000000000000000000` finished in about `1.36s`.
- yukicoder submission `#1158530`: `AC`.

## Flag

```text
CPCTF{c0ns74n7_4lmos7_3v3rywh3r3}
```
