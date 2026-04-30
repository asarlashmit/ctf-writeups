# Digit Products 2 Writeup

## Challenge

- Name: `Digit Products 2`
- Type: `PPC / interactive`
- Source: https://yukicoder.me/problems/13166
- yukicoder No.: `3501`

## Recon

The directory did not contain local challenge files, so the target was the linked yukicoder interactive problem.

The judge hides an `N`-digit decimal integer `X`. A query chooses two different digit positions and returns the product of those two digits. At most `N` queries are allowed. We must either output the unique value of `X`, or output `-1` when `X` is not uniquely determinable.

The key statement detail is that the most significant digit is never `0`.

## Solve

Index digits from least significant to most significant, so the top digit is position `N - 1`.

First ask `N - 1` hub queries:

```text
? 0 N-1
? 1 N-1
...
? N-2 N-1
```

Because digit `N - 1` is nonzero, each response is zero exactly when that lower digit is zero. So these queries reveal the positions of every nonzero lower digit and the product of each with the top digit.

There are three cases:

1. Exactly one nonzero digit total:
   All hub products are zero. The top digit could be any value from `1` to `9`, so the answer is not unique.

2. Exactly two nonzero digits total:
   Only one product is informative. This is unique only when that product appears exactly once in the `1..9` multiplication table. The possible products are:

```text
1  -> 11
25 -> 55
49 -> 77
64 -> 88
81 -> 99
```

All other products have multiple valid digit factorizations, so output `-1`.

3. At least three nonzero digits total:
   Pick two nonzero lower positions `i` and `j` and ask one more query `? i j`.

If the hub products are:

```text
p_i = d_i * h
p_j = d_j * h
q   = d_i * d_j
```

then:

```text
h^2 = p_i * p_j / q
```

This recovers the top digit `h`, and every lower digit is then `p_i / h`. The total query count is `(N - 1) + 1 = N`.

## Reference Solution

File: [solution.py](/home/kali/ctf-orc/ctf/digit-products-2/solution.py)

## Verification

I verified the solver locally with:

- Exhaustive enumeration for all valid numbers through `N = 5`.
- A subprocess interactor that ran `solution.py` against random hidden values and checked the query protocol.

I then submitted the Python solution to yukicoder:

- Submission: `#1158454`
- Result: `AC`
- Test files: `sample * 1`, `other * 72`, all accepted.

After AC, the hidden editorial became visible and contained the CPCTF flag.

## Flag

`CPCTF{num63r_w1th_thr33_0r_m0r3_n0n_z3r0_d1g175_c4n_b3_d373rm1n3d_1n_0n1y_0n3_w4y}`
