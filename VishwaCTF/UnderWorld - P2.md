# UnderWorld - P2 Writeup

Challenge type: forensics / Minecraft world analysis

## Summary

P2 uses the same Minecraft world as P1, but the flag is not stored in a container. The useful clue is the hanging sign from the visible structure:

- `Chall 2:`
- `Emreald`
- `Blocks are`
- `Beautiful`

The important hint is `emerald`. After parsing the world directly, the real payload is a second set of emerald blocks hidden far below the visible build. Rendering those hidden emerald coordinates as a top-down image reveals stacked text that reconstructs the flag.

## Recon

1. Extract the shared `UnderWorld` save.
2. Parse `level.dat` and region files.
3. Ignore the normal flat emerald floor at `y = -63`.
4. Scan for additional `minecraft:emerald_block` placements elsewhere in the world.

Key finding:

- Only 2 emerald blocks exist above ground at `y = 32`
- 454 extra emerald blocks exist at `y = -60`

Those `y = -60` emeralds are the hidden message.

## Extraction

Project the `y = -60` emerald block coordinates onto the X/Z plane and render them as a top-down image. The points form stacked text. Reading the rows from bottom to top gives:

- `Vishwa`
- `CTF{`
- `emerald`
- `_blocks`
- `}`

Combining the rows yields:

`VishwaCTF{emerald_blocks}`

## Notes

- The visible emerald blocks near the structure are decoys.
- The normal emerald floor is also a decoy; the message is the separate emerald layer at `y = -60`.
- The challenge description and sign both point toward `emerald blocks`, which matches the recovered hidden text.

## Flag

`VishwaCTF{emerald_blocks}`
