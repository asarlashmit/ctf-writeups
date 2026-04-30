# Wavesnew Writeup

`waves.png` is a rendered matplotlib plot. The useful part is not the anti-aliased curve itself, but the red marker dots placed on top of it.

## Recon

- The image is `2258x818`.
- Pure red pixels (`#cc0000`) form 23 connected components.
- Their x-centers are evenly spaced, so each dot represents one character position.

## Extraction

1. Isolate only the pure red pixels.
2. Compute connected components and keep the dot-sized components.
3. Sort the dot centers from left to right.
4. Fit the y-coordinates onto 26 evenly spaced levels.
5. Read those levels as `A-Z` from top to bottom.

That produces:

```text
AQZABADORGIHJTADOWGVOIQ
```

## Flag

```text
VishwaCTF{AQZABADORGIHJTADOWGVOIQ}
```

## Reproduction

Run:

```bash
python solve.py
```
