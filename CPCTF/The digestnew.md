# The digestnew

## Flag

`CPCTF{d1g3st_4uth_15_4_ch4ll3ng3}`

## Summary

The capture contains an HTTP Digest authentication exchange. Digest auth does not send the
password directly, but the captured `Authorization` header includes enough values to verify
password guesses offline.

Captured values:

- username: `cpctf`
- realm: `Restricted`
- nonce: `+pHR2klPBgA=dea9c5d3f34f861b03f0be19a41069cf29603de5`
- method: `GET`
- uri: `/`
- qop: `auth`
- nc: `00000001`
- cnonce: `1afdf6a5de6ae0bc`
- response: `b71427f528886528c5144cd259a83d97`

For HTTP Digest MD5:

```text
H(A1) = MD5(username:realm:password)
H(A2) = MD5(method:uri)
response = MD5(H(A1):nonce:nc:cnonce:qop:H(A2))
```

The challenge states that the password is exactly eight digits, so the search space is only
`00000000` through `99999999`. Brute forcing those values locally avoids hitting the server
rate limit.

The recovered password was:

```text
37512859
```

Using it against the updated HTTPS endpoint:

```bash
curl --digest -u 'cpctf:37512859' https://digest.web.cpctf.space/
```

returned the flag page containing:

```text
CPCTF{d1g3st_4uth_15_4_ch4ll3ng3}
```
