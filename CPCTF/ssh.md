# ssh Writeup

## Challenge
- Name: `ssh`
- Type: `unknown`

## Given
- SSH target: `ssh@133.88.122.244 -p 31219`
- Password: `cpctf2026`
- Hint: the flag is at `/flag/flag.txt`

## Recon
The service was a normal SSH login. After authenticating as user `ssh`, I checked the account and the flag path:

```bash
ssh ssh@133.88.122.244 -p 31219
# password: cpctf2026

id
ls -la /flag
cat /flag/flag.txt
```

The file permissions on `/flag/flag.txt` were world-readable:

```bash
-r--r--r-- 1 root root 27 Apr 12 03:54 /flag/flag.txt
```

That meant the intended solve was simply to log in and read the file.

## Flag

```text
CPCTF{w31c0m3_2_c11_w0r1d}
```
