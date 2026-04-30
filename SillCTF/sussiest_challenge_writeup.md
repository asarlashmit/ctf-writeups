# Sussiest Challenge

Type: OSINT

## Summary

The prompt was:

```text
Whats Amogus's Birthday. Put the format in sillyCTF{month_day}
```

This reads like a character-lore lookup rather than a software release date. Searching for `Amogus birthday` leads to the `Amogus Wiki` Fandom page for `Amogus`, which lists the character's birthday as `January 1st`.

With the requested flag format, that becomes:

```text
sillyCTF{january_1}
```

## Solve

Relevant source:

```text
https://amogus.fandom.com/wiki/Amogus
```

Key detail from the page:

```text
Birthday
January 1st
```

Convert that into the requested format:

```text
sillyCTF{january_1}
```

## Flag

```text
sillyCTF{january_1}
```
