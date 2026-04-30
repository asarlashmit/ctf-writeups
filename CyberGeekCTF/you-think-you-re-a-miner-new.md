# You Think You're a Miner?

## Challenge Overview
- Event: CyberGeek CTF
- Category: Misc / crypto
- Final Flag: geek{6342}

## Executive Summary
The challenge provided a screenshot of a simple blockchain mining form. The visible fields were:

- Block number: `1`
- Current nonce: `49051`
- Data: `you think you can mine?`
- Hash: `b4dd1133c58dd0cf8acd0445f1935d610d7e81df16b8082acb596d7f95674f01`

The target was to find the nonce produced by mining the displayed block until
the hash begins with `0000`.

## Solution Walkthrough
### Hash Format
I first used the screenshot hash as a test vector to identify how the block was serialized.

```python
import hashlib

hashlib.sha256(b"149051you think you can mine?").hexdigest()
```

This produced:

```text
b4dd1133c58dd0cf8acd0445f1935d610d7e81df16b8082acb596d7f95674f01
```

So the hash input is:

```text
<block_number><nonce><data>
```

For this challenge:

```text
1 + nonce + "you think you can mine?"
```

### Mining
I brute-forced decimal nonce values and checked:

```python
sha256(("1" + str(nonce) + "you think you can mine?").encode()).hexdigest()
```

The challenge validator appears to expect the first nonce found from a normal
zero-based brute force, not the first nonce after the screenshot's displayed
value. The first valid nonce starting from `0` was:

```text
6342
```

Verification:

```python
import hashlib

nonce = "6342"
msg = "1" + nonce + "you think you can mine?"
h = hashlib.sha256(msg.encode()).hexdigest()

print(h)
print(h.startswith("0000"))
```

Output:

```text
0000291b503d18b1b226acfa207d04eebcebabf00d85fca6b72a4c39244f51f3
True
```

## Flag
```text
geek{6342}
```
