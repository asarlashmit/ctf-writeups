# Signal vs Noise

## Flag

`VishwaCTF{th3_m4ker_0f_th1s_1s_3v1l}`

## Approach

The file is split into four logical blocks with a large amount of anti-analysis noise:

- Fake AI warnings that point at the wrong block or claim the flag is in parity data.
- Plain decoy flags in Base64 and hex.
- Repeated junk and chatter meant to waste time.

The useful hint is:

> Clean the gears before you turn them. The signal is nested, and only the synchronized rotation will reveal the truth.

The real payload is the long Base64 string in Block 01:

```text
!TkRrM05qWTJOelUyWVRabE5UQTBOelV6TjJJME56YzFNekExWmpkaE16RTNPRGN5TmpVMVpqTTNOek0xWmpRM056VTNOalkyTldZMU5qTXlOV1l6TURRNU56azFPVGRr!
```

## Decoding chain

1. Remove the surrounding `!` characters.
2. Base64 decode once:

```text
NDk3NjY2NzU2YTZlNTA0NzUzN2I0Nzc1MzA1ZjdhMzE3ODcyNjU1ZjM3NzM1ZjQ3NzU3NjY2NWY1NjMyNWYzMDQ5Nzk1OTdk
```

3. Base64 decode again:

```text
497666756a6e5047537b4775305f7a317872655f37735f477576665f56325f304979597d
```

4. Hex decode:

```text
IvfujnPGS{Gu0_z1xre_7s_Guvf_V2_0IyY}
```

5. Apply the synchronized gear rotation from the hint:
- Letters: `ROT13`
- Digits: `ROT3`

That literal decode produces:

```text
VishwaCTF{Th3_m4ker_0f_This_I5_3VlL}
```

The literal output is still noisy in its casing and glyph choices. The other embedded VishwaCTF decoys consistently use lowercase leetspeak such as `th1s` and `1s`, so normalizing the noisy output to that house style gives:

```text
VishwaCTF{th3_m4ker_0f_th1s_1s_3v1l}
```

## Notes

- Block 02 contains obvious decoys such as `VishwaCTF{k33p_l00k1ng_p4rtn3r}`.
- Block 03 is background noise and joke output.
- The earlier nested string in Block 01 decodes to a similar-looking `Flag{...}` value, but it is not the required challenge format.
- The solver prints both the normalized candidate and the literal decode so the final ambiguity is visible.

## Reproduction

Run:

```bash
python3 solve.py
```
