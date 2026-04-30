# Play Fair, Punch!

## Summary

The PNG is an IBM-style punched card. Reading the punched holes as Hollerith card code gives:

```text
OEGFDKGKRYRYOELTAGELGFPEWBF3973DCY
```

The challenge title then points to a Playfair cipher. Since normal 5x5 Playfair only uses letters, discard the digits:

```text
OEGFDKGKRYRYOELTAGELGFPEWBFDCY
```

Using the Playfair key `PUNCH`, the key square is:

```text
P U N C H
A B D E F
G I K L M
O Q R S T
V W X Y Z
```

Decrypting produces:

```text
SAMANDMISXSXSAMSPACEMACAQUEBYS
```

The `X` characters are Playfair filler inserted between the repeated `S` letters in `MISS SAM`, so the plaintext is:

```text
SAMANDMISSSAMSPACEMACAQUEBYS
```

Normalized as a JerseyCTF VI flag:

```text
jctfvi{sam_and_miss_sam_space_macaque_bys}
```

## Reproduction

1. Download `punch-card.png`.
2. Detect the black rectangular punched holes and map them to the 12 standard card rows:
   `12, 11, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9`.
3. Decode the Hollerith punches:
   `12+1..9 => A..I`, `11+1..9 => J..R`, `0+2..9 => S..Z`, single `0..9 => digits`.
4. Keep the two punches that overlap the printed left-side labels; without them the Playfair plaintext is missing the meaningful opening.
5. Strip digits and decrypt the letters as 5x5 Playfair with key `PUNCH`.
6. Remove Playfair filler `X` characters from `MISXSXSAM` to recover `MISSSAM`.

The decrypted clue identifies Sam and Miss Sam, two rhesus macaques used in Project Mercury space-flight testing.
