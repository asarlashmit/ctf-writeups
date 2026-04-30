# Waves Writeup

## Recon

The challenge ships a single image: `waves.png`.

Basic checks ruled out the usual easy stego paths:

- valid PNG structure
- no appended payload
- no useful metadata beyond `Matplotlib`
- no hidden flag in `strings`, chunk layout, or obvious bit-planes

So the visible plot had to be the real signal.

## Key Observation

The smooth curve is decorative. The actual data is in the 23 red control points placed on that curve.

After isolating the red markers and sorting them left-to-right, the centers are:

```text
[(83, 224), (178, 578), (273, 769), (368, 230), (463, 247), (558, 223),
 (654, 289), (749, 540), (844, 610), (939, 367), (1034, 405), (1129, 376),
 (1224, 425), (1320, 648), (1415, 229), (1510, 285), (1605, 536), (1700, 703),
 (1795, 363), (1891, 688), (1986, 527), (2081, 406), (2176, 589)]
```

The important detail is that a direct linear mapping from the vertical range of
those points to alphabet indices `0..19` produces a readable plaintext.

Using:

```text
value = round((ymax - y) * 19 / (ymax - ymin))
char  = chr(ord('a') + value)
```

gives:

```text
[19, 7, 0, 19, 18, 19, 17, 8, 6, 14, 13, 14, 12, 4, 19, 17, 8, 2, 14, 3, 8, 13, 6]
```

which decodes to:

```text
thatstrigonometricoding
```

This is the only simple decode branch that yields coherent plaintext from the
point heights. The earlier factoradic/permutation approach was a false lead.
For the final marker, a naive centroid is slightly skewed because the dot touches
the end of the curve. Using the local distance-transform peak gives the correct
circle center and keeps the last letter as `g`.

## Flag

```text
VishwaCTF{thatstrigonometricoding}
```

## Reproduction

Run:

```bash
python3 solve.py
```
