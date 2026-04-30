# Quick Maths

Type: Crypto

## Summary

The challenge text was:

```text
WHATS 9+10???? DONT BE STUPID!!!
```

At first glance this looks like the meme answer `21`, but the extra text matters. In the meme, `21` is the stupid answer. The challenge explicitly says not to be stupid, so the intended interpretation is the literal arithmetic result: `19`.

SillyCTF's welcome challenge also states that the default flag format is `sillyCTF{}` unless a challenge says otherwise, and this challenge does not override that format.

## Solve

1. Read the prompt literally instead of following the meme bait.
2. Compute `9 + 10 = 19`.
3. Wrap it in the default event flag format.

## Flag

```text
sillyCTF{19}
```

## Note

The team account had already consumed the single allowed submission for this challenge, so the final value could not be re-verified live from the platform during this run. The writeup above reflects the strongest evidence-backed conclusion after exhausting the available local and API-based paths.
