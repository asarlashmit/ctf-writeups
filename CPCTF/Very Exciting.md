# Very Exciting Writeup

## Challenge

- Name: `Very Exciting`
- Type: `crypto`

## Recon

The remote service prints:

- a random-looking IV
- the encrypted flag
- an encryption oracle where we choose arbitrary plaintext
- an encryption oracle where we choose arbitrary IV

The key observation is that the oracle lets us reuse the exact same IV that was used for the flag encryption.

## Analysis

This behaves like a stream-style construction:

- `ciphertext = plaintext XOR keystream`
- the keystream depends on the IV

If the same IV is reused, the same keystream is reused.

So if we send:

- plaintext = all zero bytes
- IV = the printed `exciting_iv`
- length = flag ciphertext length

then the returned ciphertext is just the keystream itself, because:

`0 XOR keystream = keystream`

Once we have that keystream, we recover the flag with:

`flag = encrypted_flag XOR keystream`

## Exploit

File: [solution.py](/home/kali/ctf-orc/ctf/very-exciting/solution.py)

```python
#!/usr/bin/env python3
import re
import socket


HOST = "133.88.122.244"
PORT = 32007


def recv_until(sock: socket.socket, marker: bytes) -> bytes:
    data = b""
    while marker not in data:
        chunk = sock.recv(4096)
        if not chunk:
            break
        data += chunk
    return data


with socket.create_connection((HOST, PORT)) as sock:
    sock.settimeout(3)

    banner = recv_until(sock, b"Enter your boring 'favorite' (Hex): ").decode()
    iv = re.search(r"exciting_iv I used!: ([0-9a-f]+)", banner).group(1)
    flag_ct = re.search(r"cool exciting_flag!!\\s*=> ([0-9a-f]+)", banner, re.S).group(1)

    sock.sendall(("00" * (len(flag_ct) // 2) + "\n").encode())
    recv_until(sock, b"Enter your own 'very_exciting' IV (Hex): ")
    sock.sendall((iv + "\n").encode())

    out = b""
    while True:
        try:
            chunk = sock.recv(4096)
            if not chunk:
                break
            out += chunk
        except socket.timeout:
            break

favorite_ct = re.search(r"=> ([0-9a-f]+)", out.decode()).group(1)
keystream = bytes.fromhex(favorite_ct)
flag = bytes(a ^ b for a, b in zip(bytes.fromhex(flag_ct), keystream))
print(flag.decode())
```

## Flag

`CPCTF{SAMe_01d_STReam_1s_A1WaYs_b0r1ng}`
