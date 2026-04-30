# Mirage

Category: Web

## Summary

The page at `https://mirage.web.cpctf.space/` is a static HTML page that prints many fake flag strings. The visible text is misleading because the page embeds a custom WOFF2 font named `Mirage` and applies it to every flag line.

The important mistake to avoid is trusting what the browser renders. The DOM text and the displayed text are different.

## Recon

The challenge metadata from the CPCTF API pointed to the real live URL:

- `https://cpctf.space/api/challenges/33826e3a-4161-4047-8ca3-716b413f9096`
- Challenge text: `https://mirage.web.cpctf.space/`

Fetching the page source showed:

- one inline script that appends 14 candidate flag strings
- one embedded base64 WOFF2 font
- no extra backend logic on the challenge host

The relevant DOM strings were:

```text
TGT7D{I_L0VE_M1KU_H4T5UN3}
CPCLD{Y0U_4R3_CL0S3_BU7_N0_C1G4R}
CNCKF{Y4K1N1KU_74B3741_D35U_N3}
TGTLD{THIS_15_A_DUmMY_FL4G!}
TGTTF{C0NGR4TULAT10N5_0N_Y0UR_ADM15510N}
CPCLD{N0T_A_FLAG_BUT_A_FR0G}
CNCKF{4DM1N_P455W0RD_15_123456}
TGTTF{1M_GE771NG_B0RED_W17H_7H15}
TGT7D{DAYBR34K_FR0NTL1N3}
TNTTF{LOREM_IPSUM_DOLOR_SIT_AMET}
CNCTF{1_4M_4_734P07_418}
WCWTF{M1KKUM1KUN1_S173Y4NY0}
CPCTF{CH4R4C73R5_4L50_D159U153}
WCWKF{CH0770_N4N1_1773RU_K4_W4K4R4N41}
```

## Font trick

I extracted the embedded font, rendered a glyph chart, and compared the codepoints with their visible shapes. Example substitutions:

- `T -> C`
- `G -> P`
- `L -> T`
- `D -> F`
- `H -> O`
- `S -> Y`
- `1 -> I`
- `4 -> E`
- `m -> R`

This explains why one rendered line looked like:

```text
CPCTF{COPY_IS_A_F1RST_STEP!}
```

but its underlying DOM text was actually:

```text
TGTLD{THIS_15_A_DUmMY_FL4G!}
```

That line is only a hint. The real answer is one of the DOM strings, not the visually rendered one.

## Submission

The first failed attempt was caused by using the wrong endpoint (`/api/code`). The current CPCTF frontend submits answers to:

```text
POST /api/challenges/<challenge_id>/answer
{"answer":"..."}
```

Submitting this DOM string to the correct endpoint solved the challenge:

```text
CPCTF{CH4R4C73R5_4L50_D159U153}
```

## Flag

```text
CPCTF{CH4R4C73R5_4L50_D159U153}
```
