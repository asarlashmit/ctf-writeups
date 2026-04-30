# TruType

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics / Steganography
- Artifact(s): 01-01-000.ttf
- Prompt: The challenge gives a TrueType font and hints at people exchanging font files to hide data. That points toward font metadata, glyph names, or OpenType layout tables rather than normal text extraction.
- Final Flag: geek{l1g4tur3_spy}

## Executive Summary
The font abuses OpenType GSUB ligatures as a substitution alphabet. Dumping the custom ligature table and reading its outputs spells the flag.

## Solution Walkthrough
### Recon
Running `file` was misleading:

```bash
file 01-01-000.ttf
```

Output:

```text
01-01-000.ttf: SIMH tape data
```

The file is still parseable as a TrueType/OpenType SFNT font. FontTools can list the tables:

```bash
python3 -m fontTools.ttx -l 01-01-000.ttf
```

Important output:

```text
tag     length    offset
----  --------  --------
GSUB       270    297136
...
```

The interesting part is the added `GSUB` table. `GSUB` is the OpenType glyph substitution table; it is commonly used for ligatures, where multiple input glyphs are rendered as one output glyph.

Dumping the font to XML makes the table easy to read:

```bash
python3 -m fontTools.ttx -o 01-01-000.ttx 01-01-000.ttf
rg -n "GSUB|Ligature" 01-01-000.ttx
```

The font contains a `liga` feature with unusual two-character substitutions.

### Hidden Channel
The `GSUB` ligature table maps pairs of letters to single output glyphs:

```text
fl -> g
fi -> e
ff -> e
fp -> k
st -> {
sp -> l
sk -> 1
ct -> g
ck -> 4
th -> t
qu -> u
wr -> r
ph -> 3
wh -> _
ch -> s
ao -> p
gh -> y
ng -> }
```

These are not normal ligatures. They are a substitution alphabet. If a message is typed with the left-hand pairs and rendered using this font, the viewer sees the right-hand characters.

### Solve
The known flag prefix `geek{` is produced by:

```text
fl fi ff fp st -> geek{
```

After that, the remaining custom ligature outputs can be arranged into a clue-matching phrase:

```text
l 1 g 4 t u r 3 _ s p y }
```

That reads as:

```text
l1g4tur3_spy}
```

So the full ligature input string is:

```text
flfifffpstspskctckthquwrphwhchaoghng
```

Applying the font substitutions:

```text
fl fi ff fp st sp sk ct ck th qu wr ph wh ch ao gh ng
g  e  e  k  {  l  1  g  4  t  u  r  3  _  s  p  y  }
```

Verification script:

```bash
python3 - <<'PY'
from fontTools.ttLib import TTFont

font = TTFont("01-01-000.ttf")
glyph_to_char = {
    "one": "1",
    "three": "3",
    "four": "4",
    "braceleft": "{",
    "braceright": "}",
    "underscore": "_",
}

rules = {}
for lookup in font["GSUB"].table.LookupList.Lookup:
    if lookup.LookupType != 4:
        continue
    for subtable in lookup.SubTable:
        for first, ligatures in subtable.ligatures.items():
            for lig in ligatures:
                pair = first + "".join(lig.Component)
                output = glyph_to_char.get(lig.LigGlyph, lig.LigGlyph)
                rules[pair] = output

pairs = [
    "fl", "fi", "ff", "fp", "st", "sp", "sk", "ct", "ck",
    "th", "qu", "wr", "ph", "wh", "ch", "ao", "gh", "ng",
]

print("".join(rules[pair] for pair in pairs))
PY
```

Output:

```text
geek{l1g4tur3_spy}
```

## Flag
```text
geek{l1g4tur3_spy}
```
