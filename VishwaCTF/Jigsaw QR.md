# Jigsaw QR Writeup

Challenge: `Jigsaw QR`

Flag: `VishwaCTF{t00_sm4rt_t0_us3_brut3_f0rc3}`

## Summary

The image looks like a `3 x 3` shuffle at first glance, but that is a trap.

The correct reconstruction was:

1. Download `shuffled_flag1.png`.
2. Inspect adjacent-row/adjacent-column discontinuities.
3. Notice strong cut lines at `x = 111, 223, 335` and `y = 111, 223, 335`.
4. Notice the last `2 px` on the right and bottom are just a black border.
5. Crop to `448 x 448` and split into a `4 x 4` grid of `112 x 112` tiles.

## Key Observations

- The real puzzle is `4 x 4`, not `3 x 3`.
- The outer border clues make the corners identifiable:
  - top-left: tile `2`
  - top-right: tile `13`
  - bottom-left: tile `0`
  - bottom-right: tile `9`
- Edge signatures also constrain the border pieces:
  - right column middle pieces: `3`, `6`
  - bottom row middle pieces: `8`, `10`
- That reduces the search space to a few hundred plausible layouts instead of brute-forcing `16!`.

## Final Tile Order

Using tile IDs from the cropped `4 x 4` image:

```text
 2  15   7  13
 5  14  12   6
11   4   1   3
 0   8  10   9
```

Reassembling the tiles in that order produces a valid QR code.

## Decoding

Decoding the reconstructed image with `pyzbar` yields:

```text
VishwaCTF{t00_sm4rt_t0_us3_brut3_f0rc3}
```

## Artifacts

- Reconstructed QR: `solved_qr.png`
- Reconstructed QR with tile labels: `solved_qr_labeled.png`
