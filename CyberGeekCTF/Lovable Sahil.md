# Lovable Sahil

## Challenge Overview
- Event: CyberGeek CTF
- Category: Reverse engineering / obfuscation
- Final Flag: geek{unp4cked}

## Executive Summary
The attachment is a layered Python loader. The visible script does not contain the flag directly; it stores a Base64 string, decompresses it with zlib, and executes the result. Running the file blindly is unnecessary. The safe path is to unpack each layer statically and inspect the final password check.

## Solution Walkthrough
### Step 1: Inspect the wrapper
`onion.py` contains:

```python
import base64, zlib
payload = "..."
exec(zlib.decompress(base64.b64decode(payload)).decode())
```

Instead of executing it, the payload can be decoded manually:

```python
import base64, zlib
decoded = zlib.decompress(base64.b64decode(payload)).decode()
print(decoded)
```

This reveals a second Python layer.

### Step 2: Decode the XOR layer
The second layer stores:

```python
secret_key = 42
payload = b'...'
decrypted = "".join([chr(b ^ secret_key) for b in payload])
exec(decrypted)
```

Again, the important part is the transformation, not the `exec`. XORing each byte with `42` reveals the real checker:

```python
user_input = input("Enter master password: ")
targets = [103, 100, 103, 104, 127, 112, 104, 119, 60, 106, 97, 110, 104, 112]
if len(user_input) == 14:
    valid = True
    for i in range(len(user_input)):
        if (ord(user_input[i]) ^ i) != targets[i]:
            valid = False
            break
    if valid:
        print("Correct!!")
        exit()
print("Wrong password.")
```

### Step 3: Reverse the password check
The condition is:

```python
ord(user_input[i]) ^ i == targets[i]
```

XOR is reversible, so:

```python
ord(user_input[i]) = targets[i] ^ i
```

Solver:

```python
targets = [103, 100, 103, 104, 127, 112, 104, 119, 60, 106, 97, 110, 104, 112]
flag = ''.join(chr(v ^ i) for i, v in enumerate(targets))
print(flag)
```

Output:

```text
geek{unp4cked}
```

### Verification
Running the recovered checker with the decoded input prints `Correct!!`:

```bash
printf 'geek{unp4cked}\n' | python3 stage2.py
```

## Flag
```text
geek{unp4cked}
```
