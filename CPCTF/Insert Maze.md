# Insert Maze Writeup

## Challenge

- Name: `Insert Maze`
- Type: `PPC`
- Source: https://yukicoder.me/problems/no/3504
- Contest: `CPCTF 2026: PPC`

## Observation

The start is fixed at `(1, 1)` and the goal is fixed at `(H, W)`.

Without inserting anything, the absolute lower bound is the Manhattan distance:

```text
H + W - 2
```

Every inserted row or column shifts the goal one cell farther away, so using one useful insertion raises the lower bound to:

```text
H + W - 1
```

Also, any maze is always solvable in `H + W` moves: insert one all-`.` column next to the start side and one all-`.` row next to the goal side, then travel through those two corridors.

Therefore the answer is always one of:

```text
H + W - 2
H + W - 1
H + W
```

## Solve

First compute monotone reachability using only moves right and down:

- `start[i][j]`: reachable from `S` without using insertions.
- `goal[i][j]`: can reach `G` without using insertions.

If `start[H-1][W-1]` is true, the answer is `H + W - 2`.

For one inserted horizontal row between rows `r` and `r+1`, a shortest path can:

1. reach some cell `(r, a)` from `S`,
2. enter the inserted row,
3. move right along it,
4. leave at some cell `(r+1, b)`,
5. continue to `G`.

This is possible iff there is a reachable entry column `a` and a goal-reachable exit column `b` with `a <= b`.

The vertical inserted-column case is symmetric: for a column inserted between `c` and `c+1`, we need a reachable entry row `a` and goal-reachable exit row `b` with `a <= b`.

If either one-insert condition holds, the answer is `H + W - 1`; otherwise it is `H + W`.

## Reference Solution

File: [solution.py](/home/kali/ctf-orc/ctf/insert-maze/solution.py)

## Verification

The solver matches all official samples:

```text
sample 1 -> 5
sample 2 -> 2
sample 3 -> 8
```

I also verified it against an exhaustive brute-force solver for all small grids up to `3 x 4` and random grids up to `5 x 5`.

Finally, I submitted the solver to yukicoder:

- Submission: `#1158466`
- Result: `AC`

After AC, the hidden editorial became visible and contained the CPCTF flag.

## Flag

`CPCTF{Y0u_4r3_4_sUperHum4n11}`
