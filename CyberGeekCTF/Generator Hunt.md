# Generator Hunt

## Challenge Overview
- Event: CyberGeek CTF
- Category: Math / Group Theory
- Prompt: Identify the identity element and all generators from the given Cayley table.
- Final Flag: geek{b,d,a}

## Executive Summary
This is a Cayley-table warmup. Identify the identity from the row and column that leave every element unchanged, then test which elements generate the full group.

## Solution Walkthrough
### Solution
The identity element `e` must satisfy:

```text
e + x = x
x + e = x
```

for every element `x` in the group.

Row `a` returns every column unchanged:

```text
a + a = a
a + b = b
a + c = c
a + d = d
```

Column `a` also returns every row unchanged:

```text
a + a = a
b + a = b
c + a = c
d + a = d
```

So the identity is:

```text
a
```

Under strict group theory, generators are found by repeatedly applying the same element to itself, starting from the identity:

```text
a: a
b: b, c, d, a
c: c, a
d: d, c, b, a
```

That means the mathematical generators are:

```text
b, d
```

The strict mathematical flag would be:

```text
geek{b,d,a}
```

```

## Flag
```text
geek{b,d,a}
```
