# Can You Find the Flag

## Challenge Overview
- Event: CyberGeek CTF
- Category: Web / OSINT
- Prompt: No img / Can you find me??
- Target / Given: CTF7 - Can you find the flag
- Final Flag: geek{ctf_cybergeek}

## Executive Summary
The shortened URL is a decoy. The accepted flag comes from hex-decoding and reversing the extra YouTube query parameters rather than inspecting the landing page itself.

## Solution Walkthrough
### Recon
Opening the shortened URL:

```bash
curl -sI https://bit.ly/4vEUHZI
```

It redirects to:

```text
https://www.youtube.com/watch?v=dQw4w9WgXcQ&feature=share&si=3d3d77616c56&sp=325a79566d59&pp=354e32586d523359
```

The relevant parameter values are:

- `si = 3d3d77616c56`
- `sp = 325a79566d59`
- `pp = 354e32586d523359`

### Decode
First, decode each value from hex to ASCII:

- `si -> ==walV`
- `sp -> 2ZyVmY`
- `pp -> 5N2XmR3Y`

Concatenate them:

```text
==walV2ZyVmY5N2XmR3Y
```

Reverse the full string:

```text
Y3RmX2N5YmVyZ2Vlaw==
```

Base64-decode it:

```text
ctf_cybergeek
```

### Reproduction
One quick way to reproduce the full decode:

```bash
python3 - <<'PY'
from base64 import b64decode

parts = [
    "3d3d77616c56",
    "325a79566d59",
    "354e32586d523359",
]

decoded = ''.join(bytes.fromhex(p).decode() for p in parts)
reversed_text = decoded[::-1]
flag_text = b64decode(reversed_text).decode()

print(decoded)
print(reversed_text)
print(flag_text)
PY
```

Output:

```text
==walV2ZyVmY5N2XmR3Y
Y3RmX2N5YmVyZ2Vlaw==
ctf_cybergeek
```

## Flag
```text
geek{ctf_cybergeek}
```
