# Messed Downnext Writeup

Challenge: `Messed Downnext`  
Category: `rev`

## Flag

`VishwaCTF{h3y_g3n1u5_w45_7h15_l177l3_h4rd3r_7h15_71m3}`

## Summary

The visible SHA-looking path is a decoy.

The real solve uses two hidden stages:

1. A dead-code branch family entered from `0x7804c` writes one of seven encrypted 8-byte data chunks to `[rsp+0x318 + index*8]`.
2. Another hidden stage entered from `0x6a2cb` writes seven 8-byte key chunks to `[rsp+0x258 + index*8]`.

XORing aligned qwords from the two arrays reveals the flag. The only nontrivial step is choosing the correct order of the seven data chunks.

## Recon

Normal execution never validates the input. It only prints the prompt, echoes the input, and exits. A stack dump also shows the hidden message:

```text
Security is Compromised!
Need to add another stage :)
```

That message is the hint that the visible runtime path is not the full challenge.

## Hidden Data Stage

Starting from the seed snapshot at `0x65646` and forcing execution into `0x7804c` reaches a compare chain on `[rsp+0x4f0]`.

The seven selector values are:

```text
0xa0e63840
0x1a198f53
0x7d6acb81
0x6627735d
0x67f0eeb3
0x16a444f0
0xe5740449
```

Forcing each selector produces one qword at `[rsp+0x318]`.

## Hidden Key Stage

Replaying the hidden serializer state and then entering `0x6a2cb` fills seven qwords starting at `[rsp+0x258]`.

Those key qwords are:

```text
0xaec44c4cccbd3678
0xc4418627a3c83fd7
0x77ab5ed77e91f7c7
0x792a652f99257525
0x87cbf5334e250c38
0x56bfb972fe6c672d
0x0000e40c87727421
```

## Recovery

Brute-forcing all `7!` permutations of the data qwords against the seven key qwords yields exactly one ASCII string matching the flag format:

```text
VishwaCTF{h3y_g3n1u5_w45_7h15_l177l3_h4rd3r_7h15_71m3}
```

## Solve

The solver in [`solve.py`](/home/kali/ctf-orc/ctf/messed-downnext/solve.py) automates:

- the hidden seed snapshot
- the data-branch forcing
- the key-array extraction
- the final permutation search

Run:

```bash
./.venv/bin/python solve.py
```

Output:

```text
VishwaCTF{h3y_g3n1u5_w45_7h15_l177l3_h4rd3r_7h15_71m3}
```
