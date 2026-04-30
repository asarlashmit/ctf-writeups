# Sign up for traP Writeup

## Challenge

- Name: `Sign up for traP`
- Type: `programming / validation`
- Source: https://yukicoder.me/problems/13201

## Recon

This is not a traditional binary, web, or crypto target. It is a small competitive-programming problem from yukicoder.

The task is to decide whether the input string can be registered as a `traQ ID`. The required conditions are:

1. The length is between `1` and `32` inclusive.
2. Every character is in `[A-Z][a-z][0-9]_-`.
3. The first and last characters are not `_` or `-`.

If all rules pass, print `200`. Otherwise, print `400`.

## Solve Strategy

The cleanest solve is a direct validator:

1. Reject strings shorter than `1` or longer than `32`.
2. Scan each character and reject anything outside the allowed alphabet.
3. Reject if `S[0]` or `S[-1]` is `_` or `-`.
4. Print `200` on success, else `400`.

I used explicit `if` and `for` checks instead of a regex so the implementation follows the statement literally.

## Reference Solution

File: [solution.py](/home/kali/ctf-orc/ctf/sign-up-for-trap/solution.py)

```python
import sys


def main() -> None:
    s = sys.stdin.readline().rstrip("\n")
    ok = True

    if len(s) < 1 or len(s) > 32:
        ok = False

    allowed = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789_-"

    if ok:
        for ch in s:
            found = False
            for candidate in allowed:
                if ch == candidate:
                    found = True
                    break
            if not found:
                ok = False
                break

    if ok:
        if s[0] == "_" or s[0] == "-" or s[-1] == "_" or s[-1] == "-":
            ok = False

    print(200 if ok else 400)


if __name__ == "__main__":
    main()
```

## Verification

Tested cases:

- `zoi_dayo` -> `200`
- `CPCTF{dummy}` -> `400`
- `_abc` -> `400`
- `abc-` -> `400`
- `A_b-9` -> `200`
- `a * 32` -> `200`
- `a * 33` -> `400`

## Flag Retrieval

This problem is part of `CPCTF 2026: PPC`. The flag is not on the public statement page and `CPCTF{dummy}` is only a sample invalid input.

The actual solve path is:

1. Submit a correct program to yukicoder for problem `13201`.
2. Wait for the submission to become `AC`.
3. Open the editorial page at `https://yukicoder.me/problems/no/3497/editorial`.
4. The editorial contains a dedicated `フラグ` block with the real CPCTF flag.

## Flag

`CPCTF{s10w1y_bu7_sure1y}`
