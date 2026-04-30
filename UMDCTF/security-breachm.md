# security-breachm writeup

Flag: `UMDCTF{cr1m3_p4ys_br34ch_m0re}`

## Summary

The provided source strongly hints at a BREACH-style compression oracle:

- `/api/suggestions` lets an attacker control `latest_suggestion`
- `/api/dashboard` reflects both `filter` and `flag`
- the admin repeatedly requests `/api/dashboard` with compression enabled

That is real, but it is not the easiest bug.

The actual faster exploit is in `admin.py`:

```python
def build_ssl_context() -> ssl.SSLContext:
    ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
    ctx.check_hostname = False
    ctx.verify_mode = ssl.CERT_NONE
    return ctx
```

The admin client disables all TLS certificate verification. Since the challenge gives us a shell on the same internal LAN (`10.0.0.3`), we can impersonate the HTTPS server at `10.0.0.2`, steal the admin credentials, then log into the real server ourselves and read the flag directly.

## Recon

From the remote shell:

- `10.0.0.1` = admin box
- `10.0.0.2` = HTTPS server
- `10.0.0.3` = attacker box

The note also confirms the admin is polling `/api/dashboard` continuously.

## Exploit

1. Add `10.0.0.2/32` to loopback on the attacker host so we can locally bind that address.
2. Start a fake HTTPS server on `10.0.0.2:443` with a throwaway self-signed cert.
3. Poison the admin’s ARP cache so `10.0.0.2` resolves to our MAC.
4. Wait for the admin client to reconnect.
5. Capture the `POST /login` credentials from the admin.
6. Shut down the fake server and remove the local `10.0.0.2` address.
7. Connect to the real `10.0.0.2`, log in with the stolen creds, and request `/api/dashboard` without `Accept-Encoding` so the response is uncompressed.

Because the admin accepts any certificate, the MITM succeeds immediately.

## Result

The stolen credentials were:

- user: `admin`
- pass: `KSr7pg9yXvV_gzr1M61mBr8EL8xUD-4w`

Fetching the real dashboard returned:

```json
{"filter":"","flag":"UMDCTF{cr1m3_p4ys_br34ch_m0re}"}
```

## Files

- Solver: [solver.py](/home/kali/ctf-orc/ctf/security-breachm/solver.py)
- Writeup: [writeup.md](/home/kali/ctf-orc/ctf/security-breachm/writeup.md)
