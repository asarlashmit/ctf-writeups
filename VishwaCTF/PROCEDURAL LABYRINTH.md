# PROCEDURAL LABYRINTH

## Flag

`VishwaCTF{p4th_1s_th3_m3ss4g3_00_dfc81edd}`

## Summary

The service returns a large text maze with a start marker `>` and an exit marker `<`.
The hint says:

> The map is just the canvas. The true message is the journey you take to escape it.

That points to the solution path rather than the maze drawing itself.

## Solve

1. Fetch the maze from the challenge URL.
2. Parse the grid and find the unique path from `>` to `<` with BFS.
3. Keep only the horizontal moves from the solved route.
4. Interpret those moves as bits.

The correct mapping is:

- `R = 0`
- `L = 1`
- skip the first bit
- group into bytes

That byte stream contains the flag in plain ASCII.

## Extraction Logic

The route looks like a mostly vertical descent with sparse single-step left/right moves. Those left/right steps are the encoded data channel. Trying both bit polarities and byte offsets quickly reveals:

`VishwaCTF{p4th_1s_th3_m3ss4g3_00_dfc81edd}`

## Reproduction

Run:

```bash
python3 solve.py
```

The script downloads the maze, solves it, decodes the route, and prints the flag.
