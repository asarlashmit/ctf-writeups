# punnnnish

## Challenge Overview
- Event: CyberGeek CTF
- Category: forensics / git
- Final Flag: geek{t1m3_tr4v3l_c0mm1t5}

## Executive Summary
The repository content only contains mundane updates to `project.txt`, but each commit timestamp is carefully spaced. Consecutive commits are almost exactly one hour apart, with a small extra offset. Subtracting `3600` seconds from each timestamp delta produces ASCII byte values.

## Solution Walkthrough
### Recon
After extracting `punish.zip`, the archive contained one Git repository:

```bash
unzip -q punish.zip
cd time_repo
git log --format='%H %at %aI %s' --reverse
```

There was only one branch, no tags, and a single text file receiving one-line updates. The commit messages and file contents did not contain the secret.

### Exploit
List commit times in chronological order, compute each gap, subtract one hour, and decode the remaining values as ASCII:

```bash
python3 - <<'PY'
import subprocess

times = [
    int(x)
    for x in subprocess.check_output(
        ['git', 'log', '--format=%at', '--reverse'],
        text=True,
    ).splitlines()
]

deltas = [b - a for a, b in zip(times, times[1:])]
values = [delta - 3600 for delta in deltas]
print(''.join(map(chr, values)))
PY
```

The decoded byte values are:

```text
103 101 101 107 123 116 49 109 51 95 116 114 52 118 51 108 95 99 48 109 109 49 116 53 125
```

Decoded as ASCII:

```text
geek{t1m3_tr4v3l_c0mm1t5}
```

## Flag
```text
geek{t1m3_tr4v3l_c0mm1t5}
```
