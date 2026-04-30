# Plain_Sightsnext Writeup

## Challenge

- Name: `Plain_Sightsnext`
- Ciphertext: `Jfr?Dirf?Fev?Qvcjxjxc`
- Flag format: `VishwaCTF{decoded_string}`

## Recon

The attachment contained:

```text
Intercepted transmission:
Jfr?Dirf?Fev?Qvcjxjxc

INTEL NOTE:
The operator was typing correct message.
```

The repeated-letter structure is strong enough to recover a likely phrase by substitution:

- `Jfr` -> `its`
- `Dirf` -> `just`
- `Fev` -> `the`
- `Qvcjxjxc` -> `begining`

That gives:

```text
its_just_the_begining
```

## Fixing The Last Word

That result is one letter off from the obvious English phrase:

```text
its_just_the_beginning
```

The challenge note is the key here: the operator was typing the correct message. That means the plaintext should be the correctly spelled phrase, not the misspelled `begining`.

The only discrepancy is one missing repeated letter in the final word:

```text
beginning
```

Under the recovered substitution, that last word would encode as:

```text
qvcjxxjxc
```

But the intercepted text contains:

```text
qvcjxjxc
```

So the capture is consistent with one repeated encoded character being dropped/collapsed in transmission or logging.

## Flag

```text
VishwaCTF{its_just_the_beginning}
```
