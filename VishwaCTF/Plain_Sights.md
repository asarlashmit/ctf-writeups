# Plain_Sights Writeup

## Challenge

- Name: `Plain_Sights`
- Type: `unknown`
- Ciphertext: `Jfr?Dirf?Fev?Qvcjxjxc`
- Flag format: `VishwaCTF{decoded_string}`

## Recon

The provided attachment contained:

```text
Intercepted transmission:
Jfr?Dirf?Fev?Qvcjxjxc

INTEL NOTE:
The operator was typing correct message.
```

The text is short, but it has a strong repeated-letter pattern:

- `Jfr` -> pattern `ABC`
- `Dirf` -> pattern `DECB`
- `Fev` -> pattern `FGH`
- `Qvcjxjxc` -> pattern `IJKLMLKJ`

That makes it suitable for a monoalphabetic substitution solve.

## Solve

Using the repeated-letter structure and ranking candidate English phrases, the best fit is:

```text
its_just_the_begining
```

This matches the ciphertext consistently with the substitution:

```text
j -> i
f -> t
r -> s
? -> _
d -> j
i -> u
e -> h
v -> e
q -> b
c -> g
x -> n
```

Applying that mapping:

```text
Jfr?Dirf?Fev?Qvcjxjxc
-> its_just_the_begining
```

## Flag

```text
VishwaCTF{its_just_the_begining}
```
