# Coding Horror Writeup

Challenge type: web

## Summary

The login page had two real clues:

1. The HTML source contained a commented "banned password list".
2. The tall profile image was scrollable and contained a code-like hint showing that the first correct login attempt fails.

The valid credentials were:

- Username: `admin`
- Password: `sup3r_s3cur3_p@ssw0rd`

The catch was that the server intentionally rejected the first correct login in a fresh session, then accepted the same credentials on the second attempt with the same session cookie.

## Recon

Fetching the page source exposed the login form and the commented password list:

```html
<!--
BANNED PASSWORD LIST (DO NOT USE):
- password
- 123456
- admin123
- letmein
- welcome
- monkey
- qwerty123
- dragon
- master
- trustno1
- abc123
- password1
- sup3r_s3cur3_p@ssw0rd
...
-->
```

Direct POSTs to `/login` showed a useful difference:

- Most wrong passwords returned: `Wrong username or password.`
- `admin` + `sup3r_s3cur3_p@ssw0rd` returned: `Wrong login or password`

That strongly suggested `sup3r_s3cur3_p@ssw0rd` was special.

## Hidden clue

The "profile image" on the page was inside a small scrollable container. Inspecting the lower part of the image revealed a pseudo-code hint equivalent to:

```text
if (isPasswordCorrect && isFirstLoginAttempt) {
    Error("Wrong login or password")
}
```

That explained the strange response for the suspicious password.

## Exploit

Use one persistent session and send the same correct credentials twice:

```python
import requests

s = requests.Session()
url = "https://codinghorror.sillyctf.psuccso.org/login"
data = {"username": "admin", "password": "sup3r_s3cur3_p@ssw0rd"}

print(s.post(url, json=data).text)  # first correct attempt fails
print(s.post(url, json=data).text)  # second correct attempt returns the flag
```

Observed response:

```json
{"success":false,"message":"Wrong login or password"}
{"success":true,"message":"Login successful!","flag":"sillyCTF{th3_w0rst_bru73f0rc3_pr0t3ct10n_3v3r}"}
```

## Flag

`sillyCTF{th3_w0rst_bru73f0rc3_pr0t3ct10n_3v3r}`
